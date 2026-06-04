# N
I
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=yes">
    <title>AI Kamera · Deteksi Ekspresi Wajah</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        body {
            background: linear-gradient(145deg, #0a0f1a 0%, #03060c 100%);
            font-family: 'Segoe UI', system-ui, 'Inter', sans-serif;
            min-height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
            padding: 20px;
        }
        .glass-card {
            max-width: 1000px;
            width: 100%;
            background: rgba(10, 20, 30, 0.65);
            backdrop-filter: blur(12px);
            border-radius: 2.5rem;
            border: 1px solid rgba(255, 200, 100, 0.4);
            padding: 1rem 1.5rem 1.8rem;
            box-shadow: 0 20px 35px -10px black;
            transition: 0.2s;
        }
        h1 {
            font-size: 1.6rem;
            font-weight: 800;
            background: linear-gradient(135deg, #FFD966, #FFB347);
            -webkit-background-clip: text;
            background-clip: text;
            color: transparent;
            display: inline-block;
        }
        .sub {
            color: #bbb;
            font-size: 0.75rem;
            margin-bottom: 1rem;
            border-left: 2px solid #FFB347;
            padding-left: 12px;
        }
        .video-container {
            position: relative;
            background: #00000055;
            border-radius: 2rem;
            overflow: hidden;
            margin: 1rem 0;
            box-shadow: 0 8px 20px rgba(0,0,0,0.4);
        }
        video, canvas {
            width: 100%;
            border-radius: 1.5rem;
            display: block;
        }
        canvas {
            position: absolute;
            top: 0;
            left: 0;
        }
        .emotion-panel {
            display: flex;
            flex-wrap: wrap;
            justify-content: space-between;
            gap: 16px;
            margin-top: 18px;
        }
        .emotion-card {
            background: #0a121ccc;
            border-radius: 1.5rem;
            padding: 10px 16px;
            flex: 1;
            text-align: center;
            border: 0.5px solid rgba(255,180,100,0.4);
            backdrop-filter: blur(4px);
        }
        .emotion-label {
            font-size: 0.7rem;
            text-transform: uppercase;
            letter-spacing: 1px;
            color: #ffd9a3;
        }
        .emotion-value {
            font-size: 1.4rem;
            font-weight: bold;
            color: #FFE4B5;
        }
        .status {
            background: #00000077;
            border-radius: 60px;
            padding: 8px 16px;
            font-size: 0.8rem;
            text-align: center;
            margin-top: 18px;
            color: #FFDCA8;
        }
        button {
            background: linear-gradient(95deg, #FFB347, #FF8C2E);
            border: none;
            padding: 8px 18px;
            border-radius: 60px;
            font-weight: bold;
            color: #1e1a0f;
            cursor: pointer;
            margin-top: 8px;
            transition: 0.1s;
        }
        button:active {
            transform: scale(0.96);
        }
        footer {
            font-size: 0.6rem;
            text-align: center;
            color: #6a8faa;
            margin-top: 1.3rem;
        }
        .loader {
            display: inline-block;
            width: 18px;
            height: 18px;
            border: 2px solid #FFB347;
            border-radius: 50%;
            border-top-color: transparent;
            animation: spin 0.8s linear infinite;
            margin-left: 12px;
            vertical-align: middle;
        }
        @keyframes spin { to { transform: rotate(360deg); } }
    </style>
    <!-- Face-api.js CDN -->
    <script src="https://cdn.jsdelivr.net/npm/face-api.js@0.22.2/dist/face-api.min.js"></script>
</head>
<body>
<div class="glass-card">
    <div style="display: flex; justify-content: space-between; align-items: center; flex-wrap: wrap;">
        <h1>🐾 DIZX · EMOTION SCAN</h1>
        <button id="resetCamBtn">🔄 Reset / Mulai Ulang</button>
    </div>
    <div class="sub">🎥 Deteksi ekspresi realtime: senang, sedih, marah, terkejut, netral, jijik, takut</div>

    <div class="video-container">
        <video id="video" autoplay muted playsinline></video>
        <canvas id="overlayCanvas"></canvas>
    </div>

    <div class="emotion-panel" id="emotionPanel">
        <div class="emotion-card"><div class="emotion-label">😊 Senang</div><div class="emotion-value" id="happyVal">-</div></div>
        <div class="emotion-card"><div class="emotion-label">😢 Sedih</div><div class="emotion-value" id="sadVal">-</div></div>
        <div class="emotion-card"><div class="emotion-label">😠 Marah</div><div class="emotion-value" id="angryVal">-</div></div>
        <div class="emotion-card"><div class="emotion-label">😲 Terkejut</div><div class="emotion-value" id="surpriseVal">-</div></div>
        <div class="emotion-card"><div class="emotion-label">😐 Netral</div><div class="emotion-value" id="neutralVal">-</div></div>
        <div class="emotion-card"><div class="emotion-label">🤢 Jijik</div><div class="emotion-value" id="disgustVal">-</div></div>
        <div class="emotion-card"><div class="emotion-label">😨 Takut</div><div class="emotion-value" id="fearVal">-</div></div>
    </div>

    <div class="status" id="statusMsg">
        🔍 Memuat model AI wajah... <span class="loader"></span>
    </div>
    <footer>📌 Model menggunakan Face-api.js (ekspresi realtime). Kamera hanya digunakan di browser lokal, privasi terjaga.</footer>
</div>

<script>
    (function(){
        const video = document.getElementById('video');
        const canvas = document.getElementById('overlayCanvas');
        const ctx = canvas.getContext('2d');
        const statusDiv = document.getElementById('statusMsg');
        
        // elemen nilai ekspresi
        const happySpan = document.getElementById('happyVal');
        const sadSpan = document.getElementById('sadVal');
        const angrySpan = document.getElementById('angryVal');
        const surpriseSpan = document.getElementById('surpriseVal');
        const neutralSpan = document.getElementById('neutralVal');
        const disgustSpan = document.getElementById('disgustVal');
        const fearSpan = document.getElementById('fearVal');

        let streaming = false;
        let animationId = null;
        let modelsLoaded = false;

        // load model face-api
        async function loadModels() {
            statusDiv.innerHTML = "📦 Mengunduh model ekspresi wajah (sekitar 5-10MB), harap tunggu... 🧠";
            try {
                const MODEL_URL = 'https://raw.githubusercontent.com/justadudewhohacks/face-api.js/master/weights';
                // Pastikan menggunakan path yang benar (menggunakan jsdelivr juga bisa)
                await faceapi.nets.tinyFaceDetector.loadFromUri('/models');
                await faceapi.nets.faceExpressionNet.loadFromUri('/models');
                statusDiv.innerHTML = "✅ Model siap! Memulai kamera...";
                modelsLoaded = true;
                startVideo();
            } catch (err) {
                console.warn('Gagal load dari /models, coba pakai CDN weight');
                try {
                    // fallback menggunakan cdn alternatif
                    await faceapi.nets.tinyFaceDetector.loadFromUri('https://cdn.jsdelivr.net/npm/face-api.js@0.22.2/weights');
                    await faceapi.nets.faceExpressionNet.loadFromUri('https://cdn.jsdelivr.net/npm/face-api.js@0.22.2/weights');
                    statusDiv.innerHTML = "✅ Model loaded (CDN). Memulai kamera.";
                    modelsLoaded = true;
                    startVideo();
                } catch (e) {
                    statusDiv.innerHTML = "❌ Gagal memuat model AI. Periksa koneksi internet dan refresh.";
                    console.error(e);
                }
            }
        }

        // fungsi start kamera
        async function startVideo() {
            try {
                const stream = await navigator.mediaDevices.getUserMedia({ video: true });
                video.srcObject = stream;
                video.addEventListener('loadedmetadata', () => {
                    canvas.width = video.videoWidth;
                    canvas.height = video.videoHeight;
                    streaming = true;
                    statusDiv.innerHTML = "🎥 Kamera aktif · Mendeteksi ekspresi wajah...";
                    requestAnimationFrame(detectFrame);
                });
            } catch (err) {
                statusDiv.innerHTML = "⚠️ Izin kamera ditolak atau kamera tidak tersedia. Refresh dan izinkan kamera.";
                console.error(err);
            }
        }

        // deteksi setiap frame
        async function detectFrame() {
            if (!streaming || !modelsLoaded) {
                requestAnimationFrame(detectFrame);
                return;
            }
            if (video.readyState !== 4 || video.videoWidth === 0) {
                requestAnimationFrame(detectFrame);
                return;
            }

            const displaySize = { width: video.videoWidth, height: video.videoHeight };
            // deteksi ekspresi
            const detections = await faceapi.detectAllFaces(video, new faceapi.TinyFaceDetectorOptions())
                .withFaceExpressions();
            
            // gambar canvas overlay
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            faceapi.matchDimensions(canvas, displaySize);
            
            const resizedDetections = faceapi.resizeResults(detections, displaySize);
            // gambar bounding box (opsional, biar keren)
            faceapi.draw.drawDetections(canvas, resizedDetections);
            faceapi.draw.drawFaceExpressions(canvas, resizedDetections, 0.05);
            
            // update panel ekspresi berdasarkan wajah pertama
            if (detections.length > 0) {
                const expressions = detections[0].expressions;
                if (expressions) {
                    happySpan.innerText = (expressions.happy * 100).toFixed(1) + '%';
                    sadSpan.innerText = (expressions.sad * 100).toFixed(1) + '%';
                    angrySpan.innerText = (expressions.angry * 100).toFixed(1) + '%';
                    surpriseSpan.innerText = (expressions.surprise * 100).toFixed(1) + '%';
                    neutralSpan.innerText = (expressions.neutral * 100).toFixed(1) + '%';
                    disgustSpan.innerText = (expressions.disgust * 100).toFixed(1) + '%';
                    fearSpan.innerText = (expressions.fear * 100).toFixed(1) + '%';
                    
                    // bonus: ubah warna status berdasarkan ekspresi dominan
                    let mainExpr = Object.keys(expressions).reduce((a,b) => expressions[a] > expressions[b] ? a : b);
                    statusDiv.innerHTML = `🧐 Ekspresi dominan: ${mainExpr.toUpperCase()}  ·  kamera realtime aktif`;
                }
            } else {
                // jika wajah tidak terdeteksi
                happySpan.innerText = '-'; sadSpan.innerText = '-'; angrySpan.innerText = '-';
                surpriseSpan.innerText = '-'; neutralSpan.innerText = '-'; disgustSpan.innerText = '-'; fearSpan.innerText = '-';
                statusDiv.innerHTML = "😿 Wajah tidak terdeteksi. Pastikan wajah terlihat jelas & pencahayaan cukup.";
            }
            
            requestAnimationFrame(detectFrame);
        }

        // reset / restart kamera (jika error bisa reload stream)
        document.getElementById('resetCamBtn').addEventListener('click', () => {
            if (video.srcObject) {
                let tracks = video.srcObject.getTracks();
                tracks.forEach(track => track.stop());
            }
            streaming = false;
            statusDiv.innerHTML = "🔄 Mereset kamera & model...";
            if (animationId) cancelAnimationFrame(animationId);
            startVideo();
        });

        // cek apakah model tersedia / inisiasi
        // Coba load dulu dengan timeout handling jika fail
        loadModels();

        // peringatan jika tidak support
        if (!navigator.mediaDevices || !navigator.mediaDevices.getUserMedia) {
            statusDiv.innerHTML = "❌ Browser tidak mendukung akses kamera. Gunakan Chrome/Edge/Firefox terbaru.";
            document.querySelectorAll('.emotion-card').forEach(c => c.style.opacity = 0.4);
        }
    })();
</script>
</body>
</html>
