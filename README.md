
<style>
  * { box-sizing: border-box; }
  body {
    margin: 0;
    overflow: hidden;
    font-family: Arial, sans-serif;
    background: #10152b;
    color: white;
  }
  canvas { display: block; }
  #hud {
    position: fixed;
    inset: 0;
    pointer-events: none;
  }
  #top {
    position: absolute;
    top: 14px;
    left: 16px;
    font-size: 18px;
    font-weight: bold;
    text-shadow: 2px 2px 4px #000;
  }
  #stats {
    position: absolute;
    top: 14px;
    right: 16px;
    text-align: right;
    line-height: 1.7;
    text-shadow: 2px 2px 4px #000;
  }
  #crosshair {
    position: absolute;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);
    font-size: 28px;
    color: white;
    text-shadow: 0 0 5px black;
  }
  #message {
    position: absolute;
    bottom: 24px;
    left: 50%;
    transform: translateX(-50%);
    padding: 10px 16px;
    background: #111a;
    border-radius: 10px;
    text-align: center;
    font-size: 14px;
  }
  #overlay {
    position: fixed;
    inset: 0;
    display: flex;
    justify-content: center;
    align-items: center;
    background: linear-gradient(135deg, #182c55ee, #25134fee);
    z-index: 5;
  }
  #panel {
    width: min(440px, 90vw);
    padding: 30px;
    border-radius: 20px;
    background: #18233bee;
    border: 2px solid #83caff;
    text-align: center;
    box-shadow: 0 0 35px #53baff55;
  }
  h1 {
    margin: 0 0 10px;
    font-size: 38px;
    color: #8ce5ff;
  }
  button {
    border: 0;
    border-radius: 10px;
    padding: 13px 24px;
    margin: 8px;
    font-size: 16px;
    font-weight: bold;
    cursor: pointer;
    background: #61d5ff;
    color: #10203b;
  }
  button:hover { background: #a1eaff; }
  #instructions {
    line-height: 1.7;
    color: #e2edff;
    font-size: 14px;
  }
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
      <b>Mouse</b> – Look around<br>
      <b>Left click</b> – Shoot<br>
      <b>Shift</b> – Sprint<br>
      <b>Space</b> – Jump<br>
      <b>Esc</b> – Unlock cursor
    </div>
  </div>
</div>

<div id="hud">
  <div id="top">🌌 GALAXY SKELETON</div>
  <div id="stats">
    Health: <span id="health">100</span><br>
    Eliminations: <span id="kills">0</span><br>
    Ammo: <span id="ammo">∞</span>
  </div>
  <div id="crosshair">+</div>
  <div id="message">Click PLAY GAME to begin</div>
</div>

<script type="module">
import * as THREE from
'https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.module.js';

const scene = new THREE.Scene();
scene.background = new THREE.Color(0x8bc9f5);
scene.fog = new THREE.Fog(0x8bc9f5, 65, 180);

const camera = new THREE.PerspectiveCamera(
  75, window.innerWidth / window.innerHeight, 0.1, 300
);
camera.position.set(0, 2, 12);

const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
renderer.shadowMap.enabled = true;
renderer.shadowMap.type = THREE.PCFSoftShadowMap;
document.body.appendChild(renderer.domElement);

// Lighting
scene.add(new THREE.HemisphereLight(0xffffff, 0x607080, 2));

const sunlight = new THREE.DirectionalLight(0xffffff, 2);
sunlight.position.set(-30, 50, 30);
sunlight.castShadow = true;
scene.add(sunlight);

// Ground
const ground = new THREE.Mesh(
  new THREE.PlaneGeometry(240, 240),
  new THREE.MeshStandardMaterial({ color: 0x7fc58b })
);
ground.rotation.x = -Math.PI / 2;
ground.receiveShadow = true;
scene.add(ground);

// Main paths
function makePath(x, z, width, depth) {
  const path = new THREE.Mesh(
    new THREE.BoxGeometry(width, 0.08, depth),
    new THREE.MeshStandardMaterial({ color: 0xd9d5c9 })
  );
  path.position.set(x, 0.04, z);
  scene.add(path);
}
makePath(0, 0, 14, 180);
makePath(0, 0, 180, 14);

