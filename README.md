<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Mini FPS Ultimate</title>

<style>
body{margin:0;overflow:hidden;background:#111;color:white;font-family:Arial}
canvas{display:block}
#hud{position:fixed;top:10px;left:10px;z-index:5}

#shoot{
position:fixed;
bottom:80px;
right:40px;
width:90px;
height:90px;
background:red;
border-radius:50%;
display:flex;
align-items:center;
justify-content:center;
font-weight:bold;
z-index:5;
user-select:none;
}

#winScreen{
position:fixed;
top:0;
left:0;
width:100%;
height:100%;
background:rgba(0,0,0,0.9);
display:none;
align-items:center;
justify-content:center;
flex-direction:column;
z-index:2000;
}

#winScreen h1{
font-size:40px;
color:gold;
margin-bottom:20px;
}

#restartBtn{
padding:15px 30px;
font-size:18px;
border:none;
border-radius:8px;
background:limegreen;
color:white;
cursor:pointer;
}

#panel{
position:fixed;
top:60px;
left:20px;
width:320px;
background:#1c1c2e;
border-radius:12px;
box-shadow:0 0 15px black;
z-index:9999;
touch-action:none;
}

#header{
background:#2a2a40;
padding:10px;
display:flex;
justify-content:space-between;
align-items:center;
border-radius:12px 12px 0 0;
cursor:grab;
}

