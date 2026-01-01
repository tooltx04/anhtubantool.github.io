<!DOCTYPE html>
<html lang="vi">
<head>
<meta charset="UTF-8">
<title>Casino Hub AI</title>

<style>
/* ===== RESET ===== */
*{margin:0;padding:0;box-sizing:border-box}

/* ===== RAINBOW BACKGROUND ===== */
body{
    min-height:100vh;
    display:flex;
    justify-content:center;
    align-items:center;
    font-family:Arial,Helvetica,sans-serif;
    background:linear-gradient(270deg,red,orange,yellow,green,cyan,blue,violet);
    background-size:1600% 1600%;
    animation:rainbow 15s ease infinite;
}
@keyframes rainbow{
    0%{background-position:0% 50%}
    50%{background-position:100% 50%}
    100%{background-position:0% 50%}
}

/* ===== MAIN CONTAINER ===== */
.container{
    width:520px;
    background:rgba(0,0,0,.65);
    border-radius:22px;
    padding:26px;
    color:#fff;
    box-shadow:0 0 40px rgba(255,255,255,.4);
}

/* ===== TITLE ===== */
h1{
    text-align:center;
    font-size:34px;
    margin-bottom:18px;
    color:#fff;
    text-shadow:
        -2px -2px 0 #fff,
         2px -2px 0 #fff,
        -2px  2px 0 #fff,
         2px  2px 0 #fff,
         0 0 12px gold;
}

/* ===== TABS ===== */
.tabs{display:flex;gap:10px;margin-bottom:18px}
.tab{
    flex:1;
    text-align:center;
    padding:12px;
    border-radius:14px;
    border:2px solid #fff;
    cursor:pointer;
    font-weight:bold;
}
.tab.active{
    background:rgba(255,255,255,.25);
    box-shadow:0 0 14px gold;
}

/* ===== CONTENT ===== */
.content{display:none}
.content.active{display:block}

