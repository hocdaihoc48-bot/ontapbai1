# ontapbai1
<!DOCTYPE html>
<html lang="vi">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Game: Phân Loại Ba Thể Của Chất</title>
<style>
    :root {
        --bg: #2d3436;
        --card-bg: #ffffff;
        --solid-color: #0984e3;
        --liquid-color: #e17055;
        --gas-color: #6c5ce7;
        --text-color: #2d3436;
        --success: #00b894;
        --error: #d63031;
    }

    body {
        font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        background-color: var(--bg);
        color: white;
        margin: 0;
        height: 100vh;
        display: flex;
        flex-direction: column;
        overflow: hidden;
        user-select: none;
    }

    /* --- MÀN HÌNH --- */
    #overlay, #win-screen {
        position: fixed; top: 0; left: 0; width: 100%; height: 100%;
        background: rgba(0,0,0,0.95); z-index: 100;
        display: flex; flex-direction: column; align-items: center; justify-content: center;
        transition: opacity 0.5s;
    }
    #win-screen { display: none; z-index: 90; }

    h1 { color: #ffeaa7; text-align: center; margin-bottom: 10px; text-shadow: 2px 2px 0 #000; }
    p { color: #dfe6e9; font-size: 1.2rem; margin-bottom: 30px; text-align: center; max-width: 600px; }

    button {
        padding: 15px 40px; font-size: 1.5rem; background: var(--success);
        color: white; border: none; border-radius: 50px; cursor: pointer;
        box-shadow: 0 5px 0 #009476; font-weight: bold; transition: transform 0.1s;
    }
    button:active { transform: translateY(3px); box-shadow: 0 2px 0 #009476; }
    #start-btn { animation: pulse 1.5s infinite; }

    @keyframes pulse { 0% { transform: scale(1); } 50% { transform: scale(1.05); } 100% { transform: scale(1); } }

    /* --- GAME UI --- */
    #header {
        padding: 15px 30px; background: rgba(0,0,0,0.3);
        display: flex; justify-content: space-between; align-items: center;
        border-bottom: 2px solid #636e72;
    }
    .score-box { font-size: 1.5rem; font-weight: bold; }
    #score { color: #f1c40f; }

    #game-area {
        flex: 1; display: flex; flex-direction: column;
        align-items: center; justify-content: space-between;
        padding: 20px;
    }

    /* --- KHU VỰC THẺ BÀI --- */
    #card-zone {
        width: 100%; height: 220px;
        display: flex; justify-content: center; align-items: center;
        position: relative;
    }
    #placeholder {
        width: 200px; height: 140px; border: 3px dashed #636e72;
        border-radius: 15px; display: flex; justify-content: center; align-items: center;
        color: #636e72; font-weight: bold;
    }

    .card {
        width: 220px; height: 160px;
        background: var(--card-bg); color: var(--text-color);
        border-radius: 15px;
        box-shadow: 0 10px 20px rgba(0,0,0,0.3);
        display: flex; flex-direction: column; justify-content: center; align-items: center;
        padding: 15px; box-sizing: border-box; text-align: center;
        position: absolute; cursor: grab;
        transition: transform 0.1s;
        border-bottom: 5px solid #bdc3c7;
    }
    .card:active { cursor: grabbing; transform: scale(1.05); }
    
    .card-category {
        font-size: 0.8rem; text-transform: uppercase; letter-spacing: 1px;
        color: #636e72; margin-bottom: 10px; font-weight: bold;
        background: #eee; padding: 4px 10px; border-radius: 10px;
    }
    .card-content { font-size: 1.1rem; font-weight: bold; line-height: 1.4; }

    /* --- KHU VỰC THÙNG CHỨA (DROP ZONES) --- */
    #bins-container {
        display: flex; gap: 20px; width: 100%; max-width: 900px; height: 250px;
    }

    .bin {
        flex: 1; border-radius: 20px 20px 0 0;
        background: rgba(255,255,255,0.05);
        border: 4px dashed rgba(255,255,255,0.2);
        display: flex; flex-direction: column; align-items: center; padding-top: 20px;
        transition: all 0.3s; position: relative;
    }

    .bin-label { font-size: 2rem; font-weight: bold; text-transform: uppercase; margin-bottom: 10px; text-shadow: 2px 2px 0 rgba(0,0,0,0.5); }
    .bin-icon { font-size: 4rem; opacity: 0.5; transition: 0.3s; }

    /* Màu sắc riêng cho từng thùng */
    .bin.solid { border-color: var(--solid-color); }
    .bin.solid .bin-label { color: var(--solid-color); }
    
    .bin.liquid { border-color: var(--liquid-color); }
    .bin.liquid .bin-label { color: var(--liquid-color); }
    
    .bin.gas { border-color: var(--gas-color); }
    .bin.gas .bin-label { color: var(--gas-color); }

    /* Hiệu ứng khi kéo thẻ vào */
    .bin.drag-over { transform: scale(1.05); background: rgba(255,255,255,0.1); border-style: solid; box-shadow: 0 0 30px rgba(255,255,255,0.1); }
    .bin.drag-over .bin-icon { opacity: 1; transform: scale(1.2); }

    /* --- EFFECTS --- */
    .feedback-score {
        position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%);
        font-size: 4rem; font-weight: bold; opacity: 0; pointer-events: none; z-index: 50;
        text-shadow: 3px 3px 0 rgba(0,0,0,0.3);
    }
    @keyframes floatUp { 
        0% { opacity: 1; transform: translate(-50%, -50%) scale(0.5); } 
        50% { transform: translate(-50%, -150%) scale(1.2); }
        100% { opacity: 0; transform: translate(-50%, -200%) scale(1); } 
    }
    
    @keyframes shake {
        0%, 100% { transform: translateX(0); }
        25% { transform: translateX(-15px) rotate(-5deg); }
        75% { transform: translateX(15px) rotate(5deg); }
    }

    .confetti { position: absolute; width: 10px; height: 10px; background: #f1c40f; animation: fall linear infinite; }
    @keyframes fall { to { transform: translateY(100vh) rotate(720deg); } }

</style>
</head>
<body>

<div id="overlay">
    <h1>KÉO THẢ: 3 THỂ CỦA CHẤT</h1>
    <p>Kéo thẻ đặc điểm vào đúng ô Rắn, Lỏng hoặc Khí.<br>Cẩn thận: Có những đặc điểm đúng với cả 2 thể!</p>
    <button id="start-btn">BẮT ĐẦU CHƠI ▶</button>
</div>

<div id="win-screen">
    <h1 style="color: #f1c40f; font-size: 3rem;">HOÀN THÀNH!</h1>
    <p>Tổng điểm của bạn:</p>
    <div style="font-size: 4rem; font-weight: bold; margin-bottom: 20px; color: #fff;" id="final-score">0</div>
    <button id="restart-btn">CHƠI LẠI ↺</button>
</div>

<div id="header">
    <div class="score-box">Điểm: <span id="score">0</span></div>
    <div style="font-size: 1.1rem; color: #b2bec3;">Thẻ còn lại: <span id="remaining">0</span></div>
</div>

<div id="game-area">
    <div id="card-zone">
        <div id="placeholder">Hết câu hỏi</div>
    </div>

    <div id="bins-container">
        <div class="bin solid" data-type="solid">
            <div class="bin-label">RẮN</div>
            <div class="bin-icon">🧊</div>
        </div>
        <div class="bin liquid" data-type="liquid">
            <div class="bin-label">LỎNG</div>
            <div class="bin-icon">💧</div>
        </div>
        <div class="bin gas" data-type="gas">
            <div class="bin-label">KHÍ</div>
            <div class="bin-icon">☁️</div>
        </div>
    </div>
</div>

<script>
    /**
     * DỮ LIỆU CHUẨN (18 CÂU)
     * validBins: Mảng chứa các thùng chấp nhận được (xử lý đa đáp án)
     */
    const FULL_DATA = [
        // RẮN
        { cat: "Khoảng cách", text: "Rất gần", validBins: ['solid'] },
        { cat: "Lực liên kết", text: "Rất mạnh", validBins: ['solid'] },
        { cat: "Chuyển động", text: "Dao động quanh vị trí cân bằng xác định", validBins: ['solid'] },
        { cat: "Hình dạng", text: "Xác định", validBins: ['solid'] },
        
        // LỎNG
        { cat: "Khoảng cách", text: "Xa hơn khoảng cách giữa các phân tử chất rắn", validBins: ['liquid'] },
        { cat: "Lực liên kết", text: "Yếu hơn liên kết chất rắn", validBins: ['liquid'] },
        { cat: "Chuyển động", text: "Dao động quanh VTCB có thể di chuyển", validBins: ['liquid'] },
        { cat: "Hình dạng", text: "Phụ thuộc bình chứa (Lỏng)", validBins: ['liquid'] },
        
        // KHÍ
        { cat: "Khoảng cách", text: "Rất lớn so với kích thước phân tử", validBins: ['gas'] },
        { cat: "Lực liên kết", text: "Rất yếu", validBins: ['gas'] },
        { cat: "Chuyển động", text: "Chuyển động hỗn loạn", validBins: ['gas'] },
        { cat: "Khả năng nén", text: "Dễ nén", validBins: ['gas'] },
        { cat: "Hình dạng", text: "Theo hình dạng bình chứa (Khí)", validBins: ['gas'] },
        { cat: "Thể tích", text: "Phụ thuộc bình chứa", validBins: ['gas'] },

        // ĐA ĐÁP ÁN (Giao thoa)
        { cat: "Khả năng nén", text: "Rất khó nén", validBins: ['solid', 'liquid'] },
        { cat: "Thể tích", text: "Thể tích xác định", validBins: ['solid', 'liquid'] },
        { cat: "Hình dạng", text: "Phụ thuộc hình dạng bình chứa", validBins: ['liquid', 'gas'] },
        { cat: "Chuyển động", text: "Các hạt luôn chuyển động/dao động", validBins: ['solid', 'liquid', 'gas'] } // Câu mẹo
    ];

    let gameQueue = [];
    let currentCardData = null;
    let currentCardEl = null;
    let score = 0;
    let audioCtx = null;

    // --- HỆ THỐNG ÂM THANH (Synth) ---
    function initAudio() {
        if (!audioCtx) audioCtx = new (window.AudioContext || window.webkitAudioContext)();
    }

    function playSound(type) {
        if (!audioCtx) return;
        const osc = audioCtx.createOscillator();
        const gain = audioCtx.createGain();
        osc.connect(gain);
        gain.connect(audioCtx.destination);
        const now = audioCtx.currentTime;

        if (type === 'correct') {
            // Ting! (Major 3rd)
            osc.type = 'sine';
            osc.frequency.setValueAtTime(523.25, now); // C5
            osc.frequency.setValueAtTime(659.25, now + 0.1); // E5
            gain.gain.setValueAtTime(0.3, now);
            gain.gain.exponentialRampToValueAtTime(0.01, now + 0.3);
            osc.start(now); osc.stop(now + 0.3);
        } else if (type === 'wrong') {
            // Buzz! (Sawtooth low)
            osc.type = 'sawtooth';
            osc.frequency.setValueAtTime(150, now);
            osc.frequency.linearRampToValueAtTime(100, now + 0.3);
            gain.gain.setValueAtTime(0.3, now);
            gain.gain.linearRampToValueAtTime(0.01, now + 0.3);
            osc.start(now); osc.stop(now + 0.3);
        } else if (type === 'win') {
            // Fanfare
            [523, 659, 784, 1046].forEach((f, i) => {
                let o = audioCtx.createOscillator();
                let g = audioCtx.createGain();
                o.connect(g); g.connect(audioCtx.destination);
                o.type = 'triangle';
                o.frequency.value = f;
                g.gain.setValueAtTime(0.2, now + i*0.15);
                g.gain.exponentialRampToValueAtTime(0.01, now + i*0.15 + 0.4);
                o.start(now + i*0.15); o.stop(now + i*0.15 + 0.4);
            });
        }
    }

    // --- GAME LOGIC ---
    function shuffleArray(array) {
        for (let i = array.length - 1; i > 0; i--) {
            const j = Math.floor(Math.random() * (i + 1));
            [array[i], array[j]] = [array[j], array[i]];
        }
        return array;
    }

    function startGame() {
        initAudio();
        document.getElementById('overlay').style.display = 'none';
        document.getElementById('win-screen').style.display = 'none';
        
        score = 0;
        updateScore();
        
        // Lấy ngẫu nhiên 10 câu
        const shuffled = shuffleArray([...FULL_DATA]);
        gameQueue = shuffled.slice(0, 10);
        
        spawnCard();
    }

    function spawnCard() {
        const zone = document.getElementById('card-zone');
        // Xóa thẻ cũ (trừ placeholder)
        const oldCard = document.querySelector('.card');
        if(oldCard) oldCard.remove();

        if (gameQueue.length === 0) {
            endGame();
            return;
        }

        currentCardData = gameQueue.pop();
        document.getElementById('remaining').innerText = gameQueue.length;

        // Tạo thẻ HTML
        const card = document.createElement('div');
        card.className = 'card';
        card.draggable = true;
        card.innerHTML = `
            <div class="card-category">${currentCardData.cat}</div>
            <div class="card-content">${currentCardData.text}</div>
        `;

        // Event Drag
        card.addEventListener('dragstart', (e) => {
            currentCardEl = card;
            setTimeout(() => card.style.opacity = '0.01', 0); // Ẩn hình gốc khi drag
        });
        
        card.addEventListener('dragend', (e) => {
            card.style.opacity = '1';
            currentCardEl = null;
        });

        // Event Touch (Mobile basic support)
        card.addEventListener('touchstart', (e) => {
            currentCardEl = card;
        });

        zone.appendChild(card);
    }

    function handleDrop(e, binType) {
        e.preventDefault();
        const bin = e.target.closest('.bin');
        bin.classList.remove('drag-over');

        if (!currentCardEl) return;

        // Logic check đúng sai (hỗ trợ đa đáp án)
        const validTargets = currentCardData.validBins;
        const isCorrect = validTargets.includes(binType);

        if (isCorrect) {
            // ĐÚNG
            score += 10;
            playSound('correct');
            showFeedback(bin, "+10", "#00b894");
            currentCardEl.remove(); // Xóa thẻ
            spawnCard(); // Ra thẻ mới
        } else {
            // SAI
            score -= 5;
            playSound('wrong');
            showFeedback(bin, "-5", "#d63031");
            
            // Hiệu ứng rung thẻ
            currentCardEl.style.opacity = '1';
            currentCardEl.style.animation = 'shake 0.4s';
            setTimeout(() => {
                if(currentCardEl) currentCardEl.style.animation = '';
            }, 400);
        }
        updateScore();
    }

    function showFeedback(element, text, color) {
        const el = document.createElement('div');
        el.className = 'feedback-score';
        el.innerText = text;
        el.style.color = color;
        element.appendChild(el);
        
        // Kích hoạt animation
        el.style.animation = "floatUp 1s forwards";
        setTimeout(() => el.remove(), 1000);
    }

    function updateScore() {
        document.getElementById('score').innerText = score;
    }

    function endGame() {
        document.getElementById('win-screen').style.display = 'flex';
        document.getElementById('final-score').innerText = score;
        playSound('win');
        createConfetti();
    }

    function createConfetti() {
        // Xóa cũ
        document.querySelectorAll('.confetti').forEach(e => e.remove());
        // Tạo mới
        for(let i=0; i<50; i++) {
            const c = document.createElement('div');
            c.className = 'confetti';
            c.style.left = Math.random() * 100 + 'vw';
            c.style.backgroundColor = `hsl(${Math.random() * 360}, 100%, 50%)`;
            c.style.animationDuration = (2 + Math.random() * 3) + 's';
            document.body.appendChild(c);
        }
    }

    // Gắn sự kiện cho các Thùng chứa
    const bins = document.querySelectorAll('.bin');
    bins.forEach(bin => {
        bin.addEventListener('dragover', (e) => {
            e.preventDefault();
            bin.classList.add('drag-over');
        });
        bin.addEventListener('dragleave', (e) => {
            bin.classList.remove('drag-over');
        });
        bin.addEventListener('drop', (e) => {
            handleDrop(e, bin.dataset.type);
        });
    });

    document.getElementById('start-btn').addEventListener('click', startGame);
    document.getElementById('restart-btn').addEventListener('click', startGame);

</script>
</body>
</html>
