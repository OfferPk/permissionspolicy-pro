/\*\*  
 \* PermissionsPolicy Pro \- Multi-Server & Iframe Embed Code Synthesizer  
 \* Generates verified configurations for Nginx, Apache, Cloudflare Workers/Pages, Caddy,  
 \* and HTML \<iframe\> allow attributes.  
 \*/

const ServerSynthesizer \= (function() {  
  'use strict';

  /\*\*  
   \* Synthesize configs across all supported server technologies  
   \* @param {string} permissionsPolicyStr \- RFC 8941 header value  
   \* @param {string} featurePolicyStr \- Legacy fallback header value  
   \* @param {string} \[domain\] \- Target domain name  
   \* @returns {object} Map of server keys to configuration snippets  
   \*/  
  function generateAll(permissionsPolicyStr, featurePolicyStr, domain \= 'example.com') {  
    return {  
      nginx: generateNginx(permissionsPolicyStr, featurePolicyStr, domain),  
      apache: generateApache(permissionsPolicyStr, featurePolicyStr, domain),  
      cloudflare: generateCloudflare(permissionsPolicyStr, featurePolicyStr, domain),  
      caddy: generateCaddy(permissionsPolicyStr, featurePolicyStr, domain),  
      iframe: generateIframeSnippets(permissionsPolicyStr)  
    };  
  }

  function generateNginx(ppStr, fpStr, domain) {  
    return \`\# Nginx Hardened Configuration for \${domain}  
\# Place inside server {} block in /etc/nginx/sites-available/\${domain}

server {  
    listen 443 ssl http2;  
    server\_name \${domain} \*.\${domain};

    \# W3C Permissions-Policy Header (Modern Browsers)  
    \# The 'always' flag guarantees header retention on 4xx/5xx error pages\!  
    add\_header Permissions-Policy "\${ppStr}" always;

    \# Optional: Legacy Feature-Policy Fallback (Older Chromium/Safari)  
    add\_header Feature-Policy "\${fpStr}" always;

    \# Defense-in-depth Security Headers  
    add\_header X-Content-Type-Options "nosniff" always;  
    add\_header X-Frame-Options "SAMEORIGIN" always;  
    add\_header Referrer-Policy "strict-origin-when-cross-origin" always;

    location / {  
        proxy\_pass http\://127.0.0.1:3000;  
        proxy\_set\_header Host \$host;  
    }  
}\`;  
  }

  function generateApache(ppStr, fpStr, domain) {  
    return \`\# Apache 2.4 Configuration (.htaccess or virtualhost)  
\# Requires mod\_headers: a2enmod headers

\<IfModule mod\_headers.c\>  
    \# W3C Permissions-Policy (Modern)  
    Header always set Permissions-Policy "\${ppStr}"  
      
    \# Legacy Feature-Policy Fallback  
    Header always set Feature-Policy "\${fpStr}"

    \# Additional Transport & Frame Controls  
    Header always set X-Content-Type-Options "nosniff"  
    Header always set X-Frame-Options "SAMEORIGIN"  
    Header always set Referrer-Policy "strict-origin-when-cross-origin"  
\</IfModule\>\`;  
  }

  function generateCloudflare(ppStr, fpStr, domain) {  
    return \`// \--- Option A: Cloudflare Pages (\_headers file placed in publish root) \---  
/\*  
  Permissions-Policy: \${ppStr}  
  Feature-Policy: \${fpStr}  
  X-Content-Type-Options: nosniff  
  X-Frame-Options: SAMEORIGIN  
  Referrer-Policy: strict-origin-when-cross-origin

// \--- Option B: Cloudflare Worker (Edge Middleware) \---  
export default {  
  async fetch(request, env, ctx) {  
    const response \= await fetch(request);

    // Clone headers and inject W3C Permissions-Policy  
    const newHeaders \= new Headers(response.headers);  
    newHeaders.set("Permissions-Policy", "\${ppStr}");  
    newHeaders.set("Feature-Policy", "\${fpStr}");  
    newHeaders.set("X-Content-Type-Options", "nosniff");

    return new Response(response.body, {  
      status: response.status,  
      statusText: response.statusText,  
      headers: newHeaders  
    });  
  }  
};\`;  
  }

  function generateCaddy(ppStr, fpStr, domain) {  
    return \`\# Caddyfile Configuration (Caddy v2)  
\# Handles automatic TLS certificates and security headers.

\${domain}, \*.\${domain} {  
    \# W3C Permissions-Policy Header  
    header Permissions-Policy "\${ppStr}"  
    header Feature-Policy "\${fpStr}"  
    header X-Content-Type-Options "nosniff"  
    header X-Frame-Options "SAMEORIGIN"  
    header Referrer-Policy "strict-origin-when-cross-origin"

    reverse\_proxy localhost:3000  
}\`;  
  }

  function generateIframeSnippets(ppStr) {  
    return \`\<\!-- Hardened HTML5 Iframe Embed Templates \--\>

\<\!-- 1\. Stripe Payment Elements Iframe (Scoped strictly to payment) \--\>  
\<iframe   
  src="https\://js.stripe.com/v3/elements-inner-card.html"  
  title="Secure Payment Gateway"  
  allow="payment"  
  sandbox="allow-scripts allow-same-origin allow-forms"  
  width="100%"   
  height="200"   
  frameborder="0"\>  
\</iframe\>

\<\!-- 2\. Google Maps Embed (Scoped to fullscreen only; zero GPS harvesting) \--\>  
\<iframe   
  src="https\://www\.google.com/maps/embed?pb=..."  
  title="Interactive Location Map"  
  allow="fullscreen"  
  loading="lazy"   
  referrerpolicy="no-referrer-when-downgrade"  
  width="600"   
  height="450"   
  style="border:0;"   
  allowfullscreen=""\>  
\</iframe\>

\<\!-- 3\. YouTube Embed (Explicit feature scoping; no camera or mic access) \--\>  
\<iframe   
  width="560"   
  height="315"   
  src="https\://www\.youtube-nocookie.com/embed/dQw4w9WgXcQ"   
  title="YouTube Video Player"   
  frameborder="0"   
  allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"   
  allowfullscreen\>  
\</iframe\>\`;  
  }

  return {  
    generateAll,  
    generateNginx,  
    generateApache,  
    generateCloudflare,  
    generateCaddy,  
    generateIframeSnippets  
  };  
})();

if (typeof module \!== 'undefined' && module.exports) {  
  module.exports \= ServerSynthesizer;  
}  
