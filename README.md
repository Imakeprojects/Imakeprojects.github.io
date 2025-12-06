
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=no" />
    <title>Train2Excavate: Volunteer Academy (Upgraded)</title>

    <script src="https://cdn.tailwindcss.com"></script>

    <!-- Three.js r128 + loaders -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/controls/PointerLockControls.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/loaders/GLTFLoader.js"></script>

    <style>
        body { margin: 0; overflow-x: hidden; touch-action: none; font-family: Inter, system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial; }
        #sim-container { position: fixed; top:0; left:0; width:100%; height:100%; z-index:1000; display:none; background:#000; }
        #ui-layer { position:absolute; inset:0; pointer-events:none; display:flex; flex-direction:column; justify-content:space-between; }
        .interactive-element { pointer-events:auto; }
        #crosshair { position:absolute; top:50%; left:50%; width:8px; height:8px; margin:-4px 0 0 -4px; border-radius:50%; background:#fcd34d; border:1px solid #78350f; z-index:1001; box-shadow:0 0 8px rgba(252,211,77,0.9); opacity:1; }
        .joystick-zone { position:absolute; bottom:40px; width:140px; height:140px; background:rgba(0,0,0,0.18); border:2px solid rgba(255,255,255,0.12); border-radius:50%; pointer-events:auto; display:none; }
        #stick-left{ left:20px } #stick-right{ right:20px }
        .stick-nub { position:absolute; top:50%; left:50%; width:60px; height:60px; transform:translate(-50%,-50%); border-radius:50%; background:rgba(245,158,11,0.85); pointer-events:none; box-shadow:0 0 18px rgba(0,0,0,0.6); }

        #dialogue-box { background: rgba(12,12,14,0.95); border:2px solid #d97706; color:#fff; padding:20px; border-radius:12px; max-width:920px; width:90%; margin:0 auto 24px auto; pointer-events:auto; display:none; }
        #notification-box { position:fixed; top:100px; right:20px; z-index:2000; padding:12px 18px; border-radius:8px; box-shadow:0 6px 18px rgba(0,0,0,0.35); display:none; opacity:0; transform:translateX(100%); transition:all .28s; }
        .progress-bar-container { width:100%; background:#44403c; border-radius:999px; overflow:hidden; height:12px; }
        .progress-bar-fill { height:100%; background:#f59e0b; transition:width .45s cubic-bezier(.2,.9,.2,1); box-shadow:0 0 8px #f59e0b; }

        /* Improved quiz buttons */
        .quiz-opt { display:block; text-align:left; padding:14px; border-radius:10px; background:#1f2937; color:#fff; margin-bottom:10px; cursor:pointer; border:1px solid rgba(255,255,255,0.03); }
        .quiz-opt.correct { background:linear-gradient(90deg,#065f46,#10b981); }
        .quiz-opt.wrong { background:linear-gradient(90deg,#7f1d1d,#ef4444); opacity:0.95; }

        .artifact-row { display:flex; gap:12px; align-items:center; padding:10px; border-radius:8px; background:#fff; margin-bottom:8px; }
        .artifact-thumb { width:72px; height:56px; border-radius:6px; background:#eee; display:flex; align-items:center; justify-content:center; border:1px solid #ddd; font-weight:bold; color:#444; }

        /* small responsive tweaks */
        @media (max-width:768px) {
            #dialogue-box { width:95%; padding:16px; }
            .artifact-thumb { width:54px; height:42px; font-size:12px; }
        }

        /* simple modal */
        .modal-backdrop { position:fixed; inset:0; background:rgba(0,0,0,0.6); display:none; align-items:center; justify-content:center; z-index:3000; }
        .modal { background:white; padding:18px; border-radius:12px; max-width:620px; width:94%; }
    </style>
</head>
<body class="bg-gray-50 text-stone-800">

    <!-- Notification -->
    <div id="notification-box" class="text-white"></div>

    <!-- Website -->
    <div id="website-container" class="min-h-screen flex flex-col">
        <nav class="bg-stone-900 text-stone-100 p-4 sticky top-0 z-50 shadow-xl border-b-4 border-amber-600">
            <div class="max-w-7xl mx-auto flex items-center justify-between gap-4">
                <div class="font-serif text-2xl font-extrabold text-amber-500">⛏️ Train2Excavate Academy</div>
                <div class="flex gap-6 items-center">
                    <div class="flex gap-6">
                        <button id="nav-home" onclick="switchSection('home')" class="nav-button active">HUB</button>
                        <button id="nav-guides" onclick="switchSection('guides')" class="nav-button">GUIDES</button>
                        <button id="nav-quiz" onclick="switchSection('quiz')" class="nav-button">ACADEMY</button>
                        <button id="nav-artifacts" onclick="switchSection('artifacts')" class="nav-button">ARTIFACTS</button>
                    </div>
                    <button onclick="startSimulation()" class="bg-amber-600 hover:bg-amber-700 text-white px-4 py-2 rounded-xl font-bold shadow-lg border border-amber-400">ENTRANCE SIM</button>
                </div>
            </div>
        </nav>

        <main id="content-area" class="flex-grow"></main>

        <footer class="bg-stone-900 text-stone-500 py-6 text-center">Train2Excavate &copy; 2025</footer>
    </div>

    <!-- 3D SIM -->
    <div id="sim-container" aria-hidden="true">
        <div id="crosshair" aria-hidden="true"></div>
        <div id="ui-layer">
            <div class="p-6 flex justify-between items-start">
                <div class="p-3 bg-black/40 rounded-lg interactive-element">
                    <h2 class="text-2xl font-bold text-amber-400">Site 42</h2>
                    <p class="text-sm text-stone-300">Objective: Explore & Document</p>
                </div>
                <div id="tool-hud" class="pointer-events-none">
                    <span class="tool-icon active" data-tool="trowel">⛏️</span>
                    <span class="tool-icon" data-tool="brush">🧹</span>
                    <span class="tool-icon" data-tool="sifter">🧺</span>
                    <span class="tool-icon" data-tool="tape">📏</span>
                </div>
                <button onclick="exitSimulation()" class="interactive-element bg-red-700 text-white px-4 py-2 rounded-xl">EXIT SIM</button>
            </div>

            <div class="w-full flex justify-center pb-20">
                <div id="dialogue-box" role="dialog" aria-modal="true">
                    <h3 id="npc-name" class="text-2xl font-bold text-amber-400">Name</h3>
                    <p id="npc-text" class="text-stone-200 mt-2">...</p>
                    <div id="quiz-options" class="mt-4"></div>
                    <div id="dialogue-controls" class="flex justify-end gap-3 mt-4">
                        <button onclick="nextDialogue()" class="interactive-element bg-amber-600 px-4 py-2 rounded-lg text-white">Continue</button>
                        <button onclick="closeDialogue()" class="interactive-element bg-stone-700 px-4 py-2 rounded-lg text-white">Dismiss</button>
                    </div>
                </div>
            </div>

            <div class="p-4 bg-black/30 backdrop-blur-sm text-white rounded-t-lg mx-auto mb-0 interactive-element">
                <div class="text-xs text-stone-400 uppercase font-bold text-center">Current Rank</div>
                <div class="text-amber-400 font-bold text-lg leading-none" id="sim-rank-display">Novice</div>
                <div class="progress-bar-container mt-1 w-48">
                    <div id="sim-xp-bar" class="progress-bar-fill" style="width:0%"></div>
                </div>
            </div>

            <div id="interaction-prompt" class="absolute top-1/2 left-1/2 transform -translate-x-1/2 -translate-y-1/2 opacity-0 pointer-events-none">
                <div class="bg-black/80 px-6 py-2 rounded-full border-2 border-amber-500 text-white font-bold">[Click / Tap] to Interact (E)</div>
            </div>

            <!-- Mobile Joysticks -->
            <div id="stick-left" class="joystick-zone"><div class="stick-nub" id="nub-left"></div></div>
            <div id="stick-right" class="joystick-zone"><div class="stick-nub" id="nub-right"></div></div>
        </div>
    </div>

    <!-- Artifact Log Modal (in-sim) -->
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

            <div class="space-y-2">
                <label class="block text-sm">Context / Unit</label>
                <input id="log-context" class="w-full px-3 py-2 border rounded" placeholder="e.g., Trench A, Layer 2">
                <label class="block text-sm">Short Description</label>
                <input id="log-desc" class="w-full px-3 py-2 border rounded" placeholder="e.g., rim sherd, iron nail">
                <label class="block text-sm">Material</label>
                <input id="log-material" class="w-full px-3 py-2 border rounded" placeholder="e.g., pottery, metal, bone">
            </div>

            <div class="mt-4 flex justify-end gap-2">
                <button onclick="closeArtifactModal()" class="px-4 py-2 rounded bg-gray-300">Cancel</button>
                <button onclick="saveArtifactLog()" class="px-4 py-2 rounded bg-amber-600 text-white">Save Log</button>
            </div>
        </div>
    </div>

<script>
/* -------------------------
   Game State & Configuration
   ------------------------- */
const gameState = {
    xp: 0,
    rank: "Novice Digger",
    quizzesCompleted: [],
    checklist: { boots:false, water:false, sunscreen:false, notebook:false, gloves:false, waiver:false },
    artifacts: [] // saved artifact logs
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
            { q:"Primary function of a sieve on site?", a:["Carry artifacts","Filter excavated dirt for small artifacts","Measure trench depth"], correct:1, explanation:"Sieves are used to catch small artifacts missed by hand-sorting." },
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

/* -------------------------
   Utilities & Persistence
   ------------------------- */
function saveGame() {
    localStorage.setItem('t2e_save_v4', JSON.stringify(gameState));
    updateStatsUI();
}
function loadGame() {
    const saved = localStorage.getItem('t2e_save_v4');
    if(saved){ const parsed = JSON.parse(saved); Object.assign(gameState, parsed); }
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
    // update page elements if present
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
    const simBar = document.getElementById('sim-xp-bar'); if(simBar) simBar.style.width = pct+'%';
    const simRank = document.getElementById('sim-rank-display'); if(simRank) simRank.innerText = gameState.rank;
    // refresh artifacts tab if open
    if(currentSection === 'artifacts') renderArtifactsTab();
}

/* Notification */
function showNotification(title, message, type='info') {
    const box = document.getElementById('notification-box');
    box.innerHTML = `<div class="font-bold">${title}</div><div class="text-sm">${message}</div>`;
    let bg='bg-blue-600', border='border-blue-400';
    if(type==='success'){ bg='bg-green-600'; border='border-green-400'; }
    if(type==='error'){ bg='bg-red-600'; border='border-red-400'; }
    if(type==='warning'){ bg='bg-amber-600'; border='border-amber-400'; }
    box.className = `text-white p-3 rounded shadow-lg ${bg} ${border}`;
    box.style.display='block';
    setTimeout(()=>{ box.style.opacity='1'; box.style.transform='translateX(0)'; },50);
    setTimeout(()=>{ box.style.opacity='0'; box.style.transform='translateX(100%)'; setTimeout(()=>box.style.display='none',300); },3500);
}

/* -------------------------
   UI Sections & Quiz fixes
   ------------------------- */
const contentArea = document.getElementById('content-area');
let currentSection = 'home';
function switchSection(name){
    window.scrollTo(0,0);
    currentSection=name;
    document.querySelectorAll('.nav-button').forEach(b=>b.classList.remove('active'));
    const nav = document.getElementById('nav-'+name); if(nav) nav.classList.add('active');

    if(name==='home'){
        const next = ranks[ranks.indexOf(ranks.find(r=>r.title===gameState.rank))+1];
        contentArea.innerHTML = `
            <div class="animate-fade-in pb-16">
                <div class="bg-stone-800 text-white py-16 px-6 text-center">
                    <h1 class="text-5xl font-serif font-extrabold text-amber-500 mb-2">${gameState.rank}</h1>
                    <p class="text-stone-300 mb-6">Advance by mastering modules. Next: ${next ? next.title : 'MAX'}</p>
                    <div class="max-w-xl mx-auto bg-stone-700 p-4 rounded-xl">
                        <div class="flex justify-between text-sm text-stone-300 mb-2 font-bold">
                            <span id="rank-display">${gameState.rank}</span><span>${next?next.title:'MASTER'}</span>
                        </div>
                        <div class="progress-bar-container h-4"><div id="xp-bar" class="progress-bar-fill" style="width:0%"></div></div>
                        <div class="text-right text-xs mt-1 text-amber-400 font-mono" id="xp-text"></div>
                    </div>
                </div>

                <div class="max-w-6xl mx-auto px-4 -mt-10 grid lg:grid-cols-3 gap-8">
                    <div class="bg-white p-6 rounded-xl shadow-lg lg:col-span-2">
                        <h2 class="text-2xl font-bold">Site Readiness Protocol</h2>
                        <p class="text-stone-600 mb-4">Prepare essentials before visiting the field.</p>
                        <div id="checklist-ui" class="grid sm:grid-cols-2 gap-4"></div>
                    </div>
                    <div class="bg-white p-6 rounded-xl shadow-lg">
                        <h2 class="text-2xl font-bold">Academy Performance</h2>
                        <div class="mt-4 space-y-3">
                            <div class="bg-green-50 p-4 rounded-lg">
                                <div class="text-3xl font-extrabold text-green-600">${gameState.quizzesCompleted.length}/${quizModules.length}</div>
                                <div class="text-xs text-stone-600 uppercase">Modules Mastered</div>
                            </div>
                            <button onclick="switchSection('quiz')" class="w-full bg-green-500 text-white py-2 rounded-lg">Go to Academy</button>
                        </div>
                    </div>
                </div>
            </div>
        `;
        renderChecklist();
        updateStatsUI();
    } else if(name==='guides') {
        contentArea.innerHTML = `
            <div class="max-w-5xl mx-auto p-8">
                <h1 class="text-3xl font-bold">Field Guides</h1>
                <p class="mt-4 text-stone-700">Detailed step-by-step guides coming soon.</p>
            </div>
        `;
    } else if(name==='quiz') {
        // render quiz modules
        contentArea.innerHTML = `
            <div class="max-w-4xl mx-auto p-8">
                <h1 class="text-3xl font-bold">Excavation Academy Quizzes</h1>
                <p class="mt-2 text-stone-700">Pass a module to earn XP. Passing thresholds are shown per module.</p>
                <div class="mt-6 space-y-6" id="quiz-list"></div>
            </div>
        `;
        const list = document.getElementById('quiz-list');
        list.innerHTML = quizModules.map(m=>{
            const completed = gameState.quizzesCompleted.includes(m.id);
            return `
                <div class="bg-white p-4 rounded-xl shadow ${completed?'border-l-8 border-green-500':'border-l-8 border-red-500'}">
                    <div class="flex justify-between items-center">
                        <div>
                            <div class="${completed?'text-green-700 text-xl font-bold':'text-red-700 text-xl font-bold'}">${m.title}</div>
                            <div class="text-sm text-stone-500">Questions: ${m.questions.length} • Pass: ${Math.round(m.passPercent*100)}% • XP: ${m.xp}</div>
                        </div>
                        <div class="flex gap-2">
                            <button onclick="startQuiz('${m.id}')" ${completed ? 'disabled' : ''} class="px-4 py-2 rounded ${completed?'bg-gray-400':'bg-amber-600 text-white'}">${completed?'Mastered':'Start'}</button>
                        </div>
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
        return `<button onclick="toggleChecklistItem('${k}')" class="flex items-center p-3 rounded ${checked?'bg-blue-50 border-2 border-blue-400':'bg-gray-100 border border-gray-200'}">
            <div class="text-2xl mr-3">${it.icon}</div>
            <div class="flex-grow">
                <div class="font-bold ${checked?'text-blue-700':'text-stone-800'}">${it.name}</div>
                <div class="text-sm text-stone-500">${checked?'Ready':'Missing'}</div>
            </div>
            <div>${checked? '✅':'⬜'}</div>
        </button>`;
    }).join('');
}
function toggleChecklistItem(k){ gameState.checklist[k]=!gameState.checklist[k]; saveGame(); renderChecklist(); }

/* -------------------------
   Quizzes (fixed + improved)
   ------------------------- */
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
    document.getElementById('npc-text').innerHTML = `<strong>Question ${qIndex+1}/${currentQuiz.questions.length}:</strong> <div class="mt-2">${q.q}</div>`;
    const opts = document.getElementById('quiz-options');
    opts.innerHTML = q.a.map((opt,i)=>`<button class="quiz-opt" onclick="submitAnswer(${i})">${String.fromCharCode(65+i)}. ${opt}</button>`).join('');
    opts.style.display='block';
    document.getElementById('dialogue-controls').style.display='none';
}

function submitAnswer(selected){
    const q = currentQuiz.questions[currentQuiz._order[qIndex]];
    const opts = Array.from(document.querySelectorAll('#quiz-options .quiz-opt'));
    opts.forEach((btn, i)=>{
        btn.disabled = true;
        if(i === q.correct) btn.classList.add('correct');
        if(i === selected && i !== q.correct) btn.classList.add('wrong');
        if(i !== q.correct && i !== selected) btn.style.opacity = '0.7';
    });
    if(selected === q.correct) qCorrect++;
    qIndex++;
    const controls = document.getElementById('dialogue-controls');
    controls.style.display='flex';
    controls.innerHTML = `<button class="interactive-element bg-amber-600 px-4 py-2 rounded text-white" onclick="showQuestion()">Next</button>`;
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

    // Show review
    openDialogue(currentQuiz.title, `Quiz Complete — Score: ${qCorrect}/${total} (${Math.round(percent*100)}%).`);
    const opts = document.getElementById('quiz-options');
    opts.innerHTML = `<div class="mt-2 text-sm text-stone-200">Review below:</div>
        ${currentQuiz.questions.map((qq, i)=> {
            const correct = qq.correct;
            return `<div class="mt-3 p-3 bg-stone-800 rounded">
                <div class="font-bold">${i+1}. ${qq.q}</div>
                <div class="text-sm mt-1">Answer: <span class="font-mono">${String.fromCharCode(65+correct)}. ${qq.a[correct]}</span></div>
                <div class="text-xs text-stone-300 mt-1">${qq.explanation||''}</div>
            </div>`;
        }).join('')}`;
    document.getElementById('dialogue-controls').innerHTML = `<div class="flex gap-2"><button onclick="closeDialogue()" class="bg-stone-700 px-3 py-2 rounded text-white">Done</button><button onclick="refreshQuizUI()" class="bg-amber-600 px-3 py-2 rounded text-white">Back to Academy</button></div>`;
    saveGame();
    updateStatsUI();
}

/* Refresh quiz UI without disrupting dialog flow */
function refreshQuizUI(){
    switchSection('quiz');
}

/* Dialog helpers */
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

/* -------------------------
   Artifact Log UI helpers
   ------------------------- */
function renderArtifactsTab(){
    currentSection = 'artifacts';
    document.querySelectorAll('.nav-button').forEach(b=>b.classList.remove('active'));
    const nav = document.getElementById('nav-artifacts'); if(nav) nav.classList.add('active');

    contentArea.innerHTML = `
        <div class="max-w-6xl mx-auto p-6">
            <h1 class="text-3xl font-bold mb-3">Artifacts Log</h1>
            <p class="text-stone-600 mb-6">All artifacts you logged during excavations. Data is saved in browser storage.</p>
            <div id="artifact-list"></div>
        </div>
    `;
    const list = document.getElementById('artifact-list');
    if(gameState.artifacts.length === 0){
        list.innerHTML = `<div class="bg-white p-6 rounded">No artifacts logged yet. Find and log artifacts in the simulation to see them here.</div>`;
        return;
    }
    list.innerHTML = gameState.artifacts.map((a, idx)=>`
        <div class="artifact-row">
            <div class="artifact-thumb">${a.type || 'ART'}</div>
            <div class="flex-1">
                <div class="font-bold">${a.desc || 'Unnamed Artifact'}</div>
                <div class="text-sm text-stone-500">Context: ${a.context || '—'} • Material: ${a.material || '—'} • Found: ${new Date(a.foundAt).toLocaleString()}</div>
            </div>
            <div class="text-sm">${a.xp ? '+'+a.xp+' XP':''}</div>
        </div>
    `).join('');
}

/* In-sim Artifact Modal controls */
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
    showNotification('Artifact Logged', `${desc} saved to your Artifacts Log. +${entry.xp} XP`, 'success');
}

/* -------------------------
   3D Simulation (Hybrid controls)
   ------------------------- */
let scene, camera, renderer, controls;
let simActive=false;
let npcs = [], playerHeight=1.8;
let prevTime = performance.now();
let moveF=false, moveB=false, moveL=false, moveR=false;
let velocity = new THREE.Vector3();
let canJump=true;
let raycaster = new THREE.Raycaster();

const simContainer = document.getElementById('sim-container');
const interactionPrompt = document.getElementById('interaction-prompt');

let soilPatches = []; // meshes that can be dug
let artifactSpawns = []; // artifact objects in world
let currentTool = 0; // 0=trowel,1=brush,2=sifter,3=tape
const toolIcons = document.querySelectorAll('.tool-icon');

function updateToolHUD(){ toolIcons.forEach((ic,i)=>{ ic.classList.toggle('active', i===currentTool); }); }
function switchTool(i){ currentTool = i % toolIcons.length; updateToolHUD(); showNotification('Tool Switched', `Active: ${toolIcons[currentTool].dataset.tool}`, 'info'); }

function initSim(){
    scene = new THREE.Scene();
    scene.fog = new THREE.FogExp2(0xa0b0c0, 0.02);

    camera = new THREE.PerspectiveCamera(75, innerWidth/innerHeight, 0.1, 500);
    camera.position.set(0, playerHeight, 8);

    renderer = new THREE.WebGLRenderer({ antialias:true });
    renderer.setSize(innerWidth, innerHeight);
    renderer.shadowMap.enabled = true;
    renderer.shadowMap.type = THREE.PCFSoftShadowMap;

    const existing = simContainer.querySelector('canvas');
    if(existing) existing.remove();
    simContainer.insertBefore(renderer.domElement, simContainer.firstChild);

    // lighting
    const hemi = new THREE.HemisphereLight(0xffffff, 0x444444, 0.6);
    hemi.position.set(0,50,0); scene.add(hemi);
    const dir = new THREE.DirectionalLight(0xffffff, 1.0);
    dir.position.set(30,50,30);
    dir.castShadow = true;
    dir.shadow.mapSize.width = dir.shadow.mapSize.height = 1024;
    dir.shadow.camera.near = 1; dir.shadow.camera.far = 200;
    dir.shadow.camera.left = -60; dir.shadow.camera.right = 60; dir.shadow.camera.top = 60; dir.shadow.camera.bottom = -60;
    scene.add(dir);
    scene.add(new THREE.AmbientLight(0xffffff, 0.12));

    // ground
    const texLoader = new THREE.TextureLoader();
    const groundTex = texLoader.load('https://threejs.org/examples/textures/terrain/grasslight-big.jpg');
    groundTex.wrapS = groundTex.wrapT = THREE.RepeatWrapping;
    groundTex.repeat.set(8,8);
    const normal = texLoader.load('https://threejs.org/examples/textures/terrain/grasslight-big-nm.jpg');
    normal.wrapS = normal.wrapT = THREE.RepeatWrapping; normal.repeat.set(8,8);

    const groundMat = new THREE.MeshStandardMaterial({ map:groundTex, normalMap:normal, roughness:1.0 });
    const groundGeo = new THREE.PlaneGeometry(400,400,32,32);
    const ground = new THREE.Mesh(groundGeo, groundMat);
    ground.rotation.x = -Math.PI/2;
    ground.receiveShadow = true;
    scene.add(ground);

    // ambient dust particles
    const dustGeo = new THREE.BufferGeometry();
    const dustCount = 300;
    const dustPos = new Float32Array(dustCount*3);
    for(let i=0;i<dustCount;i++){
        dustPos[i*3 + 0] = (Math.random()-0.5)*200;
        dustPos[i*3 + 1] = Math.random()*6 + 0.5;
        dustPos[i*3 + 2] = (Math.random()-0.5)*200;
    }
    dustGeo.setAttribute('position', new THREE.BufferAttribute(dustPos,3));
    const dustMat = new THREE.PointsMaterial({ size:1.5, transparent:true, opacity:0.06 });
    const dust = new THREE.Points(dustGeo, dustMat);
    scene.add(dust);

    // create trench area and soil patches
    createTrench(0, -0.5, 0, 12, 10, 1.2);
    createSoilPatch(0, 0, -2);
    createSoilPatch(2.3, 0, -0.6);
    createSoilPatch(-3.1, 0, 2.8);
    createSievePile(-1.8, 0.1, -3.4);

    // crates/rocks
    addCrate(3,0.2,-3); addCrate(-5,0.2,2);
    addRock(-2,0,-6); addRock(5,0,-2);

    // NPCs
    addNPC(6,0,6,'Dr. Eleanor Vance',0x8B4513,0xFAD5A5,[
        { text: `Welcome. I'm Dr. Vance. We need you to open a small trench and find any pottery sherds. Use the trowel to remove soil, then brush gently. I'll guide you.`, quizId:null, mission:{type:'find', targetCount:1} },
        { text: `Ready for the Tool Mastery quiz?`, quizId:'tools_101' }
    ]);
    addNPC(-5,0,1,'Dr. Henry Jones Jr.',0xCC7722,0xFAD5A5,[
        { text: `We're excavating a Roman-era villa. Document everything you find. Try sifting the small piles too.`, quizId:null }
    ]);
    addNPC(-8,0,-10,'Maya Sharma (Geophysicist)',0x228B22,0xFFA07A,[
        { text: `I ran the survey. Ready for geo basics?`, quizId:'geo_basics' }
    ]);

    // pointer lock controls
    controls = new THREE.PointerLockControls(camera, renderer.domElement);

    // events
    window.addEventListener('resize', ()=>{ camera.aspect = innerWidth/innerHeight; camera.updateProjectionMatrix(); renderer.setSize(innerWidth, innerHeight); }, false);
    document.addEventListener('keydown', onKeyDown);
    document.addEventListener('keyup', onKeyUp);
    renderer.domElement.addEventListener('mousedown', onMouseDown);
    renderer.domElement.addEventListener('touchstart', onTouchStartSim, {passive:false});
    renderer.domElement.addEventListener('touchmove', onTouchMoveSim, {passive:false});
    renderer.domElement.addEventListener('touchend', onTouchEndSim, {passive:false});

    // mobile joystick
    setupMobileJoystick();
}

/* --- Soil & Artifact World Objects --- */
function createSoilPatch(x,y,z){
    // soil patch represented by a layered cylinder whose scale reduces when dug
    const mat = new THREE.MeshStandardMaterial({ color:0x6b4a2b, roughness:1.0 });
    const geo = new THREE.CylinderGeometry(1.4,1.6,0.6,16);
    const mesh = new THREE.Mesh(geo, mat);
    mesh.position.set(x,y+0.3,z);
    mesh.castShadow = false; mesh.receiveShadow = true;
    mesh.userData = { type:'soil', integrity: 1.0, digging:false, artifactsHidden:[] };
    scene.add(mesh);
    soilPatches.push(mesh);

    // hide some random artifacts inside this patch
    const items = ['Pottery Sherd','Iron Nail','Bead','Flake','Coin'];
    const count = Math.floor(Math.random()*2)+1; // 1-2 artifacts per patch
    for(let i=0;i<count;i++){
        const a = {
            id: 'spawn_' + Math.random().toString(36).slice(2,9),
            name: items[Math.floor(Math.random()*items.length)],
            type: 'artifact',
            material: Math.random()>0.5? 'pottery':'metal',
            spawned: false,
            preview: (Math.random()>0.5? 'P':'M'),
            spawnContext: `Trench A - Patch ${soilPatches.length}`
        };
        mesh.userData.artifactsHidden.push(a);
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
    // add some small items
    const items = ['Flake','Bead','Small Bone','Charcoal Fragment'];
    for(let i=0;i< (Math.floor(Math.random()*3)+1); i++){
        mesh.userData.items.push({ id:'s_'+Math.random().toString(36).slice(2,8), name:items[Math.floor(Math.random()*items.length)], material:'mixed', preview:'s' });
    }
}

/* digging interaction: reduce soil integrity and spawn artifact when low */
function digSoilAt(mesh){
    if(!mesh || mesh.userData.type !== 'soil') return;
    mesh.userData.integrity -= 0.18; // each dig stroke removes 18%
    mesh.scale.y = Math.max(0.12, mesh.userData.integrity);
    mesh.position.y = 0.3 * mesh.scale.y; // sink visible top as scale changes
    if(mesh.userData.integrity <= 0.35 && mesh.userData.artifactsHidden.length){
        // reveal one artifact
        const art = mesh.userData.artifactsHidden.shift();
        spawnArtifactNear(mesh.position, art);
        showNotification('Possible find!', `You uncovered part of: ${art.name}`, 'info');
    }
    if(mesh.userData.integrity <= 0.12){
        // fully removed -> slight fade out
        mesh.material.opacity = 0.0; mesh.visible = false;
    }
}

/* spawn artifact object in world for brushing/logging */
function spawnArtifactNear(pos, artData){
    const geo = new THREE.SphereGeometry(0.18, 12, 10);
    const color = artData.material==='metal'? 0x9ea8b1 : 0x8b4a2a;
    const mat = new THREE.MeshStandardMaterial({ color: color, metalness: artData.material==='metal'?0.8:0.1, roughness:0.4 });
    const m = new THREE.Mesh(geo, mat);
    const jitter = (Math.random()-0.5)*0.8;
    m.position.set(pos.x + jitter, pos.y + 0.18, pos.z + (Math.random()-0.5)*0.8);
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

/* sifting */
function siftPile(mesh){
    if(!mesh || mesh.userData.type !== 'sieve') return;
    if(mesh.userData.sifted){ showNotification('Already sifted', 'This pile has already been sifted.', 'warning'); return; }
    mesh.userData.sifted = true;
    // reveal items into world with random throw
    mesh.userData.items.forEach(it=>{
        spawnArtifactNear(mesh.position, {name: it.name, material: it.material, preview: 's', spawnContext:'Sieve Pile'});
    });
    showNotification('Sift Complete', 'You found small artifacts in the sift.', 'success');
}

/* brush artifact to reveal it (change material & mark ready for logging) */
function brushArtifact(mesh){
    if(!mesh || mesh.userData.type !== 'artifact') return;
    if(mesh.userData.brushed){ showNotification('Already brushed', 'This artifact is already cleaned.', 'info'); return; }
    mesh.userData.brushed = true;
    // make it shinier and slightly larger to indicate reveal
    mesh.material.metalness = 0.3;
    mesh.material.roughness = 0.2;
    mesh.scale.set(1.15,1.15,1.15);
    showNotification('Artifact Cleaned', `You cleaned ${mesh.userData.name}. Log it now.`, 'success');
    // pop up logging modal
    openArtifactModal({ name: mesh.userData.name, type:mesh.userData.type, material:mesh.userData.material, preview: mesh.userData.preview, spawnContext:mesh.userData.spawnContext, xp: mesh.userData.xp });
}

/* -------------------------
   NPCs & Interaction
   ------------------------- */
function addNPC(x,y,z,name,bodyColor,headColor,dialogue){
    const group = new THREE.Group(); group.position.set(x,y,z);
    const body = new THREE.Mesh(new THREE.CylinderGeometry(0.32,0.4,1.6,16), new THREE.MeshStandardMaterial({color:bodyColor}));
    body.position.y = 0.8; group.add(body);
    const head = new THREE.Mesh(new THREE.SphereGeometry(0.34,24,16), new THREE.MeshStandardMaterial({color:headColor}));
    head.position.y = 1.8; group.add(head);
    // hat
    const brim = new THREE.Mesh(new THREE.CylinderGeometry(0.4,0.4,0.06,16), new THREE.MeshStandardMaterial({color:0x222}));
    brim.position.y = 2.12; group.add(brim);
    // name label (sprite)
    const canvas = document.createElement('canvas'); canvas.width=256; canvas.height=64;
    const cx = canvas.getContext('2d'); cx.fillStyle='rgba(0,0,0,0.6)'; cx.fillRect(0,0,256,64); cx.fillStyle='white'; cx.font='22px sans-serif'; cx.textAlign='center'; cx.fillText(name,128,40);
    const tx = new THREE.CanvasTexture(canvas);
    const spriteMat = new THREE.SpriteMaterial({ map:tx, depthTest:false });
    const sprite = new THREE.Sprite(spriteMat); sprite.scale.set(2.4,0.6,1); sprite.position.set(0,2.6,0.0); group.add(sprite);

    group.userData = { isNPC:true, name:name, dialogue:dialogue, currentDialogueIndex:0 };
    group.traverse(c => c.castShadow=true);
    scene.add(group); npcs.push(group);
}

/* interaction raycast: check NPCs, soil, artifacts, sieve piles */
function onInteractionRaycast(){
    raycaster.setFromCamera(new THREE.Vector2(0,0), camera);
    // check artifacts first (highest priority)
    const hitsArtifacts = raycaster.intersectObjects(artifactSpawns, true);
    if(hitsArtifacts.length>0){
        let mesh = hitsArtifacts[0].object;
        while(mesh.parent && !mesh.userData.type) mesh = mesh.parent;
        if(mesh.userData.type === 'artifact'){
            // depending on active tool
            if(currentTool === 1) { brushArtifact(mesh); return; }
            if(currentTool === 2) { showNotification('Use Sifter', 'Sifters work on soil piles, not on this artifact.', 'warning'); return; }
            // default: open logging if already brushed
            if(mesh.userData.brushed) openArtifactModal({ name:mesh.userData.name, material:mesh.userData.material, preview: mesh.userData.preview, spawnContext:mesh.userData.spawnContext, xp:mesh.userData.xp });
            else showNotification('Not clean', 'Brush this artifact before logging for best results.', 'warning');
            return;
        }
    }

    // check soil patches
    const hitsSoil = raycaster.intersectObjects(soilPatches, true);
    if(hitsSoil.length>0){
        let mesh = hitsSoil[0].object;
        while(mesh.parent && !mesh.userData.type) mesh = mesh.parent;
        if(mesh.userData.type === 'soil'){
            if(currentTool === 0){ digSoilAt(mesh); return; }
            if(currentTool === 2){ showNotification('Sifter Needed', 'Sifters work on piles, not soil patches.', 'warning'); return; }
            if(currentTool === 1) { showNotification('Brush', 'Brush is for artifacts after exposing them.', 'info'); return; }
        }
    }

    // check sieve piles
    const hitsSieve = raycaster.intersectObjects(scene.children, true).filter(i => i.object.userData && i.object.userData.type === 'sieve');
    if(hitsSieve.length > 0){
        let mesh = hitsSieve[0].object;
        while(mesh.parent && !mesh.userData.type) mesh = mesh.parent;
        if(mesh.userData.type === 'sieve'){
            if(currentTool === 2){ siftPile(mesh); return; }
            else { showNotification('Use Sifter', 'Use the sifter tool to sift this pile.', 'warning'); return; }
        }
    }

    // npc check
    const ints = raycaster.intersectObjects(npcs, true);
    if(ints.length>0){
        let group = ints[0].object;
        while(group.parent && !group.userData.isNPC) group = group.parent;
        if(group.userData.isNPC && group.position.distanceTo(camera.position) < 5) interactWithNPC(group);
    }
}

/* checkInteraction called from input handlers */
function checkInteraction(){
    onInteractionRaycast();
}

/* Interacting with NPCs */
let currentNPC = null;
function interactWithNPC(npc){
    currentNPC = npc;
    const dd = npc.userData.dialogue;
    npc.userData.currentDialogueIndex = 0;
    if(dd && dd.length>0){ disableControls(); const first = dd[0]; openDialogue(npc.userData.name, first.text); if(first.quizId){ document.getElementById('dialogue-controls').style.display='none'; startQuiz(first.quizId); } else { document.getElementById('dialogue-controls').style.display='flex'; } }
}

/* -------------------------
   Input Handling & Controls
   ------------------------- */
function onKeyDown(e){
    if(!simActive) return;
    if(e.code==='KeyW') moveF=true;
    if(e.code==='KeyS') moveB=true;
    if(e.code==='KeyA') moveL=true;
    if(e.code==='KeyD') moveR=true;
    if(e.code==='KeyE') { checkInteraction(); }
    if(e.code==='Digit1') switchTool(0);
    if(e.code==='Digit2') switchTool(1);
    if(e.code==='Digit3') switchTool(2);
    if(e.code==='Space' && canJump){ velocity.y += 6; canJump=false; }
}
function onKeyUp(e){
    if(!simActive) return;
    if(e.code==='KeyW') moveF=false;
    if(e.code==='KeyS') moveB=false;
    if(e.code==='KeyA') moveL=false;
    if(e.code==='KeyD') moveR=false;
}

/* Mouse / Pointer interactions */
function onMouseDown(e){
    if(!simActive) return;
    // if pointerlock is available and enabled, click interacts
    if(document.pointerLockElement === renderer.domElement){
        checkInteraction();
    } else {
        // try to lock on desktop left click
        if(!('ontouchstart' in window)) controls.lock();
    }
}

/* Touch handlers for sim (basic camera drag + tap to interact) */
let touchStartPos = null;
let touchDragging = false;
function onTouchStartSim(ev){
    if(!simActive) return;
    ev.preventDefault();
    if(ev.touches.length === 1){
        touchStartPos = { x: ev.touches[0].clientX, y: ev.touches[0].clientY, time: Date.now() };
        touchDragging = false;
    }
}
function onTouchMoveSim(ev){
    if(!simActive) return;
    if(ev.touches.length === 1 && touchStartPos){
        const dx = ev.touches[0].clientX - touchStartPos.x;
        const dy = ev.touches[0].clientY - touchStartPos.y;
        if(Math.abs(dx) > 8 || Math.abs(dy) > 8) touchDragging = true;
        // small camera rotation
        camera.rotation.y -= dx * 0.0025;
        camera.rotation.x -= dy * 0.0025;
        camera.rotation.x = Math.max(-Math.PI/2, Math.min(Math.PI/2, camera.rotation.x));
        touchStartPos.x = ev.touches[0].clientX;
        touchStartPos.y = ev.touches[0].clientY;
    }
}
function onTouchEndSim(ev){
    if(!simActive) return;
    if(!touchDragging && touchStartPos){
        // treat as tap -> interact
        checkInteraction();
    }
    touchStartPos = null; touchDragging = false;
}

/* Mobile joystick helpers */
const joystick = { left:{active:false,x:0,y:0,nub:null,zone:null,id:null}, right:{active:false,x:0,y:0,nub:null,zone:null,id:null} };
const maxDist = 70;
function setupMobileJoystick(){
    joystick.left.nub = document.getElementById('nub-left'); joystick.left.zone = document.getElementById('stick-left');
    joystick.right.nub = document.getElementById('nub-right'); joystick.right.zone = document.getElementById('stick-right');
    if(!joystick.left.zone) return;
    document.addEventListener('touchstart', onTouchStartJoystick,{passive:false});
    document.addEventListener('touchmove', onTouchMoveJoystick,{passive:false});
    document.addEventListener('touchend', onTouchEndJoystick,{passive:false});
    if('ontouchstart' in window){
        document.getElementById('stick-left').style.display='block';
        document.getElementById('stick-right').style.display='block';
    }
}
function onTouchStartJoystick(ev){
    if(!simActive) return;
    for(let i=0;i<ev.changedTouches.length;i++){
        const t = ev.changedTouches[i];
        // decide which zone by x coordinate
        const w = window.innerWidth;
        if(t.clientX < w*0.45 && !joystick.left.active){ joystick.left.active=true; joystick.left.id=t.identifier; }
        else if(t.clientX > w*0.55 && !joystick.right.active){ joystick.right.active=true; joystick.right.id=t.identifier; }
    }
}
function onTouchMoveJoystick(ev){ if(!simActive) return; ev.preventDefault();
    for(let i=0;i<ev.changedTouches.length;i++){
        const t = ev.changedTouches[i];
        if(joystick.left.active && t.identifier===joystick.left.id) updateJoystick(joystick.left,t,false);
        else if(joystick.right.active && t.identifier===joystick.right.id) updateJoystick(joystick.right,t,true);
    }
}
function updateJoystick(stick,t,isCam=false){
    const rect = { left: (isCam? window.innerWidth - 100 : 20), top: window.innerHeight - 180, width:140, height:140 };
    // center it
    const center = { x: rect.left + rect.width/2, y: rect.top + rect.height/2 };
    let x = t.clientX - center.x, y = t.clientY - center.y;
    const dist = Math.sqrt(x*x+y*y);
    if(dist>maxDist){ const ang=Math.atan2(y,x); x=Math.cos(ang)*maxDist; y=Math.sin(ang)*maxDist; }
    stick.nub.style.transform = `translate(${x}px, ${y}px)`;
    stick.x = x/maxDist; stick.y = y/maxDist;
    if(isCam){
        camera.rotation.y -= stick.x * 0.04;
        camera.rotation.x -= stick.y * 0.04;
        camera.rotation.x = Math.max(-Math.PI/2, Math.min(Math.PI/2, camera.rotation.x));
    } else {
        moveF = stick.y < -0.2; moveB = stick.y > 0.2; moveL = stick.x < -0.2; moveR = stick.x > 0.2;
    }
}
function onTouchEndJoystick(ev){
    for(let i=0;i<ev.changedTouches.length;i++){
        const t=ev.changedTouches[i];
        if(joystick.left.active && t.identifier===joystick.left.id){ joystick.left.active=false; joystick.left.id=null; joystick.left.nub.style.transform='translate(-50%,-50%)'; moveF=moveB=moveL=moveR=false; }
        if(joystick.right.active && t.identifier===joystick.right.id){ joystick.right.active=false; joystick.right.id=null; joystick.right.nub.style.transform='translate(-50%,-50%)'; }
    }
}

/* -------------------------
   Animation Loop
   ------------------------- */
function animate(){
    if(!simActive) return;
    requestAnimationFrame(animate);
    const time = performance.now();
    const delta = (time - prevTime)/1000; prevTime = time;

    // velocity friction & gravity
    velocity.x -= velocity.x * 6.0 * delta;
    velocity.z -= velocity.z * 6.0 * delta;
    velocity.y -= 9.8 * 2.5 * delta;

    const speed = 4.0;
    const dir = new THREE.Vector3();
    dir.z = (moveF?1:0) - (moveB?1:0);
    dir.x = (moveR?1:0) - (moveL?1:0);
    if(dir.length() > 0){ dir.normalize(); velocity.z = dir.z * speed; velocity.x = dir.x * speed; }
    else { velocity.x = 0; velocity.z = 0; }

    // movement relative to camera orientation
    const rotation = new THREE.Euler(0, camera.rotation.y, 0, 'YXZ');
    const forward = new THREE.Vector3(0,0,-1).applyEuler(rotation).multiplyScalar(velocity.z * delta);
    const right = new THREE.Vector3(1,0,0).applyEuler(rotation).multiplyScalar(velocity.x * delta);
    camera.position.add(forward).add(right);
    camera.position.y += velocity.y * delta;

    // ground clamp
    if(camera.position.y < playerHeight){ velocity.y = 0; camera.position.y = playerHeight; canJump=true; }

    // interaction prompt (if pointed at NPC or soil or artifact nearby)
    raycaster.setFromCamera(new THREE.Vector2(0,0), camera);
    const ints = raycaster.intersectObjects(npcs.concat(soilPatches).concat(artifactSpawns), true);
    if(ints.length>0){
        let g = ints[0].object;
        // climb up to parent containing userData
        while(g && !g.userData) g = g.parent;
        if(g && ((g.userData && g.userData.isNPC) || (g.userData && ['soil','artifact','sieve'].includes(g.userData.type))) && g.position.distanceTo(camera.position) < 5) {
            interactionPrompt.style.opacity='1';
        } else interactionPrompt.style.opacity='0';
    } else interactionPrompt.style.opacity='0';

    renderer.render(scene, camera);
}

/* -------------------------
   Simulation Control (start/stop)
   ------------------------- */
function startSimulation(){
    if(!renderer) initSim();
    simActive = true;
    simContainer.style.display = 'block';
    document.getElementById('website-container').style.display = 'none';
    prevTime = performance.now();
    animate();

    // Hybrid controls: pointerlock for desktop, joystick/touch for mobile
    if(!('ontouchstart' in window)){
        // desktop: allow pointer lock
        renderer.domElement.addEventListener('click', ()=>{ if(!controls.isLocked) controls.lock(); });
    } else {
        // mobile: show joystick controls (already set in setupMobileJoystick)
    }

    updateToolHUD();
    showNotification('Simulation Started', 'WASD/Joystick to Move, Mouse/Right Drag to Look, Tap/Click to Interact. 1-4 to switch tools.', 'warning');

    // keyboard tool switching
    document.addEventListener('keydown', toolSwitchKeyListener);
}

function toolSwitchKeyListener(ev){
    if(simActive && ev.key>='1' && ev.key<='4') switchTool(Number(ev.key)-1);
}

function exitSimulation(){ simActive = false; simContainer.style.display = 'none'; document.getElementById('website-container').style.display = 'flex'; if(document.pointerLockElement) document.exitPointerLock(); closeDialogue(); saveGame(); document.removeEventListener('keydown', toolSwitchKeyListener); }

/* -------------------------
   Helper geometry utilities
   ------------------------- */
function createTrench(x,y,z,width,depth,height){
    const mat = new THREE.MeshStandardMaterial({ color:0x8c6b4a, roughness:1.0 });
    const sideGeo = new THREE.BoxGeometry(width, height, 0.2);
    const leftGeo = new THREE.BoxGeometry(0.2, height, depth);
    const floorGeo = new THREE.PlaneGeometry(width-0.5, depth-0.5);

    const front = new THREE.Mesh(sideGeo, mat); front.position.set(x, y+height/2, z-depth/2); front.castShadow=true; front.receiveShadow=true; scene.add(front);
    const back = new THREE.Mesh(sideGeo, mat); back.position.set(x, y+height/2, z+depth/2); back.castShadow=true; back.receiveShadow=true; scene.add(back);
    const left = new THREE.Mesh(leftGeo, mat); left.position.set(x-width/2, y+height/2, z); left.castShadow=true; left.receiveShadow=true; scene.add(left);
    const right = new THREE.Mesh(leftGeo, mat); right.position.set(x+width/2, y+height/2, z); right.castShadow=true; right.receiveShadow=true; scene.add(right);

    const floorMat = new THREE.MeshStandardMaterial({ color:0x3d2b1f, roughness:1.0 });
    const floor = new THREE.Mesh(floorGeo, floorMat); floor.rotation.x = -Math.PI/2; floor.position.set(x, y, z); floor.receiveShadow=true; scene.add(floor);
}

function addCrate(x,y,z){
    const geo = new THREE.BoxGeometry(1.2,0.8,1.0);
    const mat = new THREE.MeshStandardMaterial({ color:0x6b4a2d, roughness:0.9 });
    const m = new THREE.Mesh(geo, mat);
    m.position.set(x,y+0.4,z); m.castShadow=true; m.receiveShadow=true;
    scene.add(m);
}

function addRock(x,y,z){
    const geo = new THREE.DodecahedronGeometry(0.8,0);
    const mat = new THREE.MeshStandardMaterial({ color:0x4b3b2a, roughness:1.0 });
    const r = new THREE.Mesh(geo, mat); r.position.set(x,y+0.4,z); r.rotation.set(Math.random(),Math.random(),Math.random()); r.castShadow=true; scene.add(r);
}

/* -------------------------
   Init & Expose
   ------------------------- */
window.onload = function(){
    loadGame();
    switchSection('home');
    setupControls();
};

function setupControls(){
    // intentionally left minimal as initSim wires most
}

/* Expose functions to buttons/console */
window.switchSection = switchSection;
window.startSimulation = startSimulation;
window.exitSimulation = exitSimulation;
window.startQuiz = startQuiz;
window.openDialogue = openDialogue;
window.closeDialogue = closeDialogue;
window.nextDialogue = nextDialogue;
window.toggleChecklistItem = toggleChecklistItem;
window.switchTool = switchTool;
window.openArtifactModal = openArtifactModal;
window.closeArtifactModal = closeArtifactModal;
window.saveArtifactLog = saveArtifactLog;

</script>
</body>
</html>
