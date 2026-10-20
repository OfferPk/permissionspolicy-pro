/\* PermissionsPolicy Pro \- Modern Dark UI Stylesheet \*/

:root {  
  \--bg-primary: \#0b0f19;  
  \--bg-secondary: \#111827;  
  \--bg-tertiary: \#1f2937;  
  \--border-color: \#374151;  
  \--border-focus: \#3b82f6;  
  \--text-main: \#f9fafb;  
  \--text-muted: \#9ca3af;  
  \--text-dim: \#6b7280;

  \--emerald-accent: \#10b981;  
  \--emerald-glow: rgba(16, 185, 129, 0.2);  
  \--blue-accent: \#3b82f6;  
  \--blue-glow: rgba(59, 130, 246, 0.2);  
  \--amber-accent: \#f59e0b;  
  \--amber-glow: rgba(245, 158, 11, 0.2);  
  \--red-accent: \#ef4444;  
  \--red-glow: rgba(239, 68, 68, 0.2);  
  \--purple-accent: \#8b5cf6;

  \--font-sans: \-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Oxygen, Ubuntu, Cantarell, sans-serif;  
  \--font-mono: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", monospace;  
  \--radius-sm: 6px;  
  \--radius-md: 10px;  
  \--radius-lg: 14px;  
}

\* {  
  box-sizing: border-box;  
  margin: 0;  
  padding: 0;  
}

body {  
  background-color: var(--bg-primary);  
  color: var(--text-main);  
  font-family: var(--font-sans);  
  line-height: 1.5;  
  padding-bottom: 60px;  
}

/\* Header \*/  
.header-bar {  
  background: linear-gradient(180deg, \#131b2e 0%, var(--bg-secondary) 100%);  
  border-bottom: 1px solid var(--border-color);  
  padding: 18px 28px;  
  display: flex;  
  justify-content: space-between;  
  align-items: center;  
  position: sticky;  
  top: 0;  
  z-index: 100;  
  backdrop-filter: blur(10px);  
}

.brand-wrapper {  
  display: flex;  
  align-items: center;  
  gap: 14px;  
}

.brand-logo {  
  width: 38px;  
  height: 38px;  
  background: linear-gradient(135deg, \#8b5cf6, \#3b82f6);  
  border-radius: var(--radius-sm);  
  display: flex;  
  align-items: center;  
  justify-content: center;  
  box-shadow: 0 0 15px rgba(139, 92, 246, 0.3);  
}

.brand-title {  
  font-size: 1.25rem;  
  font-weight: 700;  
  display: flex;  
  align-items: center;  
  gap: 8px;  
}

.brand-subtitle {  
  font-size: 0.75rem;  
  color: var(--text-muted);  
}

.header-badges {  
  display: flex;  
  gap: 10px;  
  align-items: center;  
}

.badge {  
  font-size: 0.75rem;  
  font-weight: 600;  
  padding: 4px 10px;  
  border-radius: 9999px;  
  display: inline-flex;  
  align-items: center;  
  gap: 6px;  
  text-transform: uppercase;  
  letter-spacing: 0.05em;  
}

.badge-success {  
  background: rgba(16, 185, 129, 0.15);  
  color: \#34d399;  
  border: 1px solid rgba(16, 185, 129, 0.4);  
}

.badge-info {  
  background: rgba(59, 130, 246, 0.15);  
  color: \#60a5fa;  
  border: 1px solid rgba(59, 130, 246, 0.4);  
}

.badge-warning {  
  background: rgba(245, 158, 11, 0.15);  
  color: \#fbbf24;  
  border: 1px solid rgba(245, 158, 11, 0.4);  
}

.badge-danger {  
  background: rgba(239, 68, 68, 0.15);  
  color: \#f87171;  
  border: 1px solid rgba(239, 68, 68, 0.4);  
}

/\* App Container \*/  
.container {  
  max-width: 1280px;  
  margin: 24px auto;  
  padding: 0 20px;  
}

/\* Control Row \*/  
.control-banner {  
  background: var(--bg-secondary);  
  border: 1px solid var(--border-color);  
  border-radius: var(--radius-lg);  
  padding: 16px 24px;  
  display: flex;  
  justify-content: space-between;  
  align-items: center;  
  flex-wrap: wrap;  
  gap: 16px;  
  margin-bottom: 24px;  
}

.preset-controls {  
  display: flex;  
  align-items: center;  
  gap: 12px;  
}

.select-input, .text-input, .number-input, .textarea-input {  
  background: var(--bg-tertiary);  
  border: 1px solid var(--border-color);  
  color: var(--text-main);  
  padding: 8px 14px;  
  border-radius: var(--radius-sm);  
  font-size: 0.875rem;  
  outline: none;  
  transition: border-color 0.2s;  
}

.select-input:focus, .text-input:focus, .number-input:focus, .textarea-input:focus {  
  border-color: var(--border-focus);  
}

/\* Score Cards \*/  
.stats-grid {  
  display: grid;  
  grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));  
  gap: 16px;  
  margin-bottom: 24px;  
}

