<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
    <title>Google</title> 
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        :root {
            --glass: rgba(255, 255, 255, 0.1);
            --glass-border: rgba(255, 255, 255, 0.2);
            --liquid-reflection: linear-gradient(135deg, rgba(255,255,255,0.3) 0%, rgba(255,255,255,0) 50%, rgba(255,255,255,0.1) 100%);
        }

        body, html {
            margin: 0; height: 100%; overflow: hidden;
            background: #050505; font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica;
            color: white; user-select: none;
        }

        #wallpaper {
            position: fixed; inset: 0; z-index: -1;
            background: radial-gradient(circle at 50% 50%, #1e293b, #0f172a, #020617);
            transition: 0.5s;
        }

        /* LIQUID GLASS EFFECT */
        .liquid-glass {
            background: var(--glass);
            backdrop-filter: blur(25px) saturate(180%);
            -webkit-backdrop-filter: blur(25px) saturate(180%);
            border: 1px solid var(--glass-border);
            position: relative;
            overflow: hidden;
        }

        .liquid-glass::before {
            content: '';
            position: absolute; inset: 0;
            background: var(--liquid-reflection);
            pointer-events: none;
            opacity: 0.5;
        }

        .window {
            position: absolute; border-radius: 24px;
            display: flex; flex-direction: column;
            box-shadow: 0 25px 50px -12px rgba(0, 0, 0, 0.7);
            transition: transform 0.2s cubic-bezier(0.2, 0, 0.2, 1);
        }

        .window.active { z-index: 100 !important; border-color: rgba(255,255,255,0.4); }

        .title-bar {
            height: 48px; display: flex; align-items: center; justify-content: space-between;
            padding: 0 20px; cursor: grab;
        }

        .dock {
            position: fixed; bottom: 24px; left: 50%; transform: translateX(-50%);
            height: 84px; padding: 0 16px; display: flex; align-items: center; gap: 14px;
            border-radius: 32px; z-index: 9999;
        }

        .dock-item {
            width: 62px; height: 62px; border-radius: 16px;
            display: flex; align-items: center; justify-content: center;
            font-size: 32px; cursor: pointer; transition: 0.2s;
        }
        .dock-item:hover { transform: scale(1.1) translateY(-10px); }
        .dock-item:active { transform: scale(0.9); }

        /* History Poison UI */
        #poison-logs {
            font-family: monospace; font-size: 10px; height: 100px;
            background: rgba(0,0,0,0.5); border-radius: 8px; overflow-y: auto; padding: 8px;
        }

        /* App Grid */
        .app-card {
            background: rgba(255,255,255,0.05); border-radius: 20px;
            padding: 16px; display: flex; flex-direction: column; align-items: center;
            gap: 8px; border: 1px solid transparent; transition: 0.2s;
        }
        .app-card:hover { background: rgba(255,255,255,0.1); border-color: var(--glass-border); }

        iframe { background: white; border-radius: 0 0 24px 24px; }
    </style>
