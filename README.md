<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>✨ Game Salon-Salonan | Stylist & Pelanggan Bergerak ✨</title>
    <style>
        * {
            user-select: none;
            -webkit-tap-highlight-color: transparent;
        }

        body {
            background: linear-gradient(145deg, #2b5e3b 0%, #1e3a2f 100%);
            min-height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
            font-family: 'Segoe UI', 'Quicksand', system-ui, -apple-system, 'Poppins', sans-serif;
            margin: 0;
            padding: 20px;
        }

        /* Container utama game */
        .game-container {
            background: #f9e5c0;
            border-radius: 64px 64px 48px 48px;
            box-shadow: 0 25px 40px rgba(0,0,0,0.5), inset 0 1px 3px rgba(255,255,240,0.8);
            padding: 20px 24px 24px 24px;
        }

        canvas {
            display: block;
            margin: 0 auto;
            border-radius: 36px;
            box-shadow: 0 12px 28px black;
            cursor: pointer;
            background-color: #ffe8d4;
        }

        /* Panel informasi & kontrol */
        .info-panel {
            margin-top: 20px;
            background: #f3d9b1;
            border-radius: 50px;
            padding: 12px 24px;
            display: flex;
            flex-wrap: wrap;
            justify-content: space-between;
            align-items: center;
            gap: 12px;
            box-shadow: inset 0 1px 5px rgba(98, 52, 16, 0.2), 0 5px 10px rgba(0,0,0,0.2);
        }

        .score-box {
            background: #5e2e1c;
            color: #ffefcf;
            padding: 6px 18px;
            border-radius: 60px;
            font-weight: bold;
            font-size: 1.3rem;
            letter-spacing: 1px;
            box-shadow: inset 0 1px 3px #a56b3a, 0 3px 5px rgba(0,0,0,0.2);
        }

        .score-box span {
            font-size: 1.8rem;
            font-weight: 800;
            color: #ffd966;
            margin-left: 8px;
        }

        .controls {
            display: flex;
            gap: 12px;
            background: #e9c892;
            padding: 6px 16px;
            border-radius: 40px;
        }

        button {
            background: #ffb347;
            border: none;
            font-size: 1.2rem;
            font-weight: bold;
            padding: 8px 20px;
            border-radius: 40px;
            color: #38200f;
            cursor: pointer;
            font-family: inherit;
            transition: 0.1s linear;
            box-shadow: 0 3px 0 #7b3f00;
        }

        button:active {
            transform: translateY(2px);
            box-shadow: 0 1px 0 #7b3f00;
        }

        .status {
            background: #f4e2b3;
            border-radius: 32px;
            padding: 6px 20px;
            font-weight: bold;
            font-size: 1rem;
            display: flex;
            align-items: center;
            gap: 12px;
            flex-wrap: wrap;
        }

        .customer-name {
            background: #ac6e3c;
            color: white;
            padding: 4px 14px;
            border-radius: 40px;
            font-size: 0.9rem;
        }

        .tooltip-msg {
            background: #1e2a1e;
            color: #ffe6b3;
            padding: 4px 12px;
            border-radius: 26px;
            font-size: 0.75rem;
        }

        @media (max-width: 650px) {
            .info-panel { flex-direction: column; align-items: stretch; }
            .controls { justify-content: center; }
            button { padding: 6px 12px; font-size: 1rem; }
            .score-box { text-align: center; }
        }
    </style>
</head>
<body>
<div>
    <div class="game-container">
        <canvas id="gameCanvas" width="900" height="550" style="width:100%; height:auto; max-width:900px; aspect-ratio:900/550"></canvas>

        <div class="info-panel">
            <div class="score-box">💇‍♀️ HASIL STYLING <span id="scoreValue">0</span></div>
            <div class="controls">
                <button id="resetBtn">🔄 RESET GAME</button>
                <button id="serviceBtn">✨ LAKUKAN PERAWATAN ✨</button>
            </div>
            <div class="status">
                <div class="customer-name">🧑‍🦱 PELANGGAN: <span id="customerName">Lala</span></div>
                <div class="tooltip-msg">💡 klik & seret stylist (wanita) ke pelanggan!</div>
            </div>
        </div>
    </div>
</div>

<script>
    (function(){
        // --- CANVAS & KONTEKS ---
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');

        // --- DIMENSI CANVAS ---
        const CW = 900;
        const CH = 550;
        canvas.width = CW;
        canvas.height = CH;

        // --- GLOBAL STATE ---
        let score = 0;                  // total poin perawatan
        let customerMood = "senang";    // senang, biasa, sedikit bosan (mempengaruhi animasi wajah)
        let lastServiceTime = 0;
        
        // --- OBJEK STYLIST (karyawan salon yang bisa digerakkan) ---
        let stylist = {
            x: 200, y: 350,
            radius: 28,
            isDragging: false,
            dragOffsetX: 0,
            dragOffsetY: 0
        };
        
        // --- OBJEK PELANGGAN (bergerak secara random di area salon) ---
        let customer = {
            x: 650, y: 280,
            radius: 30,
            vx: 0.6, vy: 0.5,
            name: "Lala",
            moodLevel: 80,     // 0-100, semakin rendah makin cemberut
            styleCount: 0      // sudah berapa kali dirawat
        };
        
        // Daftar nama random pelanggan (ganti setiap beberapa saat / setelah dirawat)
        const namePool = ["Mila", "Sari", "Tia", "Nina", "Risa", "Vina", "Luna", "Dewi", "Ica", "Ocha"];
        
        // --- AREA BATAS AGAR TIDAK KELUAR CANVAS (plus padding margin)---
        const minX = 45;
        const maxX = CW - 45;
        const minY = 70;
        const maxY = CH - 65;
        
        // --- VARIABEL UNTUK ANIMASI EFK 'HAPPY' SAAT DI SERVICE---
        let serviceEffect = { active: false, timer: 0, x: 0, y: 0 };
        
        // --- FUNGSI UPDATE NAMA PELANGGAN RANDOM (ganti sedikit setelah servis)---
        function updateRandomCustomerName() {
            let newName = namePool[Math.floor(Math.random() * namePool.length)];
            // jangan terlalu sering sama
            if(newName === customer.name && namePool.length > 1) {
                newName = namePool[(namePool.indexOf(newName)+1)%namePool.length];
            }
            customer.name = newName;
            document.getElementById('customerName').innerText = customer.name;
        }
        
        // --- UPDATE MOOD BERDASARKAN MOODLEVEL & LAST SERVICE TIME---
        function updateMoodByLevel() {
            // moodLevel turun perlahan seiring waktu? (dipanggil setiap frame)
            // Agar menarik, mood turun 0.05 per frame (kurang lebih 3 per detik di 60fps)
            // Batas bawah 0, atas 100
            if(customer.moodLevel > 0) {
                customer.moodLevel = Math.max(0, customer.moodLevel - 0.04);
            }
            // Tentukan mood string berdasarkan moodLevel
            if(customer.moodLevel >= 65) customerMood = "senang";
            else if(customer.moodLevel >= 30) customerMood = "biasa";
            else customerMood = "sedih";
            
            // jika moodLevel terlalu rendah (<15) pelanggan bisa bergerak lebih lambat? tapi biar dinamis
            if(customer.moodLevel < 15) {
                // kurangi kecepatan sedikit agar terlihat lesu
                customer.vx = Math.sign(customer.vx) * 0.3;
                customer.vy = Math.sign(customer.vy) * 0.3;
            } else {
                // kecepatan normal antara 0.4 - 1.0
                if(Math.abs(customer.vx) < 0.5) customer.vx = (customer.vx>0?0.6:-0.6);
                if(Math.abs(customer.vy) < 0.5) customer.vy = (customer.vy>0?0.6:-0.6);
                // batasi maksimal
                customer.vx = Math.min(1.2, Math.max(-1.2, customer.vx));
                customer.vy = Math.min(1.1, Math.max(-1.1, customer.vy));
            }
        }
        
        // --- GERAK PELANGGAN (BOUNCE DI DALAM BATAS) ---
        function moveCustomer() {
            let newX = customer.x + customer.vx;
            let newY = customer.y + customer.vy;
            
            // pantul jika melewati batas
            if(newX - customer.radius < minX) {
                newX = minX + customer.radius;
                customer.vx = -customer.vx;
            }
            if(newX + customer.radius > maxX) {
                newX = maxX - customer.radius;
                customer.vx = -customer.vx;
            }
            if(newY - customer.radius < minY) {
                newY = minY + customer.radius;
                customer.vy = -customer.vy;
            }
            if(newY + customer.radius > maxY) {
                newY = maxY - customer.radius;
                customer.vy = -customer.vy;
            }
            customer.x = newX;
            customer.y = newY;
            
            // sedikit variasi arah random setiap frame (agar lebih hidup)
            if(Math.random() < 0.02) {
                customer.vx += (Math.random() - 0.5) * 0.3;
                customer.vy += (Math.random() - 0.5) * 0.3;
                // batasi kecepatan
                let maxSpeed = 1.4;
                customer.vx = Math.min(maxSpeed, Math.max(-maxSpeed, customer.vx));
                customer.vy = Math.min(maxSpeed, Math.max(-maxSpeed, customer.vy));
            }
        }
        
        // --- PROSES PERAWATAN (Stylist menyentuh customer) ---
        function performService() {
            // hitung jarak antara stylist dan customer
            const dx = stylist.x - customer.x;
            const dy = stylist.y - customer.y;
            const dist = Math.hypot(dx, dy);
            const collisionDist = stylist.radius + customer.radius;
            
            if(dist < collisionDist + 12) {  // dalam jangkauan sentuhan
                // tambah skor sesuai mood dan style count
                let pointsGain = 0;
                if(customerMood === "senang") pointsGain = 15;
                else if(customerMood === "biasa") pointsGain = 10;
                else pointsGain = 20;   // kalau sedih, memperbaiki mood lebih valuable (poin lebih)
                
                // tambah bonus jika pertama kali?
                let multiplier = 1;
                if(customer.styleCount === 0) multiplier = 1.2;
                let finalGain = Math.floor(pointsGain * multiplier);
                score += finalGain;
                
                // update tampilan score
                document.getElementById('scoreValue').innerText = score;
                
                // naikkan mood level secara drastis (pelanggan senang)
                customer.moodLevel = Math.min(100, customer.moodLevel + 38);
                customer.styleCount++;
                
                // efek partikel senang
                serviceEffect.active = true;
                serviceEffect.timer = 18;
                serviceEffect.x = customer.x;
                serviceEffect.y = customer.y - 12;
                
                // ganti nama pelanggan setelah 2x servis? supaya segar
                if(customer.styleCount % 2 === 0) {
                    updateRandomCustomerName();
                }
                
                // sedikit gerakan mundur pada pelanggan karena terkejut senang (opsional)
                customer.vx += (Math.random() - 0.5)*0.8;
                customer.vy += (Math.random() - 0.5)*0.8;
                
                // update mood setelah service
                updateMoodByLevel(); 
                // feedback suara kecil via getaran style? tidak pakai suara biar ringan
                return true;
            }
            return false;
        }
        
        // --- RESET GAME ---
        function resetGame() {
            score = 0;
            document.getElementById('scoreValue').innerText = "0";
            customer.styleCount = 0;
            customer.moodLevel = 75;
            customer.name = "Lala";
            document.getElementById('customerName').innerText = "Lala";
            customer.x = 680; customer.y = 280;
            customer.vx = 0.7; customer.vy = 0.6;
            stylist.x = 220; stylist.y = 350;
            updateMoodByLevel();
            serviceEffect.active = false;
        }
        
        // --- DRAW SEMUA ELEMEN (Stylist, Customer, Efek, Salon)---
        function drawSalonBackground() {
            // Lantai kayu salon
            ctx.fillStyle = "#e7bc8e";
            ctx.fillRect(0, 0, CW, CH);
            // Dekorasi lantai pola ubin
            ctx.fillStyle = "#d4a373";
            for(let i=0;i<80;i++) {
                ctx.beginPath();
                ctx.arc(40 + (i*37)%CW, CH-15 + (Math.sin(i)*6), 4, 0, Math.PI*2);
                ctx.fill();
            }
            // Meja rias/kursi salon (dekorasi)
            ctx.fillStyle = "#b97f45";
            ctx.shadowBlur = 0;
            ctx.fillRect(30, CH-90, 130, 40);
            ctx.fillStyle = "#c29261";
            ctx.fillRect(40, CH-100, 40, 30);
            ctx.fillStyle = "#9c6b42";
            ctx.fillRect(750, CH-110, 120, 50);
            // Cermin besar
            ctx.fillStyle = "#cbab89";
            ctx.fillRect(780, 70, 90, 120);
            ctx.fillStyle = "#ffe6d0";
            ctx.fillRect(785, 75, 80, 110);
            ctx.fillStyle = "#9e7b5c";
            ctx.beginPath();
            ctx.rect(785, 75, 80, 110);
            ctx.stroke();
            // hiasan
            ctx.font = "24px 'Segoe UI Emoji'";
            ctx.fillStyle = "#ffaa66";
            ctx.fillText("💄", 810, 150);
            ctx.fillText("✂️", 835, 180);
            
            // rak botol
            ctx.fillStyle = "#ad7a4b";
            ctx.fillRect(140, 30, 110, 18);
            for(let i=0;i<5;i++) ctx.fillStyle = "#ffe0b5", ctx.fillRect(150+i*20, 12, 12, 20);
        }
        
        function drawStylist() {
            // stylist (wanita dengan seragam salon)
            ctx.save();
            ctx.shadowBlur = 5;
            ctx.shadowColor = "rgba(0,0,0,0.2)";
            ctx.beginPath();
            ctx.arc(stylist.x, stylist.y, stylist.radius, 0, Math.PI*2);
            ctx.fillStyle = "#f7cfb0";
            ctx.fill();
            ctx.fillStyle = "#ffb882";
            ctx.beginPath();
            ctx.ellipse(stylist.x-4, stylist.y-6, 4, 6, 0, 0, Math.PI*2);
            ctx.ellipse(stylist.x+4, stylist.y-6, 4, 6, 0, 0, Math.PI*2);
            ctx.fill();
            // mata
            ctx.fillStyle = "#3f2a1a";
            ctx.beginPath();
            ctx.arc(stylist.x-5, stylist.y-9, 2, 0, Math.PI*2);
            ctx.arc(stylist.x+5, stylist.y-9, 2, 0, Math.PI*2);
            ctx.fill();
            ctx.fillStyle = "white";
            ctx.beginPath();
            ctx.arc(stylist.x-6, stylist.y-10, 0.7, 0, Math.PI*2);
            ctx.arc(stylist.x+4, stylist.y-10, 0.7, 0, Math.PI*2);
            ctx.fill();
            // senyum
            ctx.beginPath();
            ctx.arc(stylist.x, stylist.y-2, 10, 0.05, Math.PI - 0.05);
            ctx.strokeStyle = "#a5532b";
            ctx.lineWidth = 2;
            ctx.stroke();
            // rambut / topi salon
            ctx.fillStyle = "#966542";
            ctx.beginPath();
            ctx.ellipse(stylist.x, stylist.y-17, 15, 12, 0, 0, Math.PI*2);
            ctx.fill();
            ctx.fillStyle = "#faeba5";
            ctx.font = "bold 20px sans-serif";
            ctx.fillText("💇‍♀️", stylist.x-12, stylist.y-20);
            // seragam apron
            ctx.fillStyle = "#acd3a3";
            ctx.beginPath();
            ctx.rect(stylist.x-15, stylist.y-2, 30, 22);
            ctx.fill();
            ctx.fillStyle = "#fc9e5f";
            ctx.font = "bold 16px monospace";
            ctx.fillText("SALON", stylist.x-12, stylist.y+8);
            ctx.restore();
        }
        
        function drawCustomer() {
            ctx.save();
            ctx.shadowBlur = 3;
            let faceColor = "#fee3c0";
            if(customerMood === "sedih") faceColor = "#f7cfaa";
            ctx.beginPath();
            ctx.arc(customer.x, customer.y, customer.radius, 0, Math.PI*2);
            ctx.fillStyle = faceColor;
            ctx.fill();
            ctx.fillStyle = "#916e48";
            // rambut
            ctx.beginPath();
            ctx.ellipse(customer.x, customer.y-19, 18, 16, 0, 0, Math.PI*2);
            ctx.fillStyle = "#7a5338";
            ctx.fill();
            // mata sesuai mood
            ctx.fillStyle = "#2b251c";
            if(customerMood === "senang") {
                ctx.beginPath();
                ctx.arc(customer.x-9, customer.y-6, 2.5, 0, Math.PI*2);
                ctx.arc(customer.x+9, customer.y-6, 2.5, 0, Math.PI*2);
                ctx.fill();
                ctx.fillStyle = "white";
                ctx.beginPath();
                ctx.arc(customer.x-10, customer.y-7, 1, 0, Math.PI*2);
                ctx.arc(customer.x+8, customer.y-7, 1, 0, Math.PI*2);
                ctx.fill();
                // mulut senyum lebar
                ctx.beginPath();
                ctx.arc(customer.x, customer.y+3, 12, 0.1, Math.PI - 0.1);
                ctx.strokeStyle = "#b35f2a";
                ctx.lineWidth = 2.5;
                ctx.stroke();
            } else if(customerMood === "biasa") {
                ctx.beginPath();
                ctx.arc(customer.x-9, customer.y-6, 2.2, 0, Math.PI*2);
                ctx.arc(customer.x+9, customer.y-6, 2.2, 0, Math.PI*2);
                ctx.fill();
                ctx.beginPath();
                ctx.moveTo(customer.x-8, customer.y+4);
                ctx.lineTo(customer.x+8, customer.y+4);
                ctx.stroke();
            } else { // sedih
                ctx.beginPath();
                ctx.arc(customer.x-9, customer.y-5, 2, 0, Math.PI*2);
                ctx.arc(customer.x+9, customer.y-5, 2, 0, Math.PI*2);
                ctx.fill();
                ctx.beginPath();
                ctx.arc(customer.x, customer.y+4, 8, 0.05, Math.PI - 0.05);
                ctx.strokeStyle = "#aa5f2e";
                ctx.stroke();
                // air mata
                ctx.fillStyle = "#73b9ff";
                ctx.beginPath();
                ctx.ellipse(customer.x-13, customer.y-2, 2, 3, 0, 0, Math.PI*2);
                ctx.fill();
            }
            // blush on
            ctx.fillStyle = "#ffbc9a";
            ctx.globalAlpha = 0.5;
            ctx.beginPath();
            ctx.ellipse(customer.x-13, customer.y+2, 4, 2.5, 0, 0, Math.PI*2);
            ctx.ellipse(customer.x+13, customer.y+2, 4, 2.5, 0, 0, Math.PI*2);
            ctx.fill();
            ctx.globalAlpha = 1;
            
            // aksesori pelanggan
            ctx.fillStyle = "#ffc0cb";
            ctx.font = "20px sans-serif";
            ctx.fillText("💅", customer.x+18, customer.y-12);
            ctx.restore();
        }
        
        function drawServiceEffect() {
            if(serviceEffect.active && serviceEffect.timer > 0) {
                for(let i=0;i<5;i++) {
                    ctx.fillStyle = `rgba(255, 200, 100, ${serviceEffect.timer/15})`;
                    ctx.beginPath();
                    ctx.arc(serviceEffect.x + (Math.sin(Date.now()+i)*6), serviceEffect.y - i*3 + (Math.cos(Date.now()*0.01)*4), 5, 0, Math.PI*2);
                    ctx.fill();
                }
                ctx.font = "bold 24px 'Segoe UI Emoji'";
                ctx.fillStyle = `rgba(255,220,80,${serviceEffect.timer/12})`;
                ctx.fillText("✨✨", serviceEffect.x-15, serviceEffect.y-12);
                serviceEffect.timer--;
                if(serviceEffect.timer<=0) serviceEffect.active=false;
            }
        }
        
        // info mood bar (display)
        function drawMoodBar() {
            ctx.fillStyle = "#a76b3f";
            ctx.fillRect(customer.x-28, customer.y-32, 56, 9);
            ctx.fillStyle = "#fdd17e";
            let moodPercent = customer.moodLevel / 100;
            ctx.fillRect(customer.x-28, customer.y-32, 56 * moodPercent, 9);
            ctx.fillStyle = "#4d2f1a";
            ctx.font = "bold 12px monospace";
            ctx.fillText("kepuasan", customer.x-20, customer.y-35);
        }
        
        function drawChatHint() {
            if(!serviceEffect.active && (customer.moodLevel < 45)) {
                ctx.font = "italic 15px 'Segoe UI'";
                ctx.fillStyle = "#bc6f3c";
                ctx.shadowBlur = 0;
                ctx.fillText("💬 Aku ingin di salonin...", customer.x-40, customer.y-44);
            } else if(customerMood === "senang" && customer.moodLevel>85) {
                ctx.fillStyle = "#4f8533";
                ctx.fillText("💖 stylist keren!", customer.x-33, customer.y-44);
            }
        }
        
        // --- ANIMASI LOOP ---
        function animate() {
            // update mood pelanggan secara natural
            updateMoodByLevel();
            moveCustomer();
            
            // redraw semua
            drawSalonBackground();
            drawStylist();
            drawCustomer();
            drawMoodBar();
            drawChatHint();
            drawServiceEffect();
            
            // jika stylist sedang dalam jangkauan pelanggan, tampilkan efek hover
            const distToCust = Math.hypot(stylist.x - customer.x, stylist.y - customer.y);
            if(distToCust < stylist.radius + customer.radius + 10) {
                ctx.font = "bold 14px system-ui";
                ctx.fillStyle = "#e28743";
                ctx.shadowBlur = 4;
                ctx.fillText("✨ klik 'PERAWATAN' atau sentuh ✨", stylist.x-60, stylist.y-25);
            }
            
            requestAnimationFrame(animate);
        }
        
        // --- EVENT MOUSE / TOUCH UNTUK MENGGERAKKAN STYLIST (drag)---
        function handleDragStart(e) {
            e.preventDefault();
            const rect = canvas.getBoundingClientRect();
            const scaleX = canvas.width / rect.width;
            const scaleY = canvas.height / rect.height;
            let clientX, clientY;
            if(e.touches) {
                clientX = e.touches[0].clientX;
                clientY = e.touches[0].clientY;
            } else {
                clientX = e.clientX;
                clientY = e.clientY;
            }
            let canvasX = (clientX - rect.left) * scaleX;
            let canvasY = (clientY - rect.top) * scaleY;
            const dx = canvasX - stylist.x;
            const dy = canvasY - stylist.y;
            if(Math.hypot(dx, dy) < stylist.radius + 10) {
                stylist.isDragging = true;
                stylist.dragOffsetX = canvasX - stylist.x;
                stylist.dragOffsetY = canvasY - stylist.y;
                canvas.style.cursor = 'grabbing';
            }
        }
        
        function handleDragMove(e) {
            if(!stylist.isDragging) return;
            e.preventDefault();
            const rect = canvas.getBoundingClientRect();
            const scaleX = canvas.width / rect.width;
            const scaleY = canvas.height / rect.height;
            let clientX, clientY;
            if(e.touches) {
                clientX = e.touches[0].clientX;
                clientY = e.touches[0].clientY;
            } else {
                clientX = e.clientX;
                clientY = e.clientY;
            }
            let canvasX = (clientX - rect.left) * scaleX;
            let canvasY = (clientY - rect.top) * scaleY;
            let newX = canvasX - stylist.dragOffsetX;
            let newY = canvasY - stylist.dragOffsetY;
            // batasi area
            newX = Math.min(maxX - stylist.radius, Math.max(minX + stylist.radius, newX));
            newY = Math.min(maxY - stylist.radius, Math.max(minY + stylist.radius, newY));
            stylist.x = newX;
            stylist.y = newY;
        }
        
        function handleDragEnd(e) {
            stylist.isDragging = false;
            canvas.style.cursor = 'grab';
        }
        
        // --- BUTTON SERVICE (manual) & Reset---
        document.getElementById('serviceBtn').addEventListener('click', () => {
            performService();
        });
        document.getElementById('resetBtn').addEventListener('click', () => {
            resetGame();
        });
        
        // Pasang event mouse + touch
        canvas.addEventListener('mousedown', handleDragStart);
        window.addEventListener('mousemove', handleDragMove);
        window.addEventListener('mouseup', handleDragEnd);
        canvas.addEventListener('touchstart', handleDragStart, {passive:false});
        window.addEventListener('touchmove', handleDragMove, {passive:false});
        window.addEventListener('touchend', handleDragEnd);
        canvas.style.cursor = 'grab';
        
        // inisialisasi
        resetGame();
        animate();
    })();
</script>
</body>
</html>
