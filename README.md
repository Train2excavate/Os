# Os
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>CrystalOS Infinity Pro</title>
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
            --bg: #2e0219;
            --glass: rgba(46, 2, 25, 0.8);
            --accent: #ff00ff;
            --text: #00ffff;
            --border: rgba(255, 0, 255, 0.3);
        }

        [data-theme="light"] {
            --bg: #f1f5f9;
            --glass: rgba(255, 255, 255, 0.8);
            --accent: #2563eb;
            --text: #1e293b;
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
            background: linear-gradient(45deg, var(--bg) 0%, #1e293b 100%);
        }

        .window {
            position: absolute; display: flex; flex-direction: column;
            background: var(--glass); backdrop-filter: blur(25px);
            border: 1px solid var(--border); border-radius: 12px;
            box-shadow: 0 20px 50px rgba(0,0,0,0.5);
            overflow: hidden; touch-action: none; min-width: 300px; min-height: 200px;
        }

        .window.maximized {
            inset: 10px !important;
            width: calc(100vw - 20px) !important;
            height: calc(100vh - 100px) !important;
            transform: none !important;
        }

        .window.minimized {
            display: none;
        }

        .title-bar {
            height: 38px; display: flex; align-items: center; padding: 0 12px;
            background: rgba(255,255,255,0.05); border-bottom: 1px solid var(--border);
        }

        .win-btn { width: 12px; height: 12px; border-radius: 50%; border: none; cursor: pointer; }
        .win-close { background: #ff5f56; }
        .win-min { background: #ffbd2e; }
        .win-max { background: #27c93f; }

        #taskbar {
            position: fixed; bottom: 12px; left: 50%; transform: translateX(-50%);
            height: 60px; background: var(--glass); backdrop-filter: blur(20px);
            border: 1px solid var(--border); border-radius: 18px;
            display: flex; align-items: center; padding: 0 10px; gap: 6px; z-index: 9999;
        }

        .dock-item {
            width: 42px; height: 42px; border-radius: 10px;
            display: flex; align-items: center; justify-content: center;
            font-size: 22px; cursor: pointer; transition: 0.2s;
            position: relative;
        }
        .dock-item:hover { transform: translateY(-4px); background: rgba(255,255,255,0.1); }
        .dock-item.running::after {
            content: ''; position: absolute; bottom: 2px; width: 4px; height: 4px;
            background: var(--accent); border-radius: 50%;
        }

        #desktop {
            display: grid; grid-template-columns: repeat(auto-fill, 90px);
            grid-auto-rows: 100px; gap: 15px; padding: 25px;
        }

        .icon {
            display: flex; flex-direction: column; align-items: center;
            cursor: pointer; padding: 8px; border-radius: 8px; transition: 0.2s;
        }
        .icon:hover { background: rgba(255,255,255,0.1); }
        .icon span { font-size: 32px; }
        .icon label { font-size: 11px; margin-top: 4px; text-align: center; font-weight: 500; }

        iframe { border: none; width: 100%; height: 100%; background: white; }
        
        /* App Specific Styles */
        .paint-canvas { cursor: crosshair; background: white; touch-action: none; }
        .settings-card { background: rgba(255,255,255,0.05); padding: 15px; border-radius: 10px; border: 1px solid var(--border); }
    </style>
</head>
<body data-theme="midnight">

    <div id="wallpaper"></div>
    <div id="desktop"></div>

    <div id="taskbar">
        <div class="dock-item" onclick="toggleStart()">💎</div>
        <div id="dock-running" class="flex gap-1"></div>
        <div class="w-[1px] h-6 bg-white/10 mx-2"></div>
        <div class="px-3 text-right hidden sm:block">
            <div id="time" class="text-xs font-bold">12:00 PM</div>
            <div id="date" class="text-[9px] opacity-60">DEC 18</div>
        </div>
    </div>

    <div id="win-layer"></div>

    <script>
        const APP_STORE = {
            frogies: { name: 'FrogiesArcade', icon: '🐸', type: 'iframe', url: 'https://frogiesarcade.win', installed: true },
            youtube: { name: 'YouTube', icon: '📺', type: 'iframe', url: 'https://www.youtube.com/embed/', installed: true },
            paint: { name: 'Paint Pro', icon: '🎨', type: 'native', installed: true },
            monitor: { name: 'Activity', icon: '📈', type: 'native', installed: true },
            code: { name: 'CrystalCode', icon: '💻', type: 'native', installed: true },
            docs: { name: 'Docs', icon: '📝', type: 'native', installed: true },
            store: { name: 'App Store', icon: '🛍️', type: 'system', installed: true },
            settings: { name: 'Settings', icon: '⚙️', type: 'system', installed: true },
            backrooms: { name: 'Backrooms', icon: '🔦', type: 'game', installed: true },
            chess: { name: 'Chess', icon: '♟️', type: 'iframe', url: 'https://lichess.org/export/embed/light/v2', installed: false }
        };

        let state = {
            windows: [], // {id, z, status: 'normal'|'min'|'max'}
            zIndex: 100,
            theme: localStorage.getItem('os_theme') || 'midnight',
            dragging: null,
            dragOff: {x:0, y:0}
        };

        function init() {
            document.body.setAttribute('data-theme', state.theme);
            renderDesktop();
            renderDock();
            updateClock();
            setInterval(updateClock, 1000);

            window.addEventListener('pointermove', e => {
                if(!state.dragging) return;
                state.dragging.style.left = (e.clientX - state.dragOff.x) + 'px';
                state.dragging.style.top = (e.clientY - state.dragOff.y) + 'px';
            });
            window.addEventListener('pointerup', () => state.dragging = null);
        }

        function renderDesktop() {
            const desk = document.getElementById('desktop');
            desk.innerHTML = '';
            Object.entries(APP_STORE).forEach(([id, app]) => {
                if(!app.installed) return;
                const div = document.createElement('div');
                div.className = 'icon';
                div.onclick = () => launch(id);
                div.innerHTML = `<span>${app.icon}</span><label>${app.name}</label>`;
                desk.appendChild(div);
            });
        }

        function renderDock() {
            const dock = document.getElementById('dock-running');
            dock.innerHTML = '';
            state.windows.forEach(win => {
                const app = APP_STORE[win.id];
                const div = document.createElement('div');
                div.className = `dock-item running`;
                div.innerHTML = app.icon;
                div.onclick = () => restore(win.id);
                dock.appendChild(div);
            });
        }

        function launch(id) {
            if(state.windows.find(w => w.id === id)) {
                restore(id);
                return;
            }

            const win = document.createElement('div');
            win.id = `win-${id}`;
            win.className = 'window';
            win.style.width = '700px';
            win.style.height = '500px';
            win.style.left = (100 + state.windows.length * 30) + 'px';
            win.style.top = (50 + state.windows.length * 30) + 'px';
            win.style.zIndex = ++state.zIndex;

            win.innerHTML = `
                <div class="title-bar" onpointerdown="startDrag(event, '${id}')">
                    <div class="flex gap-2 mr-4">
                        <div class="win-btn win-close" onclick="closeWin('${id}')"></div>
                        <div class="win-btn win-min" onclick="minimizeWin('${id}')"></div>
                        <div class="win-btn win-max" onclick="maximizeWin('${id}')"></div>
                    </div>
                    <span class="text-[10px] font-bold uppercase opacity-50">${APP_STORE[id].name}</span>
                </div>
                <div class="flex-1 overflow-hidden relative bg-black/10" id="body-${id}">
                    ${getAppContent(id)}
                </div>
            `;

            document.getElementById('win-layer').appendChild(win);
            state.windows.push({id, z: state.zIndex, status: 'normal'});
            renderDock();
            initAppLogic(id);
        }

        function getAppContent(id) {
            const app = APP_STORE[id];
            if(app.type === 'iframe') return `<iframe src="${app.url}"></iframe>`;
            
            switch(id) {
                case 'paint': return `<canvas class="paint-canvas w-full h-full"></canvas>`;
                case 'code': return `<textarea class="w-full h-full bg-[#1e1e1e] text-green-400 p-4 font-mono text-sm outline-none" spellcheck="false">/* CrystalCode v1.0 */\nfunction hello() {\n  console.log("Hello World");\n}</textarea>`;
                case 'monitor': return `
                    <div class="p-4 space-y-4">
                        <div class="settings-card">
                            <div class="flex justify-between mb-1 text-xs"><span>CPU Usage</span><span id="cpu-val">12%</span></div>
                            <div class="w-full bg-white/10 h-1 rounded-full overflow-hidden"><div id="cpu-bar" class="h-full bg-green-400" style="width: 12%"></div></div>
                        </div>
                        <div class="settings-card">
                            <div class="flex justify-between mb-1 text-xs"><span>Memory</span><span>1.4GB / 8GB</span></div>
                            <div class="w-full bg-white/10 h-1 rounded-full overflow-hidden"><div class="h-full bg-blue-400" style="width: 45%"></div></div>
                        </div>
                    </div>`;
                case 'settings': return `
                    <div class="p-6 space-y-6">
                        <h2 class="text-xl font-bold">Personalization</h2>
                        <div class="grid grid-cols-3 gap-4">
                            <button onclick="setTheme('midnight')" class="p-4 rounded-lg bg-slate-900 border border-white/10 text-xs">Midnight</button>
                            <button onclick="setTheme('cyberpunk')" class="p-4 rounded-lg bg-pink-900 border border-pink-500/50 text-xs">Cyberpunk</button>
                            <button onclick="setTheme('light')" class="p-4 rounded-lg bg-white border border-black/10 text-slate-900 text-xs">Light Mode</button>
                        </div>
                        <div class="settings-card mt-4">
                            <label class="text-xs opacity-50 block mb-2">OS Version</label>
                            <div class="text-sm font-bold">CrystalOS Infinity Pro v4.2.0-stable</div>
                        </div>
                    </div>`;
                case 'store': return `
                    <div class="p-6 overflow-auto h-full">
                        <h2 class="text-2xl font-bold mb-4">Marketplace</h2>
                        <div class="grid gap-3">
                            ${Object.entries(APP_STORE).map(([key, a]) => `
                                <div class="settings-card flex items-center justify-between">
                                    <div class="flex items-center gap-3">
                                        <div class="text-3xl">${a.icon}</div>
                                        <div>
                                            <div class="font-bold text-sm">${a.name}</div>
                                            <div class="text-[10px] opacity-50 uppercase">Professional App</div>
                                        </div>
                                    </div>
                                    <button onclick="install('${key}', this)" class="bg-blue-600 px-4 py-1 rounded-full text-[10px] font-bold ${a.installed ? 'opacity-20' : ''}" ${a.installed ? 'disabled' : ''}>
                                        ${a.installed ? 'INSTALLED' : 'GET'}
                                    </button>
                                </div>
                            `).join('')}
                        </div>
                    </div>`;
                default: return `<div class="p-10 opacity-50">App content loading...</div>`;
            }
        }

        function initAppLogic(id) {
            if(id === 'paint') {
                const canvas = document.querySelector(`#win-${id} canvas`);
                const ctx = canvas.getContext('2d');
                canvas.width = 700; canvas.height = 462;
                let drawing = false;
                ctx.strokeStyle = state.theme === 'light' ? '#000' : '#fff';
                ctx.lineWidth = 2;

                canvas.onpointerdown = () => drawing = true;
                canvas.onpointerup = () => { drawing = false; ctx.beginPath(); };
                canvas.onpointermove = (e) => {
                    if(!drawing) return;
                    const rect = canvas.getBoundingClientRect();
                    ctx.lineTo(e.clientX - rect.left, e.clientY - rect.top);
                    ctx.stroke();
                };
            }
            if(id === 'monitor') {
                setInterval(() => {
                    const cpu = Math.floor(Math.random() * 20) + 5;
                    const bar = document.getElementById('cpu-bar');
                    const val = document.getElementById('cpu-val');
                    if(bar) bar.style.width = cpu + '%';
                    if(val) val.innerText = cpu + '%';
                }, 2000);
            }
        }

        function startDrag(e, id) {
            const win = document.getElementById(`win-${id}`);
            if(win.classList.contains('maximized')) return;
            state.dragging = win;
            state.dragOff = { x: e.clientX - win.offsetLeft, y: e.clientY - win.offsetTop };
            win.style.zIndex = ++state.zIndex;
            e.target.setPointerCapture(e.pointerId);
        }

        function closeWin(id) {
            document.getElementById(`win-${id}`).remove();
            state.windows = state.windows.filter(w => w.id !== id);
            renderDock();
        }

        function minimizeWin(id) {
            const win = document.getElementById(`win-${id}`);
            win.classList.add('minimized');
        }

        function maximizeWin(id) {
            const win = document.getElementById(`win-${id}`);
            win.classList.toggle('maximized');
        }

        function restore(id) {
            const win = document.getElementById(`win-${id}`);
            if(win.classList.contains('minimized')) {
                win.classList.remove('minimized');
            }
            win.style.zIndex = ++state.zIndex;
        }

        function setTheme(t) {
            state.theme = t;
            localStorage.setItem('os_theme', t);
            document.body.setAttribute('data-theme', t);
        }

        function install(id, btn) {
            btn.innerText = '...';
            setTimeout(() => {
                APP_STORE[id].installed = true;
                btn.innerText = 'INSTALLED';
                btn.disabled = true;
                renderDesktop();
            }, 800);
        }

        function updateClock() {
            const d = new Date();
            document.getElementById('time').innerText = d.toLocaleTimeString([], {hour:'2-digit', minute:'2-digit'});
            document.getElementById('date').innerText = d.toLocaleDateString([], {month:'short', day:'numeric'}).toUpperCase();
        }

        function toggleStart() {
            // Placeholder for start menu logic
            launch('store');
        }

        window.onload = init;
    </script>
</body>
</html>