/* ===== SICBO ===== */
.sicbo-box{
    background:#181c3a;
    padding:25px;
    border-radius:14px;
}
.sicbo-box h2{
    text-align:center;
    color:#7cf3ff;
    margin-bottom:16px;
    text-shadow:0 0 8px #7cf3ff;
}
.sicbo-box input{
    width:100%;
    padding:10px;
    border-radius:6px;
    border:none;
    font-size:16px;
    margin-bottom:10px;
}
.sicbo-box button{
    width:100%;
    padding:10px;
    font-size:16px;
    border:none;
    border-radius:6px;
    background:#3b82f6;
    color:white;
    cursor:pointer;
}
.sicbo-box button:hover{background:#2563eb}
.sicbo-box pre{
    background:#0b0e1d;
    padding:15px;
    border-radius:8px;
    margin-top:15px;
    white-space:pre-wrap;
}

/* ===== SUNWIN ===== */
.sunwin-box{
    background:linear-gradient(145deg,#6a2b2b,#2a0f0f);
    border-radius:18px;
    padding:26px;
    border:2px solid #caa46a;
    box-shadow:0 0 22px rgba(202,164,106,.45);
}
.sunwin-box h2{
    text-align:center;
    margin-bottom:10px;
    color:#fff2dc;
    text-shadow:0 0 10px rgba(255,210,150,.7);
}
.timer{
    text-align:center;
    margin-bottom:16px;
    color:#ffd9a3;
}
.row{
    display:flex;
    justify-content:space-between;
    padding:8px 0;
    border-bottom:1px solid rgba(255,255,255,.15);
}
.row:last-child{border-bottom:none}
.label{color:#ffd9a3}
.value{font-weight:bold;text-align:right}
.footer{
    margin-top:14px;
    text-align:center;
    font-size:13px;
    color:#e6cfa5;
}
</style>
</head>

<body>
<div class="container">
<h1>🎰 CASINO HUB AI</h1>

<div class="tabs">
    <div class="tab active" onclick="showTab('sicbo',this)">🎲 SICBO</div>
    <div class="tab" onclick="showTab('sunwin',this)">🔥 SUNWIN</div>
</div>

<!-- ===== SICBO ===== -->
<div id="sicbo" class="content active">
<div class="sicbo-box">
<h2>🎲 SICBO AI</h2>
<input id="md5Input" placeholder="Nhập MD5 32 ký tự hex">
<button onclick="analyze()">PHÂN TÍCH</button>
<pre id="output"></pre>
</div>
</div>

<!-- ===== SUNWIN ===== -->
<div id="sunwin" class="content">
<div class="sunwin-box">
<h2>🎲 SUNWIN AI</h2>

<div class="timer">⏳ Làm mới sau <span id="countdown">60</span> giây</div>

<div class="row"><span class="label">Phiên</span><span id="phien" class="value">--</span></div>
<div class="row"><span class="label">Xúc xắc</span><span id="xx" class="value">--</span></div>
<div class="row"><span class="label">Tổng</span><span id="tong" class="value">--</span></div>
<div class="row"><span class="label">Kết quả</span><span id="ketqua" class="value">--</span></div>
<div class="row"><span class="label">Phiên hiện tại</span><span id="phienht" class="value">--</span></div>
<div class="row"><span class="label">Dự đoán</span><span id="dudoan" class="value">--</span></div>
<div class="row"><span class="label">Cầu</span><span id="cau" class="value">--</span></div>

<div class="footer">Sunwin Casino Predictor</div>
</div>
</div>
</div>

<script>
/* ===== TAB ===== */
function showTab(id,el){
    document.querySelectorAll('.tab').forEach(t=>t.classList.remove('active'));
    document.querySelectorAll('.content').forEach(c=>c.classList.remove('active'));
    el.classList.add('active');
    document.getElementById(id).classList.add('active');
}

/* ================= SICBO AI – GIỮ NGUYÊN LOGIC ================= */
// (code SICBO bạn gửi – không thay đổi)
const AI_RAND_BOOST_THRESHOLD=3.5,AI_RAND_BOOST_FACTOR=15,DEEP_ENTROPY_WEIGHT=10,DEEP_ENTROPY_DIFF_BASELINE=50,
DEEP_ENTROPY_DIFF_PENALTY=20,DEEP_SYMMETRY_WEIGHT=30,DEEP_REPEAT_PENALTY_WEIGHT=15;

function hexEntropy(hex){const f={};for(let c of hex)f[c]=(f[c]||0)+1;let e=0;for(let c in f){let p=f[c]/hex.length;e-=p*Math.log2(p)}return e}
function deepScore(md5){
    const ent=hexEntropy(md5),freq={};for(let c of md5)freq[c]=(freq[c]||0)+1;
    const repeated=Object.values(freq).filter(v=>v>2).length;
    let sym=0;for(let i=0;i<16;i++)if(md5[i]===md5[i+16])sym++;
    return Math.max(0,Math.min(100,
        ent*DEEP_ENTROPY_WEIGHT+
        (DEEP_ENTROPY_DIFF_BASELINE-ent*DEEP_ENTROPY_DIFF_PENALTY)+
        (sym/16)*DEEP_SYMMETRY_WEIGHT-
        (repeated/Object.keys(freq).length)*DEEP_REPEAT_PENALTY_WEIGHT));
}
function seededRandom(s){let x=Math.sin(s++)*10000;return x-Math.floor(x)}
function analyze(){
    const md5=md5Input.value.trim().toLowerCase();
    if(!/^[0-9a-f]{32}$/.test(md5)){output.textContent="❌ MD5 không hợp lệ";return;}
    let s=parseInt(md5.slice(0,16),16);
    let d1=1+Math.floor(seededRandom(s++)*6),
        d2=1+Math.floor(seededRandom(s++)*6),
        d3=1+Math.floor(seededRandom(s++)*6);
    let sum=d1+d2+d3;
    let entropy=hexEntropy(md5),deep=deepScore(md5);
    output.textContent=
`🎲 ${d1}-${d2}-${d3}
🎯 Tổng: ${sum}
🔑 Entropy: ${entropy.toFixed(3)}
🧠 Deep Score: ${deep.toFixed(3)}`;
}

/* ================= SUNWIN ================= */
let cd=60;
async function loadSunwin(){
    try{
        const r=await fetch("https://sunwinsaygex-tzz9.onrender.com/api/sun");
        const d=await r.json();
        phien.textContent=d.phien;
        xx.textContent=`${d.xuc_xac_1}-${d.xuc_xac_2}-${d.xuc_xac_3}`;
        tong.textContent=d.tong;
        ketqua.textContent=d.ket_qua;
        phienht.textContent=d.phien_hien_tai;
        dudoan.textContent=d.du_doan;
        cau.textContent=d.cau;
    }catch{ketqua.textContent="Lỗi API";}
}
setInterval(()=>{
    countdown.textContent=cd;
    cd--;
    if(cd<0){cd=60;loadSunwin();}
},1000);
loadSunwin();
</script>
</body>
</html>
