<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>CrystalOS Infinity - Fixed</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@400;500;600;700&family=JetBrains+Mono:wght@400&display=swap" rel="stylesheet">
    <style>
        :root {
            --bg-color: #0f172a;
            --glass-bg: rgba(15, 23, 42, 0.65);
            --glass-border: rgba(255, 255, 255, 0.1);
            --text-color: #f8fafc;
            --accent-color: #38bdf8;
            --radius-win: 18px;
            --font-main: 'Plus Jakarta Sans', sans-serif;
        }

        body {
            margin: 0; padding: 0; height: 100vh; width: 100vw;
            font-family: var(--font-main);
            overflow: hidden; background: #000; color: var(--text-color);
            user-select: none; -webkit-user-select: none;
            touch-action: none;
        }

        /* Animated Wallpaper */
        #wallpaper {
            position: fixed; inset: 0; z-index: -1;
            background: radial-gradient(circle at 50% 50%, #1e1b4b 0%, #020617 100%);
            transition: background 0.5s ease;
        }
        
        .blob {
            position: absolute; border-radius: 50%;
            filter: blur(80px); opacity: 0.6;
            animation: float 20s infinite alternate;
        }
        @keyframes float { 0% {transform:translate(0,0)} 100% {transform:translate(30px, 50px)} }

        /* Window System */
        .window {
            position: absolute; display: flex; flex-direction: column;
            background: var(--glass-bg);
            backdrop-filter: blur(25px); -webkit-backdrop-filter: blur(25px);
            border: 1px solid var(--glass-border); border-radius: var(--radius-win);
            box-shadow: 0 25px 50px -12px rgba(0,0,0,0.5);
            overflow: hidden; touch-action: none;
            transition: transform 0.1s, opacity 0.15s, width 0.2s, height 0.2s;
            min-width: 300px; min-height: 200px;
        }
        
        .window.active {
            z-index: 100;
            border-color: rgba(255,255,255,0.3);
            box-shadow: 0 40px 80px -15px rgba(0,0,0,0.6);
        }
        
        .window.maximized {
            top: 0 !important; left: 0 !important;
            width: 100% !important; height: calc(100% - 70px) !important;
            border-radius: 0;
        }
        
        .window.minimized {
            transform: scale(0.7) translateY(120%);
            opacity: 0; pointer-events: none;
        }

        .title-bar {
            height: 44px; display: flex; align-items: center; justify-content: space-between;
            padding: 0 16px; background: rgba(255,255,255,0.05); border-bottom: 1px solid var(--glass-border);
            cursor: move;
        }

        .win-controls { display: flex; gap: 8px; }
        .win-btn { width: 12px; height: 12px; border-radius: 50%; border: none; cursor: pointer; }
        .close { background: #ff5f56; } .min { background: #ffbd2e; } .max { background: #27c93f; }

        /* Desktop & Taskbar */
        #desktop {
            display: grid; grid-template-columns: repeat(auto-fill, 90px);
            grid-template-rows: repeat(auto-fill, 100px);
            gap: 12px; padding: 24px; height: calc(100vh - 80px);
            align-content: start; pointer-events: auto;
        }

        .icon {
            display: flex; flex-direction: column; align-items: center;
            cursor: pointer; padding: 10px; border-radius: 14px;
            transition: background 0.2s;
        }
        .icon:hover { background: rgba(255,255,255,0.1); }
        .icon-box {
            width: 56px; height: 56px; border-radius: 14px;
            display: flex; align-items: center; justify-content: center;
            font-size: 30px; background: linear-gradient(135deg, rgba(255,255,255,0.1), rgba(255,255,255,0.02));
            border: 1px solid var(--glass-border); box-shadow: 0 4px 10px rgba(0,0,0,0.2);
            margin-bottom: 6px;
        }
        .icon span { font-size: 11px; font-weight: 600; text-shadow: 0 2px 4px rgba(0,0,0,0.5); text-align: center; }

        #taskbar {
            position: fixed; bottom: 12px; left: 50%; transform: translateX(-50%);
            height: 64px; background: rgba(15, 23, 42, 0.8); backdrop-filter: blur(30px);
            border: 1px solid var(--glass-border); border-radius: 20px;
            display: flex; align-items: center; padding: 0 16px; gap: 8px; z-index: 9999;
            box-shadow: 0 10px 30px rgba(0,0,0,0.4);
        }

        .dock-item {
            width: 44px; height: 44px; border-radius: 12px;
            display: flex; align-items: center; justify-content: center;
            font-size: 24px; cursor: pointer; transition: 0.2s;
            position: relative;
        }
        .dock-item:hover { background: rgba(255,255,255,0.15); transform: translateY(-5px); }
        .dock-item.active::after {
            content: ''; position: absolute; bottom: -4px; width: 4px; height: 4px;
            background: var(--accent-color); border-radius: 50%;
        }

        /* App Specifics */
        .terminal-box { font-family: 'JetBrains Mono', monospace; background: rgba(0,0,0,0.9); padding: 10px; color: #4ade80; height: 100%; overflow: auto; }
        .editor { width: 100%; height: 100%; background: white; color: black; padding: 20px; outline: none; overflow-y: auto; }
        iframe { border: none; width: 100%; height: 100%; background: white; }
    </style>
</head>
<body>

    <div id="wallpaper">
        <div class="blob bg-purple-600 w-[500px] h-[500px] top-[-10%] left-[-10%]"></div>
        <div class="blob bg-blue-600 w-[400px] h-[400px] bottom-[10%] right-[-5%]"></div>
    </div>

    <div id="desktop"></div>
    <div id="window-layer"></div>

    <div id="taskbar">
        <div class="dock-item bg-white/10" onclick="launchApp('store')">💎</div>
        <div class="w-[1px] h-6 bg-white/20 mx-1"></div>
        <div id="dock-apps" class="flex gap-2"></div>
        <div class="w-[1px] h-6 bg-white/20 mx-1"></div>
        <div class="text-right leading-tight pr-2">
            <div id="clock" class="text-xs font-bold">12:00</div>
            <div id="date" class="text-[9px] opacity-60">DEC 18</div>
        </div>
    </div>

    <script>
        const APP_DB = {
            store: { name: 'Marketplace', icon: '🛍️', installed: true, type: 'native' },
            settings: { name: 'Settings', icon: '⚙️', installed: true, type: 'native' },
            files: { name: 'Files', icon: '📂', installed: true, type: 'native' },
            browser: { name: 'Browser', icon: '🌐', installed: true, type: 'native' },
            terminal: { name: 'Terminal', icon: '💻', installed: true, type: 'native' },
            calc: { name: 'Calculator', icon: '🧮', installed: true, type: 'native' },
            notes: { name: 'Notes', icon: '📝', installed: true, type: 'native' },
            mail: { name: 'Mail', icon: '✉️', installed: true, type: 'native' },
            calendar: { name: 'Calendar', icon: '📅', installed: true, type: 'native' },
            docs: { name: 'Docs', icon: '📄', installed: true, type: 'native' },
            sheets: { name: 'Sheets', icon: '📊', installed: true, type: 'native' },
            slides: { name: 'Slides', icon: '📽️', installed: true, type: 'native' },
            paint: { name: 'Paint', icon: '🎨', installed: true, type: 'native' },
            youtube: { name: 'YouTube', icon: '📺', installed: true, type: 'iframe', url: 'https://613tube.com' },
            selenite: { name: 'Selenite', icon: '🌑', installed: true, type: 'iframe', url: 'https://selenite.cc' },
            frogies: { name: 'Frogies', icon: '🐸', installed: true, type: 'iframe', url: 'https://frogiesarcade.com' },
            snake: { name: 'Snake', icon: '🐍', installed: false, type: 'game' },
            tetris: { name: 'Tetris', icon: '🧱', installed: false, type: 'game' }
        };

        let state = {
            windows: [], // Array of IDs
            zIndex: 100,
            draggingId: null,
            dragOffset: {x:0, y:0}
        };

        function init() {
            renderDesktop();
            renderDock();
            updateClock();
            setInterval(updateClock, 1000);

            // Global Pointer Events for robust dragging
            window.addEventListener('pointermove', onPointerMove);
            window.addEventListener('pointerup', onPointerUp);
        }

        function renderDesktop() {
            const el = document.getElementById('desktop');
            el.innerHTML = '';
            Object.entries(APP_DB).forEach(([id, app]) => {
                if(!app.installed) return;
                const div = document.createElement('div');
                div.className = 'icon';
                div.onclick = () => launchApp(id);
                div.innerHTML = `<div class="icon-box">${app.icon}</div><span>${app.name}</span>`;
                el.appendChild(div);
            });
        }

        function renderDock() {
            const el = document.getElementById('dock-apps');
            el.innerHTML = '';
            const pinned = ['browser', 'mail', 'docs'];
            const unique = [...new Set([...pinned, ...state.windows])];
            
            unique.forEach(id => {
                if(!APP_DB[id]) return;
                const div = document.createElement('div');
                div.className = `dock-item ${state.windows.includes(id) ? 'active' : ''}`;
                div.innerHTML = APP_DB[id].icon;
                div.onclick = () => launchApp(id);
                el.appendChild(div);
            });
        }

        function launchApp(id) {
            if(document.getElementById(`win-${id}`)) {
                restoreWindow(id);
                return;
            }

            const app = APP_DB[id];
            const win = document.createElement('div');
            win.id = `win-${id}`;
            win.className = 'window active';
            win.style.left = `${100 + (state.windows.length * 30)}px`;
            win.style.top = `${50 + (state.windows.length * 30)}px`;
            win.style.width = '800px'; win.style.height = '550px';
            win.style.zIndex = ++state.zIndex;

            // Header with pointerdown for dragging
            win.innerHTML = `
                <div class="title-bar" onpointerdown="startDrag(event, '${id}')">
                    <div class="win-controls">
                        <button class="win-btn close" onpointerdown="stopProp(event)" onclick="closeWindow('${id}')"></button>
                        <button class="win-btn min" onpointerdown="stopProp(event)" onclick="minWindow('${id}')"></button>
                        <button class="win-btn max" onpointerdown="stopProp(event)" onclick="maxWindow('${id}')"></button>
                    </div>
                    <div class="flex-1 text-center text-xs font-bold uppercase tracking-widest opacity-60 pointer-events-none">${app.name}</div>
                    <div style="width: 40px"></div>
                </div>
                <div class="flex-1 overflow-hidden relative flex flex-col bg-black/20">
                    ${getAppContent(id, app)}
                </div>
            `;

            document.getElementById('window-layer').appendChild(win);
            state.windows.push(id);
            renderDock();

            if(id === 'paint') initPaint();
            if(id === 'snake') initSnake();
        }

        function getAppContent(id, app) {
            if(app.type === 'iframe') return `<iframe src="${app.url}"></iframe>`;

            switch(id) {
                case 'store': return `
                    <div class="p-6 h-full overflow-y-auto">
                        <h2 class="text-2xl font-bold mb-4">App Store</h2>
                        <div class="grid grid-cols-2 gap-4">
                            ${Object.entries(APP_DB).filter(a => !a[1].installed).map(([k, a]) => `
                                <div class="bg-white/5 p-4 rounded-xl flex items-center justify-between border border-white/5">
                                    <div class="flex items-center gap-3">
                                        <div class="text-3xl">${a.icon}</div>
                                        <div class="font-bold text-sm">${a.name}</div>
                                    </div>
                                    <button onclick="installApp('${k}', this)" class="bg-blue-600 px-4 py-1 rounded-full text-xs font-bold">GET</button>
                                </div>
                            `).join('') || '<div class="col-span-2 text-center opacity-50">All apps installed</div>'}
                        </div>
                    </div>`;
                case 'browser': return `<div class="flex flex-col h-full bg-white"><div class="p-2 bg-gray-100 flex gap-2"><input id="url-${id}" class="flex-1 bg-white border px-2 text-sm text-black" value="https://www.bing.com"><button class="text-black text-xs px-2" onclick="document.getElementById('fr-${id}').src=document.getElementById('url-${id}').value">Go</button></div><iframe id="fr-${id}" src="https://www.bing.com" class="flex-1"></iframe></div>`;
                case 'terminal': return `<div class="terminal-box" id="term-out"><div>CrystalOS Shell</div><br><div class="flex gap-1"><span>$</span><input class="bg-transparent outline-none flex-1 border-none text-green-400" onkeydown="handleTerm(event)"></div></div>`;
                case 'calc': return `<div class="p-4 grid grid-cols-4 gap-2 h-full content-center bg-gray-900"><div id="calc-d" class="col-span-4 bg-black/40 text-right p-4 text-2xl mb-2 font-mono">0</div>${['C','/','*','-','7','8','9','+','4','5','6','=','1','2','3','0'].map(k=>`<button onclick="calc('${k}')" class="bg-white/10 p-3 rounded hover:bg-white/20">${k}</button>`).join('')}</div>`;
                case 'docs': return `<div class="flex flex-col h-full bg-white text-black"><div class="p-2 border-b bg-gray-50"><button class="font-bold px-2">B</button><button class="italic px-2">I</button></div><div class="editor" contenteditable="true"><h1>Document</h1><p>Type here...</p></div></div>`;
                case 'mail': return `<div class="flex h-full"><div class="w-48 bg-white/5 border-r border-white/10 p-2"><div class="p-2 bg-blue-600 rounded text-center text-sm font-bold mb-2">Compose</div><div class="p-2 hover:bg-white/5 rounded text-sm">Inbox (1)</div></div><div class="flex-1 p-6"><h2 class="text-xl font-bold">Welcome</h2><p class="text-sm opacity-60 mt-2">Welcome to CrystalOS Mail.</p></div></div>`;
                case 'paint': return `<canvas id="c-paint" class="w-full h-full bg-white cursor-crosshair"></canvas>`;
                case 'snake': return `<canvas id="c-snake" class="w-full h-full bg-black"></canvas>`;
                default: return `<div class="p-10 text-center opacity-50">App Loaded</div>`;
            }
        }

        // --- Logic ---
        function stopProp(e) { e.stopPropagation(); }

        function startDrag(e, id) {
            const win = document.getElementById(`win-${id}`);
            win.style.zIndex = ++state.zIndex;
            // Highlight window
            document.querySelectorAll('.window').forEach(w => w.classList.remove('active'));
            win.classList.add('active');
            
            const rect = win.getBoundingClientRect();
            state.draggingId = id;
            state.dragOffset = { 
                x: (e.clientX || e.touches[0].clientX) - rect.left, 
                y: (e.clientY || e.touches[0].clientY) - rect.top 
            };
            
            // Required for touch dragging to not scroll page
            if(e.type === 'touchstart') document.body.style.overflow = 'hidden';
        }

        function onPointerMove(e) {
            if(!state.draggingId) return;
            const win = document.getElementById(`win-${state.draggingId}`);
            if(!win) return;
            
            const clientX = e.clientX || (e.touches ? e.touches[0].clientX : 0);
            const clientY = e.clientY || (e.touches ? e.touches[0].clientY : 0);
            
            win.style.left = (clientX - state.dragOffset.x) + 'px';
            win.style.top = (clientY - state.dragOffset.y) + 'px';
        }

        function onPointerUp() {
            state.draggingId = null;
            document.body.style.overflow = 'hidden'; // Restore
        }

        // Window Controls
        function closeWindow(id) {
            document.getElementById(`win-${id}`).remove();
            state.windows = state.windows.filter(x => x !== id);
            renderDock();
        }
        function minWindow(id) { document.getElementById(`win-${id}`).classList.add('minimized'); }
        function maxWindow(id) { document.getElementById(`win-${id}`).classList.toggle('maximized'); }
        function restoreWindow(id) {
            const win = document.getElementById(`win-${id}`);
            win.classList.remove('minimized');
            win.style.zIndex = ++state.zIndex;
        }

        // Features
        function installApp(id, btn) {
            btn.innerText = '...';
            setTimeout(() => {
                APP_DB[id].installed = true;
                renderDesktop();
                launchApp(id); // Auto launch
            }, 800);
        }

        function calc(k) {
            const d = document.getElementById('calc-d');
            if(k==='C') d.innerText='0';
            else if(k==='=') { try{d.innerText=eval(d.innerText)}catch{d.innerText='Err'} }
            else d.innerText = d.innerText==='0'?k:d.innerText+k;
        }

        function handleTerm(e) {
            if(e.key === 'Enter') {
                const cmd = e.target.value.trim();
                const out = document.getElementById('term-out');
                out.innerHTML += `<div>$ ${cmd}</div>`;
                if(cmd === 'help') out.innerHTML += `<div class="opacity-60">ls, date, clear, echo</div>`;
                else if(cmd === 'date') out.innerHTML += `<div>${new Date().toString()}</div>`;
                else if(cmd === 'clear') out.innerHTML = '<div>CrystalOS Shell</div>';
                else out.innerHTML += `<div class="text-red-400">Not found</div>`;
                e.target.value = '';
            }
        }

        function initPaint() {
            const c = document.getElementById('c-paint');
            if(!c) return;
            const ctx = c.getContext('2d');
            c.width = c.clientWidth; c.height = c.clientHeight;
            let d = false;
            c.onpointerdown = e => { d=true; ctx.beginPath(); ctx.moveTo(e.offsetX, e.offsetY); };
            c.onpointermove = e => { if(d) { ctx.lineTo(e.offsetX, e.offsetY); ctx.stroke(); } };
            c.onpointerup = () => d=false;
        }

        function updateClock() {
            const d = new Date();
            document.getElementById('clock').innerText = d.toLocaleTimeString([], {hour:'2-digit', minute:'2-digit'});
            document.getElementById('date').innerText = d.toLocaleDateString([], {month:'short', day:'numeric'}).toUpperCase();
        }

        window.onload = init;
    </script>
</body>
</html>