#content{padding:10px}
.tab{margin-top:8px;background:#24243a;padding:8px;border-radius:8px}
.close{color:red;cursor:pointer}
.arrow{cursor:pointer}
</style>
</head>
<body>

<div id="hud">Kills: <span id="kills">0</span></div>
<canvas id="game"></canvas>
<div id="shoot">FIRE</div>

<div id="winScreen">
<h1>🏆 VOCÊ GANHOU!</h1>
<button id="restartBtn">REINICIAR</button>
</div>

<div id="panel">
<div id="header">
<span id="toggle" class="arrow">⬇</span>
<span>Painel</span>
<span id="close" class="close">✖</span>
</div>

<div id="content">

<div class="tab">
<b>🎯 Combate</b><br>
<label><input type="checkbox" id="aimbot"> Aimbot (Head)</label><br>
<label><input type="checkbox" id="autoFire"> Auto Fire</label><br>
FOV: <input type="range" id="fov" min="50" max="600" value="250">
</div>

<div class="tab">
<b>👁 ESP</b><br>
<label><input type="checkbox" id="espBox"> Caixa</label><br>
<label><input type="checkbox" id="espLine"> Linha</label><br>
<label><input type="checkbox" id="espHP"> Vida</label><br>
<label><input type="checkbox" id="espDist"> Distância</label><br>
<label><input type="checkbox" id="espSkeleton"> Esqueleto</label>
</div>

</div>
</div>

<script>
const canvas=document.getElementById("game");
const ctx=canvas.getContext("2d");

function resize(){
canvas.width=window.innerWidth;
canvas.height=window.innerHeight;
player.x=canvas.width/2;
player.y=canvas.height/2;
}
window.addEventListener("resize",resize);

let player={x:0,y:0};
resize();

let enemies=[];
let bullets=[];
let damages=[];
let kills=0;
let firing=false;
let lastShot=0;
let fireDelay=300;
let gameOver=false;

function spawn(){
enemies.push({
x:Math.random()*canvas.width,
y:Math.random()*canvas.height,
hp:200
});
}

function resetGame(){
gameOver=false;
enemies.length=0;
bullets.length=0;
damages.length=0;
kills=0;
document.getElementById("kills").innerText=0;
for(let i=0;i<5;i++)spawn();
document.getElementById("winScreen").style.display="none";
}

for(let i=0;i<5;i++)spawn();

function dist(x1,y1,x2,y2){
return Math.hypot(x2-x1,y2-y1);
}

function shoot(){
if(gameOver)return;
let now=Date.now();
if(now-lastShot<fireDelay)return;
lastShot=now;

let fovSize=document.getElementById("fov").value;
let target=null;

for(let e of enemies){
if(dist(player.x,player.y,e.x,e.y)<=fovSize){
target=e;
break;
}
}

if(!target)return;

let angle=Math.atan2(target.y-player.y,target.x-player.x);

bullets.push({
x:player.x,
y:player.y,
dx:Math.cos(angle)*12,
dy:Math.sin(angle)*12
});
}

document.getElementById("shoot").addEventListener("pointerdown",()=>firing=true);
document.getElementById("shoot").addEventListener("pointerup",()=>firing=false);

function update(){
ctx.clearRect(0,0,canvas.width,canvas.height);

if(!gameOver){

let fovSize=document.getElementById("fov").value;
ctx.beginPath();
ctx.arc(player.x,player.y,fovSize,0,Math.PI*2);
ctx.strokeStyle="red";
ctx.stroke();

if(document.getElementById("autoFire").checked){
shoot();
}else if(firing){
shoot();
}

for(let e of enemies){

ctx.fillStyle="black";
ctx.fillRect(e.x-15,e.y-30,30,60);
ctx.fillStyle="gray";
ctx.fillRect(e.x-10,e.y-45,20,20);

/* ESP */
if(document.getElementById("espBox").checked){
ctx.strokeStyle="cyan";
ctx.strokeRect(e.x-20,e.y-50,40,90);
}
if(document.getElementById("espLine").checked){
ctx.beginPath();
ctx.moveTo(player.x,player.y);
ctx.lineTo(e.x,e.y);
ctx.strokeStyle="yellow";
ctx.stroke();
}
if(document.getElementById("espHP").checked){
ctx.fillStyle="lime";
ctx.fillText("HP:"+e.hp,e.x-25,e.y-65);
}
if(document.getElementById("espDist").checked){
ctx.fillStyle="white";
ctx.fillText(Math.floor(dist(player.x,player.y,e.x,e.y)),e.x-25,e.y-80);
}
if(document.getElementById("espSkeleton").checked){
ctx.strokeStyle="white";
ctx.beginPath();
ctx.moveTo(e.x,e.y-45);
ctx.lineTo(e.x,e.y-10);
ctx.stroke();
}
}

for(let bi=bullets.length-1;bi>=0;bi--){
let b=bullets[bi];
b.x+=b.dx;
b.y+=b.dy;

ctx.fillStyle="orange";
ctx.fillRect(b.x-3,b.y-3,6,6);

for(let ei=enemies.length-1;ei>=0;ei--){
let e=enemies[ei];
let head=document.getElementById("aimbot").checked;

if(head && dist(b.x,b.y,e.x,e.y)<40){
e.hp-=100;
damages.push({x:e.x,y:e.y-60,text:"100",color:"red",time:Date.now()});
bullets.splice(bi,1);
break;
}
if(!head && dist(b.x,b.y,e.x,e.y)<30){
e.hp-=50;
damages.push({x:e.x,y:e.y-30,text:"50",color:"white",time:Date.now()});
bullets.splice(bi,1);
break;
}

if(e.hp<=0){
enemies.splice(ei,1);
kills++;
document.getElementById("kills").innerText=kills;
spawn();

if(kills>=50){
gameOver=true;
document.getElementById("winScreen").style.display="flex";
}
}
}
}

for(let di=damages.length-1;di>=0;di--){
let d=damages[di];
ctx.fillStyle=d.color;
ctx.font="20px Arial";
ctx.fillText(d.text,d.x,d.y);
if(Date.now()-d.time>200){
damages.splice(di,1);
}
}

}

requestAnimationFrame(update);
}
update();

/* PAINEL DRAG */
const panel=document.getElementById("panel");
const header=document.getElementById("header");
const content=document.getElementById("content");

let isDragging=false,offsetX=0,offsetY=0;

header.addEventListener("pointerdown",(e)=>{
isDragging=true;
offsetX=e.clientX-panel.offsetLeft;
offsetY=e.clientY-panel.offsetTop;
});

document.addEventListener("pointermove",(e)=>{
if(!isDragging)return;
panel.style.left=(e.clientX-offsetX)+"px";
panel.style.top=(e.clientY-offsetY)+"px";
});

document.addEventListener("pointerup",()=>isDragging=false);

document.getElementById("close").onclick=()=>panel.style.display="none";
document.getElementById("toggle").onclick=()=>{
content.style.display=content.style.display==="none"?"block":"none";
};
document.getElementById("restartBtn").onclick=resetGame;
</script>

</body>
</html>
