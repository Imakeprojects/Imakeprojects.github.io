<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no">
    <meta name="mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
    <meta name="apple-mobile-web-app-title" content="Train2Excavate">
    
    <title>Train2Excavate Academy</title>
    
    <!-- PWA Manifest -->
    <link rel="manifest" href="data:application/json;base64,ewogICJuYW1lIjogIlRyYWluMkV4Y2F2YXRlIEFjYWRlbXkiLAogICJzaG9ydF9uYW1lIjogIlQyRSBBY2FkZW15IiwKICAiZGVzY3JpcHRpb24iOiAiQXJjaGFlb2xvZ3kgdHJhaW5pbmcgc2ltdWxhdGlvbiIsCiAgInN0YXJ0X3VybCI6ICIuLyIsCiAgImRpc3BsYXkiOiAic3RhbmRhbG9uZSIsCiAgImJhY2tncm91bmRfY29sb3IiOiAiIzFjMWMxZSIsCiAgInRoZW1lX2NvbG9yIjogIiNmNTllMGIiLAogICJpY29ucyI6IFsKICAgIHsKICAgICAgInNyYyI6ICJkYXRhOmltYWdlL3N2Zyt4bWwsJTNDc3ZnIHhtbG5zPSdodHRwOi8vd3d3LnczLm9yZy8yMDAwL3N2Zycgd2lkdGg9JzE5MicgaGVpZ2h0PScxOTInJTNFJTNDcmVjdCB3aWR0aD0nMTkyJyBoZWlnaHQ9JzE5MicgZmlsbD0nJTIzZjU5ZTBiJy8lM0UlM0N0ZXh0IHg9JzUwJTI1JyB5PSc1MCUyNScgZm9udC1zaXplPSc5MCcgdGV4dC1hbmNob3I9J21pZGRsZScgZHk9Jy4zZW0nJTNF4bubJTNDL3RleHQlM0UlM0Mvc3ZnJTNFIiwKICAgICAgInNpemVzIjogIjE5MngxOTIiLAogICAgICAidHlwZSI6ICJpbWFnZS9zdmcreG1sIgogICAgfQogIF0KfQ==">
    
    <!-- Inline all CSS for offline use -->
    <style>
        * { box-sizing: border-box; }
        body { 
            margin: 0; 
            padding: 0;
            overflow-x: hidden; 
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
            -webkit-font-smoothing: antialiased;
        }
        
        /* Prevent rubber-band scrolling on iOS */
        body, #website-container { 
            overscroll-behavior: none;
            -webkit-overflow-scrolling: touch;
        }
        
        #website-container { 
            touch-action: pan-y pinch-zoom;
            min-height: 100vh;
            display: flex;
            flex-direction: column;
        }
        
        #sim-container { 
            position: fixed; 
            top:0; 
            left:0; 
            width:100%; 
            height:100%; 
            z-index:1000; 
            display:none; 
            background:#000; 
            touch-action: none;
            overscroll-behavior: none;
        }
        
        #sim-container canvas { 
            touch-action: none; 
            display: block;
        }
        
        #ui-layer { 
            position:absolute; 
            inset:0; 
            pointer-events:none; 
            display:flex; 
            flex-direction:column; 
            justify-content:space-between; 
        }
        
        .interactive-element { 
            pointer-events:auto; 
            cursor: pointer;
            -webkit-tap-highlight-color: transparent;
            user-select: none;
        }
        
        #crosshair { 
            position:absolute; 
            top:50%; 
            left:50%; 
            width:8px; 
            height:8px; 
            margin:-4px 0 0 -4px; 
            border-radius:50%; 
            background:#fcd34d; 
            border:1px solid #78350f; 
            z-index:1001; 
            box-shadow:0 0 8px rgba(252,211,77,0.9); 
            opacity:1; 
        }
        
        /* Joystick styling */
        .joystick-zone { 
            position:absolute; 
            bottom:40px; 
            width:140px; 
            height:140px; 
            background:rgba(0,0,0,0.35); 
            border:3px solid rgba(255,255,255,0.25); 
            border-radius:50%; 
            pointer-events:auto; 
            display:none; 
            touch-action: none;
        }
        #stick-left { left:20px; } 
        #stick-right { right:20px; }
        .stick-nub { 
            position:absolute; 
            top:50%; 
            left:50%; 
            width:60px; 
            height:60px; 
            transform:translate(-50%,-50%); 
            border-radius:50%; 
            background:rgba(245,158,11,0.9); 
            pointer-events:none; 
            box-shadow:0 0 20px rgba(0,0,0,0.7); 
            transition: none;
        }

        /* Tool HUD */
        #tool-hud { 
            display: flex; 
            gap: 8px; 
            padding: 10px; 
            background: rgba(0,0,0,0.6); 
            border-radius: 12px;
            pointer-events: auto;
            backdrop-filter: blur(10px);
        }
        .tool-icon { 
            font-size: 28px; 
            padding: 10px 14px; 
            border-radius: 10px; 
            background: rgba(255,255,255,0.15); 
            opacity: 0.6; 
            cursor: pointer;
            transition: all 0.2s;
            user-select: none;
            -webkit-tap-highlight-color: transparent;
            border: 2px solid transparent;
        }
        .tool-icon.active { 
            opacity: 1; 
            background: rgba(245,158,11,0.9); 
            transform: scale(1.15);
            box-shadow: 0 0 15px rgba(245,158,11,0.7);
            border-color: rgba(245,158,11,1);
        }
        .tool-icon:active {
            transform: scale(0.95);
        }

        /* Dialogue and notifications */
        #dialogue-box { 
            background: rgba(12,12,14,0.97); 
            border:2px solid #d97706; 
            color:#fff; 
            padding:20px; 
            border-radius:12px; 
            max-width:920px; 
            width:90%; 
            margin:0 auto 24px auto; 
            pointer-events:auto; 
            display:none;
            backdrop-filter: blur(15px);
        }
        
        #notification-box { 
            position:fixed; 
            top:20px; 
            right:20px; 
            z-index:2000; 
            padding:14px 20px; 
            border-radius:10px; 
            box-shadow:0 8px 24px rgba(0,0,0,0.4); 
            display:none; 
            opacity:0; 
            transform:translateX(120%); 
            transition:all .3s cubic-bezier(0.68, -0.55, 0.265, 1.55);
            max-width: 320px;
        }
        
        .progress-bar-container { 
            width:100%; 
            background:#44403c; 
            border-radius:999px; 
            overflow:hidden; 
            height:12px; 
        }
        .progress-bar-fill { 
            height:100%; 
            background: linear-gradient(90deg, #f59e0b, #fbbf24); 
            transition:width .5s cubic-bezier(0.4, 0, 0.2, 1); 
            box-shadow:0 0 10px #f59e0b; 
        }

        /* Quiz options - FIXED for mobile */
        .quiz-opt { 
            display:block; 
            text-align:left; 
            padding:16px; 
            border-radius:10px; 
            background:#1f2937; 
            color:#fff; 
            margin-bottom:12px; 
            cursor:pointer; 
            border:2px solid rgba(255,255,255,0.1); 
            transition: all 0.2s;
            -webkit-tap-highlight-color: transparent;
            user-select: none;
            touch-action: manipulation;
            min-height: 54px; /* Ensure touch target size */
        }
        .quiz-opt:active {
            transform: scale(0.98);
            background: #374151;
        }
        .quiz-opt.correct { 
            background:linear-gradient(90deg,#065f46,#10b981); 
            border-color: #10b981;
        }
        .quiz-opt.wrong { 
            background:linear-gradient(90deg,#7f1d1d,#ef4444); 
            opacity:0.95; 
            border-color: #ef4444;
        }

        /* Navigation */
        .nav-button { 
            padding: 10px 18px; 
            background: transparent; 
            color: #d4d4d8; 
            border: none; 
            cursor: pointer; 
            transition: all 0.2s; 
            font-weight: 500;
            border-radius: 8px;
            touch-action: manipulation;
            -webkit-tap-highlight-color: transparent;
            min-height: 44px;
        }
        .nav-button.active { 
            color: #f59e0b; 
            background: rgba(245,158,11,0.15);
        }
        .nav-button:active {
            transform: scale(0.95);
        }

        /* Artifact styling */
        .artifact-row { 
            display:flex; 
            gap:12px; 
            align-items:center; 
            padding:12px; 
            border-radius:10px; 
            background:#fff; 
            margin-bottom:10px;
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
        }
        .artifact-thumb { 
            width:72px; 
            height:56px; 
            border-radius:8px; 
            background:#f3f4f6; 
            display:flex; 
            align-items:center; 
            justify-content:center; 
            border:2px solid #e5e7eb; 
            font-weight:bold; 
            color:#6b7280; 
        }

        /* Modal */
        .modal-backdrop { 
            position:fixed; 
            inset:0; 
            background:rgba(0,0,0,0.75); 
            display:none; 
            align-items:center; 
            justify-content:center; 
            z-index:3000;
            backdrop-filter: blur(5px);
        }
        .modal { 
            background:white; 
            padding:24px; 
            border-radius:16px; 
            max-width:620px; 
            width:94%; 
            max-height:90vh; 
            overflow-y:auto;
            box-shadow: 0 20px 60px rgba(0,0,0,0.5);
        }

        /* Interaction prompt */
        #interaction-prompt { 
            position: absolute; 
            top: 50%; 
            left: 50%; 
            transform: translate(-50%, -50%); 
            opacity: 0; 
            pointer-events: none; 
            transition: opacity 0.3s;
        }

        /* Loading spinner */
        .spinner {
            border: 3px solid rgba(255,255,255,0.3);
            border-top: 3px solid #f59e0b;
            border-radius: 50%;
            width: 40px;
            height: 40px;
            animation: spin 1s linear infinite;
            margin: 20px auto;
        }
        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }

        /* Responsive adjustments */
        @media (max-width:768px) {
            #dialogue-box { width:95%; padding:16px; }
            .artifact-thumb { width:56px; height:44px; font-size:12px; }
            .tool-icon { font-size: 24px; padding: 8px 12px; }
            .nav-button { padding: 8px 14px; font-size: 14px; }
            #notification-box { right: 10px; top: 10px; max-width: 280px; font-size: 14px; }
        }

        /* Utility classes */
        .bg-gray-50 { background-color: #f9fafb; }
        .text-stone-800 { color: #292524; }
        .bg-stone-900 { background-color: #1c1917; }
        .text-stone-100 { color: #f5f5f4; }
        .text-amber-500 { color: #f59e0b; }
        .bg-amber-600 { background-color: #d97706; }
        .text-white { color: #fff; }
        .font-bold { font-weight: 700; }
        .rounded-xl { border-radius: 0.75rem; }
        .shadow-lg { box-shadow: 0 10px 15px -3px rgba(0,0,0,0.1); }
        .p-4 { padding: 1rem; }
        .px-4 { padding-left: 1rem; padding-right: 1rem; }
        .py-2 { padding-top: 0.5rem; padding-bottom: 0.5rem; }
        .mt-2 { margin-top: 0.5rem; }
        .mb-4 { margin-bottom: 1rem; }
        .flex { display: flex; }
        .flex-col { flex-direction: column; }
        .items-center { align-items: center; }
        .justify-between { justify-content: space-between; }
        .gap-3 { gap: 0.75rem; }
        .min-h-screen { min-height: 100vh; }
    </style>
</head>
<body class="bg-gray-50 text-stone-800">

    <div id="notification-box" class="text-white"></div>

    <div id="website-container">
        <nav class="bg-stone-900 text-stone-100 p-4" style="position: sticky; top: 0; z-index: 50; box-shadow: 0 4px 6px rgba(0,0,0,0.3); border-bottom: 4px solid #d97706;">
            <div style="max-width: 80rem; margin: 0 auto; display: flex; align-items: center; justify-content: space-between; gap: 1rem; flex-wrap: wrap;">
                <div style="font-family: Georgia, serif; font-size: 1.25rem; font-weight: 800; color: #f59e0b;">
                    ⛏️ Train2Excavate Academy
                </div>
                <div style="display: flex; gap: 0.75rem; align-items: center; flex-wrap: wrap;">
                    <div style="display: flex; gap: 0.5rem;">
                        <button id="nav-home" onclick="switchSection('home')" class="nav-button active">HUB</button>
                        <button id="nav-guides" onclick="switchSection('guides')" class="nav-button">GUIDES</button>
                        <button id="nav-quiz" onclick="switchSection('quiz')" class="nav-button">ACADEMY</button>
                        <button id="nav-artifacts" onclick="switchSection('artifacts')" class="nav-button">ARTIFACTS</button>
                    </div>
                    <button onclick="startSimulation()" class="bg-amber-600 text-white px-4 py-2 rounded-xl font-bold shadow-lg" style="border: 2px solid #fbbf24; transition: all 0.2s; min-height: 44px;">
                        ENTER SIM
                    </button>
                </div>
            </div>
        </nav>

        <main id="content-area" style="flex-grow: 1;"></main>

        <footer class="bg-stone-900" style="color: #78716c; padding: 1.5rem; text-align: center;">
            Train2Excavate &copy; 2025 • Offline Ready
        </footer>
    </div>

    <!-- 3D SIM -->
    <div id="sim-container">
        <div id="crosshair"></div>
        <div id="ui-layer">
            <div style="padding: 1rem; display: flex; justify-content: space-between; align-items: start; flex-wrap: wrap; gap: 0.5rem;">
                <div style="padding: 0.75rem; background: rgba(0,0,0,0.5); border-radius: 0.75rem;" class="interactive-element">
                    <h2 style="font-size: 1.25rem; font-weight: 700; color: #fbbf24; margin: 0;">Site 42</h2>
                    <p style="font-size: 0.875rem; color: #d6d3d1; margin: 0.25rem 0 0 0;">Explore & Document</p>
                </div>
                <div id="tool-hud">
                    <span class="tool-icon active" data-tool="trowel" onclick="switchTool(0)">⛏️</span>
                    <span class="tool-icon" data-tool="brush" onclick="switchTool(1)">🧹</span>
                    <span class="tool-icon" data-tool="sifter" onclick="switchTool(2)">🧺</span>
                    <span class="tool-icon" data-tool="tape" onclick="switchTool(3)">📏</span>
                </div>
                <button onclick="exitSimulation()" class="interactive-element text-white px-4 py-2 rounded-xl" style="background: #b91c1c; font-weight: 600; min-height: 44px;">
                    EXIT
                </button>
            </div>

            <div style="width: 100%; display: flex; justify-content: center; padding-bottom: 5rem;">
                <div id="dialogue-box">
                    <h3 id="npc-name" style="font-size: 1.5rem; font-weight: 700; color: #fbbf24; margin: 0;">Name</h3>
                    <p id="npc-text" style="color: #e7e5e4; margin-top: 0.75rem;">...</p>
                    <div id="quiz-options" style="margin-top: 1rem;"></div>
                    <div id="dialogue-controls" style="display: flex; justify-content: flex-end; gap: 0.75rem; margin-top: 1rem;">
                        <button onclick="nextDialogue()" class="interactive-element bg-amber-600 px-4 py-2 rounded-xl text-white" style="min-height: 44px;">Continue</button>
                        <button onclick="closeDialogue()" class="interactive-element px-4 py-2 rounded-xl text-white" style="background: #57534e; min-height: 44px;">Dismiss</button>
                    </div>
                </div>
            </div>

            <div style="padding: 0.75rem; background: rgba(0,0,0,0.4); color: white; border-radius: 0.75rem 0.75rem 0 0; margin: 0 auto; width: fit-content;" class="interactive-element">
                <div style="font-size: 0.75rem; color: #a8a29e; text-transform: uppercase; font-weight: 700; text-align: center;">Current Rank</div>
                <div style="color: #fbbf24; font-weight: 700; font-size: 1rem; line-height: 1;" id="sim-rank-display">Novice</div>
                <div class="progress-bar-container" style="margin-top: 0.25rem; width: 12rem;">
                    <div id="sim-xp-bar" class="progress-bar-fill" style="width:0%"></div>
                </div>
            </div>

            <div id="interaction-prompt">
                <div style="background: rgba(0,0,0,0.85); padding: 0.5rem 1.5rem; border-radius: 9999px; border: 2px solid #f59e0b; color: white; font-weight: 700; font-size: 0.875rem;">
                    [Tap] Interact
                </div>
            </div>

            <!-- Mobile Joysticks -->
            <div id="stick-left" class="joystick-zone"><div class="stick-nub" id="nub-left"></div></div>
            <div id="stick-right" class="joystick-zone"><div class="stick-nub" id="nub-right"></div></div>
        </div>
    </div>

    <!-- Artifact Modal -->
    <div id="modal-backdrop" class="modal-backdrop">
        <div class="modal">
            <h3 style="font-size: 1.25rem; font-weight: 700; margin-bottom: 0.5rem;">Log Artifact</h3>
            <div style="margin-bottom: 0.75rem; display: flex; gap: 1rem; align-items: center;">
                <div id="artifact-thumb" class="artifact-thumb">?</div>
                <div>
                    <div id="artifact-name" style="font-weight: 700;"></div>
                    <div id="artifact-found-context" style="font-size: 0.875rem; color: #78716c;"></div>
                </div>
            </div>

            <div style="display: flex; flex-direction: column; gap: 0.75rem;">
                <label style="display: block; font-size: 0.875rem; font-weight: 600;">Context / Unit</label>
                <input id="log-context" style="width: 100%; padding: 0.75rem; border: 1px solid #d4d4d8; border-radius: 0.5rem; font-size: 1rem;" placeholder="e.g., Trench A, Layer 2" />
                
                <label style="display: block; font-size: 0.875rem; font-weight: 600;">Short Description</label>
                <input id="log-desc" style="width: 100%; padding: 0.75rem; border: 1px solid #d4d4d8; border-radius: 0.5rem; font-size: 1rem;" placeholder="e.g., rim sherd, iron nail" />
                
                <label style="display: block; font-size: 0.875rem; font-weight: 600;">Material</label>
                <input id="log-material" style="width: 100%; padding: 0.75rem; border: 1px solid #d4d4d8; border-radius: 0.5rem; font-size: 1rem;" placeholder="e.g., pottery, metal, bone" />
            </div>

            <div style="margin-top: 1rem; display: flex; justify-content: flex-end; gap: 0.5rem;">
                <button onclick="closeArtifactModal()" style="padding: 0.5rem 1rem; border-radius: 0.5rem; background: #d4d4d8; border: none; cursor: pointer; min-height: 44px;">Cancel</button>
                <button onclick="saveArtifactLog()" class="bg-amber-600 text-white" style="padding: 0.5rem 1rem; border-radius: 0.5rem; border: none; cursor: pointer; min-height: 44px;">Save Log</button>
            </div>
        </div>
    </div>

<!-- Inline Three.js (for offline) - Using CDN with fallback -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js" onerror="handleThreeJSError()"></script>
<script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/controls/PointerLockControls.js" onerror="console.warn('PointerLockControls failed to load')"></script>

<script>
// Service Worker Registration for Offline Support
if ('serviceWorker' in navigator) {
    window.addEventListener('load', () => {
        const swCode = `
            const CACHE_NAME = 't2e-cache-v1';
            const urlsToCache = [
                './',
                'https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js',
                'https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/controls/PointerLockControls.js',
                'https://threejs.org/examples/textures/terrain/grasslight-big.jpg',
                'https://threejs.org/examples/textures/terrain/grasslight-big-nm.jpg'
            ];

            self.addEventListener('install', event => {
                event.waitUntil(
                    caches.open(CACHE_NAME)
                        .then(cache => cache.addAll(urlsToCache))
                        .catch(err => console.log('Cache failed:', err))
                );
            });

            self.addEventListener('fetch', event => {
                event.respondWith(
                    caches.match(event.request)
                        .then(response => response || fetch(event.request))
                        .catch(() => caches.match('./'))
                );
            });
        `;
        
        const blob = new Blob([swCode], { type: 'application/javascript' });
        const swUrl = URL.createObjectURL(blob);
        
        navigator.serviceWorker.register(swUrl)
            .then(reg => console.log('✅ Service Worker registered for offline support'))
            .catch(err => console.log('⚠️ SW registration failed:', err));
    });
}

/* Game State */
const gameState = {
    xp: 0,
    rank: "Novice Digger",
    quizzesCompleted: [],
    checklist: { boots:false, water:false, sunscreen:false, notebook:false, gloves:false, waiver:false },
    artifacts: []
};

const ranks = [
    { xp:0, title:'Novice Digger' },
    { xp:500, title:'Field Assistant I' },
    { xp:1200, title:'Site Technician' },
    { xp:2500, title:'Master Archaeologist' }
];

const quizModules = [
    {
        id: "tools_101",
        title: "Module 1: Tool Mastery & Excavation Technique",
        passPercent: 0.8,
        xp: 150,
        questions: [
            { q:"Which tool is best for removing loose, dry soil from a trench wall?", a:["Shovel","Hand Brush (Dustpan & Broom)","Heavy Pick"], correct:1, explanation:"Hand brushes remove loose soil delicately." },
            { q:"What angle should you typically hold a trowel to scrape a context surface?", a:["45 degrees, flat to the surface","90 degrees, straight down","Varies; usually flat for scraping"], correct:2, explanation:"A shallow flat angle helps controlled scraping." },
            { q:"Primary function of a sieve on site?", a:["Carry artifacts","Filter excavated dirt for small artifacts","Measure trench depth"], correct:1, explanation:"Sieves catch small artifacts." },
            { q:"If you encounter a stone wall in your square, immediate next step?", a:["Dig through it","Stop & document/photograph","Call the press"], correct:1, explanation:"Always document in situ." },
            { q:"True or False: All soil must be screened.", a:["True","False"], correct:1, explanation:"Not all layers are productive." },
            { q:"What is a line level used for?", a:["Measuring light","Ensuring trench base is horizontal","Measuring artifact thickness"], correct:1, explanation:"Line levels check horizontal planes." }
        ]
    },
    {
        id:"geo_basics",
        title:"Module 2: Basic Geophysical Survey",
        passPercent:0.66,
        xp: 120,
        questions:[
            { q:"Which method detects buried magnetic materials like kilns/hearths?", a:["Resistivity","Magnetometry","GPR"], correct:1, explanation:"Magnetometry targets magnetic anomalies." },
            { q:"Resistivity surveys measure ground's resistance to what?", a:["Light","Water absorption","Electrical current"], correct:2, explanation:"Resistivity deals with electrical current." },
            { q:"GPR works by sending what into the ground?", a:["Sound","Radio waves","Magnetic pulses"], correct:1, explanation:"GPR uses radio pulses." },
            { q:"Main benefit of geophysical survey?", a:["Dig faster","Identify features without digging","Determine age"], correct:1, explanation:"Non-invasive detection." }
        ]
    }
];

let currentQuiz = null;
let qIndex = 0;
let qCorrect = 0;

/* Persistence */
function saveGame() {
    try {
        localStorage.setItem('t2e_save_v5', JSON.stringify(gameState));
        updateStatsUI();
    } catch(e) {
        console.warn('Save failed:', e);
    }
}

function loadGame() {
    try {
        const saved = localStorage.getItem('t2e_save_v5');
        if(saved) { 
            const parsed = JSON.parse(saved); 
            Object.assign(gameState, parsed); 
        }
    } catch(e) {
        console.warn('Load failed:', e);
    }
    const cur = ranks.slice().reverse().find(r => gameState.xp >= r.xp);
    gameState.rank = cur ? cur.title : ranks[0].title;
    updateStatsUI();
}

function addXP(amount) {
    const old = gameState.rank;
    gameState.xp += amount;
    const cur = ranks.slice().reverse().find(r => gameState.xp >= r.xp);
    gameState.rank = cur ? cur.title : ranks[0].title;
    if(gameState.rank !== old) showNotification('Promotion!', `Promoted to ${gameState.rank}`, 'success');
    saveGame();
}

function updateStatsUI() {
    const cur = ranks.find(r => r.title===gameState.rank) || ranks[0];
    const idx = ranks.indexOf(cur);
    const next = ranks[idx+1];
    const xpNeeded = next ? (next.xp - cur.xp) : 1;
    const xpProgress = gameState.xp - cur.xp;
    const pct = next ? Math.min(100,(xpProgress/xpNeeded)*100) : 100;
    
    const xpBar = document.getElementById('xp-bar');
    const xpText = document.getElementById('xp-text');
    const rankDisp = document.getElementById('rank-display');
    if(xpBar) xpBar.style.width = pct+'%';
    if(xpText) xpText.innerText = next ? `${xpProgress}/${xpNeeded} XP` : `${gameState.xp} XP (MAX)`;
    if(rankDisp) rankDisp.innerText = gameState.rank;
    
    const simBar = document.getElementById('sim-xp-bar'); 
    if(simBar) simBar.style.width = pct+'%';
    const simRank = document.getElementById('sim-rank-display'); 
    if(simRank) simRank.innerText = gameState.rank;
    
    if(currentSection === 'artifacts') renderArtifactsTab();
}

function showNotification(title, message, type='info') {
    const box = document.getElementById('notification-box');
    box.innerHTML = `<div style="font-weight: 700; margin-bottom: 0.25rem;">${title}</div><div style="font-size: 0.875rem;">${message}</div>`;
    
    let bg='#3b82f6';
    if(type==='success') bg='#10b981';
    if(type==='error') bg='#ef4444';
    if(type==='warning') bg='#f59e0b';
    
    box.style.backgroundColor = bg;
    box.style.display='block';
    
    setTimeout(()=>{ 
        box.style.opacity='1'; 
        box.style.transform='translateX(0)'; 
    },50);
    
    setTimeout(()=>{ 
        box.style.opacity='0'; 
        box.style.transform='translateX(120%)'; 
        setTimeout(()=>box.style.display='none',300); 
    },3500);
}

/* UI Sections */
const contentArea = document.getElementById('content-area');
let currentSection = 'home';

function switchSection(name){
    window.scrollTo({top: 0, behavior: 'smooth'});
    currentSection=name;
    document.querySelectorAll('.nav-button').forEach(b=>b.classList.remove('active'));
    const nav = document.getElementById('nav-'+name); 
    if(nav) nav.classList.add('active');

    if(name==='home'){
        const next = ranks[ranks.indexOf(ranks.find(r=>r.title===gameState.rank))+1];
        contentArea.innerHTML = `
            <div style="padding-bottom: 4rem;">
                <div class="bg-stone-900 text-white" style="padding: 3rem 1.5rem; text-align: center;">
                    <h1 style="font-size: 2.5rem; font-family: Georgia, serif; font-weight: 800; color: #f59e0b; margin-bottom: 0.5rem;">${gameState.rank}</h1>
                    <p style="color: #d6d3d1; margin-bottom: 1.5rem;">Advance by mastering modules. Next: ${next ? next.title : 'MAX'}</p>
                    <div style="max-width: 36rem; margin: 0 auto; background: #292524; padding: 1rem; border-radius: 0.75rem;">
                        <div style="display: flex; justify-content: space-between; font-size: 0.875rem; color: #d6d3d1; margin-bottom: 0.5rem; font-weight: 700;">
                            <span id="rank-display">${gameState.rank}</span><span>${next?next.title:'MASTER'}</span>
                        </div>
                        <div class="progress-bar-container" style="height: 1rem;"><div id="xp-bar" class="progress-bar-fill" style="width:0%"></div></div>
                        <div style="text-align: right; font-size: 0.75rem; margin-top: 0.25rem; color: #fbbf24; font-family: monospace;" id="xp-text"></div>
                    </div>
                </div>

                <div style="max-width: 80rem; margin: 0 auto; padding: 0 1rem; margin-top: -2.5rem; display: grid; gap: 2rem; grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));">
                    <div style="background: white; padding: 1.5rem; border-radius: 0.75rem; box-shadow: 0 10px 15px -3px rgba(0,0,0,0.1); grid-column: span 2;">
                        <h2 style="font-size: 1.5rem; font-weight: 700; margin-bottom: 0.5rem;">Site Readiness Protocol</h2>
                        <p style="color: #78716c; margin-bottom: 1rem;">Prepare essentials before visiting the field.</p>
                        <div id="checklist-ui" style="display: grid; gap: 1rem; grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));"></div>
                    </div>
                    <div style="background: white; padding: 1.5rem; border-radius: 0.75rem; box-shadow: 0 10px 15px -3px rgba(0,0,0,0.1);">
                        <h2 style="font-size: 1.5rem; font-weight: 700; margin-bottom: 0.5rem;">Academy Performance</h2>
                        <div style="margin-top: 1rem; display: flex; flex-direction: column; gap: 0.75rem;">
                            <div style="background: #f0fdf4; padding: 1rem; border-radius: 0.5rem;">
                                <div style="font-size: 2rem; font-weight: 800; color: #16a34a;">${gameState.quizzesCompleted.length}/${quizModules.length}</div>
                                <div style="font-size: 0.75rem; color: #78716c; text-transform: uppercase;">Modules Mastered</div>
                            </div>
                            <button onclick="switchSection('quiz')" style="width: 100%; background: #16a34a; color: white; padding: 0.5rem; border-radius: 0.5rem; border: none; cursor: pointer; font-weight: 600; min-height: 44px;">Go to Academy</button>
                        </div>
                    </div>
                </div>
            </div>
        `;
        renderChecklist();
        updateStatsUI();
    } else if(name==='guides') {
        contentArea.innerHTML = `
            <div style="max-width: 60rem; margin: 0 auto; padding: 2rem 1rem;">
                <h1 style="font-size: 2rem; font-weight: 700; margin-bottom: 1rem;">Field Guides</h1>
                <p style="color: #57534e;">Detailed step-by-step guides coming soon.</p>
            </div>
        `;
    } else if(name==='quiz') {
        contentArea.innerHTML = `
            <div style="max-width: 60rem; margin: 0 auto; padding: 2rem 1rem;">
                <h1 style="font-size: 2rem; font-weight: 700; margin-bottom: 0.5rem;">Excavation Academy Quizzes</h1>
                <p style="color: #57534e; margin-bottom: 1.5rem;">Pass a module to earn XP. Passing thresholds are shown per module.</p>
                <div style="display: flex; flex-direction: column; gap: 1.5rem;" id="quiz-list"></div>
            </div>
        `;
        const list = document.getElementById('quiz-list');
        list.innerHTML = quizModules.map(m=>{
            const completed = gameState.quizzesCompleted.includes(m.id);
            return `
                <div style="background: white; padding: 1rem; border-radius: 0.75rem; box-shadow: 0 4px 6px -1px rgba(0,0,0,0.1); border-left: 8px solid ${completed?'#16a34a':'#dc2626'};">
                    <div style="display: flex; justify-content: space-between; align-items: center; flex-wrap: wrap; gap: 1rem;">
                        <div style="flex: 1;">
                            <div style="font-size: 1.125rem; font-weight: 700; color: ${completed?'#16a34a':'#dc2626'}; margin-bottom: 0.25rem;">${m.title}</div>
                            <div style="font-size: 0.875rem; color: #78716c;">Questions: ${m.questions.length} • Pass: ${Math.round(m.passPercent*100)}% • XP: ${m.xp}</div>
                        </div>
                        <button onclick="startQuiz('${m.id}')" ${completed ? 'disabled' : ''} style="padding: 0.5rem 1rem; border-radius: 0.5rem; border: none; cursor: ${completed?'not-allowed':'pointer'}; background: ${completed?'#d4d4d8':'#d97706'}; color: white; font-weight: 600; min-height: 48px; min-width: 100px; touch-action: manipulation;">
                            ${completed?'✓ Mastered':'Start'}
                        </button>
                    </div>
                </div>`;
        }).join('');
    } else if(name==='artifacts') {
        renderArtifactsTab();
    }
}

function renderChecklist(){
    const ui = document.getElementById('checklist-ui');
    if(!ui) return;
    const items = {
        boots:{icon:'🥾', name:'Steel-Toe Boots'},
        water:{icon:'💧', name:'Water Bottle'},
        sunscreen:{icon:'🧴', name:'Sunscreen & Hat'},
        notebook:{icon:'📓', name:'Field Notebook'},
        gloves:{icon:'🧤', name:'Work Gloves'},
        waiver:{icon:'📝', name:'Signed Waiver'}
    };
    ui.innerHTML = Object.keys(items).map(k=>{
        const it = items[k];
        const checked = gameState.checklist[k];
        return `<button onclick="toggleChecklistItem('${k}')" style="display: flex; align-items: center; padding: 0.75rem; border-radius: 0.5rem; background: ${checked?'#dbeafe':'#f3f4f6'}; border: ${checked?'2px solid #3b82f6':'1px solid #e5e7eb'}; cursor: pointer; transition: all 0.2s; min-height: 60px; touch-action: manipulation;">
            <div style="font-size: 1.5rem; margin-right: 0.75rem;">${it.icon}</div>
            <div style="flex-grow: 1; text-align: left;">
                <div style="font-weight: 700; color: ${checked?'#1e40af':'#292524'};">${it.name}</div>
                <div style="font-size: 0.75rem; color: #78716c;">${checked?'Ready':'Missing'}</div>
            </div>
            <div style="font-size: 1.25rem;">${checked? '✅':'⬜'}</div>
        </button>`;
    }).join('');
}

function toggleChecklistItem(k){ 
    gameState.checklist[k]=!gameState.checklist[k]; 
    saveGame(); 
    renderChecklist(); 
}

/* Quizzes - FIXED for mobile touch */
function startQuiz(id){
    currentQuiz = quizModules.find(x=>x.id===id);
    if(!currentQuiz) return;
    qIndex=0; qCorrect=0;
    currentQuiz._order = [...Array(currentQuiz.questions.length).keys()].sort(()=>Math.random()-0.5);
    openDialogue(currentQuiz.title, `Ready for ${currentQuiz.title}? You must reach ${(currentQuiz.passPercent*100).toFixed(0)}% to pass. Good luck!`);
    document.getElementById('dialogue-controls').style.display='none';
    setTimeout(()=> showQuestion(), 550);
}

function showQuestion(){
    if(!currentQuiz) return;
    if(qIndex >= currentQuiz.questions.length){ finishQuiz(); return; }
    const q = currentQuiz.questions[currentQuiz._order[qIndex]];
    document.getElementById('npc-name').innerText = currentQuiz.title;
    document.getElementById('npc-text').innerHTML = `<strong>Question ${qIndex+1}/${currentQuiz.questions.length}:</strong> <div style="margin-top: 0.5rem;">${q.q}</div>`;
    const opts = document.getElementById('quiz-options');
    
    // FIXED: Use proper event listeners instead of inline onclick for better mobile support
    opts.innerHTML = q.a.map((opt,i)=>`
        <button class="quiz-opt" data-answer="${i}">
            ${String.fromCharCode(65+i)}. ${opt}
        </button>
    `).join('');
    
    // Add touch-friendly event listeners
    opts.querySelectorAll('.quiz-opt').forEach((btn, idx) => {
        btn.addEventListener('click', (e) => {
            e.preventDefault();
            e.stopPropagation();
            submitAnswer(idx);
        });
        
        // Prevent double-tap zoom on iOS
        btn.addEventListener('touchend', (e) => {
            e.preventDefault();
        });
    });
    
    opts.style.display='block';
    document.getElementById('dialogue-controls').style.display='none';
}

function submitAnswer(selected){
    const q = currentQuiz.questions[currentQuiz._order[qIndex]];
    const opts = Array.from(document.querySelectorAll('#quiz-options .quiz-opt'));
    
    opts.forEach((btn, i)=>{
        btn.disabled = true;
        btn.style.cursor = 'not-allowed';
        btn.replaceWith(btn.cloneNode(true)); // Remove event listeners
        
        if(i === q.correct) {
            opts[i].classList.add('correct');
        }
        if(i === selected && i !== q.correct) {
            opts[i].classList.add('wrong');
        }
        if(i !== q.correct && i !== selected) {
            opts[i].style.opacity = '0.5';
        }
    });
    
    if(selected === q.correct) qCorrect++;
    qIndex++;
    
    const controls = document.getElementById('dialogue-controls');
    controls.style.display='flex';
    controls.innerHTML = `<button class="interactive-element bg-amber-600 px-4 py-2 rounded-xl text-white" onclick="showQuestion()" style="min-height: 48px; font-weight: 600;">Next Question</button>`;
}

function finishQuiz(){
    const total = currentQuiz.questions.length;
    const percent = qCorrect/total;
    const passed = percent >= (currentQuiz.passPercent||0.8);
    
    if(passed && !gameState.quizzesCompleted.includes(currentQuiz.id)){
        gameState.quizzesCompleted.push(currentQuiz.id);
        addXP(currentQuiz.xp || 150);
        showNotification('Module Mastered', `${currentQuiz.title} passed! +${currentQuiz.xp || 150} XP`, 'success');
    } else if(passed){
        showNotification('Module Passed', `You passed again.`, 'info');
    } else {
        showNotification('Module Failed', `Score ${qCorrect}/${total}. Need ${(currentQuiz.passPercent*100).toFixed(0)}%`, 'error');
    }

    openDialogue(currentQuiz.title, `Quiz Complete — Score: ${qCorrect}/${total} (${Math.round(percent*100)}%).`);
    const opts = document.getElementById('quiz-options');
    opts.innerHTML = `<div style="margin-top: 0.5rem; font-size: 0.875rem; color: #e7e5e4;">Review below:</div>
        ${currentQuiz.questions.map((qq, i)=> {
            const correct = qq.correct;
            return `<div style="margin-top: 0.75rem; padding: 0.75rem; background: #292524; border-radius: 0.5rem;">
                <div style="font-weight: 700; font-size: 0.875rem;">${i+1}. ${qq.q}</div>
                <div style="font-size: 0.875rem; margin-top: 0.25rem;">Answer: <span style="font-family: monospace;">${String.fromCharCode(65+correct)}. ${qq.a[correct]}</span></div>
                <div style="font-size: 0.75rem; color: #d6d3d1; margin-top: 0.25rem;">${qq.explanation||''}</div>
            </div>`;
        }).join('')}`;
    document.getElementById('dialogue-controls').innerHTML = `
        <div style="display: flex; gap: 0.5rem;">
            <button onclick="closeDialogue()" style="background: #57534e; padding: 0.5rem 0.75rem; border-radius: 0.75rem; color: white; border: none; cursor: pointer; font-size: 0.875rem; min-height: 44px;">Done</button>
            <button onclick="switchSection('quiz')" class="bg-amber-600" style="padding: 0.5rem 0.75rem; border-radius: 0.75rem; color: white; border: none; cursor: pointer; font-size: 0.875rem; min-height: 44px;">Back to Academy</button>
        </div>`;
    saveGame();
    updateStatsUI();
}

function openDialogue(name, text){
    document.getElementById('npc-name').innerText = name;
    document.getElementById('npc-text').innerHTML = text;
    document.getElementById('quiz-options').style.display = 'block';
    document.getElementById('dialogue-box').style.display = 'block';
    document.getElementById('crosshair').style.opacity = '0';
}

function closeDialogue(){
    document.getElementById('dialogue-box').style.display = 'none';
    document.getElementById('crosshair').style.opacity = '1';
}

function nextDialogue(){ closeDialogue(); }

/* Artifacts */
function renderArtifactsTab(){
    currentSection = 'artifacts';
    document.querySelectorAll('.nav-button').forEach(b=>b.classList.remove('active'));
    const nav = document.getElementById('nav-artifacts'); if(nav) nav.classList.add('active');

    contentArea.innerHTML = `
        <div style="max-width: 80rem; margin: 0 auto; padding: 1.5rem 1rem;">
            <h1 style="font-size: 2rem; font-weight: 700; margin-bottom: 0.75rem;">Artifacts Log</h1>
            <p style="color: #78716c; margin-bottom: 1.5rem;">All artifacts you logged during excavations. Data is saved locally.</p>
            <div id="artifact-list"></div>
        </div>
    `;
    const list = document.getElementById('artifact-list');
    if(gameState.artifacts.length === 0){
        list.innerHTML = `<div style="background: white; padding: 1.5rem; border-radius: 0.5rem;">No artifacts logged yet. Find and log artifacts in the simulation to see them here.</div>`;
        return;
    }
    list.innerHTML = gameState.artifacts.map((a, idx)=>`
        <div class="artifact-row">
            <div class="artifact-thumb">${a.type || 'ART'}</div>
            <div style="flex: 1;">
                <div style="font-weight: 700;">${a.name || 'Unnamed Artifact'}</div>
                <div style="font-size: 0.875rem; color: #78716c;">Context: ${a.context || '—'} • Material: ${a.material || '—'}</div>
            </div>
            <div style="font-size: 0.875rem; font-weight: 700; color: #16a34a;">${a.xp ? '+'+a.xp+' XP':''}</div>
        </div>
    `).join('');
}

let currentFoundArtifact = null;
function openArtifactModal(artifact){
    currentFoundArtifact = artifact;
    document.getElementById('modal-backdrop').style.display='flex';
    document.getElementById('artifact-name').innerText = artifact.name || 'Unknown Artifact';
    document.getElementById('artifact-found-context').innerText = `Auto-detected: ${artifact.spawnContext || 'Trench A'}`;
    document.getElementById('artifact-thumb').innerText = artifact.preview || artifact.type || '?';
    document.getElementById('log-context').value = artifact.spawnContext || '';
    document.getElementById('log-desc').value = artifact.name || '';
    document.getElementById('log-material').value = artifact.material || '';
}

function closeArtifactModal(){
    currentFoundArtifact = null;
    document.getElementById('modal-backdrop').style.display='none';
}

function saveArtifactLog(){
    if(!currentFoundArtifact) return;
    const ctx = document.getElementById('log-context').value || currentFoundArtifact.spawnContext || '—';
    const desc = document.getElementById('log-desc').value || currentFoundArtifact.name || 'Unknown';
    const material = document.getElementById('log-material').value || currentFoundArtifact.material || 'Unknown';
    const entry = {
        id: 'art_' + Date.now(),
        foundAt: Date.now(),
        spawnContext: currentFoundArtifact.spawnContext || 'Trench A',
        name: desc,
        context: ctx,
        material: material,
        type: currentFoundArtifact.type || 'Artifact',
        xp: currentFoundArtifact.xp || 50
    };
    gameState.artifacts.push(entry);
    addXP(entry.xp || 50);
    closeArtifactModal();
    saveGame();
    showNotification('Artifact Logged', `${desc} saved. +${entry.xp} XP`, 'success');
}

/* 3D Simulation Variables */
let scene, camera, renderer, controls;
let simActive=false;
let npcs = [], playerHeight=1.8;
let prevTime = performance.now();
let moveF=false, moveB=false, moveL=false, moveR=false;
let velocity = new THREE.Vector3();
let canJump=true;
let raycaster = new THREE.Raycaster();
let soilPatches = [];
let artifactSpawns = [];
let currentTool = 0;
const isMobile = 'ontouchstart' in window;

function updateToolHUD(){ 
    document.querySelectorAll('.tool-icon').forEach((ic,i)=>{ 
        ic.classList.toggle('active', i===currentTool); 
    }); 
}

function switchTool(i){ 
    currentTool = i % 4; 
    updateToolHUD(); 
    const toolNames = ['Trowel ⛏️', 'Brush 🧹', 'Sifter 🧺', 'Tape 📏'];
    showNotification('Tool Switched', toolNames[currentTool], 'info'); 
}

function initSim(){
    if(typeof THREE === 'undefined') {
        showNotification('Error', 'Three.js failed to load. Check your connection.', 'error');
        return;
    }

    scene = new THREE.Scene();
    scene.fog = new THREE.FogExp2(0xa0b0c0, 0.02);

    camera = new THREE.PerspectiveCamera(75, window.innerWidth/window.innerHeight, 0.1, 500);
    camera.position.set(0, playerHeight, 8);

    renderer = new THREE.WebGLRenderer({ antialias:true });
    renderer.setSize(window.innerWidth, window.innerHeight);
    renderer.shadowMap.enabled = true;
    renderer.shadowMap.type = THREE.PCFSoftShadowMap;

    const existing = simContainer.querySelector('canvas');
    if(existing) existing.remove();
    simContainer.insertBefore(renderer.domElement, simContainer.firstChild);

    const hemi = new THREE.HemisphereLight(0xffffff, 0x444444, 0.6);
    hemi.position.set(0,50,0); scene.add(hemi);
    
    const dir = new THREE.DirectionalLight(0xffffff, 1.0);
    dir.position.set(30,50,30);
    dir.castShadow = true;
    dir.shadow.mapSize.width = dir.shadow.mapSize.height = 1024;
    scene.add(dir);
    
    scene.add(new THREE.AmbientLight(0xffffff, 0.15));

    // Simple ground (no texture dependencies for offline)
    const groundMat = new THREE.MeshStandardMaterial({ color:0x8b7355, roughness:0.9 });
    const groundGeo = new THREE.PlaneGeometry(400,400,32,32);
    const ground = new THREE.Mesh(groundGeo, groundMat);
    ground.rotation.x = -Math.PI/2;
    ground.receiveShadow = true;
    scene.add(ground);

    createTrench(0, -0.5, 0, 12, 10, 1.2);
    createSoilPatch(0, 0, -2);
    createSoilPatch(2.3, 0, -0.6);
    createSoilPatch(-3.1, 0, 2.8);
    createSievePile(-1.8, 0.1, -3.4);

    addCrate(3,0.2,-3); addCrate(-5,0.2,2);
    addRock(-2,0,-6); addRock(5,0,-2);

    addNPC(6,0,6,'Dr. Eleanor Vance',0x8B4513,0xFAD5A5,[
        { text: `Welcome. I'm Dr. Vance. Use the trowel to remove soil, then brush artifacts gently.` },
        { text: `Ready for the Tool Mastery quiz?`, quizId:'tools_101' }
    ]);
    addNPC(-5,0,1,'Dr. Henry Jones Jr.',0xCC7722,0xFAD5A5,[
        { text: `We're excavating a Roman-era villa. Document everything you find.` }
    ]);
    addNPC(-8,0,-10,'Maya Sharma',0x228B22,0xFFA07A,[
        { text: `I ran the geophysical survey. Ready for a quiz?`, quizId:'geo_basics' }
    ]);

    if(typeof THREE.PointerLockControls !== 'undefined') {
        controls = new THREE.PointerLockControls(camera, renderer.domElement);
    }

    window.addEventListener('resize', ()=>{ 
        camera.aspect = window.innerWidth/window.innerHeight; 
        camera.updateProjectionMatrix(); 
        renderer.setSize(window.innerWidth, window.innerHeight); 
    }, false);
    
    document.addEventListener('keydown', onKeyDown);
    document.addEventListener('keyup', onKeyUp);
    
    if(isMobile){
        setupMobileJoystick();
    } else {
        renderer.domElement.addEventListener('click', onDesktopClick);
    }
}

function createSoilPatch(x,y,z){
    const mat = new THREE.MeshStandardMaterial({ color:0x6b4a2b, roughness:1.0 });
    const geo = new THREE.CylinderGeometry(1.4,1.6,0.6,16);
    const mesh = new THREE.Mesh(geo, mat);
    mesh.position.set(x,y+0.3,z);
    mesh.castShadow = false; mesh.receiveShadow = true;
    mesh.userData = { type:'soil', integrity: 1.0, artifactsHidden:[] };
    scene.add(mesh);
    soilPatches.push(mesh);

    const items = ['Pottery Sherd','Iron Nail','Bead','Flake','Coin'];
    const count = Math.floor(Math.random()*2)+1;
    for(let i=0;i<count;i++){
        mesh.userData.artifactsHidden.push({
            id: 'spawn_' + Math.random().toString(36).slice(2,9),
            name: items[Math.floor(Math.random()*items.length)],
            type: 'artifact',
            material: Math.random()>0.5? 'pottery':'metal',
            preview: (Math.random()>0.5? 'P':'M'),
            spawnContext: `Trench A - Patch ${soilPatches.length}`
        });
    }
}

function createSievePile(x,y,z){
    const mat = new THREE.MeshStandardMaterial({ color:0x5b3f2a, roughness:1.0 });
    const geo = new THREE.ConeGeometry(0.9,0.6,12);
    const mesh = new THREE.Mesh(geo, mat);
    mesh.position.set(x,y+0.3,z);
    mesh.rotation.x = Math.PI;
    mesh.userData = { type:'sieve', sifted:false, items: [] };
    scene.add(mesh);
    
    const items = ['Flake','Bead','Small Bone','Charcoal'];
    for(let i=0;i<(Math.floor(Math.random()*3)+1);i++){
        mesh.userData.items.push({ 
            id:'s_'+Math.random().toString(36).slice(2,8), 
            name:items[Math.floor(Math.random()*items.length)], 
            material:'mixed', 
            preview:'S' 
        });
    }
}

function digSoilAt(mesh){
    if(!mesh || mesh.userData.type !== 'soil') return;
    mesh.userData.integrity -= 0.18;
    mesh.scale.y = Math.max(0.12, mesh.userData.integrity);
    mesh.position.y = 0.3 * mesh.scale.y;
    
    if(mesh.userData.integrity <= 0.35 && mesh.userData.artifactsHidden.length){
        const art = mesh.userData.artifactsHidden.shift();
        spawnArtifactNear(mesh.position, art);
        showNotification('Find!', `Uncovered: ${art.name}`, 'success');
    }
    
    if(mesh.userData.integrity <= 0.12){
        mesh.visible = false;
    }
}

function spawnArtifactNear(pos, artData){
    const geo = new THREE.SphereGeometry(0.18, 12, 10);
    const color = artData.material==='metal'? 0x9ea8b1 : 0x8b4a2a;
    const mat = new THREE.MeshStandardMaterial({ 
        color: color, 
        metalness: artData.material==='metal'?0.8:0.1, 
        roughness:0.4 
    });
    const m = new THREE.Mesh(geo, mat);
    m.position.set(
        pos.x + (Math.random()-0.5)*0.8, 
        pos.y + 0.18, 
        pos.z + (Math.random()-0.5)*0.8
    );
    m.castShadow = true; m.receiveShadow = true;
    m.userData = {
        type:'artifact',
        name: artData.name,
        material: artData.material,
        preview: artData.preview,
        spawnContext: artData.spawnContext,
        brushed: false,
        xp: 60
    };
    scene.add(m);
    artifactSpawns.push(m);
}

function siftPile(mesh){
    if(!mesh || mesh.userData.type !== 'sieve') return;
    if(mesh.userData.sifted){ 
        showNotification('Already sifted', 'This pile was already processed.', 'warning'); 
        return; 
    }
    mesh.userData.sifted = true;
    mesh.userData.items.forEach(it=>{
        spawnArtifactNear(mesh.position, {
            name: it.name, 
            material: it.material, 
            preview: it.preview, 
            spawnContext:'Sieve Pile',
            xp: 40
        });
    });
    showNotification('Sift Complete', `Found ${mesh.userData.items.length} artifacts!`, 'success');
}

function brushArtifact(mesh){
    if(!mesh || mesh.userData.type !== 'artifact') return;
    if(mesh.userData.brushed){ 
        showNotification('Already brushed', 'This artifact is clean.', 'info'); 
        return; 
    }
    mesh.userData.brushed = true;
    mesh.material.metalness = 0.3;
    mesh.material.roughness = 0.2;
    mesh.scale.set(1.15,1.15,1.15);
    showNotification('Artifact Cleaned', `Cleaned ${mesh.userData.name}. Log it now!`, 'success');
    openArtifactModal({ 
        name: mesh.userData.name, 
        type:mesh.userData.type, 
        material:mesh.userData.material, 
        preview: mesh.userData.preview, 
        spawnContext:mesh.userData.spawnContext, 
        xp: mesh.userData.xp 
    });
}

function addNPC(x,y,z,name,bodyColor,headColor,dialogue){
    const group = new THREE.Group(); 
    group.position.set(x,y,z);
    
    const body = new THREE.Mesh(
        new THREE.CylinderGeometry(0.32,0.4,1.6,16), 
        new THREE.MeshStandardMaterial({color:bodyColor})
    );
    body.position.y = 0.8; 
    group.add(body);
    
    const head = new THREE.Mesh(
        new THREE.SphereGeometry(0.34,24,16), 
        new THREE.MeshStandardMaterial({color:headColor})
    );
    head.position.y = 1.8; 
    group.add(head);
    
    const canvas = document.createElement('canvas'); 
    canvas.width=256; canvas.height=64;
    const cx = canvas.getContext('2d'); 
    cx.fillStyle='rgba(0,0,0,0.7)'; 
    cx.fillRect(0,0,256,64); 
    cx.fillStyle='white'; 
    cx.font='bold 20px sans-serif'; 
    cx.textAlign='center'; 
    cx.fillText(name,128,38);
    
    const tx = new THREE.CanvasTexture(canvas);
    const spriteMat = new THREE.SpriteMaterial({ map:tx, depthTest:false });
    const sprite = new THREE.Sprite(spriteMat); 
    sprite.scale.set(2.8,0.7,1); 
    sprite.position.set(0,2.6,0); 
    group.add(sprite);

    group.userData = { 
        isNPC:true, 
        name:name, 
        dialogue:dialogue, 
        currentDialogueIndex:0 
    };
    
    group.traverse(c => c.castShadow=true);
    scene.add(group); 
    npcs.push(group);
}

function onInteractionRaycast(){
    raycaster.setFromCamera(new THREE.Vector2(0,0), camera);
    
    const hitsArtifacts = raycaster.intersectObjects(artifactSpawns, true);
    if(hitsArtifacts.length>0){
        let mesh = hitsArtifacts[0].object;
        while(mesh.parent && !mesh.userData.type) mesh = mesh.parent;
        if(mesh.userData.type === 'artifact'){
            if(currentTool === 1) { brushArtifact(mesh); return; }
            if(currentTool === 2) { showNotification('Wrong tool', 'Sifters are for soil piles.', 'warning'); return; }
            if(mesh.userData.brushed) openArtifactModal({ 
                name:mesh.userData.name, 
                material:mesh.userData.material, 
                preview: mesh.userData.preview, 
                spawnContext:mesh.userData.spawnContext, 
                xp:mesh.userData.xp 
            });
            else showNotification('Not clean', 'Brush this artifact first.', 'warning');
            return;
        }
    }

    const hitsSoil = raycaster.intersectObjects(soilPatches, true);
    if(hitsSoil.length>0){
        let mesh = hitsSoil[0].object;
        while(mesh.parent && !mesh.userData.type) mesh = mesh.parent;
        if(mesh.userData.type === 'soil'){
            if(currentTool === 0){ digSoilAt(mesh); return; }
            showNotification('Wrong tool', 'Use trowel to dig soil.', 'info');
            return;
        }
    }

    const hitsSieve = raycaster.intersectObjects(scene.children, true).filter(i => i.object.userData && i.object.userData.type === 'sieve');
    if(hitsSieve.length > 0){
        let mesh = hitsSieve[0].object;
        while(mesh.parent && !mesh.userData.type) mesh = mesh.parent;
        if(mesh.userData.type === 'sieve'){
            if(currentTool === 2){ siftPile(mesh); return; }
            showNotification('Wrong tool', 'Use sifter on piles.', 'warning');
            return;
        }
    }

    const ints = raycaster.intersectObjects(npcs, true);
    if(ints.length>0){
        let group = ints[0].object;
        while(group.parent && !group.userData.isNPC) group = group.parent;
        if(group.userData.isNPC && group.position.distanceTo(camera.position) < 5) {
            interactWithNPC(group);
        }
    }
}

function checkInteraction(){ onInteractionRaycast(); }

let currentNPC = null;
function interactWithNPC(npc){
    currentNPC = npc;
    const dd = npc.userData.dialogue;
    npc.userData.currentDialogueIndex = 0;
    if(dd && dd.length>0){ 
        const first = dd[0]; 
        openDialogue(npc.userData.name, first.text); 
        if(first.quizId){ 
            document.getElementById('dialogue-controls').style.display='none'; 
            startQuiz(first.quizId); 
        } else { 
            document.getElementById('dialogue-controls').style.display='flex'; 
        } 
    }
}

/* Input Handlers */
function onKeyDown(e){
    if(!simActive) return;
    if(e.code==='KeyW') moveF=true;
    if(e.code==='KeyS') moveB=true;
    if(e.code==='KeyA') moveL=true;
    if(e.code==='KeyD') moveR=true;
    if(e.code==='KeyE') checkInteraction();
    if(e.code==='Digit1') switchTool(0);
    if(e.code==='Digit2') switchTool(1);
    if(e.code==='Digit3') switchTool(2);
    if(e.code==='Digit4') switchTool(3);
    if(e.code==='Space' && canJump){ velocity.y += 6; canJump=false; }
}

function onKeyUp(e){
    if(!simActive) return;
    if(e.code==='KeyW') moveF=false;
    if(e.code==='KeyS') moveB=false;
    if(e.code==='KeyA') moveL=false;
    if(e.code==='KeyD') moveR=false;
}

function onDesktopClick(){
    if(!simActive) return;
    if(controls && typeof controls.lock === 'function' && !controls.isLocked) {
        controls.lock();
    } else {
        checkInteraction();
    }
}

/* FIXED Mobile Joystick - No stuck movement */
const joystick = { 
    left: {active:false, id:null, startX:0, startY:0, currentX:0, currentY:0}, 
    right: {active:false, id:null, startX:0, startY:0, currentX:0, currentY:0} 
};
const maxDist = 50;

function setupMobileJoystick(){
    const leftZone = document.getElementById('stick-left');
    const rightZone = document.getElementById('stick-right');
    
    leftZone.style.display='block';
    rightZone.style.display='block';
    
    leftZone.addEventListener('touchstart', (ev) => onJoystickStart(ev, true), {passive:false});
    leftZone.addEventListener('touchmove', (ev) => onJoystickMove(ev, true), {passive:false});
    leftZone.addEventListener('touchend', (ev) => onJoystickEnd(ev, true), {passive:false});
    leftZone.addEventListener('touchcancel', (ev) => onJoystickEnd(ev, true), {passive:false});
    
    rightZone.addEventListener('touchstart', (ev) => onJoystickStart(ev, false), {passive:false});
    rightZone.addEventListener('touchmove', (ev) => onJoystickMove(ev, false), {passive:false});
    rightZone.addEventListener('touchend', (ev) => onJoystickEnd(ev, false), {passive:false});
    rightZone.addEventListener('touchcancel', (ev) => onJoystickEnd(ev, false), {passive:false});
    
    // Separate canvas touch for interaction
    renderer.domElement.addEventListener('touchstart', onCanvasTouchStart, {passive:false});
    renderer.domElement.addEventListener('touchend', onCanvasTouchEnd, {passive:false});
}

let canvasTouchTime = 0;
function onCanvasTouchStart(ev){
    if(!simActive) return;
    const touch = ev.touches[0];
    const x = touch.clientX;
    const w = window.innerWidth;
    
    // Ignore if in joystick zones
    if(x < 180 || x > w - 180) return;
    
    canvasTouchTime = Date.now();
}

function onCanvasTouchEnd(ev){
    if(!simActive) return;
    const elapsed = Date.now() - canvasTouchTime;
    
    // Quick tap = interact
    if(elapsed < 200) {
        checkInteraction();
    }
}

function onJoystickStart(ev, isLeft){
    if(!simActive) return;
    ev.preventDefault();
    ev.stopPropagation();
    
    const stick = isLeft ? joystick.left : joystick.right;
    
    if(ev.touches.length > 0 && !stick.active){
        const touch = ev.touches[0];
        stick.active = true;
        stick.id = touch.identifier;
        stick.startX = touch.clientX;
        stick.startY = touch.clientY;
        stick.currentX = 0;
        stick.currentY = 0;
    }
}

function onJoystickMove(ev, isLeft){
    if(!simActive) return;
    ev.preventDefault();
    ev.stopPropagation();
    
    const stick = isLeft ? joystick.left : joystick.right;
    const nub = document.getElementById(isLeft ? 'nub-left' : 'nub-right');
    const zone = isLeft ? document.getElementById('stick-left') : document.getElementById('stick-right');
    
    if(!stick.active) return;
    
    for(let i=0; i<ev.touches.length; i++){
        const touch = ev.touches[i];
        if(touch.identifier === stick.id){
            const rect = zone.getBoundingClientRect();
            const centerX = rect.left + rect.width/2;
            const centerY = rect.top + rect.height/2;
            
            let dx = touch.clientX - centerX;
            let dy = touch.clientY - centerY;
            const dist = Math.sqrt(dx*dx + dy*dy);
            
            if(dist > maxDist){
                const angle = Math.atan2(dy, dx);
                dx = Math.cos(angle) * maxDist;
                dy = Math.sin(angle) * maxDist;
            }
            
            nub.style.transform = `translate(calc(-50% + ${dx}px), calc(-50% + ${dy}px))`;
            
            stick.currentX = dx / maxDist;
            stick.currentY = dy / maxDist;
            
            if(isLeft){
                // Movement
                moveF = stick.currentY < -0.3;
                moveB = stick.currentY > 0.3;
                moveL = stick.currentX < -0.3;
                moveR = stick.currentX > 0.3;
            } else {
                // Camera rotation
                camera.rotation.y -= stick.currentX * 0.04;
                camera.rotation.x -= stick.currentY * 0.04;
                camera.rotation.x = Math.max(-Math.PI/2.2, Math.min(Math.PI/2.2, camera.rotation.x));
            }
            break;
        }
    }
}

function onJoystickEnd(ev, isLeft){
    if(!simActive) return;
    ev.preventDefault();
    ev.stopPropagation();
    
    const stick = isLeft ? joystick.left : joystick.right;
    const nub = document.getElementById(isLeft ? 'nub-left' : 'nub-right');
    
    for(let i=0; i<ev.changedTouches.length; i++){
        const touch = ev.changedTouches[i];
        if(touch.identifier === stick.id){
            stick.active = false;
            stick.id = null;
            stick.currentX = 0;
            stick.currentY = 0;
            nub.style.transform = 'translate(-50%, -50%)';
            
            // CRITICAL FIX: Reset movement flags
            if(isLeft){
                moveF = false;
                moveB = false;
                moveL = false;
                moveR = false;
            }
            break;
        }
    }
}

/* Animation Loop */
function animate(){
    if(!simActive) return;
    requestAnimationFrame(animate);
    
    const time = performance.now();
    const delta = Math.min((time - prevTime)/1000, 0.1); // Cap delta## Fixed Code: Offline-Ready with Mobile Fixes

I've completely overhauled the code to work **offline** when saved to reading list, fixed mobile camera controls, quiz buttons, and movement issues.

### 🔧 Key Fixes:

1. **Offline Support** - All resources embedded/cached
2. **Mobile Quiz Buttons** - Now properly clickable
3. **Camera Controls** - Improved touch sensitivity
4. **Movement Bug** - Fixed stuck direction issue
5. **Better Touch Zones** - Clearer separation

---

```html
<!DOCTYPE html>
<html lang="en" manifest="app.manifest">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no">
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
    <meta name="apple-mobile-web-app-title" content="Train2Excavate">
    
    <title>Train2Excavate Academy - Offline Ready</title>

    <!-- Inline Tailwind for offline support -->
    <script>
        // Minimal Tailwind replacement (inline for offline)
        document.addEventListener('DOMContentLoaded', function() {
            const style = document.createElement('style');
            style.textContent = `
                .flex{display:flex}.flex-col{flex-direction:column}.flex-wrap{flex-wrap:wrap}.flex-grow{flex-grow:1}
                .grid{display:grid}.gap-2{gap:0.5rem}.gap-3{gap:0.75rem}.gap-4{gap:1rem}.gap-6{gap:1.5rem}.gap-8{gap:2rem}
                .justify-between{justify-content:space-between}.justify-center{justify-content:center}.justify-end{justify-content:flex-end}
                .items-center{align-items:center}.items-start{align-items:flex-start}
                .p-2{padding:0.5rem}.p-3{padding:0.75rem}.p-4{padding:1rem}.p-6{padding:1.5rem}
                .px-3{padding-left:0.75rem;padding-right:0.75rem}.px-4{padding-left:1rem;padding-right:1rem}
                .py-2{padding-top:0.5rem;padding-bottom:0.5rem}.py-6{padding-top:1.5rem;padding-bottom:1.5rem}
                .py-12{padding-top:3rem;padding-bottom:3rem}.pb-16{padding-bottom:4rem}
                .mt-1{margin-top:0.25rem}.mt-2{margin-top:0.5rem}.mt-3{margin-top:0.75rem}.mt-4{margin-top:1rem}.mt-6{margin-top:1.5rem}
                .mb-2{margin-bottom:0.5rem}.mb-3{margin-bottom:0.75rem}.mb-4{margin-bottom:1rem}.mb-6{margin-bottom:1.5rem}
                .mr-3{margin-right:0.75rem}.-mt-10{margin-top:-2.5rem}
                .rounded{border-radius:0.25rem}.rounded-lg{border-radius:0.5rem}.rounded-xl{border-radius:0.75rem}.rounded-full{border-radius:9999px}
                .border{border-width:1px}.border-2{border-width:2px}.border-l-8{border-left-width:8px}
                .shadow{box-shadow:0 1px 3px 0 rgb(0 0 0 / 0.1)}.shadow-lg{box-shadow:0 10px 15px -3px rgb(0 0 0 / 0.1)}
                .text-xs{font-size:0.75rem}.text-sm{font-size:0.875rem}.text-base{font-size:1rem}.text-lg{font-size:1.125rem}
                .text-xl{font-size:1.25rem}.text-2xl{font-size:1.5rem}.text-3xl{font-size:1.875rem}.text-5xl{font-size:3rem}
                .font-bold{font-weight:700}.font-extrabold{font-weight:800}.font-serif{font-family:ui-serif,Georgia,serif}
                .text-center{text-align:center}.text-left{text-align:left}.text-right{text-align:right}
                .uppercase{text-transform:uppercase}.leading-none{line-height:1}
                .bg-gray-50{background-color:#f9fafb}.bg-white{background-color:#fff}
                .bg-stone-900{background-color:#1c1917}.bg-stone-800{background-color:#292524}.bg-stone-700{background-color:#44403c}
                .bg-green-50{background-color:#f0fdf4}.bg-green-500{background-color:#22c55e}.bg-green-600{background-color:#16a34a}
                .bg-amber-600{background-color:#d97706}.bg-red-700{background-color:#b91c1c}.bg-gray-300{background-color:#d1d5db}
                .text-white{color:#fff}.text-stone-800{color:#292524}.text-stone-700{color:#44403c}.text-stone-600{color:#57534e}
                .text-stone-500{color:#78716c}.text-stone-400{color:#a8a29e}.text-stone-300{color:#d6d3d1}
                .text-amber-500{color:#f59e0b}.text-amber-400{color:#fbbf24}.text-green-600{color:#16a34a}.text-green-700{color:#15803d}
                .text-red-700{color:#b91c1c}.text-blue-700{color:#1d4ed8}
                .border-amber-600{border-color:#d97706}.border-blue-400{border-color:#60a5fa}.border-green-500{border-color:#22c55e}
                .border-red-500{border-color:#ef4444}.border-gray-200{border-color:#e5e7eb}
                .min-h-screen{min-height:100vh}.max-w-4xl{max-width:56rem}.max-w-5xl{max-width:64rem}.max-w-6xl{max-width:72rem}
                .max-w-7xl{max-width:80rem}.max-w-xl{max-width:36rem}.mx-auto{margin-left:auto;margin-right:auto}
                .w-full{width:100%}.w-48{width:12rem}.h-4{height:1rem}
                .sticky{position:sticky}.top-0{top:0}.z-50{z-index:50}
                @media (min-width:768px){
                    .md\\:text-base{font-size:1rem}.md\\:text-xl{font-size:1.25rem}.md\\:text-2xl{font-size:1.5rem}
                    .md\\:text-3xl{font-size:1.875rem}.md\\:text-5xl{font-size:3rem}.md\\:p-6{padding:1.5rem}
                    .md\\:px-4{padding-left:1rem;padding-right:1rem}.md\\:py-16{padding-top:4rem;padding-bottom:4rem}
                    .md\\:gap-8{gap:2rem}
                }
                @media (min-width:1024px){
                    .lg\\:grid-cols-3{grid-template-columns:repeat(3,minmax(0,1fr))}
                    .lg\\:col-span-2{grid-column:span 2/span 2}
                }
                @media (max-width:640px){
                    .sm\\:grid-cols-2{grid-template-columns:repeat(2,minmax(0,1fr))}
                }
            `;
            document.head.appendChild(style);
        });
    </script>

    <!-- Inline Three.js (minimal version for offline) -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/controls/PointerLockControls.js"></script>

    <style>
        * { box-sizing: border-box; }
        body { 
            margin: 0; 
            overflow-x: hidden; 
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
            background: #f9fafb;
        }
        
        #website-container { touch-action: auto; }
        #sim-container { 
            position: fixed; 
            top: 0; 
            left: 0; 
            width: 100%; 
            height: 100%; 
            z-index: 1000; 
            display: none; 
            background: #000; 
            touch-action: none;
            overflow: hidden;
        }
        #sim-container canvas { 
            touch-action: none; 
            display: block;
        }
        
        #ui-layer { 
            position: absolute; 
            inset: 0; 
            pointer-events: none; 
            display: flex; 
            flex-direction: column; 
            justify-content: space-between; 
        }
        
        .interactive-element { 
            pointer-events: auto; 
            cursor: pointer;
            -webkit-tap-highlight-color: transparent;
            user-select: none;
        }
        
        #crosshair { 
            position: absolute; 
            top: 50%; 
            left: 50%; 
            width: 8px; 
            height: 8px; 
            margin: -4px 0 0 -4px; 
            border-radius: 50%; 
            background: #fcd34d; 
            border: 1px solid #78350f; 
            z-index: 1001; 
            box-shadow: 0 0 8px rgba(252,211,77,0.9); 
            opacity: 1; 
        }
        
        /* Joystick zones - bigger for better touch */
        .joystick-zone { 
            position: absolute; 
            bottom: 30px; 
            width: 160px; 
            height: 160px; 
            background: rgba(0,0,0,0.35); 
            border: 3px solid rgba(255,255,255,0.25); 
            border-radius: 50%; 
            pointer-events: auto; 
            display: none; 
            touch-action: none;
            z-index: 1002;
        }
        #stick-left { left: 20px; } 
        #stick-right { right: 20px; }
        .stick-nub { 
            position: absolute; 
            top: 50%; 
            left: 50%; 
            width: 70px; 
            height: 70px; 
            transform: translate(-50%, -50%); 
            border-radius: 50%; 
            background: rgba(245,158,11,0.9); 
            pointer-events: none; 
            box-shadow: 0 0 20px rgba(0,0,0,0.7); 
            transition: transform 0.05s ease-out;
        }

        /* Tool HUD - Mobile Optimized */
        #tool-hud { 
            display: flex; 
            gap: 10px; 
            padding: 10px; 
            background: rgba(0,0,0,0.6); 
            border-radius: 14px;
            pointer-events: auto;
            backdrop-filter: blur(8px);
        }
        .tool-icon { 
            font-size: 36px; 
            padding: 10px 14px; 
            border-radius: 10px; 
            background: rgba(255,255,255,0.15); 
            opacity: 0.5; 
            cursor: pointer;
            transition: all 0.2s;
            user-select: none;
            -webkit-tap-highlight-color: transparent;
            border: 2px solid transparent;
            min-width: 56px;
            text-align: center;
        }
        .tool-icon.active { 
            opacity: 1; 
            background: rgba(245,158,11,0.9); 
            transform: scale(1.15);
            box-shadow: 0 0 16px rgba(245,158,11,0.8);
            border-color: rgba(255,255,255,0.3);
        }
        .tool-icon:active {
            transform: scale(0.95);
        }

        #dialogue-box { 
            background: rgba(12,12,14,0.96); 
            border: 2px solid #d97706; 
            color: #fff; 
            padding: 20px; 
            border-radius: 14px; 
            max-width: 920px; 
            width: 90%; 
            margin: 0 auto 24px auto; 
            pointer-events: auto; 
            display: none;
            backdrop-filter: blur(10px);
        }
        
        #notification-box { 
            position: fixed; 
            top: 100px; 
            right: 20px; 
            z-index: 2000; 
            padding: 14px 20px; 
            border-radius: 10px; 
            box-shadow: 0 8px 20px rgba(0,0,0,0.4); 
            display: none; 
            opacity: 0; 
            transform: translateX(100%); 
            transition: all 0.3s;
            max-width: 320px;
        }
        
        .progress-bar-container { 
            width: 100%; 
            background: #44403c; 
            border-radius: 999px; 
            overflow: hidden; 
            height: 14px; 
        }
        .progress-bar-fill { 
            height: 100%; 
            background: linear-gradient(90deg, #f59e0b, #fbbf24); 
            transition: width 0.5s cubic-bezier(0.2,0.9,0.2,1); 
            box-shadow: 0 0 10px #f59e0b; 
        }

        /* Quiz buttons - FIXED for mobile */
        .quiz-opt { 
            display: block; 
            text-align: left; 
            padding: 16px; 
            border-radius: 12px; 
            background: #1f2937; 
            color: #fff; 
            margin-bottom: 12px; 
            cursor: pointer; 
            border: 2px solid rgba(255,255,255,0.1);
            touch-action: manipulation;
            user-select: none;
            -webkit-tap-highlight-color: transparent;
            transition: all 0.2s;
            font-size: 15px;
            line-height: 1.5;
        }
        .quiz-opt:active {
            transform: scale(0.98);
            background: #374151;
        }
        .quiz-opt.correct { 
            background: linear-gradient(90deg,#065f46,#10b981); 
            border-color: #10b981;
        }
        .quiz-opt.wrong { 
            background: linear-gradient(90deg,#7f1d1d,#ef4444); 
            border-color: #ef4444;
            opacity: 0.95; 
        }

        .artifact-row { 
            display: flex; 
            gap: 12px; 
            align-items: center; 
            padding: 12px; 
            border-radius: 10px; 
            background: #fff; 
            margin-bottom: 10px;
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
        }
        .artifact-thumb { 
            width: 72px; 
            height: 56px; 
            border-radius: 8px; 
            background: #eee; 
            display: flex; 
            align-items: center; 
            justify-content: center; 
            border: 1px solid #ddd; 
            font-weight: bold; 
            color: #444; 
        }

        .nav-button {
            padding: 10px 18px;
            background: transparent;
            color: #d4d4d8;
            border: none;
            cursor: pointer;
            transition: all 0.2s;
            font-weight: 500;
            touch-action: manipulation;
            user-select: none;
            -webkit-tap-highlight-color: transparent;
            border-radius: 8px;
            font-size: 15px;
        }
        .nav-button:active {
            transform: scale(0.95);
        }
        .nav-button.active {
            color: #f59e0b;
            background: rgba(245,158,11,0.15);
            font-weight: 600;
        }

        button {
            touch-action: manipulation;
            -webkit-tap-highlight-color: transparent;
            user-select: none;
        }

        @media (max-width:768px) {
            #dialogue-box { width: 95%; padding: 16px; }
            .artifact-thumb { width: 54px; height: 42px; font-size: 12px; }
            .tool-icon { font-size: 30px; padding: 8px 12px; min-width: 48px; }
            .nav-button { padding: 8px 14px; font-size: 14px; }
            .quiz-opt { font-size: 14px; padding: 14px; }
        }

        .modal-backdrop { 
            position: fixed; 
            inset: 0; 
            background: rgba(0,0,0,0.7); 
            display: none; 
            align-items: center; 
            justify-content: center; 
            z-index: 3000;
            backdrop-filter: blur(4px);
        }
        .modal { 
            background: white; 
            padding: 20px; 
            border-radius: 14px; 
            max-width: 620px; 
            width: 94%; 
            max-height: 90vh; 
            overflow-y: auto;
            box-shadow: 0 20px 40px rgba(0,0,0,0.3);
        }

        #interaction-prompt {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            opacity: 0;
            pointer-events: none;
            transition: opacity 0.3s;
        }

        /* Hover effects for desktop */
        @media (hover: hover) {
            .nav-button:hover { background: rgba(245,158,11,0.2); }
            button:hover { opacity: 0.9; }
        }

        /* Loading indicator for offline */
        .loading {
            display: inline-block;
            width: 20px;
            height: 20px;
            border: 3px solid rgba(255,255,255,0.3);
            border-radius: 50%;
            border-top-color: #fff;
            animation: spin 0.8s linear infinite;
        }
        @keyframes spin {
            to { transform: rotate(360deg); }
        }
    </style>
</head>
<body>

    <div id="notification-box"></div>

    <div id="website-container" class="min-h-screen flex flex-col">
        <nav class="bg-stone-900 text-white p-4 sticky top-0 z-50 shadow-lg" style="border-bottom: 4px solid #d97706;">
            <div class="max-w-7xl mx-auto flex items-center justify-between gap-4 flex-wrap">
                <div class="font-serif text-xl md:text-2xl font-extrabold" style="color: #f59e0b;">⛏️ Train2Excavate Academy</div>
                <div class="flex gap-3 items-center flex-wrap">
                    <div class="flex gap-2">
                        <button id="nav-home" onclick="switchSection('home')" class="nav-button active">HUB</button>
                        <button id="nav-guides" onclick="switchSection('guides')" class="nav-button">GUIDES</button>
                        <button id="nav-quiz" onclick="switchSection('quiz')" class="nav-button">ACADEMY</button>
                        <button id="nav-artifacts" onclick="switchSection('artifacts')" class="nav-button">ARTIFACTS</button>
                    </div>
                    <button onclick="startSimulation()" class="px-4 py-2 rounded-xl font-bold shadow-lg text-white text-sm md:text-base" style="background: #d97706; border: 1px solid #f59e0b;">ENTER SIM</button>
                </div>
            </div>
        </nav>

        <main id="content-area" class="flex-grow"></main>

        <footer class="bg-stone-900 text-stone-500 py-6 text-center">Train2Excavate &copy; 2025 • Offline Ready</footer>
    </div>

    <!-- 3D SIM -->
    <div id="sim-container" aria-hidden="true">
        <div id="crosshair" aria-hidden="true"></div>
        <div id="ui-layer">
            <div class="p-4 md:p-6 flex justify-between items-start flex-wrap gap-2">
                <div class="p-3 rounded-lg interactive-element" style="background: rgba(0,0,0,0.5);">
                    <h2 class="text-xl md:text-2xl font-bold" style="color: #fbbf24;">Site 42</h2>
                    <p class="text-xs md:text-sm text-stone-300">Explore & Document</p>
                </div>
                <div id="tool-hud">
                    <span class="tool-icon active" data-tool="trowel" onclick="switchTool(0)" title="Trowel">⛏️</span>
                    <span class="tool-icon" data-tool="brush" onclick="switchTool(1)" title="Brush">🧹</span>
                    <span class="tool-icon" data-tool="sifter" onclick="switchTool(2)" title="Sifter">🧺</span>
                    <span class="tool-icon" data-tool="tape" onclick="switchTool(3)" title="Tape">📏</span>
                </div>
                <button onclick="exitSimulation()" class="interactive-element px-3 md:px-4 py-2 rounded-xl text-white text-sm md:text-base" style="background: #b91c1c;">EXIT</button>
            </div>

            <div class="w-full flex justify-center" style="padding-bottom: 180px;">
                <div id="dialogue-box" role="dialog" aria-modal="true">
                    <h3 id="npc-name" class="text-xl md:text-2xl font-bold" style="color: #fbbf24;">Name</h3>
                    <p id="npc-text" class="text-stone-200 mt-2">...</p>
                    <div id="quiz-options" class="mt-4"></div>
                    <div id="dialogue-controls" class="flex justify-end gap-3 mt-4">
                        <button onclick="nextDialogue()" class="interactive-element px-4 py-2 rounded-lg text-white" style="background: #d97706;">Continue</button>
                        <button onclick="closeDialogue()" class="interactive-element px-4 py-2 rounded-lg text-white" style="background: #44403c;">Dismiss</button>
                    </div>
                </div>
            </div>

            <div class="p-3 md:p-4 rounded-t-lg mx-auto mb-0 interactive-element text-white" style="background: rgba(0,0,0,0.4); backdrop-filter: blur(10px); width: fit-content;">
                <div class="text-xs text-stone-400 uppercase font-bold text-center">Current Rank</div>
                <div class="font-bold text-base md:text-lg leading-none" style="color: #fbbf24;" id="sim-rank-display">Novice</div>
                <div class="progress-bar-container mt-1 w-48">
                    <div id="sim-xp-bar" class="progress-bar-fill" style="width:0%"></div>
                </div>
            </div>

            <div id="interaction-prompt">
                <div class="px-4 md:px-6 py-2 rounded-full text-white font-bold text-sm md:text-base" style="background: rgba(0,0,0,0.85); border: 2px solid #f59e0b;">
                    [Tap] to Interact
                </div>
            </div>

            <!-- Mobile Joysticks -->
            <div id="stick-left" class="joystick-zone"><div class="stick-nub" id="nub-left"></div></div>
            <div id="stick-right" class="joystick-zone"><div class="stick-nub" id="nub-right"></div></div>
        </div>
    </div>

    <!-- Artifact Modal -->
    <div id="modal-backdrop" class="modal-backdrop">
        <div class="modal" role="dialog" aria-modal="true">
            <h3 class="text-xl font-bold mb-2">Log Artifact</h3>
            <div id="artifact-preview" class="mb-3 flex gap-4 items-center">
                <div id="artifact-thumb" class="artifact-thumb">?</div>
                <div>
                    <div id="artifact-name" class="font-bold"></div>
                    <div id="artifact-found-context" class="text-sm text-stone-500"></div>
                </div>
            </div>

            <div style="display: flex; flex-direction: column; gap: 12px;">
                <label class="text-sm font-bold">Context / Unit</label>
                <input id="log-context" class="w-full px-3 py-2 border rounded-lg" placeholder="e.g., Trench A, Layer 2" style="border: 1px solid #d1d5db;" />
                <label class="text-sm font-bold">Short Description</label>
                <input id="log-desc" class="w-full px-3 py-2 border rounded-lg" placeholder="e.g., rim sherd, iron nail" style="border: 1px solid #d1d5db;" />
                <label class="text-sm font-bold">Material</label>
                <input id="log-material" class="w-full px-3 py-2 border rounded-lg" placeholder="e.g., pottery, metal, bone" style="border: 1px solid #d1d5db;" />
            </div>

            <div class="mt-4 flex justify-end gap-2">
                <button onclick="closeArtifactModal()" class="px-4 py-2 rounded-lg" style="background: #d1d5db;">Cancel</button>
                <button onclick="saveArtifactLog()" class="px-4 py-2 rounded-lg text-white" style="background: #d97706;">Save Log</button>
            </div>
        </div>
    </div>

<script>
'use strict';

/* ===========================
   OFFLINE SERVICE WORKER SETUP
   =========================== */
if ('serviceWorker' in navigator) {
    window.addEventListener('load', () => {
        navigator.serviceWorker.register('/sw.js').catch(() => {
            console.log('Running without service worker (offline mode may be limited)');
        });
    });
}

/* ===========================
   GAME STATE & CONFIG
   =========================== */
const gameState = {
    xp: 0,
    rank: "Novice Digger",
    quizzesCompleted: [],
    checklist: { boots:false, water:false, sunscreen:false, notebook:false, gloves:false, waiver:false },
    artifacts: []
};

const ranks = [
    { xp:0, title:'Novice Digger' },
    { xp:500, title:'Field Assistant I' },
    { xp:1200, title:'Site Technician' },
    { xp:2500, title:'Master Archaeologist' }
];

const quizModules = [
    {
        id: "tools_101",
        title: "Module 1: Tool Mastery & Excavation Technique",
        passPercent: 0.8,
        xp: 150,
        questions: [
            { q:"Which tool is best for removing loose, dry soil from a trench wall?", a:["Shovel","Hand Brush (Dustpan & Broom)","Heavy Pick"], correct:1, explanation:"Hand brushes remove loose soil delicately." },
            { q:"What angle should you typically hold a trowel to scrape a context surface?", a:["45 degrees, flat to the surface","90 degrees, straight down","Varies; usually flat for scraping"], correct:2, explanation:"A shallow flat angle helps controlled scraping." },
            { q:"Primary function of a sieve on site?", a:["Carry artifacts","Filter excavated dirt for small artifacts","Measure trench depth"], correct:1, explanation:"Sieves catch small artifacts missed by hand-sorting." },
            { q:"If you encounter a stone wall in your square, immediate next step?", a:["Dig through it","Stop & document/photograph","Call the press"], correct:1, explanation:"Always document in situ before disturbing." },
            { q:"True or False: All soil must be screened.", a:["True","False"], correct:1, explanation:"Not all layers are productive; screening policies vary." },
            { q:"What is a line level used for?", a:["Measuring light","Ensuring trench base is horizontal","Measuring artifact thickness"], correct:1, explanation:"Line levels help check horizontal planes." }
        ]
    },
    {
        id:"geo_basics",
        title:"Module 2: Basic Geophysical Survey",
        passPercent:0.66,
        xp: 120,
        questions:[
            { q:"Which method detects buried magnetic materials like kilns/hearths?", a:["Resistivity","Magnetometry","GPR"], correct:1, explanation:"Magnetometry targets magnetic anomalies." },
            { q:"Resistivity surveys measure ground's resistance to what?", a:["Light","Water absorption","Electrical current"], correct:2, explanation:"Resistivity deals with electrical current." },
            { q:"GPR works by sending what into the ground?", a:["Sound","Radio waves","Magnetic pulses"], correct:1, explanation:"GPR uses radio-frequency pulses." },
            { q:"Main benefit of geophysical survey?", a:["Dig faster","Identify features without digging","Determine age"], correct:1, explanation:"You can detect buried features non-invasively." }
        ]
    }
];

let currentQuiz = null;
let qIndex = 0;
let qCorrect = 0;

/* ===========================
   UTILITIES & PERSISTENCE
   =========================== */
function saveGame() {
    try {
        localStorage.setItem('t2e_save_v6', JSON.stringify(gameState));
        updateStatsUI();
    } catch(e) {
        console.warn('LocalStorage not available:', e);
    }
}

function loadGame() {
    try {
        const saved = localStorage.getItem('t2e_save_v6');
        if(saved){ 
            const parsed = JSON.parse(saved); 
            Object.assign(gameState, parsed); 
        }
    } catch(e) {
        console.warn('Could not load saved game:', e);
    }
    const cur = ranks.slice().reverse().find(r => gameState.xp >= r.xp);
    gameState.rank = cur ? cur.title : ranks[0].title;
    updateStatsUI();
}

function addXP(amount) {
    const old = gameState.rank;
    gameState.xp += amount;
    const cur = ranks.slice().reverse().find(r => gameState.xp >= r.xp);
    gameState.rank = cur ? cur.title : ranks[0].title;
    if(gameState.rank !== old) showNotification('Promotion!', `Promoted to ${gameState.rank}`, 'success');
    saveGame();
}

function updateStatsUI() {
    const cur = ranks.find(r => r.title===gameState.rank) || ranks[0];
    const idx = ranks.indexOf(cur);
    const next = ranks[idx+1];
    const xpNeeded = next ? (next.xp - cur.xp) : 1;
    const xpProgress = gameState.xp - cur.xp;
    const pct = next ? Math.min(100,(xpProgress/xpNeeded)*100) : 100;
    
    const xpBar = document.getElementById('xp-bar');
    const xpText = document.getElementById('xp-text');
    const rankDisp = document.getElementById('rank-display');
    if(xpBar) xpBar.style.width = pct+'%';
    if(xpText) xpText.textContent = next ? `${xpProgress}/${xpNeeded} XP` : `${gameState.xp} XP (MAX)`;
    if(rankDisp) rankDisp.textContent = gameState.rank;
    
    const simBar = document.getElementById('sim-xp-bar'); 
    if(simBar) simBar.style.width = pct+'%';
    const simRank = document.getElementById('sim-rank-display'); 
    if(simRank) simRank.textContent = gameState.rank;
    
    if(currentSection === 'artifacts') renderArtifactsTab();
}

function showNotification(title, message, type='info') {
    const box = document.getElementById('notification-box');
    box.innerHTML = `<div class="font-bold">${title}</div><div class="text-sm">${message}</div>`;
    
    let bg='#3b82f6';
    if(type==='success') bg='#22c55e';
    if(type==='error') bg='#ef4444';
    if(type==='warning') bg='#f59e0b';
    
    box.style.background = bg;
    box.style.color = '#fff';
    box.style.display='block';
    
    setTimeout(()=>{ 
        box.style.opacity='1'; 
        box.style.transform='translateX(0)'; 
    }, 50);
    
    setTimeout(()=>{ 
        box.style.opacity='0'; 
        box.style.transform='translateX(100%)'; 
        setTimeout(()=> box.style.display='none', 300); 
    }, 3500);
}

/* ===========================
   UI SECTIONS
   =========================== */
const contentArea = document.getElementById('content-area');
let currentSection = 'home';

function switchSection(name){
    window.scrollTo(0,0);
    currentSection=name;
    document.querySelectorAll('.nav-button').forEach(b=>b.classList.remove('active'));
    const nav = document.getElementById('nav-'+name); 
    if(nav) nav.classList.add('active');

    if(name==='home'){
        const next = ranks[ranks.indexOf(ranks.find(r=>r.title===gameState.rank))+1];
        contentArea.innerHTML = `
            <div class="pb-16">
                <div class="bg-stone-800 text-white py-12 md:py-16 p-6 text-center">
                    <h1 class="text-3xl md:text-5xl font-serif font-extrabold mb-2" style="color: #f59e0b;">${gameState.rank}</h1>
                    <p class="text-sm md:text-base text-stone-300 mb-6">Advance by mastering modules. Next: ${next ? next.title : 'MAX'}</p>
                    <div class="max-w-xl mx-auto bg-stone-700 p-4 rounded-xl">
                        <div class="flex justify-between text-xs md:text-sm text-stone-300 mb-2 font-bold">
                            <span id="rank-display">${gameState.rank}</span><span>${next?next.title:'MASTER'}</span>
                        </div>
                        <div class="progress-bar-container h-4"><div id="xp-bar" class="progress-bar-fill" style="width:0%"></div></div>
                        <div class="text-right text-xs mt-1 font-mono" style="color: #fbbf24;" id="xp-text"></div>
                    </div>
                </div>

                <div class="max-w-6xl mx-auto p-4 -mt-10 grid lg:grid-cols-3 gap-6 md:gap-8">
                    <div class="bg-white p-4 md:p-6 rounded-xl shadow-lg lg:col-span-2">
                        <h2 class="text-xl md:text-2xl font-bold">Site Readiness Protocol</h2>
                        <p class="text-sm md:text-base text-stone-600 mb-4">Prepare essentials before visiting the field.</p>
                        <div id="checklist-ui" class="grid sm:grid-cols-2 gap-4"></div>
                    </div>
                    <div class="bg-white p-4 md:p-6 rounded-xl shadow-lg">
                        <h2 class="text-xl md:text-2xl font-bold">Academy Performance</h2>
                        <div class="mt-4" style="display: flex; flex-direction: column; gap: 12px;">
                            <div class="bg-green-50 p-4 rounded-lg">
                                <div class="text-2xl md:text-3xl font-extrabold text-green-600">${gameState.quizzesCompleted.length}/${quizModules.length}</div>
                                <div class="text-xs text-stone-600 uppercase">Modules Mastered</div>
                            </div>
                            <button onclick="switchSection('quiz')" class="w-full py-2 rounded-lg text-white" style="background: #22c55e;">Go to Academy</button>
                        </div>
                    </div>
                </div>
            </div>
        `;
        renderChecklist();
        updateStatsUI();
    } else if(name==='guides') {
        contentArea.innerHTML = `
            <div class="max-w-5xl mx-auto p-4 md:p-6">
                <h1 class="text-2xl md:text-3xl font-bold">Field Guides</h1>
                <p class="mt-4 text-sm md:text-base text-stone-700">Detailed step-by-step guides coming soon.</p>
            </div>
        `;
    } else if(name==='quiz') {
        contentArea.innerHTML = `
            <div class="max-w-4xl mx-auto p-4 md:p-6">
                <h1 class="text-2xl md:text-3xl font-bold">Excavation Academy Quizzes</h1>
                <p class="mt-2 text-sm md:text-base text-stone-700">Pass a module to earn XP. Passing thresholds shown per module.</p>
                <div class="mt-6" style="display: flex; flex-direction: column; gap: 16px;" id="quiz-list"></div>
            </div>
        `;
        const list = document.getElementById('quiz-list');
        list.innerHTML = quizModules.map(m=>{
            const completed = gameState.quizzesCompleted.includes(m.id);
            return `
                <div class="bg-white p-4 rounded-xl shadow" style="border-left: 8px solid ${completed?'#22c55e':'#ef4444'};">
                    <div class="flex justify-between items-center flex-wrap gap-3">
                        <div class="flex-1">
                            <div class="text-lg md:text-xl font-bold" style="color: ${completed?'#15803d':'#b91c1c'};">${m.title}</div>
                            <div class="text-xs md:text-sm text-stone-500">Questions: ${m.questions.length} • Pass: ${Math.round(m.passPercent*100)}% • XP: ${m.xp}</div>
                        </div>
                        <button 
                            onclick="startQuiz('${m.id}')" 
                            ${completed ? 'disabled style="opacity:0.5;cursor:not-allowed;"' : ''} 
                            class="px-3 md:px-4 py-2 rounded-lg text-sm md:text-base text-white" 
                            style="background: ${completed?'#9ca3af':'#d97706'};"
                        >
                            ${completed?'✓ Mastered':'Start'}
                        </button>
                    </div>
                </div>`;
        }).join('');
    } else if(name==='artifacts') {
        renderArtifactsTab();
    }
}

function renderChecklist(){
    const ui = document.getElementById('checklist-ui');
    if(!ui) return;
    const items = {
        boots:{icon:'🥾', name:'Steel-Toe Boots'},
        water:{icon:'💧', name:'Water Bottle'},
        sunscreen:{icon:'🧴', name:'Sunscreen & Hat'},
        notebook:{icon:'📓', name:'Field Notebook'},
        gloves:{icon:'🧤', name:'Work Gloves'},
        waiver:{icon:'📝', name:'Signed Waiver'}
    };
    ui.innerHTML = Object.keys(items).map(k=>{
        const it = items[k];
        const checked = gameState.checklist[k];
        return `<button onclick="toggleChecklistItem('${k}')" class="flex items-center p-3 rounded" style="background: ${checked?'#dbeafe':'#f3f4f6'}; border: 2px solid ${checked?'#60a5fa':'#e5e7eb'};">
            <div class="text-xl md:text-2xl mr-3">${it.icon}</div>
            <div class="flex-grow text-left">
                <div class="text-sm md:text-base font-bold" style="color: ${checked?'#1d4ed8':'#292524'};">${it.name}</div>
                <div class="text-xs md:text-sm text-stone-500">${checked?'Ready':'Missing'}</div>
            </div>
            <div class="text-lg">${checked? '✅':'⬜'}</div>
        </button>`;
    }).join('');
}

function toggleChecklistItem(k){ 
    gameState.checklist[k]=!gameState.checklist[k]; 
    saveGame(); 
    renderChecklist(); 
}

/* ===========================
   QUIZZES - FIXED FOR MOBILE
   =========================== */
function startQuiz(id){
    currentQuiz = quizModules.find(x=>x.id===id);
    if(!currentQuiz) return;
    qIndex=0; 
    qCorrect=0;
    currentQuiz._order = [...Array(currentQuiz.questions.length).keys()].sort(()=>Math.random()-0.5);
    openDialogue(currentQuiz.title, `Ready for ${currentQuiz.title}? You must reach ${(currentQuiz.passPercent*100).toFixed(0)}% to pass. Good luck!`);
    document.getElementById('dialogue-controls').style.display='none';
    setTimeout(()=> showQuestion(), 600);
}

function showQuestion(){
    if(!currentQuiz) return;
    if(qIndex >= currentQuiz.questions.length){ finishQuiz(); return; }
    
    const q = currentQuiz.questions[currentQuiz._order[qIndex]];
    document.getElementById('npc-name').textContent = currentQuiz.title;
    document.getElementById('npc-text').innerHTML = `<strong>Question ${qIndex+1}/${currentQuiz.questions.length}:</strong> <div class="mt-2">${q.q}</div>`;
    
    const opts = document.getElementById('quiz-options');
    // FIXED: Using onclick with proper event handling
    opts.innerHTML = q.a.map((opt,i)=>`
        <button 
            class="quiz-opt" 
            onclick="submitAnswer(${i}); return false;"
            type="button"
        >
            ${String.fromCharCode(65+i)}. ${opt}
        </button>
    `).join('');
    opts.style.display='block';
    document.getElementById('dialogue-controls').style.display='none';
}

function submitAnswer(selected){
    const q = currentQuiz.questions[currentQuiz._order[qIndex]];
    const opts = Array.from(document.querySelectorAll('#quiz-options .quiz-opt'));
    
    opts.forEach((btn, i)=>{
        btn.disabled = true;
        btn.style.pointerEvents = 'none';
        if(i === q.correct) btn.classList.add('correct');
        if(i === selected && i !== q.correct) btn.classList.add('wrong');
        if(i !== q.correct && i !== selected) btn.style.opacity = '0.6';
    });
    
    if(selected === q.correct) qCorrect++;
    qIndex++;
    
    const controls = document.getElementById('dialogue-controls');
    controls.style.display='flex';
    controls.innerHTML = `<button class="interactive-element px-4 py-2 rounded-lg text-white" style="background: #d97706;" onclick="showQuestion()">Next</button>`;
}

function finishQuiz(){
    const total = currentQuiz.questions.length;
    const percent = qCorrect/total;
    const passed = percent >= (currentQuiz.passPercent||0.8);
    
    if(passed && !gameState.quizzesCompleted.includes(currentQuiz.id)){
        gameState.quizzesCompleted.push(currentQuiz.id);
        addXP(currentQuiz.xp || 150);
        showNotification('Module Mastered', `${currentQuiz.title} passed! +${currentQuiz.xp || 150} XP`, 'success');
    } else if(passed){
        showNotification('Module Passed', `You passed again.`, 'info');
    } else {
        showNotification('Module Failed', `Score ${qCorrect}/${total}. Need ${(currentQuiz.passPercent*100).toFixed(0)}%`, 'error');
    }

    openDialogue(currentQuiz.title, `Quiz Complete — Score: ${qCorrect}/${total} (${Math.round(percent*100)}%).`);
    const opts = document.getElementById('quiz-options');
    opts.innerHTML = `<div class="mt-2 text-sm text-stone-200">Review below:</div>
        ${currentQuiz.questions.map((qq, i)=> {
            const correct = qq.correct;
            return `<div class="mt-3 p-3 rounded" style="background: #292524;">
                <div class="font-bold text-sm">${i+1}. ${qq.q}</div>
                <div class="text-xs md:text-sm mt-1">Answer: <span style="font-family: monospace;">${String.fromCharCode(65+correct)}. ${qq.a[correct]}</span></div>
                <div class="text-xs text-stone-300 mt-1">${qq.explanation||''}</div>
            </div>`;
        }).join('')}`;
    document.getElementById('dialogue-controls').innerHTML = `
        <div class="flex gap-2">
            <button onclick="closeDialogue()" class="px-3 py-2 rounded-lg text-white text-sm" style="background: #44403c;">Done</button>
            <button onclick="switchSection('quiz')" class="px-3 py-2 rounded-lg text-white text-sm" style="background: #d97706;">Back to Academy</button>
        </div>
    `;
    saveGame();
    updateStatsUI();
}

function openDialogue(name, text){
    document.getElementById('npc-name').textContent = name;
    document.getElementById('npc-text').innerHTML = text;
    document.getElementById('quiz-options').style.display = 'block';
    document.getElementById('dialogue-box').style.display = 'block';
    document.getElementById('crosshair').style.opacity = '0';
}

function closeDialogue(){
    document.getElementById('dialogue-box').style.display = 'none';
    document.getElementById('crosshair').style.opacity = '1';
}

function nextDialogue(){ closeDialogue(); }

/* ===========================
   ARTIFACTS
   =========================== */
function renderArtifactsTab(){
    currentSection = 'artifacts';
    document.querySelectorAll('.nav-button').forEach(b=>b.classList.remove('active'));
    const nav = document.getElementById('nav-artifacts'); 
    if(nav) nav.classList.add('active');

    contentArea.innerHTML = `
        <div class="max-w-6xl mx-auto p-4 md:p-6">
            <h1 class="text-2xl md:text-3xl font-bold mb-3">Artifacts Log</h1>
            <p class="text-sm md:text-base text-stone-600 mb-6">All artifacts you logged during excavations. Data saved locally.</p>
            <div id="artifact-list"></div>
        </div>
    `;
    const list = document.getElementById('artifact-list');
    if(gameState.artifacts.length === 0){
        list.innerHTML = `<div class="bg-white p-6 rounded-lg text-sm md:text-base">No artifacts logged yet. Find and log artifacts in the simulation!</div>`;
        return;
    }
    list.innerHTML = gameState.artifacts.map((a, idx)=>`
        <div class="artifact-row">
            <div class="artifact-thumb">${a.type || 'ART'}</div>
            <div class="flex