.stat-card {  
  background: var(--bg-secondary);  
  border: 1px solid var(--border-color);  
  border-radius: var(--radius-md);  
  padding: 16px;  
  display: flex;  
  flex-direction: column;  
  gap: 6px;  
  position: relative;  
  overflow: hidden;  
}

.stat-card::after {  
  content: "";  
  position: absolute;  
  top: 0;  
  left: 0;  
  width: 4px;  
  height: 100%;  
}

.stat-card.grade-stat::after { background: var(--blue-accent); }  
.stat-card.score-stat::after { background: var(--emerald-accent); }  
.stat-card.sensor-stat::after { background: var(--purple-accent); }  
.stat-card.coverage-stat::after { background: var(--amber-accent); }

.stat-title {  
  font-size: 0.75rem;  
  text-transform: uppercase;  
  letter-spacing: 0.05em;  
  color: var(--text-dim);  
  font-weight: 600;  
}

.stat-value {  
  font-size: 1.6rem;  
  font-weight: 700;  
  display: flex;  
  align-items: baseline;  
  gap: 6px;  
}

.stat-desc {  
  font-size: 0.8rem;  
  color: var(--text-muted);  
}

/\* Tabs \*/  
.tabs-nav {  
  display: flex;  
  gap: 8px;  
  border-bottom: 1px solid var(--border-color);  
  margin-bottom: 24px;  
  overflow-x: auto;  
  padding-bottom: 2px;  
}

.tab-btn {  
  background: transparent;  
  border: none;  
  color: var(--text-muted);  
  font-size: 0.9rem;  
  font-weight: 600;  
  padding: 10px 18px;  
  cursor: pointer;  
  border-radius: var(--radius-sm) var(--radius-sm) 0 0;  
  transition: all 0.2s ease;  
  display: flex;  
  align-items: center;  
  gap: 8px;  
  white-space: nowrap;  
}

.tab-btn:hover {  
  color: var(--text-main);  
  background: rgba(255, 255, 255, 0.03);  
}

.tab-btn.active {  
  color: var(--blue-accent);  
  background: var(--bg-secondary);  
  border-bottom: 2px solid var(--blue-accent);  
}

/\* Tab Panels \*/  
.tab-panel {  
  display: none;  
}

.tab-panel.active {  
  display: block;  
}

/\* Layouts \*/  
.panel-grid {  
  display: grid;  
  grid-template-columns: 1.1fr 0.9fr;  
  gap: 24px;  
}

@media (max-width: 960px) {  
  .panel-grid {  
    grid-template-columns: 1fr;  
  }  
}

.card {  
  background: var(--bg-secondary);  
  border: 1px solid var(--border-color);  
  border-radius: var(--radius-md);  
  padding: 20px;  
  margin-bottom: 20px;  
}

.card-title {  
  font-size: 1rem;  
  font-weight: 700;  
  margin-bottom: 14px;  
  display: flex;  
  justify-content: space-between;  
  align-items: center;  
}

