<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Walkie-Talkie Online Pro</title>
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
                <span class="text-zinc-500 text-xs font-bold tracking-widest uppercase">Modelo RX-2026 CLOUD</span>
                <h1 class="text-2xl font-black italic tracking-tighter text-white">WALKIE-TALKIE</h1>
            </div>
            <div id="status-led" class="w-3 h-3 rounded-full bg-zinc-700"></div>
        </div>

        <div id="display-container" class="lcd-display rounded-xl p-4 mb-8 h-32 flex flex-col justify-between relative">
            <div class="flex justify-between text-[10px] font-bold uppercase opacity-70">
                <span>Canal</span>
                <span id="signal-strength">Usuários: 0</span>
            </div>
            
            <div id="lcd-status" class="flex flex-col items-center justify-center flex-grow text-center px-2">
                <span id="channel-name-display" class="text-xl font-bold italic tracking-wider">OFFLINE</span>
                <span id="transmission-status" class="text-[10px] mt-1 italic tracking-widest uppercase">Aguardando entrada</span>
            </div>

            <div class="flex justify-between items-end">
                <div class="flex gap-1">
                    <div id="sig-1" class="w-1 h-3 bg-zinc-800 opacity-20"></div>
                    <div id="sig-2" class="w-1 h-3 bg-zinc-800 opacity-20"></div>
                    <div id="sig-3" class="w-1 h-3 bg-zinc-800 opacity-20"></div>
                </div>
                <span id="user-id-display" class="text-[8px] font-bold opacity-50">ID: ----</span>
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
            <button id="connect-btn" class="w-full bg-blue-600 hover:bg-blue-500 text-white font-bold py-3 rounded-lg shadow-lg transition-transform active:scale-95">
                SINTONIZAR CANAL
            </button>
            <p class="text-[10px] text-zinc-600 text-center italic mt-2">Permita o uso do microfone para conectar.</p>
        </div>

        <div id="radio-view" class="hidden flex flex-col items-center space-y-8 py-4">
            <div class="ptt-button" id="ptt-btn">
                <i class="fa-solid fa-microphone-lines"></i>
            </div>
            
            <div class="text-center">
                <p class="text-zinc-400 font-semibold mb-2">SEGURE PARA FALAR</p>
                <button onclick="window.location.reload()" class="px-4 py-2 bg-zinc-800 hover:bg-zinc-700 text-zinc-300 text-xs font-bold rounded-full border border-zinc-700 uppercase">
                    Desconectar
                </button>
            </div>

            <div class="w-full px-4">
                <div class="flex justify-between text-[10px] text-zinc-500 font-bold mb-1 uppercase">
                    <span>Ganho de Áudio</span>
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

    <!-- Firebase SDK -->
    <script type="module">
        import { initializeApp } from 'https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js';
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from 'https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js';
        import { getFirestore, doc, setDoc, onSnapshot, collection, addDoc, deleteDoc, query } from 'https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js';

        const firebaseConfig = JSON.parse(__firebase_config);
        const app = initializeApp(firebaseConfig);
        const auth = getAuth(app);
        const db = getFirestore(app);
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'walkie-talkie-default';

        let localStream = null;
        let peerConnections = {};
        let userId = null;
        let isTalking = false;
        let audioCtx = null;
        
        let usersCollectionRef = null;
        let signalsCollectionRef = null;

        const pttBtn = document.getElementById('ptt-btn');
        const setupView = document.getElementById('setup-view');
        const radioView = document.getElementById('radio-view');
        const lcdStatus = document.getElementById('transmission-status');
        const channelDisplay = document.getElementById('channel-name-display');
        const displayBox = document.getElementById('display-container');
        const led = document.getElementById('status-led');
        const userDisplay = document.getElementById('user-id-display');

        const configuration = {
            iceServers: [{ urls: 'stun:stun.l.google.com:19302' }]
        };

        function playTone(type) {
            try {
                if (!audioCtx) audioCtx = new (window.AudioContext || window.webkitAudioContext)();
                const osc = audioCtx.createOscillator();
                const gain = audioCtx.createGain();
                osc.connect(gain); gain.connect(audioCtx.destination);
                if(type === 'start') {
                    osc.frequency.setValueAtTime(880, audioCtx.currentTime);
                    gain.gain.setValueAtTime(0, audioCtx.currentTime);
                    gain.gain.linearRampToValueAtTime(0.1, audioCtx.currentTime + 0.05);
                    osc.start(); osc.stop(audioCtx.currentTime + 0.15);
                } else {
                    osc.frequency.setValueAtTime(1000, audioCtx.currentTime);
                    osc.frequency.exponentialRampToValueAtTime(600, audioCtx.currentTime + 0.2);
                    gain.gain.setValueAtTime(0.1, audioCtx.currentTime);
                    gain.gain.linearRampToValueAtTime(0, audioCtx.currentTime + 0.2);
                    osc.start(); osc.stop(audioCtx.currentTime + 0.2);
                }
            } catch(e) {}
        }

        async function connectChannel() {
            const channelName = document.getElementById('channel-input').value.trim();
            const password = document.getElementById('password-input').value.trim();

            if (!channelName || !password) {
                lcdStatus.innerText = "ERRO: DADOS INVÁLIDOS";
                return;
            }

            lcdStatus.innerText = "INICIANDO MIC...";

            try {
                // Passo 1: Solicitar Mic antes de tudo
                localStream = await navigator.mediaDevices.getUserMedia({ audio: true });
                localStream.getAudioTracks()[0].enabled = false;
                
                lcdStatus.innerText = "AUTENTICANDO...";

                // Passo 2: Autenticação
                if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
                    await signInWithCustomToken(auth, __initial_auth_token);
                } else {
                    await signInAnonymously(auth);
                }

                // Esperar pelo ID do usuário
                const user = await new Promise((resolve) => {
                    const unsubscribe = onAuthStateChanged(auth, (u) => {
                        if (u) {
                            unsubscribe();
                            resolve(u);
                        }
                    });
                });

                userId = user.uid;
                userDisplay.innerText = `ID: ${userId.substring(0,6)}`;

                // Passo 3: Configurar coleções (Estrutura obrigatória para permissões)
                const safeChannelId = btoa(`${channelName}_${password}`).replace(/[/+=]/g, '').substring(0, 20);
                
                usersCollectionRef = collection(db, 'artifacts', appId, 'public', 'data', `users_${safeChannelId}`);
                signalsCollectionRef = collection(db, 'artifacts', appId, 'public', 'data', `signals_${safeChannelId}`);

                // Mudar UI
                setupView.classList.add('hidden');
                radioView.classList.remove('hidden');
                channelDisplay.innerText = channelName.toUpperCase();
                lcdStatus.innerText = "CANAL ATIVO";
                led.classList.replace('bg-zinc-700', 'bg-green-500');

                // Registrar presença
                await setDoc(doc(usersCollectionRef, userId), {
                    id: userId,
                    lastSeen: Date.now()
                });

                // Iniciar escutas
                initFirestoreListeners();

            } catch (e) {
                console.error("Erro na conexão:", e);
                lcdStatus.innerText = "ERRO NA PERMISSÃO";
            }
        }

        function initFirestoreListeners() {
            // Ouvir outros usuários
            onSnapshot(usersCollectionRef, (snapshot) => {
                document.getElementById('signal-strength').innerText = `Usuários: ${snapshot.size}`;
                snapshot.docChanges().forEach(async (change) => {
                    if (change.type === 'added' && change.doc.id !== userId) {
                        createPeerConnection(change.doc.id, true);
                    }
                });
            }, (err) => {
                lcdStatus.innerText = "ERRO DE REDE";
            });

            // Ouvir mensagens direcionadas (Sinalização WebRTC)
            onSnapshot(signalsCollectionRef, (snapshot) => {
                snapshot.docChanges().forEach(async (change) => {
                    if (change.type === 'added') {
                        const data = change.doc.data();
                        if (data.to === userId) {
                            handleSignal(data, change.doc.ref);
                        }
                    }
                });
            });
        }

        async function handleSignal(data, docRef) {
            const fromId = data.from;
            if (!peerConnections[fromId]) createPeerConnection(fromId, false);
            const pc = peerConnections[fromId];

            try {
                if (data.type === 'offer') {
                    await pc.setRemoteDescription(new RTCSessionDescription(data.offer));
                    const answer = await pc.createAnswer();
                    await pc.setLocalDescription(answer);
                    await addDoc(signalsCollectionRef, {
                        type: 'answer', from: userId, to: fromId, answer: answer
                    });
                } else if (data.type === 'answer') {
                    await pc.setRemoteDescription(new RTCSessionDescription(data.answer));
                } else if (data.type === 'ice') {
                    await pc.addIceCandidate(new RTCIceCandidate(data.candidate));
                }
            } catch (e) {}
            await deleteDoc(docRef);
        }

        function createPeerConnection(remoteId, isOfferer) {
            if (peerConnections[remoteId]) return;
            
            const pc = new RTCPeerConnection(configuration);
            peerConnections[remoteId] = pc;

            localStream.getTracks().forEach(track => pc.addTrack(track, localStream));

            pc.onicecandidate = (event) => {
                if (event.candidate) {
                    addDoc(signalsCollectionRef, {
                        type: 'ice', from: userId, to: remoteId, candidate: event.candidate.toJSON()
                    });
                }
            };

            pc.ontrack = (event) => {
                const remoteAudio = new Audio();
                remoteAudio.srcObject = event.streams[0];
                remoteAudio.play().catch(() => {});
            };

            if (isOfferer) {
                pc.onnegotiationneeded = async () => {
                    try {
                        const offer = await pc.createOffer();
                        await pc.setLocalDescription(offer);
                        await addDoc(signalsCollectionRef, {
                            type: 'offer', from: userId, to: remoteId, offer: offer
                        });
                    } catch (e) {}
                };
            }
        }

        function startTalking() {
            if (!userId || isTalking) return;
            isTalking = true;
            playTone('start');
            if (localStream) localStream.getAudioTracks()[0].enabled = true;
            pttBtn.classList.add('active');
            displayBox.classList.add('transmitting');
            lcdStatus.innerText = "TRANSMITINDO...";
            led.classList.replace('bg-green-500', 'bg-red-500');
        }

        function stopTalking() {
            if (!isTalking) return;
            isTalking = false;
            playTone('end');
            if (localStream) localStream.getAudioTracks()[0].enabled = false;
            pttBtn.classList.remove('active');
            displayBox.classList.remove('transmitting');
            lcdStatus.innerText = "STANDBY / RECEBENDO";
            led.classList.replace('bg-red-500', 'bg-green-500');
        }

        document.getElementById('connect-btn').onclick = connectChannel;
        
        // PTT Eventos
        pttBtn.addEventListener('mousedown', startTalking);
        window.addEventListener('mouseup', stopTalking);
        pttBtn.addEventListener('touchstart', (e) => { e.preventDefault(); startTalking(); }, {passive: false});
        pttBtn.addEventListener('touchend', (e) => { e.preventDefault(); stopTalking(); }, {passive: false});
        pttBtn.addEventListener('contextmenu', e => e.preventDefault());

        window.onbeforeunload = () => {
            if (userId && usersCollectionRef) {
                deleteDoc(doc(usersCollectionRef, userId));
            }
        };
    </script>
</body>
</html>