// Building materials
const buildingColors = [
  0xffc9a8, 0xffe6a6, 0xb9dcff,
  0xd4c1ff, 0xb9efd2, 0xffc5d7
];

const buildings = [];

function createBuilding(x, z, color, width = 10, height = 8, depth = 9) {
  const group = new THREE.Group();

  const wallMat = new THREE.MeshStandardMaterial({
    color,
    roughness: 0.85
  });

  const wall = new THREE.Mesh(
    new THREE.BoxGeometry(width, height, depth),
    wallMat
  );
  wall.position.y = height / 2;
  wall.castShadow = true;
  wall.receiveShadow = true;
  group.add(wall);

  // Friendly roof
  const roof = new THREE.Mesh(
    new THREE.ConeGeometry(width * 0.72, 3.5, 4),
    new THREE.MeshStandardMaterial({ color: 0x6b83bd })
  );
  roof.position.y = height + 1.7;
  roof.rotation.y = Math.PI / 4;
  roof.castShadow = true;
  group.add(roof);

  // Front door
  const door = new THREE.Mesh(
    new THREE.BoxGeometry(1.7, 3, 0.18),
    new THREE.MeshStandardMaterial({ color: 0x79513d })
  );
  door.position.set(0, 1.5, depth / 2 + 0.1);
  group.add(door);

  // Door knob
  const knob = new THREE.Mesh(
    new THREE.SphereGeometry(0.09, 10, 10),
    new THREE.MeshStandardMaterial({ color: 0xffd75e })
  );
  knob.position.set(0.55, 1.5, depth / 2 + 0.22);
  group.add(knob);

  // Front windows
  for (const wx of [-width * 0.28, width * 0.28]) {
    const windowMesh = new THREE.Mesh(
      new THREE.BoxGeometry(1.7, 1.7, 0.15),
      new THREE.MeshStandardMaterial({
        color: 0x8de5ff,
        emissive: 0x1c536a,
        roughness: 0.25
      })
    );
    windowMesh.position.set(wx, height * 0.65, depth / 2 + 0.1);
    group.add(windowMesh);

    const frame = new THREE.Mesh(
      new THREE.BoxGeometry(1.95, 1.95, 0.12),
      new THREE.MeshStandardMaterial({ color: 0xffffff })
    );
    frame.position.set(wx, height * 0.65, depth / 2 + 0.02);
    frame.position.z -= 0.03;
    group.add(frame);
    frame.material.transparent = true;
    frame.material.opacity = 0.7;
  }

  group.position.set(x, 0, z);
  scene.add(group);
  buildings.push(group);
}

// City layout
createBuilding(-23, -24, 0xffc9a8, 11, 9, 10);
createBuilding(23, -24, 0xb9dcff, 11, 10, 10);
createBuilding(-25, 25, 0xd4c1ff, 12, 8, 11);
createBuilding(25, 25, 0xb9efd2, 12, 9, 11);
createBuilding(-48, -10, 0xffe6a6, 10, 7, 10);
createBuilding(48, 10, 0xffc5d7, 10, 8, 10);
createBuilding(-48, 42, 0xb9dcff, 11, 9, 10);
createBuilding(48, -42, 0xd4c1ff, 11, 9, 10);

// Trees
function createTree(x, z) {
  const trunk = new THREE.Mesh(
    new THREE.CylinderGeometry(0.3, 0.4, 2.5, 8),
    new THREE.MeshStandardMaterial({ color: 0x805637 })
  );
  trunk.position.set(x, 1.25, z);
  scene.add(trunk);

  const leaves = new THREE.Mesh(
    new THREE.SphereGeometry(1.5, 12, 10),
    new THREE.MeshStandardMaterial({ color: 0x36a86b })
  );
  leaves.position.set(x, 3.1, z);
  leaves.castShadow = true;
  scene.add(leaves);
}
for (let i = 0; i < 25; i++) {
  const x = (Math.random() - 0.5) * 150;
  const z = (Math.random() - 0.5) * 150;
  if (Math.abs(x) > 12 && Math.abs(z) > 12) createTree(x, z);
}

