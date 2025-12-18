<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>CrystalOS Infinity Ultra - V2.0</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;600;700&display=swap" rel="stylesheet">
    <style>
        :root {
            --bg: #020617;
            --glass: rgba(15, 23, 42, 0.75);
            --accent: #38bdf8;
            --text: #f8fafc;
            --border: rgba(255, 255, 255, 0.1);
        }

        [data-theme="cyberpunk"] {
            --bg: #0f0014;
            --glass: rgba(30, 0, 40, 0.85);
            --accent: #ff00ff;
            --text: #00ffff;
            --border: rgba(255, 0, 255, 0.3);
        }

        [data-theme="light"] {
            --bg: #f1f5f9;
            --glass: rgba(255, 255, 255, 0.8);
            --accent: #2563eb;
            --text: #0f172a;
            --border: rgba(0, 0, 0, 0.1);
        }

        body {
            margin: 0; padding: 0; height: 100vh; width: 100vw;
            font-family: 'Inter', sans-serif; overflow: hidden;
            background: var(--bg); color: var(--text);
            transition: background 0.5s ease;
        }

        #wallpaper {
            position: fixed; inset: 0; z-index: -1;
            background: radial-gradient(circle at top right, #1e293b, var(--bg));
            transition: filter 0.5s ease;
        }

        .window {
            position: absolute; display: flex; flex-direction: column;
            background: var(--glass); backdrop-filter: blur(25px);
            border: 1px solid var(--border); border-radius: 16px;
            box-shadow: 0 25px 50px -12px rgba(0, 0, 0, 0.6);
            overflow: hidden; touch-action: none;
            transition: transform 0.2s cubic-bezier(0.175, 0.885, 0.32, 1.275), opacity 0.2s, width 0.3s, height 0.3s, top 0.3s, left 0.3s;
        }

        .window.full {
            inset: 0 !important; width: 100vw !important; height: 100vh !important;
            border-radius: 0 !important; z-index: 99999 !important; transform: none !important;
        }

        .window.minimized { transform: scale(0.7) translateY(200px); opacity: 0; pointer-events: none; }

        .title-bar {
            height: 48px; display: flex; align-items: center; padding: 0 16px;
            background: rgba(255,255,255,0.03); border-bottom: 1px solid var(--border);
            user-select: none;
        }

        .win-btn { width: 12px; height: 12px; border-radius: 50%; cursor: pointer; border: none; }
        .win-close { background: #ff5f56; }
        .win-min { background: #ffbd2e; }
        .win-max { background: #27c93f; }

        #taskbar {
            position: fixed; bottom: 12px; left: 50%; transform: translateX(-50%);
            height: 68px; background: var(--glass); backdrop-filter: blur(30px);
            border: 1px solid var(--border); border-radius: 24px;
            display: flex; align-items: center; padding: 0 12px; gap: 4px; z-index: 10000;
            transition: transform 0.4s cubic-bezier(0.4, 0, 0.2, 1);
        }

        body.fullscreen-active #taskbar { transform: translate(-50%, 120px); }

        .dock-item {
            width: 52px; height: 52px; border-radius: 14px;
            display: flex; align-items: center; justify-content: center;
            font-size: 26px; cursor: pointer; transition: all 0.2s ease;
            position: relative;
        }
        .dock-item:hover { transform: scale(1.15) translateY(-8px); background: rgba(255,255,255,0.1); }
        .dock-item.active::after {
            content: ''; position: absolute; bottom: 6px; width: 4px; height: 4px;
            background: var(--accent); border-radius: 50%;
        }

        .app-grid {
            display: grid; grid-template-columns: repeat(auto-fill, minmax(110px, 1fr));
            gap: 20px; padding: 40px;
        }

        .app-icon {
            display: flex; flex-direction: column; align-items: center;
            cursor: pointer; padding: 15px; border-radius: 20px; transition: 0.3s;
            user-select: none; text-align: center;
        }
        .app-icon:hover { background: rgba(255,255,255,0.1); transform: translateY(-5px); }
        .app-icon span { font-size: 44px; filter: drop-shadow(0 8px 10px rgba(0,0,0,0.4)); margin-bottom: 8px; }
        .app-icon label { font-size: 13px; font-weight: 500; text-shadow: 0 2px 4px rgba(0,0,0,0.8); }

        #start-menu {
            position: fixed; bottom: 85px; left: 50%; transform: translateX(-50%) translateY(20px);
            width: 500px; height: 450px; background: var(--glass); backdrop-filter: blur(40px);
            border: 1px solid var(--border); border-radius: 28px; z-index: 9999;
            display: none; opacity: 0; transition: 0.3s ease;
            padding: 24px; box-shadow: 0 30px 60px -15px rgba(0,0,0,0.7);
        }
        #start-menu.open { display: block; opacity: 1; transform: translateX(-50%) translateY(0); }

        .quick-toggle {
            padding: 10px; border-radius: 12px; background: rgba(255,255,255,0.05);
            display: flex; align-items: center; gap: 10px; font-size: 12px;
            cursor: pointer; transition: 0.2s; border: 1px solid transparent;
        }
        .quick-toggle:hover { background: rgba(255,255,255,0.1); border-color: var(--accent); }
        .quick-toggle.on { background: var(--accent); color: white; }

        .loading-dots:after { content: '.'; animation: dots 1.5s steps(5, end) infinite;}
        @keyframes dots { 0%, 20% { content: '.'; } 40% { content: '..'; } 60% { content: '...'; } 80%, 100% { content: ''; } }

        canvas#drift-game { background: #111; border-radius: 8px; }
    </style>
</head>
<body data-theme="midnight">

    <div id="wallpaper"></div>

    <div id="start-menu">
        <div class="flex h-full gap-6">
            <div class="flex-1">
                <h3 class="text-xs font-bold opacity-40 uppercase tracking-widest mb-4">Recommended</h3>
                <div class="grid grid-cols-2 gap-3" id="start-apps"></div>
            </div>
            <div class="w-32 flex flex-col gap-3 border-l border-white/5 pl-6">
                <h3 class="text-xs font-bold opacity-40 uppercase mb-4">Quick</h3>
                <div class="quick-toggle" onclick="toggleQuick('wifi', this)">📡 WiFi</div>
                <div class="quick-toggle" onclick="toggleQuick('blue', this)">🌙 Night</div>
                <div class="quick-toggle" onclick="toggleQuick('focus', this)">⚡ Focus</div>
                <div class="mt-auto pt-4 border-t border-white/10">
                    <div class="text-[10px] opacity-40">SYSTEM</div>
                    <div class="text-xs font-bold">💎 Crystal Core</div>
                </div>
            </div>
        </div>
    </div>

    <div id="desktop" class="app-grid"></div>

    <div id="taskbar">
        <div class="dock-item" onclick="toggleStart()">💎</div>
        <div id="dock-running" class="flex gap-2"></div>
        <div class="w-[1px] h-8 bg-white/10 mx-2"></div>
        <div class="pr-2 text-right hidden sm:block pointer-events-none min-w-[80px]">
            <div id="time" class="text-sm font-bold leading-tight">12:00</div>
            <div id="date" class="text-[10px] opacity-60 font-semibold uppercase">Dec 18</div>
        </div>
    </div>

    <div id="win-layer"></div>

    <script type="module">
        const apiKey = ""; // Provided by env

        const APP_REGISTRY = {
            // SYSTEM
            store: { name: 'Marketplace', icon: '🛍️', type: 'system', installed: true, desc: 'Get new software' },
            updates: { name: 'Updates', icon: '🚀', type: 'system', installed: true, desc: 'System versioning' },
            settings: { name: 'Settings', icon: '⚙️', type: 'system', installed: true, desc: 'Customize OS' },
            
            // DAILY / SOCIAL
            frogies: { name: 'FrogiesArcade', icon: '🐸', type: 'iframe', url: 'https://frogiesarcade.win', installed: true, desc: 'Web Games' },
            mail: { name: 'C-Mail', icon: '📧', type: 'native', installed: false, desc: 'Fast messaging' },
            calendar: { name: 'Crystal Calendar', icon: '📅', type: 'native', installed: false, desc: 'Stay organized' },
            
            // PROFESSIONAL
            vision: { name: 'Crystal Vision', icon: '👁️', type: 'ai', installed: false, desc: 'AI Image Creator' },
            excel: { name: 'Excelerator', icon: '📊', type: 'native', installed: false, desc: 'Spreadsheet tool' },
            flow: { name: 'Project Flow', icon: '📋', type: 'native', installed: false, desc: 'Task Manager' },
            codepad: { name: 'CodePad', icon: '💻', type: 'native', installed: true, desc: 'Dev IDE' },
            
            // GAMES
            drift: { name: 'Neon Drift', icon: '🏎️', type: 'native', installed: false, desc: 'Racing Sim' },
            tiles: { name: 'Zen Tiles', icon: '🧱', type: 'native', installed: false, desc: 'Logic Puzzle' }
        };

        let state = {
            windows: [],
            zIndex: 100,
            theme: localStorage.getItem('os_theme') || 'midnight'
        };

        function init() {
            renderDesktop();
            renderStart();
            updateClock();
            setInterval(updateClock, 1000);
            document.body.setAttribute('data-theme', state.theme);
            
            // Global click to close start
            window.addEventListener('click', (e) => {
                if(!e.target.closest('#start-menu') && !e.target.closest('.dock-item')) {
                    document.getElementById('start-menu').classList.remove('open');
                }
            });
        }

        function renderDesktop() {
            const desk = document.getElementById('desktop');
            desk.innerHTML = '';
            Object.entries(APP_REGISTRY).forEach(([id, app]) => {
                if(!app.installed) return;
                const div = document.createElement('div');
                div.className = 'app-icon';
                div.onclick = () => launch(id);
                div.innerHTML = `<span>${app.icon}</span><label>${app.name}</label>`;
                desk.appendChild(div);
            });
        }

        function renderStart() {
            const startApps = document.getElementById('start-apps');
            startApps.innerHTML = Object.entries(APP_REGISTRY).slice(0, 6).map(([id, app]) => `
                <div onclick="launch('${id}')" class="flex items-center gap-3 p-2 hover:bg-white/10 rounded-xl cursor-pointer transition-all">
                    <div class="text-2xl">${app.icon}</div>
                    <div class="text-xs font-bold">${app.name}</div>
                </div>
            `).join('');
        }

        function launch(id) {
            document.getElementById('start-menu').classList.remove('open');
            if(state.windows.find(w => w.id === id)) {
                restore(id); return;
            }

            const win = document.createElement('div');
            win.id = `win-${id}`;
            win.className = 'window';
            win.style.width = '800px';
            win.style.height = '580px';
            win.style.left = (120 + state.windows.length * 30) + 'px';
            win.style.top = (80 + state.windows.length * 30) + 'px';
            win.style.zIndex = ++state.zIndex;

            win.innerHTML = `
                <div class="title-bar" onpointerdown="startDrag(event, '${id}')">
                    <div class="flex gap-2 mr-6">
                        <div class="win-btn win-close" onclick="closeWin('${id}')"></div>
                        <div class="win-btn win-min" onclick="minimizeWin('${id}')"></div>
                        <div class="win-btn win-max" onclick="toggleFull('${id}')"></div>
                    </div>
                    <span class="text-[10px] font-black tracking-widest uppercase opacity-40">${APP_REGISTRY[id].name}</span>
                </div>
                <div class="flex-1 overflow-hidden" id="body-${id}">
                    ${getAppContent(id)}
                </div>
            `;

            document.getElementById('win-layer').appendChild(win);
            state.windows.push({id, z: state.zIndex});
            updateDock();
            initAppLogic(id);
        }

        function getAppContent(id) {
            const app = APP_REGISTRY[id];
            if(app.type === 'iframe') return `<iframe src="${app.url}" class="w-full h-full border-0"></iframe>`;
            
            switch(id) {
                case 'store': return `
                    <div class="p-8">
                        <div class="flex justify-between items-center mb-8">
                            <h2 class="text-3xl font-black italic uppercase">Crystal Market</h2>
                            <div class="text-[10px] font-bold opacity-30">V2.0.4</div>
                        </div>
                        <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                            ${Object.entries(APP_REGISTRY).filter(a => !a[1].installed).map(([id, a]) => `
                                <div class="bg-white/5 border border-white/5 p-4 rounded-2xl flex items-center justify-between group hover:border-white/20 transition-all">
                                    <div class="flex items-center gap-4">
                                        <div class="text-4xl group-hover:scale-110 transition-transform">${a.icon}</div>
                                        <div>
                                            <div class="font-bold text-sm">${a.name}</div>
                                            <div class="text-[10px] opacity-40">${a.desc}</div>
                                        </div>
                                    </div>
                                    <button onclick="installApp('${id}', this)" class="bg-white text-black text-[10px] font-black px-4 py-2 rounded-full hover:bg-blue-400 hover:text-white transition-colors">GET</button>
                                </div>
                            `).join('') || '<div class="col-span-2 text-center py-20 opacity-30">Environment fully equipped.</div>'}
                        </div>
                    </div>`;
                case 'vision': return `
                    <div class="p-8 flex flex-col h-full bg-[#050505]">
                        <div class="flex-1 flex items-center justify-center border-2 border-dashed border-white/5 rounded-2xl mb-6 relative overflow-hidden" id="vision-output">
                            <div class="text-center opacity-20">
                                <div class="text-6xl mb-4">✨</div>
                                <p class="text-xs font-bold uppercase tracking-widest">Enter prompt below</p>
                            </div>
                        </div>
                        <div class="flex gap-2">
                            <input type="text" id="vision-prompt" placeholder="A futuristic neon city in 4k..." class="flex-1 bg-white/5 border border-white/10 rounded-xl px-4 py-3 outline-none focus:border-blue-500 transition-all">
                            <button id="vision-btn" class="bg-blue-600 px-8 rounded-xl font-bold text-sm hover:bg-blue-500">GENERATE</button>
                        </div>
                    </div>`;
                case 'drift': return `
                    <div class="p-4 h-full bg-black flex flex-col">
                        <canvas id="drift-game" class="w-full flex-1"></canvas>
                        <div class="mt-4 flex justify-between text-[10px] font-bold opacity-40">
                            <span>ARROWS TO MOVE</span>
                            <span>SCORE: <span id="drift-score">0</span></span>
                        </div>
                    </div>`;
                case 'mail': return `
                    <div class="flex h-full">
                        <div class="w-48 border-r border-white/5 p-4 space-y-2">
                            <div class="bg-blue-600 rounded-lg p-2 text-center text-xs font-bold mb-4">COMPOSE</div>
                            <div class="text-[10px] font-bold opacity-40 uppercase tracking-tighter">Inbox</div>
                            <div class="bg-white/5 p-2 rounded-lg text-xs font-medium">Internal System</div>
                            <div class="p-2 text-xs opacity-50">Updates</div>
                        </div>
                        <div class="flex-1 p-8">
                            <h2 class="text-xl font-bold mb-2">Welcome to Crystal OS</h2>
                            <p class="text-sm opacity-60 leading-relaxed">Thank you for installing the latest build of the Crystal Environment. We are excited for you to explore the new Marketplace...</p>
                        </div>
                    </div>`;
                case 'codepad': return `<textarea class="w-full h-full bg-[#0d1117] text-gray-300 p-8 font-mono text-sm outline-none resize-none leading-loose">// Crystal IDE v2.1\nfunction startCore() {\n  console.log("Kernel active.");\n}</textarea>`;
                case 'settings': return `
                    <div class="p-10 space-y-8">
                        <h2 class="text-4xl font-black uppercase italic tracking-tighter">Environment</h2>
                        <div class="grid grid-cols-2 gap-6">
                            <div class="bg-white/5 p-6 rounded-3xl border border-white/5">
                                <label class="text-[10px] font-bold uppercase opacity-30 mb-4 block">Visual Profile</label>
                                <div class="flex gap-4">
                                    <div onclick="setOSTheme('midnight')" class="w-12 h-12 rounded-full bg-slate-900 border-4 border-white/10 cursor-pointer"></div>
                                    <div onclick="setOSTheme('cyberpunk')" class="w-12 h-12 rounded-full bg-purple-900 border-4 border-white/10 cursor-pointer"></div>
                                    <div onclick="setOSTheme('light')" class="w-12 h-12 rounded-full bg-slate-100 border-4 border-black/10 cursor-pointer"></div>
                                </div>
                            </div>
                            <div class="bg-white/5 p-6 rounded-3xl border border-white/5">
                                <label class="text-[10px] font-bold uppercase opacity-30 mb-4 block">System Info</label>
                                <div class="text-xs space-y-2">
                                    <div class="flex justify-between"><span>Kernel</span><span>2.5.0-v6</span></div>
                                    <div class="flex justify-between"><span>Uptime</span><span>0h 12m</span></div>
                                </div>
                            </div>
                        </div>
                    </div>`;
                case 'updates': return `
                    <div class="p-12 text-center max-w-sm mx-auto">
                        <div class="text-6xl mb-6">🚀</div>
                        <h2 class="text-2xl font-bold mb-2">OS Build 2025</h2>
                        <p class="text-xs opacity-40 mb-8">Everything is synchronized and running optimally.</p>
                        <div class="bg-white/5 py-4 px-6 rounded-2xl border border-white/5 text-[10px] font-bold uppercase tracking-widest">Checking for updates...</div>
                    </div>`;
                case 'excel': return `
                    <div class="h-full bg-white text-black p-4">
                        <div class="grid grid-cols-6 border-l border-t border-gray-300">
                            ${Array(60).fill(0).map((_, i) => `<div class="border-r border-b border-gray-200 h-8 flex items-center px-2 text-[10px] ${i < 6 ? 'bg-gray-100 font-bold' : 'bg-white'}" contenteditable="true">${i < 6 ? String.fromCharCode(65+i) : ''}</div>`).join('')}
                        </div>
                    </div>`;
                case 'flow': return `
                    <div class="p-8">
                        <h2 class="text-2xl font-bold mb-6">Today's Flow</h2>
                        <div class="space-y-3">
                            ${['Optimize Kernel', 'Update Graphics Driver', 'Review Mail'].map(t => `
                                <div class="flex items-center gap-4 bg-white/5 p-4 rounded-xl">
                                    <input type="checkbox" class="w-4 h-4 rounded bg-white/10 border-0">
                                    <span class="text-sm font-medium">${t}</span>
                                </div>
                            `).join('')}
                        </div>
                    </div>`;
                case 'tiles': return `
                    <div class="h-full bg-[#1a1a1a] flex items-center justify-center">
                        <div class="grid grid-cols-4 gap-2">
                            ${Array(16).fill(0).map((_, i) => `<div onclick="this.classList.toggle('bg-blue-500')" class="w-14 h-14 bg-white/10 rounded-lg cursor-pointer transition-colors"></div>`).join('')}
                        </div>
                    </div>`;
                default: return `<div class="p-20 text-center">System application error</div>`;
            }
        }

        async function initAppLogic(id) {
            if(id === 'vision') {
                const btn = document.getElementById('vision-btn');
                const output = document.getElementById('vision-output');
                const promptIn = document.getElementById('vision-prompt');
                
                btn.onclick = async () => {
                    const prompt = promptIn.value.trim();
                    if(!prompt) return;
                    
                    btn.disabled = true;
                    output.innerHTML = `<div class="text-xs font-bold uppercase tracking-widest text-blue-400">Dreaming<span class="loading-dots"></span></div>`;
                    
                    try {
                        const response = await fetch(`https://generativelanguage.googleapis.com/v1beta/models/imagen-4.0-generate-001:predict?key=${apiKey}`, {
                            method: "POST",
                            body: JSON.stringify({ instances: { prompt: prompt }, parameters: { sampleCount: 1 } })
                        });
                        const data = await response.json();
                        const imgUrl = `data:image/png;base64,${data.predictions[0].bytesBase64Encoded}`;
                        output.innerHTML = `<img src="${imgUrl}" class="w-full h-full object-cover">`;
                    } catch (e) {
                        output.innerHTML = `<div class="text-red-400 text-xs">AI Service Unavailable</div>`;
                    }
                    btn.disabled = false;
                };
            }
            if(id === 'drift') {
                const canvas = document.getElementById('drift-game');
                const scoreEl = document.getElementById('drift-score');
                const ctx = canvas.getContext('2d');
                canvas.width = 750; canvas.height = 450;
                let player = { x: 375, y: 400, w: 20, h: 40 };
                let obstacles = [];
                let score = 0;
                let keys = {};
                window.onkeydown = (e) => keys[e.code] = true;
                window.onkeyup = (e) => keys[e.code] = false;

                function loop() {
                    if(!document.getElementById(`win-${id}`)) return;
                    ctx.fillStyle = '#111'; ctx.fillRect(0,0,750,450);
                    
                    // Input
                    if(keys['ArrowLeft'] && player.x > 0) player.x -= 5;
                    if(keys['ArrowRight'] && player.x < 730) player.x += 5;
                    
                    // Obstacles
                    if(Math.random() < 0.05) obstacles.push({ x: Math.random()*700, y: -50, w: 30, h: 30 });
                    obstacles.forEach((o, i) => {
                        o.y += 4;
                        ctx.fillStyle = '#ff00ff'; ctx.fillRect(o.x, o.y, o.w, o.h);
                        if(o.y > 450) { obstacles.splice(i, 1); score++; scoreEl.innerText = score; }
                        // Collision
                        if(player.x < o.x + o.w && player.x + player.w > o.x && player.y < o.y + o.h && player.y + player.h > o.y) {
                            score = 0; scoreEl.innerText = score; obstacles = [];
                        }
                    });

                    ctx.fillStyle = '#00ffff'; ctx.fillRect(player.x, player.y, player.w, player.h);
                    requestAnimationFrame(loop);
                }
                loop();
            }
        }

        // Window Management Logic
        window.startDrag = (e, id) => {
            const win = document.getElementById(`win-${id}`);
            if(win.classList.contains('full')) return;
            const offX = e.clientX - win.offsetLeft;
            const offY = e.clientY - win.offsetTop;
            win.style.zIndex = ++state.zIndex;
            
            const move = (ev) => {
                win.style.left = (ev.clientX - offX) + 'px';
                win.style.top = (ev.clientY - offY) + 'px';
            };
            const up = () => {
                window.removeEventListener('pointermove', move);
                window.removeEventListener('pointerup', up);
            };
            window.addEventListener('pointermove', move);
            window.addEventListener('pointerup', up);
            e.target.setPointerCapture(e.pointerId);
        };

        window.toggleFull = (id) => {
            const win = document.getElementById(`win-${id}`);
            win.classList.toggle('full');
            document.body.classList.toggle('fullscreen-active', win.classList.contains('full'));
        };

        window.minimizeWin = (id) => {
            const win = document.getElementById(`win-${id}`);
            win.classList.add('minimized');
            document.body.classList.remove('fullscreen-active');
        };

        window.closeWin = (id) => {
            const win = document.getElementById(`win-${id}`);
            win.style.opacity = '0';
            win.style.transform = 'scale(0.8) translateY(20px)';
            setTimeout(() => {
                win.remove();
                state.windows = state.windows.filter(w => w.id !== id);
                document.body.classList.remove('fullscreen-active');
                updateDock();
            }, 200);
        };

        window.restore = (id) => {
            const win = document.getElementById(`win-${id}`);
            win.classList.remove('minimized');
            win.style.zIndex = ++state.zIndex;
            if(win.classList.contains('full')) document.body.classList.add('fullscreen-active');
        };

        window.updateDock = () => {
            const container = document.getElementById('dock-running');
            container.innerHTML = state.windows.map(w => `
                <div class="dock-item active" onclick="restore('${w.id}')">${APP_REGISTRY[w.id].icon}</div>
            `).join('');
        };

        window.toggleStart = () => {
            document.getElementById('start-menu').classList.toggle('open');
        };

        window.installApp = (id, btn) => {
            btn.innerHTML = '...'; btn.disabled = true;
            setTimeout(() => {
                APP_REGISTRY[id].installed = true;
                renderDesktop();
                renderStart();
                launch(id);
            }, 1000);
        };

        window.setOSTheme = (t) => {
            state.theme = t;
            localStorage.setItem('os_theme', t);
            document.body.setAttribute('data-theme', t);
        };

        window.toggleQuick = (type, el) => {
            el.classList.toggle('on');
            if(type === 'blue') {
                document.getElementById('wallpaper').style.filter = el.classList.contains('on') ? 'sepia(0.5) brightness(0.8)' : 'none';
            }
        };

        function updateClock() {
            const now = new Date();
            document.getElementById('time').innerText = now.toLocaleTimeString([], {hour:'2-digit', minute:'2-digit'});
            document.getElementById('date').innerText = now.toLocaleDateString([], {month:'short', day:'numeric'});
        }

        init();
    </script>
</body>
</html>
