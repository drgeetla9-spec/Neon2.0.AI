<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>NeonCanvas Smart AI</title>
    <style>
        :root {
            --bg-color: #1e1e24;
            --panel-color: #2b2b36;
            --accent-color: #00e5ff;
            --accent-hover: #00b8cc;
            --text-color: #e0e0e0;
            --border-color: #444450;
            --success-color: #00e676;
            --danger-color: #ff1744;
            --shape-hover: rgba(255, 255, 255, 0.1);
            --shape-active: rgba(0, 229, 255, 0.2);
            --chat-bg: rgba(30, 30, 36, 0.98);
        }

        * { box-sizing: border-box; user-select: none; }

        body {
            margin: 0;
            padding: 0;
            font-family: 'Segoe UI', Roboto, sans-serif;
            background-color: var(--bg-color);
            color: var(--text-color);
            height: 100vh;
            display: flex;
            flex-direction: column;
            overflow: hidden;
        }

        /* =========================================
           LOADOUT / LOADING SCREEN STYLES
           ========================================= */
        #loadout-screen {
            position: fixed;
            top: 0; left: 0; width: 100%; height: 100%;
            background: #050505;
            z-index: 10000;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            transition: opacity 1s ease-out, visibility 1s;
            font-family: 'Courier New', Courier, monospace;
            overflow: hidden;
        }

        #loadout-screen::after {
            content: " ";
            display: block;
            position: absolute;
            top: 0; left: 0; bottom: 0; right: 0;
            background: linear-gradient(rgba(18, 16, 16, 0) 50%, rgba(0, 0, 0, 0.25) 50%), linear-gradient(90deg, rgba(255, 0, 0, 0.06), rgba(0, 255, 0, 0.02), rgba(0, 0, 255, 0.06));
            z-index: 2;
            background-size: 100% 2px, 3px 100%;
            pointer-events: none;
        }

        .hologram-container {
            position: relative;
            width: 300px;
            height: 300px;
            perspective: 1000px;
            margin-bottom: 40px;
        }

        .globe {
            width: 100%; height: 100%;
            position: relative;
            transform-style: preserve-3d;
            animation: rotateEarth 15s linear infinite;
        }

        .ring {
            position: absolute;
            top: 50%; left: 50%;
            transform: translate(-50%, -50%);
            border-radius: 50%;
            border: 1px solid var(--accent-color);
            box-shadow: 0 0 10px var(--accent-color), inset 0 0 10px var(--accent-color);
            opacity: 0.6;
        }

        .ring:nth-child(1) { width: 300px; height: 300px; transform: translate(-50%, -50%) rotateY(0deg); }
        .ring:nth-child(2) { width: 300px; height: 300px; transform: translate(-50%, -50%) rotateY(60deg); }
        .ring:nth-child(3) { width: 300px; height: 300px; transform: translate(-50%, -50%) rotateY(-60deg); }
        .ring:nth-child(4) { width: 300px; height: 300px; transform: translate(-50%, -50%) rotateX(60deg); }
        .ring:nth-child(5) { width: 300px; height: 300px; transform: translate(-50%, -50%) rotateX(-60deg); }
        
        .globe-core {
            position: absolute;
            top: 50%; left: 50%;
            transform: translate(-50%, -50%);
            width: 100px; height: 100px;
            background: radial-gradient(circle, var(--accent-color), transparent);
            border-radius: 50%;
            box-shadow: 0 0 40px var(--accent-color);
            animation: pulse 2s ease-in-out infinite;
        }

        @keyframes rotateEarth {
            0% { transform: rotateY(0deg) rotateX(15deg); }
            100% { transform: rotateY(360deg) rotateX(15deg); }
        }

        @keyframes pulse {
            0%, 100% { transform: translate(-50%, -50%) scale(1); opacity: 0.8; }
            50% { transform: translate(-50%, -50%) scale(1.2); opacity: 1; }
        }

        .loadout-title {
            font-size: 3rem; color: #fff;
            text-shadow: 0 0 10px var(--accent-color), 0 0 20px var(--accent-color);
            letter-spacing: 5px;
            margin-bottom: 10px; z-index: 10;
            text-transform: uppercase;
        }

        .loadout-subtitle { color: #888; margin-bottom: 40px; letter-spacing: 2px; font-size: 0.9rem; z-index: 10; }

        .loadout-options { display: flex; gap: 20px; z-index: 10; }

        .loadout-btn {
            background: transparent;
            border: 1px solid #333;
            color: #555;
            padding: 15px 30px;
            cursor: pointer;
            font-family: inherit;
            font-size: 1rem;
            transition: all 0.3s;
            text-transform: uppercase;
            position: relative;
            overflow: hidden;
        }

        .loadout-btn:hover, .loadout-btn.active {
            color: var(--accent-color);
            border-color: var(--accent-color);
            box-shadow: 0 0 15px var(--accent-color);
            text-shadow: 0 0 5px var(--accent-color);
        }

        .enter-btn {
            margin-top: 30px;
            background: var(--accent-color);
            color: #000;
            border: none;
            padding: 15px 60px;
            font-weight: bold;
            font-size: 1.2rem;
            cursor: pointer;
            clip-path: polygon(10% 0, 100% 0, 100% 70%, 90% 100%, 0 100%, 0 30%);
            transition: transform 0.2s;
            z-index: 10;
        }

        .enter-btn:hover { transform: scale(1.05); box-shadow: 0 0 20px var(--accent-color); }

        #main-app-wrapper {
            opacity: 0;
            transition: opacity 1s ease-in;
            height: 100%;
            display: flex;
            flex-direction: column;
        }

        /* =========================================
           APP STYLES
           ========================================= */
        
        header {
            height: 60px;
            background-color: var(--panel-color);
            display: flex;
            align-items: center;
            justify-content: space-between;
            padding: 0 20px;
            border-bottom: 1px solid var(--border-color);
            z-index: 10;
        }
        .logo { font-size: 1.2rem; font-weight: 700; color: var(--accent-color); display: flex; align-items: center; gap: 10px; }
        .header-controls { display: flex; gap: 10px; }

        main { flex: 1; display: flex; overflow: hidden; position: relative; }

        .toolbar {
            width: 70px;
            background-color: var(--panel-color);
            display: flex;
            flex-direction: column;
            align-items: center;
            padding: 15px 0;
            gap: 10px;
            border-right: 1px solid var(--border-color);
            overflow-y: auto;
        }

        .tool-btn {
            width: 45px; height: 45px;
            border-radius: 8px;
            border: 1px solid transparent;
            background: transparent;
            color: var(--text-color);
            cursor: pointer;
            display: flex;
            justify-content: center;
            align-items: center;
            font-size: 1.2rem;
            position: relative;
            transition: 0.2s;
        }
        .tool-btn:hover { background-color: rgba(255,255,255,0.1); }
        .tool-btn.active { background-color: var(--accent-color); color: #000; box-shadow: 0 0 15px var(--accent-color); }

        .tool-btn .tooltip {
            position: absolute; left: 60px; background: #000; padding: 5px 10px;
            border-radius: 4px; font-size: 0.8rem; opacity: 0; pointer-events: none;
            transition: opacity 0.2s; white-space: nowrap; z-index: 20;
        }
        .tool-btn:hover .tooltip { opacity: 1; }

        .canvas-container {
            flex: 1; position: relative; background-color: #121215;
            display: flex; justify-content: center; align-items: center;
            background-image: radial-gradient(#333 1px, transparent 1px); background-size: 20px 20px;
        }
        canvas { background-color: #fff; box-shadow: 0 0 30px rgba(0,0,0,0.5); cursor: crosshair; }

        .properties-panel {
            width: 300px;
            background-color: var(--panel-color);
            border-left: 1px solid var(--border-color);
            display: flex; flex-direction: column; padding: 15px; gap: 20px;
            overflow-y: auto;
        }

        .panel-section { display: flex; flex-direction: column; gap: 10px; }
        .panel-title { font-size: 0.85rem; text-transform: uppercase; letter-spacing: 1px; color: #888; font-weight: 600; margin-bottom: 5px; }

        input[type="range"] { width: 100%; accent-color: var(--accent-color); }
        input[type="color"] { width: 100%; height: 40px; border: none; cursor: pointer; border-radius: 4px; background: none; }
        input[type="text"] { 
            width: 100%; padding: 10px; border-radius: 4px; border: 1px solid var(--border-color); 
            background: #1e1e24; color: #fff; outline: none; margin-bottom: 5px;
        }
        input[type="text"]:focus { border-color: var(--accent-color); }
        
        .btn {
            padding: 8px 16px; border-radius: 6px; border: none; font-weight: 600; cursor: pointer; transition: 0.2s; width: 100%;
        }
        .btn:active { transform: scale(0.98); }
        .btn-primary { background: linear-gradient(135deg, var(--accent-color), var(--accent-hover)); color: #000; }
        .btn-primary:disabled { opacity: 0.5; cursor: wait; }
        .btn-danger { background-color: rgba(255, 23, 68, 0.2); color: var(--danger-color); width: auto; }
        .btn-secondary { background-color: #444; color: #fff; width: auto;}

        .switch-row { display: flex; justify-content: space-between; align-items: center; }

        .shape-grid { display: grid; grid-template-columns: repeat(5, 1fr); gap: 8px; }
        .shape-item {
            background: #383844;
            border-radius: 6px;
            height: 45px;
            display: flex;
            justify-content: center;
            align-items: center;
            cursor: pointer;
            transition: 0.2s;
            border: 1px solid transparent;
        }
        .shape-item:hover { background: #444; transform: translateY(-2px); }
        .shape-item.selected { border-color: var(--accent-color); background: var(--shape-active); }
        .shape-item svg { width: 24px; height: 24px; fill: #e0e0e0; }

        .sticker-grid { display: grid; grid-template-columns: repeat(5, 1fr); gap: 5px; }
        .sticker-item { background: rgba(255,255,255,0.05); border-radius: 4px; text-align: center; font-size: 1.2rem; cursor: pointer; padding: 5px; }
        .sticker-item:hover { background: rgba(255,255,255,0.15); }
        .sticker-item.selected { background: var(--accent-color); }

        .switch { position: relative; display: inline-block; width: 40px; height: 20px; }
        .switch input { opacity: 0; width: 0; height: 0; }
        .slider { position: absolute; cursor: pointer; top: 0; left: 0; right: 0; bottom: 0; background-color: #ccc; transition: .4s; border-radius: 20px; }
        .slider:before { position: absolute; content: ""; height: 16px; width: 16px; left: 2px; bottom: 2px; background-color: white; transition: .4s; border-radius: 50%; }
        input:checked + .slider { background-color: var(--accent-color); }
        input:checked + .slider:before { transform: translateX(20px); }

        .toast {
            position: fixed; bottom: 20px; left: 50%; transform: translateX(-50%) translateY(100px);
            background: #333; color: #fff; padding: 10px 20px; border-radius: 30px;
            box-shadow: 0 4px 15px rgba(0,0,0,0.3); transition: transform 0.3s; z-index: 1000;
            display: flex; align-items: center; gap: 10px; border: 1px solid #555;
            text-align: center; max-width: 80%; font-size: 0.9rem;
        }
        .toast.show { transform: translateX(-50%) translateY(0); }

        /* =========================================
           CHATBOT STYLES
           ========================================= */
        .chatbot-widget {
            position: fixed;
            bottom: 30px; right: 30px;
            width: 370px; height: 550px;
            background-color: var(--chat-bg);
            border: 1px solid var(--border-color);
            border-radius: 20px;
            box-shadow: 0 15px 50px rgba(0,0,0,0.7);
            display: flex;
            flex-direction: column;
            z-index: 2000;
            transform: translateY(120%);
            transition: transform 0.5s cubic-bezier(0.175, 0.885, 0.32, 1.275);
            backdrop-filter: blur(12px);
        }

        .chatbot-widget.open { transform: translateY(0); }

        .chat-header {
            padding: 18px 20px;
            background: linear-gradient(135deg, #22222c, #2a2a38);
            border-top-left-radius: 20px;
            border-top-right-radius: 20px;
            display: flex;
            justify-content: space-between;
            align-items: center;
            border-bottom: 1px solid #333;
        }

        .chat-header h3 { 
            margin: 0; 
            font-size: 1rem; 
            color: #fff; 
            font-weight: 700; 
            display: flex; 
            align-items: center; 
            gap: 10px; 
        }
        .chat-header h3 .status-dot { width: 8px; height: 8px; background: #0f0; border-radius: 50%; box-shadow: 0 0 10px #0f0; }

        .chat-close { font-size: 1.4rem; cursor: pointer; color: #fff; opacity: 0.7; font-weight: 300; }
        .chat-close:hover { opacity: 1; }

        .chat-messages {
            flex: 1;
            padding: 20px;
            overflow-y: auto;
            display: flex;
            flex-direction: column;
            gap: 15px;
            font-size: 0.95rem;
        }

        .msg {
            padding: 12px 18px;
            border-radius: 18px;
            max-width: 90%;
            line-height: 1.5;
            word-wrap: break-word;
            position: relative;
            animation: popIn 0.3s ease-out;
        }
        
        @keyframes popIn {
            from { opacity: 0; transform: translateY(10px) scale(0.9); }
            to { opacity: 1; transform: translateY(0) scale(1); }
        }

        .msg.bot { 
            background: linear-gradient(145deg, #2b2b38, #242430); 
            color: #fff; 
            align-self: flex-start; 
            border-bottom-left-radius: 4px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.2);
        }

        .msg.user { 
            background: linear-gradient(135deg, var(--accent-color), var(--accent-hover)); 
            color: #000; 
            align-self: flex-end; 
            border-bottom-right-radius: 4px;
            font-weight: 600;
        }

        .msg .math-result { display: block; margin-top: 10px; font-size: 1.3em; color: var(--accent-color); font-weight: bold; }

        .chat-input-area {
            padding: 20px;
            border-top: 1px solid rgba(255,255,255,0.05);
            display: flex;
            gap: 10px;
        }

        .chat-input-area input {
            flex: 1;
            background: #16161a;
            border: 1px solid #333;
            color: #fff;
            padding: 12px 18px;
            border-radius: 30px;
            outline: none;
            font-size: 0.95rem;
            transition: 0.3s;
        }
        .chat-input-area input:focus { border-color: var(--accent-color); box-shadow: 0 0 10px rgba(0, 229, 255, 0.2); }

        .chat-toggle-btn {
            position: fixed;
            bottom: 30px; right: 30px;
            width: 65px; height: 65px;
            background: linear-gradient(135deg, var(--accent-color), var(--accent-hover));
            border-radius: 50%;
            display: flex;
            justify-content: center;
            align-items: center;
            font-size: 1.6rem;
            box-shadow: 0 0 30px rgba(0, 229, 255, 0.5);
            cursor: pointer;
            z-index: 2001;
            transition: transform 0.3s, opacity 0.3s;
            border: 2px solid rgba(255,255,255,0.2);
        }
        .chat-toggle-btn:hover { transform: scale(1.1) rotate(10deg); }
        .chat-toggle-btn.hidden { opacity: 0; pointer-events: none; transform: scale(0); }

        /* Modal & Other styles kept same */
        .modal-overlay { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0, 0, 0, 0.8); z-index: 3000; display: none; justify-content: center; align-items: center; backdrop-filter: blur(5px); }
        .modal-overlay.open { display: flex; animation: fadeIn 0.2s ease-out; }
        .modal-box { background: var(--panel-color); border: 1px solid var(--accent-color); box-shadow: 0 0 30px rgba(0, 229, 255, 0.3); padding: 25px; border-radius: 15px; width: 320px; display: flex; flex-direction: column; gap: 15px; }
        .modal-title { margin: 0; color: var(--accent-color); font-size: 1.2rem; text-transform: uppercase; letter-spacing: 1px; }
        @keyframes fadeIn { from { opacity: 0; transform: scale(0.95); } to { opacity: 1; transform: scale(1); } }
        .wiki-loading { display: inline-block; width: 12px; height: 12px; border: 2px solid rgba(0,0,0,0.3); border-radius: 50%; border-top-color: #000; animation: spin 1s ease-in-out infinite; margin-left: 5px; vertical-align: middle; }
        @keyframes spin { to { transform: rotate(360deg); } }
        .typing-dots { display: inline-flex; gap: 5px; height: 20px; align-items: center; }
        .typing-dots span { width: 8px; height: 8px; background: #888; border-radius: 50%; animation: bounce 1.4s infinite ease-in-out both; }
        .typing-dots span:nth-child(1) { animation-delay: -0.32s; }
        .typing-dots span:nth-child(2) { animation-delay: -0.16s; }
        @keyframes bounce { 0%, 80%, 100% { transform: scale(0); } 40% { transform: scale(1); } }

    </style>
</head>
<body>

    <!-- LOADOUT SCREEN -->
    <div id="loadout-screen">
        <div class="hologram-container">
            <div class="globe">
                <div class="ring"></div><div class="ring"></div><div class="ring"></div><div class="ring"></div><div class="ring"></div>
                <div class="globe-core"></div>
            </div>
        </div>
        <h1 class="loadout-title">Neon Canvas</h1>
        <p class="loadout-subtitle">Select Neural Engine Interface</p>
        <div class="loadout-options">
            <button class="loadout-btn active" onclick="setTheme('cyan', this)">CYAN CORE</button>
            <button class="loadout-btn" onclick="setTheme('magenta', this)">MAGMA CORE</button>
            <button class="loadout-btn" onclick="setTheme('green', this)">MATRIX CORE</button>
        </div>
        <button class="enter-btn" onclick="enterApp()">INITIALIZE SYSTEM</button>
    </div>

    <!-- MAIN APP -->
    <div id="main-app-wrapper">
        <header>
            <div class="logo">NeonCanvas <span style="color:#fff">Smart AI</span></div>
            <div class="header-controls">
                <button class="btn btn-secondary" onclick="undo()">Undo</button>
                <button class="btn btn-secondary" onclick="redo()">Redo</button>
                <button class="btn btn-danger" onclick="clearCanvas()">Clear</button>
                <button class="btn btn-primary" onclick="downloadImage()">Save</button>
            </div>
        </header>

        <main>
            <div class="toolbar">
                <button class="tool-btn active" onclick="setTool('brush')" id="btn-brush">🖌️<span class="tooltip">Brush</span></button>
                <button class="tool-btn" onclick="setTool('eraser')" id="btn-eraser">🧽<span class="tooltip">Eraser</span></button>
                <button class="tool-btn" onclick="setTool('fill')" id="btn-fill">🪣<span class="tooltip">Fill</span></button>
                <button class="tool-btn" onclick="setTool('text')" id="btn-text">ABC<span class="tooltip">Text</span></button>
                <div style="width: 80%; height: 1px; background: #444; margin: 5px 0;"></div>
                <button class="tool-btn" onclick="setTool('shape')" id="btn-shape">📐<span class="tooltip">Shape</span></button>
                <button class="tool-btn" onclick="setTool('sticker')" id="btn-sticker">😀<span class="tooltip">Sticker</span></button>
            </div>

            <div class="canvas-container" id="canvas-wrapper">
                <canvas id="drawing-board"></canvas>
            </div>

            <div class="properties-panel">
                <div class="panel-section">
                    <div class="panel-title">AI Generator</div>
                    <div style="font-size:0.8rem; color:#888; margin-bottom:5px;">Paste any topic image:</div>
                    <input type="text" id="gen-prompt" placeholder="e.g. Eiffel Tower, Lion...">
                    <button class="btn btn-primary" id="gen-btn" onclick="generateFromWikipedia()">
                        <span id="gen-btn-text">Search & Paste</span>
                    </button>
                </div>
                <hr style="width:100%; border:0; border-top:1px solid #444;">
                <div class="panel-section" id="brush-settings">
                    <div class="panel-title">Settings</div>
                    <div class="switch-row"><span>Fill Shapes</span>
                        <label class="switch">
                            <input type="checkbox" id="fill-shape-toggle">
                            <span class="slider"></span>
                        </label>
                    </div>
                    <label>Color</label>
                    <input type="color" id="color-picker" value="#000000">
                    <label>Size <span id="size-val" style="color:#888">5px</span></label>
                    <input type="range" id="size-slider" min="1" max="100" value="5">
                </div>
                <div class="panel-section" id="shape-library-panel">
                    <div class="panel-title">Shape Library</div>
                    <div class="shape-grid" id="shape-grid"></div>
                </div>
                <div class="panel-section">
                    <div class="panel-title">Stickers</div>
                    <div class="sticker-grid" id="sticker-grid"></div>
                </div>
            </div>
        </main>

        <div class="chat-toggle-btn" id="chat-toggle" onclick="toggleChat()">💬</div>

        <!-- CHATBOT WINDOW -->
        <div class="chatbot-widget" id="chat-widget">
            <div class="chat-header">
                <div class="chat-header-left">
                    <h3><span class="status-dot"></span><span id="chat-title-text">Smart Assistant</span></h3>
                </div>
                <span class="chat-close" onclick="toggleChat()">×</span>
            </div>
            <div class="chat-messages" id="chat-messages"></div>
            <div class="chat-input-area">
                <input type="text" id="chat-input" placeholder="Chat with me..." onkeypress="handleChatKey(event)">
                <button class="btn btn-primary" style="width: auto; padding: 8px 20px; border-radius: 20px;" onclick="sendChat()">Send</button>
            </div>
        </div>

        <div class="modal-overlay" id="text-modal">
            <div class="modal-box">
                <h3 class="modal-title">Add Text</h3>
                <input type="text" id="text-modal-input" placeholder="Type here..." autocomplete="off">
                <div style="display:flex; gap:10px;">
                    <button class="btn btn-secondary" onclick="closeTextModal()">Cancel</button>
                    <button class="btn btn-primary" onclick="confirmText()">Add</button>
                </div>
            </div>
        </div>
        <div class="toast" id="toast">Action Successful</div>
    </div>

    <script>
        // --- LOADOUT LOGIC ---
        function setTheme(colorName, btnElement) {
            const root = document.documentElement;
            document.querySelectorAll('.loadout-btn').forEach(btn => btn.classList.remove('active'));
            btnElement.classList.add('active');
            if (colorName === 'cyan') { root.style.setProperty('--accent-color', '#00e5ff'); root.style.setProperty('--accent-hover', '#00b8cc'); }
            else if (colorName === 'magenta') { root.style.setProperty('--accent-color', '#ff0055'); root.style.setProperty('--accent-hover', '#cc0044'); }
            else if (colorName === 'green') { root.style.setProperty('--accent-color', '#0aff0a'); root.style.setProperty('--accent-hover', '#08cc08'); }
        }

        function enterApp() {
            const loader = document.getElementById('loadout-screen');
            const mainApp = document.getElementById('main-app-wrapper');
            loader.style.opacity = '0';
            loader.style.pointerEvents = 'none'; 
            mainApp.style.opacity = '1';
            setTimeout(() => {
                loader.style.display = 'none';
                setTimeout(() => { addWelcomeMessage(); }, 500);
            }, 1000);
        }

        // --- APP STATE ---
        const canvas = document.getElementById('drawing-board');
        const ctx = canvas.getContext('2d', { willReadFrequently: true });
        const wrapper = document.getElementById('canvas-wrapper');
        
        let isDrawing = false, startX, startY, snapshot;
        let history = [], historyStep = -1;
        let currentTool = 'brush', currentColor = '#000000', currentSize = 5, currentShape = 'rect', selectedSticker = '😀', fillShape = false;
        let textPosX = 0, textPosY = 0;

        // --- Shapes & Stickers ---
        const shapes = [
            { id: 'rect', name: 'Rectangle', path: 'M3 3 h18 v18 h-18 z' }, { id: 'square', name: 'Square', path: 'M5 5 h14 v14 h-14 z' },
            { id: 'circle', name: 'Circle', path: 'M12 2 a10 10 0 0 1 0 20 a10 10 0 0 1 0 -20' }, { id: 'ellipse', name: 'Ellipse', path: 'M3 12 a9 5 0 0 1 18 0 a9 5 0 0 1 -18 0' },
            { id: 'triangle', name: 'Triangle', path: 'M12 2 L22 20 L2 20 z' }, { id: 'pentagon', name: 'Pentagon', path: 'M12 2 L22 8.5 L18 20 L6 20 L2 8.5 z' },
            { id: 'hexagon', name: 'Hexagon', path: 'M12 2 L21 7 L21 17 L12 22 L3 17 L3 7 z' }, { id: 'octagon', name: 'Octagon', path: 'M7 2 L17 2 L22 7 L22 17 L17 22 L7 22 L2 17 L2 7 z' },
            { id: 'star', name: 'Star', path: 'M12 2 l2.4 7.2 h7.6 l-6 4.8 2.4 7.2 -6 -4.8 -6 4.8 2.4 -7.2 -6 -4.8 h7.6 z' }, { id: 'star4', name: 'Burst', path: 'M12 2 L14 10 L22 12 L14 14 L12 22 L10 14 L2 12 L10 10 z' },
            { id: 'heart', name: 'Heart', path: 'M12 21.35 l-1.45 -1.32 C5.4 15.36 2 12.28 2 8.5 C2 5.42 4.42 3 7.5 3 c1.74 0 3.41 0.81 4.5 2.09 C13.09 3.81 14.76 3 16.5 3 C19.58 3 22 5.42 22 8.5 c0 3.78 -3.4 6.86 -8.55 11.54 L12 21.35 z' },
            { id: 'arrow', name: 'Arrow', path: 'M20 12 L4 12 M15 7 L20 12 L15 17' }, { id: 'diamond', name: 'Diamond', path: 'M12 2 L22 12 L12 22 L2 12 z' },
            { id: 'cross', name: 'Cross', path: 'M18 6 L6 18 M6 6 L18 18' }, { id: 'donut', name: 'Donut', path: 'M12 2 a10 10 0 0 1 0 20 a10 10 0 0 1 0 -20 M12 6 a6 6 0 0 0 0 12 a6 6 0 0 0 0 -12' },
            { id: 'cloud', name: 'Cloud', path: 'M17.5 19c2.5 0 4.5-2 4.5-4.5 0-2.1-1.4-3.9-3.4-4.4C18.1 7.4 15.3 5 12 5c-3.6 0-6.6 2.8-6.9 6.4C2.4 11.8 0 14.2 0 17c0 3.3 2.7 6 6 6h11.5z' },
            { id: 'shield', name: 'Shield', path: 'M12 1L3 5v6c0 5.55 3.84 10.74 9 12 5.16-1.26 9-6.45 9-12V5l-9-4z' }
        ];
        const stickers = ['😀', '😂', '😎', '🤔', '😴', '🥶', '🤖', '👽', '👻', '💀', '🐶', '🐱', '🐭', '🐸', '🦁', '🦄', '🐙', '🦋', '🌵', '🌻', '🍕', '🍔', '🍩', '🎂', '🎁'];
        const colorMap = {'red': '#ff0000', 'blue': '#0000ff', 'green': '#00ff00', 'yellow': '#ffff00', 'black': '#000000', 'white': '#ffffff', 'purple': '#800080', 'orange': '#ffa500', 'pink': '#ff69b4', 'cyan': '#00ffff', 'brown': '#8b4513', 'gray': '#808080'};

        // --- Setup ---
        window.onload = () => { resizeCanvas(); initShapesUI(); initStickersUI(); saveHistory(); };
        function initShapesUI() {
            const grid = document.getElementById('shape-grid');
            shapes.forEach(s => {
                const div = document.createElement('div');
                div.className = `shape-item ${s.id === currentShape ? 'selected' : ''}`;
                div.innerHTML = `<svg viewBox="0 0 24 24"><path d="${s.path}"/></svg>`;
                div.title = s.name;
                div.onclick = () => { document.querySelectorAll('.shape-item').forEach(i => i.classList.remove('selected')); div.classList.add('selected'); currentShape = s.id; setTool('shape'); };
                grid.appendChild(div);
            });
        }
        function initStickersUI() {
            const grid = document.getElementById('sticker-grid');
            stickers.forEach(s => {
                const div = document.createElement('div');
                div.className = 'sticker-item'; div.innerText = s;
                div.onclick = () => { document.querySelectorAll('.sticker-item').forEach(i => i.classList.remove('selected')); div.classList.add('selected'); selectedSticker = s; setTool('sticker'); };
                grid.appendChild(div);
            });
        }
        window.addEventListener('resize', resizeCanvas);
        function resizeCanvas() {
            let tempContent; if(canvas.width > 0) tempContent = canvas.toDataURL();
            canvas.width = wrapper.clientWidth - 40; canvas.height = wrapper.clientHeight - 40;
            ctx.fillStyle = '#ffffff'; ctx.fillRect(0, 0, canvas.width, canvas.height);
            if (tempContent) { const img = new Image(); img.src = tempContent; img.onload = () => ctx.drawImage(img, 0, 0); } else { saveHistory(); }
        }
        function setTool(tool) { currentTool = tool; document.querySelectorAll('.tool-btn').forEach(btn => btn.classList.remove('active')); const btn = document.getElementById(`btn-${tool}`); if(btn) btn.classList.add('active'); showToast(`Tool: ${tool}`); }

        // --- Event Listeners ---
        const colorPicker = document.getElementById('color-picker'), sizeSlider = document.getElementById('size-slider'), fillShapeToggle = document.getElementById('fill-shape-toggle'), sizeVal = document.getElementById('size-val');
        colorPicker.addEventListener('input', (e) => currentColor = e.target.value);
        sizeSlider.addEventListener('input', (e) => { currentSize = e.target.value; sizeVal.innerText = e.target.value + 'px'; });
        fillShapeToggle.addEventListener('change', (e) => fillShape = e.target.checked);
        canvas.addEventListener('mousedown', startDraw); canvas.addEventListener('mousemove', draw); canvas.addEventListener('mouseup', stopDraw); canvas.addEventListener('mouseout', stopDraw);
        canvas.addEventListener('touchstart', (e) => { e.preventDefault(); const t = e.touches[0]; canvas.dispatchEvent(new MouseEvent('mousedown', {clientX: t.clientX, clientY: t.clientY})); }, {passive: false});
        canvas.addEventListener('touchmove', (e) => { e.preventDefault(); const t = e.touches[0]; canvas.dispatchEvent(new MouseEvent('mousemove', {clientX: t.clientX, clientY: t.clientY})); }, {passive: false});
        canvas.addEventListener('touchend', () => canvas.dispatchEvent(new MouseEvent('mouseup')));
        function getPos(e) { const r = canvas.getBoundingClientRect(); return { x: e.clientX - r.left, y: e.clientY - r.top }; }

        function startDraw(e) {
            isDrawing = true; const p = getPos(e); startX = p.x; startY = p.y;
            ctx.lineWidth = currentSize; ctx.strokeStyle = currentTool === 'eraser' ? '#ffffff' : currentColor; ctx.fillStyle = currentColor;
            ctx.lineCap = 'round'; ctx.lineJoin = 'round';
            if (currentTool === 'text') { textPosX = p.x; textPosY = p.y; return; }
            if (currentTool === 'sticker') { ctx.font = `${currentSize * 5}px Arial`; ctx.textAlign = 'center'; ctx.textBaseline = 'middle'; ctx.fillText(selectedSticker, startX, startY); isDrawing = false; saveHistory(); return; }
            if (currentTool === 'fill') { floodFill(Math.floor(startX), Math.floor(startY), hexToRgba(currentColor)); isDrawing = false; saveHistory(); return; }
            snapshot = ctx.getImageData(0, 0, canvas.width, canvas.height);
            if(currentTool === 'brush' || currentTool === 'eraser') { ctx.beginPath(); ctx.moveTo(startX, startY); }
        }
        function draw(e) {
            if (!isDrawing) return; const p = getPos(e);
            if (currentTool === 'brush' || currentTool === 'eraser') { ctx.lineTo(p.x, p.y); ctx.stroke(); startX = p.x; startY = p.y; }
            else if (currentTool === 'shape') { ctx.putImageData(snapshot, 0, 0); renderShape(currentShape, startX, startY, p.x, p.y); }
        }
        function stopDraw(e) { if (!isDrawing) return; if (currentTool === 'text') { isDrawing = false; openTextModal(); return; } isDrawing = false; ctx.closePath(); saveHistory(); }

        const modal = document.getElementById('text-modal'), modalInput = document.getElementById('text-modal-input');
        function openTextModal() { modal.classList.add('open'); modalInput.value = ''; modalInput.focus(); }
        function closeTextModal() { modal.classList.remove('open'); }
        function confirmText() { const text = modalInput.value; if (text) { ctx.font = `${currentSize * 5}px Arial`; ctx.fillStyle = currentColor; ctx.textBaseline = 'middle'; ctx.fillText(text, textPosX, textPosY); saveHistory(); } closeTextModal(); }
        modalInput.addEventListener('keypress', (e) => { if (e.key === 'Enter') confirmText(); });

        // Shape Rendering Helpers
        function renderShape(type, x1, y1, x2, y2) {
            const w = x2 - x1, h = y2 - y1, cx = x1 + w / 2, cy = y1 + h / 2; ctx.beginPath();
            switch(type) {
                case 'rect': ctx.rect(x1, y1, w, h); break; case 'square': const side = Math.min(Math.abs(w), Math.abs(h)); ctx.rect(x1, y1, w < 0 ? -side : side, h < 0 ? -side : side); break;
                case 'circle': const r = Math.sqrt(w*w + h*h) / 2; ctx.arc(cx, cy, r, 0, 2 * Math.PI); break; case 'ellipse': ctx.ellipse(cx, cy, Math.abs(w/2), Math.abs(h/2), 0, 0, 2 * Math.PI); break;
                case 'triangle': ctx.moveTo(cx, y1); ctx.lineTo(x2, y2); ctx.lineTo(x1, y2); ctx.closePath(); break;
                case 'pentagon': drawPolygon(cx, cy, Math.abs(w/2), 5); break; case 'hexagon': drawPolygon(cx, cy, Math.abs(w/2), 6); break; case 'octagon': drawPolygon(cx, cy, Math.abs(w/2), 8); break;
                case 'star': drawStar(cx, cy, 5, Math.abs(w/2), Math.abs(w/4)); break; case 'star4': drawStar(cx, cy, 4, Math.abs(w/2), Math.abs(w/4)); break;
                case 'heart': drawHeart(cx, cy, Math.abs(w), Math.abs(h)); break; case 'arrow': drawArrow(x1, y1, x2, y2); break;
                case 'diamond': ctx.moveTo(cx, y1); ctx.lineTo(x2, cy); ctx.lineTo(cx, y2); ctx.lineTo(x1, cy); ctx.closePath(); break;
                case 'donut': ctx.ellipse(cx, cy, Math.abs(w/2), Math.abs(h/2), 0, 0, 2*Math.PI); break;
                case 'cloud': drawCloud(cx, cy, Math.abs(w/2)); break; case 'shield': drawShield(cx, cy, w, h); break;
            }
            if (fillShape) ctx.fill(); else ctx.stroke();
        }
        function drawPolygon(x, y, radius, sides) { if (sides < 3) return; ctx.moveTo(x + radius * Math.cos(0), y + radius * Math.sin(0)); for (let i = 1; i <= sides; i++) ctx.lineTo(x + radius * Math.cos(i * 2 * Math.PI / sides), y + radius * Math.sin(i * 2 * Math.PI / sides)); ctx.closePath(); }
        function drawStar(cx, cy, spikes, outerRadius, innerRadius) { let rot = Math.PI / 2 * 3, x = cx, y = cy, step = Math.PI / spikes; ctx.moveTo(cx, cy - outerRadius); for (let i = 0; i < spikes; i++) { x = cx + Math.cos(rot) * outerRadius; y = cy + Math.sin(rot) * outerRadius; ctx.lineTo(x, y); rot += step; x = cx + Math.cos(rot) * innerRadius; y = cy + Math.sin(rot) * innerRadius; ctx.lineTo(x, y); rot += step; } ctx.lineTo(cx, cy - outerRadius); ctx.closePath(); }
        function drawHeart(x, y, width, height) { const topCurveHeight = height * 0.3; ctx.moveTo(x, y + topCurveHeight); ctx.bezierCurveTo(x, y, x - width / 2, y, x - width / 2, y + topCurveHeight); ctx.bezierCurveTo(x - width / 2, y + (height + topCurveHeight) / 2, x, y + (height + topCurveHeight) / 2, x, y + height); ctx.bezierCurveTo(x, y + (height + topCurveHeight) / 2, x + width / 2, y + (height + topCurveHeight) / 2, x + width / 2, y + topCurveHeight); ctx.bezierCurveTo(x + width / 2, y, x, y, x, y + topCurveHeight); }
        function drawArrow(x1, y1, x2, y2) { const headLength = currentSize * 3, angle = Math.atan2(y2 - y1, x2 - x1); ctx.moveTo(x1, y1); ctx.lineTo(x2, y2); ctx.lineTo(x2 - headLength * Math.cos(angle - Math.PI / 6), y2 - headLength * Math.sin(angle - Math.PI / 6)); ctx.moveTo(x2, y2); ctx.lineTo(x2 - headLength * Math.cos(angle + Math.PI / 6), y2 - headLength * Math.sin(angle + Math.PI / 6)); }
        function drawCloud(x, y, r) { ctx.arc(x, y, r, 0, Math.PI * 2); ctx.arc(x - r * 0.8, y + r * 0.2, r * 0.7, 0, Math.PI * 2); ctx.arc(x + r * 0.8, y + r * 0.2, r * 0.7, 0, Math.PI * 2); ctx.arc(x - r * 0.4, y - r * 0.6, r * 0.7, 0, Math.PI * 2); ctx.arc(x + r * 0.4, y - r * 0.6, r * 0.7, 0, Math.PI * 2); }
        function drawShield(x, y, w, h) { ctx.moveTo(x - w/2, y - h/2); ctx.lineTo(x + w/2, y - h/2); ctx.lineTo(x + w/2, y); ctx.quadraticCurveTo(x, y + h/2, x - w/2, y); ctx.lineTo(x - w/2, y - h/2); }

        // --- WIKIPEDIA IMAGE GENERATOR ---
        async function generateFromWikipedia() {
            const prompt = document.getElementById('gen-prompt').value.trim();
            const btn = document.getElementById('gen-btn'); const btnText = document.getElementById('gen-btn-text');
            if(!prompt) { showToast("Enter a topic!"); return; }
            btn.disabled = true; btnText.innerHTML = 'Searching... <span class="wiki-loading"></span>';
            try {
                const searchRes = await fetch(`https://en.wikipedia.org/w/api.php?origin=*&action=query&list=search&srsearch=${encodeURIComponent(prompt)}&limit=1&format=json`);
                const searchData = await searchRes.json();
                if (!searchData.query.search.length) throw new Error("No page found.");
                const title = searchData.query.search[0].title;
                showToast(`Found: ${title}`);
                const imgRes = await fetch(`https://en.wikipedia.org/w/api.php?origin=*&action=query&prop=pageimages&pithumbsize=800&titles=${encodeURIComponent(title)}&format=json`);
                const imgData = await imgRes.json();
                const pages = imgData.query.pages; const pageId = Object.keys(pages)[0];
                if (!pages[pageId].thumbnail) throw new Error("No image.");
                const img = new Image(); img.crossOrigin = "Anonymous";
                img.onload = () => { ctx.fillStyle = '#ffffff'; ctx.fillRect(0, 0, canvas.width, canvas.height); const scale = Math.min(canvas.width / img.width, canvas.height / img.height); ctx.drawImage(img, (canvas.width/2)-(img.width/2)*scale, (canvas.height/2)-(img.height/2)*scale, img.width*scale, img.height*scale); saveHistory(); showToast("Pasted!"); btn.disabled = false; btnText.innerText = 'Search & Paste'; };
                img.onerror = () => { throw new Error("Load fail."); };
                img.src = pages[pageId].thumbnail.source;
            } catch (err) { showToast("Error: " + err.message); btn.disabled = false; btnText.innerText = 'Search & Paste'; }
        }

        function hexToRgba(hex) { const r = parseInt(hex.slice(1, 3), 16), g = parseInt(hex.slice(3, 5), 16), b = parseInt(hex.slice(5, 7), 16); return [r, g, b, 255]; }
        function floodFill(startX, startY, newColor) {
            const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height), data = imageData.data, width = canvas.width, height = canvas.height;
            const startPos = (startY * width + startX) * 4, startR = data[startPos], startG = data[startPos + 1], startB = data[startPos + 2], startA = data[startPos + 3];
            if (startR === newColor[0] && startG === newColor[1] && startB === newColor[2] && startA === newColor[3]) return;
            const tolerance = 32;
            const stack = [[startX, startY]], visited = new Uint8Array(width * height);
            while (stack.length) {
                const [x, y] = stack.pop(), pixelIndex = y * width + x;
                if (visited[pixelIndex]) continue;
                const pos = pixelIndex * 4;
                if (x >= 0 && x < width && y >= 0 && y < height && Math.abs(data[pos]-startR) <= tolerance && Math.abs(data[pos+1]-startG) <= tolerance && Math.abs(data[pos+2]-startB) <= tolerance && Math.abs(data[pos+3]-startA) <= tolerance) {
                    data[pos] = newColor[0]; data[pos+1] = newColor[1]; data[pos+2] = newColor[2]; data[pos+3] = newColor[3]; visited[pixelIndex] = 1;
                    stack.push([x + 1, y], [x - 1, y], [x, y + 1], [x, y - 1]);
                }
            }
            ctx.putImageData(imageData, 0, 0);
        }

        function clearCanvas() { ctx.clearRect(0, 0, canvas.width, canvas.height); ctx.fillStyle = '#ffffff'; ctx.fillRect(0, 0, canvas.width, canvas.height); saveHistory(); showToast("Cleared!"); }
        function saveHistory() { if (historyStep < history.length - 1) history = history.slice(0, historyStep + 1); history.push(canvas.toDataURL()); historyStep++; }
        function undo() { if (historyStep > 0) { historyStep--; restoreHistory(history[historyStep]); } }
        function redo() { if (historyStep < history.length - 1) { historyStep++; restoreHistory(history[historyStep]); } }
        function restoreHistory(d) { const i = new Image(); i.src = d; i.onload = () => { ctx.clearRect(0,0,canvas.width,canvas.height); ctx.drawImage(i,0,0); }; }
        function downloadImage() { try { const a = document.createElement('a'); a.download = 'neon-canvas-art.png'; a.href = canvas.toDataURL(); a.click(); showToast("Saved!"); } catch (e) { alert("Save error. Screenshot instead."); } }
        function showToast(m) { const t = document.getElementById('toast'); t.innerText = m; t.classList.add('show'); setTimeout(()=>t.classList.remove('show'), 3000); }
        document.addEventListener('keydown', e => { if((e.ctrlKey||e.metaKey)&&e.key==='z') { e.preventDefault(); undo(); } if((e.ctrlKey||e.metaKey)&&e.key==='y') { e.preventDefault(); redo(); } });

        // =========================================
        // UNIFIED SMART CHATBOT LOGIC
        // =========================================
        
        function toggleChat() {
            const widget = document.getElementById('chat-widget');
            const toggle = document.getElementById('chat-toggle');
            widget.classList.toggle('open');
            toggle.classList.toggle('hidden');
        }

        function handleChatKey(e) { if (e.key === 'Enter') sendChat(); }
        function getRandomItem(array) { return array[Math.floor(Math.random() * array.length)]; }

        function addWelcomeMessage() {
            const container = document.getElementById('chat-messages');
            container.innerHTML = '';
            addBotMessage("Hey there! I'm your Smart Assistant. I can help you draw, calculate math, tell jokes, or just chat. What's on your mind?");
        }

        // Math Logic
        function isMathQuery(text) { return /[0-9]/.test(text) && /[\+\-\*\/\%]/.test(text); }
        function safeCalculate(expression) { try { const sanitized = expression.replace(/[^0-9+\-*/().%\s]/g, ''); if (!sanitized.trim()) return null; const result = new Function('return ' + sanitized)(); return isFinite(result) ? result : null; } catch (e) { return null; } }

        // --- MAIN RESPONSE FUNCTION ---
        async function sendChat() {
            const input = document.getElementById('chat-input');
            const rawText = input.value.trim();
            if (!rawText) return;

            addMessage(rawText, 'user');
            input.value = '';

            const loadingId = addMessage(`<span class="typing-dots"><span></span><span></span><span></span></span>`, 'bot', true);
            
            // Simulate thinking
            await new Promise(resolve => setTimeout(resolve, 400 + Math.random() * 600));

            let response = generateSmartResponse(rawText);

            removeMessage(loadingId);
            addBotMessage(response);
        }

        function generateSmartResponse(text) {
            const lower = text.toLowerCase().trim();

            // 1. Greetings & Conversational
            if (matches(lower, ["hello", "hi", "hey", "howdy", "yo", "greetings"])) {
                return getRandomItem([
                    "Hey! Good to see you. Ready to create something cool?",
                    "Hi there! I was just admiring the canvas. What shall we make?",
                    "Hello! How are you doing today? Ready to draw?",
                    "Yo! What's up? Let's make some art."
                ]);
            }
            
            if (matches(lower, ["how are you", "how you doing", "you good", "what's up"])) {
                return getRandomItem([
                    "I'm doing fantastic, thanks for asking! My circuits are buzzing. How about you?",
                    "I'm great! Just happy to be here helping you create.",
                    "All systems operational and feeling good! What can I do for you?"
                ]);
            }

            if (matches(lower, ["thank", "thanks", "thx", "appreciate"])) {
                return getRandomItem([
                    "You're welcome! Happy to help.",
                    "Anytime! That's what I'm here for.",
                    "No problem at all! Let me know if you need anything else.",
                    "Glad I could help! Keep creating."
                ]);
            }

            if (matches(lower, ["bye", "goodbye", "see ya", "later", "cya"])) {
                return getRandomItem([
                    "See you later! Don't forget to save your work!",
                    "Bye for now! It was fun chatting.",
                    "Take care! Come back anytime."
                ]);
            }

            // 2. Jokes & Fun
            if (matches(lower, ["joke", "funny", "laugh", "make me laugh"])) {
                const jokes = [
                    "Why did the artist go to jail? Because they were framed! 😂",
                    "What do you call a sleeping dinosaur? A Dino-snore! 🦖",
                    "Why did the computer go to the doctor? Because it had a virus! 🤒",
                    "I told my computer I needed a break, and now it won't stop sending me Kit-Kat ads.",
                    "Why don't scientists trust atoms? Because they make up everything!",
                    "I'm reading a book about anti-gravity. It's impossible to put down!"
                ];
                return getRandomItem(jokes);
            }

            // 3. Help
            if (matches(lower, ["help", "what can you do", "commands"])) {
                return "I can do a lot! Here's a quick list:<br>• <b>Tools</b>: Ask for 'brush', 'eraser', etc.<br>• <b>Colors</b>: Say 'use red' or 'I want blue'.<br>• <b>Shapes</b>: Ask for 'circle', 'star', etc.<br>• <b>Math</b>: I can calculate things like '25 * 4'.<br>• <b>Fun</b>: Ask me to tell a joke!<br>Just talk to me like a friend!";
            }

            // 4. Tools
            if (matches(lower, ["brush", "paint", "pencil"])) { setTool('brush'); return getRandomItem(["Switched to the brush! Go wild.", "Brush tool is ready. Let's paint!", "Got it! Brush is active."]); }
            if (matches(lower, ["eraser", "erase"])) { setTool('eraser'); return getRandomItem(["Eraser is up. Clean slate time.", "Okay, eraser ready. Mistakes? What mistakes?", "Eraser selected."]); }
            if (matches(lower, ["fill", "bucket"])) { setTool('fill'); return "Fill bucket is ready. Click an area to splash some color!"; }
            if (matches(lower, ["text", "type"])) { setTool('text'); return "Text tool selected. Click on the canvas to type something."; }
            if (matches(lower, ["sticker", "emoji"])) { setTool('sticker'); return "Sticker time! Pick one from the right panel and click to stamp it."; }

            // 5. Colors
            for (const [name, code] of Object.entries(colorMap)) {
                if (lower.includes(name)) { 
                    currentColor = code; 
                    document.getElementById('color-picker').value = code; 
                    return getRandomItem([`Ooh, ${name}! Nice choice. Done.`, `Setting color to ${name} for you.`, `${name} is a great color. Go for it!`]); 
                }
            }

            // 6. Shapes
            for (const s of shapes) { 
                if (lower.includes(s.name.toLowerCase())) { 
                    currentShape = s.id; setTool('shape'); 
                    document.querySelectorAll('.shape-item').forEach(i => i.classList.remove('selected'));
                    const idx = shapes.findIndex(sh => sh.id === s.id);
                    if(document.querySelectorAll('.shape-item')[idx]) document.querySelectorAll('.shape-item')[idx].classList.add('selected');
                    return getRandomItem([`${s.name}? Classic choice.`, `Drawing a ${s.name}? You got it.`, `Okay, let's make a ${s.name}.`]); 
                } 
            }

            // 7. Actions
            if (matches(lower, ["clear", "reset"])) { clearCanvas(); return "Wiped it clean! Fresh start."; }
            if (matches(lower, ["save", "download"])) { downloadImage(); return "Saved! Check your downloads folder."; }
            if (matches(lower, ["undo"])) { undo(); return "Oops! Undid that for you."; }
            if (matches(lower, ["redo"])) { redo(); return "Brought it back!"; }

            // 8. Math
            if (isMathQuery(text)) { 
                const res = safeCalculate(text); 
                if (res !== null) return `Let me crunch the numbers... the answer is ${res}.`; 
                return "Hmm, that math looks a bit weird. Try something like '5 + 5'."; 
            }

            // 9. Wikipedia Fallback
            return searchWikipedia(text);
        }

        // Helper to match keywords
        function matches(text, keywords) { return keywords.some(k => text.includes(k)); }

        // Wikipedia Search
        async function searchWikipedia(query) {
            try {
                const searchRes = await fetch(`https://en.wikipedia.org/w/api.php?origin=*&action=query&list=search&srsearch=${encodeURIComponent(query)}&limit=1&format=json`);
                const searchData = await searchRes.json();
                if (!searchData.query.search.length) return `I looked for "${query}" but couldn't find anything. Maybe try a different word?`;

                const title = searchData.query.search[0].title;
                const extractRes = await fetch(`https://en.wikipedia.org/w/api.php?origin=*&action=query&prop=extracts&exintro&explaintext&titles=${encodeURIComponent(title)}&format=json`);
                const extractData = await extractRes.json();
                const pages = extractData.query.pages; const pageId = Object.keys(pages)[0];
                let text = pages[pageId].extract;
                if (!text) return "I found it, but there's no info. Weird!";
                if (text.length > 250) text = text.substring(0, 250) + "...";

                return `I found some info on ${title}:<br>${text}`;
            } catch (e) {
                return "My search skills failed me there. Try again?";
            }
        }

        function addMessage(text, sender, isLoading = false) {
            const container = document.getElementById('chat-messages');
            const div = document.createElement('div');
            div.className = `msg ${sender}`;
            div.innerHTML = text;
            if(isLoading) div.id = 'msg-' + Date.now();
            container.appendChild(div);
            container.scrollTop = container.scrollHeight;
            return div.id;
        }
        function addBotMessage(text) { addMessage(text, 'bot'); }
        function removeMessage(id) { const el = document.getElementById(id); if(el) el.remove(); }

    </script>
</body>
</html>
