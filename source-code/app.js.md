/\*\*  
 \* PermissionsPolicy Pro \- Studio UI Application Controller  
 \*/

document.addEventListener('DOMContentLoaded', function() {  
  'use strict';

  // \--- Initial State \---  
  const state \= {  
    targetDomain: 'checkout.fintechshield.com',  
    isEcommerce: true,  
    features: {  
      camera: { type: 'none' },  
      microphone: { type: 'none' },  
      geolocation: { type: 'none' },  
      'display-capture': { type: 'none' },  
      payment: { type: 'custom', origins: \['self', 'https\://js.stripe.com'\] },  
      usb: { type: 'none' },  
      bluetooth: { type: 'none' },  
      serial: { type: 'none' },  
      hid: { type: 'none' },  
      accelerometer: { type: 'none' },  
      gyroscope: { type: 'none' },  
      magnetometer: { type: 'none' },  
      'interest-cohort': { type: 'none' },  
      'browsing-topics': { type: 'none' },  
      'run-ad-auction': { type: 'none' },  
      'join-ad-group': { type: 'none' },  
      autoplay: { type: 'none' },  
      fullscreen: { type: 'self' },  
      'screen-wake-lock': { type: 'self' }  
    },  
    currentServer: 'nginx'  
  };

  // \--- Presets Data \---  
  const PRESETS \= {  
    'fintech-pci-dss-strict': {  
      targetDomain: 'checkout.fintechshield.com',  
      isEcommerce: true,  
      features: {  
        camera: { type: 'none' },  
        microphone: { type: 'none' },  
        geolocation: { type: 'none' },  
        'display-capture': { type: 'none' },  
        payment: { type: 'custom', origins: \['self', 'https\://js.stripe.com'\] },  
        usb: { type: 'none' },  
        bluetooth: { type: 'none' },  
        serial: { type: 'none' },  
        hid: { type: 'none' },  
        accelerometer: { type: 'none' },  
        gyroscope: { type: 'none' },  
        magnetometer: { type: 'none' },  
        'interest-cohort': { type: 'none' },  
        'browsing-topics': { type: 'none' },  
        'run-ad-auction': { type: 'none' },  
        'join-ad-group': { type: 'none' },  
        autoplay: { type: 'none' },  
        fullscreen: { type: 'self' },  
        'screen-wake-lock': { type: 'self' }  
      }  
    },  
    'vulnerable-permissive-iframe': {  
      targetDomain: 'portal.vulnerablestore.com',  
      isEcommerce: true,  
      features: {  
        camera: { type: 'wildcard' },  
        microphone: { type: 'wildcard' },  
        geolocation: { type: 'wildcard' },  
        'display-capture': { type: 'wildcard' },  
        payment: { type: 'wildcard' },  
        usb: { type: 'wildcard' },  
        accelerometer: { type: 'wildcard' },  
        gyroscope: { type: 'wildcard' },  
        'interest-cohort': { type: 'wildcard' }  
      }  
    },  
    'media-rich-saas': {  
      targetDomain: 'meet.collabsuite.io',  
      isEcommerce: false,  
      features: {  
        camera: { type: 'self' },  
        microphone: { type: 'self' },  
        'display-capture': { type: 'self' },  
        'screen-wake-lock': { type: 'self' },  
        'speaker-selection': { type: 'self' },  
        autoplay: { type: 'self' },  
        fullscreen: { type: 'self' },  
        geolocation: { type: 'none' },  
        accelerometer: { type: 'none' },  
        gyroscope: { type: 'none' },  
        usb: { type: 'none' },  
        'interest-cohort': { type: 'none' },  
        'browsing-topics': { type: 'none' },  
        'run-ad-auction': { type: 'none' }  
      }  
    }  
  };

  // \--- DOM Elements \---  
  const presetSelect \= document.getElementById('preset-select');  
  const targetDomainInput \= document.getElementById('target-domain');  
  const featureMatrixContainer \= document.getElementById('feature-matrix-container');

  const statGrade \= document.getElementById('stat-grade');  
  const statGradeDesc \= document.getElementById('stat-grade-desc');  
  const statScore \= document.getElementById('stat-score');  
  const statScoreDesc \= document.getElementById('stat-score-desc');  
  const statSensors \= document.getElementById('stat-sensors');  
  const statSensorsDesc \= document.getElementById('stat-sensors-desc');  
  const statCoverage \= document.getElementById('stat-coverage');  
  const statCoverageDesc \= document.getElementById('stat-coverage-desc');

  const outputPPBlock \= document.getElementById('output-pp-block');  
  const outputFPBlock \= document.getElementById('output-fp-block');

  const infoMediaStatus \= document.getElementById('info-media-status');  
  const infoGeoStatus \= document.getElementById('info-geo-status');  
  const infoHardwareStatus \= document.getElementById('info-hardware-status');  
  const infoAdsStatus \= document.getElementById('info-ads-status');

  const auditChecksContainer \= document.getElementById('audit-checks-container');  
  const criticalRisksBox \= document.getElementById('critical-risks-box');  
  const criticalRisksList \= document.getElementById('critical-risks-list');

  const simScenarioSelect \= document.getElementById('sim-scenario-select');  
  const btnRunSim \= document.getElementById('btn-run-sim');  
  const simOutcomeTitle \= document.getElementById('sim-outcome-title');  
  const simOutcomeBadge \= document.getElementById('sim-outcome-badge');  
  const simOutcomeDesc \= document.getElementById('sim-outcome-desc');  
  const simTimelineContainer \= document.getElementById('sim-timeline-container');

  const serverCodeDisplay \= document.getElementById('server-code-display');  
  const serverFilenameLabel \= document.getElementById('server-filename-label');  
  const referenceTableBody \= document.getElementById('reference-table-body');  
  const toast \= document.getElementById('toast');

  // \--- Initialize Studio \---  
  renderFeatureMatrix();  
  renderReferenceTable();  
  initEventListeners();  
  refreshAll();  
  runActiveSimulation();

  // \--- Render Feature Matrix \---  
  function renderFeatureMatrix() {  
    featureMatrixContainer.innerHTML \= '';  
    const catalog \= PermissionsEngine.FEATURE\_CATALOG;

    Object.keys(catalog).forEach(catKey \=\> {  
      const cat \= catalog\[catKey\];  
      const block \= document.createElement('div');  
      block.className \= 'category-block';

      let rowsHtml \= '';  
      Object.keys(cat.features).forEach(featKey \=\> {  
        const feat \= cat.features\[featKey\];  
        const currentType \= (state.features\[featKey\] && state.features\[featKey\].type) || 'unspecified';

        const riskBadge \= feat.risk \=== 'CRITICAL'   
          ? '\<span class="badge badge-danger" style="font-size: 0.65rem; padding: 2px 6px;"\>CRITICAL\</span\>'   
          : (feat.risk \=== 'HIGH' ? '\<span class="badge badge-warning" style="font-size: 0.65rem; padding: 2px 6px;"\>HIGH\</span\>' : '');

        rowsHtml \+= \`  
          \<div class="feature-row" data-feature="\${featKey}"\>  
            \<div class="feature-meta"\>  
              \<div class="feature-name"\>\${featKey} \${riskBadge}\</div\>  
              \<div class="feature-label"\>\${feat.label}\</div\>  
            \</div\>  
            \<div class="segment-group" data-feature="\${featKey}"\>  
              \<button class="segment-btn \${currentType \=== 'none' ? 'active' : ''}" data-type="none" title="Disabled completely ()"\>None ()\</button\>  
              \<button class="segment-btn \${currentType \=== 'self' ? 'active' : ''}" data-type="self" title="Origin only (self)"\>Self\</button\>  
              \<button class="segment-btn \${currentType \=== 'custom' ? 'active' : ''}" data-type="custom" title="Custom trusted domains"\>Custom\</button\>  
              \<button class="segment-btn \${currentType \=== 'wildcard' ? 'active' : ''}" data-type="wildcard" title="Allowed on all origins (\*)"\>\*\</button\>  
            \</div\>  
          \</div\>  
        \`;  
      });

      block.innerHTML \= \`  
        \<div class="category-title"\>  
          \<span\>\${cat.categoryName}\</span\>  
        \</div\>  
        \<div\>\${rowsHtml}\</div\>  
      \`;  
      featureMatrixContainer.appendChild(block);  
    });

    // Attach segment button listeners  
    featureMatrixContainer.querySelectorAll('.segment-btn').forEach(btn \=\> {  
      btn.addEventListener('click', function() {  
        const featKey \= btn.closest('.segment-group').getAttribute('data-feature');  
        const type \= btn.getAttribute('data-type');

        btn.closest('.segment-group').querySelectorAll('.segment-btn').forEach(b \=\> b.classList.remove('active'));  
        btn.classList.add('active');

        if (type \=== 'custom') {  
          const current \= (state.features\[featKey\] && state.features\[featKey\].origins) ? state.features\[featKey\].origins.join(' ') : 'self https\://trusted.com';  
          const promptVal \= prompt(\`Enter authorized origins for "\${featKey}" (space-separated, e.g. self https\://js.stripe.com):\`, current);  
          if (promptVal \!== null && promptVal.trim()) {  
            state.features\[featKey\] \= {  
              type: 'custom',  
              origins: promptVal.split(/\\s+/).filter(Boolean)  
            };  
          } else {  
            state.features\[featKey\] \= { type: 'self' };  
            btn.closest('.segment-group').querySelector('\[data-type="self"\]').classList.add('active');  
            btn.classList.remove('active');  
          }  
        } else {  
          state.features\[featKey\] \= { type };  
        }

        presetSelect.value \= 'custom';  
        refreshAll();  
      });  
    });  
  }

  // \--- Render Feature Reference Table \---  
  function renderReferenceTable() {  
    referenceTableBody.innerHTML \= '';  
    const catalog \= PermissionsEngine.FEATURE\_CATALOG;

    Object.keys(catalog).forEach(catKey \=\> {  
      const cat \= catalog\[catKey\];  
      Object.keys(cat.features).forEach(featKey \=\> {  
        const feat \= cat.features\[featKey\];  
        const tr \= document.createElement('tr');  
          
        const riskColor \= feat.risk \=== 'CRITICAL' ? '\#f87171' : (feat.risk \=== 'HIGH' ? '\#fbbf24' : '\#60a5fa');  
        const safeState \= feat.defaultState \=== 'none' ? 'Disabled \=()' : 'Origin Only \=(self)';

        tr.innerHTML \= \`  
          \<td\>\<code style="color: \#38bdf8; font-weight: bold;"\>\${featKey}\</code\>\</td\>  
          \<td\>\${cat.categoryName}\</td\>  
          \<td\>\${feat.label}\</td\>  
          \<td\>\<strong style="color: \${riskColor};"\>\${feat.risk}\</strong\>\</td\>  
          \<td\>\<span style="color: \#34d399; font-weight: 600;"\>\${safeState}\</span\>\</td\>  
        \`;  
        referenceTableBody.appendChild(tr);  
      });  
    });  
  }

  // \--- Event Listeners \---  
  function initEventListeners() {  
    // Preset dropdown  
    presetSelect.addEventListener('change', function(e) {  
      const selected \= e.target.value;  
      if (PRESETS\[selected\]) {  
        const p \= PRESETS\[selected\];  
        state.targetDomain \= p.targetDomain;  
        state.isEcommerce \= p.isEcommerce;  
        state.features \= JSON.parse(JSON.stringify(p.features));

        targetDomainInput.value \= state.targetDomain;  
        renderFeatureMatrix();  
        refreshAll();  
        runActiveSimulation();  
        showToast(\`Loaded \${presetSelect.options\[presetSelect.selectedIndex\].text}\`);  
      }  
    });

    targetDomainInput.addEventListener('input', function(e) {  
      state.targetDomain \= e.target.value.trim() || 'example.com';  
      refreshAll();  
    });

    // Tab buttons  
    document.querySelectorAll('.tab-btn').forEach(btn \=\> {  
      btn.addEventListener('click', function() {  
        document.querySelectorAll('.tab-btn').forEach(b \=\> b.classList.remove('active'));  
        document.querySelectorAll('.tab-panel').forEach(p \=\> p.classList.remove('active'));  
        btn.classList.add('active');  
        const tabId \= btn.getAttribute('data-tab');  
        document.getElementById(tabId).classList.add('active');  
      });  
    });

    // Server switch buttons  
    document.querySelectorAll('.server-switch-btn').forEach(btn \=\> {  
      btn.addEventListener('click', function() {  
        document.querySelectorAll('.server-switch-btn').forEach(b \=\> {  
          b.classList.remove('active');  
          b.classList.remove('btn-primary');  
        });  
        btn.classList.add('active');  
        btn.classList.add('btn-primary');  
        state.currentServer \= btn.getAttribute('data-server');  
        renderServerSnippet();  
      });  
    });

    // Copy buttons  
    document.getElementById('btn-copy-pp').addEventListener('click', function() {  
      copyToClipboard(outputPPBlock.textContent);  
    });

    document.getElementById('btn-copy-fp').addEventListener('click', function() {  
      copyToClipboard(outputFPBlock.textContent);  
    });

    document.getElementById('btn-copy-server-code').addEventListener('click', function() {  
      copyToClipboard(serverCodeDisplay.textContent);  
    });

    document.getElementById('btn-re-audit').addEventListener('click', function() {  
      refreshAll();  
      showToast('Permissions policy audit scan complete\!');  
    });

    // Simulator  
    btnRunSim.addEventListener('click', runActiveSimulation);  
    simScenarioSelect.addEventListener('change', runActiveSimulation);  
  }

  // \--- Main Refresh Routine \---  
  function refreshAll() {  
    const syn \= PermissionsEngine.synthesizeHeaders(state.features);  
    outputPPBlock.textContent \= \`Permissions-Policy: \${syn.permissionsPolicy}\`;  
    outputFPBlock.textContent \= \`Feature-Policy: \${syn.featurePolicy}\`;

    // Run 10-point audit  
    const auditRes \= PrivilegeAuditor.audit(syn.permissionsPolicy, {  
      isEcommerce: state.isEcommerce  
    });

    // Update Stats  
    statGrade.textContent \= auditRes.grade;  
    statGrade.style.color \= auditRes.grade \=== 'A+' || auditRes.grade \=== 'A' ? '\#34d399' : (auditRes.grade \=== 'B' ? '\#60a5fa' : (auditRes.grade \=== 'C' ? '\#fbbf24' : '\#f87171'));  
    statGradeDesc.textContent \= auditRes.grade \=== 'A+' ? 'Zero-Trust Hardware Lockdown' : (auditRes.grade \=== 'A' ? 'High Security Posture' : 'Permissive Privilege Delegation');

    statScore.innerHTML \= \`\${auditRes.score}\<span style="font-size: 1rem; color: var(--text-dim);"\>/100\</span\>\`;  
    statScoreDesc.textContent \= \`\${auditRes.summary.passedChecks} of \${auditRes.summary.totalChecks} vectors passed\`;

    const geoRule \= state.features.geolocation ? state.features.geolocation.type : 'unspecified';  
    if (geoRule \=== 'none') {  
      statSensors.textContent \= 'ISOLATED (=())';  
      statSensors.style.color \= '\#34d399';  
      statSensorsDesc.textContent \= 'No third-party GPS/motion leaks';  
    } else if (geoRule \=== 'self') {  
      statSensors.textContent \= 'SELF ONLY';  
      statSensors.style.color \= '\#60a5fa';  
      statSensorsDesc.textContent \= 'Scoped to origin host';  
    } else {  
      statSensors.textContent \= 'EXPOSED';  
      statSensors.style.color \= '\#f87171';  
      statSensorsDesc.textContent \= 'Ad trackers can query GPS\!';  
    }

    statCoverage.textContent \= \`\${syn.totalFeaturesConfigured} Features\`;  
    statCoverageDesc.textContent \= syn.totalFeaturesConfigured \>= 15 ? 'Comprehensive policy baseline' : 'Partial policy baseline';

    // Summary lines  
    const camType \= state.features.camera ? state.features.camera.type : 'unspecified';  
    const micType \= state.features.microphone ? state.features.microphone.type : 'unspecified';  
    if (camType \=== 'none' && micType \=== 'none') {  
      infoMediaStatus.textContent \= 'Disabled (=())';  
      infoMediaStatus.style.color \= '\#34d399';  
    } else if (camType \=== 'self' && micType \=== 'self') {  
      infoMediaStatus.textContent \= 'Origin Only (self)';  
      infoMediaStatus.style.color \= '\#60a5fa';  
    } else if (camType \=== 'wildcard' || micType \=== 'wildcard') {  
      infoMediaStatus.textContent \= 'WILDCARD EXPOSED (\*)';  
      infoMediaStatus.style.color \= '\#f87171';  
    } else {  
      infoMediaStatus.textContent \= \`\${camType} / \${micType}\`;  
      infoMediaStatus.style.color \= '\#fbbf24';  
    }

    infoGeoStatus.textContent \= geoRule \=== 'none' ? 'Disabled (=())' : (geoRule \=== 'self' ? 'Origin Only (self)' : 'WILDCARD EXPOSED (\*)');  
    infoGeoStatus.style.color \= geoRule \=== 'none' ? '\#34d399' : (geoRule \=== 'self' ? '\#60a5fa' : '\#f87171');

    const usbType \= state.features.usb ? state.features.usb.type : 'unspecified';  
    infoHardwareStatus.textContent \= usbType \=== 'none' ? 'WebUSB & Bus Disabled' : 'Unconstrained';  
    infoHardwareStatus.style.color \= usbType \=== 'none' ? '\#34d399' : '\#f87171';

    const flocType \= state.features\['interest-cohort'\] ? state.features\['interest-cohort'\].type : 'unspecified';  
    infoAdsStatus.textContent \= flocType \=== 'none' ? 'FLoC & Topics Disabled' : 'Ad Tracking Allowed';  
    infoAdsStatus.style.color \= flocType \=== 'none' ? '\#34d399' : '\#fbbf24';

    // Render Checklist  
    renderChecklist(auditRes);

    // Render Server Config  
    renderServerSnippet();  
  }

  // \--- Render Checklist \---  
  function renderChecklist(auditRes) {  
    auditChecksContainer.innerHTML \= '';  
    auditRes.checks.forEach(check \=\> {  
      const item \= document.createElement('div');  
      const cls \= check.status \=== 'PASS' ? 'pass' : (check.status \=== 'WARN' ? 'warn' : 'fail');  
      item.className \= \`check-item \${cls}\`;

      const badgeHtml \= check.status \=== 'PASS'  
        ? '\<span class="badge badge-success"\>PASS\</span\>'  
        : (check.status \=== 'WARN' ? '\<span class="badge badge-warning"\>WARN\</span\>' : '\<span class="badge badge-danger"\>FAIL\</span\>');

      item.innerHTML \= \`  
        \<div class="check-header-row"\>  
          \<div class="check-title"\>  
            \${badgeHtml}  
            \<span\>\${escapeHtml(check.name)}\</span\>  
          \</div\>  
          \<div class="check-points"\>\${check.points}/\${check.maxPoints} pts\</div\>  
        \</div\>  
        \<div class="check-summary"\>\${escapeHtml(check.summary)}\</div\>  
        \<div class="check-details"\>\${escapeHtml(check.details)}\</div\>  
      \`;  
      auditChecksContainer.appendChild(item);  
    });

    if (auditRes.criticalRisks.length \> 0\) {  
      criticalRisksBox.style.display \= 'block';  
      criticalRisksList.innerHTML \= auditRes.criticalRisks.map(r \=\> \`\<li\>\${escapeHtml(r)}\</li\>\`).join('');  
    } else {  
      criticalRisksBox.style.display \= 'none';  
    }  
  }

  // \--- Render Server Snippet \---  
  function renderServerSnippet() {  
    const syn \= PermissionsEngine.synthesizeHeaders(state.features);  
    const configs \= ServerSynthesizer.generateAll(syn.permissionsPolicy, syn.featurePolicy, state.targetDomain);

    const labels \= {  
      nginx: \`/etc/nginx/sites-available/\${state.targetDomain}\`,  
      apache: \`.htaccess or /etc/apache2/sites-available/\${state.targetDomain}.conf\`,  
      cloudflare: \`\_headers or workers/\${state.targetDomain}.js\`,  
      caddy: \`Caddyfile\`,  
      iframe: \`HTML5 \<iframe\> allow attribute templates\`  
    };

    serverFilenameLabel.textContent \= labels\[state.currentServer\] || state.currentServer;  
    serverCodeDisplay.textContent \= configs\[state.currentServer\] || '';  
  }

  // \--- Run Active Simulation \---  
  function runActiveSimulation() {  
    const scenario \= simScenarioSelect.value;  
    const simResult \= HardwareSimulator.runSimulation(scenario, {  
      siteDomain: state.targetDomain,  
      iframeDomain: 'untrusted-widget-cdn.com'  
    });

    simOutcomeTitle.textContent \= \`Simulation Result: \${simResult.name}\`;  
    simOutcomeDesc.textContent \= simResult.outcomeSummary;

    if (simResult.outcome \=== 'ATTACK BLOCKED') {  
      simOutcomeBadge.className \= 'badge badge-success';  
      simOutcomeBadge.textContent \= simResult.outcome;  
    } else {  
      simOutcomeBadge.className \= 'badge badge-danger';  
      simOutcomeBadge.textContent \= simResult.outcome;  
    }

    renderSimulationSteps(simResult.steps);  
  }

  function renderSimulationSteps(steps) {  
    simTimelineContainer.innerHTML \= '';  
    steps.forEach(s \=\> {  
      const stepElem \= document.createElement('div');  
      const isOk \= s.securityState \=== 'POLICY\_ACTIVE' || s.securityState \=== 'BLOCKED\_BY\_POLICY' || s.securityState \=== 'ATTACK\_NEUTRALIZED';  
      stepElem.className \= \`sim-step \${isOk ? 'pass' : 'fail'}\`;

      stepElem.innerHTML \= \`  
        \<div class="sim-step-num"\>\${s.step}\</div\>  
        \<div class="sim-step-card"\>  
          \<div class="sim-step-meta"\>  
            \<span class="sim-actor"\>\${escapeHtml(s.actor)} — \<em\>\${escapeHtml(s.title)}\</em\>\</span\>  
            \<span class="badge \${s.badgeClass}"\>\${escapeHtml(s.action)}\</span\>  
          \</div\>  
          \<div style="font-size: 0.8rem; color: var(--text-muted);"\>\${escapeHtml(s.details)}\</div\>  
        \</div\>  
      \`;  
      simTimelineContainer.appendChild(stepElem);  
    });  
  }

  // \--- Clipboard Utility \---  
  function copyToClipboard(text) {  
    if (navigator.clipboard && navigator.clipboard.writeText) {  
      navigator.clipboard.writeText(text).then(() \=\> showToast()).catch(() \=\> fallbackCopy(text));  
    } else {  
      fallbackCopy(text);  
    }  
  }

  function fallbackCopy(text) {  
    const ta \= document.createElement('textarea');  
    ta.value \= text;  
    ta.style.position \= 'fixed';  
    ta.style.opacity \= '0';  
    document.body.appendChild(ta);  
    ta.select();  
    try {  
      document.execCommand('copy');  
      showToast();  
    } catch (e) {  
      console.error('Copy failed', e);  
    }  
    document.body.removeChild(ta);  
  }

  function showToast(msg \= 'Copied to clipboard\!') {  
    toast.textContent \= msg;  
    toast.style.display \= 'block';  
    setTimeout(() \=\> { toast.style.display \= 'none'; }, 2200);  
  }

  function escapeHtml(str) {  
    if (\!str) return '';  
    return String(str)  
      .replace(/&/g, '\&amp;')  
      .replace(/\</g, '\&lt;')  
      .replace(/\>/g, '\&gt;')  
      .replace(/"/g, '\&quot;');  
  }  
});  