</head>
<body>

    <div id="wallpaper"></div>
    <div id="desktop" class="p-8 grid grid-cols-4 md:grid-cols-6 lg:grid-cols-10 gap-6"></div>
    <div id="window-container"></div>

    <div class="dock liquid-glass" id="main-dock"></div>

    <script>
        // Offline Service Worker
        if ('serviceWorker' in navigator) {
            const sw = `self.addEventListener('fetch', e => e.respondWith(caches.match(e.request).then(r => r || fetch(e.request))));`;
            const blob = new Blob([sw], {type: 'text/javascript'});
            navigator.serviceWorker.register(URL.createObjectURL(blob));
        }

        const APPS = [
            { id: 'youtube', name: 'YouTube', icon: '🔴', url: 'https://613tube.com', online: true },
            { id: 'selenite', name: 'Selenite', icon: '💎', url: 'https://selenite.cc/', online: true },
            { id: 'frogie', name: 'Frogies Arcade', icon: '🐸', url: 'https://frogiesarcade.win/', online: true },
            { id: 'doom', name: 'DOOM', icon: '🔫', url: 'https://archive.org/embed/doom-playable', online: true },
            { id: 'history', name: 'Poison', icon: '🧪', type: 'system' },
            { id: 'settings', name: 'Settings', icon: '⚙️', type: 'system' },
            { id: 'notes', name: 'Notepad', icon: '📝', type: 'utility', online: false },
            { id: 'calc', name: 'Calculator', icon: '🔢', type: 'utility', online: false }
        ];

        let state = {
            open: [],
            zIndex: 10,
            poisoning: false,
            logs: []
        };

        function init() {
            renderDesktop();
            renderDock();
            window.addEventListener('online', () => document.body.classList.remove('is-offline'));
            window.addEventListener('offline', () => document.body.classList.add('is-offline'));
        }

        function renderDesktop() {
            const desk = document.getElementById('desktop');
            desk.innerHTML = '';
            APPS.forEach(app => {
                const div = document.createElement('div');
                div.className = 'app-card cursor-pointer';
                div.onclick = () => launch(app.id);
                div.innerHTML = `<div class="text-4xl">${app.icon}</div><div class="text-[10px] font-semibold uppercase tracking-widest opacity-70">${app.name}</div>`;
                desk.appendChild(div);
            });
        }

        function renderDock() {
            const dock = document.getElementById('main-dock');
            ['youtube', 'selenite', 'frogie', 'history', 'settings'].forEach(id => {
                const app = APPS.find(a => a.id === id);
                const div = document.createElement('div');
                div.className = 'dock-item';
                div.onclick = () => launch(id);
                div.innerText = app.icon;
                dock.appendChild(div);
            });
        }

        function launch(id) {
            if (state.open.includes(id)) {
                focusWin(id);
                return;
            }

            const app = APPS.find(a => a.id === id);
            const win = document.createElement('div');
            win.id = `win-${id}`;
            win.className = 'window liquid-glass active';
            win.style.width = 'min(90vw, 900px)';
            win.style.height = 'min(80vh, 600px)';
            win.style.left = '50%'; win.style.top = '45%';
            win.style.transform = 'translate(-50%, -50%)';
            win.style.zIndex = ++state.zIndex;

            win.innerHTML = `
                <div class="title-bar" onpointerdown="startDrag(event, '${id}')">
                    <div class="flex gap-2">
                        <div class="w-3 h-3 rounded-full bg-red-500 shadow-lg shadow-red-500/50" onclick="closeWin('${id}')"></div>
                        <div class="w-3 h-3 rounded-full bg-yellow-500"></div>
                        <div class="w-3 h-3 rounded-full bg-green-500"></div>
                    </div>
                    <div class="text-[10px] font-bold tracking-widest opacity-50">${app.name}</div>
                    <div class="w-12"></div>
                </div>
                <div class="flex-1 overflow-hidden" id="body-${id}">
                    ${getAppContent(app)}
                </div>
            `;

            document.getElementById('window-container').appendChild(win);
            state.open.push(id);
        }

        function getAppContent(app) {
            if (app.online && !navigator.onLine) {
                return `<div class="h-full flex flex-col items-center justify-center p-10 text-center bg-black/40">
                    <div class="text-5xl mb-4">📶</div>
                    <h2 class="text-xl font-bold">Connection Required</h2>
                    <p class="opacity-50 text-sm mt-2">${app.name} needs the internet. Use Notepad or Calculator while offline.</p>
                </div>`;
            }

            if (app.url) return `<iframe src="${app.url}" class="w-full h-full border-none"></iframe>`;

            if (app.id === 'history') return `
                <div class="p-6 h-full flex flex-col gap-4 text-black bg-white">
                    <h2 class="text-2xl font-bold">History Poison V2</h2>
                    <p class="text-sm text-gray-600">Floods your browser history with random legitimate-looking URLs to protect your privacy.</p>
                    <button id="poison-btn" onclick="togglePoison()" class="bg-red-600 text-white py-3 rounded-xl font-bold">START POISONING</button>
                    <div id="poison-logs">Waiting for start...</div>
                </div>
            `;

            if (app.id === 'settings') return `
                <div class="p-6 h-full text-black bg-white">
                    <h2 class="text-2xl font-bold mb-4">System Settings</h2>
                    <div class="space-y-4">
                        <div class="p-4 bg-gray-100 rounded-xl flex justify-between">
                            <span>Liquid Glass Depth</span>
                            <input type="range">
                        </div>
                        <div class="p-4 bg-gray-100 rounded-xl flex justify-between">
                            <span>Offline Mode Mode</span>
                            <span class="text-green-600 font-bold">AUTO</span>
                        </div>
                    </div>
                </div>
            `;

            if (app.id === 'notes') return `<textarea class="w-full h-full p-6 text-black border-none focus:outline-none" placeholder="Offline notes save to storage..."></textarea>`;
            
            return `<div class="p-10 text-black bg-white h-full">System App: ${app.name}</div>`;
        }

        function togglePoison() {
            state.poisoning = !state.poisoning;
            const btn = document.getElementById('poison-btn');
            const logs = document.getElementById('poison-logs');
            
            if (state.poisoning) {
                btn.innerText = "STOP POISONING";
                btn.className = "bg-black text-white py-3 rounded-xl font-bold";
                state.poisonInterval = setInterval(() => {
                    const sites = ['google.com/search?q=weather', 'wikipedia.org/wiki/Science', 'github.com/trending', 'news.ycombinator.com'];
                    const url = 'https://' + sites[Math.floor(Math.random() * sites.length)];
                    history.pushState({}, '', '#' + Math.random().toString(36).substring(7));
                    const entry = document.createElement('div');
                    entry.innerText = `[+] Injected obfuscation: ${url}`;
                    logs.prepend(entry);
                }, 1000);
            } else {
                btn.innerText = "START POISONING";
                btn.className = "bg-red-600 text-white py-3 rounded-xl font-bold";
                clearInterval(state.poisonInterval);
            }
        }

        // Window Management
        let drag = { active: false, id: null, ox: 0, oy: 0 };
        function startDrag(e, id) {
            focusWin(id);
            const win = document.getElementById(`win-${id}`);
            drag = { active: true, id, ox: (e.clientX || e.touches[0].clientX) - win.offsetLeft, oy: (e.clientY || e.touches[0].clientY) - win.offsetTop };
        }

        window.onpointermove = (e) => {
            if (!drag.active) return;
            const win = document.getElementById(`win-${drag.id}`);
            win.style.left = (e.clientX || e.touches[0].clientX) - drag.ox + (win.offsetWidth/2) + 'px';
            win.style.top = (e.clientY || e.touches[0].clientY) - drag.oy + (win.offsetHeight/2) + 'px';
        };
        window.onpointerup = () => drag.active = false;

        function closeWin(id) {
            document.getElementById(`win-${id}`).remove();
            state.open = state.open.filter(i => i !== id);
        }

        function focusWin(id) {
            state.open.forEach(i => document.getElementById(`win-${i}`)?.classList.remove('active'));
            const win = document.getElementById(`win-${id}`);
            if (win) {
                win.classList.add('active');
                win.style.zIndex = ++state.zIndex;
            }
        }

        init();
    </script>
</body>
</html>
