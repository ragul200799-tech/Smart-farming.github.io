<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>AgroSense — Smart Irrigation Dashboard</title>
<link href="https://fonts.googleapis.com/css2?family=Syne:wght@400;600;700;800&family=DM+Mono:wght@300;400;500&display=swap" rel="stylesheet">
<style>
  :root {
    --soil-dark: #1a1208;
    --soil-mid: #2d1f0a;
    --leaf-deep: #0d3320;
    --leaf-mid: #1a5c3a;
    --leaf-bright: #2d9156;
    --leaf-glow: #3dbf72;
    --leaf-light: #6ee09a;
    --leaf-pale: #b8f0cc;
    --sun-gold: #f5c842;
    --water-blue: #4ab8e8;
    --danger: #e85c4a;
    --text-primary: #e8f5ec;
    --text-secondary: #8dbfa0;
    --text-muted: #4d7a5e;
    --card-bg: rgba(26, 42, 30, 0.85);
    --card-border: rgba(61, 191, 114, 0.15);
    --glow: rgba(61, 191, 114, 0.4);
  }

  * { margin: 0; padding: 0; box-sizing: border-box; }

  body {
    font-family: 'DM Mono', monospace;
    background: var(--soil-dark);
    color: var(--text-primary);
    min-height: 100vh;
    overflow-x: hidden;
    position: relative;
  }

  .bg-layer {
    position: fixed; inset: 0; z-index: 0;
    background:
      radial-gradient(ellipse 80% 60% at 10% 90%, rgba(26, 92, 58, 0.3) 0%, transparent 60%),
      radial-gradient(ellipse 60% 80% at 90% 10%, rgba(13, 51, 32, 0.5) 0%, transparent 60%),
      radial-gradient(ellipse 100% 100% at 50% 50%, #1a1208 0%, #0d1a10 100%);
  }

  .bg-grid {
    position: fixed; inset: 0; z-index: 0;
    background-image: linear-gradient(rgba(61,191,114,0.04) 1px, transparent 1px), linear-gradient(90deg, rgba(61,191,114,0.04) 1px, transparent 1px);
    background-size: 48px 48px;
    animation: gridDrift 30s linear infinite;
  }
  @keyframes gridDrift { 100% { transform: translate(48px, 48px); } }

  .particle {
    position: fixed; border-radius: 50%; pointer-events: none; z-index: 0;
    animation: floatUp linear infinite; opacity: 0;
  }
  @keyframes floatUp {
    0% { transform: translateY(100vh) scale(0); opacity: 0; }
    10% { opacity: 0.6; } 90% { opacity: 0.2; }
    100% { transform: translateY(-10vh) scale(1); opacity: 0; }
  }

  .wrapper { position: relative; z-index: 1; max-width: 1280px; margin: 0 auto; padding: 0 24px 48px; }

  header {
    display: flex; align-items: center; justify-content: space-between;
    padding: 28px 0 32px; border-bottom: 1px solid var(--card-border); margin-bottom: 36px;
  }

  .logo { display: flex; align-items: center; gap: 14px; }
  .logo-icon { width: 44px; height: 44px; display: flex; align-items: center; justify-content: center; }
  .logo-icon svg { width: 44px; height: 44px; filter: drop-shadow(0 0 12px var(--leaf-glow)); }
  .logo-text { font-family: 'Syne', sans-serif; font-size: 22px; font-weight: 800; color: var(--text-primary); letter-spacing: -0.5px; }
  .logo-sub { font-size: 10px; color: var(--text-muted); letter-spacing: 3px; text-transform: uppercase; display: block; margin-top: 2px; }

  .header-right { display: flex; align-items: center; gap: 20px; }
  .live-badge {
    display: flex; align-items: center; gap: 8px;
    background: rgba(61, 191, 114, 0.1); border: 1px solid rgba(61, 191, 114, 0.25);
    padding: 6px 14px; border-radius: 100px; font-size: 11px; letter-spacing: 2px; text-transform: uppercase; color: var(--leaf-light);
  }
  .live-dot { width: 7px; height: 7px; border-radius: 50%; background: var(--leaf-glow); box-shadow: 0 0 8px var(--leaf-glow); animation: pulse 2s ease-in-out infinite; }
  @keyframes pulse { 0%, 100% { transform: scale(1); opacity: 1; } 50% { transform: scale(1.4); opacity: 0.7; } }
  .timestamp { font-size: 11px; color: var(--text-muted); letter-spacing: 1px; }

  .section-label {
    font-size: 10px; letter-spacing: 3px; text-transform: uppercase; color: var(--text-muted);
    margin-bottom: 16px; display: flex; align-items: center; gap: 10px;
  }
  .section-label::after { content: ''; flex: 1; height: 1px; background: var(--card-border); }

  .dashboard-grid { display: grid; grid-template-columns: 1fr 1fr 1fr; gap: 20px; margin-bottom: 24px; }

  .card {
    background: var(--card-bg); border: 1px solid var(--card-border); border-radius: 20px; padding: 28px;
    backdrop-filter: blur(20px); position: relative; overflow: hidden;
    transition: border-color 0.3s, box-shadow 0.3s; animation: fadeInUp 0.6s ease both;
  }
  .card:hover { border-color: rgba(61,191,114,0.3); box-shadow: 0 0 40px rgba(61,191,114,0.08); }
  .card::before {
    content: ''; position: absolute; top: 0; left: 0; right: 0; height: 1px;
    background: linear-gradient(90deg, transparent, rgba(61,191,114,0.4), transparent);
  }
  @keyframes fadeInUp { from { opacity: 0; transform: translateY(20px); } to { opacity: 1; transform: translateY(0); } }

  .card-moisture {
    grid-column: 1 / 3; display: flex; align-items: center; gap: 40px; padding: 36px;
    animation-delay: 0.1s;
  }
  .gauge-wrap { position: relative; width: 200px; height: 200px; flex-shrink: 0; }
  .gauge-svg { width: 200px; height: 200px; transform: rotate(-90deg); }
  .gauge-track { fill: none; stroke: rgba(61,191,114,0.1); stroke-width: 12; }
  .gauge-fill {
    fill: none; stroke-width: 12; stroke-linecap: round; stroke: url(#moistureGrad);
    stroke-dasharray: 502; stroke-dashoffset: 502;
    transition: stroke-dashoffset 1.5s cubic-bezier(0.34, 1.56, 0.64, 1);
    filter: drop-shadow(0 0 8px rgba(61,191,114,0.6));
  }
  .gauge-center { position: absolute; inset: 0; display: flex; flex-direction: column; align-items: center; justify-content: center; }
  .gauge-value { font-family: 'Syne', sans-serif; font-size: 48px; font-weight: 800; color: var(--text-primary); line-height: 1; transition: color 0.5s; }
  .gauge-unit { font-size: 16px; color: var(--leaf-light); margin-top: 4px; }

  .moisture-info { flex: 1; }
  .moisture-label { font-size: 11px; letter-spacing: 3px; text-transform: uppercase; color: var(--text-muted); margin-bottom: 8px; }
  .moisture-title { font-family: 'Syne', sans-serif; font-size: 30px; font-weight: 700; color: var(--text-primary); margin-bottom: 12px; }
  .moisture-status-text { font-size: 13px; color: var(--text-secondary); line-height: 1.7; margin-bottom: 24px; }
  .moisture-status-text strong { font-weight: 500; }

  .zones { display: flex; gap: 6px; }
  .zone { flex: 1; height: 5px; border-radius: 3px; opacity: 0.3; transition: opacity 0.5s, box-shadow 0.5s; }
  .zone.active { opacity: 1; }
  .zone-dry { background: var(--danger); } .zone-dry.active { box-shadow: 0 0 8px var(--danger); }
  .zone-low { background: var(--sun-gold); } .zone-low.active { box-shadow: 0 0 8px var(--sun-gold); }
  .zone-ok { background: var(--leaf-bright); } .zone-ok.active { box-shadow: 0 0 8px var(--leaf-bright); }
  .zone-good { background: var(--leaf-glow); } .zone-good.active { box-shadow: 0 0 8px var(--leaf-glow); }
  .zone-high { background: var(--water-blue); } .zone-high.active { box-shadow: 0 0 8px var(--water-blue); }
  .zone-labels { display: flex; justify-content: space-between; margin-top: 6px; font-size: 9px; color: var(--text-muted); letter-spacing: 1px; }

  .card-pump { display: flex; flex-direction: column; animation-delay: 0.2s; }
  .pump-header { display: flex; justify-content: space-between; align-items: flex-start; margin-bottom: 28px; }
  .card-tag { font-size: 10px; letter-spacing: 2.5px; text-transform: uppercase; color: var(--text-muted); }

  .pump-status-badge { display: flex; align-items: center; gap: 7px; padding: 5px 12px; border-radius: 100px; font-size: 10px; letter-spacing: 2px; text-transform: uppercase; font-weight: 500; transition: all 0.4s; }
  .pump-status-badge.on { background: rgba(74, 184, 232, 0.15); border: 1px solid rgba(74, 184, 232, 0.4); color: var(--water-blue); }
  .pump-status-badge.off { background: rgba(77, 122, 94, 0.15); border: 1px solid rgba(77, 122, 94, 0.3); color: var(--text-muted); }
  .pump-status-dot { width: 6px; height: 6px; border-radius: 50%; transition: all 0.4s; }
  .pump-status-badge.on .pump-status-dot { background: var(--water-blue); box-shadow: 0 0 8px var(--water-blue); animation: pulse 1.5s ease-in-out infinite; }
  .pump-status-badge.off .pump-status-dot { background: var(--text-muted); }

  .pump-icon-wrap { display: flex; justify-content: center; margin: 10px 0 24px; flex: 1; align-items: center; }
  .pump-icon { width: 90px; height: 90px; border-radius: 50%; display: flex; align-items: center; justify-content: center; background: rgba(26, 42, 30, 0.9); border: 2px solid rgba(74,184,232,0.2); transition: all 0.5s; position: relative; }
  .pump-icon.on { border-color: rgba(74,184,232,0.6); box-shadow: 0 0 30px rgba(74,184,232,0.3), inset 0 0 20px rgba(74,184,232,0.05); }
  .pump-icon.on::before { content: ''; position: absolute; inset: -8px; border-radius: 50%; border: 1px solid rgba(74,184,232,0.2); animation: ripple 2s linear infinite; }
  .pump-icon.on::after { content: ''; position: absolute; inset: -16px; border-radius: 50%; border: 1px solid rgba(74,184,232,0.1); animation: ripple 2s linear 0.5s infinite; }
  @keyframes ripple { 0% { transform: scale(0.8); opacity: 1; } 100% { transform: scale(1.3); opacity: 0; } }
  .pump-icon svg { width: 44px; height: 44px; transition: all 0.5s; }

  .pump-toggle { width: 100%; padding: 14px; border-radius: 12px; font-family: 'DM Mono', monospace; font-size: 12px; letter-spacing: 2px; text-transform: uppercase; font-weight: 500; cursor: pointer; border: 1px solid; transition: all 0.3s; }
  .pump-toggle:hover { transform: translateY(-1px); }
  .pump-toggle:active { transform: scale(0.98); }
  .pump-toggle.turn-off { background: rgba(232, 92, 74, 0.1); border-color: rgba(232, 92, 74, 0.4); color: #e85c4a; }
  .pump-toggle.turn-on { background: rgba(74, 184, 232, 0.1); border-color: rgba(74, 184, 232, 0.4); color: var(--water-blue); }

  .cards-row2 { display: grid; grid-template-columns: repeat(3, 1fr); gap: 20px; margin-bottom: 24px; }
  .cards-row2 .card:nth-child(1) { animation-delay: 0.3s; }
  .cards-row2 .card:nth-child(2) { animation-delay: 0.4s; }
  .cards-row2 .card:nth-child(3) { animation-delay: 0.5s; }

  .stat-value { font-family: 'Syne', sans-serif; font-size: 36px; font-weight: 700; line-height: 1; margin: 20px 0 6px; }
  .stat-sub { font-size: 12px; color: var(--text-muted); }
  .stat-icon { width: 40px; height: 40px; display: flex; align-items: center; justify-content: center; border-radius: 10px; margin-bottom: 4px; }

  .card-history { grid-column: 1 / 3; }
  .chart-wrap { margin-top: 16px; height: 100px; position: relative; }
  .chart-svg { width: 100%; height: 100%; overflow: visible; }

  .auto-mode-wrap { display: flex; flex-direction: column; gap: 16px; margin-top: 16px; }
  .toggle-row { display: flex; justify-content: space-between; align-items: center; }
  .toggle-label { font-size: 13px; color: var(--text-secondary); }
  .toggle-switch { width: 46px; height: 24px; background: rgba(61,191,114,0.1); border: 1px solid rgba(61,191,114,0.2); border-radius: 100px; cursor: pointer; position: relative; transition: all 0.3s; }
  .toggle-switch.active { background: rgba(61,191,114,0.25); border-color: rgba(61,191,114,0.5); }
  .toggle-knob { width: 18px; height: 18px; border-radius: 50%; background: var(--text-muted); position: absolute; top: 2px; left: 3px; transition: all 0.3s; }
  .toggle-switch.active .toggle-knob { background: var(--leaf-glow); left: 25px; box-shadow: 0 0 8px rgba(61,191,114,0.5); }
  .threshold-info { font-size: 11px; color: var(--text-muted); padding: 10px 14px; background: rgba(0,0,0,0.2); border-radius: 10px; line-height: 1.6; }

  .alert-bar { display: flex; align-items: center; gap: 12px; padding: 14px 20px; border-radius: 12px; margin-bottom: 20px; font-size: 12px; border: 1px solid; transition: all 0.5s; opacity: 0; transform: translateY(-4px); }
  .alert-bar.visible { opacity: 1; transform: translateY(0); }
  .alert-bar.warning { background: rgba(245, 200, 66, 0.1); border-color: rgba(245, 200, 66, 0.3); color: var(--sun-gold); }
  .alert-bar.info { background: rgba(74, 184, 232, 0.1); border-color: rgba(74, 184, 232, 0.3); color: var(--water-blue); }
  .alert-bar.success { background: rgba(61, 191, 114, 0.1); border-color: rgba(61, 191, 114, 0.3); color: var(--leaf-light); }

  .water-drop { position: fixed; pointer-events: none; font-size: 14px; opacity: 0; animation: dropFall 1s ease-in forwards; z-index: 100; }
  @keyframes dropFall { 0% { opacity: 1; transform: translateY(0); } 100% { opacity: 0; transform: translateY(30px); } }

  @media (max-width: 900px) {
    .dashboard-grid { grid-template-columns: 1fr; }
    .card-moisture { grid-column: 1; flex-direction: column; text-align: center; }
    .cards-row2 { grid-template-columns: 1fr 1fr; }
    .card-history { grid-column: 1 / 3; }
  }
  @media (max-width: 600px) {
    .cards-row2 { grid-template-columns: 1fr; }
    .card-history { grid-column: 1; }
    header { flex-direction: column; gap: 16px; }
  }
</style>
</head>
<body>
<div class="bg-layer"></div>
<div class="bg-grid"></div>
<div id="particles"></div>

<div class="wrapper">

  <header>
    <div class="logo">
      <div class="logo-icon">
        <svg viewBox="0 0 44 44" fill="none">
          <circle cx="22" cy="22" r="20" fill="rgba(45,145,86,0.15)" stroke="rgba(61,191,114,0.4)" stroke-width="1.5"/>
          <path d="M22 32C22 32 14 26 14 19C14 14.58 17.58 11 22 11C26.42 11 30 14.58 30 19C30 26 22 32 22 32Z" fill="rgba(45,145,86,0.5)" stroke="#3dbf72" stroke-width="1.5"/>
          <path d="M22 20V28" stroke="#6ee09a" stroke-width="1.5" stroke-linecap="round"/>
          <circle cx="22" cy="19" r="2.5" fill="#3dbf72"/>
        </svg>
      </div>
      <div>
        <span class="logo-text">AgroSense</span>
        <span class="logo-sub">IoT Irrigation System</span>
      </div>
    </div>
    <div class="header-right">
      <div class="live-badge"><div class="live-dot"></div>Live Monitoring</div>
      <div class="timestamp" id="timestamp">--:--:--</div>
    </div>
  </header>

  <div class="alert-bar" id="alertBar">
    <svg width="16" height="16" viewBox="0 0 16 16" fill="currentColor"><path d="M8 1a7 7 0 100 14A7 7 0 008 1zm0 4a.75.75 0 01.75.75v3.5a.75.75 0 01-1.5 0v-3.5A.75.75 0 018 5zm0 6.5a1 1 0 110-2 1 1 0 010 2z"/></svg>
    <span id="alertText">Initializing sensors...</span>
  </div>

  <div class="section-label">Primary Sensors</div>

  <div class="dashboard-grid">

    <!-- MOISTURE CARD -->
    <div class="card card-moisture">
      <div class="gauge-wrap">
        <svg class="gauge-svg" viewBox="0 0 200 200">
          <defs>
            <linearGradient id="moistureGrad" x1="0%" y1="0%" x2="100%" y2="100%">
              <stop offset="0%" stop-color="#2d9156"/>
              <stop offset="50%" stop-color="#3dbf72"/>
              <stop offset="100%" stop-color="#4ab8e8"/>
            </linearGradient>
          </defs>
          <circle class="gauge-track" cx="100" cy="100" r="80"/>
          <circle class="gauge-fill" id="gaugeFill" cx="100" cy="100" r="80"/>
        </svg>
        <div class="gauge-center">
          <div class="gauge-value" id="moistureValue">--</div>
          <div class="gauge-unit">%</div>
        </div>
      </div>
      <div class="moisture-info">
        <div class="moisture-label">Soil Moisture Level</div>
        <div class="moisture-title" id="moistureTitle">Connecting...</div>
        <div class="moisture-status-text" id="moistureDesc">Loading sensor data...<br><strong>Please wait.</strong></div>
        <div class="zones">
          <div class="zone zone-dry" id="zone-dry"></div>
          <div class="zone zone-low" id="zone-low"></div>
          <div class="zone zone-ok" id="zone-ok"></div>
          <div class="zone zone-good" id="zone-good"></div>
          <div class="zone zone-high" id="zone-high"></div>
        </div>
        <div class="zone-labels"><span>Dry</span><span>Low</span><span>Optimal</span><span>Good</span><span>Saturated</span></div>
      </div>
    </div>

    <!-- PUMP CARD -->
    <div class="card card-pump">
      <div class="pump-header">
        <div class="card-tag">Water Pump</div>
        <div class="pump-status-badge off" id="pumpBadge">
          <div class="pump-status-dot"></div>
          <span id="pumpBadgeText">Standby</span>
        </div>
      </div>
      <div class="pump-icon-wrap">
        <div class="pump-icon off" id="pumpIcon">
          <svg id="pumpSvg" viewBox="0 0 44 44" fill="none">
            <path d="M10 30C10 25.5 14 22 18 22H26C30 22 34 25.5 34 30" stroke="#4d7a5e" stroke-width="1.8" stroke-linecap="round"/>
            <circle cx="22" cy="16" r="6" stroke="#4d7a5e" stroke-width="1.8"/>
            <path d="M22 22V26" stroke="#4d7a5e" stroke-width="1.8" stroke-linecap="round"/>
            <rect x="14" y="30" width="16" height="6" rx="2" stroke="#4d7a5e" stroke-width="1.8"/>
            <path d="M18 30V36M26 30V36" stroke="#4d7a5e" stroke-width="1.2" stroke-linecap="round" opacity="0.5"/>
            <path d="M7 34H12M32 34H37" stroke="#4d7a5e" stroke-width="1.5" stroke-linecap="round" opacity="0.4"/>
          </svg>
        </div>
      </div>
      <button class="pump-toggle turn-on" id="pumpToggle" onclick="togglePump()">⚡ Turn Pump ON</button>
    </div>

  </div>

  <div class="section-label">Analytics &amp; Controls</div>

  <div class="cards-row2">

    <!-- Temperature -->
    <div class="card">
      <div class="card-tag">Ambient Temperature</div>
      <div class="stat-icon" style="background:rgba(245,200,66,0.1); margin-top:8px;">
        <svg width="22" height="22" viewBox="0 0 22 22" fill="none">
          <path d="M11 4v8.5" stroke="#f5c842" stroke-width="1.5" stroke-linecap="round"/>
          <circle cx="11" cy="15.5" r="3" stroke="#f5c842" stroke-width="1.5"/>
          <path d="M13.5 4h-1m1 2.5h-1m1 2.5h-1" stroke="#f5c842" stroke-width="1.3" stroke-linecap="round"/>
        </svg>
      </div>
      <div class="stat-value" id="tempValue" style="color: var(--sun-gold);">--°C</div>
      <div class="stat-sub" id="tempStatus">Reading sensor...</div>
    </div>

    <!-- History Chart -->
    <div class="card card-history">
      <div class="card-tag">Moisture History — Last 24 Readings</div>
      <div class="chart-wrap">
        <svg class="chart-svg" id="historyChart" viewBox="0 0 600 100" preserveAspectRatio="none">
          <defs>
            <linearGradient id="chartGrad" x1="0" y1="0" x2="0" y2="1">
              <stop offset="0%" stop-color="#3dbf72" stop-opacity="0.35"/>
              <stop offset="100%" stop-color="#3dbf72" stop-opacity="0"/>
            </linearGradient>
            <linearGradient id="chartLineGrad" x1="0%" y1="0%" x2="100%" y2="0%">
              <stop offset="0%" stop-color="#2d9156"/>
              <stop offset="60%" stop-color="#3dbf72"/>
              <stop offset="100%" stop-color="#4ab8e8"/>
            </linearGradient>
          </defs>
          <!-- Threshold line at 35% -->
          <line id="threshLine" x1="0" y1="75" x2="600" y2="75" stroke="rgba(245,200,66,0.3)" stroke-width="1" stroke-dasharray="5,4"/>
          <path id="chartArea" fill="url(#chartGrad)"/>
          <path id="chartLine" fill="none" stroke="url(#chartLineGrad)" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"/>
          <circle id="chartDot" r="4" fill="#3dbf72" filter="drop-shadow(0 0 4px #3dbf72)"/>
        </svg>
      </div>
    </div>

    <!-- Automation -->
    <div class="card">
      <div class="card-tag">Automation</div>
      <div class="auto-mode-wrap">
        <div class="toggle-row">
          <span class="toggle-label">Auto Mode</span>
          <div class="toggle-switch active" id="autoSwitch" onclick="toggleAuto(this)">
            <div class="toggle-knob"></div>
          </div>
        </div>
        <div class="toggle-row">
          <span class="toggle-label">Alerts</span>
          <div class="toggle-switch active" id="alertSwitch" onclick="toggleThisSwitch(this)">
            <div class="toggle-knob"></div>
          </div>
        </div>
        <div class="threshold-info" id="thresholdInfo">
          Auto-irrigation triggers at<br>
          &lt; <strong style="color:var(--leaf-light)">35%</strong> moisture — stops at <strong style="color:var(--leaf-light)">75%</strong>
        </div>
      </div>
    </div>

  </div>
</div>

<script>
  let moisture = 62, pumpOn = false, autoMode = true;
  let history = [], temp = 26.3, direction = 1;

  function simulateSensor() {
    const drift = (Math.random() - 0.48) * 2.5;
    moisture = Math.max(5, Math.min(95, moisture + drift));
    if (Math.random() < 0.04) direction *= -1;
    if (pumpOn) {
      moisture = Math.min(95, moisture + 0.9);
      if (moisture >= 75 && autoMode) pumpOn = false;
    } else if (autoMode && moisture < 35) {
      pumpOn = true;
      spawnDrops();
    }
    temp += (Math.random() - 0.5) * 0.3;
    temp = Math.max(18, Math.min(42, temp));
    history.push(Math.round(moisture));
    if (history.length > 24) history.shift();
    updateUI();
  }

  function updateUI() {
    const val = Math.round(moisture);
    document.getElementById('moistureValue').textContent = val;

    const circumference = 502;
    document.getElementById('gaugeFill').style.strokeDashoffset = circumference - (val / 100) * circumference;

    ['dry','low','ok','good','high'].forEach(z => document.getElementById('zone-'+z).classList.remove('active'));
    const zoneMap = [[0,20,'dry'],[20,35,'low'],[35,50,'ok'],[50,70,'good'],[70,101,'high']];
    for (const [lo, hi, id] of zoneMap) {
      if (val >= lo && val < hi) { document.getElementById('zone-'+id).classList.add('active'); break; }
    }

    let title, desc, alertType, alertMsg;
    if (val < 20) {
      title = 'Critically Dry'; desc = 'Soil moisture is dangerously low. Crops may be experiencing <strong>severe stress</strong>. Immediate irrigation needed.';
      alertType = 'warning'; alertMsg = '⚠  Critical: Moisture below 20%. Auto-irrigation activated.';
    } else if (val < 35) {
      title = 'Low Moisture'; desc = 'Below optimal range. <strong>Irrigation recommended</strong> to prevent crop stress and yield loss.';
      alertType = 'warning'; alertMsg = '⚠  Low moisture detected. Pump has been activated automatically.';
    } else if (val < 50) {
      title = 'Adequate'; desc = 'Moisture is in an acceptable range. Monitor for any downward <strong>trend</strong> in the next few hours.';
      alertType = 'success'; alertMsg = '✓  Soil moisture within acceptable range. System nominal.';
    } else if (val < 70) {
      title = 'Well Hydrated'; desc = 'Soil moisture at an <strong>optimal level</strong>. Plants are well hydrated. No irrigation needed now.';
      alertType = 'success'; alertMsg = '✓  Optimal moisture level. All sensors operating normally.';
    } else {
      title = 'Saturated'; desc = 'Soil is near saturation. <strong>Irrigation paused</strong> to prevent waterlogging and root damage.';
      alertType = 'info'; alertMsg = 'ℹ  High moisture detected. Pump has been paused to prevent overwatering.';
    }

    document.getElementById('moistureTitle').textContent = title;
    document.getElementById('moistureDesc').innerHTML = desc + '<br><span style="font-size:11px;color:var(--text-muted);margin-top:4px;display:block">Sensor: Field A — Zone 3 — Node 07</span>';

    const bar = document.getElementById('alertBar');
    bar.className = 'alert-bar visible ' + alertType;
    document.getElementById('alertText').textContent = alertMsg;

    updatePumpUI();

    document.getElementById('tempValue').textContent = temp.toFixed(1) + '°C';
    const ts = temp > 35 ? 'High — monitor crops closely' : temp > 28 ? 'Warm — within range' : temp > 20 ? 'Mild — optimal growing' : 'Cool — frost watch';
    document.getElementById('tempStatus').textContent = ts;

    updateChart();
    document.getElementById('timestamp').textContent = new Date().toLocaleTimeString();
  }

  function updatePumpUI() {
    const badge = document.getElementById('pumpBadge');
    const icon = document.getElementById('pumpIcon');
    const toggle = document.getElementById('pumpToggle');
    const paths = document.getElementById('pumpSvg').querySelectorAll('path, circle, rect, line');
    const col = pumpOn ? '#4ab8e8' : '#4d7a5e';
    paths.forEach(el => { if (el.getAttribute('stroke')) el.setAttribute('stroke', col); });
    if (pumpOn) {
      badge.className = 'pump-status-badge on'; badge.querySelector('span').textContent = 'Running';
      icon.className = 'pump-icon on'; toggle.className = 'pump-toggle turn-off'; toggle.textContent = '⏹ Turn Pump OFF';
    } else {
      badge.className = 'pump-status-badge off'; badge.querySelector('span').textContent = 'Standby';
      icon.className = 'pump-icon off'; toggle.className = 'pump-toggle turn-on'; toggle.textContent = '⚡ Turn Pump ON';
    }
  }

  function updateChart() {
    if (history.length < 2) return;
    const w = 600, h = 100, pad = 8;
    const minV = Math.max(0, Math.min(...history) - 8);
    const maxV = Math.min(100, Math.max(...history) + 8);
    const range = maxV - minV || 1;
    const pts = history.map((v, i) => {
      const x = (i / (history.length - 1)) * w;
      const y = h - pad - ((v - minV) / range) * (h - pad * 2);
      return [x, y];
    });
    // Smooth curve using catmull-rom
    let path = `M${pts[0][0]},${pts[0][1]}`;
    for (let i = 1; i < pts.length; i++) {
      const p0 = pts[Math.max(0,i-2)], p1 = pts[i-1], p2 = pts[i], p3 = pts[Math.min(pts.length-1,i+1)];
      const cp1x = p1[0] + (p2[0] - p0[0]) / 6, cp1y = p1[1] + (p2[1] - p0[1]) / 6;
      const cp2x = p2[0] - (p3[0] - p1[0]) / 6, cp2y = p2[1] - (p3[1] - p1[1]) / 6;
      path += ` C${cp1x},${cp1y} ${cp2x},${cp2y} ${p2[0]},${p2[1]}`;
    }
    const last = pts[pts.length-1];
    document.getElementById('chartLine').setAttribute('d', path);
    document.getElementById('chartArea').setAttribute('d', path + ` L${last[0]},${h} L0,${h} Z`);
    document.getElementById('chartDot').setAttribute('cx', last[0]);
    document.getElementById('chartDot').setAttribute('cy', last[1]);
    // threshold line
    const threshY = h - pad - ((35 - minV) / range) * (h - pad*2);
    document.getElementById('threshLine').setAttribute('y1', threshY);
    document.getElementById('threshLine').setAttribute('y2', threshY);
  }

  function togglePump() {
    pumpOn = !pumpOn;
    updatePumpUI();
    if (pumpOn) spawnDrops();
  }

  function toggleAuto(el) {
    el.classList.toggle('active');
    autoMode = el.classList.contains('active');
    document.getElementById('thresholdInfo').innerHTML = autoMode
      ? 'Auto-irrigation triggers at<br>&lt; <strong style="color:var(--leaf-light)">35%</strong> moisture — stops at <strong style="color:var(--leaf-light)">75%</strong>'
      : 'Auto mode <strong style="color:var(--danger)">disabled</strong>. Manual control only.';
  }

  function toggleThisSwitch(el) { el.classList.toggle('active'); }

  function spawnDrops() {
    const icon = document.getElementById('pumpIcon');
    const rect = icon.getBoundingClientRect();
    for (let i = 0; i < 6; i++) {
      setTimeout(() => {
        const d = document.createElement('div');
        d.className = 'water-drop'; d.textContent = '💧';
        d.style.left = (rect.left + rect.width/2 + (Math.random()-0.5)*50) + 'px';
        d.style.top = (rect.top + rect.height/2) + 'px';
        document.body.appendChild(d);
        setTimeout(() => d.remove(), 1000);
      }, i * 120);
    }
  }

  // Particles
  const pc = document.getElementById('particles');
  for (let i = 0; i < 14; i++) {
    const p = document.createElement('div');
    p.className = 'particle';
    const s = Math.random()*4+2;
    p.style.cssText = `width:${s}px;height:${s}px;left:${Math.random()*100}vw;background:${Math.random()>.5?'rgba(61,191,114,0.5)':'rgba(74,184,232,0.35)'};animation-duration:${Math.random()*15+10}s;animation-delay:${Math.random()*10}s`;
    pc.appendChild(p);
  }

  // Init history
  let seed = 58;
  for (let i = 0; i < 20; i++) {
    seed += (Math.random()-0.48)*4;
    seed = Math.max(20, Math.min(80, seed));
    history.push(Math.round(seed));
  }

  updateUI();
  setInterval(simulateSensor, 3000);
  setInterval(() => { document.getElementById('timestamp').textContent = new Date().toLocaleTimeString(); }, 1000);
</script>
</body>
</html>
