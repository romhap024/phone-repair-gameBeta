<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>3D Phone Repair Master - Beta 0.1</title>
    <!-- Three.js, OrbitControls & GSAP -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/controls/OrbitControls.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.2/gsap.min.js"></script>
    <style>
        :root {
            --bg-color: #010307;
            --panel-color: #060913;
            --card-color: #0b111e;
            --accent-color: #3b82f6;
            --accent-glow: rgba(59, 130, 246, 0.6);
            --text-color: #f8fafc;
            --text-muted: #64748b;
            --success-color: #10b981;
            --danger-color: #ef4444;
            --warning-color: #f59e0b;
            --purple-color: #8b5cf6;
        }

        * {
            box-sizing: border-box; margin: 0; padding: 0;
            user-select: none; -webkit-user-select: none;
            touch-action: manipulation;
            font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
        }

        body {
            background-color: var(--bg-color); color: var(--text-color);
            display: flex; justify-content: center; align-items: center;
            min-height: 100vh; padding: 4px; overflow: hidden;
            transition: background 0.5s ease;
        }

        body.theme-cyber { background: radial-gradient(circle, #1a0b2e 0%, #030108 100%); }
        body.theme-day { background: radial-gradient(circle, #1e293b 0%, #0f172a 100%); }

        /* Экран загрузки */
        #loaderOverlay {
            position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            background: #010307; z-index: 5000; display: flex; flex-direction: column;
            justify-content: center; align-items: center; gap: 16px;
            transition: opacity 0.5s ease, visibility 0.5s ease;
        }
        #loaderOverlay.fade-out { opacity: 0; visibility: hidden; pointer-events: none; }
        .loader-logo { font-size: 24px; font-weight: 800; color: var(--purple-color); text-shadow: 0 0 20px rgba(139, 92, 246, 0.6); animation: pulseLogo 1s infinite alternate; }
        @keyframes pulseLogo { from { transform: scale(1); filter: brightness(1); } to { transform: scale(1.05); filter: brightness(1.3); } }
        .loader-bar-bg { width: 220px; height: 6px; background: #0b111e; border-radius: 10px; overflow: hidden; border: 1px solid rgba(255,255,255,0.1); }
        .loader-bar-fill { width: 0%; height: 100%; background: linear-gradient(90deg, var(--accent-color), var(--purple-color)); transition: width 2s linear; }

        .overlay {
            position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(1, 3, 7, 0.96); backdrop-filter: blur(24px);
            z-index: 2000; display: flex; justify-content: center; align-items: center;
            opacity: 1; transition: opacity 0.3s ease, visibility 0.3s ease;
        }
        .overlay.hidden { opacity: 0; visibility: hidden; pointer-events: none; }
        #passwordOverlay { z-index: 3000 !important; }

        .auth-card, .menu-card, .upgrade-card, .password-card, .profile-card {
            background: var(--panel-color); border: 1px solid rgba(59, 130, 246, 0.4);
            padding: 24px; border-radius: 24px; text-align: center; width: 350px;
            box-shadow: 0 25px 60px rgba(0,0,0,0.95), inset 0 1px 0 rgba(255,255,255,0.1);
            display: flex; flex-direction: column; gap: 12px;
        }

        .auth-card input, .password-card input {
            width: 100%; padding: 12px; background: var(--card-color);
            border: 1px solid #1e293b; border-radius: 12px; color: #fff;
            font-size: 14px; text-align: center; outline: none; transition: border-color 0.2s;
        }
        .auth-card input:focus, .password-card input:focus { border-color: var(--accent-color); }

        .mode-btn {
            width: 100%; padding: 14px; border: none; border-radius: 14px;
            font-weight: 600; color: #fff; cursor: pointer; font-size: 13px;
            transition: transform 0.2s ease, filter 0.2s ease, box-shadow 0.2s ease;
            display: flex; align-items: center; justify-content: center; gap: 8px;
        }
        .mode-btn.orders { background: linear-gradient(135deg, #3b82f6, #1d4ed8); box-shadow: 0 4px 20px var(--accent-glow); }
        .mode-btn.sandbox { background: linear-gradient(135deg, #8b5cf6, #6d28d9); box-shadow: 0 4px 20px rgba(139, 92, 246, 0.4); }
        .mode-btn.upgrade { background: linear-gradient(135deg, #10b981, #047857); box-shadow: 0 4px 20px rgba(16, 185, 129, 0.4); }
        .mode-btn.profile { background: linear-gradient(135deg, #f59e0b, #d97706); box-shadow: 0 4px 20px rgba(245, 158, 11, 0.4); }
        .mode-btn:hover { transform: translateY(-2px); filter: brightness(1.15); }

        .game-wrapper {
            width: 100%; max-width: 460px; height: 98vh;
            background: var(--panel-color); border-radius: 24px;
            box-shadow: 0 30px 60px rgba(0,0,0,0.85), inset 0 1px 0 rgba(255,255,255,0.08);
            display: flex; flex-direction: column; overflow: hidden;
            border: 1px solid rgba(255,255,255,0.08); position: relative;
        }

        /* Метка Beta в правом нижнем углу */
        .beta-watermark {
            position: absolute; bottom: 6px; right: 12px;
            font-size: 10px; font-weight: 700; color: var(--text-muted);
            letter-spacing: 1px; pointer-events: none; z-index: 100;
            opacity: 0.7; text-transform: uppercase;
        }

        header {
            background: #03060d; padding: 10px 16px;
            display: flex; justify-content: space-between; align-items: center;
            border-bottom: 1px solid rgba(255,255,255,0.06);
        }
        .stats-group { display: flex; gap: 6px; align-items: center; }
        .stat-item {
            background: var(--card-color); padding: 5px 10px; border-radius: 30px;
            font-weight: 600; display: flex; align-items: center; gap: 6px; font-size: 11px;
            border: 1px solid rgba(255,255,255,0.06); cursor: pointer;
        }
        .stat-item:hover { border-color: var(--accent-color); }

        .main-layout { display: flex; flex-direction: column; height: 100%; overflow-y: auto; padding: 10px; gap: 8px; }

        .guide-panel, .order-panel, .sandbox-panel, .shop-panel {
            background: #040812; border: 1px solid rgba(245, 158, 11, 0.3);
            padding: 8px 12px; border-radius: 14px; font-size: 10px; display: flex; flex-direction: column; gap: 4px;
        }
        .order-panel { border-color: rgba(59, 130, 246, 0.3); }
        .sandbox-panel { border-color: rgba(139, 92, 246, 0.3); max-height: 160px; overflow-y: auto; }
        .shop-panel { border-color: rgba(16, 185, 129, 0.3); }

        .sandbox-row { display: flex; gap: 4px; justify-content: space-between; flex-wrap: wrap; }
        .sandbox-btn, .shop-btn {
            background: rgba(139, 92, 246, 0.15); color: var(--purple-color); border: 1px solid rgba(139, 92, 246, 0.3);
            padding: 5px 6px; border-radius: 8px; font-weight: 600; cursor: pointer; font-size: 8px; flex: 1; min-width: 30%;
            text-align: center; transition: all 0.2s ease;
        }
        .shop-btn { background: rgba(16, 185, 129, 0.15); color: var(--success-color); border-color: rgba(16, 185, 129, 0.3); }
        .sandbox-btn:hover, .shop-btn:hover { filter: brightness(1.2); }

        .table-area {
            position: relative; width: 100%; height: 240px;
            background: radial-gradient(circle, #10192e 0%, #020408 100%);
            border-radius: 16px; border: 1px solid rgba(255,255,255,0.08);
            overflow: hidden; flex-shrink: 0; box-shadow: inset 0 0 30px rgba(0,0,0,0.7);
        }

        #threeCanvas { width: 100%; height: 100%; display: block; cursor: grab; }
        #threeCanvas:active { cursor: grabbing; }

        .camera-controls {
            position: absolute; bottom: 8px; left: 50%; transform: translateX(-50%);
            display: flex; gap: 6px; z-index: 10;
        }
        .cam-btn {
            background: rgba(11, 17, 30, 0.9); backdrop-filter: blur(10px);
            border: 1px solid rgba(255,255,255,0.15); color: #fff;
            padding: 5px 9px; border-radius: 20px; font-size: 10px; font-weight: 600;
            cursor: pointer; transition: all 0.2s ease;
        }
        .cam-btn:hover { background: var(--accent-color); border-color: var(--accent-color); transform: scale(1.05); }

        .parts-tray {
            background: #03060d; border: 1px dashed #1e293b; border-radius: 12px;
            min-height: 40px; max-height: 52px; overflow-y: auto; padding: 6px; display: flex; flex-wrap: wrap; gap: 6px; align-items: center;
        }
        .tray-item {
            background: var(--card-color); border: 1px solid rgba(139, 92, 246, 0.5); padding: 4px 8px;
            border-radius: 8px; font-size: 9px; color: var(--purple-color); cursor: pointer; transition: transform 0.1s;
        }
        .tray-item:hover { transform: scale(1.05); background: rgba(139, 92, 246, 0.25); }

        .tools-grid { display: grid; grid-template-columns: repeat(5, 1fr); gap: 4px; }
        .tool-card {
            background: var(--card-color); border: 2px solid transparent; padding: 5px 2px;
            border-radius: 10px; display: flex; flex-direction: column; align-items: center; cursor: pointer;
            transition: all 0.2s ease;
        }
        .tool-card.active { border-color: var(--accent-color); background: rgba(59, 130, 246, 0.2); transform: scale(1.04); box-shadow: 0 0 15px var(--accent-glow); }
        .tool-icon { font-size: 15px; }
        .tool-name { font-size: 6px; font-weight: 600; text-align: center; color: var(--text-muted); margin-top: 2px; }

        .free-actions { display: grid; grid-template-columns: repeat(2, 1fr); gap: 8px; }
        .action-btn {
            background: var(--accent-color); color: #fff; border: none; padding: 10px;
            border-radius: 10px; font-size: 12px; font-weight: 600; cursor: pointer; text-align: center;
            transition: filter 0.2s, transform 0.1s; box-shadow: 0 4px 15px var(--accent-glow);
        }
        .action-btn.danger { background: var(--danger-color); box-shadow: 0 4px 15px rgba(239, 68, 68, 0.4); }
        .action-btn.success { background: var(--success-color); box-shadow: 0 4px 15px rgba(16, 185, 129, 0.4); }
        .action-btn:hover { filter: brightness(1.1); }
        .action-btn:active { transform: scale(0.98); }

        #toastContainer {
            position: fixed; top: 20px; left: 50%; transform: translateX(-50%);
            z-index: 4000; display: flex; flex-direction: column; gap: 8px; pointer-events: none;
        }
        .toast {
            background: rgba(11, 17, 30, 0.96); backdrop-filter: blur(14px);
            border: 1px solid rgba(59, 130, 246, 0.5); color: #fff; padding: 10px 20px;
            border-radius: 12px; font-size: 12px; font-weight: 600;
            box-shadow: 0 10px 30px rgba(0,0,0,0.8); animation: toastIn 0.3s ease, toastOut 0.3s ease 2.3s forwards;
        }
        @keyframes toastIn { from { opacity: 0; transform: translateY(-20px); } to { opacity: 1; transform: translateY(0); } }
        @keyframes toastOut { from { opacity: 1; transform: translateY(0); } to { opacity: 0; transform: translateY(-20px); } }
    </style>
</head>
<body class="theme-cyber">

<!-- Экран загрузки (2 секунды) -->
<div id="loaderOverlay">
    <div class="loader-logo">⚡ REPAIR MASTER 2026</div>
    <div class="loader-bar-bg">
        <div class="loader-bar-fill" id="loaderBar"></div>
    </div>
    <div style="font-size: 10px; color: var(--text-muted);">Загрузка лаборатории...</div>
</div>

<div id="toastContainer"></div>

<!-- Авторизация -->
<div id="authOverlay" class="overlay">
    <div class="auth-card">
        <h2>🔑 Вход / Регистрация</h2>
        <p style="font-size:11px; color:var(--text-muted);">Облачная система профилей</p>
        <input type="text" id="loginInput" placeholder="Логин мастера (мин. 3 символа)">
        <input type="password" id="passInput" placeholder="Пароль (мин. 4 символа)">
        <button class="action-btn success" onclick="loginOrRegister()">Войти в систему</button>
    </div>
</div>

<!-- Пароль песочницы -->
<div id="passwordOverlay" class="overlay hidden">
    <div class="password-card">
        <h3 style="color:var(--purple-color);">🔒 Пароль песочницы</h3>
        <p style="font-size:11px; color:var(--text-muted);">Введите секретный код доступа</p>
        <input type="password" id="sandboxPassInput" placeholder="Код доступа">
        <button class="action-btn success" onclick="verifySandboxPassword()">Подтвердить</button>
        <button class="action-btn danger" onclick="hideOverlay('passwordOverlay')">Отмена</button>
    </div>
</div>

<!-- Меню -->
<div id="modeSelectOverlay" class="overlay hidden">
    <div class="menu-card">
        <h2>📱 Repair Master Ultimate</h2>
        <p style="font-size:11px; color:var(--text-muted);" id="userWelcome">Привет, Мастер!</p>
        <button class="mode-btn orders" onclick="startOrdersMode()"><span>📋 Карьера (Заказы клиентов)</span></button>
        <button class="mode-btn sandbox" onclick="requestSandboxMode()"><span>🧪 Режим Песочницы (PRO)</span></button>
        <button class="mode-btn upgrade" onclick="openUpgradeModal()"><span>⭐ Магазин оборудования</span></button>
        <button class="mode-btn profile" onclick="openProfileModal()"><span>👤 Личный кабинет профиля</span></button>
        <div style="display:flex; gap:6px; margin-top:6px;">
            <button class="shop-btn" onclick="toggleTheme()">💡 Тема</button>
            <button class="action-btn danger" style="flex:1;" onclick="logout()">🚪 Сменить аккаунт</button>
        </div>
    </div>
</div>

<!-- Личный кабинет -->
<div id="profileOverlay" class="overlay hidden">
    <div class="profile-card">
        <h3 style="color:var(--warning-color);">👤 Кабинет Мастера</h3>
        <div style="background:var(--card-color); padding:12px; border-radius:12px; text-align:left; font-size:11px; display:flex; flex-direction:column; gap:6px;">
            <div><b>Имя мастера:</b> <span id="profName" style="color:var(--accent-color);">-</span></div>
            <div><b>Ранг:</b> <span id="profRank" style="color:var(--success-color);">-</span></div>
            <div><b>Баланс счета:</b> <span id="profMoney" style="color:var(--warning-color);">-</span></div>
            <div><b>Выполнено заказов:</b> <span id="profRepairs" style="color:var(--purple-color);">-</span></div>
            <div><b>Синхронизация с облаком:</b> <span style="color:var(--success-color);">Активна 🟢</span></div>
        </div>
        <button class="action-btn success" onclick="exportProfileData()">📤 Экспорт сейва</button>
        <button class="action-btn danger" onclick="closeProfileModal()">Закрыть</button>
    </div>
</div>

<!-- Магазин -->
<div id="upgradeOverlay" class="overlay hidden">
    <div class="upgrade-card">
        <h3 style="color:var(--success-color);">⭐ Апгрейды лаборатории</h3>
        <div style="display:flex; flex-direction:column; gap:8px; margin:10px 0;">
            <button class="shop-btn" onclick="buyUpgrade('turboHeat', 1000)">🔥 Турбо-Фен (+50% скорость) (1000 ₽)</button>
            <button class="shop-btn" onclick="buyUpgrade('proTools', 1500)">🪛 Премиум Отвертка (+20% к доходу) (1500 ₽)</button>
            <button class="shop-btn" onclick="buyUpgrade('solderStation', 2500)">⚡ Проф. Паяльная станция (2500 ₽)</button>
        </div>
        <button class="action-btn danger" onclick="closeUpgradeModal()">Закрыть</button>
    </div>
</div>

<div class="game-wrapper">
    <!-- Надпись Beta 0.1 в правом нижнем углу интерфейса игры -->
    <div class="beta-watermark">Beta 0.1</div>

    <header>
        <div style="font-weight: 700; font-size: 11px; color: var(--purple-color);" id="modeTitle">⚡ Next-Gen Teardown</div>
        <div class="stats-group">
            <button style="background:var(--card-color); border:1px solid rgba(255,255,255,0.1); color:#fff; border-radius:20px; padding:4px 9px; font-size:10px; cursor:pointer;" onclick="exitToMenu()">🚪 Меню</button>
            <div class="stat-item" onclick="openProfileModal()"><span>💰</span><span id="moneyText">2500 ₽</span></div>
        </div>
    </header>

    <div class="main-layout">
        <div class="guide-panel" id="guidePanel">
            <div style="font-weight:600; color:var(--warning-color); display:flex; justify-content:space-between;">
                <span>📘 Инструкция</span>
                <span id="guideStep" style="font-size:9px; color:var(--accent-color);">Шаг 1 из 4</span>
            </div>
            <div id="guideText" style="color:var(--text-muted); font-size:10px;">1. Прогрейте крышку Феном (🔥) и снимите её.<br>2. Выкрутите винты Отверткой (🪛) и включите ОС (⚡).</div>
        </div>

        <div class="order-panel" id="orderPanel">
            <div style="display:flex; justify-content:space-between; font-weight:600; color:var(--accent-color);">
                <span id="clientName">Клиент: Артем</span>
                <span id="orderReward" style="color:var(--success-color);">Награда: +3500 ₽</span>
            </div>
            <div class="order-desc" id="orderDesc" style="color:var(--text-muted); font-size:10px;">"У меня смартфон, экран разбит!"</div>
        </div>

        <div class="sandbox-panel" id="sandboxPanel" style="display:none;">
            <div style="font-weight: 600; color: var(--purple-color);">⚙️ Песочница (Выбор моделей):</div>
            <div class="sandbox-row">
                <button class="sandbox-btn" onclick="changePhoneModel('iqoo13')">iQOO 13</button>
                <button class="sandbox-btn" onclick="changePhoneModel('redmi15')">Redmi 15 Pro</button>
                <button class="sandbox-btn" onclick="changePhoneModel('realmegt8')">Realme GT 8</button>
            </div>
            <div class="sandbox-row" style="margin-top:4px;">
                <button class="sandbox-btn" onclick="setState('broken_screen')">💥 Битый Экран</button>
                <button class="sandbox-btn" onclick="setState('swollen_battery')">🔋 Вздуть АКБ</button>
                <button class="sandbox-btn" onclick="setState('short_circuit')">⚡ КЗ на плате</button>
            </div>
            <div class="sandbox-row" style="margin-top:4px;">
                <button class="sandbox-btn" style="background:var(--success-color); color:#fff;" onclick="toggleInfiniteMoney()">💰 Беск. $</button>
                <button class="sandbox-btn" style="background:var(--warning-color); color:#fff;" onclick="randomizeColor()">🎨 Цвет</button>
                <button class="sandbox-btn" style="background:var(--accent-color); color:#fff;" onclick="powerButtonPress()">⚡ Вкл/Выкл</button>
            </div>
        </div>

        <div class="shop-panel">
            <div style="font-weight: 600; color: var(--success-color);">🛒 Закупка запчастей:</div>
            <div class="sandbox-row">
                <button class="shop-btn" onclick="buyPart('Новый Дисплей', 1500)">📱 Дисплей (1500₽)</button>
                <button class="shop-btn" onclick="buyPart('Новый АКБ', 1000)">🔋 АКБ (1000₽)</button>
                <button class="shop-btn" onclick="buyPart('Изопропиловый Спирт', 300)">🧪 Спирт (300₽)</button>
            </div>
        </div>

        <div class="table-area">
            <canvas id="threeCanvas"></canvas>
            <div class="camera-controls">
                <button class="cam-btn" onclick="smoothRotateTo(Math.PI)">🔄 Крышка</button>
                <button class="cam-btn" onclick="smoothRotateTo(0)">📱 Экран</button>
                <button class="cam-btn" onclick="powerButtonPress()">⚡ Питание</button>
                <button class="cam-btn" onclick="spin360()">🌀 360°</button>
            </div>
        </div>

        <div>
            <div style="font-size: 10px; font-weight: 600; margin-bottom: 3px; color: var(--text-muted);">📦 Лоток деталей:</div>
            <div class="parts-tray" id="partsTray"><span style="font-size: 9px; color: var(--text-muted);">Лоток пуст</span></div>
        </div>

        <div class="tools-grid">
            <div class="tool-card active" onclick="selectTool('hand', this)"><div class="tool-icon">✋</div><div class="tool-name">Рука</div></div>
            <div class="tool-card" onclick="selectTool('heatgun', this)"><div class="tool-icon">🔥</div><div class="tool-name">Фен</div></div>
            <div class="tool-card" onclick="selectTool('screwdriver', this)"><div class="tool-icon">🪛</div><div class="tool-name">Отв.</div></div>
            <div class="tool-card" onclick="selectTool('tweezers', this)"><div class="tool-icon">✂️</div><div class="tool-name">Пинц.</div></div>
            <div class="tool-card" onclick="selectTool('multimeter', this)"><div class="tool-icon">⚡</div><div class="tool-name">Тестер</div></div>
        </div>

        <div class="free-actions">
            <button class="action-btn success" id="finishBtn" onclick="finishOrder()">📋 Сдать заказ</button>
            <button class="action-btn danger" onclick="resetPhone()">🔄 Сбросить</button>
        </div>
    </div>
</div>

<script>
    // Запуск 2-секундной анимации загрузки при открытии
    window.addEventListener('load', () => {
        setTimeout(() => {
            document.getElementById('loaderBar').style.width = '100%';
        }, 50);
        setTimeout(() => {
            document.getElementById('loaderOverlay').classList.add('fade-out');
        }, 2000);
    });

    let audioCtx = null, audioEnabled = true;
    function initAudio() {
        if (!audioCtx) audioCtx = new (window.AudioContext || window.webkitAudioContext)();
    }

    function playSound(type) {
        if (!audioEnabled || !audioCtx) return;
        try {
            if (audioCtx.state === 'suspended') audioCtx.resume();
            let osc = audioCtx.createOscillator(), gain = audioCtx.createGain();
            osc.connect(gain); gain.connect(audioCtx.destination);
            let now = audioCtx.currentTime;

            if (type === 'click') {
                osc.type = 'sine'; osc.frequency.setValueAtTime(600, now); osc.frequency.exponentialRampToValueAtTime(200, now + 0.04);
                gain.gain.setValueAtTime(0.1, now); gain.gain.linearRampToValueAtTime(0.01, now + 0.04);
                osc.start(now); osc.stop(now + 0.04);
            } else if (type === 'screw') {
                osc.type = 'triangle'; osc.frequency.setValueAtTime(350, now); osc.frequency.linearRampToValueAtTime(120, now + 0.07);
                gain.gain.setValueAtTime(0.12, now); gain.gain.linearRampToValueAtTime(0.01, now + 0.07);
                osc.start(now); osc.stop(now + 0.07);
            } else if (type === 'success') {
                osc.type = 'sine'; osc.frequency.setValueAtTime(523.25, now); osc.frequency.setValueAtTime(659.25, now + 0.1);
                gain.gain.setValueAtTime(0.2, now); gain.gain.linearRampToValueAtTime(0.01, now + 0.3);
                osc.start(now); osc.stop(now + 0.3);
            }
        } catch(e) {}
    }

    function showToast(text) {
        let container = document.getElementById('toastContainer');
        let toast = document.createElement('div');
        toast.className = 'toast'; toast.innerText = text;
        container.appendChild(toast);
        setTimeout(() => toast.remove(), 2300);
    }

    function toggleTheme() {
        let body = document.body;
        if (body.classList.contains('theme-cyber')) {
            body.classList.remove('theme-cyber'); body.classList.add('theme-day');
            showToast("💡 Тема: Дневная лаборатория");
        } else {
            body.classList.remove('theme-day'); body.classList.add('theme-cyber');
            showToast("💡 Тема: Киберпанк");
        }
    }

    let scene, camera, renderer, controls;
    let phoneGroup;
    let backCoverMesh, screenMesh, midFrameShieldMesh, batteryMesh, mainBoardMesh, powerButtonMesh;
    let screwsGroup = [];
    let raycaster = new THREE.Raycaster();
    let mouse = new THREE.Vector2();

    let currentModelKey = 'iqoo13';
    let shieldRemoved = false, screwsCount = 4, hoveredScrew = null;
    let isPhoneBooting = false, isPhoneOn = false, batteryConnected = true, batteryLevel = 87;

    const Z_POS = { CHASSIS: 0, VAPOR_CHAMBER: -0.12, BOARD_AND_BATT: -0.18, MID_SHIELD: -0.26, BACK_COVER: -0.34, SCREEN: 0.22 };

    function createGlassTexture(colorHex, text) {
        const canvas = document.createElement('canvas'); canvas.width = 1024; canvas.height = 2048;
        const ctx = canvas.getContext('2d');
        const grad = ctx.createLinearGradient(0, 0, 1024, 2048);
        grad.addColorStop(0, colorHex); grad.addColorStop(0.5, '#0b1329'); grad.addColorStop(1, '#010307');
        ctx.fillStyle = grad; ctx.fillRect(0, 0, 1024, 2048);
        for (let i = 0; i < 30000; i++) {
            ctx.fillStyle = `rgba(255, 255, 255, ${Math.random() * 0.03})`;
            ctx.fillRect(Math.random() * 1024, Math.random() * 2048, 1.5, 1.5);
        }
        ctx.font = '900 64px system-ui, sans-serif'; ctx.fillStyle = '#f8fafc'; ctx.textAlign = 'center';
        ctx.shadowColor = 'rgba(0, 0, 0, 0.8)'; ctx.shadowBlur = 16;
        ctx.fillText(text, 512, 1840);
        return new THREE.CanvasTexture(canvas);
    }

    function createBatteryTexture(brandName) {
        const canvas = document.createElement('canvas'); canvas.width = 1024; canvas.height = 1024;
        const ctx = canvas.getContext('2d');
        ctx.fillStyle = '#0f172a'; ctx.fillRect(0, 0, 1024, 1024);
        ctx.fillStyle = '#ef4444'; ctx.fillRect(40, 40, 240, 70);
        ctx.font = 'bold 36px sans-serif'; ctx.fillStyle = '#ffffff'; ctx.fillText('PULL TO REMOVE', 55, 88);
        ctx.font = 'bold 44px sans-serif'; ctx.fillText(`${brandName} Li-ion BATTERY`, 60, 220);
        ctx.font = '32px monospace'; ctx.fillStyle = '#cbd5e1';
        ctx.fillText('Capacity: 5500 mAh / 21.2 Wh', 60, 290);
        ctx.fillText('Nominal Voltage: 3.87 V ==', 60, 350);
        ctx.fillStyle = '#f59e0b'; ctx.font = 'bold 30px sans-serif';
        ctx.fillText('⚠️ WARNING: DO NOT PUNCTURE OR HEAT.', 60, 460);
        return new THREE.CanvasTexture(canvas);
    }

    function createBoardTexture() {
        const canvas = document.createElement('canvas'); canvas.width = 1024; canvas.height = 1024;
        const ctx = canvas.getContext('2d');
        ctx.fillStyle = '#03251c'; ctx.fillRect(0, 0, 1024, 1024);
        ctx.strokeStyle = '#059669'; ctx.lineWidth = 2;
        for (let i = 0; i < 100; i++) {
            ctx.beginPath(); let x = Math.random() * 1024, y = Math.random() * 1024;
            ctx.moveTo(x, y); ctx.lineTo(x + (Math.random()-0.5)*180, y + (Math.random()-0.5)*180); ctx.stroke();
        }
        ctx.fillStyle = '#f8fafc'; ctx.font = 'bold 36px monospace';
        ctx.fillText('HDI MAINBOARD v3.2', 80, 100);
        return new THREE.CanvasTexture(canvas);
    }

    function createOSScreenTexture(state, progress = 0) {
        const canvas = document.createElement('canvas'); canvas.width = 512; canvas.height = 1024;
        const ctx = canvas.getContext('2d');

        if (state === 'off') {
            ctx.fillStyle = '#050505'; ctx.fillRect(0, 0, 512, 1024);
        } else if (state === 'boot') {
            ctx.fillStyle = '#020617'; ctx.fillRect(0, 0, 512, 1024);
            ctx.font = '900 48px system-ui, sans-serif'; ctx.fillStyle = '#3b82f6';
            ctx.textAlign = 'center'; ctx.fillText('SYSTEM BOOT', 256, 450);
            ctx.strokeStyle = '#1e293b'; ctx.lineWidth = 10; ctx.lineCap = 'round';
            ctx.beginPath(); ctx.moveTo(100, 600); ctx.lineTo(412, 600); ctx.stroke();
            ctx.strokeStyle = '#3b82f6';
            ctx.beginPath(); ctx.moveTo(100, 600); ctx.lineTo(100 + (312 * progress), 600); ctx.stroke();
        } else if (state === 'on') {
            const bgGrad = ctx.createLinearGradient(0, 0, 512, 1024);
            bgGrad.addColorStop(0, '#0f172a'); bgGrad.addColorStop(1, '#311042');
            ctx.fillStyle = bgGrad; ctx.fillRect(0, 0, 512, 1024);
            ctx.fillStyle = 'rgba(0,0,0,0.4)'; ctx.fillRect(0, 0, 512, 50);
            ctx.font = 'bold 20px system-ui, sans-serif'; ctx.fillStyle = '#ffffff'; ctx.textAlign = 'left';
            ctx.fillText('20:45', 24, 34);
            ctx.textAlign = 'right'; ctx.fillText(`5G 📶  ${batteryLevel}% 🔋`, 488, 34);
            ctx.textAlign = 'center'; ctx.font = '200 80px system-ui, sans-serif';
            ctx.fillText('20:45', 256, 220);
        }
        return new THREE.CanvasTexture(canvas);
    }

    function init3D() {
        const container = document.querySelector('.table-area');
        scene = new THREE.Scene();
        camera = new THREE.PerspectiveCamera(45, container.clientWidth / container.clientHeight, 0.1, 1000);
        camera.position.set(0, 0, 18);

        renderer = new THREE.WebGLRenderer({ canvas: document.getElementById('threeCanvas'), antialias: true, alpha: true });
        renderer.setSize(container.clientWidth, container.clientHeight);
        renderer.setPixelRatio(window.devicePixelRatio);
        renderer.shadowMap.enabled = true;

        controls = new THREE.OrbitControls(camera, renderer.domElement);
        controls.enableDamping = true; controls.dampingFactor = 0.05;

        scene.add(new THREE.AmbientLight(0xffffff, 1.3));
        const dirLight = new THREE.DirectionalLight(0xffffff, 1.8);
        dirLight.position.set(5, 10, 7);
        scene.add(dirLight);

        buildPhone3DModel(currentModelKey);
        animate3D();

        renderer.domElement.addEventListener('pointerdown', on3DClick);
        renderer.domElement.addEventListener('pointermove', on3DPointerMove);
    }

    function buildPhone3DModel(modelKey) {
        if (phoneGroup) scene.remove(phoneGroup);
        currentModelKey = modelKey;
        shieldRemoved = false; screwsCount = 4; screwsGroup = []; batteryConnected = true;
        phoneGroup = new THREE.Group();

        let chassisColor = 0x1e293b, backColorStr = '#0b1120', brandText = "iQOO 13", batteryBrand = "iQOO";
        if (modelKey === 'redmi15') { chassisColor = 0x1e293b; backColorStr = '#0284c7'; brandText = "Redmi Note 15"; batteryBrand = "Redmi"; }
        else if (modelKey === 'realmegt8') { chassisColor = 0x18181b; backColorStr = '#7c3aed'; brandText = "Realme GT 8"; batteryBrand = "Realme"; }

        const vcGeo = new THREE.BoxGeometry(4.2, 8.5, 0.04);
        const vcMat = new THREE.MeshStandardMaterial({ color: 0xb87333, metalness: 0.9, roughness: 0.2 });
        const vcMesh = new THREE.Mesh(vcGeo, vcMat); vcMesh.position.set(0, 0, Z_POS.VAPOR_CHAMBER);
        phoneGroup.add(vcMesh);

        const chassisGeo = new THREE.BoxGeometry(4.7, 9.6, 0.35);
        const chassisMat = new THREE.MeshStandardMaterial({ color: chassisColor, metalness: 0.8, roughness: 0.2 });
        const chassisMesh = new THREE.Mesh(chassisGeo, chassisMat); chassisMesh.position.set(0, 0, Z_POS.CHASSIS);
        phoneGroup.add(chassisMesh);

        const btnGeo = new THREE.BoxGeometry(0.12, 1.2, 0.12);
        const btnMat = new THREE.MeshStandardMaterial({ color: 0xef4444, metalness: 0.9 });
        powerButtonMesh = new THREE.Mesh(btnGeo, btnMat); powerButtonMesh.position.set(2.4, 1.5, 0); powerButtonMesh.name = "power_button";
        phoneGroup.add(powerButtonMesh);

        const boardTexture = createBoardTexture();
        const boardGeo = new THREE.BoxGeometry(4.3, 3.8, 0.08);
        const boardMat = new THREE.MeshStandardMaterial({ map: boardTexture, roughness: 0.4 });
        mainBoardMesh = new THREE.Mesh(boardGeo, boardMat); mainBoardMesh.position.set(0, 2.3, Z_POS.BOARD_AND_BATT); mainBoardMesh.name = "board";
        phoneGroup.add(mainBoardMesh);

        const battTexture = createBatteryTexture(batteryBrand);
        const battGeo = new THREE.BoxGeometry(3.6, 4.6, 0.12);
        const battMat = new THREE.MeshStandardMaterial({ map: battTexture, roughness: 0.5 });
        batteryMesh = new THREE.Mesh(battGeo, battMat); batteryMesh.position.set(-0.2, -0.8, Z_POS.BOARD_AND_BATT); batteryMesh.name = "battery";
        phoneGroup.add(batteryMesh);

        const shieldGeo = new THREE.BoxGeometry(4.5, 9.2, 0.06);
        const shieldMat = new THREE.MeshStandardMaterial({ color: 0x334155, roughness: 0.4, metalness: 0.3 });
        midFrameShieldMesh = new THREE.Mesh(shieldGeo, shieldMat); midFrameShieldMesh.position.set(0, 0, Z_POS.MID_SHIELD); midFrameShieldMesh.name = "shield";
        phoneGroup.add(midFrameShieldMesh);

        const screwCoords = [[-1.8, 3.8], [1.8, 3.8], [-1.8, -3.8], [1.8, -3.8]];
        screwCoords.forEach((coord, idx) => {
            const screwGeo = new THREE.CylinderGeometry(0.14, 0.14, 0.1, 16);
            const screwMat = new THREE.MeshStandardMaterial({ color: 0x94a3b8, metalness: 0.9, roughness: 0.2, emissive: 0x000000, emissiveIntensity: 0 });
            const screw = new THREE.Mesh(screwGeo, screwMat); screw.rotation.x = Math.PI / 2; screw.position.set(coord[0], coord[1], -0.05);

            const hitGeo = new THREE.CylinderGeometry(0.5, 0.5, 0.3, 16);
            const hitMat = new THREE.MeshBasicMaterial({ visible: false });
            const hitBox = new THREE.Mesh(hitGeo, hitMat); hitBox.rotation.x = Math.PI / 2; hitBox.name = "screw"; hitBox.userData = { visualScrew: screw, id: idx };

            screw.add(hitBox); midFrameShieldMesh.add(screw);
            screwsGroup.push({ screw, hitBox });
        });

        const screenTexture = createOSScreenTexture(isPhoneOn ? 'on' : 'off');
        const screenGeo = new THREE.BoxGeometry(4.7, 9.5, 0.08);
        const screenMat = new THREE.MeshStandardMaterial({ map: screenTexture, roughness: 0.1, metalness: 0.8 });
        screenMesh = new THREE.Mesh(screenGeo, screenMat); screenMesh.position.set(0, 0, Z_POS.SCREEN); screenMesh.name = "screen";
        phoneGroup.add(screenMesh);

        const backTexture = createGlassTexture(backColorStr, brandText);
        const backGeo = new THREE.BoxGeometry(4.75, 9.55, 0.08);
        const backMat = new THREE.MeshStandardMaterial({ map: backTexture, roughness: 0.35, metalness: 0.4 });
        backCoverMesh = new THREE.Mesh(backGeo, backMat); backCoverMesh.position.set(0, 0, Z_POS.BACK_COVER); backCoverMesh.name = "backCover";
        phoneGroup.add(backCoverMesh);

        let bumpGeo = new THREE.BoxGeometry(2.1, 2.1, 0.18);
        if (modelKey === 'realmegt8') bumpGeo = new THREE.BoxGeometry(3.6, 1.6, 0.18);
        const bumpMat = new THREE.MeshStandardMaterial({ color: 0x111827, metalness: 0.85, roughness: 0.15 });
        const cameraBump = new THREE.Mesh(bumpGeo, bumpMat);
        cameraBump.position.set(modelKey === 'realmegt8' ? 0 : 0.9, 2.6, 0.05);
        backCoverMesh.add(cameraBump);

        let lensCount = modelKey === 'redmi15' ? 4 : 3;
        for (let i = 0; i < lensCount; i++) {
            const lensBaseGeo = new THREE.CylinderGeometry(0.32, 0.32, 0.1, 32);
            const lensBaseMat = new THREE.MeshStandardMaterial({ color: 0x1e293b, metalness: 0.95 });
            const lensBase = new THREE.Mesh(lensBaseGeo, lensBaseMat); lensBase.rotation.x = Math.PI / 2;

            const glassGeo = new THREE.CylinderGeometry(0.22, 0.22, 0.12, 32);
            const glassMat = new THREE.MeshPhysicalMaterial({ color: 0x0284c7, roughness: 0.05, metalness: 0.9, transmission: 0.6, transparent: true });
            const glass = new THREE.Mesh(glassGeo, glassMat); lensBase.add(glass);

            if (modelKey === 'realmegt8') {
                lensBase.position.set((i * 1.1) - 1.1, 0, 0.1);
            } else {
                lensBase.position.set((i % 2) * 0.8 - 0.4, Math.floor(i / 2) * -0.8 + 0.4, 0.1);
            }
            cameraBump.add(lensBase);
        }

        scene.add(phoneGroup);
        phoneGroup.rotation.y = Math.PI;
        updateScrewsGlow();
    }

    function animate3D() {
        requestAnimationFrame(animate3D);
        controls.update();
        renderer.render(scene, camera);
    }

    function powerButtonPress() {
        initAudio(); playSound('click');
        if (isPhoneBooting) return;
        if (!batteryConnected) { showToast("⚠️ Ошибка: Аккумулятор не подключен!"); return; }

        if (isPhoneOn) {
            isPhoneOn = false;
            screenMesh.material.map = createOSScreenTexture('off');
            screenMesh.material.needsUpdate = true;
            showToast("📴 Смартфон выключен");
            return;
        }

        isPhoneBooting = true;
        smoothRotateTo(0);
        let elapsed = 0, bootTime = 8;
        showToast("⚡ Запуск ОС...");

        const interval = setInterval(() => {
            if (!batteryConnected) {
                clearInterval(interval);
                isPhoneBooting = false; isPhoneOn = false;
                screenMesh.material.map = createOSScreenTexture('off');
                screenMesh.material.needsUpdate = true;
                showToast("❌ Питание прервано: батарея извлечена!");
                return;
            }
            elapsed += 0.5;
            let progress = elapsed / bootTime;
            screenMesh.material.map = createOSScreenTexture('boot', Math.min(progress, 1));
            screenMesh.material.needsUpdate = true;

            if (elapsed >= bootTime) {
                clearInterval(interval);
                isPhoneBooting = false; isPhoneOn = true;
                screenMesh.material.map = createOSScreenTexture('on');
                screenMesh.material.needsUpdate = true;
                playSound('success');
                showToast("✨ Система успешно загружена!");
            }
        }, 500);
    }

    function updateScrewsGlow() {
        screwsGroup.forEach(item => {
            if (!item.screw.parent) return;
            if (selectedTool === 'screwdriver' && coverRemoved && !shieldRemoved) {
                item.screw.material.emissive.setHex(item.hitBox === hoveredScrew ? 0xff5500 : 0x3b82f6);
                item.screw.material.emissiveIntensity = item.hitBox === hoveredScrew ? 1.5 : 0.8;
            } else {
                item.screw.material.emissive.setHex(0x000000); item.screw.material.emissiveIntensity = 0;
            }
        });
    }

    function on3DPointerMove(event) {
        const rect = renderer.domElement.getBoundingClientRect();
        mouse.x = ((event.clientX - rect.left) / rect.width) * 2 - 1;
        mouse.y = -((event.clientY - rect.top) / rect.height) * 2 + 1;
        raycaster.setFromCamera(mouse, camera);
        const intersects = raycaster.intersectObjects(screwsGroup.map(s => s.hitBox), true);
        hoveredScrew = (intersects.length > 0 && selectedTool === 'screwdriver') ? intersects[0].object : null;
        updateScrewsGlow();
    }

    function smoothRotateTo(targetY) {
        initAudio(); playSound('click');
        gsap.to(phoneGroup.rotation, { y: targetY, x: 0, duration: 1.0, ease: "power2.inOut" });
    }

    function spin360() {
        initAudio(); playSound('click');
        gsap.to(phoneGroup.rotation, { y: phoneGroup.rotation.y + Math.PI * 2, duration: 1.4, ease: "back.out(1.2)" });
    }

    function changePhoneModel(modelKey) {
        isPhoneOn = false; buildPhone3DModel(modelKey); renderWorkbench();
        showToast("📱 Модель смартфона изменена");
    }

    function animatePartRemoval(mesh, targetZ, onComplete) {
        initAudio(); playSound('success');
        gsap.timeline()
            .to(mesh.position, { z: targetZ, duration: 0.35, ease: "power2.out" })
            .to(mesh.position, { x: mesh.position.x + 3.2, duration: 0.45, ease: "power2.in" }, "-=0.1")
            .call(() => { mesh.visible = false; if (onComplete) onComplete(); });
    }

    function animatePartReturn(mesh, origZ) {
        initAudio(); playSound('click');
        mesh.visible = true;
        mesh.position.set(mesh.position.x + 3.2, mesh.position.y, origZ + (origZ < 0 ? -1 : 1));
        gsap.timeline()
            .to(mesh.position, { x: 0, duration: 0.45, ease: "power2.out" })
            .to(mesh.position, { z: origZ, duration: 0.35, ease: "bounce.out" });
    }

    function on3DClick(event) {
        const rect = renderer.domElement.getBoundingClientRect();
        mouse.x = ((event.clientX - rect.left) / rect.width) * 2 - 1;
        mouse.y = -((event.clientY - rect.top) / rect.height) * 2 + 1;
        raycaster.setFromCamera(mouse, camera);
        const intersects = raycaster.intersectObjects(phoneGroup.children, true);

        if (intersects.length > 0) {
            let hitObj = intersects[0].object;
            if (hitObj.name === "power_button") { powerButtonPress(); return; }

            if (hitObj.name === "screw" && selectedTool === 'screwdriver') {
                initAudio(); playSound('screw');
                const visScrew = hitObj.userData.visualScrew;
                gsap.timeline()
                    .to(visScrew.rotation, { z: visScrew.rotation.z + Math.PI * 4, duration: 0.4 })
                    .to(visScrew.position, { z: visScrew.position.z - 0.3, duration: 0.4 }, "<")
                    .to(visScrew.scale, { x: 0, y: 0, z: 0, duration: 0.2 })
                    .call(() => {
                        hitObj.parent.parent.remove(visScrew);
                        screwsGroup = screwsGroup.filter(s => s.hitBox !== hitObj);
                        screwsCount--;
                    });
                return;
            }

            while (hitObj.parent && hitObj.parent !== phoneGroup) hitObj = hitObj.parent;

            if (hitObj.name === "backCover") removeBackCoverHandler();
            else if (hitObj.name === "shield") removeShieldHandler();
            else if (hitObj.name === "screen") removeScreenHandler();
            else if (hitObj.name === "battery") interactComponent('battery');
            else if (hitObj.name === "board") interactComponent('board');
        }
    }

    let currentUser = null, userData = { money: 2500, repairsCount: 0, upgrades: {} };
    let selectedTool = 'hand', phoneState = 'broken_screen';
    let coverRemoved = false, screenRemoved = false, removedParts = [], isInfiniteMoney = false;

    function hideOverlay(id) { document.getElementById(id).classList.add('hidden'); }
    function showOverlay(id) { document.getElementById(id).classList.remove('hidden'); }

    function loginOrRegister() {
        initAudio(); playSound('click');
        let login = document.getElementById('loginInput').value.trim();
        let pass = document.getElementById('passInput').value.trim();
        
        if (login.length < 3) return showToast("⚠️ Логин должен содержать от 3 символов!");
        if (pass.length < 4) return showToast("⚠️ Пароль должен быть от 4 символов!");

        let accounts = JSON.parse(localStorage.getItem('phoneLabAccounts2026_v4') || '{}');
        if (accounts[login]) {
            if (accounts[login].password === pass) { 
                currentUser = login; userData = accounts[login].data; 
                if(!userData.repairsCount) userData.repairsCount = 0;
            } else {
                return showToast("❌ Неверный пароль аккаунта!");
            }
        } else {
            accounts[login] = { password: pass, data: { money: 2500, repairsCount: 0, upgrades: {} } };
            localStorage.setItem('phoneLabAccounts2026_v4', JSON.stringify(accounts));
            currentUser = login; userData = accounts[login].data;
        }
        hideOverlay('authOverlay'); showOverlay('modeSelectOverlay');
        document.getElementById('userWelcome').innerText = `Мастер: ${currentUser}`;
        updateMoneyDisplay(); init3D();
        showToast(`👋 С возвращением, ${currentUser}!`);
    }

    function saveProgress() {
        if (!currentUser) return;
        let accounts = JSON.parse(localStorage.getItem('phoneLabAccounts2026_v4') || '{}');
        if (accounts[currentUser]) { 
            accounts[currentUser].data = userData; 
            localStorage.setItem('phoneLabAccounts2026_v4', JSON.stringify(accounts)); 
        }
    }

    function logout() { saveProgress(); location.reload(); }
    
    function updateMoneyDisplay() { 
        document.getElementById('moneyText').innerText = isInfiniteMoney ? "∞ ₽" : (userData.money + ' ₽'); 
    }
    
    function addMoney(amt) { 
        if (!isInfiniteMoney) userData.money += amt; 
        updateMoneyDisplay(); saveProgress(); 
    }
    
    function toggleInfiniteMoney() { 
        isInfiniteMoney = !isInfiniteMoney; 
        updateMoneyDisplay(); 
        showToast(isInfiniteMoney ? "💰 Беск. деньги ВКЛ" : "💰 Беск. деньги ВЫКЛ"); 
    }

    function openProfileModal() {
        initAudio(); playSound('click');
        document.getElementById('profName').innerText = currentUser;
        document.getElementById('profMoney').innerText = userData.money + ' ₽';
        document.getElementById('profRepairs').innerText = userData.repairsCount || 0;
        
        let rank = "Новичок";
        let count = userData.repairsCount || 0;
        if (count >= 5) rank = "Опытный мастер";
        if (count >= 15) rank = "Главный Инженер";
        if (count >= 30) rank = "Гуру микроэлектроники 🏆";
        document.getElementById('profRank').innerText = rank;

        showOverlay('profileOverlay');
    }

    function closeProfileModal() { hideOverlay('profileOverlay'); }

    function exportProfileData() {
        let dataStr = "data:text/json;charset=utf-8," + encodeURIComponent(JSON.stringify({ user: currentUser, data: userData }));
        let downloadAnchor = document.createElement('a');
        downloadAnchor.setAttribute("href", dataStr);
        downloadAnchor.setAttribute("download", `repair_master_${currentUser}_save.json`);
        document.body.appendChild(downloadAnchor);
        downloadAnchor.click();
        downloadAnchor.remove();
        showToast("📤 Сейв профиля успешно выгружен!");
    }

    function randomizeColor() {
        if (backCoverMesh) {
            gsap.to(backCoverMesh.material.color, { r: Math.random(), g: Math.random(), b: Math.random(), duration: 0.5 });
            showToast("🎨 Цвет корпуса изменен!");
        }
    }

    function setState(st) {
        phoneState = st;
        if (st === 'swollen_battery' && batteryMesh) {
            gsap.to(batteryMesh.scale, { x: 1.3, z: 1.8, duration: 0.6, ease: "back.out(1.7)" });
        } else if (batteryMesh) {
            gsap.to(batteryMesh.scale, { x: 1.0, z: 1.0, duration: 0.4 });
        }
        showToast(`⚙️ Состояние: ${st}`);
    }

    function startOrdersMode() {
        initAudio(); playSound('click');
        hideOverlay('modeSelectOverlay');
        document.getElementById('orderPanel').style.display = 'flex';
        document.getElementById('sandboxPanel').style.display = 'none';
        resetPhone();
    }

    function requestSandboxMode() { 
        initAudio(); playSound('click');
        document.getElementById('sandboxPassInput').value = '';
        showOverlay('passwordOverlay'); 
    }

    function verifySandboxPassword() {
        initAudio();
        const pass = document.getElementById('sandboxPassInput').value.trim();
        if (pass === "6767") {
            playSound('success');
            hideOverlay('passwordOverlay'); hideOverlay('modeSelectOverlay');
            document.getElementById('orderPanel').style.display = 'none';
            document.getElementById('sandboxPanel').style.display = 'flex';
            resetPhone();
            showToast("🧪 Режим Песочницы активирован!");
        } else {
            playSound('click'); showToast("❌ Неверный пароль!");
        }
    }

    function exitToMenu() { saveProgress(); showOverlay('modeSelectOverlay'); }
    function openUpgradeModal() { initAudio(); playSound('click'); showOverlay('upgradeOverlay'); }
    function closeUpgradeModal() { initAudio(); playSound('click'); hideOverlay('upgradeOverlay'); }

    function buyUpgrade(key, cost) {
        initAudio();
        if (userData.money >= cost && !userData.upgrades[key]) {
            userData.money -= cost; userData.upgrades[key] = true; addMoney(0);
            playSound('success'); showToast("🎉 Апгрейд приобретен!");
        } else { showToast("❌ Недостаточно средств!"); }
    }

    function selectTool(key, el) {
        initAudio(); playSound('click');
        selectedTool = key;
        document.querySelectorAll('.tool-card').forEach(c => c.classList.remove('active'));
        el.classList.add('active');
        updateScrewsGlow();
    }

    function buyPart(name, price) {
        initAudio();
        if (isInfiniteMoney || userData.money >= price) {
            if(!isInfiniteMoney) userData.money -= price;
            addMoney(0); removedParts.push(name); renderWorkbench();
            playSound('success'); showToast(`🛒 Куплено: ${name}`);
        } else { showToast("❌ Недостаточно средств!"); }
    }

    function resetPhone() {
        coverRemoved = false; screenRemoved = false; isPhoneOn = false; batteryConnected = true; removedParts = [];
        buildPhone3DModel(currentModelKey);
        smoothRotateTo(Math.PI);
        renderWorkbench();
    }

    function removeBackCoverHandler() {
        if ((selectedTool === 'heatgun' || selectedTool === 'screwdriver') && !coverRemoved) {
            coverRemoved = true;
            animatePartRemoval(backCoverMesh, -2, () => {
                removedParts.push('🔲 Задняя крышка');
                renderWorkbench(); updateScrewsGlow();
            });
        }
    }

    function removeShieldHandler() {
        if (screwsCount > 0) { showToast(`⚠️ Выкрутите винты (осталось: ${screwsCount})!`); return; }
        if (selectedTool === 'screwdriver' && coverRemoved && !shieldRemoved) {
            shieldRemoved = true;
            animatePartRemoval(midFrameShieldMesh, -1.5, () => {
                removedParts.push('🔲 Экранирующий кожух');
                renderWorkbench();
            });
        }
    }

    function removeScreenHandler() {
        if ((selectedTool === 'heatgun' || selectedTool === 'tweezers') && coverRemoved && !screenRemoved && phoneState === 'broken_screen') {
            screenRemoved = true;
            animatePartRemoval(screenMesh, 2, () => {
                removedParts.push('📱 Битый дисплей');
                renderWorkbench();
            });
        }
    }

    function interactComponent(type) {
        if (!shieldRemoved) { showToast("⚠️ Снимите защитный кожух!"); return; }

        if (selectedTool === 'tweezers' || selectedTool === 'screwdriver') {
            if (type === 'battery' && !removedParts.includes('🔋 Аккумулятор')) {
                batteryConnected = false;
                if (isPhoneOn) {
                    isPhoneOn = false;
                    screenMesh.material.map = createOSScreenTexture('off');
                    screenMesh.material.needsUpdate = true;
                    showToast("⚡ Батарея отключена — смартфон выключился!");
                } else { showToast("🔋 Аккумулятор отсоединен."); }
                animatePartRemoval(batteryMesh, 2, () => { removedParts.push('🔋 Аккумулятор'); renderWorkbench(); });
            } else if (type === 'board' && !removedParts.includes('🖲️ Плата')) {
                animatePartRemoval(mainBoardMesh, 2, () => { removedParts.push('🖲️ Плата'); renderWorkbench(); });
            }
        } else if (selectedTool === 'multimeter') {
            showToast(phoneState === 'short_circuit' ? "⚡ Тестер: КЗ в цепи питания!" : "⚡ Тестер: Напряжение в норме.");
        }
    }

    function returnPart(index) {
        let part = removedParts[index];
        if (part.includes('Задняя крышка')) { coverRemoved = false; animatePartReturn(backCoverMesh, Z_POS.BACK_COVER); }
        else if (part.includes('кожух')) { shieldRemoved = false; screwsCount = 0; animatePartReturn(midFrameShieldMesh, Z_POS.MID_SHIELD); }
        else if (part.includes('Новый Дисплей')) { screenRemoved = false; phoneState = 'normal'; animatePartReturn(screenMesh, Z_POS.SCREEN); }
        else if (part.includes('Новый АКБ') || part.includes('Аккумулятор')) { phoneState = 'normal'; batteryConnected = true; animatePartReturn(batteryMesh, Z_POS.BOARD_AND_BATT); }
        else if (part.includes('Плата')) { animatePartReturn(mainBoardMesh, Z_POS.BOARD_AND_BATT); }
        removedParts.splice(index, 1);
        renderWorkbench();
    }

    function finishOrder() {
        initAudio();
        if (phoneState === 'normal' && coverRemoved && removedParts.length === 0) {
            let reward = userData.upgrades['proTools'] ? 4200 : 3500;
            userData.repairsCount = (userData.repairsCount || 0) + 1;
            addMoney(reward); playSound('success');
            showToast(`🎉 Заказ выполнен! Награда: +${reward} ₽`);
            resetPhone();
        } else { showToast("❌ Смартфон не собран или неисправен!"); }
    }

    function renderWorkbench() {
        const tray = document.getElementById('partsTray');
        tray.innerHTML = removedParts.length === 0 ? `<span style="font-size: 9px; color: var(--text-muted);">Лоток пуст</span>` : '';
        removedParts.forEach((p, idx) => {
            tray.innerHTML += `<div class="tray-item" onclick="returnPart(${idx})">${p} 🔄</div>`;
        });
    }
</script>
</body>
</html>
