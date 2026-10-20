/\*\*  
 \* PermissionsPolicy Pro \- W3C Permissions-Policy & RFC 8941 Structured Fields Engine  
 \* Handles feature catalog definitions, header parsing, structured syntax generation,  
 \* and legacy Feature-Policy backward translation.  
 \*/

const PermissionsEngine \= (function() {  
  'use strict';

  // Feature Categories & Metadata  
  const FEATURE\_CATALOG \= {  
    media: {  
      categoryName: 'Hardware & Media Capture',  
      description: 'Controls access to cameras, microphones, audio output, and display capture.',  
      features: {  
        'camera': { label: 'Camera Video Capture', defaultState: 'self', risk: 'HIGH' },  
        'microphone': { label: 'Microphone Audio Capture', defaultState: 'self', risk: 'HIGH' },  
        'display-capture': { label: 'Screen / Window Sharing', defaultState: 'none', risk: 'CRITICAL' },  
        'speaker-selection': { label: 'Audio Output Device Selection', defaultState: 'none', risk: 'LOW' },  
        'screen-wake-lock': { label: 'Screen Wake Lock API', defaultState: 'self', risk: 'LOW' }  
      }  
    },  
    sensors: {  
      categoryName: 'Geolocation & Motion Sensors',  
      description: 'Controls physical location tracking and motion sensor hardware.',  
      features: {  
        'geolocation': { label: 'High-Precision GPS Location', defaultState: 'none', risk: 'CRITICAL' },  
        'accelerometer': { label: 'Device Accelerometer (Motion)', defaultState: 'none', risk: 'MEDIUM' },  
        'gyroscope': { label: 'Device Gyroscope (Orientation)', defaultState: 'none', risk: 'MEDIUM' },  
        'magnetometer': { label: 'Device Magnetometer (Compass)', defaultState: 'none', risk: 'LOW' },  
        'ambient-light-sensor': { label: 'Ambient Light Photometer', defaultState: 'none', risk: 'LOW' }  
      }  
    },  
    hardware: {  
      categoryName: 'Direct Hardware & Bus Access',  
      description: 'Controls low-level physical interfaces (USB, Bluetooth, Serial, HID).',  
      features: {  
        'usb': { label: 'WebUSB Direct Hardware Access', defaultState: 'none', risk: 'CRITICAL' },  
        'bluetooth': { label: 'Web Bluetooth Peripheral Access', defaultState: 'none', risk: 'HIGH' },  
        'serial': { label: 'Web Serial COM Port Access', defaultState: 'none', risk: 'CRITICAL' },  
        'hid': { label: 'WebHID Human Interface Device', defaultState: 'none', risk: 'HIGH' },  
        'midi': { label: 'Web MIDI Device Interface', defaultState: 'none', risk: 'LOW' }  
      }  
    },  
    privacy: {  
      categoryName: 'Privacy, Monetization & Ads',  
      description: 'Controls payment execution, cross-site profiling, and Privacy Sandbox ad APIs.',  
      features: {  
        'payment': { label: 'Payment Request API (PCI DSS)', defaultState: 'self', risk: 'HIGH' },  
        'autoplay': { label: 'Media Autoplay with Audio', defaultState: 'self', risk: 'LOW' },  
        'fullscreen': { label: 'HTML5 Fullscreen API', defaultState: 'self', risk: 'LOW' },  
        'interest-cohort': { label: 'Google FLoC Cohort Profiling', defaultState: 'none', risk: 'HIGH' },  
        'browsing-topics': { label: 'Privacy Sandbox Topics API', defaultState: 'none', risk: 'HIGH' },  
        'run-ad-auction': { label: 'FLEDGE Protected Audience Auction', defaultState: 'none', risk: 'HIGH' },  
        'join-ad-group': { label: 'FLEDGE Interest Group Membership', defaultState: 'none', risk: 'MEDIUM' },  
        'identity-credentials-get': { label: 'FedCM Identity Credential API', defaultState: 'self', risk: 'MEDIUM' }  
      }  
    }  
  };

  /\*\*  
   \* Parse a W3C Permissions-Policy RFC 8941 header string  
   \* e.g. "camera=(self), geolocation=(), microphone=(\\"https\://trusted.com\\")"  
   \* @param {string} headerStr   
   \* @returns {object} structured parsed directives  
   \*/  
  function parsePermissionsPolicy(headerStr) {  
    if (\!headerStr || typeof headerStr \!== 'string') {  
      return { valid: false, error: 'Empty or non-string header', directives: {}, raw: '' };  
    }

    const trimmed \= headerStr.trim();  
    const directives \= {};  
    const errors \= \[\];  
    const warnings \= \[\];

    // Split on commas that are not inside quotes or parentheses  
    const tokens \= \[\];  
    let current \= '';  
    let inParen \= false;  
    let inQuote \= false;

    for (let i \= 0; i \< trimmed.length; i++) {  
      const c \= trimmed\[i\];  
      if (c \=== '"' && trimmed\[i \- 1\] \!== '\\\\') {  
        inQuote \= \!inQuote;  
        current \+= c;  
      } else if (c \=== '(' && \!inQuote) {  
        inParen \= true;  
        current \+= c;  
      } else if (c \=== ')' && \!inQuote) {  
        inParen \= false;  
        current \+= c;  
      } else if (c \=== ',' && \!inParen && \!inQuote) {  
        if (current.trim()) tokens.push(current.trim());  
        current \= '';  
      } else {  
        current \+= c;  
      }  
    }  
    if (current.trim()) tokens.push(current.trim());

    tokens.forEach(tok \=\> {  
      const eqIdx \= tok.indexOf('=');  
      if (eqIdx \=== \-1) {  
        errors.push(\`Invalid token syntax without equals sign: "\${tok}"\`);  
        return;  
      }

      const feature \= tok.slice(0, eqIdx).trim().toLowerCase();  
      const valStr \= tok.slice(eqIdx \+ 1).trim();

      // Check allowlist syntax  
      if (valStr \=== '\*') {  
        directives\[feature\] \= { type: 'wildcard', raw: '\*' };  
      } else if (valStr.startsWith('(') && valStr.endsWith(')')) {  
        const inner \= valStr.slice(1, \-1).trim();  
        if (\!inner) {  
          directives\[feature\] \= { type: 'none', raw: '()' };  
        } else {  
          // split inner tokens  
          const origins \= inner.split(/\\s+/).map(o \=\> {  
            if (o \=== 'self') return 'self';  
            if (o.startsWith('"') && o.endsWith('"')) {  
              return o.slice(1, \-1);  
            }  
            return o;  
          }).filter(Boolean);

          const hasSelf \= origins.includes('self');  
          const customOrigins \= origins.filter(o \=\> o \!== 'self');

          if (hasSelf && customOrigins.length \=== 0\) {  
            directives\[feature\] \= { type: 'self', raw: '(self)' };  
          } else {  
            directives\[feature\] \= { type: 'custom', origins, raw: valStr };  
          }  
        }  
      } else {  
        warnings.push(\`Non-standard RFC 8941 value for feature "\${feature}": "\${valStr}"\`);  
        directives\[feature\] \= { type: 'raw', raw: valStr };  
      }  
    });

    return {  
      valid: errors.length \=== 0,  
      errors,  
      warnings,  
      raw: trimmed,  
      directives  
    };  
  }

  /\*\*  
   \* Synthesize canonical Permissions-Policy and fallback Feature-Policy headers  
   \* @param {object} featureState \- map of featureName \-\> { type: 'none'|'self'|'wildcard'|'custom', customOrigins?: string\[\] }  
   \* @returns {object} { permissionsPolicy, featurePolicy, iframeAllow }  
   \*/  
  function synthesizeHeaders(featureState) {  
    const ppDirectives \= \[\];  
    const fpDirectives \= \[\];  
    const iframeAllowDirectives \= \[\];

    Object.keys(featureState).forEach(feat \=\> {  
      const conf \= featureState\[feat\];  
      const type \= conf.type || 'none';

      if (type \=== 'none') {  
        ppDirectives.push(\`\${feat}=()\`);  
        fpDirectives.push(\`\${feat} 'none'\`);  
      } else if (type \=== 'self') {  
        ppDirectives.push(\`\${feat}=(self)\`);  
        fpDirectives.push(\`\${feat} 'self'\`);  
        iframeAllowDirectives.push(feat);  
      } else if (type \=== 'wildcard') {  
        ppDirectives.push(\`\${feat}=\*\`);  
        fpDirectives.push(\`\${feat} \*\`);  
        iframeAllowDirectives.push(feat);  
      } else if (type \=== 'custom') {  
        const origins \= conf.origins || \[\];  
        const formattedOrigins \= origins.map(o \=\> o \=== 'self' ? 'self' : \`"\${o}"\`).join(' ');  
        ppDirectives.push(\`\${feat}=(\${formattedOrigins})\`);  
          
        const fpOrigins \= origins.map(o \=\> o \=== 'self' ? "'self'" : o).join(' ');  
        fpDirectives.push(\`\${feat} \${fpOrigins}\`);  
        iframeAllowDirectives.push(feat);  
      }  
    });

    const permissionsPolicy \= ppDirectives.join(', ');  
    const featurePolicy \= fpDirectives.join('; ');  
    const iframeAllow \= iframeAllowDirectives.join('; ');

    return {  
      permissionsPolicy,  
      featurePolicy,  
      iframeAllow,  
      totalFeaturesConfigured: Object.keys(featureState).length  
    };  
  }

  return {  
    FEATURE\_CATALOG,  
    parsePermissionsPolicy,  
    synthesizeHeaders  
  };  
})();

if (typeof module \!== 'undefined' && module.exports) {  
  module.exports \= PermissionsEngine;  
}  