// Stars
const starGeometry = new THREE.BufferGeometry();
const starPositions = [];
for (let i = 0; i < 500; i++) {
  starPositions.push(
    (Math.random() - 0.5) * 220,
    20 + Math.random() * 80,
    (Math.random() - 0.5) * 220
  );
}
starGeometry.setAttribute(
  'position',
  new THREE.Float32BufferAttribute(starPositions, 3)
);
const stars = new THREE.Points(
  starGeometry,
  new THREE.PointsMaterial({ color: 0xffffff, size: 0.5 })
);
scene.add(stars);

// Player controls
const player = {
  position: new THREE.Vector3(0, 1.7, 12),
  velocityY: 0,
  speed: 9,
  onGround: true,
  yaw: 0,
  pitch: 0,
  health: 100,
  kills: 0
};

camera.position.copy(player.position);

const keys = {};
let isLocked = false;
let gameStarted = false;
let canShoot = true;

const overlay = document.getElementById('overlay');
const message = document.getElementById('message');

document.getElementById('playButton').addEventListener('click', () => {
  gameStarted = true;
  renderer.domElement.requestPointerLock();
});

document.addEventListener('pointerlockchange', () => {
  isLocked = document.pointerLockElement === renderer.domElement;
  overlay.style.display = isLocked ? 'none' : 'flex';
  message.textContent = isLocked
    ? 'WASD to move • Mouse to aim • Click to shoot'
    : 'Click PLAY GAME to continue';
});

document.addEventListener('keydown', event => {
  keys[event.code] = true;

  if (event.code === 'Space' && player.onGround && isLocked) {
    player.velocityY = 8;
    player.onGround = false;
  }
});

document.addEventListener('keyup', event => {
  keys[event.code] = false;
});

// Mouse look: pointer lock keeps the cursor centered while playing
document.addEventListener('mousemove', event => {
  if (!isLocked) return;

  const sensitivity = 0.002;
  player.yaw -= event.movementX * sensitivity;
  player.pitch -= event.movementY * sensitivity;

  player.pitch = THREE.MathUtils.clamp(
    player.pitch, -1.35, 1.35
  );

  camera.rotation.order = 'YXZ';
  camera.rotation.y = player.yaw;
  camera.rotation.x = player.pitch;
});

// Enemy skeletons
const enemies = [];

function createEnemy(x, z) {
  const enemy = new THREE.Group();

  const bone = new THREE.MeshStandardMaterial({ color: 0xf1ead9 });
  const dark = new THREE.MeshStandardMaterial({ color: 0x202333 });
  const red = new THREE.MeshStandardMaterial({
    color: 0xff3333,
    emissive: 0x550000
  });

  const body = new THREE.Mesh(
    new THREE.CapsuleGeometry(0.45, 0.8, 4, 8), dark
  );
  body.position.y = 1.2;
  enemy.add(body);

  const skull = new THREE.Mesh(
    new THREE.SphereGeometry(0.48, 12, 12), bone
  );
  skull.position.y = 2.05;
  enemy.add(skull);

  for (const eyeX of [-0.17, 0.17]) {
    const eye = new THREE.Mesh(
      new THREE.SphereGeometry(0.09, 8, 8), red
    );
    eye.position.set(eyeX, 2.1, 0.4);
    enemy.add(eye);
  }

  for (const side of [-1, 1]) {
    const arm = new THREE.Mesh(
      new THREE.CylinderGeometry(0.12, 0.14, 0.9, 8), bone
    );
    arm.position.set(side * 0.58, 1.3, 0);
    arm.rotation.z = side * -0.25;
    enemy.add(arm);

    const leg = new THREE.Mesh(
      new THREE.CylinderGeometry(0.13, 0.16, 0.8, 8), bone
    );
    leg.position.set(side * 0.22, 0.4, 0);
    enemy.add(leg);
  }

  enemy.position.set(x, 0, z);
  scene.add(enemy);
  enemies.push({ mesh: enemy, hp: 3, alive: true });
}

createEnemy(-12, -18);
createEnemy(12, -20);
createEnemy(-18, 18);
createEnemy(18, 18);
createEnemy(0, -35);

