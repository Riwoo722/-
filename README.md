# -
재낭머니 LOL
<!DOCTYPE html>
<html lang="ko">
<head>
<meta charset="UTF-8">
<title>재낭 머니 시스템</title>
<style>
body { font-family: Arial; background:#111; color:white; text-align:center; }
.panel { background:#1e1e1e; padding:20px; margin:20px auto; width:90%; max-width:500px; border-radius:15px; }
input { padding:8px; margin:5px; border-radius:8px; border:none; }
button { padding:8px 12px; margin:5px; border-radius:8px; border:none; cursor:pointer; }
.shop-grid { display:grid; grid-template-columns:repeat(auto-fit,minmax(140px,1fr)); gap:10px; }
.card { background:#2a2a2a; padding:10px; border-radius:12px; }
.premium { border:2px solid gold; }
</style>
</head>
<body>

<div class="panel" id="authPanel">
<h2>🔐 회원가입 / 로그인</h2>
<input type="text" id="username" placeholder="아이디">
<input type="password" id="password" placeholder="비밀번호"><br>
<button onclick="register()">회원가입</button>
<button onclick="login()">로그인</button>
<p id="msg"></p>
</div>

<div id="main" style="display:none;">

<div class="panel">
<h2>👤 유저 정보</h2>
<p>아이디: <span id="user"></span></p>
<p>💰 코인: <span id="money"></span></p>
<p>🏆 레벨: <span id="level"></span></p>
<p>✨ 경험치: <span id="exp"></span></p>
<button onclick="attendance()">출석</button>
<button onclick="gamblePrompt()">도박</button>
<button onclick="logout()">로그아웃</button>
</div>

<div class="panel">
<h2>🛒 상점</h2>
<div class="shop-grid">
<div class="card"><h4>소형 출연권</h4><p>2000</p><button onclick="buy('소형 출연권',2000)">구매</button></div>
<div class="card premium"><h4>슈퍼 낭이</h4><p>15000</p><button onclick="buy('슈퍼 낭이',15000)">구매</button></div>
</div>
</div>

<div class="panel">
<h2>🏆 랭킹</h2>
<div id="ranking"></div>
</div>

<div class="panel" id="adminPanel" style="display:none;">
<h2>👑 관리자</h2>
<button onclick="resetAll()">전체 초기화</button>
<button onclick="giveMoney()">코인 지급</button>
<div id="adminOut"></div>
</div>

</div>

<script>
const WEBHOOK_URL="여기에_웹후크_URL";

let currentUser=null;

function sendLog(title,msg){
fetch(WEBHOOK_URL,{method:"POST",headers:{"Content-Type":"application/json"},
body:JSON.stringify({content:`📢 **${title}**\n👤 ${currentUser}\n${msg}\n🕒 ${new Date().toLocaleString()}`})});
}

function register(){
let u=username.value; let p=password.value;
if(localStorage.getItem("user_"+u)) return msg.innerText="이미 존재";
let data={password:p,money:0,level:1,exp:0,streak:0,lastGamble:0};
localStorage.setItem("user_"+u,JSON.stringify(data));
msg.innerText="가입 완료!";
}

function login(){
let u=username.value; let p=password.value;
let data=JSON.parse(localStorage.getItem("user_"+u));
if(!data||data.password!==p) return msg.innerText="로그인 실패";
currentUser=u;
authPanel.style.display="none";
main.style.display="block";
if(u==="admin") adminPanel.style.display="block";
updateUI(); updateRanking();
sendLog("로그인","성공");
}

function logout(){ location.reload(); }

function getData(){ return JSON.parse(localStorage.getItem("user_"+currentUser)); }
function saveData(d){ localStorage.setItem("user_"+currentUser,JSON.stringify(d)); }

function updateUI(){
let d=getData();
user.innerText=currentUser;
money.innerText=d.money;
level.innerText=d.level;
exp.innerText=d.exp+" / "+(d.level*100);
}

function addExp(a){
let d=getData();
d.exp+=a;
while(d.exp>=d.level*100){
d.exp-=d.level*100;
d.level++;
sendLog("레벨업","레벨 "+d.level);
}
saveData(d);
}

function attendance(){
let d=getData();
let reward=Math.floor(Math.random()*251)+50;
if(d.streak>0) reward=Math.floor(reward*1.1);
d.money+=reward; d.streak++;
addExp(Math.floor(reward/2));
saveData(d); updateUI(); updateRanking();
sendLog("출석",`+${reward} 코인`);
}

function gamblePrompt(){
let amt=parseInt(prompt("베팅 금액"));
gamble(amt);
}

function gamble(a){
let d=getData();
if(Date.now()-d.lastGamble<60000) return alert("1분 쿨타임");
if(d.money<a) return alert("코인 부족");
d.money-=a;
let r=Math.floor(Math.random()*11)*10;
if(r===100){ let win=a*5; d.money+=win; sendLog("잭팟",`+${win}`);}
else if(r>=50){ let win=a*2; d.money+=win; sendLog("도박 성공",`+${win}`);}
else sendLog("도박 실패",`-${a}`);
d.lastGamble=Date.now();
addExp(Math.floor(a/2));
saveData(d); updateUI(); updateRanking();
}

function buy(name,price){
let d=getData();
if(d.money<price) return alert("코인 부족");
d.money-=price;
saveData(d); updateUI(); updateRanking();
sendLog("구매",`${name} -${price}`);
}

function updateRanking(){
let arr=[];
for(let i=0;i<localStorage.length;i++){
let k=localStorage.key(i);
if(k.startsWith("user_")){
let d=JSON.parse(localStorage.getItem(k));
arr.push({name:k.replace("user_",""),money:d.money});
}}
arr.sort((a,b)=>b.money-a.money);
ranking.innerHTML=arr.slice(0,10).map((u,i)=>`${i+1}위 ${u.name} - ${u.money}`).join("<br>");
}

function resetAll(){ localStorage.clear(); alert("초기화"); }

function giveMoney(){
let u=prompt("유저"); let a=parseInt(prompt("금액"));
let d=JSON.parse(localStorage.getItem("user_"+u));
d.money+=a;
localStorage.setItem("user_"+u,JSON.stringify(d));
sendLog("관리자 지급",`${u} +${a}`);
}
</script>

</body>
</html>
