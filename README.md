
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Galaxy Skeleton</title>
<style>
*{box-sizing:border-box}
body{margin:0;overflow:hidden;font-family:Arial;background:#10152b;color:white}
canvas{display:block}
#hud{position:fixed;inset:0;pointer-events:none}
#top{position:absolute;top:14px;left:16px;font-size:18px;font-weight:bold;text-shadow:2px 2px 4px #000}
#stats{position:absolute;top:14px;right:16px;text-align:right;line-height:1.7;text-shadow:2px 2px 4px #000}
#crosshair{position:fixed;left:50%;top:50%;transform:translate(-50%,-50%);font-size:28px;line-height:1;color:white;text-shadow:0 0 5px black,0 0 10px cyan;pointer-events:none;z-index:4}
#message{position:absolute;bottom:24px;left:50%;transform:translateX(-50%);padding:10px 16px;background:#111a;border-radius:10px;text-align:center;font-size:14px}
#overlay{position:fixed;inset:0;display:flex;justify-content:center;align-items:center;background:linear-gradient(135deg,#182c55ee,#25134fee);z-index:5}
#panel{width:min(440px,90vw);padding:30px;border-radius:20px;background:#18233bee;border:2px solid #83caff;text-align:center;box-shadow:0 0 35px #53baff55}
h1{margin:0 0 10px;font-size:38px;color:#8ce5ff}
button{border:0;border-radius:10px;padding:13px 24px;margin:8px;font-size:16px;font-weight:bold;cursor:pointer;background:#61d5ff;color:#10203b}
button:hover{background:#a1eaff}
#instructions{line-height:1.7;color:#e2edff;font-size:14px}
</style>
</head>
<body>

<div id="overlay">
<div id="panel">
<h1>GALAXY SKELETON</h1>
<p>Explore the galaxy city and defeat the enemy skeletons!</p>
<button id="playButton">PLAY GAME</button>
<div id="instructions">
<b>W A S D</b> – Move<br>
<b>Mouse</b> – Move crosshair and aim<br>
<b>Left click</b> – Shoot<br>
<b>Arrow keys</b> – Turn camera<br>
<b>Shift</b> – Sprint<br>
<b>Space</b> – Jump
</div>
</div>
</div>

<div id="hud">
<div id="top">🌌 GALAXY SKELETON</div>
<div id="stats">
Health: <span id="health">100</span><br>
Eliminations: <span id="kills">0</span><br>
Ammo: ∞<br>
Boss HP: <span id="bossHealth">100</span>%
</div>
<div id="crosshair">+</div>
<div id="message">Click PLAY GAME to begin</div>
</div>

<script type="module">
import * as THREE from 'https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.module.js';

const scene=new THREE.Scene();
scene.background=new THREE.Color(0x8bc9f5);
scene.fog=new THREE.Fog(0x8bc9f5,65,180);

const camera=new THREE.PerspectiveCamera(75,innerWidth/innerHeight,0.1,300);
camera.position.set(0,2,12);

const renderer=new THREE.WebGLRenderer({antialias:true});
renderer.setSize(innerWidth,innerHeight);
renderer.setPixelRatio(Math.min(devicePixelRatio,2));
renderer.shadowMap.enabled=true;
renderer.shadowMap.type=THREE.PCFSoftShadowMap;
document.body.appendChild(renderer.domElement);

scene.add(new THREE.HemisphereLight(0xffffff,0x607080,2));

const sunlight=new THREE.DirectionalLight(0xffffff,2);
sunlight.position.set(-30,50,30);
sunlight.castShadow=true;
scene.add(sunlight);

// Ground
const ground=new THREE.Mesh(
new THREE.PlaneGeometry(240,240),
new THREE.MeshStandardMaterial({color:0x7fc58b})
);
ground.rotation.x=-Math.PI/2;
ground.receiveShadow=true;
scene.add(ground);

// Paths
function makePath(x,z,width,depth){
const path=new THREE.Mesh(
new THREE.BoxGeometry(width,0.08,depth),
new THREE.MeshStandardMaterial({color:0xd9d5c9})
);
path.position.set(x,0.04,z);
scene.add(path);
}
makePath(0,0,14,180);
makePath(0,0,180,14);

// Buildings
function createBuilding(x,z,color,width=10,height=8,depth=9){
const group=new THREE.Group();

const wall=new THREE.Mesh(
new THREE.BoxGeometry(width,height,depth),
new THREE.MeshStandardMaterial({color,roughness:0.85})
);
wall.position.y=height/2;
wall.castShadow=true;
wall.receiveShadow=true;
group.add(wall);

const roof=new THREE.Mesh(
new THREE.ConeGeometry(width*0.72,3.5,4),
new THREE.MeshStandardMaterial({color:0x6b83bd})
);
roof.position.y=height+1.7;
roof.rotation.y=Math.PI/4;
group.add(roof);

const door=new THREE.Mesh(
new THREE.BoxGeometry(1.7,3,0.18),
new THREE.MeshStandardMaterial({color:0x79513d})
);
door.position.set(0,1.5,depth/2+0.1);
group.add(door);

for(const wx of [-width*0.28,width*0.28]){
const windowMesh=new THREE.Mesh(
new THREE.BoxGeometry(1.7,1.7,0.15),
new THREE.MeshStandardMaterial({
color:0x8de5ff,
emissive:0x1c536a
})
);
windowMesh.position.set(wx,height*0.65,depth/2+0.1);
group.add(windowMesh);
}
group.position.set(x,0,z);
scene.add(group);
}

createBuilding(-23,-24,0xffc9a8,11,9,10);
createBuilding(23,-24,0xb9dcff,11,10,10);
createBuilding(-25,25,0xd4c1ff,12,8,11);
createBuilding(25,25,0xb9efd2,12,9,11);
createBuilding(-48,-10,0xffe6a6);
createBuilding(48,10,0xffc5d7);
createBuilding(-48,42,0xb9dcff);
createBuilding(48,-42,0xd4c1ff);

// Trees
function createTree(x,z){
const trunk=new THREE.Mesh(
new THREE.CylinderGeometry(0.3,0.4,2.5,8),
new THREE.MeshStandardMaterial({color:0x805637})
);
trunk.position.set(x,1.25,z);
scene.add(trunk);

const leaves=new THREE.Mesh(
new THREE.SphereGeometry(1.5,12,10),
new THREE.MeshStandardMaterial({color:0x36a86b})
);
leaves.position.set(x,3.1,z);
scene.add(leaves);
}

for(let i=0;i<25;i++){
const x=(Math.random()-0.5)*150;
const z=(Math.random()-0.5)*150;
if(Math.abs(x)>12&&Math.abs(z)>12)createTree(x,z);
}

// Stars
const starGeometry=new THREE.BufferGeometry();
const starPositions=[];
for(let i=0;i<500;i++){
starPositions.push(
(Math.random()-0.5)*220,
20+Math.random()*80,
(Math.random()-0.5)*220
);
}
starGeometry.setAttribute('position',
new THREE.Float32BufferAttribute(starPositions,3));
scene.add(new THREE.Points(starGeometry,
new THREE.PointsMaterial({color:0xffffff,size:0.5})));

// Player
const player={
position:new THREE.Vector3(0,1.7,12),
velocityY:0,
speed:9,
onGround:true,
yaw:0,
pitch:0,
health:100,
kills:0
};
camera.position.copy(player.position);

const keys={};
let gameStarted=false;
let canShoot=true;

const mouseNDC=new THREE.Vector2(0,0);
const crosshair=document.getElementById('crosshair');
const overlay=document.getElementById('overlay');
const message=document.getElementById('message');

document.getElementById('playButton').addEventListener('click',()=>{
gameStarted=true;
overlay.style.display='none';
message.textContent='Move your crosshair over an enemy and click to shoot!';
crosshair.style.left=innerWidth/2+'px';
crosshair.style.top=innerHeight/2+'px';
mouseNDC.set(0,0);
});

// FREE MOUSE AIM: no pointer lock
document.addEventListener('mousemove',event=>{
if(!gameStarted)return;

const x=THREE.MathUtils.clamp(event.clientX,0,innerWidth);
const y=THREE.MathUtils.clamp(event.clientY,0,innerHeight);

crosshair.style.left=x+'px';
crosshair.style.top=y+'px';

mouseNDC.x=(x/innerWidth)*2-1;
mouseNDC.y=-(y/innerHeight)*2+1;
});

document.addEventListener('keydown',event=>{
keys[event.code]=true;

if(event.code==='Space'&&player.onGround&&gameStarted){
player.velocityY=8;
player.onGround=false;
}
});

document.addEventListener('keyup',event=>{
keys[event.code]=false;
});

// Camera turning
document.addEventListener('keydown',event=>{
if(!gameStarted)return;
const turnSpeed=0.045;

if(event.code==='ArrowLeft')player.yaw+=turnSpeed;
if(event.code==='ArrowRight')player.yaw-=turnSpeed;
if(event.code==='ArrowUp')player.pitch=Math.max(-1.2,player.pitch-turnSpeed);
if(event.code==='ArrowDown')player.pitch=Math.min(1.2,player.pitch+turnSpeed);
});

// Enemies and boss
const enemies=[];
let bossEnemy=null;

function createEnemy(x,z,isBoss=false){
const enemy=new THREE.Group();

const enemyData={
mesh:enemy,
hp:isBoss?20:3,
maxHp:isBoss?20:3,
alive:true,
isBoss
};
enemy.userData.enemyData=enemyData;

const bone=new THREE.MeshStandardMaterial({color:0xf1ead9});
const dark=new THREE.MeshStandardMaterial({color:0x202333});
const red=new THREE.MeshStandardMaterial({
color:0xff3333,
emissive:0x550000
});

const body=new THREE.Mesh(
new THREE.CapsuleGeometry(0.45,0.8,4,8),dark
);
body.position.y=1.2;
enemy.add(body);

const skull=new THREE.Mesh(
new THREE.SphereGeometry(0.48,12,12),bone
);
skull.position.y=2.05;
enemy.add(skull);

for(const eyeX of [-0.17,0.17]){
const eye=new THREE.Mesh(
new THREE.SphereGeometry(0.09,8,8),red
);
eye.position.set(eyeX,2.1,0.4);
enemy.add(eye);
}

for(const side of [-1,1]){
const arm=new THREE.Mesh(
new THREE.CylinderGeometry(0.12,0.14,0.9,8),bone
);
arm.position.set(side*0.58,1.3,0);
arm.rotation.z=side*-0.25;
enemy.add(arm);

const leg=new THREE.Mesh(
new THREE.CylinderGeometry(0.13,0.16,0.8,8),bone
);
leg.position.set(side*0.22,0.4,0);
enemy.add(leg);
}

enemy.position.set(x,0,z);

if(isBoss){
enemy.scale.setScalar(2.6);
const crown=new THREE.Mesh(
new THREE.TorusGeometry(0.48,0.09,8,18),
new THREE.MeshStandardMaterial({
color:0xffc52e,
emissive:0x664000
})
);
crown.position.y=2.65;
enemy.add(crown);
}

enemy.traverse(part=>{
if(part.isMesh)part.userData.enemyData=enemyData;
});

scene.add(enemy);
enemies.push(enemyData);
if(isBoss)bossEnemy=enemyData;
}

createEnemy(-12,-18);
createEnemy(12,-20);
createEnemy(-18,18);
createEnemy(18,18);
createEnemy(0,-35,true);

// Shooting
const raycaster=new THREE.Raycaster();

function shoot(){
if(!gameStarted||!canShoot)return;
canShoot=false;
setTimeout(()=>canShoot=true,180);

raycaster.setFromCamera(mouseNDC,camera);
raycaster.far=180;

const targets=enemies.filter(e=>e.alive).map(e=>e.mesh);
const hits=raycaster.intersectObjects(targets,true);

if(!hits.length)return;

const enemyData=hits[0].object.userData.enemyData;
if(!enemyData||!enemyData.alive)return;

enemyData.hp=Math.max(
0,
enemyData.hp-(enemyData.isBoss?2:1)
);

const material=hits[0].object.material;
if(material&&material.emissive){
const previous=material.emissive.getHex();
material.emissive.setHex(0x552222);
setTimeout(()=>material.emissive.setHex(previous),100);
}

if(enemyData.isBoss){
document.getElementById('bossHealth').textContent=
Math.ceil(enemyData.hp/enemyData.maxHp*100);
}

if(enemyData.hp<=0){
enemyData.alive=false;
scene.remove(enemyData.mesh);
player.kills++;
document.getElementById('kills').textContent=player.kills;

if(enemyData.isBoss){
document.getElementById('bossHealth').textContent='DEFEATED';
message.textContent='BOSS ELIMINATED! YOU SAVED THE GALAXY!';
}else{
message.textContent='Enemy eliminated!';
}
}else{
message.textContent=enemyData.isBoss?
`BOSS HIT! ${Math.ceil(enemyData.hp/enemyData.maxHp*100)}% health remaining`:
'Hit! Keep shooting!';
}
}

renderer.domElement.addEventListener('mousedown',event=>{
if(event.button===0)shoot();
});

// Movement
const clock=new THREE.Clock();

function updatePlayer(delta){
if(!gameStarted)return;

const forward=new THREE.Vector3(
-Math.sin(player.yaw),0,-Math.cos(player.yaw)
);
const right=new THREE.Vector3(
Math.cos(player.yaw),0,-Math.sin(player.yaw)
);

const move=new THREE.Vector3();

if(keys.KeyW)move.add(forward);
if(keys.KeyS)move.sub(forward);
if(keys.KeyD)move.add(right);
if(keys.KeyA)move.sub(right);

if(move.lengthSq()>0)move.normalize();

const speed=keys.ShiftLeft||keys.ShiftRight?
player.speed*1.65:player.speed;

player.position.addScaledVector(move,speed*delta);

player.position.x=THREE.MathUtils.clamp(player.position.x,-85,85);
player.position.z=THREE.MathUtils.clamp(player.position.z,-85,85);

player.velocityY-=20*delta;
player.position.y+=player.velocityY*delta;

if(player.position.y<1.7){
player.position.y=1.7;
player.velocityY=0;
player.onGround=true;
}

camera.position.copy(player.position);
camera.rotation.order='YXZ';
camera.rotation.y=player.yaw;
camera.rotation.x=player.pitch;
}

// Enemy movement
function updateEnemies(delta){
if(!gameStarted)return;

for(const enemy of enemies){
if(!enemy.alive)continue;

const direction=new THREE.Vector3(
player.position.x-enemy.mesh.position.x,
0,
player.position.z-enemy.mesh.position.z
);

const distance=direction.length();

if(distance>2.2&&distance<35){
direction.normalize();
enemy.mesh.position.addScaledVector(direction,delta*2.2);
enemy.mesh.rotation.y=Math.atan2(direction.x,direction.z);
}
}
}

// Main loop
function animate(){
requestAnimationFrame(animate);

const delta=Math.min(clock.getDelta(),0.05);
updatePlayer(delta);
updateEnemies(delta);
renderer.render(scene,camera);
}
animate();

// Resize
window.addEventListener('resize',()=>{
camera.aspect=innerWidth/innerHeight;
camera.updateProjectionMatrix();
renderer.setSize(innerWidth,innerHeight);
});
</script>
</body>
</html>