/\* Feature Categories Matrix \*/  
.category-block {  
  margin-bottom: 20px;  
  background: var(--bg-tertiary);  
  border: 1px solid var(--border-color);  
  border-radius: var(--radius-sm);  
  padding: 14px;  
}

.category-title {  
  font-size: 0.85rem;  
  font-weight: 700;  
  color: \#93c5fd;  
  text-transform: uppercase;  
  letter-spacing: 0.05em;  
  margin-bottom: 10px;  
  display: flex;  
  justify-content: space-between;  
  align-items: center;  
}

.feature-row {  
  display: flex;  
  justify-content: space-between;  
  align-items: center;  
  padding: 8px 0;  
  border-bottom: 1px solid rgba(255, 255, 255, 0.05);  
}

.feature-row:last-child {  
  border-bottom: none;  
}

.feature-meta {  
  display: flex;  
  flex-direction: column;  
  gap: 2px;  
}

.feature-name {  
  font-size: 0.85rem;  
  font-weight: 600;  
  color: var(--text-main);  
  font-family: var(--font-mono);  
}

.feature-label {  
  font-size: 0.75rem;  
  color: var(--text-muted);  
}

/\* 4-Way Segment Buttons \*/  
.segment-group {  
  display: flex;  
  background: \#060911;  
  border-radius: var(--radius-sm);  
  padding: 2px;  
  border: 1px solid var(--border-color);  
}

.segment-btn {  
  background: transparent;  
  border: none;  
  color: var(--text-dim);  
  font-size: 0.75rem;  
  font-weight: 600;  
  padding: 4px 8px;  
  cursor: pointer;  
  border-radius: 4px;  
  transition: all 0.15s;  
}

.segment-btn:hover {  
  color: var(--text-main);  
}

.segment-btn.active\[data-type="none"\] {  
  background: rgba(16, 185, 129, 0.25);  
  color: \#34d399;  
  font-weight: 700;  
}

.segment-btn.active\[data-type="self"\] {  
  background: rgba(59, 130, 246, 0.25);  
  color: \#60a5fa;  
  font-weight: 700;  
}

.segment-btn.active\[data-type="custom"\] {  
  background: rgba(139, 92, 246, 0.25);  
  color: \#c084fc;  
  font-weight: 700;  
}

.segment-btn.active\[data-type="wildcard"\] {  
  background: rgba(239, 68, 68, 0.25);  
  color: \#f87171;  
  font-weight: 700;  
}

/\* Code Snippet Box \*/  
.code-box {  
  background: \#060911;  
  border: 1px solid var(--border-color);  
  border-radius: var(--radius-sm);  
  padding: 14px;  
  font-family: var(--font-mono);  
  font-size: 0.85rem;  
  color: \#38bdf8;  
  position: relative;  
  overflow-x: auto;  
  white-space: pre-wrap;  
  word-break: break-all;  
}

.code-header {  
  display: flex;  
  justify-content: space-between;  
  align-items: center;  
  margin-bottom: 8px;  
}

/\* Buttons \*/  
.btn {  
  background: var(--bg-tertiary);  
  border: 1px solid var(--border-color);  
  color: var(--text-main);  
  padding: 8px 16px;  
  border-radius: var(--radius-sm);  
  font-size: 0.85rem;  
  font-weight: 600;  
  cursor: pointer;  
  display: inline-flex;  
  align-items: center;  
  gap: 8px;  
  transition: all 0.2s;  
}

.btn:hover {  
  background: var(--border-color);  
}

.btn-primary {  
  background: var(--blue-accent);  
  border-color: var(--blue-accent);  
  color: \#ffffff;  
}

.btn-primary:hover {  
  background: \#2563eb;  
}

.btn-success {  
  background: var(--emerald-accent);  
  border-color: var(--emerald-accent);  
  color: \#ffffff;  
}

.btn-success:hover {  
  background: \#059669;  
}

.btn-sm {  
  padding: 4px 10px;  
  font-size: 0.75rem;  
}

/\* Checklist Items \*/  
.check-list {  
  display: flex;  
  flex-direction: column;  
  gap: 10px;  
}

