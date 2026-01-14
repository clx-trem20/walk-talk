<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Walkie-Talkie Online</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Orbitron:wght@400;700&family=Inter:wght@300;400;600&display=swap');

        :root {
            --lcd-green: #94ad1a;
            --lcd-bg: #c5d671;
        }

        body {
            background-color: #1a1a1a;
            font-family: 'Inter', sans-serif;
            color: #e2e8f0;
        }

        .lcd-display {
            background-color: var(--lcd-bg);
            color: #2b300a;
            font-family: 'Orbitron', sans-serif;
            box-shadow: inset 0 4px 10px rgba(0,0,0,0.3);
            border: 4px solid #333;
            text-shadow: 1px 1px 0px rgba(255,255,255,0.2);
        }

        .ptt-button {
            width: 180px;
            height: 180px;
            border-radius: 50%;
            background: radial-gradient(circle, #ff4d4d 0%, #cc0000 100%);
            box-shadow: 0 10px 0 #800000, 0 15px 20px rgba(0,0,0,0.5);
            transition: all 0.1s ease;
            display: flex;
            align-items: center;
            justify-content: center;
            user-select: none;
            cursor: pointer;
            touch-action: manipulation;
        }

        .ptt-button:active, .ptt-button.active {
            transform: translateY(8px);
            box-shadow: 0 2px 0 #800000, 0 5px 10px rgba(0,0,0,0.3);
            background: radial-gradient(circle, #ff1a1a 0%, #990000 100%);
        }

        .ptt-button i {
            font-size: 4rem;
            color: white;
            text-shadow: 0 2px 4px rgba(0,0,0,0.3);
        }

        .noise-overlay {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: url('https://www.transparenttextures.com/patterns/stardust.png');
            opacity: 0.05;
            pointer-events: none;
            z-index: 50;
        }

        @keyframes pulse {
            0% { transform: scale(1); opacity: 1; }
            50% { transform: scale(1.05); opacity: 0.8; }
            100% { transform: scale(1); opacity: 1; }
        }

        .transmitting {
            animation: pulse 1s infinite;
            border-color: #ff4d4d !important;
        }
    </style>
</head>
<body class="min-h-screen flex items-center justify-center p-4">
    <div class="noise-overlay"></div>

    <div id="app" class="w-full max-w-md bg-zinc-900 rounded-[3rem] p-8 border-8 border-zinc-800 shadow-2xl relative overflow-hidden">
        
        <div class="flex justify-between items-center mb-6">
            <div class="flex flex-col">
                <span class="text-zinc-500 text-xs font-bold tracking-widest uppercase">Modelo RX-2026</span>
                <h1 class="text-2xl font-black italic tracking-tighter text-white">WALKIE-TALKIE</h1>
            </div>
            <div id="status-led" class="w-3 h-3 rounded-full bg-zinc-700"></div>
        </div>

        <div id="display-container" class="lcd-display rounded-xl p-4 mb-8 h-32 flex flex-col justify-between relative">
            <div class="flex justify-between text-[10px] font-bold uppercase opacity-70">
                <span>Canal Ativo</span>
                <span id="signal-strength">Sinal: ---</span>
            </div>
            
            <div id="lcd-status" class="flex flex-col items-center justify-center flex-grow">
                <span id="channel-name-display" class="text-xl font-bold">---</span>
                <span id="transmission-status" class="text-xs mt-1 italic tracking-widest">OFFLINE</span>
            </div>

            <div class="flex justify-between items-end">
                <div class="flex gap-1">
                    <div class="w-1 h-3 bg-zinc-800 opacity-20"></div>
                    <div class="w-1 h-3 bg-zinc-800 opacity-20"></div>
                    <div class="w-1 h-3 bg-zinc-800"></div>
                </div>
                <span class="text-[10px] font-bold">FREQ: 446.0 MHz</span>
            </div>
        </div>

        <div id="setup-view" class="space-y-4">
            <div>
                <label class="block text-xs font-bold text-zinc-500 mb-1 uppercase">Nome do Canal</label>
                <input type="text" id="channel-input" placeholder="Ex: Time-Alfa" class="w-full bg-zinc-800 border-2 border-zinc-700 rounded-lg p-3 text-white focus:outline-none focus:border-blue-500">
            </div>
            <div>
                <label class="block text-xs font-bold text-zinc-500 mb-1 uppercase">Senha do Canal</label>
                <input type="password" id="password-input" placeholder="Senha secreta" class="w-full bg-zinc-800 border-2 border-zinc-700 rounded-lg p-3 text-white focus:outline-none focus:border-blue-500">
            </div>
            <button onclick="connectChannel()" class="w-full bg-blue-600 hover:bg-blue-500 text-white font-bold py-3 rounded-lg shadow-lg transition-transform active:scale-95">
                SINTONIZAR CANAL
            </button>
            <p class="text-[10px] text-zinc-600 text-center italic mt-2">Certifique-se que seus companheiros usem a mesma senha.</p>
        </div>

        <div id="radio-view" class="hidden flex flex-col items-center space-y-8 py-4">
            <div class="ptt-button" id="ptt-btn">
                <i class="fa-solid fa-microphone-lines"></i>
            </div>
            
            <div class="text-center">
                <p class="text-zinc-400 font-semibold mb-2">SEGURE PARA FALAR</p>
                <button onclick="disconnect()" class="px-4 py-2 bg-zinc-800 hover:bg-zinc-700 text-zinc-300 text-xs font-bold rounded-full border border-zinc-700 uppercase">
                    Sair do Canal
                </button>
            </div>

            <div class="w-full px-4">
                <div class="flex justify-between text-[10px] text-zinc-500 font-bold mb-1 uppercase">
                    <span>Volume de Recebimento</span>
                    <span id="vol-label">80%</span>
                </div>
                <input type="range" id="volume-slider" min="0" max="100" value="80" class="w-full h-2 bg-zinc-800 rounded-lg appearance-none cursor-pointer accent-blue-600">
            </div>
        </div>

        <div class="mt-8 flex justify-center gap-2">
            <div class="w-12 h-1 bg-zinc-800 rounded-full"></div>
            <div class="w-12 h-1 bg-zinc-800 rounded-full"></div>
            <div class="w-12 h-1 bg-zinc-800 rounded-full"></div>
        </div>
    </div>

    <script>
        let isConnected = false;
        let isTalking = false;
        let audioCtx = null;
        
        const setupView = document.getElementById('setup-view');
        const radioView = document.getElementById('radio-view');
        const lcdStatus = document.getElementById('transmission-status');
        const channelDisplay = document.getElementById('channel-name-display');
        const displayBox = document.getElementById('display-container');
        const led = document.getElementById('status-led');
        const pttBtn = document.getElementById('ptt-btn');
        const volSlider = document.getElementById('volume-slider');
        const volLabel = document.getElementById('vol-label');

        // Inicializa o contexto de áudio apenas após interação do usuário
        function initAudio() {
            if (!audioCtx) {
                audioCtx = new (window.AudioContext || window.webkitAudioContext)();
            }
        }

        // Função para gerar sons de rádio sinteticamente (resolve o erro de fontes externas)
        function playRadioTone(type) {
            if (!audioCtx) return;
            
            const osc = audioCtx.createOscillator();
            const gain = audioCtx.createGain();
            
            osc.connect(gain);
            gain.connect(audioCtx.destination);
            
            if (type === 'start') {
                // Som de início (agudo curto)
                osc.type = 'sine';
                osc.frequency.setValueAtTime(880, audioCtx.currentTime);
                gain.gain.setValueAtTime(0, audioCtx.currentTime);
                gain.gain.linearRampToValueAtTime(0.1, audioCtx.currentTime + 0.05);
                gain.gain.linearRampToValueAtTime(0, audioCtx.currentTime + 0.15);
                osc.start();
                osc.stop(audioCtx.currentTime + 0.2);
            } else if (type === 'end') {
                // Som de fim "Roger Beep" (dois tons)
                osc.type = 'sine';
                osc.frequency.setValueAtTime(1000, audioCtx.currentTime);
                osc.frequency.setValueAtTime(800, audioCtx.currentTime + 0.1);
                gain.gain.setValueAtTime(0.1, audioCtx.currentTime);
                gain.gain.linearRampToValueAtTime(0, audioCtx.currentTime + 0.2);
                osc.start();
                osc.stop(audioCtx.currentTime + 0.2);
            }
        }

        function connectChannel() {
            const channel = document.getElementById('channel-input').value.trim();
            const pass = document.getElementById('password-input').value.trim();

            if (!channel || !pass) {
                updateLCD("ERRO: DADOS VAZIOS");
                return;
            }

            initAudio();
            
            setupView.classList.add('hidden');
            radioView.classList.remove('hidden');
            
            channelDisplay.innerText = "CH: " + channel.toUpperCase();
            lcdStatus.innerText = "SINTONIZADO / STANDBY";
            document.getElementById('signal-strength').innerText = "Sinal: FORTE";
            led.classList.replace('bg-zinc-700', 'bg-green-500');
            led.classList.add('shadow-[0_0_10px_#22c55e]');
            
            isConnected = true;
            
            navigator.mediaDevices.getUserMedia({ audio: true })
                .catch(() => updateLCD("SEM MIC"));
        }

        function startTalking() {
            if (!isConnected || isTalking) return;
            initAudio();
            
            isTalking = true;
            pttBtn.classList.add('active');
            displayBox.classList.add('transmitting');
            lcdStatus.innerText = "TRANSMITINDO...";
            led.classList.replace('bg-green-500', 'bg-red-500');
            led.classList.replace('shadow-[0_0_10px_#22c55e]', 'shadow-[0_0_10px_#ef4444]');
            
            playRadioTone('start');
        }

        function stopTalking() {
            if (!isTalking) return;
            
            isTalking = false;
            pttBtn.classList.remove('active');
            displayBox.classList.remove('transmitting');
            lcdStatus.innerText = "STANDBY";
            led.classList.replace('bg-red-500', 'bg-green-500');
            led.classList.replace('shadow-[0_0_10px_#ef4444]', 'shadow-[0_0_10px_#22c55e]');
            
            playRadioTone('end');
        }

        function disconnect() {
            isConnected = false;
            radioView.classList.add('hidden');
            setupView.classList.remove('hidden');
            channelDisplay.innerText = "---";
            lcdStatus.innerText = "OFFLINE";
            displayBox.classList.remove('transmitting');
            document.getElementById('signal-strength').innerText = "Sinal: ---";
            led.classList.remove('bg-green-500', 'bg-red-500', 'shadow-[0_0_10px_#22c55e]', 'shadow-[0_0_10px_#ef4444]');
            led.classList.add('bg-zinc-700');
        }

        function updateLCD(msg) {
            const current = lcdStatus.innerText;
            lcdStatus.innerText = msg;
            setTimeout(() => {
                if (isConnected) lcdStatus.innerText = "STANDBY";
            }, 2000);
        }

        volSlider.addEventListener('input', (e) => {
            volLabel.innerText = e.target.value + "%";
        });

        // Eventos de Mouse
        pttBtn.addEventListener('mousedown', startTalking);
        window.addEventListener('mouseup', stopTalking);

        // Eventos de Touch (Mobile)
        pttBtn.addEventListener('touchstart', (e) => {
            e.preventDefault();
            startTalking();
        });
        pttBtn.addEventListener('touchend', (e) => {
            e.preventDefault();
            stopTalking();
        });

        // Bloquear menu de contexto
        pttBtn.addEventListener('contextmenu', e => e.preventDefault());
    </script>
</body>
</html>
