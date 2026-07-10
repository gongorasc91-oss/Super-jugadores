<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>SÚPER CAMPEONES 3D</title>
<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
<style>
body{margin:0;overflow:hidden;background:linear-gradient(#87CEEB,#32CD32);font-family:'Arial Black'}
#menu{position:absolute;width:100%;height:100%;background:#000000dd;display:flex;flex-direction:column;align-items:center;justify-content:center;color:white}
#personajes{display:flex;gap:10px;flex-wrap:wrap;justify-content:center;max-width:400px}
.char{padding:10px;background:#222;border:3px solid gold;border-radius:10px;cursor:pointer;text-align:center;width:100px}
.char:hover{background:#444}
#ui{position:absolute;top:10px;width:100%;display:flex;justify-content:space-around;color:white;font-size:22px;text-shadow:2px 2px #000}
#tecnica{position:absolute;bottom:80px;width:100%;text-align:center;font-size:50px;color:yellow;text-shadow:0 0 20px red;display:none}
#btnTiro{position:absolute;bottom:20px;left:50%;transform:translateX(-50%);padding:25px 60px;font-size:28px;background:linear-gradient(45deg,red,orange);border:none;border-radius:30px;color:white;box-shadow:0 0 40px red}
</style>
</head>
<body>
<div id="menu">
  <h1>ELIGE A TU LEYENDA</h1>
  <div id="personajes"></div>
</div>

<div id="ui">
  <div>⚽ GOLES: <span id="goles">0</span></div>
  <div id="jugadorSel"></div>
</div>
<div id="tecnica"></div>
<button id="btnTiro">DESLIZA Y TIRA</button>

<script>
const PERSONAJES = [
  {nombre:"OLIVER ATOM", color:0x0000FF, tecnica:"TIRO CON EFECTO", poder:90},
  {nombre:"STEVE HYUGA", color:0xFF0000, tecnica:"TIRO DEL TIGRE", poder:100},
  {nombre:"MARK LANDERS", color:0x00FF00, tecnica:"TIRO COMETA", poder:95},
  {nombre:"JULIAN ROSS", color:0xFFFF00, tecnica:"TIRO ARCOÍRIS", poder:88},
  {nombre:"RICHARD TEX-TEX", color:0xFF00FF, tecnica:"TIRO DE LA NIEVE", poder:92},
  {nombre:"ROBERTO", color:0x00FFFF, tecnica:"TIRO DE LA PALOMA", poder:97}
];

let scene,camera,renderer,balon,arquero;
let jugadorActual=null, goles=0, arrastrando=false, startX=0;

function init(){
  // CREAR MENU DE PERSONAJES
  const cont=document.getElementById('personajes');
  PERSONAJES.forEach((p,i)=>{
    let div=document.createElement('div');
    div.className='char';
    div.innerHTML=`<div style="width:50px;height:50px;background:#${p.color.toString(16)};border-radius:50%;margin:auto"></div>${p.nombre}<br><small>${p.tecnica}</small>`;
    div.onclick=()=>seleccionar(i);
    cont.appendChild(div);
  });

  // ESCENA 3D
  scene=new THREE.Scene();
  camera=new THREE.PerspectiveCamera(60,innerWidth/innerHeight,0.1,1000);
  camera.position.set(0,4,16);

  renderer=new THREE.WebGLRenderer({antialias:true});
  renderer.setSize(innerWidth,innerHeight);
  document.body.appendChild(renderer.domElement);

  scene.add(new THREE.AmbientLight(0xffffff,0.8));
  let sol=new THREE.DirectionalLight(0xffffff,1); sol.position.set(5,10,5); scene.add(sol);

  let cancha=new THREE.Mesh(new THREE.PlaneGeometry(20,30), new THREE.MeshStandardMaterial({color:0x32CD32}));
  cancha.rotation.x=-Math.PI/2; scene.add(cancha);

  let arco=new THREE.Mesh(new THREE.BoxGeometry(4.5,2.5,0.3), new THREE.MeshStandardMaterial({color:0xffffff}));
  arco.position.set(0,1.2,-12); scene.add(arco);

  balon=new THREE.Mesh(new THREE.SphereGeometry(0.4,32,32), new THREE.MeshStandardMaterial({color:0xffffff}));
  balon.position.set(0,0.4,8); scene.add(balon);

  arquero=new THREE.Mesh(new THREE.CapsuleGeometry(0.6,1.8,4,8), new THREE.MeshStandardMaterial({color:0x0000FF}));
  arquero.position.set(0,1.5,-11); scene.add(arquero);

  animate();
}

function seleccionar(i){
  jugadorActual=PERSONAJES[i];
  document.getElementById('menu').style.display='none';
  document.getElementById('jugadorSel').innerText=jugadorActual.nombre;
}

function dispararTecnica(){
  if(!jugadorActual) return;

  // MOSTRAR NOMBRE DE TÉCNICA
  let tech=document.getElementById('tecnica');
  tech.innerText="¡"+jugadorActual.tecnica+"!";
  tech.style.display='block';
  setTimeout(()=>tech.style.display='none',1200);

  // EFECTO SEGÚN PERSONAJE
  let velocidad=jugadorActual.poder/18;
  let efecto=0, color=0xffffff;

  if(jugadorActual.nombre.includes("HYUGA")){efecto=3; color=0xFF4500} // Fuego
  if(jugadorActual.nombre.includes("ATOM")){efecto=Math.sin(Date.now()*0.01)*2} // Efecto
  if(jugadorActual.nombre.includes("LANDERS")){velocidad*=1.3; color=0xFFD700} // Rápido

  balon.material.color.set(color);

  let anim=setInterval(()=>{
    balon.position.z-=velocidad;
    balon.position.x+=efecto*0.05;
    balon.rotation.x+=0.3;
