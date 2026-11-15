<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>PRISM Live Broadcast Scoreboard</title>
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&family=Rajdhani:wght@600;700;800&display=swap" rel="stylesheet">
  <!-- PeerJS for 2-device PRISM Live Remote Sync -->
  <script src="https://cdn.jsdelivr.net/npm/peerjs@1.5.5/dist/peerjs.min.js"></script>

  <style>
    :root {
      --bg-dark: #090d16;
      --panel-bg: #111827;
      --panel-border: rgba(255, 255, 255, 0.08);
      --text-main: #f8fafc;
      --text-muted: #94a3b8;
      --team-a-color: #00d2ff;
      --team-b-color: #ff2a55;
      --yellow-card: #facc15;
      --red-card: #ef4444;
      --accent-green: #10b981;
    }

    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
    }

    html, body {
      min-height: 100vh;
    }

    body {
      font-family: 'Inter', -apple-system, BlinkMacSystemFont, sans-serif;
      background: radial-gradient(circle at top, #162036 0%, var(--bg-dark) 65%);
      color: var(--text-main);
      display: flex;
      flex-direction: column;
      align-items: center;
      padding: 20px 14px 50px;
    }

    /* Transparent Mode for PRISM Live Studio Overlay */
    body.transparent-bg,
    html.transparent-bg {
      background: transparent !important;
    }

    body.green-bg,
    html.green-bg {
      background: #00b140 !important;
    }

    body.overlay-mode {
      padding: 12px;
      justify-content: flex-start;
    }

    body.overlay-mode .top-bar,
    body.overlay-mode .remote-bar,
    body.overlay-mode .control-deck {
      display: none !important;
    }

    /* ================= TOP HEADER ================= */
    .top-bar {
      width: 100%;
      max-width: 1050px;
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin-bottom: 14px;
      gap: 10px;
      flex-wrap: wrap;
    }

    .app-title {
      display: flex;
      align-items: center;
      gap: 10px;
      font-family: 'Rajdhani', sans-serif;
      font-size: 1.3rem;
      font-weight: 700;
      letter-spacing: 1px;
      text-transform: uppercase;
      color: #e2e8f0;
    }

    .live-dot {
      width: 10px;
      height: 10px;
      background: #ef4444;
      border-radius: 50%;
      box-shadow: 0 0 10px #ef4444;
      animation: pulse 1.4s infinite;
    }

    @keyframes pulse {
      0%, 100% { opacity: 1; transform: scale(1); }
      50% { opacity: 0.45; transform: scale(0.85); }
    }

    .header-actions {
      display: flex;
      gap: 8px;
      flex-wrap: wrap;
    }

    .header-btn {
      background: rgba(255, 255, 255, 0.08);
      color: var(--text-main);
      border: 1px solid rgba(255, 255, 255, 0.15);
      padding: 8px 14px;
      border-radius: 8px;
      font-size: 0.82rem;
      font-weight: 600;
      cursor: pointer;
      transition: all 0.2s ease;
    }

    .header-btn:hover { background: rgba(255, 255, 255, 0.16); }
    .header-btn.active {
      background: #10b981;
      color: #06281e;
      border-color: #10b981;
    }

    /* ================= PRISM REMOTE SYNC BAR ================= */
    .remote-bar {
      width: 100%;
      max-width: 1050px;
      background: rgba(15, 23, 42, 0.9);
      border: 1px solid rgba(56, 189, 248, 0.28);
      border-radius: 12px;
      padding: 12px 16px;
      margin-bottom: 20px;
      display: flex;
      align-items: center;
      justify-content: space-between;
      gap: 12px;
      flex-wrap: wrap;
    }

    .remote-info {
      display: flex;
      align-items: center;
      gap: 12px;
      flex-wrap: wrap;
    }

    .sync-badge {
      display: inline-flex;
      align-items: center;
      gap: 6px;
      font-size: 0.8rem;
      font-weight: 700;
      padding: 4px 10px;
      border-radius: 99px;
      background: rgba(250, 204, 21, 0.15);
      color: #fde047;
      border: 1px solid rgba(250, 204, 21, 0.3);
    }

    .sync-badge.connected {
      background: rgba(16, 185, 129, 0.15);
      color: #34d399;
      border-color: rgba(16, 185, 129, 0.35);
    }

    .room-box {
      display: flex;
      align-items: center;
      gap: 8px;
      font-size: 0.85rem;
      color: var(--text-muted);
    }

    .room-box input {
      width: 85px;
      text-align: center;
      font-family: 'Rajdhani', sans-serif;
      font-weight: 700;
      font-size: 1rem;
      letter-spacing: 1px;
      padding: 5px 8px;
    }

    .remote-actions {
      display: flex;
      align-items: center;
      gap: 8px;
      flex-wrap: wrap;
    }

    .copy-url-btn {
      background: linear-gradient(135deg, #0284c7 0%, #2563eb 100%);
      color: #fff;
      border: none;
      padding: 8px 14px;
      border-radius: 8px;
      font-size: 0.82rem;
      font-weight: 700;
      cursor: pointer;
      transition: filter 0.2s;
    }

    .copy-url-btn:hover { filter: brightness(1.12); }

    /* ================= TV BROADCAST OVERLAY (SCOREBUG) ================= */
    .broadcast-wrapper {
      width: 100%;
      max-width: 1050px;
      margin-bottom: 24px;
    }

    body.overlay-mode .broadcast-wrapper {
      margin-bottom: 0;
    }

    .scoreboard-overlay {
      background: linear-gradient(180deg, rgba(20, 29, 48, 0.95) 0%, rgba(10, 15, 26, 0.97) 100%);
      border: 1px solid rgba(255, 255, 255, 0.16);
      border-radius: 14px;
      box-shadow: 0 12px 35px rgba(0, 0, 0, 0.6), inset 0 1px 0 rgba(255, 255, 255, 0.14);
      overflow: hidden;
      display: grid;
      grid-template-columns: 1fr auto 1fr;
      align-items: stretch;
      position: relative;
    }

    .overlay-team {
      display: flex;
      align-items: center;
      padding: 14px 18px;
      gap: 14px;
      position: relative;
      min-width: 0;
    }

    .overlay-team.team-a {
      border-left: 7px solid var(--team-a-color);
      background: linear-gradient(90deg, rgba(0, 210, 255, 0.14) 0%, transparent 70%);
      justify-content: flex-start;
    }

    .overlay-team.team-b {
      border-right: 7px solid var(--team-b-color);
      background: linear-gradient(-90deg, rgba(255, 42, 85, 0.14) 0%, transparent 70%);
      flex-direction: row-reverse;
      justify-content: flex-start;
      text-align: right;
    }

    .team-logo-box {
      width: 54px;
      height: 54px;
      border-radius: 10px;
      background: rgba(255, 255, 255, 0.08);
      border: 1px solid rgba(255, 255, 255, 0.16);
      display: flex;
      align-items: center;
      justify-content: center;
      flex-shrink: 0;
      overflow: hidden;
      font-family: 'Rajdhani', sans-serif;
      font-weight: 800;
      font-size: 1.15rem;
      letter-spacing: 1px;
      color: #ffffff;
    }

    .team-logo-box img {
      width: 100%;
      height: 100%;
      object-fit: cover;
    }

    .team-info {
      display: flex;
      flex-direction: column;
      min-width: 0;
      flex: 1;
    }

    .team-short-tag {
      font-family: 'Rajdhani', sans-serif;
      font-size: 0.82rem;
      font-weight: 700;
      letter-spacing: 2px;
      color: var(--text-muted);
      text-transform: uppercase;
    }

    .team-full-name {
      font-family: 'Rajdhani', sans-serif;
      font-size: 1.75rem;
      font-weight: 800;
      letter-spacing: 0.5px;
      text-transform: uppercase;
      white-space: nowrap;
      overflow: hidden;
      text-overflow: ellipsis;
      line-height: 1.1;
    }

    .overlay-cards {
      display: flex;
      gap: 6px;
      margin-top: 5px;
    }

    .team-b .overlay-cards {
      justify-content: flex-end;
    }

    .card-badge {
      display: inline-flex;
      align-items: center;
      gap: 4px;
      font-size: 0.75rem;
      font-weight: 700;
      background: rgba(0, 0, 0, 0.4);
      padding: 2px 7px;
      border-radius: 5px;
      border: 1px solid rgba(255, 255, 255, 0.09);
    }

    .card-icon {
      width: 8px;
      height: 12px;
      border-radius: 2px;
      display: inline-block;
    }

    .card-icon.yellow { background: var(--yellow-card); }
    .card-icon.red { background: var(--red-card); }

    /* Score & Center Clock */
    .overlay-center {
      display: flex;
      align-items: stretch;
      background: rgba(5, 8, 15, 0.8);
      border-left: 1px solid rgba(255, 255, 255, 0.08);
      border-right: 1px solid rgba(255, 255, 255, 0.08);
    }

    .score-box {
      font-family: 'Rajdhani', sans-serif;
      font-size: 3.3rem;
      font-weight: 800;
      min-width: 72px;
      display: flex;
      align-items: center;
      justify-content: center;
      padding: 0 12px;
      color: #ffffff;
    }

    .clock-box {
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      padding: 8px 16px;
      background: linear-gradient(180deg, #1e293b 0%, #0f172a 100%);
      border-left: 1px solid rgba(255, 255, 255, 0.08);
      border-right: 1px solid rgba(255, 255, 255, 0.08);
      min-width: 118px;
    }

    .match-period {
      font-size: 0.68rem;
      font-weight: 700;
      letter-spacing: 1.2px;
      text-transform: uppercase;
      color: var(--accent-green);
      background: rgba(16, 185, 129, 0.15);
      padding: 2px 8px;
      border-radius: 99px;
      margin-bottom: 3px;
      white-space: nowrap;
    }

    .match-clock {
      font-family: 'Rajdhani', sans-serif;
      font-size: 2rem;
      font-weight: 800;
      letter-spacing: 1.5px;
      font-variant-numeric: tabular-nums;
      color: #ffffff;
      line-height: 1;
    }

    /* ================= CONTROL DECK ================= */
    .control-deck {
      width: 100%;
      max-width: 1050px;
      display: grid;
      grid-template-columns: 1fr 310px 1fr;
      gap: 18px;
    }

    .control-deck.hidden { display: none; }

    .panel {
      background: var(--panel-bg);
      border: 1px solid var(--panel-border);
      border-radius: 14px;
      padding: 18px;
      box-shadow: 0 10px 30px rgba(0, 0, 0, 0.35);
      display: flex;
      flex-direction: column;
      gap: 16px;
    }

    .panel-header {
      display: flex;
      align-items: center;
      justify-content: space-between;
      border-bottom: 1px solid rgba(255, 255, 255, 0.07);
      padding-bottom: 10px;
    }

    .panel-title {
      font-family: 'Rajdhani', sans-serif;
      font-size: 1.15rem;
      font-weight: 700;
      letter-spacing: 1px;
      text-transform: uppercase;
    }

    .panel-a .panel-title { color: var(--team-a-color); }
    .panel-b .panel-title { color: var(--team-b-color); }

    .score-control-group {
      display: grid;
      grid-template-columns: 1fr 1fr;
      gap: 12px;
    }

    .btn-score {
      font-family: 'Rajdhani', sans-serif;
      font-size: 2.2rem;
      font-weight: 800;
      height: 66px;
      border-radius: 12px;
      border: none;
      cursor: pointer;
      color: #fff;
      display: flex;
      align-items: center;
      justify-content: center;
      transition: transform 0.1s ease, filter 0.2s ease;
      user-select: none;
    }

    .btn-score:active { transform: scale(0.95); }
    .btn-plus {
      background: linear-gradient(135deg, #10b981 0%, #059669 100%);
      box-shadow: 0 6px 18px rgba(16, 185, 129, 0.28);
    }
    .btn-minus {
      background: linear-gradient(135deg, #334155 0%, #1e293b 100%);
      border: 1px solid rgba(255, 255, 255, 0.12);
    }

    .form-row { display: flex; gap: 10px; }

    .input-group {
      display: flex;
      flex-direction: column;
      gap: 5px;
      flex: 1;
    }

    .input-group label {
      font-size: 0.74rem;
      font-weight: 600;
      color: var(--text-muted);
      text-transform: uppercase;
    }

    input[type="text"],
    select {
      background: #0b1120;
      border: 1px solid rgba(255, 255, 255, 0.12);
      border-radius: 8px;
      padding: 9px 11px;
      color: #fff;
      font-size: 0.92rem;
      font-family: inherit;
      outline: none;
      width: 100%;
    }

    input[type="text"]:focus, select:focus { border-color: #38bdf8; }

    input[type="color"] {
      width: 42px;
      height: 36px;
      border: 1px solid rgba(255, 255, 255, 0.15);
      border-radius: 8px;
      background: #0b1120;
      cursor: pointer;
      padding: 2px;
    }

    .logo-upload-row { display: flex; align-items: center; gap: 8px; }

    .file-btn {
      flex: 1;
      background: rgba(255, 255, 255, 0.06);
      border: 1px dashed rgba(255, 255, 255, 0.22);
      color: #cbd5e1;
      padding: 8px 10px;
      border-radius: 8px;
      font-size: 0.82rem;
      text-align: center;
      cursor: pointer;
    }

    .clear-logo-btn {
      background: rgba(239, 68, 68, 0.15);
      color: #fca5a5;
      border: 1px solid rgba(239, 68, 68, 0.3);
      padding: 8px 10px;
      border-radius: 8px;
      font-size: 0.8rem;
      cursor: pointer;
    }

    .cards-control-section {
      display: grid;
      grid-template-columns: 1fr 1fr;
      gap: 10px;
      padding-top: 6px;
      border-top: 1px solid rgba(255, 255, 255, 0.07);
    }

    .card-ctrl-box {
      background: #0b1120;
      border: 1px solid rgba(255, 255, 255, 0.08);
      border-radius: 10px;
      padding: 10px;
      display: flex;
      flex-direction: column;
      align-items: center;
      gap: 8px;
    }

    .card-ctrl-label {
      display: flex;
      align-items: center;
      gap: 6px;
      font-size: 0.8rem;
      font-weight: 600;
    }

    .card-ctrl-actions { display: flex; align-items: center; gap: 10px; }

    .btn-mini {
      width: 30px;
      height: 30px;
      border-radius: 6px;
      border: none;
      background: #1e293b;
      color: #fff;
      font-size: 1.1rem;
      font-weight: 700;
      cursor: pointer;
    }

    .card-count-num {
      font-family: 'Rajdhani', sans-serif;
      font-size: 1.3rem;
      font-weight: 700;
      min-width: 20px;
      text-align: center;
    }

    .timer-display-preview {
      background: #0b1120;
      border: 1px solid rgba(255, 255, 255, 0.08);
      border-radius: 10px;
      padding: 12px;
      text-align: center;
      font-family: 'Rajdhani', sans-serif;
      font-size: 2.4rem;
      font-weight: 800;
      letter-spacing: 2px;
      font-variant-numeric: tabular-nums;
    }

    .timer-buttons {
      display: grid;
      grid-template-columns: 1fr 1fr;
      gap: 10px;
    }

    .btn-timer {
      padding: 12px;
      border-radius: 9px;
      border: none;
      font-size: 0.9rem;
      font-weight: 700;
      cursor: pointer;
      color: #fff;
      text-transform: uppercase;
    }

    .btn-start { background: #10b981; }
    .btn-pause { background: #f59e0b; color: #090d16; }
    .btn-reset { background: #ef4444; grid-column: span 2; }

    .btn-reset-all {
      margin-top: auto;
      background: rgba(255, 255, 255, 0.05);
      border: 1px solid rgba(255, 255, 255, 0.12);
      color: #cbd5e1;
      padding: 10px;
      border-radius: 8px;
      font-size: 0.85rem;
      font-weight: 600;
      cursor: pointer;
    }

    /* ================= COMPACT MOBILE / PRISM WIDGET RESPONSIVE ================= */
    @media (max-width: 880px) {
      .control-deck { grid-template-columns: 1fr; }
    }

    /* Keep Scoreboard Sleek Horizontal Even on Mobile Portrait (PRISM Mobile Web Widget) */
    @media (max-width: 720px) {
      .scoreboard-overlay {
        grid-template-columns: 1fr auto 1fr;
        border-radius: 10px;
      }
      .overlay-team {
        padding: 8px 10px;
        gap: 8px;
      }
      .overlay-team.team-a { border-left-width: 5px; }
      .overlay-team.team-b { border-right-width: 5px; }
      .team-logo-box {
        width: 34px;
        height: 34px;
        font-size: 0.8rem;
        border-radius: 7px;
      }
      .team-short-tag { display: none; }
      .team-full-name { font-size: 1rem; }
      .overlay-cards { gap: 4px; margin-top: 3px; }
      .card-badge { font-size: 0.65rem; padding: 1px 5px; }
      .card-icon { width: 6px; height: 9px; }
      .score-box { font-size: 1.9rem; min-width: 36px; padding: 0 7px; }
      .clock-box { min-width: 68px; padding: 6px 8px; }
      .match-period { font-size: 0.55rem; padding: 1px 5px; letter-spacing: 0.5px; }
      .match-clock { font-size: 1.15rem; letter-spacing: 0.5px; }
    }
  </style>
</head>
<body>

  <!-- Top Header -->
  <div class="top-bar">
    <div class="app-title">
      <span class="live-dot"></span>
      <span>PRISM Broadcast Scoreboard</span>
    </div>
    <div class="header-actions">
      <button class="header-btn" id="transparentBtn" onclick="toggleTransparent()">Transparent BG: OFF</button>
      <button class="header-btn" id="toggleControlsBtn" onclick="toggleControls()">Hide Controls</button>
    </div>
  </div>

  <!-- PRISM Live Remote Sync Bar -->
  <div class="remote-bar">
    <div class="remote-info">
      <span class="sync-badge" id="syncStatus">Initializing Remote...</span>
      <div class="room-box">
        <span>Room Code:</span>
        <input type="text" id="roomInput" maxlength="8" onchange="changeRoomCode(this.value)">
      </div>
    </div>
    <div class="remote-actions">
      <button class="copy-url-btn" onclick="copyPrismUrl(false)">Copy PRISM Overlay URL</button>
      <button class="header-btn" onclick="copyPrismUrl(true)">Copy Green-Screen URL</button>
    </div>
  </div>

  <!-- TV Broadcast Scoreboard Overlay -->
  <div class="broadcast-wrapper">
    <div class="scoreboard-overlay">
      <!-- Team A Display -->
      <div class="overlay-team team-a">
        <div class="team-logo-box" id="logoDisplayA">TMA</div>
        <div class="team-info">
          <span class="team-short-tag" id="shortDisplayA">TMA</span>
          <span class="team-full-name" id="nameDisplayA">TEAM A</span>
          <div class="overlay-cards">
            <span class="card-badge"><span class="card-icon yellow"></span> <span id="yellowDisplayA">0</span></span>
            <span class="card-badge"><span class="card-icon red"></span> <span id="redDisplayA">0</span></span>
          </div>
        </div>
      </div>

      <!-- Scores & Match Clock -->
      <div class="overlay-center">
        <div class="score-box" id="scoreDisplayA">0</div>
        <div class="clock-box">
          <span class="match-period" id="periodDisplay">1ST HALF</span>
          <span class="match-clock" id="clockDisplay">00:00</span>
        </div>
        <div class="score-box" id="scoreDisplayB">0</div>
      </div>

      <!-- Team B Display -->
      <div class="overlay-team team-b">
        <div class="team-logo-box" id="logoDisplayB">TMB</div>
        <div class="team-info">
          <span class="team-short-tag" id="shortDisplayB">TMB</span>
          <span class="team-full-name" id="nameDisplayB">TEAM B</span>
          <div class="overlay-cards">
            <span class="card-badge"><span class="card-icon yellow"></span> <span id="yellowDisplayB">0</span></span>
            <span class="card-badge"><span class="card-icon red"></span> <span id="redDisplayB">0</span></span>
          </div>
        </div>
      </div>
    </div>
  </div>

  <!-- Operator Control Deck -->
  <div class="control-deck" id="controlDeck">

    <!-- TEAM A CONTROLS -->
    <div class="panel panel-a">
      <div class="panel-header">
        <span class="panel-title">Team A Controls</span>
        <input type="color" id="colorInputA" value="#00d2ff" onchange="updateTeamColor('A', this.value)">
      </div>

      <div class="score-control-group">
        <button class="btn-score btn-plus" onclick="changeScore('A', 1)">+</button>
        <button class="btn-score btn-minus" onclick="changeScore('A', -1)">−</button>
      </div>

      <div class="form-row">
        <div class="input-group" style="flex: 2;">
          <label>Team Name</label>
          <input type="text" id="nameInputA" value="TEAM A" maxlength="20" oninput="updateTeamName('A', this.value)">
        </div>
        <div class="input-group" style="flex: 1;">
          <label>Short Name</label>
          <input type="text" id="shortInputA" value="TMA" maxlength="4" oninput="updateTeamShort('A', this.value)">
        </div>
      </div>

      <div class="input-group">
        <label>Team Logo (Optional)</label>
        <div class="logo-upload-row">
          <label class="file-btn">
            Upload Logo Image
            <input type="file" accept="image/*" style="display:none" onchange="uploadLogo('A', event)">
          </label>
          <button class="clear-logo-btn" onclick="clearLogo('A')">Clear</button>
        </div>
      </div>

      <div class="cards-control-section">
        <div class="card-ctrl-box">
          <span class="card-ctrl-label"><span class="card-icon yellow"></span> Yellow Card</span>
          <div class="card-ctrl-actions">
            <button class="btn-mini" onclick="changeCard('A', 'yellow', -1)">−</button>
            <span class="card-count-num" id="yellowCtrlA">0</span>
            <button class="btn-mini" onclick="changeCard('A', 'yellow', 1)">+</button>
          </div>
        </div>
        <div class="card-ctrl-box">
          <span class="card-ctrl-label"><span class="card-icon red"></span> Red Card</span>
          <div class="card-ctrl-actions">
            <button class="btn-mini" onclick="changeCard('A', 'red', -1)">−</button>
            <span class="card-count-num" id="redCtrlA">0</span>
            <button class="btn-mini" onclick="changeCard('A', 'red', 1)">+</button>
          </div>
        </div>
      </div>
    </div>

    <!-- MATCH TIMER CONTROLS -->
    <div class="panel">
      <div class="panel-header">
        <span class="panel-title" style="color:#e2e8f0;">Match Clock</span>
      </div>

      <div class="timer-display-preview" id="timerPreview">00:00</div>

      <div class="timer-buttons">
        <button class="btn-timer btn-start" onclick="startTimer()">Start</button>
        <button class="btn-timer btn-pause" onclick="pauseTimer()">Pause</button>
        <button class="btn-timer btn-reset" onclick="resetTimer()">Reset Timer</button>
      </div>

      <div class="input-group">
        <label>Match Status / Half</label>
        <select id="periodSelect" onchange="updatePeriod(this.value)">
          <option value="1ST HALF">1ST HALF</option>
          <option value="HALF TIME">HALF TIME</option>
          <option value="2ND HALF">2ND HALF</option>
          <option value="EXTRA TIME">EXTRA TIME</option>
          <option value="PENALTIES">PENALTIES</option>
          <option value="FULL TIME">FULL TIME</option>
        </select>
      </div>

      <button class="btn-reset-all" onclick="resetMatch()">Reset Full Match (Score & Cards)</button>
    </div>

    <!-- TEAM B CONTROLS -->
    <div class="panel panel-b">
      <div class="panel-header">
        <span class="panel-title">Team B Controls</span>
        <input type="color" id="colorInputB" value="#ff2a55" onchange="updateTeamColor('B', this.value)">
      </div>

      <div class="score-control-group">
        <button class="btn-score btn-plus" onclick="changeScore('B', 1)">+</button>
        <button class="btn-score btn-minus" onclick="changeScore('B', -1)">−</button>
      </div>

      <div class="form-row">
        <div class="input-group" style="flex: 2;">
          <label>Team Name</label>
          <input type="text" id="nameInputB" value="TEAM B" maxlength="20" oninput="updateTeamName('B', this.value)">
        </div>
        <div class="input-group" style="flex: 1;">
          <label>Short Name</label>
          <input type="text" id="shortInputB" value="TMB" maxlength="4" oninput="updateTeamShort('B', this.value)">
        </div>
      </div>

      <div class="input-group">
        <label>Team Logo (Optional)</label>
        <div class="logo-upload-row">
          <label class="file-btn">
            Upload Logo Image
            <input type="file" accept="image/*" style="display:none" onchange="uploadLogo('B', event)">
          </label>
          <button class="clear-logo-btn" onclick="clearLogo('B')">Clear</button>
        </div>
      </div>

      <div class="cards-control-section">
        <div class="card-ctrl-box">
          <span class="card-ctrl-label"><span class="card-icon yellow"></span> Yellow Card</span>
          <div class="card-ctrl-actions">
            <button class="btn-mini" onclick="changeCard('B', 'yellow', -1)">−</button>
            <span class="card-count-num" id="yellowCtrlB">0</span>
            <button class="btn-mini" onclick="changeCard('B', 'yellow', 1)">+</button>
          </div>
        </div>
        <div class="card-ctrl-box">
          <span class="card-ctrl-label"><span class="card-icon red"></span> Red Card</span>
          <div class="card-ctrl-actions">
            <button class="btn-mini" onclick="changeCard('B', 'red', -1)">−</button>
            <span class="card-count-num" id="redCtrlB">0</span>
            <button class="btn-mini" onclick="changeCard('B', 'red', 1)">+</button>
          </div>
        </div>
      </div>
    </div>

  </div>

  <script>
    const params = new URLSearchParams(window.location.search);
    const isOverlayMode = params.get('mode') === 'overlay';
    const bgParam = params.get('bg');

    // Initialize Overlay Styles if launched via PRISM URL
    if (isOverlayMode) {
      document.body.classList.add('overlay-mode');
      if (bgParam === 'green') {
        document.documentElement.classList.add('green-bg');
        document.body.classList.add('green-bg');
      } else {
        document.documentElement.classList.add('transparent-bg');
        document.body.classList.add('transparent-bg');
      }
    }

    // Room Code setup
    let roomCode = params.get('room') || localStorage.getItem('prism_sb_room') || String(Math.floor(1000 + Math.random() * 9000));
    if (!isOverlayMode) {
      localStorage.setItem('prism_sb_room', roomCode);
      document.getElementById('roomInput').value = roomCode;
    }

    // Main Shared State
    const state = {
      A: { score: 0, yellow: 0, red: 0, name: 'TEAM A', short: 'TMA', color: '#00d2ff', logo: '' },
      B: { score: 0, yellow: 0, red: 0, name: 'TEAM B', short: 'TMB', color: '#ff2a55', logo: '' },
      period: '1ST HALF',
      elapsedSeconds: 0,
      isRunning: false
    };

    let timerInterval = null;
    let peer = null;
    let connections = [];
    let bc = null;

    // BroadcastChannel for same-browser/PC instant sync
    function initBroadcastChannel() {
      if ('BroadcastChannel' in window) {
        if (bc) bc.close();
        bc = new BroadcastChannel('prism_sb_' + roomCode);
        if (isOverlayMode) {
          bc.onmessage = (e) => applyState(e.data);
        }
      }
    }

    // Apply received state (Used by PRISM Overlay)
    function applyState(newState) {
      if (!newState) return;
      Object.assign(state, newState);

      ['A', 'B'].forEach(team => {
        document.getElementById(`scoreDisplay${team}`).textContent = state[team].score;
        document.getElementById(`yellowDisplay${team}`).textContent = state[team].yellow;
        document.getElementById(`redDisplay${team}`).textContent = state[team].red;
        document.getElementById(`nameDisplay${team}`).textContent = state[team].name;
        document.getElementById(`shortDisplay${team}`).textContent = state[team].short;

        const prop = team === 'A' ? '--team-a-color' : '--team-b-color';
        document.documentElement.style.setProperty(prop, state[team].color);

        const logoBox = document.getElementById(`logoDisplay${team}`);
        if (state[team].logo) {
          logoBox.innerHTML = `<img src="${state[team].logo}" alt="Logo">`;
        } else {
          logoBox.textContent = state[team].short;
        }
      });

      document.getElementById('periodDisplay').textContent = state.period;
      renderTime();

      // Sync local timer ticking on overlay for smooth seconds
      if (state.isRunning && !timerInterval) {
        timerInterval = setInterval(() => {
          state.elapsedSeconds++;
          renderTime();
        }, 1000);
      } else if (!state.isRunning && timerInterval) {
        clearInterval(timerInterval);
        timerInterval = null;
      }
    }

    // Send state from Controller to all connected PRISM Overlays
    function broadcastState() {
      const payload = JSON.parse(JSON.stringify(state));
      if (bc) bc.postMessage(payload);
      connections = connections.filter(c => c.open);
      connections.forEach(c => {
        try { c.send(payload); } catch (err) {}
      });
    }

    // PeerJS Setup
    function initPeer() {
      if (typeof Peer === 'undefined') return;
      if (peer) peer.destroy();

      const statusEl = document.getElementById('syncStatus');

      if (!isOverlayMode) {
        // CONTROLLER MODE
        peer = new Peer('prism-sb-ctrl-' + roomCode);
        peer.on('open', () => {
          updateControllerStatus();
        });
        peer.on('connection', (conn) => {
          conn.on('open', () => {
            connections.push(conn);
            conn.send(JSON.parse(JSON.stringify(state)));
            updateControllerStatus();
          });
          conn.on('close', () => updateControllerStatus());
        });
        peer.on('error', () => {
          if (statusEl) {
            statusEl.textContent = 'Offline / Retry Room';
            statusEl.classList.remove('connected');
          }
        });
      } else {
        // PRISM OVERLAY MODE
        peer = new Peer();
        peer.on('open', () => connectToController());
        peer.on('error', () => setTimeout(connectToController, 3000));
      }
    }

    function connectToController() {
      if (!peer || peer.destroyed) return;
      const conn = peer.connect('prism-sb-ctrl-' + roomCode, { reliable: true });
      conn.on('data', (data) => applyState(data));
      conn.on('close', () => setTimeout(connectToController, 3000));
      conn.on('error', () => setTimeout(connectToController, 3000));
    }

    function updateControllerStatus() {
      const statusEl = document.getElementById('syncStatus');
      if (!statusEl) return;
      const activeCount = connections.filter(c => c.open).length;
      if (activeCount > 0) {
        statusEl.textContent = `PRISM Connected (${activeCount})`;
        statusEl.classList.add('connected');
      } else {
        statusEl.textContent = 'Ready for PRISM Link';
        statusEl.classList.remove('connected');
      }
    }

    function changeRoomCode(val) {
      const cleaned = val.replace(/[^a-zA-Z0-9_-]/g, '').trim() || '1234';
      roomCode = cleaned;
      document.getElementById('roomInput').value = roomCode;
      localStorage.setItem('prism_sb_room', roomCode);
      connections = [];
      initBroadcastChannel();
      initPeer();
    }

    function copyPrismUrl(greenScreen) {
      if (window.location.protocol === 'file:') {
        alert('PRISM Live মোবাইল অ্যাপে চালাতে হলে ফাইলটি আগে Netlify Drop বা GitHub Pages-এ আপলোড করে https:// লিংক তৈরি করুন (নিচের নির্দেশনা দেখুন)।');
      }
      const baseUrl = window.location.href.split('?')[0];
      const url = `${baseUrl}?mode=overlay&room=${encodeURIComponent(roomCode)}${greenScreen ? '&bg=green' : ''}`;
      if (navigator.clipboard && navigator.clipboard.writeText) {
        navigator.clipboard.writeText(url).then(() => {
          alert('PRISM Overlay URL কপি হয়েছে!\n\nএখন PRISM Live Studio-এর Web/Browser অপশনে গিয়ে এটি পেস্ট করুন:\n' + url);
        });
      } else {
        prompt('এই লিংকটি কপি করে PRISM Live-এ বসান:', url);
      }
    }

    // ================= CONTROLLER ACTIONS =================
    function changeScore(team, delta) {
      state[team].score = Math.max(0, state[team].score + delta);
      document.getElementById(`scoreDisplay${team}`).textContent = state[team].score;
      broadcastState();
    }

    function changeCard(team, type, delta) {
      state[team][type] = Math.max(0, state[team][type] + delta);
      document.getElementById(`${type}Display${team}`).textContent = state[team][type];
      document.getElementById(`${type}Ctrl${team}`).textContent = state[team][type];
      broadcastState();
    }

    function updateTeamName(team, value) {
      const clean = value.trim() || `TEAM ${team}`;
      state[team].name = clean;
      document.getElementById(`nameDisplay${team}`).textContent = clean;
      broadcastState();
    }

    function updateTeamShort(team, value) {
      const cleanShort = (value.trim() || `TM${team}`).toUpperCase();
      state[team].short = cleanShort;
      document.getElementById(`shortDisplay${team}`).textContent = cleanShort;
      if (!state[team].logo) {
        document.getElementById(`logoDisplay${team}`).textContent = cleanShort;
      }
      broadcastState();
    }

    function updateTeamColor(team, color) {
      state[team].color = color;
      const prop = team === 'A' ? '--team-a-color' : '--team-b-color';
      document.documentElement.style.setProperty(prop, color);
      broadcastState();
    }

    // Resize Logo to 120x120 max for instant WebRTC sync
    function uploadLogo(team, event) {
      const file = event.target.files[0];
      if (!file) return;
      const reader = new FileReader();
      reader.onload = function(e) {
        const img = new Image();
        img.onload = function() {
          const canvas = document.createElement('canvas');
          const max = 120;
          let w = img.width, h = img.height;
          if (w > h) { if (w > max) { h *= max / w; w = max; } }
          else { if (h > max) { w *= max / h; h = max; } }
          canvas.width = w;
          canvas.height = h;
          canvas.getContext('2d').drawImage(img, 0, 0, w, h);
          const compressed = canvas.toDataURL('image/png');
          state[team].logo = compressed;
          document.getElementById(`logoDisplay${team}`).innerHTML = `<img src="${compressed}" alt="Logo">`;
          broadcastState();
        };
        img.src = e.target.result;
      };
      reader.readAsDataURL(file);
    }

    function clearLogo(team) {
      state[team].logo = '';
      document.getElementById(`logoDisplay${team}`).textContent = state[team].short;
      broadcastState();
    }

    function updatePeriod(value) {
      state.period = value;
      document.getElementById('periodDisplay').textContent = value;
      broadcastState();
    }

    function formatTime(totalSeconds) {
      const mins = String(Math.floor(totalSeconds / 60)).padStart(2, '0');
      const secs = String(totalSeconds % 60).padStart(2, '0');
      return `${mins}:${secs}`;
    }

    function renderTime() {
      const formatted = formatTime(state.elapsedSeconds);
      document.getElementById('clockDisplay').textContent = formatted;
      const preview = document.getElementById('timerPreview');
      if (preview) preview.textContent = formatted;
    }

    function startTimer() {
      if (state.isRunning) return;
      state.isRunning = true;
      broadcastState();
      timerInterval = setInterval(() => {
        state.elapsedSeconds++;
        renderTime();
        // Periodic sync every 5 seconds to keep overlay clock exact
        if (state.elapsedSeconds % 5 === 0) broadcastState();
      }, 1000);
    }

    function pauseTimer() {
      if (!state.isRunning) return;
      state.isRunning = false;
      clearInterval(timerInterval);
      timerInterval = null;
      broadcastState();
    }

    function resetTimer() {
      pauseTimer();
      state.elapsedSeconds = 0;
      renderTime();
      broadcastState();
    }

    function resetMatch() {
      resetTimer();
      ['A', 'B'].forEach(team => {
        state[team].score = 0;
        state[team].yellow = 0;
        state[team].red = 0;
        document.getElementById(`scoreDisplay${team}`).textContent = '0';
        document.getElementById(`yellowDisplay${team}`).textContent = '0';
        document.getElementById(`yellowCtrl${team}`).textContent = '0';
        document.getElementById(`redDisplay${team}`).textContent = '0';
        document.getElementById(`redCtrl${team}`).textContent = '0';
      });
      broadcastState();
    }

    function toggleControls() {
      const deck = document.getElementById('controlDeck');
      const btn = document.getElementById('toggleControlsBtn');
      deck.classList.toggle('hidden');
      btn.textContent = deck.classList.contains('hidden') ? 'Show Controls' : 'Hide Controls';
    }

    function toggleTransparent() {
      const isTrans = document.body.classList.toggle('transparent-bg');
      document.documentElement.classList.toggle('transparent-bg', isTrans);
      const btn = document.getElementById('transparentBtn');
      btn.classList.toggle('active', isTrans);
      btn.textContent = isTrans ? 'Transparent BG: ON' : 'Transparent BG: OFF';
    }

    // Start sync engines
    initBroadcastChannel();
    initPeer();
  </script>
</body>
</html>
