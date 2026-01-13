
<html lang="sq">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Bitcoin Vision Ultimate</title>
<style>
html,body{margin:0;padding:0;width:100%;height:100%;overflow:hidden;font-family:monospace;background:#000;color:#0ff;}
body::before{
  content:"";
  position:fixed;inset:0;
  background:linear-gradient(90deg, rgba(0,255,255,0.05) 1px, transparent 1px),
             linear-gradient(rgba(0,255,255,0.05) 1px, transparent 1px);
  background-size:40px 40px;
  animation:slideGrid 10s linear infinite;
  z-index:-1;
}
@keyframes slideGrid{0%{background-position:0 0,0 0;}100%{background-position:40px 40px,40px 40px;}}

#loadingScreen{
  position:fixed;inset:0;display:flex;flex-direction:column;justify-content:center;align-items:center;background:#111;color:#0f0;text-align:center;cursor:pointer;
}
#loadingScreen h1{font-size:48px;margin:0;animation:textGlow 1s infinite alternate;}
#loadingScreen p{font-size:24px;margin-top:10px;}
@keyframes textGlow{0%{text-shadow:0 0 5px #0f0;}100%{text-shadow:0 0 25px #0f0,0 0 50px #0f0;}}

#player{position:absolute;width:50px;height:50px;font-size:42px;text-align:center;line-height:50px;pointer-events:none;filter:drop-shadow(0 0 20px cyan);animation:pulse 1s infinite alternate;display:none;}
@keyframes pulse{0%{transform:scale(1)}50%{transform:scale(1.3)}100%{transform:scale(1)}}

.enemy{position:absolute;width:50px;height:50px;background:#120000;border:2px solid red;box-shadow:0 0 20px red,inset 0 0 10px red;color:#ff4444;text-align:center;line-height:50px;font-weight:bold;border-radius:8px;animation:flicker 0.8s infinite alternate;}
@keyframes flicker{0%,100%{opacity:1}50%{opacity:0.6}}

.coin{position:absolute;width:35px;height:35px;border-radius:50%;background:radial-gradient(circle at 30% 30%, #fffa00,#ffaa00);color:#000;text-align:center;line-height:35px;box-shadow:0 0 20px #fffa00, 0 0 40px #ffaa00 inset;animation:coinGlow 1s infinite alternate;}
@keyframes coinGlow{0%{transform:scale(1)}100%{transform:scale(1.2)}}

#hud{position:fixed;top:10px;left:10px;font-size:20px;color:#0ff;text-shadow:0 0 10px #0ff,0 0 25px #0ff;display:none;}
#barWrap{position:fixed;bottom:10px;left:10px;right:10px;height:18px;background:#111;border-radius:10px;display:none;}
#bar{height:100%;width:0%;background:#0ff;border-radius:10px;box-shadow:0 0 15px #0ff;transition:width 0.3s ease;}

.levelUpText{position:absolute;color:#0ff;font-size:28px;font-weight:bold;text-shadow:0 0 10px #0ff,0 0 25px #0ff;animation:floatUp 1s ease forwards;pointer-events:none;}
@keyframes floatUp{0%{transform:translate(-50%,0) scale(1);opacity:1;}100%{transform:translate(-50%,-60px) scale(1.5);opacity:0;}}

.spark{position:absolute;width:6px;height:6px;background:cyan;border-radius:50%;box-shadow:0 0 10px cyan;pointer-events:none;animation:sparkAnim 0.6s forwards;}
@keyframes sparkAnim{0%{opacity:1;transform:translate(0,0) scale(1);}100%{opacity:0;transform:translate(calc(var(--dx)*15px), calc(var(--dy)*15px)) scale(0.2);}}

.coinSpark{position:absolute;width:4px;height:4px;background:yellow;border-radius:50%;pointer-events:none;box-shadow:0 0 10px yellow;animation:coinS 0.5s forwards;}
@keyframes coinS{0%{transform:scale(1);opacity:1;}100%{transform:scale(0.2) translateY(-30px);opacity:0;}}

#gameOver{display:none;position:fixed;inset:0;background:rgba(0,0,0,0.9);color:#ff0055;font-family:monospace;display:flex;flex-direction:column;justify-content:center;align-items:center;text-shadow:0 0 20px #ff0055,0 0 40px #ff55aa;z-index:1000;}
#gameOver button{padding:12px 25px;font-size:18px;background:#0ff;color:#000;border:none;border-radius:8px;margin-top:15px;font-weight:bold;}
#highscoreList{text-align:center;color:#0f0;margin-top:10px;}
</style>
</head>
<body>

<div id="loadingScreen">
  <h1>BITCOIN VISION ULTIMATE</h1>
  <p>Tap to Start</p>
</div>

<div id="player"></div>
<div id="hud"></div>
<div id="barWrap"><div id="bar"></div></div>

<div id="gameOver">
  <h1>GAME OVER</h1>
  <p id="finalScore"></p>
  <div id="highscoreList"></div>
  <button id="restartBtn">Restart Full Screen</button>
</div>

<script>
let score=0,level=1,isOver=false;
let enemies=[],coins=[];
let playerName="";
const PPL=25;
const emojis=["😈","🤖","👻","🧟","🐉","👽","🦈","🦄"];
const player=document.getElementById("player");
const hud=document.getElementById("hud");
let highscoreList=JSON.parse(localStorage.getItem("highscoreList")||"[]");

// Tap to start
document.getElementById("loadingScreen").onclick=()=>{
  playerName=prompt("Fut emrin tënd:")||"💎Anonim";
  document.getElementById("loadingScreen").style.display="none";
  hud.style.display="block";
  document.getElementById("barWrap").style.display="block";
  if(document.documentElement.requestFullscreen)document.documentElement.requestFullscreen();
  score=0;level=1;isOver=false;
  player.style.display="block";player.style.left="50%";player.style.top="50%";
  startLevel();
}

// Game functions
function startLevel(){
  clearAll();
  player.innerText=emojis[(level-1)%emojis.length];
  spawnEnemies(2+level);spawnCoins(5+level);
  updateHUD();showLevelUp(level);
}

function safePos(size){let x,y;do{x=Math.random()*(innerWidth-size);y=Math.random()*(innerHeight-size);}while(Math.abs(x-innerWidth/2)<80 && Math.abs(y-innerHeight/2)<80);return {x,y};}
function spawnEnemies(n){for(let i=0;i<n;i++){let e=document.createElement("div");e.className="enemy";e.innerText="X";let p=safePos(50);e.style.left=p.x+"px";e.style.top=p.y+"px";e.dx=(Math.random()-0.5)*(1+level);e.dy=(Math.random()-0.5)*(1+level);document.body.appendChild(e);enemies.push(e);}}
function spawnCoins(n){for(let i=0;i<n;i++){let c=document.createElement("div");c.className="coin";c.innerText="₿";let p=safePos(35);c.style.left=p.x+"px";c.style.top=p.y+"px";c.dx=(Math.random()-0.5)*(1+level*0.3);c.dy=(Math.random()-0.5)*(1+level*0.3);document.body.appendChild(c);coins.push(c);}}

document.addEventListener("touchstart",e=>{
  if(isOver) return;
  let t=e.touches[0];
  player.style.left=(t.clientX-25)+"px";
  player.style.top=(t.clientY-25)+"px";
  coins.forEach(c=>{if(hit(player,c,25)){score++;coinEffect(c);let p=safePos(35);c.style.left=p.x+"px";c.style.top=p.y+"px";updateProgressBar();if(score%PPL===0){level++;startLevel();}updateHUD();}});
});

function coinEffect(c){
  const rect=c.getBoundingClientRect();
  for(let i=0;i<8;i++){
    const cs=document.createElement("div");cs.className="coinSpark";
    cs.style.left=(rect.left+17.5)+"px";cs.style.top=(rect.top+17.5)+"px";
    cs.style.setProperty('--dx',(Math.random()-0.5)*2);cs.style.setProperty('--dy',(Math.random()-0.5)*2);
    document.body.appendChild(cs);setTimeout(()=>cs.remove(),500);
  }
}

function updateHUD(){hud.innerText=`${playerName} | Pikë: ${score} | Level: ${level}`;}
function updateProgressBar(){let pct=(score%PPL)/PPL*100;bar.style.width=pct+"%";if(pct>=100){bar.style.boxShadow="0 0 20px #0ff,0 0 40px #0ff inset";setTimeout(()=>bar.style.boxShadow="0 0 15px #0ff",300);}}

setInterval(()=>{
  if(isOver) return;
  enemies.forEach(e=>{move(e,50);if(hit(player,e,25)) gameOver();});
  coins.forEach(c=>move(c,35));
},40);

function move(el,size){let x=parseFloat(el.style.left)+el.dx;let y=parseFloat(el.style.top)+el.dy;if(x<0||x>innerWidth-size) el.dx*=-1;if(y<0||y>innerHeight-size) el.dy*=-1;el.style.left=x+"px";el.style.top=y+"px";}
function hit(a,b,d){return Math.abs(parseInt(a.style.left)-parseInt(b.style.left))<d && Math.abs(parseInt(a.style.top)-parseInt(b.style.top))<d;}
function clearAll(){enemies.forEach(e=>e.remove());coins.forEach(c=>c.remove());enemies=[];coins=[];}

function showLevelUp(lvl){
  const txt=document.createElement("div");txt.className="levelUpText";txt.innerText=`LEVEL ${lvl}!`;
  const rect=player.getBoundingClientRect();
  txt.style.left=rect.left+25+"px";txt.style.top=rect.top+"px";
  document.body.appendChild(txt);
  setTimeout(()=>txt.remove(),1000);
  for(let i=0;i<12;i++){
    const spark=document.createElement("div");spark.className="spark";
    spark.style.left=(rect.left+25)+"px";spark.style.top=(rect.top+25)+"px";
    spark.style.setProperty('--dx',(Math.random()-0.5)*2);
    spark.style.setProperty('--dy',(Math.random()-0.5)*2);
    document.body.appendChild(spark);
    setTimeout(()=>spark.remove(),600);
  }
  player.style.transition="transform 0.6s ease";
  player.style.transform="scale(1.5) rotate(360deg)";
  setTimeout(()=>player.style.transform="scale(1) rotate(0deg)",600);
}

function gameOver(){
  isOver=true;
  highscoreList.push({name:playerName,score:score});
  highscoreList.sort((a,b)=>b.score-a.score);
  if(highscoreList.length>5) highscoreList=highscoreList.slice(0,5);
  localStorage.setItem("highscoreList",JSON.stringify(highscoreList));
  document.getElementById("finalScore").innerText=`${playerName} | Pikë: ${score} | Level: ${level}`;
  document.getElementById("highscoreList").innerHTML=highscoreList.map(h=>`${h.name}: ${h.score}`).join("<br>");
  document.getElementById("gameOver").style.display="flex";
}

document.getElementById("restartBtn").onclick=()=>{
  score=0;level=1;isOver=false;bar.style.width="0%";player.style.left="50%";player.style.top="50%";
  document.getElementById("gameOver").style.display="none";
  startLevel();
}
</script>
</body>
</html>