// Shooting
const raycaster = new THREE.Raycaster();
const center = new THREE.Vector2(0, 0);

function shoot() {
  if (!isLocked || !canShoot) return;
  canShoot = false;

  setTimeout(() => canShoot = true, 220);

  raycaster.setFromCamera(center, camera);

  const targets = enemies
    .filter(enemy => enemy.alive)
    .map(enemy => enemy.mesh);

  const hits = raycaster.intersectObjects(targets, true);

  if (hits.length > 0) {
    let hitObject = hits[0].object;
    let enemyData = enemies.find(enemy =>
      enemy.alive && enemy.mesh === hitObject.parent
    );

    // Find the enemy group containing the hit mesh
    if (!enemyData) {
      enemyData = enemies.find(enemy =>
        enemy.alive && enemy.mesh.children.includes(hitObject)
      );
    }

    if (!enemyData) {
      enemyData = enemies.find(enemy =>
        enemy.alive && enemy.mesh === hitObject
      );
    }

    if (enemyData) {
      enemyData.hp--;

      // Hit flash
      enemyData.mesh.scale.setScalar(0.92);
      setTimeout(() => {
        if (enemyData.alive) enemyData.mesh.scale.setScalar(1);
      }, 90);

      if (enemyData.hp <= 0) {
        enemyData.alive = false;
        scene.remove(enemyData.mesh);
        player.kills++;
        document.getElementById('kills').textContent = player.kills;
        message.textContent = 'Enemy eliminated!';
      } else {
        message.textContent = 'Hit! Keep shooting!';
      }
    }
  }
}

document.addEventListener('mousedown', event => {
  if (event.button === 0) shoot();
});

// Movement and collision boundary
const clock = new THREE.Clock();

function updatePlayer(delta) {
  if (!isLocked) return;

  const forward = new THREE.Vector3(
    -Math.sin(player.yaw), 0, -Math.cos(player.yaw)
  );
  const right = new THREE.Vector3(
    Math.cos(player.yaw), 0, -Math.sin(player.yaw)
  );

  const move = new THREE.Vector3();

  if (keys.KeyW) move.add(forward);
  if (keys.KeyS) move.sub(forward);
  if (keys.KeyD) move.add(right);
  if (keys.KeyA) move.sub(right);

  if (move.lengthSq() > 0) move.normalize();

  const speed = keys.ShiftLeft || keys.ShiftRight
    ? player.speed * 1.65
    : player.speed;

  player.position.addScaledVector(move, speed * delta);

  // Keep player inside the city
  player.position.x = THREE.MathUtils.clamp(
    player.position.x, -85, 85
  );
  player.position.z = THREE.MathUtils.clamp(
    player.position.z, -85, 85
  );

  // Gravity and jumping
  player.velocityY -= 20 * delta;
  player.position.y += player.velocityY * delta;

  if (player.position.y < 1.7) {
    player.position.y = 1.7;
    player.velocityY = 0;
    player.onGround = true;
  }

  camera.position.copy(player.position);
  camera.rotation.order = 'YXZ';
  camera.rotation.y = player.yaw;
  camera.rotation.x = player.pitch;
}

// Enemy movement
function updateEnemies(delta) {
  for (const enemy of enemies) {
    if (!enemy.alive) continue;

    const direction = new THREE.Vector3(
      player.position.x - enemy.mesh.position.x,
      0,
      player.position.z - enemy.mesh.position.z
    );

    const distance = direction.length();

    if (distance > 2.2 && distance < 35) {
      direction.normalize();
      enemy.mesh.position.addScaledVector(direction, delta * 2.2);
      enemy.mesh.rotation.y = Math.atan2(
        direction.x, direction.z
      );
    }
  }
}

// Main loop
function animate() {
  requestAnimationFrame(animate);

  const delta = Math.min(clock.getDelta(), 0.05);

  updatePlayer(delta);
  updateEnemies(delta);

  renderer.render(scene, camera);
}
animate();

// Resize
window.addEventListener('resize', () => {
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(window.innerWidth, window.innerHeight);
});
</script>
</body>
</html>