.check-item {  
  background: var(--bg-tertiary);  
  border: 1px solid var(--border-color);  
  border-radius: var(--radius-sm);  
  padding: 12px 16px;  
  display: flex;  
  flex-direction: column;  
  gap: 4px;  
}

.check-item.pass { border-left: 4px solid var(--emerald-accent); }  
.check-item.fail { border-left: 4px solid var(--red-accent); }  
.check-item.warn { border-left: 4px solid var(--amber-accent); }

.check-header-row {  
  display: flex;  
  justify-content: space-between;  
  align-items: center;  
}

.check-title {  
  font-size: 0.875rem;  
  font-weight: 700;  
  display: flex;  
  align-items: center;  
  gap: 8px;  
}

.check-points {  
  font-family: var(--font-mono);  
  font-size: 0.75rem;  
  color: var(--text-dim);  
}

.check-summary {  
  font-size: 0.8rem;  
  color: var(--text-muted);  
}

.check-details {  
  font-size: 0.75rem;  
  color: var(--text-dim);  
  font-family: var(--font-mono);  
}

/\* Simulator View \*/  
.sim-controls {  
  display: flex;  
  gap: 12px;  
  flex-wrap: wrap;  
  align-items: center;  
  background: var(--bg-tertiary);  
  padding: 14px;  
  border-radius: var(--radius-sm);  
}

.sim-timeline {  
  display: flex;  
  flex-direction: column;  
  gap: 12px;  
  position: relative;  
  margin-top: 10px;  
}

.sim-timeline::before {  
  content: "";  
  position: absolute;  
  top: 14px;  
  bottom: 14px;  
  left: 20px;  
  width: 2px;  
  background: var(--border-color);  
  z-index: 1;  
}

.sim-step {  
  display: flex;  
  gap: 16px;  
  align-items: flex-start;  
  position: relative;  
  z-index: 2;  
}

.sim-step-num {  
  width: 40px;  
  height: 40px;  
  border-radius: 50%;  
  background: var(--bg-secondary);  
  border: 2px solid var(--border-color);  
  display: flex;  
  align-items: center;  
  justify-content: center;  
  font-weight: 700;  
  font-size: 0.85rem;  
  flex-shrink: 0;  
}

.sim-step.pass .sim-step-num {  
  border-color: var(--emerald-accent);  
  color: var(--emerald-accent);  
}

.sim-step.fail .sim-step-num {  
  border-color: var(--red-accent);  
  color: var(--red-accent);  
}

.sim-step-card {  
  background: var(--bg-secondary);  
  border: 1px solid var(--border-color);  
  border-radius: var(--radius-sm);  
  padding: 12px 16px;  
  flex-grow: 1;  
}

.sim-step-meta {  
  display: flex;  
  justify-content: space-between;  
  align-items: center;  
  margin-bottom: 6px;  
}

.sim-actor {  
  font-size: 0.8rem;  
  font-weight: 700;  
  color: var(--text-main);  
}

/\* Feature Reference Table \*/  
.feature-table {  
  width: 100%;  
  border-collapse: collapse;  
  margin-top: 12px;  
}

.feature-table th, .feature-table td {  
  padding: 10px 14px;  
  text-align: left;  
  border-bottom: 1px solid var(--border-color);  
  font-size: 0.85rem;  
}

.feature-table th {  
  background: var(--bg-tertiary);  
  color: var(--text-dim);  
  text-transform: uppercase;  
  font-size: 0.7rem;  
  letter-spacing: 0.05em;  
}

.feature-table tr:hover td {  
  background: rgba(255, 255, 255, 0.02);  
}

/\* Toast \*/  
.toast {  
  position: fixed;  
  bottom: 24px;  
  right: 24px;  
  background: \#1f2937;  
  border: 1px solid var(--emerald-accent);  
  color: \#ffffff;  
  padding: 10px 18px;  
  border-radius: var(--radius-sm);  
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.5);  
  font-size: 0.85rem;  
  font-weight: 600;  
  display: none;  
  z-index: 1000;  
}  
