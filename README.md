<!doctype html>
<html lang="vi">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1">
<meta name="theme-color" content="#08111f">
<title>MineCoin</title>
<style>
:root{color-scheme:dark;--bg:#08111f;--panel:#111e31;--cyan:#45e6df;--green:#b4f36b;--muted:#9aacc2}
*{box-sizing:border-box}
body{margin:0;background:radial-gradient(circle at 50% 0,#18334b 0,#08111f 55%);font-family:system-ui,-apple-system,sans-serif;color:#f4f8ff;min-height:100vh}
main{max-width:480px;margin:auto;padding:22px 16px 36px}
header{display:flex;align-items:center;justify-content:space-between;margin-bottom:18px}
.logo{font-size:21px;font-weight:900;letter-spacing:1px}.logo span{color:var(--cyan)}
.pill{border:1px solid #2b465d;border-radius:99px;padding:6px 10px;color:var(--muted);font-size:12px}
.card{background:linear-gradient(145deg,#15263b,#0e1929);border:1px solid #243b53;border-radius:22px;padding:18px;margin-bottom:14px;box-shadow:0 10px 28px #0003}
.label{font-size:12px;text-transform:uppercase;letter-spacing:1.3px;color:var(--muted)}
.balance{font-size:36px;font-weight:900;margin:5px 0;color:var(--green);overflow-wrap:anywhere}
.unit{font-size:14px;color:var(--muted)}
.machine-area{text-align:center;padding:16px 0 8px}
.machine{font-size:94px;filter:drop-shadow(0 0 18px #45e6df55);line-height:1.2}
.machine-caption{font-size:14px;color:var(--muted);margin:8px 0 16px}
button{font:inherit;border:0;cursor:pointer;touch-action:manipulation}
button:active{transform:scale(.98)}
.primary{width:100%;border-radius:15px;padding:16px;font-weight:900;color:#06131d;background:linear-gradient(90deg,var(--cyan),var(--green));box-shadow:0 6px 20px #45e6df22}
.stats{display:grid;grid-template-columns:1fr 1fr;gap:10px}
.stat{background:#0b1727;border:1px solid #243b53;border-radius:15px;padding:13px}
.stat strong{display:block;font-size:20px;margin-top:5px}
.section-title{font-size:17px;font-weight:800;margin:0 0 12px}
.upgrade-row{display:flex;gap:12px;align-items:center}
.upgrade-icon{font-size:34px;background:#0b1727;border-radius:14px;padding:10px}
.upgrade-info{flex:1;min-width:0}
.upgrade-info strong{display:block}
.upgrade-info small{display:block;color:var(--muted);margin-top:4px}
.buy{background:#203a4d;color:var(--cyan);padding:12px;border-radius:12px;font-weight:800}
.buy:disabled{opacity:.4;cursor:not-allowed}
.note{font-size:12px;line-height:1.5;color:var(--muted);text-align:center;padding:4px 12px}
.toast{position:fixed;bottom:20px;left:50%;transform:translateX(-50%);background:#eafffa;color:#07151d;padding:10px 16px;border-radius:99px;font-weight:800;display:none;white-space:nowrap}
</style>
</head>
<body>
<main>
<header><div class="logo">⛏️ MINE<span>COIN</span></div><div class="pill">BẢN THỬ NGHIỆM</div></header>
<section class="card">
  <div class="label">Số dư của bạn</div>
  <div class="balance"><span id="balance">0</span> <span class="unit">MC</span></div>
  <div class="machine-area">
    <div class="machine">⛏️</div>
    <div class="machine-caption">Chạm để đào coin và nâng cấp máy</div>
    <button class="primary" id="mine">⚡ ĐÀO COIN +<span id="tapPower">1</span></button>
  </div>
  <div class="stats">
    <div class="stat"><div class="label">Tốc độ đào</div><strong><span id="rate">0</span> MC/s</strong></div>
    <div class="stat"><div class="label">Cấp máy đào</div><strong>Cấp <span id="level">1</span></strong></div>
  </div>
</section>
<section class="card">
  <h2 class="section-title">🔧 Nâng cấp máy đào</h2>
  <div class="upgrade-row">
    <div class="upgrade-icon">🖥️</div>
    <div class="upgrade-info">
      <strong>Máy đào cơ bản</strong>
      <small>+1 MC mỗi giây sau khi nâng cấp</small>
      <small>Giá: <b><span id="cost">25</span> MC</b></small>
    </div>
    <button class="buy" id="upgrade">NÂNG CẤP</button>
  </div>
</section>
<section class="card">
  <h2 class="section-title">📋 Nhiệm vụ hằng ngày</h2>
  <div class="upgrade-row">
    <div class="upgrade-icon">👆</div>
    <div class="upgrade-info"><strong>Đào 10 lần</strong><small id="questStatus">Tiến độ: 0/10</small></div>
    <button class="buy" id="claim" disabled>+10 MC</button>
  </div>
</section>
<p class="note">MineCoin là game thử nghiệm. MC chỉ là điểm ảo lưu trên thiết bị này, không có giá trị tiền mặt và chưa thể rút. Xóa dữ liệu trình duyệt có thể làm mất tiến trình.</p>
</main>
<div class="toast" id="toast"></div>
<script>
const KEY="minecoin_demo_v1";
const initial={coins:0,level:1,taps:0,claimed:false,last:Date.now()};
let game;
try{game=Object.assign({},initial,JSON.parse(localStorage.getItem(KEY)||"{}"));}catch(e){game={...initial};}
game.last=Date.now();
const $=id=>document.getElementById(id);
const fmt=n=>Math.floor(n).toLocaleString("vi-VN");
const cost=()=>Math.floor(25*Math.pow(1.65,game.level-1));
function save(){try{localStorage.setItem(KEY,JSON.stringify(game));}catch(e){}}
function render(){
 $("balance").textContent=fmt(game.coins);
 $("level").textContent=game.level;
 $("rate").textContent=game.level-1;
 $("tapPower").textContent=game.level;
 $("cost").textContent=fmt(cost());
 $("upgrade").disabled=game.coins<cost();
 $("questStatus").textContent="Tiến độ: "+Math.min(game.taps,10)+"/10";
 $("claim").disabled=game.taps<10||game.claimed;
 $("claim").textContent=game.claimed?"ĐÃ NHẬN":"+10 MC";
}
function toast(msg){$("toast").textContent=msg;$("toast").style.display="block";setTimeout(()=>$("toast").style.display="none",1500);}
$("mine").addEventListener("click",()=>{game.coins+=game.level;game.taps++;save();render();});
$("upgrade").addEventListener("click",()=>{let c=cost();if(game.coins>=c){game.coins-=c;game.level++;save();render();toast("Nâng cấp thành công!");}});
$("claim").addEventListener("click",()=>{if(game.taps>=10&&!game.claimed){game.coins+=10;game.claimed=true;save();render();toast("Đã nhận 10 MC!");}});
setInterval(()=>{game.coins+=Math.max(0,game.level-1);save();render();},1000);
render();
</script>
</body>
</html>
