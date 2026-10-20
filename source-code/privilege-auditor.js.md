/\*\*  
 \* PermissionsPolicy Pro \- 10-Point Hardware Privilege & Iframe Security Auditor  
 \* Evaluates camera, mic, geolocation, WebUSB, motion sensors, FLoC/Topics,  
 \* Payment Request API (PCI DSS 4.0), and RFC 8941 syntax compliance.  
 \*/

const PrivilegeAuditor \= (function() {  
  'use strict';

  let Engine \= null;  
  if (typeof PermissionsEngine \!== 'undefined') {  
    Engine \= PermissionsEngine;  
  } else if (typeof require \!== 'undefined') {  
    Engine \= require('./permissions-engine.js');  
  }

  /\*\*  
   \* Run 10-point audit on Permissions-Policy header or feature state map  
   \* @param {string|object} input \- header string or feature state map  
   \* @param {object} \[options\]  
   \* @param {boolean} \[options.isEcommerce\] \- whether site processes payments (PCI DSS 4.0 scope)  
   \* @returns {object} Audit report with score, grade, checks, and actionable remediation steps  
   \*/  
  function audit(input, options \= {}) {  
    let parsed;  
    if (typeof input \=== 'string') {  
      parsed \= Engine.parsePermissionsPolicy(input);  
    } else if (typeof input \=== 'object' && input \!== null) {  
      if (input.directives) {  
        parsed \= input;  
      } else {  
        // synthesize temporary header and parse  
        const syn \= Engine.synthesizeHeaders(input);  
        parsed \= Engine.parsePermissionsPolicy(syn.permissionsPolicy);  
      }  
    } else {  
      parsed \= { valid: false, directives: {}, errors: \['Empty input'\], warnings: \[\] };  
    }

    const directives \= parsed.directives || {};  
    const checks \= \[\];  
    let score \= 0;  
    const criticalRisks \= \[\];  
    const remediationSteps \= \[\];  
    const isEcommerce \= options.isEcommerce \=== true;

    // Helper to get feature rule type  
    function getRule(feat) {  
      return directives\[feat\] ? directives\[feat\].type : 'unspecified';  
    }

    // \--- Check 1: Camera & Microphone Hardware Disarmament (15 pts) \---  
    const camRule \= getRule('camera');  
    const micRule \= getRule('microphone');

    if (camRule \=== 'wildcard' || micRule \=== 'wildcard') {  
      checks.push({  
        id: 'camera\_mic\_scope',  
        name: 'Critical Hardware Disarmament (Camera & Mic)',  
        passed: false,  
        points: 0,  
        maxPoints: 15,  
        status: 'FAIL',  
        summary: 'Fatal: Camera or Microphone delegated with wildcard (\*).',  
        details: 'Any embedded third-party iframe can activate user camera and microphone without domain restriction\!'  
      });  
      criticalRisks.push('CRITICAL SURVEILLANCE RISK: Wildcard (\*) on camera or microphone allows rogue iframes to record audio/video.');  
      remediationSteps.push('Restrict camera and microphone to (self) or explicit trusted origins, or disable entirely (=()).');  
    } else if (camRule \=== 'unspecified' && micRule \=== 'unspecified') {  
      checks.push({  
        id: 'camera\_mic\_scope',  
        name: 'Critical Hardware Disarmament (Camera & Mic)',  
        passed: false,  
        points: 4,  
        maxPoints: 15,  
        status: 'WARN',  
        summary: 'Camera and Microphone directives omitted.',  
        details: 'Browser defaults apply, allowing same-origin iframes to prompt users for recording permissions.'  
      });  
      score \+= 4;  
      remediationSteps.push('Explicitly configure camera=(self) and microphone=(self) or camera=(), microphone=().');  
    } else if ((camRule \=== 'none' || camRule \=== 'self' || camRule \=== 'custom') &&  
               (micRule \=== 'none' || micRule \=== 'self' || micRule \=== 'custom')) {  
      checks.push({  
        id: 'camera\_mic\_scope',  
        name: 'Critical Hardware Disarmament (Camera & Mic)',  
        passed: true,  
        points: 15,  
        maxPoints: 15,  
        status: 'PASS',  
        summary: 'Camera and Microphone strictly scoped.',  
        details: \`camera: \${directives.camera ? directives.camera.raw : 'none'}, microphone: \${directives.microphone ? directives.microphone.raw : 'none'}\`  
      });  
      score \+= 15;  
    } else {  
      checks.push({  
        id: 'camera\_mic\_scope',  
        name: 'Critical Hardware Disarmament (Camera & Mic)',  
        passed: true,  
        points: 10,  
        maxPoints: 15,  
        status: 'PASS',  
        summary: 'Partial camera/mic restriction applied.',  
        details: 'One or more directives active.'  
      });  
      score \+= 10;  
    }

    // \--- Check 2: Geolocation Surveillance Defense (15 pts) \---  
    const geoRule \= getRule('geolocation');  
    if (geoRule \=== 'wildcard') {  
      checks.push({  
        id: 'geolocation\_defense',  
        name: 'Geolocation Surveillance Defense',  
        passed: false,  
        points: 0,  
        maxPoints: 15,  
        status: 'FAIL',  
        summary: 'Severe Privacy Violation: Geolocation delegated to wildcard (\*).',  
        details: 'Embedded ad iframes and third-party widgets can silently harvest high-precision GPS coordinates.'  
      });  
      criticalRisks.push('CRITICAL PRIVACY RISK: Geolocation wildcard exposes user physical coordinates to ad tech trackers.');  
      remediationSteps.push('Set geolocation=() or restrict to geolocation=(self).');  
    } else if (geoRule \=== 'none') {  
      checks.push({  
        id: 'geolocation\_defense',  
        name: 'Geolocation Surveillance Defense',  
        passed: true,  
        points: 15,  
        maxPoints: 15,  
        status: 'PASS',  
        summary: 'Geolocation completely disabled (=()). Maximum privacy posture.',  
        details: 'Zero possibility of GPS exfiltration across all contexts.'  
      });  
      score \+= 15;  
    } else if (geoRule \=== 'self' || geoRule \=== 'custom') {  
      checks.push({  
        id: 'geolocation\_defense',  
        name: 'Geolocation Surveillance Defense',  
        passed: true,  
        points: 12,  
        maxPoints: 15,  
        status: 'PASS',  
        summary: 'Geolocation scoped strictly to authorized origin.',  
        details: \`geolocation=\${directives.geolocation.raw}\`  
      });  
      score \+= 12;  
    } else {  
      checks.push({  
        id: 'geolocation\_defense',  
        name: 'Geolocation Surveillance Defense',  
        passed: false,  
        points: 3,  
        maxPoints: 15,  
        status: 'WARN',  
        summary: 'Geolocation directive is omitted.',  
        details: 'Unrestricted same-origin iframe access permitted by browser default.'  
      });  
      score \+= 3;  
      remediationSteps.push('Add geolocation=() to prevent physical tracking.');  
    }

    // \--- Check 3: Privacy Sandbox & Ad Profiling Restrictions (10 pts) \---  
    const flocRule \= getRule('interest-cohort');  
    const topicsRule \= getRule('browsing-topics');  
    const auctionRule \= getRule('run-ad-auction');

    const adRestrictionsActive \= flocRule \=== 'none' || topicsRule \=== 'none';  
    if (flocRule \=== 'none' && (topicsRule \=== 'none' || auctionRule \=== 'none')) {  
      checks.push({  
        id: 'ad\_profiling',  
        name: 'Privacy Sandbox & Cross-Site Ad Profiling Defense',  
        passed: true,  
        points: 10,  
        maxPoints: 10,  
        status: 'PASS',  
        summary: 'Cross-site cohort tracking and Topics API disabled.',  
        details: 'interest-cohort=(), browsing-topics=(), run-ad-auction=() prevent browser profiling.'  
      });  
      score \+= 10;  
    } else if (adRestrictionsActive) {  
      checks.push({  
        id: 'ad\_profiling',  
        name: 'Privacy Sandbox & Cross-Site Ad Profiling Defense',  
        passed: true,  
        points: 7,  
        maxPoints: 10,  
        status: 'PASS',  
        summary: 'Partial ad tracking restrictions active (FLoC disabled).',  
        details: 'interest-cohort=() declared.'  
      });  
      score \+= 7;  
    } else {  
      checks.push({  
        id: 'ad\_profiling',  
        name: 'Privacy Sandbox & Cross-Site Ad Profiling Defense',  
        passed: false,  
        points: 0,  
        maxPoints: 10,  
        status: 'WARN',  
        summary: 'Browser ad profiling APIs are unconstrained.',  
        details: 'Third-party scripts can participate in FLoC or Topics profiling on your users.'  
      });  
      remediationSteps.push('Add interest-cohort=(), browsing-topics=(), run-ad-auction=() to protect user privacy.');  
    }

    // \--- Check 4: Motion & Sensor Eavesdropping Defense (10 pts) \---  
    const accRule \= getRule('accelerometer');  
    const gyroRule \= getRule('gyroscope');  
    const magRule \= getRule('magnetometer');

    if (accRule \=== 'none' && gyroRule \=== 'none') {  
      checks.push({  
        id: 'motion\_sensors',  
        name: 'Motion Sensor Eavesdropping Defense',  
        passed: true,  
        points: 10,  
        maxPoints: 10,  
        status: 'PASS',  
        summary: 'High-frequency accelerometer and gyroscope disabled (=()).',  
        details: 'Eliminates mobile PIN sniffing and acoustic side-channel attacks.'  
      });  
      score \+= 10;  
    } else if (accRule \=== 'self' && gyroRule \=== 'self') {  
      checks.push({  
        id: 'motion\_sensors',  
        name: 'Motion Sensor Eavesdropping Defense',  
        passed: true,  
        points: 7,  
        maxPoints: 10,  
        status: 'PASS',  
        summary: 'Motion sensors scoped to self only.',  
        details: 'Third-party iframes cannot access accelerometer or gyroscope.'  
      });  
      score \+= 7;  
    } else {  
      checks.push({  
        id: 'motion\_sensors',  
        name: 'Motion Sensor Eavesdropping Defense',  
        passed: false,  
        points: 2,  
        maxPoints: 10,  
        status: 'WARN',  
        summary: 'Motion sensors (accelerometer/gyroscope) are unconstrained.',  
        details: 'Third-party scripts can monitor device orientation and motion data.'  
      });  
      score \+= 2;  
      remediationSteps.push('Set accelerometer=(), gyroscope=(), magnetometer=().');  
    }

    // \--- Check 5: Direct Hardware Bus Isolation (10 pts) \---  
    const usbRule \= getRule('usb');  
    const btRule \= getRule('bluetooth');  
    const serialRule \= getRule('serial');  
    const hidRule \= getRule('hid');

    const busDisabled \= (usbRule \=== 'none' || usbRule \=== 'unspecified') &&   
                        (btRule \=== 'none' || btRule \=== 'unspecified') &&  
                        (serialRule \=== 'none' || serialRule \=== 'unspecified') &&  
                        (hidRule \=== 'none' || hidRule \=== 'unspecified');

    const hasExplicitBusNone \= usbRule \=== 'none' || serialRule \=== 'none' || hidRule \=== 'none';

    if (usbRule \=== 'wildcard' || serialRule \=== 'wildcard') {  
      checks.push({  
        id: 'hardware\_bus',  
        name: 'Direct Hardware Bus Isolation (USB/HID/Serial)',  
        passed: false,  
        points: 0,  
        maxPoints: 10,  
        status: 'FAIL',  
        summary: 'Fatal: WebUSB or WebSerial opened to wildcard (\*)\!',  
        details: 'Malicious iframes can interact directly with host USB peripherals, smartcards, and serial ports.'  
      });  
      criticalRisks.push('CRITICAL HARDWARE EXPOSURE: WebUSB/WebSerial wildcard allows malicious physical device communication.');  
      remediationSteps.push('Explicitly disable usb=(), serial=(), hid=(), bluetooth=().');  
    } else if (hasExplicitBusNone) {  
      checks.push({  
        id: 'hardware\_bus',  
        name: 'Direct Hardware Bus Isolation (USB/HID/Serial)',  
        passed: true,  
        points: 10,  
        maxPoints: 10,  
        status: 'PASS',  
        summary: 'Physical hardware interfaces explicitly disabled (=()).',  
        details: 'WebUSB, WebSerial, and WebHID cannot be invoked by any web context.'  
      });  
      score \+= 10;  
    } else {  
      checks.push({  
        id: 'hardware\_bus',  
        name: 'Direct Hardware Bus Isolation (USB/HID/Serial)',  
        passed: true,  
        points: 6,  
        maxPoints: 10,  
        status: 'PASS',  
        summary: 'Default hardware bus state (unspecified).',  
        details: 'Recommend explicitly declaring usb=(), serial=(), hid=(), bluetooth=() for zero-trust posture.'  
      });  
      score \+= 6;  
      remediationSteps.push('Explicitly declare usb=(), serial=(), hid=() to lock down hardware access.');  
    }

    // \--- Check 6: Screen Capture & Display Theft Defense (10 pts) \---  
    const displayRule \= getRule('display-capture');  
    if (displayRule \=== 'wildcard') {  
      checks.push({  
        id: 'display\_capture',  
        name: 'Display & Screen Capture Defense',  
        passed: false,  
        points: 0,  
        maxPoints: 10,  
        status: 'FAIL',  
        summary: 'display-capture delegated to wildcard (\*).',  
        details: 'Iframes can prompt users for full desktop screen sharing.'  
      });  
      criticalRisks.push('SCREEN SHARE RISK: display-capture=\* allows unauthorized third-party screen recording prompts.');  
      remediationSteps.push('Set display-capture=() or display-capture=(self).');  
    } else if (displayRule \=== 'none' || displayRule \=== 'self') {  
      checks.push({  
        id: 'display\_capture',  
        name: 'Display & Screen Capture Defense',  
        passed: true,  
        points: 10,  
        maxPoints: 10,  
        status: 'PASS',  
        summary: \`Screen sharing restricted: display-capture=\${directives\['display-capture'\] ? directives\['display-capture'\].raw : 'none'}.\`,  
        details: 'Third-party iframes cannot trigger screen recording.'  
      });  
      score \+= 10;  
    } else {  
      checks.push({  
        id: 'display\_capture',  
        name: 'Display & Screen Capture Defense',  
        passed: true,  
        points: 6,  
        maxPoints: 10,  
        status: 'PASS',  
        summary: 'display-capture omitted (default browser security controls apply).',  
        details: 'Explicitly setting display-capture=() recommended.'  
      });  
      score \+= 6;  
    }

    // \--- Check 7: RFC 8941 Structured Field Syntax Compliance (10 pts) \---  
    if (parsed.valid && parsed.errors.length \=== 0\) {  
      const deduction \= parsed.warnings.length \> 0 ? 2 : 0;  
      const pts \= 10 \- deduction;  
      checks.push({  
        id: 'rfc8941\_syntax',  
        name: 'RFC 8941 Structured Field Values Syntax',  
        passed: true,  
        points: pts,  
        maxPoints: 10,  
        status: pts \=== 10 ? 'PASS' : 'WARN',  
        summary: 'Valid W3C Permissions-Policy RFC 8941 syntax.',  
        details: parsed.warnings.length \> 0 ? parsed.warnings.join(' ') : 'Clean structured field formatting (parentheses, double-quotes, commas).'  
      });  
      score \+= pts;  
    } else {  
      checks.push({  
        id: 'rfc8941\_syntax',  
        name: 'RFC 8941 Structured Field Values Syntax',  
        passed: false,  
        points: 0,  
        maxPoints: 10,  
        status: 'FAIL',  
        summary: 'Syntax Error in Permissions-Policy header\!',  
        details: parsed.errors.join('; ')  
      });  
      remediationSteps.push('Fix syntax errors: feature=() for none, feature=(self) for self, feature=("https\://...") for origins.');  
    }

    // \--- Check 8: Payment Request API Scoping (PCI DSS 4.0 Req 6.4.3) (10 pts) \---  
    const payRule \= getRule('payment');  
    if (payRule \=== 'wildcard') {  
      checks.push({  
        id: 'payment\_scoping',  
        name: 'Payment Request API Scoping (PCI DSS 4.0)',  
        passed: false,  
        points: 0,  
        maxPoints: 10,  
        status: 'FAIL',  
        summary: 'Payment API delegated to wildcard (\*). PCI DSS non-compliance\!',  
        details: 'Allows untrusted iframes to invoke Payment Request modal on checkout pages.'  
      });  
      criticalRisks.push('PCI DSS 4.0 VIOLATION: payment=\* exposes payment APIs to unverified third parties.');  
      remediationSteps.push('Set payment=(self "https\://js.stripe.com") or payment=(self).');  
    } else if (payRule \=== 'self' || payRule \=== 'custom' || payRule \=== 'none') {  
      checks.push({  
        id: 'payment\_scoping',  
        name: 'Payment Request API Scoping (PCI DSS 4.0)',  
        passed: true,  
        points: 10,  
        maxPoints: 10,  
        status: 'PASS',  
        summary: 'Payment API strictly scoped to authorized checkout domain.',  
        details: \`payment=\${directives.payment ? directives.payment.raw : 'none'}\`  
      });  
      score \+= 10;  
    } else {  
      checks.push({  
        id: 'payment\_scoping',  
        name: 'Payment Request API Scoping (PCI DSS 4.0)',  
        passed: isEcommerce ? false : true,  
        points: isEcommerce ? 2 : 7,  
        maxPoints: 10,  
        status: isEcommerce ? 'WARN' : 'PASS',  
        summary: isEcommerce ? 'Missing explicit payment directive on e-commerce origin.' : 'Payment directive unspecified.',  
        details: 'PCI DSS v4.0 Requirement 6.4.3 recommends explicit payment API scoping.'  
      });  
      score \+= isEcommerce ? 2 : 7;  
      if (isEcommerce) remediationSteps.push('Explicitly configure payment=(self) for cardholder data security.');  
    }

    // \--- Check 9: Media Autoplay & Fullscreen Governance (5 pts) \---  
    const autoRule \= getRule('autoplay');  
    const fullRule \= getRule('fullscreen');  
    if (autoRule \=== 'wildcard' || fullRule \=== 'wildcard') {  
      checks.push({  
        id: 'media\_governance',  
        name: 'Media Autoplay & Fullscreen Governance',  
        passed: false,  
        points: 1,  
        maxPoints: 5,  
        status: 'WARN',  
        summary: 'Autoplay or Fullscreen set to wildcard (\*).',  
        details: 'Third-party ads can force fullscreen overlay or blare loud audio.'  
      });  
      score \+= 1;  
    } else {  
      checks.push({  
        id: 'media\_governance',  
        name: 'Media Autoplay & Fullscreen Governance',  
        passed: true,  
        points: 5,  
        maxPoints: 5,  
        status: 'PASS',  
        summary: 'Autoplay and Fullscreen appropriately scoped.',  
        details: 'User interface controls preserved.'  
      });  
      score \+= 5;  
    }

    // \--- Check 10: Third-Party Iframe Isolation & Deployment (5 pts) \---  
    const totalConfigured \= Object.keys(directives).length;  
    if (totalConfigured \>= 10\) {  
      checks.push({  
        id: 'iframe\_isolation',  
        name: 'Comprehensive Browser Feature Coverage',  
        passed: true,  
        points: 5,  
        maxPoints: 5,  
        status: 'PASS',  
        summary: \`Hardened feature coverage: \${totalConfigured} features explicitly governed.\`,  
        details: 'Provides robust defense-in-depth across sensors, hardware, media, and ad profiling.'  
      });  
      score \+= 5;  
    } else if (totalConfigured \>= 5\) {  
      checks.push({  
        id: 'iframe\_isolation',  
        name: 'Comprehensive Browser Feature Coverage',  
        passed: true,  
        points: 3,  
        maxPoints: 5,  
        status: 'PASS',  
        summary: \`Moderate coverage: \${totalConfigured} features governed.\`,  
        details: 'Consider expanding coverage to include WebUSB, WebSerial, and FLoC/Topics.'  
      });  
      score \+= 3;  
    } else {  
      checks.push({  
        id: 'iframe\_isolation',  
        name: 'Comprehensive Browser Feature Coverage',  
        passed: false,  
        points: 1,  
        maxPoints: 5,  
        status: 'WARN',  
        summary: \`Minimal coverage: Only \${totalConfigured} features specified.\`,  
        details: 'Leaves unmentioned hardware and sensor APIs at default browser privilege levels.'  
      });  
      score \+= 1;  
      remediationSteps.push('Adopt full 15+ feature policy baseline.');  
    }

    // Compute Grade  
    let grade \= 'F';  
    if (score \>= 95\) grade \= 'A+';  
    else if (score \>= 85\) grade \= 'A';  
    else if (score \>= 70\) grade \= 'B';  
    else if (score \>= 50\) grade \= 'C';  
    else if (score \>= 30\) grade \= 'D';

    return {  
      score,  
      maxScore: 100,  
      grade,  
      parsed,  
      checks,  
      criticalRisks,  
      remediationSteps,  
      summary: {  
        totalChecks: checks.length,  
        passedChecks: checks.filter(c \=\> c.passed).length,  
        failedChecks: checks.filter(c \=\> \!c.passed).length  
      }  
    };  
  }

  return {  
    audit  
  };  
})();

if (typeof module \!== 'undefined' && module.exports) {  
  module.exports \= PrivilegeAuditor;  
}  
