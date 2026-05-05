<!DOCTYPE html>  
<html lang="en">  
<head>  
<meta charset="UTF-8">  
<meta name="viewport" content="width=device-width, initial-scale=1.0">  
<title>NoiseGuard — Smart Noise Pollution Monitoring</title>  
<style>  
  @import url('https://fonts.googleapis.com/css2?family=Space+Mono:wght@400;700&family=DM+Sans:wght@300;400;500;600&display=swap');  
  
  *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }  
  
  :root {  
    --bg: #0a0e17;  
    --surface: #111827;  
    --surface2: #1a2233;  
    --border: rgba(255,255,255,0.08);  
    --accent: #f59e0b;  
    --red: #ef4444;  
    --orange: #f97316;  
    --green: #22c55e;  
    --blue: #3b82f6;  
    --cyan: #06b6d4;  
    --text: #e2e8f0;  
    --muted: #64748b;  
    --mono: 'Space Mono', monospace;  
    --sans: 'DM Sans', sans-serif;  
  }  
  
  body { background: var(--bg); color: var(--text); font-family: var(--sans); min-height: 100vh; }  
  
  /* NAV */  
  nav {  
    display: flex; align-items: center; justify-content: space-between;  
    padding: 0 28px; height: 56px;  
    background: rgba(10,14,23,0.95); border-bottom: 1px solid var(--border);  
    position: sticky; top: 0; z-index: 100; backdrop-filter: blur(8px);  
  }  
  .logo { display: flex; align-items: center; gap: 10px; }  
  .logo-icon {  
    width: 32px; height: 32px; background: var(--accent);  
    border-radius: 6px; display: flex; align-items: center; justify-content: center;  
    font-size: 16px;  
  }  
  .logo-text { font-family: var(--mono); font-size: 15px; font-weight: 700; color: #fff; }  
  .logo-text span { color: var(--accent); }  
  .nav-tabs { display: flex; gap: 2px; }  
  .nav-tab {  
    padding: 6px 14px; border-radius: 6px; font-size: 13px; cursor: pointer;  
    color: var(--muted); border: none; background: none; font-family: var(--sans);  
    transition: all 0.15s;  
  }  
  .nav-tab:hover { color: var(--text); background: var(--surface2); }  
  .nav-tab.active { color: #fff; background: var(--surface2); }  
  .live-badge {  
    display: flex; align-items: center; gap: 6px;  
    font-family: var(--mono); font-size: 12px; color: var(--green);  
  }  
  .live-dot { width: 7px; height: 7px; border-radius: 50%; background: var(--green); animation: pulse 1.5s infinite; }  
  @keyframes pulse { 0%,100%{opacity:1;} 50%{opacity:0.4;} }  
  
  /* LAYOUT */  
  .content { padding: 20px 28px; display: none; }  
  .content.active { display: block; }  
  
  /* STAT CARDS */  
  .stat-row { display: grid; grid-template-columns: repeat(5,1fr); gap: 12px; margin-bottom: 18px; }  
  .stat-card {  
    background: var(--surface); border: 1px solid var(--border);  
    border-radius: 10px; padding: 14px 16px;  
  }  
  .stat-label { font-size: 10px; letter-spacing: 0.08em; color: var(--muted); text-transform: uppercase; margin-bottom: 6px; }  
  .stat-val { font-family: var(--mono); font-size: 26px; font-weight: 700; }  
  .stat-sub { font-size: 11px; color: var(--muted); margin-top: 2px; }  
  .val-green { color: var(--green); }  
  .val-red { color: var(--red); }  
  .val-orange { color: var(--orange); }  
  .val-blue { color: var(--blue); }  
  .val-cyan { color: var(--cyan); }  
  
  /* TWO COLS */  
  .two-col { display: grid; grid-template-columns: 1fr 1fr; gap: 14px; margin-bottom: 14px; }  
  .three-col { display: grid; grid-template-columns: 1fr 1fr 1fr; gap: 14px; margin-bottom: 14px; }  
  .card {  
    background: var(--surface); border: 1px solid var(--border);  
    border-radius: 10px; padding: 16px;  
  }  
  .card-title {  
    font-size: 11px; letter-spacing: 0.08em; text-transform: uppercase;  
    color: var(--cyan); margin-bottom: 14px; font-family: var(--mono);  
  }  
  
  /* ZONE MAP */  
  .zone-map { position: relative; height: 240px; background: var(--surface2); border-radius: 8px; overflow: hidden; }  
  .zone-grid { display: grid; grid-template-columns: repeat(4,1fr); grid-template-rows: repeat(3,1fr); height: 100%; gap: 2px; padding: 2px; }  
  .zone-cell {  
    border-radius: 4px; display: flex; flex-direction: column;  
    align-items: center; justify-content: center; cursor: pointer;  
    transition: all 0.2s; position: relative; overflow: hidden;  
  }  
  .zone-cell:hover { transform: scale(1.03); z-index: 2; }  
  .zone-cell.safe { background: rgba(34,197,94,0.18); border: 1px solid rgba(34,197,94,0.35); }  
  .zone-cell.moderate { background: rgba(234,179,8,0.18); border: 1px solid rgba(234,179,8,0.35); }  
  .zone-cell.high { background: rgba(249,115,22,0.18); border: 1px solid rgba(249,115,22,0.35); }  
  .zone-cell.critical { background: rgba(239,68,68,0.22); border: 1px solid rgba(239,68,68,0.5); }  
  .zone-name { font-size: 10px; color: rgba(255,255,255,0.7); margin-bottom: 2px; }  
  .zone-db { font-family: var(--mono); font-size: 18px; font-weight: 700; }  
  .zone-cell.safe .zone-db { color: var(--green); }  
  .zone-cell.moderate .zone-db { color: #eab308; }  
  .zone-cell.high .zone-db { color: var(--orange); }  
  .zone-cell.critical .zone-db { color: var(--red); }  
  .zone-tag { font-size: 9px; color: rgba(255,255,255,0.5); }  
  
  /* BAR CHART */  
  .bar-chart { display: flex; align-items: flex-end; gap: 6px; height: 120px; padding-bottom: 20px; position: relative; }  
  .bar-wrap { flex: 1; display: flex; flex-direction: column; align-items: center; gap: 4px; height: 100%; justify-content: flex-end; }  
  .bar { width: 100%; border-radius: 4px 4px 0 0; min-height: 4px; transition: all 0.3s; }  
  .bar-label { font-size: 10px; color: var(--muted); }  
  .bar-val { font-family: var(--mono); font-size: 10px; color: var(--text); }  
  
  /* LINE CHART */  
  .line-chart-wrap { position: relative; height: 120px; }  
  svg.line-chart { width: 100%; height: 100%; }  
  
  /* ALERT LOG */  
  .alert-item {  
    display: flex; align-items: flex-start; gap: 10px;  
    padding: 10px 0; border-bottom: 1px solid var(--border);  
  }  
  .alert-item:last-child { border-bottom: none; }  
  .alert-dot { width: 8px; height: 8px; border-radius: 50%; margin-top: 4px; flex-shrink: 0; }  
  .alert-dot.critical { background: var(--red); }  
  .alert-dot.warn { background: var(--orange); }  
  .alert-dot.info { background: var(--blue); }  
  .alert-dot.ok { background: var(--green); }  
  .alert-title { font-size: 13px; color: var(--text); margin-bottom: 2px; }  
  .alert-sub { font-size: 11px; color: var(--muted); }  
  .alert-badge {  
    margin-left: auto; padding: 2px 8px; border-radius: 4px;  
    font-size: 10px; font-family: var(--mono); flex-shrink: 0;  
  }  
  .badge-red { background: rgba(239,68,68,0.15); color: var(--red); border: 1px solid rgba(239,68,68,0.3); }  
  .badge-orange { background: rgba(249,115,22,0.15); color: var(--orange); border: 1px solid rgba(249,115,22,0.3); }  
  .badge-blue { background: rgba(59,130,246,0.15); color: var(--blue); border: 1px solid rgba(59,130,246,0.3); }  
  .badge-green { background: rgba(34,197,94,0.15); color: var(--green); border: 1px solid rgba(34,197,94,0.3); }  
  
  /* PIPE STATUS style for sensor list */  
  .sensor-row {  
    display: flex; align-items: center; justify-content: space-between;  
    padding: 10px 12px; border-radius: 6px; margin-bottom: 6px;  
    background: var(--surface2);  
  }  
  .sensor-left { display: flex; align-items: center; gap: 10px; }  
  .sensor-icon { font-size: 16px; }  
  .sensor-name { font-size: 13px; color: var(--text); }  
  .sensor-loc { font-size: 11px; color: var(--muted); }  
  .sensor-val { font-family: var(--mono); font-size: 14px; font-weight: 700; }  
  .sensor-status {  
    padding: 2px 8px; border-radius: 4px; font-size: 10px; font-family: var(--mono);  
  }  
  .status-normal { background: rgba(34,197,94,0.15); color: var(--green); }  
  .status-warn { background: rgba(249,115,22,0.15); color: var(--orange); }  
  .status-critical { background: rgba(239,68,68,0.15); color: var(--red); }  
  
  /* SENSITIVE ZONES */  
  .sz-item {  
    display: flex; align-items: center; gap: 12px; padding: 10px;  
    border-radius: 6px; background: var(--surface2); margin-bottom: 6px;  
  }  
  .sz-icon { font-size: 20px; width: 36px; text-align: center; }  
  .sz-name { font-size: 13px; color: var(--text); }  
  .sz-type { font-size: 11px; color: var(--muted); }  
  .sz-db { font-family: var(--mono); font-size: 16px; font-weight: 700; margin-left: auto; }  
  .sz-limit { font-size: 10px; color: var(--muted); text-align: right; }  
  
  /* PROGRESS BAR */  
  .progress-bar-wrap { margin-bottom: 12px; }  
  .progress-label { display: flex; justify-content: space-between; font-size: 12px; color: var(--muted); margin-bottom: 5px; }  
  .progress-track { height: 6px; background: var(--surface2); border-radius: 3px; overflow: hidden; }  
  .progress-fill { height: 100%; border-radius: 3px; }  
  
  /* FORM STYLES */  
  .form-row { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; margin-bottom: 10px; }  
  .form-group { display: flex; flex-direction: column; gap: 4px; }  
  label { font-size: 11px; text-transform: uppercase; letter-spacing: 0.06em; color: var(--muted); }  
  input, select, textarea {  
    background: var(--surface2); border: 1px solid var(--border); color: var(--text);  
    padding: 8px 10px; border-radius: 6px; font-size: 13px; font-family: var(--sans);  
    outline: none;  
  }  
  input:focus, select:focus { border-color: var(--cyan); }  
  .btn {  
    padding: 8px 18px; border-radius: 6px; font-size: 13px; cursor: pointer;  
    border: none; font-family: var(--sans); font-weight: 600;  
  }  
  .btn-primary { background: var(--cyan); color: #0a0e17; }  
  .btn-secondary { background: var(--surface2); color: var(--text); border: 1px solid var(--border); }  
  
  /* CONSERVATION */  
  .cons-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 8px; }  
  .cons-item {  
    background: var(--surface2); border-radius: 8px; padding: 12px;  
    border-left: 3px solid var(--cyan);  
  }  
  .cons-num { font-size: 10px; color: var(--cyan); font-family: var(--mono); margin-bottom: 4px; }  
  .cons-title { font-size: 13px; color: var(--text); font-weight: 500; margin-bottom: 4px; }  
  .cons-desc { font-size: 11px; color: var(--muted); line-height: 1.5; }  
  
  /* SLIDER */  
  .slider-wrap { background: var(--surface2); padding: 16px; border-radius: 8px; margin-top: 12px; }  
  .slider-label { display: flex; justify-content: space-between; font-size: 12px; color: var(--muted); margin-bottom: 8px; }  
  input[type=range] { width: 100%; accent-color: var(--cyan); }  
  .savings-row { display: grid; grid-template-columns: 1fr 1fr 1fr; gap: 8px; margin-top: 12px; }  
  .savings-box { background: var(--surface); border-radius: 6px; padding: 10px; text-align: center; }  
  .savings-val { font-family: var(--mono); font-size: 18px; font-weight: 700; color: var(--cyan); }  
  .savings-lbl { font-size: 10px; color: var(--muted); margin-top: 2px; }  
  .total-save { background: var(--surface); border-radius: 6px; padding: 10px 14px; margin-top: 8px; font-family: var(--mono); font-size: 13px; color: var(--green); text-align: center; }  
</style>  
</head>  
<body>  
  
<nav>  
  <div class="logo">  
    <div class="logo-icon">🔊</div>  
    <div class="logo-text">Noise<span>Guard</span></div>  
  </div>  
  <div class="nav-tabs">  
    <button class="nav-tab active" onclick="showTab('dashboard',this)">Dashboard</button>  
    <button class="nav-tab" onclick="showTab('sensors',this)">Sensors</button>  
    <button class="nav-tab" onclick="showTab('zones',this)">Sensitive Zones</button>  
    <button class="nav-tab" onclick="showTab('reports',this)">Reports</button>  
    <button class="nav-tab" onclick="showTab('mitigation',this)">Mitigation</button>  
  </div>  
  <div class="live-badge">  
    <div class="live-dot"></div>  
    LIVE · <span id="clock">00:00:00</span>  
  </div>  
</nav>  
  
<!-- DASHBOARD -->  
<div id="tab-dashboard" class="content active">  
  <div class="stat-row">  
    <div class="stat-card">  
      <div class="stat-label">Avg Noise Level</div>  
      <div class="stat-val val-orange">67.4 <span style="font-size:14px">dB</span></div>  
      <div class="stat-sub">City-wide average</div>  
    </div>  
    <div class="stat-card">  
      <div class="stat-label">Critical Zones</div>  
      <div class="stat-val val-red">4</div>  
      <div class="stat-sub">Exceeding WHO limit</div>  
    </div>  
    <div class="stat-card">  
      <div class="stat-label">Active Alerts</div>  
      <div class="stat-val val-orange">7</div>  
      <div class="stat-sub">Require attention</div>  
    </div>  
    <div class="stat-card">  
      <div class="stat-label">Sensors Online</div>  
      <div class="stat-val val-green">22/26</div>  
      <div class="stat-sub">4 offline</div>  
    </div>  
    <div class="stat-card">  
      <div class="stat-label">Violations Today</div>  
      <div class="stat-val val-red">12</div>  
      <div class="stat-sub">Sensitive zone breaches</div>  
    </div>  
  </div>  
  
  <div class="two-col">  
    <div class="card">  
      <div class="card-title">Zone Noise Map (dB)</div>  
      <div class="zone-map">  
        <div class="zone-grid">  
          <div class="zone-cell safe"><div class="zone-name">Park A</div><div class="zone-db">48</div><div class="zone-tag">SAFE</div></div>  
          <div class="zone-cell moderate"><div class="zone-name">Residential N</div><div class="zone-db">59</div><div class="zone-tag">MODERATE</div></div>  
          <div class="zone-cell high"><div class="zone-name">Market B</div><div class="zone-db">74</div><div class="zone-tag">HIGH</div></div>  
          <div class="zone-cell critical"><div class="zone-name">Highway Z1</div><div class="zone-db">89</div><div class="zone-tag">CRITICAL</div></div>  
          <div class="zone-cell moderate"><div class="zone-name">School S1</div><div class="zone-db">63</div><div class="zone-tag">⚠ SCHOOL</div></div>  
          <div class="zone-cell high"><div class="zone-name">Industrial E</div><div class="zone-db">78</div><div class="zone-tag">HIGH</div></div>  
          <div class="zone-cell critical"><div class="zone-name">Junction C4</div><div class="zone-db">91</div><div class="zone-tag">CRITICAL</div></div>  
          <div class="zone-cell moderate"><div class="zone-name">Residential S</div><div class="zone-db">57</div><div class="zone-tag">MODERATE</div></div>  
          <div class="zone-cell critical"><div class="zone-name">Hospital H1</div><div class="zone-db">72</div><div class="zone-tag">⚠ HOSPITAL</div></div>  
          <div class="zone-cell safe"><div class="zone-name">Garden W</div><div class="zone-db">44</div><div class="zone-tag">SAFE</div></div>  
          <div class="zone-cell high"><div class="zone-name">Rail Yard R</div><div class="zone-db">82</div><div class="zone-tag">HIGH</div></div>  
          <div class="zone-cell critical"><div class="zone-name">Flyover F2</div><div class="zone-db">94</div><div class="zone-tag">CRITICAL</div></div>  
        </div>  
      </div>  
    </div>  
    <div class="card">  
      <div class="card-title">Recent Alerts</div>  
      <div class="alert-item">  
        <div class="alert-dot critical"></div>  
        <div>  
          <div class="alert-title">Hospital H1 — Noise Breach</div>  
          <div class="alert-sub">09:14 · Level exceeded 70 dB (limit: 45 dB)</div>  
        </div>  
        <span class="alert-badge badge-red">CRITICAL</span>  
      </div>  
      <div class="alert-item">  
        <div class="alert-dot warn"></div>  
        <div>  
          <div class="alert-title">School S1 — Morning Traffic</div>  
          <div class="alert-sub">08:52 · Noise at 63 dB during school hours</div>  
        </div>  
        <span class="alert-badge badge-orange">WARN</span>  
      </div>  
      <div class="alert-item">  
        <div class="alert-dot critical"></div>  
        <div>  
          <div class="alert-title">Junction C4 — Peak Noise</div>  
          <div class="alert-sub">08:30 · 91 dB sustained over 15 min</div>  
        </div>  
        <span class="alert-badge badge-red">CRITICAL</span>  
      </div>  
      <div class="alert-item">  
        <div class="alert-dot info"></div>  
        <div>  
          <div class="alert-title">Sensor S-09 — Offline</div>  
          <div class="alert-sub">07:45 · Connection lost at North zone</div>  
        </div>  
        <span class="alert-badge badge-blue">INFO</span>  
      </div>  
      <div class="alert-item">  
        <div class="alert-dot ok"></div>  
        <div>  
          <div class="alert-title">Park A — Baseline Normal</div>  
          <div class="alert-sub">06:00 · Daily calibration complete</div>  
        </div>  
        <span class="alert-badge badge-green">OK</span>  
      </div>  
    </div>  
  </div>  
  
  <div class="two-col">  
    <div class="card">  
      <div class="card-title">Hourly Avg Noise Today (dB)</div>  
      <div class="bar-chart" id="hourly-chart"></div>  
    </div>  
    <div class="card">  
      <div class="card-title">Zone Noise Comparison (dB)</div>  
      <div class="bar-chart" id="zone-chart"></div>  
    </div>  
  </div>  
</div>  
  
<!-- SENSORS -->  
<div id="tab-sensors" class="content">  
  <div class="card" style="margin-bottom:14px">  
    <div class="card-title">Live Sensor Readings</div>  
    <div id="sensor-grid" style="display:grid;grid-template-columns:repeat(3,1fr);gap:8px;"></div>  
  </div>  
  <div class="card">  
    <div class="card-title">Add / Edit Sensor</div>  
    <div class="form-row">  
      <div class="form-group"><label>Sensor ID</label><input placeholder="e.g. S-27"></div>  
      <div class="form-group"><label>Zone</label>  
        <select><option>North Residential</option><option>South Industrial</option><option>East Commercial</option><option>Hospital Zone</option><option>School Zone</option></select>  
      </div>  
    </div>  
    <div class="form-row">  
      <div class="form-group"><label>Type</label>  
        <select><option>Acoustic (dB)</option><option>Traffic Counter</option><option>Vibration</option></select>  
      </div>  
      <div class="form-group"><label>Install Date</label><input type="date"></div>  
    </div>  
    <div style="display:flex;gap:8px;margin-top:4px">  
      <button class="btn btn-primary">Add Sensor</button>  
      <button class="btn btn-secondary">Cancel</button>  
    </div>  
  </div>  
</div>  
  
<!-- SENSITIVE ZONES -->  
<div id="tab-zones" class="content">  
  <div class="two-col">  
    <div class="card">  
      <div class="card-title">Sensitive Zone Monitor</div>  
      <div class="sz-item">  
        <div class="sz-icon">🏥</div>  
        <div><div class="sz-name">City Hospital H1</div><div class="sz-type">Hospital · North Zone</div></div>  
        <div style="text-align:right"><div class="sz-db val-red">72 dB</div><div class="sz-limit">Limit: 45 dB</div></div>  
      </div>  
      <div class="sz-item">  
        <div class="sz-icon">🏫</div>  
        <div><div class="sz-name">St. Mary's School</div><div class="sz-type">School · East Zone</div></div>  
        <div style="text-align:right"><div class="sz-db val-orange">63 dB</div><div class="sz-limit">Limit: 50 dB</div></div>  
      </div>  
      <div class="sz-item">  
        <div class="sz-icon">🏛️</div>  
        <div><div class="sz-name">District Court</div><div class="sz-type">Silence Zone · Central</div></div>  
        <div style="text-align:right"><div class="sz-db val-orange">61 dB</div><div class="sz-limit">Limit: 40 dB</div></div>  
      </div>  
      <div class="sz-item">  
        <div class="sz-icon">📚</div>  
        <div><div class="sz-name">City Library</div><div class="sz-type">Silence Zone · West</div></div>  
        <div style="text-align:right"><div class="sz-db" style="color:#eab308">54 dB</div><div class="sz-limit">Limit: 50 dB</div></div>  
      </div>  
      <div class="sz-item">  
        <div class="sz-icon">⛪</div>  
        <div><div class="sz-name">Central Temple</div><div class="sz-type">Residential · South</div></div>  
        <div style="text-align:right"><div class="sz-db val-green">46 dB</div><div class="sz-limit">Limit: 55 dB</div></div>  
      </div>  
    </div>  
    <div class="card">  
      <div class="card-title">WHO Compliance Status</div>  
      <div class="progress-bar-wrap">  
        <div class="progress-label"><span>Hospital Zone</span><span style="color:var(--red)">72/45 dB — CRITICAL</span></div>  
        <div class="progress-track"><div class="progress-fill" style="width:100%;background:var(--red)"></div></div>  
      </div>  
      <div class="progress-bar-wrap">  
        <div class="progress-label"><span>School Zone</span><span style="color:var(--orange)">63/50 dB — HIGH</span></div>  
        <div class="progress-track"><div class="progress-fill" style="width:85%;background:var(--orange)"></div></div>  
      </div>  
      <div class="progress-bar-wrap">  
        <div class="progress-label"><span>Silence Zone</span><span style="color:var(--orange)">61/40 dB — WARN</span></div>  
        <div class="progress-track"><div class="progress-fill" style="width:75%;background:var(--orange)"></div></div>  
      </div>  
      <div class="progress-bar-wrap">  
        <div class="progress-label"><span>Residential Night</span><span style="color:#eab308">54/55 dB — OK</span></div>  
        <div class="progress-track"><div class="progress-fill" style="width:55%;background:#eab308"></div></div>  
      </div>  
      <div class="progress-bar-wrap">  
        <div class="progress-label"><span>Commercial Zone</span><span style="color:var(--green)">46/65 dB — SAFE</span></div>  
        <div class="progress-track"><div class="progress-fill" style="width:40%;background:var(--green)"></div></div>  
      </div>  
      <div style="margin-top:14px;padding:12px;background:var(--surface2);border-radius:8px;">  
        <div style="font-size:11px;color:var(--muted);margin-bottom:6px;">Log Noise Complaint</div>  
        <div class="form-row">  
          <div class="form-group"><label>Location</label><input placeholder="Zone/area"></div>  
          <div class="form-group"><label>Severity</label><select><option>Minor</option><option>Moderate</option><option>Severe</option></select></div>  
        </div>  
        <button class="btn btn-primary" style="margin-top:4px">Submit Report</button>  
      </div>  
    </div>  
  </div>  
</div>  
  
<!-- REPORTS -->  
<div id="tab-reports" class="content">  
  <div class="stat-row">  
    <div class="stat-card"><div class="stat-label">Total Readings</div><div class="stat-val val-cyan">48,320</div><div class="stat-sub">This week</div></div>  
    <div class="stat-card"><div class="stat-label">Violations</div><div class="stat-val val-red">84</div><div class="stat-sub">Sensitive zones</div></div>  
    <div class="stat-card"><div class="stat-label">Avg Reduction</div><div class="stat-val val-green">6.2 dB</div><div class="stat-sub">vs last month</div></div>  
    <div class="stat-card"><div class="stat-label">Peak Hour</div><div class="stat-val val-orange">8–9 AM</div><div class="stat-sub">Highest noise window</div></div>  
    <div class="stat-card"><div class="stat-label">Quietest Zone</div><div class="stat-val val-green">Park A</div><div class="stat-sub">44 dB avg</div></div>  
  </div>  
  <div class="two-col">  
    <div class="card">  
      <div class="card-title">Daily Noise Levels — This Week (dB Avg)</div>  
      <div class="bar-chart" id="weekly-chart"></div>  
    </div>  
    <div class="card">  
      <div class="card-title">Noise Source Breakdown</div>  
      <div style="display:flex;flex-direction:column;gap:8px;margin-top:4px">  
        <div class="progress-bar-wrap">  
          <div class="progress-label"><span>🚗 Road Traffic</span><span>52%</span></div>  
          <div class="progress-track"><div class="progress-fill" style="width:52%;background:var(--red)"></div></div>  
        </div>  
        <div class="progress-bar-wrap">  
          <div class="progress-label"><span>🏭 Industrial</span><span>21%</span></div>  
          <div class="progress-track"><div class="progress-fill" style="width:21%;background:var(--orange)"></div></div>  
        </div>  
        <div class="progress-bar-wrap">  
          <div class="progress-label"><span>🚂 Rail/Transit</span><span>14%</span></div>  
          <div class="progress-track"><div class="progress-fill" style="width:14%;background:#eab308"></div></div>  
        </div>  
        <div class="progress-bar-wrap">  
          <div class="progress-label"><span>🔨 Construction</span><span>9%</span></div>  
          <div class="progress-track"><div class="progress-fill" style="width:9%;background:var(--blue)"></div></div>  
        </div>  
        <div class="progress-bar-wrap">  
          <div class="progress-label"><span>🎉 Events/Other</span><span>4%</span></div>  
          <div class="progress-track"><div class="progress-fill" style="width:4%;background:var(--cyan)"></div></div>  
        </div>  
      </div>  
    </div>  
  </div>  
  <div class="card">  
    <div class="card-title">Generate Custom Report</div>  
    <div class="form-row">  
      <div class="form-group"><label>From Date</label><input type="date"></div>  
      <div class="form-group"><label>To Date</label><input type="date"></div>  
      <div class="form-group"><label>Zone Filter</label><select><option>All Zones</option><option>Sensitive Zones</option><option>Hospital</option><option>School</option></select></div>  
      <div class="form-group"><label>Format</label><select><option>PDF Report</option><option>CSV Export</option><option>Dashboard View</option></select></div>  
    </div>  
    <button class="btn btn-primary">Generate Report</button>  
  </div>  
</div>  
  
<!-- MITIGATION -->  
<div id="tab-mitigation" class="content">  
  <div class="card" style="margin-bottom:14px">  
    <div class="card-title">Noise Mitigation Strategies</div>  
    <div class="cons-grid">  
      <div class="cons-item">  
        <div class="cons-num">01</div>  
        <div class="cons-title">Night-Time Traffic Control</div>  
        <div class="cons-desc">Restrict heavy vehicles near hospitals and schools between 10 PM–6 AM. Any sustained noise above 55 dB triggers automated alerts.</div>  
      </div>  
      <div class="cons-item">  
        <div class="cons-num">02</div>  
        <div class="cons-title">Acoustic Barrier Monitoring</div>  
        <div class="cons-desc">Track effectiveness of noise barriers. Sensors before and after barriers measure noise reduction in real-time.</div>  
      </div>  
      <div class="cons-item">  
        <div class="cons-num">03</div>  
        <div class="cons-title">Zone Isolation Testing</div>  
        <div class="cons-desc">Identify dominant noise corridors by temporarily redirecting traffic and monitoring dB drops per zone.</div>  
      </div>  
      <div class="cons-item">  
        <div class="cons-num">04</div>  
        <div class="cons-title">Construction Hour Enforcement</div>  
        <div class="cons-desc">Flag construction activity exceeding 70 dB in residential zones between 7–8 PM or on Sundays using acoustic event detection.</div>  
      </div>  
      <div class="cons-item">  
        <div class="cons-num">05</div>  
        <div class="cons-title">Smart Traffic Signal Sync</div>  
        <div class="cons-desc">Coordinate traffic lights near sensitive zones to reduce engine idling and horn usage at peak congestion windows.</div>  
      </div>  
      <div class="cons-item">  
        <div class="cons-num">06</div>  
        <div class="cons-title">Green Buffer Mapping</div>  
        <div class="cons-desc">Identify zones where tree-belt or green wall planting can reduce ambient noise by 3–8 dB based on sensor data.</div>  
      </div>  
      <div class="cons-item">  
        <div class="cons-num">07</div>  
        <div class="cons-title">Industrial Shift Scheduling</div>  
        <div class="cons-desc">Track noise spikes from factories by shift and recommend off-peak scheduling for high-noise processes near residential zones.</div>  
      </div>  
      <div class="cons-item">  
        <div class="cons-num">08</div>  
        <div class="cons-title">Seasonal Pattern Analysis</div>  
        <div class="cons-desc">Monitor seasonal noise changes (monsoon traffic, festival spikes) and update zone limits proactively with historical ML models.</div>  
      </div>  
    </div>  
  </div>  
  <div class="card">  
    <div class="card-title">Estimate Noise Reduction Potential</div>  
    <div class="slider-wrap">  
      <div class="slider-label"><span>Current Peak dB in Zone</span><span id="dbl-val" style="color:var(--cyan);font-family:var(--mono)">80 dB</span></div>  
      <input type="range" min="50" max="100" value="80" id="dbl-slider" oninput="updateSavings(this.value)">  
      <div class="savings-row">  
        <div class="savings-box"><div class="savings-val" id="s1">4 dB</div><div class="savings-lbl">Traffic Control</div></div>  
        <div class="savings-box"><div class="savings-val" id="s2">6 dB</div><div class="savings-lbl">Acoustic Barriers</div></div>  
        <div class="savings-box"><div class="savings-val" id="s3">3 dB</div><div class="savings-lbl">Green Buffers</div></div>  
      </div>  
      <div class="total-save" id="total-save">Total potential reduction: 13 dB (−16.3%) → Expected: 67 dB</div>  
    </div>  
  </div>  
</div>  
  
<script>  
// Clock  
function updateClock() {  
  const now = new Date();  
  document.getElementById('clock').textContent =  
    now.toTimeString().slice(0,8);  
}  
setInterval(updateClock, 1000); updateClock();  
  
// Tab switch  
function showTab(id, btn) {  
  document.querySelectorAll('.content').forEach(c => c.classList.remove('active'));  
  document.querySelectorAll('.nav-tab').forEach(b => b.classList.remove('active'));  
  document.getElementById('tab-' + id).classList.add('active');  
  btn.classList.add('active');  
}  
  
// Hourly chart  
const hourlyData = [32,29,38,45,62,74,82,78,71,68,65,70,74,72,68,65,67,71,76,78,72,65,55,44];  
const hourlyChart = document.getElementById('hourly-chart');  
hourlyData.forEach((v, i) => {  
  if(i % 3 !== 0) return;  
  const wrap = document.createElement('div'); wrap.className = 'bar-wrap';  
  const bar = document.createElement('div'); bar.className = 'bar';  
  bar.style.height = (v/100*100) + '%';  
  bar.style.background = v > 75 ? 'var(--red)' : v > 60 ? 'var(--orange)' : 'var(--green)';  
  const lbl = document.createElement('div'); lbl.className = 'bar-label'; lbl.textContent = i + 'h';  
  const val = document.createElement('div'); val.className = 'bar-val'; val.textContent = v;  
  wrap.append(bar, val, lbl); hourlyChart.appendChild(wrap);  
});  
  
// Zone chart  
const zoneData = [{n:'Park A',v:48},{n:'Res N',v:59},{n:'Market',v:74},{n:'Hwy Z1',v:89},{n:'School',v:63},{n:'Ind E',v:78},{n:'Junc C4',v:91},{n:'Hosp H1',v:72}];  
const zoneChart = document.getElementById('zone-chart');  
zoneData.forEach(z => {  
  const wrap = document.createElement('div'); wrap.className = 'bar-wrap';  
  const bar = document.createElement('div'); bar.className = 'bar';  
  bar.style.height = (z.v/100*100) + '%';  
  bar.style.background = z.v > 75 ? 'var(--red)' : z.v > 60 ? 'var(--orange)' : 'var(--green)';  
  const lbl = document.createElement('div'); lbl.className = 'bar-label'; lbl.style.fontSize = '9px'; lbl.textContent = z.n;  
  const val = document.createElement('div'); val.className = 'bar-val'; val.textContent = z.v;  
  wrap.append(bar, val, lbl); zoneChart.appendChild(wrap);  
});  
  
// Weekly chart  
const weeklyData = [{n:'Mon',v:68},{n:'Tue',v:71},{n:'Wed',v:69},{n:'Thu',v:73},{n:'Fri',v:76},{n:'Sat',v:62},{n:'Sun',v:55}];  
const weeklyChart = document.getElementById('weekly-chart');  
weeklyData.forEach(d => {  
  const wrap = document.createElement('div'); wrap.className = 'bar-wrap';  
  const bar = document.createElement('div'); bar.className = 'bar';  
  bar.style.height = (d.v/100*100) + '%';  
  bar.style.background = d.v > 72 ? 'var(--orange)' : 'var(--blue)';  
  const lbl = document.createElement('div'); lbl.className = 'bar-label'; lbl.textContent = d.n;  
  const val = document.createElement('div'); val.className = 'bar-val'; val.textContent = d.v;  
  wrap.append(bar, val, lbl); weeklyChart.appendChild(wrap);  
});  
  
// Sensor grid  
const sensors = [  
  {id:'S-01',zone:'North',type:'🔊',val:'99 L/min',status:'CRITICAL',sc:'critical'},  
  {id:'S-02',zone:'North',type:'📡',val:'4.7 bar',status:'NORMAL',sc:'normal'},  
  {id:'S-03',zone:'East',type:'🔊',val:'54 dB',status:'NORMAL',sc:'normal'},  
  {id:'S-04',zone:'East',type:'📡',val:'2.9 bar',status:'WARN',sc:'warn'},  
  {id:'S-05',zone:'South',type:'🔊',val:'129 dB',status:'CRITICAL',sc:'critical'},  
  {id:'S-06',zone:'South',type:'📡',val:'4.4 bar',status:'NORMAL',sc:'normal'},  
  {id:'S-07',zone:'West',type:'🔊',val:'61 dB',status:'NORMAL',sc:'normal'},  
  {id:'S-08',zone:'West',type:'🌡️',val:'22°C',status:'NORMAL',sc:'normal'},  
  {id:'S-09',zone:'North',type:'⚠️',val:'OFFLINE',status:'OFFLINE',sc:'warn'},  
];  
const sgrid = document.getElementById('sensor-grid');  
sensors.forEach(s => {  
  const card = document.createElement('div');  
  card.style.cssText = 'background:var(--surface2);border-radius:8px;padding:12px;';  
  card.innerHTML = `  
    <div style="display:flex;justify-content:space-between;align-items:flex-start">  
      <div style="font-size:10px;color:var(--muted);font-family:var(--mono)">${s.id} · ${s.zone}</div>  
      <span class="sensor-status status-${s.sc}" style="font-size:9px">${s.status}</span>  
    </div>  
    <div style="font-family:var(--mono);font-size:22px;font-weight:700;margin:8px 0;color:${s.sc==='critical'?'var(--red)':s.sc==='warn'?'var(--orange)':'var(--cyan)'}">${s.val}</div>  
    <div style="font-size:11px;color:var(--muted)">${s.type} ${s.type.includes('🔊')?'Acoustic Sensor':'Pressure Sensor'}</div>  
  `;  
  sgrid.appendChild(card);  
});  
  
// Savings slider  
function updateSavings(v) {  
  v = parseInt(v);  
  document.getElementById('dbl-val').textContent = v + ' dB';  
  const t = Math.round(v * 0.05), b = Math.round(v * 0.075), g = Math.round(v * 0.038);  
  document.getElementById('s1').textContent = t + ' dB';  
  document.getElementById('s2').textContent = b + ' dB';  
  document.getElementById('s3').textContent = g + ' dB';  
  const total = t + b + g;  
  const pct = (total / v * 100).toFixed(1);  
  const after = v - total;  
  document.getElementById('total-save').textContent = `Total potential reduction: ${total} dB (−${pct}%) → Expected: ${after} dB`;  
}  
</script>  
</body>  
</html>  
