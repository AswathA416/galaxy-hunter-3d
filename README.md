
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>Galaxy Skeleton</title>
<style>
*{box-sizing:border-box}
body{margin:0;overflow:hidden;background:#080019;color:white;font-family:Arial}
canvas{display:block}
button,select,input{font:inherit}
button{cursor:pointer;border:1px solid #a36bff;background:#32136b;color:white;padding:11px 17px;border-radius:8px}
button:hover{background:#7435cc}
#hud{position:fixed;top:12px;left:12px;background:#10072bdc;padding:13px;border:1px solid #914bff;border-radius:10px;line-height:1.8;z-index:3}
#menuButtons{position:fixed;right:12px;top:12px;display:flex;gap:8px;z-index:4}
#crosshair{position:fixed;left:50%;top:50%;transform:translate(-50%,-50%);pointer-events:none;z-index:3;font-size:24px;color:white}
#message{position:fixed;bottom:20px;left:50%;transform:translateX(-50%);background:#16072cdd;padding:12px 20px;border-radius:9px;z-index:3;text-align:center}
#overlay{position:fixed;inset:0;background:#080019ed;z-index:10;display:none;align-items:center;justify-content:center}
.panel{background:#170b35;border:1px solid #a36bff;border-radius:15px;padding:25px;width:min(480px,92vw);max-height:85vh;overflow:auto}
.panel h2{color:#c5a1ff;margin-top:0}
.panel label{display:block;margin:15px 0}
.panel input[type=range]{width:100%}
.panel select{background:#29144c;color:white;padding:7px;border-radius:5px}
#start{position:fixed;inset:0;background:#080019ed;z-index:20;display:flex;align-items:center;justify-content:center;text-align:center}
#start h1{font-size:clamp(35px,7vw,65px);color:#c69aff;text-shadow:0 0 20px #903cff}
#start button{font-size:20px;padding:15px 35px}
#startContent{max-width:600px;padding:20px}
.small{color:#cbb8e9;font-size:13px}
</style>
</head>
<body>

<div id="hud">
🌌 GALAXY SKELETON<br>
❤️ Health: <span id="health">100</span><br>
⭐ Score: <span id="score">0</span><br>
💰 Coins: <span id="coins">0</span><br>
🔫 Weapon: <span id="weapon">Blaster</span>
</div>

<div id="menuButtons">
<button onclick="openShop()">SHOP</button>
<button onclick="openSettings()">SETTINGS</button>
</div>

<div id="crosshair">+</div>
<div id="message">Welcome to Galaxy City!</div>

<div id="overlay">
<div class="panel" id="panel"></div>
</div>

<div id="start">
<div id="startContent">
<h1>🌌 GALAXY SKELETON</h1>
<h2>THE COSMIC BATTLE</h2>
<p>Explore the lobby. Buy weapons. Enter the portal. Survive.</p>
<button id="enter">ENTER LOBBY</button>
<p class="small">WASD Move | Mouse Look | E Interact | Click Shoot | Space Jump</p>
</div>
</div>

<script type="module">
import * as THREE from 'https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.module.js';

const scene=new THREE.Scene();
scene.background=new THREE.Color(0x080019);
scene.fog=new THREE.Fog(0x080019,35,100);

const camera=new THREE.PerspectiveCamera(80,innerWidth/innerHeight,.1,200);
const renderer=new THREE.WebGLRenderer({antialias:true});
renderer.setSize(innerWidth,innerHeight);
renderer.setPixelRatio(Math.min(devicePixelRatio,2));
document.body.appendChild(renderer.domElement);

scene.add(new THREE.HemisphereLight(0x9999ff,0x18002c,2));

const sun=new THREE.DirectionalLight(0xffffff,2);
sun.position.set(10,20,10);
scene.add(sun);

const purple=new THREE.PointLight(0x963cff,150,40);
purple.position.set(0,6,0);
scene.add(purple);

const mat=(color,emissive=0)=>new THREE.MeshStandardMaterial({
color,emissive,emissiveIntensity:.8
});

const bone=mat(0xdedbff);
const galaxy=mat(0x6033cc,0x321080);
const dark=mat(0x100a27);
const glow=new THREE.MeshBasicMaterial({color:0xc35aff});
const enemyMat=mat(0x6e145e,0x40004f);

function box(x,y,z,sx,sy,sz,material,parent=scene){
const m=new THREE.Mesh(new THREE.BoxGeometry(sx,sy,sz),material);
m.position.set(x,y,z);
parent.add(m);
return m;
}

function ball(x,y,z,sx,sy,sz,material,parent=scene){
const m=new THREE.Mesh(new THREE.SphereGeometry(1,16,12),material);
m.position.set(x,y,z);
m.scale.set(sx,sy,sz);
parent.add(m);
return m;
}

function line(a,b,r,material,parent){
const start=new THREE.Vector3(...a);
const end=new THREE.Vector3(...b);
const d=end.clone().sub(start);
const m=new THREE.Mesh(
new THREE.CylinderGeometry(r,r,d.length(),10),material
);
m.position.copy(start).add(end).multiplyScalar(.5);
m.quaternion.setFromUnitVectors(new THREE.Vector3(0,1,0),d.normalize());
parent.add(m);
return m;
}

// STARS
const positions=[];
for(let i=0;i<1100;i++){
positions.push(
(Math.random()-.5)*120,
Math.random()*40+3,
(Math.random()-.5)*120
);
}
const sg=new THREE.BufferGeometry();
sg.setAttribute('position',new THREE.Float32BufferAttribute(positions,3));
scene.add(new THREE.Points(sg,new THREE.PointsMaterial({
color:0xc5a4ff,size:.15
})));

// GROUND
box(0,-.2,0,120,.3,120,mat(0x10072b));
const grid=new THREE.GridHelper(120,60,0x793aff,0x29124f);
grid.position.y=0;
scene.add(grid);

// CHARACTER
function makeCharacter(isEnemy=false){
const g=new THREE.Group();
const bodyMat=isEnemy?enemyMat:galaxy;

ball(0,2.05,0,.53,.63,.5,dark,g);
ball(0,2.12,-.3,.39,.42,.34,bone,g);

ball(-.16,2.17,-.58,.12,.14,.06,dark,g);
ball(.16,2.17,-.58,.12,.14,.06,dark,g);
ball(-.16,2.17,-.64,.055,.07,.03,glow,g);
ball(.16,2.17,-.64,.055,.07,.03,glow,g);

line([0,1.3,0],[0,1.85,0],.25,bodyMat,g);

for(let i=0;i<4;i++){
const rib=new THREE.Mesh(
new THREE.TorusGeometry(.23,.035,8,18,Math.PI),bone
);
rib.position.set(0,1.43+i*.12,-.05);
rib.rotation.z=Math.PI;
g.add(rib);
}

line([-.23,1.7,0],[-.48,1.25,0],.09,bone,g);
line([-.48,1.25,0],[-.45,.95,0],.065,bodyMat,g);
line([.23,1.7,0],[.48,1.25,0],.09,bone,g);
line([.48,1.25,0],[.45,.95,0],.065,bodyMat,g);

line([-.15,1.2,0],[-.18,.55,0],.11,bone,g);
line([.15,1.2,0],[.18,.55,0],.11,bone,g);
line([-.18,.55,0],[-.18,.12,0],.08,bodyMat,g);
line([.18,.55,0],[.18,.12,0],.08,bodyMat,g);

ball(-.18,.1,-.08,.15,.1,.22,dark,g);
ball(.18,.1,-.08,.15,.1,.22,dark,g);

const hood=new THREE.Mesh(
new THREE.ConeGeometry(.57,.7,16,1,true),dark
);
hood.position.set(0,2.38,.1);
g.add(hood);

return g;
}

const player=makeCharacter();
scene.add(player);
player.visible=false;

// WEAPON
const gun=new THREE.Group();
player.add(gun);
gun.position.set(.5,1.05,-.2);
box(0,0,-.25,.22,.2,.55,galaxy,gun);
box(0,.1,-.5,.09,.08,.5,dark,gun);
box(0,.13,-.4,.04,.04,.3,glow,gun);

// BUILDINGS
function buildBuilding(x,z,w,h,d,color){
box(x,h/2,z,w,h,d,mat(color,0x1c0750));
box(x,h+.1,z,w+.2,.2,d+.2,glow);
}

buildBuilding(-13,-8,7,5,6,0x291052);
buildBuilding(13,-8,7,5,6,0x291052);
buildBuilding(-13,9,7,5,6,0x291052);
buildBuilding(13,9,7,5,6,0x291052);

// PORTAL
const portal=new THREE.Group();
portal.position.set(0,0,-13);
scene.add(portal);

const ring=new THREE.Mesh(
new THREE.TorusGeometry(2,0.22,16,64),glow
);
ring.position.y=2.2;
portal.add(ring);

const portalCore=new THREE.Mesh(
new THREE.CircleGeometry(1.8,48),
new THREE.MeshBasicMaterial({
color:0x5520bb,transparent:true,opacity:.6,
side:THREE.DoubleSide
})
);
portalCore.position.y=2.2;
portal.add(portalCore);

const portalLight=new THREE.PointLight(0x9c35ff,80,15);
portalLight.position.y=2;
portal.add(portalLight);

// SHOPKEEPER
const shopkeeper=makeCharacter();
shopkeeper.position.set(9,0,3);
shopkeeper.rotation.y=-Math.PI/2;
scene.add(shopkeeper);

ball(9,2.25,3,.7,.8,.65,glow);
box(9,1.4,5,3,1,1,galaxy);
box(9,2,5,2.6,.15,.8,glow);

// LEADERBOARD
box(-9,2,4,5,4,.3,dark);
box(-9,4,3.8,4.5,.2,.15,glow);

// ENEMIES
const enemies=[];

function spawnEnemies(){
for(let i=0;i<5;i++){
const e=makeCharacter(true);
e.position.set(
(Math.random()-.5)*24,
0,
-22-Math.random()*15
);
e.userData.hp=3;
scene.add(e);
enemies.push(e);
}
}

// GAME STATE
let mode='menu';
let health=100;
let score=0;
let coins=0;
let yaw=0;
let pitch=.1;
let vertical=0;
let grounded=true;
let sensitivity=.002;
let invertY=false;
let fov=80;
let crosshairStyle='+';
let showHUD=true;
let graphics='High';
let volume=70;
let selectedWeapon='Blaster';
let weapons=['Blaster'];
let lastDamage=0;
let gameOver=false;
const keys={};
const clock=new THREE.Clock();
const ray=new THREE.Raycaster();

const hud=document.getElementById('hud');
const overlay=document.getElementById('overlay');
const cross=document.getElementById('crosshair');
const message=document.getElementById('message');

function updateHUD(){
document.getElementById('health').textContent=health;
document.getElementById('score').textContent=score;
document.getElementById('coins').textContent=coins;
document.getElementById('weapon').textContent=selectedWeapon;
hud.style.display=showHUD?'block':'none';
}

function say(text){
message.textContent=text;
}

function startLobby(){
mode='lobby';
health=100;
score=0;
player.visible=true;
player.position.set(0,0,5);
player.rotation.y=0;
updateHUD();
document.getElementById('start').style.display='none';
say('Welcome! Find the shopkeeper or enter the portal.');
}

document.getElementById('enter').onclick=startLobby;

// SHOP
function openShop(){
if(mode==='menu')return;

overlay.style.display='flex';
document.exitPointerLock();

document.getElementById('panel').innerHTML=`
<h2>🌌 GALAXY WEAPON SHOP</h2>
<p>Your coins: <b>${coins}</b></p>
<div id="shopItems"></div>
<button onclick="closePanel()">CLOSE SHOP</button>`;

const items=[
['Blaster',0,'Starter weapon'],
['Cosmic Rifle',50,'Faster firing'],
['Void Cannon',100,'Heavy weapon']
];

const list=document.getElementById('shopItems');

items.forEach(([name,price,desc])=>{
const owned=weapons.includes(name);
const b=document.createElement('button');
b.textContent=name+' — '+(owned?'OWNED':price+' coins')+' | '+desc;

b.onclick=()=>{
if(owned){
selectedWeapon=name;
updateHUD();
openShop();
return;
}

if(coins>=price){
coins-=price;
weapons.push(name);
selectedWeapon=name;
updateHUD();
openShop();
}else{
alert('Not enough coins!');
}
};

list.appendChild(b);
});
}
window.openShop=openShop;

// SETTINGS
function openSettings(){
overlay.style.display='flex';
document.exitPointerLock();

document.getElementById('panel').innerHTML=`
<h2>⚙️ SETTINGS</h2>

<label>Mouse sensitivity: <span id="sensVal">${sensitivity.toFixed(3)}</span>
<input id="sens" type="range" min=".0005" max=".008" step=".0005" value="${sensitivity}">
</label>

<label>Field of view: <span id="fovVal">${fov}</span>
<input id="fovInput" type="range" min="60" max="110" value="${fov}">
</label>

<label>Crosshair
<select id="crossSelect">
<option>+</option><option>×</option><option>●</option><option>⊙</option><option>✛</option>
</select>
</label>

<label><input type="checkbox" id="invert" ${invertY?'checked':''}> Invert mouse Y</label>
<label><input type="checkbox" id="hudCheck" ${showHUD?'checked':''}> Show HUD</label>

<label>Graphics
<select id="graphics">
<option>Low</option><option>Medium</option><option>High</option>
</select>
</label>

<label>Sound volume
<input id="volume" type="range" min="0" max="100" value="${volume}">
</label>

<button onclick="saveSettings()">SAVE SETTINGS</button>
<button onclick="closePanel()">CLOSE</button>`;

document.getElementById('sens').oninput=e=>{
document.getElementById('sensVal').textContent=Number(e.target.value).toFixed(3);
};

document.getElementById('fovInput').oninput=e=>{
document.getElementById('fovVal').textContent=e.target.value;
};

document.getElementById('crossSelect').value=crosshairStyle;
document.getElementById('graphics').value=graphics;
}
window.openSettings=openSettings;

window.saveSettings=()=>{
sensitivity=Number(document.getElementById('sens').value);
fov=Number(document.getElementById('fovInput').value);
crosshairStyle=document.getElementById('crossSelect').value;
invertY=document.getElementById('invert').checked;
showHUD=document.getElementById('hudCheck').checked;
graphics=document.getElementById('graphics').value;
volume=Number(document.getElementById('volume').value);

camera.fov=fov;
camera.updateProjectionMatrix();
cross.textContent=crosshairStyle;

renderer.setPixelRatio(Math.min(devicePixelRatio,
graphics==='Low'?1:graphics==='Medium'?1.5:2));

updateHUD();
closePanel();
};

window.closePanel=()=>{
overlay.style.display='none';
if(mode!=='menu'){
renderer.domElement.requestPointerLock();
}
};

// LEADERBOARD
function leaderboard(){
overlay.style.display='flex';
document.exitPointerLock();

document.getElementById('panel').innerHTML=`
<h2>🏆 GALAXY LEADERBOARD</h2>
<p>1. CosmicKing — 2500</p>
<p>2. ShadowSkull — 1800</p>
<p>3. StarHunter — 1200</p>
<p>4. GalaxyWarrior — 900</p>
<p>5. YOU — ${score}</p>
<p style="font-size:12px;color:#cbb8e9">Demo leaderboard. Online rankings require a server.</p>
<button onclick="closePanel()">CLOSE</button>`;
}
window.leaderboard=leaderboard;

// ENTER BATTLE
function enterPortal(){
mode='battle';
gameOver=false;
player.position.set(0,0,5);
spawnEnemies();
say('BATTLE STARTED! Defeat all enemy skeletons!');
}

// KEYBOARD
document.addEventListener('keydown',e=>{
keys[e.code]=true;

if(e.code==='KeyE'&&mode==='lobby'){
if(player.position.distanceTo(shopkeeper.position)<4){
openShop();
}else if(player.position.distanceTo(portal.position)<4){
enterPortal();
}else if(player.position.distanceTo(new THREE.Vector3(-9,0,4))<5){
leaderboard();
}
}
});

document.addEventListener('keyup',e=>{
keys[e.code]=false;
});

// MOUSE
document.addEventListener('mousemove',e=>{
if(document.pointerLockElement===renderer.domElement){
yaw-=e.movementX*sensitivity;
pitch+=(invertY?1:-1)*e.movementY*sensitivity;
pitch=THREE.MathUtils.clamp(pitch,-.5,.8);
}
});

renderer.domElement.addEventListener('click',()=>{
if(mode!=='menu'&&overlay.style.display!=='flex'){
renderer.domElement.requestPointerLock();
}
});

// SHOOTING
function shoot(){
if(mode!=='battle'||gameOver)return;

ray.setFromCamera(new THREE.Vector2(0,0),camera);

const targets=[];
enemies.forEach(e=>{
e.traverse(o=>{
if(o.isMesh)targets.push(o);
});
});

const hits=ray.intersectObjects(targets,false);
if(!hits.length)return;

let target=hits[0].object;

while(target&&!enemies.includes(target)){
target=target.parent;
}

if(!target)return;

target.userData.hp--;

if(target.userData.hp<=0){
scene.remove(target);
enemies.splice(enemies.indexOf(target),1);
score+=10;
coins+=10;
updateHUD();
say('Enemy eliminated! +10 coins');

if(enemies.length===0)finishGame();
}
}

document.addEventListener('mousedown',e=>{
if(e.button===0&&mode==='battle'&&overlay.style.display!=='flex'){
shoot();
}
});

// END GAME
function finishGame(){
mode='lobby';
enemies.forEach(e=>scene.remove(e));
enemies.length=0;
player.position.set(0,0,5);
health=100;
updateHUD();
say('VICTORY! You earned coins. Return to the lobby!');
}

// CAMERA
function updateCamera(){
const distance=6;

const target=new THREE.Vector3(
player.position.x+Math.sin(yaw)*distance,
player.position.y+3+pitch*2,
player.position.z+Math.cos(yaw)*distance
);

camera.position.lerp(target,.16);
camera.lookAt(
player.position.x,
player.position.y+1.5,
player.position.z
);
}

// MAIN GAME LOOP
function animate(){
requestAnimationFrame(animate);

const dt=Math.min(clock.getDelta(),.05);

portal.rotation.y+=dt*.7;
ring.rotation.z+=dt*.4;

if(mode==='lobby'||mode==='battle'){

let forward=0,side=0;

if(keys.KeyW)forward++;
if(keys.KeyS)forward--;
if(keys.KeyA)side--;
if(keys.KeyD)side++;

const length=Math.hypot(forward,side);

if(length){
forward/=length;
side/=length;

const speed=8*dt;

player.position.x+=(
-Math.sin(yaw)*forward+Math.cos(yaw)*side
)*speed;

player.position.z+=(
-Math.cos(yaw)*forward-Math.sin(yaw)*side
)*speed;

player.rotation.y=yaw;
}

// JUMP
if(keys.Space&&grounded){
vertical=8;
grounded=false;
}

player.position.y+=vertical*dt;
vertical-=20*dt;

if(player.position.y<=0){
player.position.y=0;
vertical=0;
grounded=true;
}

// ARENA BOUNDARIES
player.position.x=THREE.MathUtils.clamp(player.position.x,-27,27);
player.position.z=THREE.MathUtils.clamp(player.position.z,-27,27);

// LOBBY INTERACTIONS
if(mode==='lobby'){
if(player.position.distanceTo(portal.position)<3.5){
say('Press E to enter the battle portal');
}else if(player.position.distanceTo(shopkeeper.position)<4){
say('Press E to visit the weapon shop');
}else if(player.position.distanceTo(new THREE.Vector3(-9,0,4))<5){
say('Press E to view the leaderboard');
}else{
say('Explore the lobby. Find the portal!');
}

// AUTOMATIC PORTAL ENTRY
if(player.position.distanceTo(portal.position)<2.8){
enterPortal();
}
}

// ENEMY AI
if(mode==='battle'){
enemies.forEach(e=>{
const dx=player.position.x-e.position.x;
const dz=player.position.z-e.position.z;
const d=Math.hypot(dx,dz);

if(d>1.5){
e.position.x+=dx/d*2.1*dt;
e.position.z+=dz/d*2.1*dt;
}else if(Date.now()-lastDamage>1000){
health-=10;
lastDamage=Date.now();
updateHUD();
}
});

if(health<=0){
mode='lobby';
health=100;

enemies.forEach(e=>scene.remove(e));
enemies.length=0;

player.position.set(0,0,5);
updateHUD();
say('You were eliminated! Back in the lobby.');
}
}

updateCamera();
}

renderer.render(scene,camera);
}

animate();

// RESIZE
window.addEventListener('resize',()=>{
camera.aspect=innerWidth/innerHeight;
camera.updateProjectionMatrix();
renderer.setSize(innerWidth,innerHeight);
});

</script>
</body>
</html>
