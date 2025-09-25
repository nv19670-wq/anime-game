<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Anime 3D Mini-Game</title>
  <style>
    html,body{height:100%;margin:0;background:#081125;color:#eef;font-family:sans-serif}
    #container{width:100%;height:100vh;display:block}
    .ui{position:absolute;left:12px;top:12px;z-index:12;background:rgba(0,0,0,0.5);padding:10px;border-radius:8px}
    .hud{position:absolute;right:12px;top:12px;z-index:12;text-align:right}
    .bar{width:200px;height:14px;border-radius:6px;background:rgba(255,255,255,0.1);overflow:hidden;margin-bottom:6px}
    .bar .fill{height:100%;width:100%;background:linear-gradient(90deg,#ff6b6b,#ffcc99)}
    .score{font-weight:700;font-size:16px}
    .centerMsg{position:absolute;left:50%;top:50%;transform:translate(-50%,-50%);z-index:13;
               padding:16px 20px;background:rgba(0,0,0,0.7);border-radius:10px;display:none}
  </style>
</head>
<body>
  <div id="container"></div>
  <div class="ui">
    <div style="font-weight:bold">Controls</div>
    <div>WASD = Move</div>
    <div>Space = Jump</div>
    <div>Left Click = Attack</div>
    <div>Right Click = Ranged Attack</div>
  </div>
  <div class="hud">
    <div class="bar"><div id="hpFill" class="fill"></div></div>
    <div class="score">Score: <span id="score">0</span></div>
  </div>
  <div id="centerMsg" class="centerMsg"></div>

  <script type="module">
    import * as THREE from 'https://cdn.jsdelivr.net/npm/three@0.180.0/build/three.module.js';
    import { OrbitControls } from 'https://cdn.jsdelivr.net/npm/three@0.180.0/examples/jsm/controls/OrbitControls.js';
    import { GLTFLoader } from 'https://cdn.jsdelivr.net/npm/three@0.180.0/examples/jsm/loaders/GLTFLoader.js';

    const container = document.getElementById('container');
    const scoreEl = document.getElementById('score');
    const hpFill = document.getElementById('hpFill');
    const centerMsg = document.getElementById('centerMsg');

    const scene = new THREE.Scene();
    scene.background = new THREE.Color(0x081125);
    const camera = new THREE.PerspectiveCamera(55, innerWidth/innerHeight, 0.1, 500);
    camera.position.set(0,4,9);
    const renderer = new THREE.WebGLRenderer({antialias:true});
    renderer.setSize(innerWidth, innerHeight);
    document.body.appendChild(renderer.domElement);

    const controls = new OrbitControls(camera, renderer.domElement);
    controls.target.set(0,1.2,0);

    // Lights
    scene.add(new THREE.HemisphereLight(0xffffff,0x333355,0.6));
    const dirLight = new THREE.DirectionalLight(0xffffff,1);
    dirLight.position.set(5,10,5);
    scene.add(dirLight);

    // Ground
    const ground = new THREE.Mesh(new THREE.PlaneGeometry(50,50),
                   new THREE.MeshStandardMaterial({color:0x334455}));
    ground.rotation.x = -Math.PI/2;
    scene.add(ground);

    // Player model (Robot Expressive, always works)
    const loader = new GLTFLoader();
    const player = new THREE.Group();
    scene.add(player);
    let mixer, actions={}, activeAction;
    loader.load(
      'https://cdn.jsdelivr.net/gh/KhronosGroup/glTF-Sample-Models@master/2.0/RobotExpressive/glTF-Binary/RobotExpressive.glb',
      gltf=>{
        player.add(gltf.scene);
        mixer = new THREE.AnimationMixer(gltf.scene);
        gltf.animations.forEach(clip=>{
          if(clip.name.toLowerCase().includes('idle')) actions.idle=mixer.clipAction(clip);
          if(clip.name.toLowerCase().includes('walk')) actions.walk=mixer.clipAction(clip);
          if(clip.name.toLowerCase().includes('dance')||clip.name.toLowerCase().includes('attack'))
            actions.attack=mixer.clipAction(clip);
        });
        Object.values(actions).forEach(a=>a.play());
        activeAction = actions.idle;
      }
    );

    // Health + Score
    let hp=100, score=0;
    function updateHP(){ hp=Math.max(0,hp); hpFill.style.width=hp+'%'; if(hp<=0) gameOver(); }
    function addScore(n){ score+=n; scoreEl.textContent=score; }

    // Simple cube enemy
    const enemy = new THREE.Mesh(new THREE.BoxGeometry(1,1,1), new THREE.MeshStandardMaterial({color:0xff6666}));
    enemy.position.set(3,0.5,-3);
    scene.add(enemy);

    // Keys
    const keys={w:0,a:0,s:0,d:0,space:0};
    window.addEventListener('keydown',e=>{if(keys.hasOwnProperty(e.key)) keys[e.key]=1;});
    window.addEventListener('keyup',e=>{if(keys.hasOwnProperty(e.key)) keys[e.key]=0;});

    function gameOver(){ centerMsg.style.display='block'; centerMsg.innerHTML='Game Over<br><small>Refresh to try again</small>'; }

    // Animate
    const clock=new THREE.Clock();
    function animate(){
      const dt=clock.getDelta();
      if(mixer) mixer.update(dt);
      controls.update();
      renderer.render(scene,camera);
      requestAnimationFrame(animate);
    }
    animate();
  </script>
</body>
</html>
