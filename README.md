<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>AgriLens AI - Multilingual Plant Pathology Scanner</title>
  <style>
    * { box-sizing: border-box; margin: 0; padding: 0; font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif; }
    body { background-color: #0d1117; color: #f0f6fc; padding: 14px; min-height: 100vh; display: flex; justify-content: center; }
    .container { width: 100%; max-width: 500px; }

    /* Navbar */
    .navbar { display: flex; justify-content: space-between; align-items: center; padding: 10px 0; margin-bottom: 12px; gap: 8px; }
    .brand { display: flex; align-items: center; gap: 8px; font-weight: 700; font-size: 1.15rem; color: #fff; }
    .logo-badge { background: #238636; width: 34px; height: 34px; border-radius: 8px; display: flex; align-items: center; justify-content: center; font-size: 1.1rem; }
    .lang-select { background: #161b22; border: 1px solid #30363d; padding: 7px 10px; border-radius: 20px; font-size: 0.82rem; color: #f0f6fc; outline: none; cursor: pointer; max-width: 175px; }
    .lang-select:focus { border-color: #3fb950; }

    /* Header */
    .terminal-tag { display: inline-block; background: #0f2d1e; color: #3fb950; border: 1px solid #238636; padding: 4px 10px; border-radius: 20px; font-size: 0.72rem; font-weight: 700; letter-spacing: 0.5px; margin-bottom: 8px; }
    h1 { font-size: 1.5rem; font-weight: 800; line-height: 1.25; margin-bottom: 6px; }
    p.subtitle { color: #8b949e; font-size: 0.86rem; line-height: 1.4; margin-bottom: 14px; }

    /* API Key Bar */
    .api-bar { background: #161b22; border: 1px solid #30363d; border-radius: 12px; padding: 10px 12px; margin-bottom: 14px; display: flex; flex-direction: column; gap: 6px; }
    .api-row { display: flex; gap: 8px; align-items: center; }
    .api-input { flex: 1; background: #0d1117; border: 1px solid #30363d; color: #f0f6fc; padding: 8px 10px; border-radius: 8px; font-size: 0.8rem; outline: none; }
    .api-input:focus { border-color: #3fb950; }
    .api-hint { font-size: 0.72rem; color: #8b949e; }
    .api-hint a { color: #3fb950; text-decoration: none; }

    /* Scanner Card */
    .scanner-card { background: #161b22; border: 1px solid #30363d; border-radius: 16px; padding: 16px; margin-bottom: 16px; }
    .card-title { font-size: 0.95rem; font-weight: 600; margin-bottom: 12px; display: flex; justify-content: space-between; align-items: center; color: #e6edf3; }

    /* Off-screen file inputs (safe for all mobile & desktop browsers) */
    .sr-file-input { position: fixed; left: -9999px; top: -9999px; width: 1px; height: 1px; opacity: 0; }

    /* Action Buttons for Camera & Upload */
    .capture-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; margin-bottom: 12px; }
    .capture-btn {
      background: #0d1117;
      border: 1.5px dashed #238636;
      border-radius: 12px;
      padding: 16px 10px;
      text-align: center;
      cursor: pointer;
      transition: 0.2s;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      gap: 6px;
      color: #f0f6fc;
      width: 100%;
    }
    .capture-btn:hover, .capture-btn:active { background: #13231a; border-style: solid; border-color: #3fb950; }
    .cap-icon { font-size: 1.6rem; pointer-events: none; }
    .cap-label { font-size: 0.85rem; font-weight: 700; color: #f0f6fc; pointer-events: none; }
    .cap-sub { font-size: 0.72rem; color: #8b949e; pointer-events: none; }

    /* Drag & Drop / Preview Box */
    .drop-zone {
      background: #0d1117;
      border: 1px solid #30363d;
      border-radius: 12px;
      padding: 14px;
      text-align: center;
      margin-bottom: 12px;
      transition: 0.2s;
    }
    .drop-zone.dragover { border-color: #3fb950; background: #13231a; }
    .drop-placeholder { font-size: 0.8rem; color: #8b949e; padding: 12px 0; }
    #previewImg { display: none; width: 100%; max-height: 260px; object-fit: contain; border-radius: 8px; margin: 0 auto; }
    .file-meta { font-size: 0.76rem; color: #3fb950; margin-top: 8px; font-weight: 600; display: none; }

    /* Diagnose Button */
    .diagnose-btn { width: 100%; background: #238636; color: #fff; border: none; padding: 14px; border-radius: 10px; font-weight: 700; font-size: 0.98rem; cursor: pointer; transition: 0.2s; display: flex; align-items: center; justify-content: center; gap: 8px; }
    .diagnose-btn:hover:not(:disabled) { background: #2ea043; }
    .diagnose-btn:disabled { background: #21262d; color: #8b949e; cursor: not-allowed; }

    /* Auto-speak toggle */
    .auto-voice-row { display: flex; align-items: center; justify-content: space-between; margin-top: 10px; font-size: 0.8rem; color: #c9d1d9; }
    .auto-voice-row input { accent-color: #3fb950; width: 16px; height: 16px; cursor: pointer; }

    /* Result Box */
    .result-box { display: none; background: #0d1117; border: 1px solid #238636; border-radius: 12px; padding: 14px; margin-top: 16px; }
    .res-head { display: flex; justify-content: space-between; align-items: flex-start; gap: 8px; margin-bottom: 8px; flex-wrap: wrap; }
    .res-title { color: #3fb950; font-size: 1.1rem; font-weight: 700; }
    .head-actions { display: flex; align-items: center; gap: 6px; }
    .badge { background: #3fb950; color: #0d1117; font-weight: 700; font-size: 0.72rem; padding: 4px 8px; border-radius: 12px; }
    .voice-btn { background: #1f3b2b; border: 1px solid #238636; color: #3fb950; font-size: 0.78rem; font-weight: 700; padding: 5px 11px; border-radius: 14px; cursor: pointer; display: inline-flex; align-items: center; gap: 5px; }
    .voice-btn.playing { background: #da3633; border-color: #f85149; color: #fff; }
    .crop-line { font-size: 0.85rem; color: #8b949e; margin-bottom: 10px; }
    .crop-line strong { color: #f0f6fc; }
    .remedy-section { background: #161b22; border-left: 3px solid #3fb950; padding: 10px 12px; border-radius: 6px; margin-top: 8px; font-size: 0.84rem; line-height: 1.5; color: #e6edf3; }
    .remedy-section h4 { color: #3fb950; font-size: 0.82rem; text-transform: uppercase; letter-spacing: 0.4px; margin-bottom: 4px; }
    .remedy-section ul { padding-left: 18px; }
    .remedy-section li { margin-bottom: 4px; }
    .error-msg { display: none; background: #2d1215; border: 1px solid #f85149; color: #ffa198; padding: 10px; border-radius: 8px; font-size: 0.82rem; margin-top: 12px; line-height: 1.4; }
  </style>
</head>
<body>
  <div class="container">
    <!-- Top Navbar -->
    <div class="navbar">
      <div class="brand">
        <div class="logo-badge">🌿</div>
        <div>AgriLens <span style="font-size:0.7rem; background:#1f3b2b; color:#3fb950; padding:2px 6px; border-radius:4px;">AI</span></div>
      </div>

      <!-- Major Indian Languages Selector -->
      <select id="langSelect" class="lang-select" aria-label="Select Language">
        <option value="en-IN" data-name="English">🌐 English (India)</option>
        <option value="hi-IN" data-name="Hindi">🇮🇳 हिन्दी (Hindi)</option>
        <option value="kn-IN" data-name="Kannada">🇮🇳 ಕನ್ನಡ (Kannada)</option>
        <option value="te-IN" data-name="Telugu">🇮🇳 తెలుగు (Telugu)</option>
        <option value="ta-IN" data-name="Tamil">🇮🇳 தமிழ் (Tamil)</option>
        <option value="mr-IN" data-name="Marathi">🇮🇳 मराठी (Marathi)</option>
        <option value="bn-IN" data-name="Bengali">🇮🇳 বাংলা (Bengali)</option>
        <option value="gu-IN" data-name="Gujarati">🇮🇳 ગુજરાતી (Gujarati)</option>
        <option value="ml-IN" data-name="Malayalam">🇮🇳 മലയാളം (Malayalam)</option>
        <option value="pa-IN" data-name="Punjabi">🇮🇳 ਪੰਜਾਬੀ (Punjabi)</option>
      </select>
    </div>

    <div class="terminal-tag">GLOBAL BOTANICAL VISION AI</div>
    <h1>AI Leaf Scanner &amp; Pathology Engine</h1>
    <p class="subtitle">Capture a live leaf photo or upload from your gallery. Identifies any plant species, detects diseases, and speaks solutions in your language.</p>

    <!-- Gemini API Key Setup -->
    <div class="api-bar">
      <div class="api-row">
        <input type="password" id="apiKeyInput" class="api-input" placeholder="Paste Gemini API Key (or leave blank for Demo Mode)" />
      </div>
      <div class="api-hint">
        Get a free key at <a href="https://aistudio.google.com/apikey" target="_blank" rel="noopener">Google AI Studio</a> to scan all real-world plant species live.
      </div>
    </div>

    <!-- Main Scanner Card -->
    <div class="scanner-card">
      <div class="card-title">
        <span>📷 Capture or Upload Leaf Photo</span>
      </div>

      <!-- Dedicated Hidden Inputs -->
      <input type="file" id="cameraInput" class="sr-file-input" accept="image/*" capture="environment" />
      <input type="file" id="galleryInput" class="sr-file-input" accept="image/*" />

      <!-- Explicit Native Buttons Triggering Inputs -->
      <div class="capture-grid">
        <button type="button" id="openCameraBtn" class="capture-btn">
          <span class="cap-icon">📸</span>
          <span class="cap-label">Take Live Photo</span>
          <span class="cap-sub">Opens Rear Camera</span>
        </button>

        <button type="button" id="openGalleryBtn" class="capture-btn">
          <span class="cap-icon">🖼️</span>
          <span class="cap-label">Upload Picture</span>
          <span class="cap-sub">From Device Gallery</span>
        </button>
      </div>

      <!-- Image Preview & Drag/Drop Zone -->
      <div id="dropZone" class="drop-zone">
        <div id="dropPlaceholder" class="drop-placeholder">
          No leaf photo selected yet. Tap a button above or drag &amp; drop an image here.
        </div>
        <img id="previewImg" alt="Selected plant leaf" />
        <div id="file
