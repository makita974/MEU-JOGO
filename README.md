<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Mini FPS Completo</title>
<style>
body{margin:0;overflow:hidden;background:#111;font-family:Arial;color:white}
canvas{display:block}

#hud{
position:fixed;
top:10px;
left:10px;
z-index:5;
font-size:18px;
}

#shoot{
position:fixed;
bottom:80px;
right:40px;
width:80px;
height:80px;
background:red;
border-radius:50%;
display:flex;
align-items:center;
justify-content:center;
font-weight:bold;
z-index:5;
}

#winScreen{
position:fixed;
top:0;
left:0;
width:100%;
height:100%;
background:rgba(0,0,0,0.9);
display:none;
flex-direction:column;
align-items:center;
justify-content:center;
z-index:9999;
}

#winScreen button{
padding:15px 30px;
font-size:18px;
margin-top:20px;
}

#panel{
position:fixed;
top:60px;
left:20px;
width:300px;
background:#1c1c2e;
border-radius:12px;
box-shadow:0 0 15px black;
z-index:999;
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

.tab{
margin-top:8px;
background:#24243a;
padding:8px;
border-radius:8px;
}

.close{color:red;cursor:pointer}
.arrow{cursor:pointer}
</style>
</head>
<body>

<div id="hud">Kills: <span id="kills">0</span></div>
<canvas id="game"></canvas>
<div id="shoot">FIRE</div>

<div id="winScreen">
<h1>VOCÊ GANHOU!</h1>
<button onclick="location.reload()">Reiniciar</button>
</div>

<div id="panel">
<div id="header">
<span id="toggle" class="arrow">⬇</span>
<span>Painel</span>
<span id="close" class="close">✖</span>
</div>

<div id="content">

<div class="tab">
<b>🎯 Mira</b><br>
<label><input type="checkbox" id="aimbot"> Mira Automática</label><br>
FOV: <input type="range" id="fov" min="50" max="600" value="200">
</div>

<div class="tab">
<b>👁 ESP</b><br>
<label><input type="checkbox" id="espBox"> Caixa</label><br>
<label><input type="checkbox" id="espLine"> Linha</label><br>
<label><input type="checkbox" id="espHP"> Vida</label><br>
<label><input type="checkbox" id="espDist"> Distância</label>
</div>

</div>
</div>

<script>
const canvas=document.getElementById("game");
const ctx=canvas.getContext("2d");
canvas.width=innerWidth;
canvas.height=innerHeight;

let player={x:canvas.width/2,y:canvas.height/2};
let enemies=[];
let bullets=[];
let kills=0;

function spawn(){
enemies.push({
x:Math.random()*canvas.width,
y:Math.random()*canvas.height,
hp:100
});
}
for(let i=0;i<5;i++)spawn();

function dist(x1,y1,x2,y2){
return Math.hypot(x2-x1,y2-y1);
}

function shoot(){
let target=null;
let fovSize=document.getElementById("fov").value;

if(document.getElementById("aimbot").checked){
for(let e of enemies){
if(dist(player.x,player.y,e.x,e.y)<=fovSize){
target=e;
break;
}
}
}

if(target){
let angle=Math.atan2(target.y-player.y,target.x-player.x);
bullets.push({
x:player.x,
y:player.y,
dx:Math.cos(angle)*8,
dy:Math.sin(angle)*8
});
}else{
bullets.push({x:player.x,y:player.y,dx:0,dy:-8});
}
}

document.getElementById("shoot").onclick=shoot;

function update(){
ctx.clearRect(0,0,canvas.width,canvas.height);

let fovSize=document.getElementById("fov").value;

ctx.beginPath();
ctx.arc(player.x,player.y,fovSize,0,Math.PI*2);
ctx.strokeStyle="red";
ctx.stroke();

for(let ei=enemies.length-1;ei>=0;ei--){
let e=enemies[ei];

ctx.fillStyle="black";
ctx.fillRect(e.x-10,e.y-20,20,40);

if(document.getElementById("espBox").checked){
ctx.strokeStyle="cyan";
ctx.strokeRect(e.x-15,e.y-25,30,50);
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
ctx.fillText("HP:"+e.hp,e.x-15,e.y-30);
}

if(document.getElementById("espDist").checked){
ctx.fillStyle="white";
ctx.fillText(Math.floor(dist(player.x,player.y,e.x,e.y)),e.x-15,e.y-45);
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
if(dist(b.x,b.y,e.x,e.y)<20){
e.hp-=50;
bullets.splice(bi,1);
if(e.hp<=0){
enemies.splice(ei,1);
kills++;
document.getElementById("kills").innerText=kills;
spawn();
}
break;
}
}
}

if(kills>=50){
document.getElementById("winScreen").style.display="flex";
}

requestAnimationFrame(update);
}
update();

/* Painel arrastável */
const panel=document.getElementById("panel");
const header=document.getElementById("header");
let offsetX=0,offsetY=0,dragging=false;

header.addEventListener("mousedown",startDrag);
header.addEventListener("touchstart",startDrag);

function startDrag(e){
dragging=true;
let rect=panel.getBoundingClientRect();
let x=e.touches?e.touches[0].clientX:e.clientX;
let y=e.touches?e.touches[0].clientY:e.clientY;
offsetX=x-rect.left;
offsetY=y-rect.top;
}

document.addEventListener("mousemove",drag);
document.addEventListener("touchmove",drag);

function drag(e){
if(!dragging)return;
let x=e.touches?e.touches[0].clientX:e.clientX;
let y=e.touches?e.touches[0].clientY:e.clientY;
panel.style.left=(x-offsetX)+"px";
panel.style.top=(y-offsetY)+"px";
}

document.addEventListener("mouseup",()=>dragging=false);
document.addEventListener("touchend",()=>dragging=false);

document.getElementById("close").onclick=()=>panel.style.display="none";
document.getElementById("toggle").onclick=function(){
content.style.display=content.style.display==="none"?"block":"none";
this.innerText=content.style.display==="none"?"⬆":"⬇";
};
</script>
</body>
</html>
