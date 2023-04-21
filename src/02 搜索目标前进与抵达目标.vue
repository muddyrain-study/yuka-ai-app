<script setup>
import * as THREE from "three"
import * as YUKA from "yuka"
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls"
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader"
import { DRACOLoader } from "three/examples/jsm/loaders/DRACOLoader"

// 创建场景
const scene = new THREE.Scene();
// 创建相机
const camera = new THREE.PerspectiveCamera(
  75,
  window.innerWidth / window.innerHeight,
  0.1,
  1000
);
camera.position.set(0, 10, 20);
camera.lookAt(0, 0, 0);


// 创建渲染器
const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.shadowMap.enabled = true;
renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);


// 创建地面
const planeGeometry = new THREE.PlaneGeometry(50, 50)
const planeMaterial = new THREE.MeshStandardMaterial({
  color: 0x999999
})
const plane = new THREE.Mesh(planeGeometry, planeMaterial)
plane.receiveShadow = true
plane.rotation.x = -Math.PI / 2
plane.position.y = -0.5
scene.add(plane);

// 创建灯光
const light = new THREE.SpotLight(0xffffff, 3, 100, Math.PI / 6, 0.5);
light.position.set(10, 40, 10);
light.castShadow = true;
scene.add(light);

// 创建控制器
const controls = new OrbitControls(camera, renderer.domElement);
controls.enableDamping = true;

const time = new YUKA.Time();
requestAnimationFrame(function animate() {
  const delta = time.update().getDelta();
  controls.update();
  entityManager.update(delta);
  renderer.render(scene, camera);
  requestAnimationFrame(animate);
});

// 创建椎体
// const coneGeometry = new THREE.ConeGeometry(0.2, 1, 32);
// coneGeometry.rotateX(Math.PI / 2);
// const coneMaterial = new THREE.MeshStandardMaterial({ color: 0xff0000 });
// const cone = new THREE.Mesh(coneGeometry, coneMaterial);
// // cone.matrixAutoUpdate = false;
// cone.receiveShadow = true;
// cone.castShadow = true;
// scene.add(cone);

// 加载模型
const loader = new GLTFLoader()
const dracoLoader = new DRACOLoader()
dracoLoader.setDecoderPath('/draco/')
loader.setDRACOLoader(dracoLoader)
loader.load("./model/car.gltf", function (gltf) {
  const car = gltf.scene
  car.children[0].rotation.y = Math.PI / 2
  car.children[0].scale.set(0.25, 0.25, 0.25)
  scene.add(car)
  vehicle.setRenderComponent(car, callback);

})

// 创建 yuka 车辆
const vehicle = new YUKA.Vehicle()
vehicle.maxSpeed = 5;
// 设置车辆的渲染对象
function callback(entity, renderComponent) {
  renderComponent.position.copy(entity.position);
  renderComponent.quaternion.copy(entity.rotation);
  // renderComponent.matrix.copy(entity.worldMatrix);
}
// 创建目标小球
const sphereGeometry = new THREE.SphereGeometry(0.2, 32, 32)
const sphereMaterial = new THREE.MeshStandardMaterial({
  color: 0xff00ff
})
const sphere = new THREE.Mesh(sphereGeometry, sphereMaterial)
sphere.receiveShadow = true
sphere.castShadow = true

scene.add(sphere)

// 创建目标
const target = new YUKA.GameEntity();
target.setRenderComponent(sphere, callback)

target.position.set(
  Math.random() * 20 - 10,
  0,
  Math.random() * 20 - 10,
)
// 创建对实体管理对象
const entityManager = new YUKA.EntityManager();
entityManager.add(vehicle);
entityManager.add(target);

// 抵达行为
const arriveBehavior = new YUKA.ArriveBehavior(target.position, 1)
vehicle.steering.add(arriveBehavior)

// 搜索目标行为
// const seekBehavior = new YUKA.SeekBehavior(target.position)
// vehicle.steering.add(seekBehavior)

setInterval(() => {
  target.position.set(
    Math.random() * 20 - 10,
    0,
    Math.random() * 20 - 10,
  )
}, 2000);

function showPathLine(path) {
  console.log(path);
  const positions = []
  for (let i = 0; i < path._waypoints.length; i++) {
    positions.push(
      path._waypoints[i].x,
      path._waypoints[i].y,
      path._waypoints[i].z,
    )
  }
  const geometry = new THREE.BufferGeometry()
  geometry.setAttribute(
    'position',
    new THREE.Float32BufferAttribute(positions, 3)
  )
  const material = new THREE.LineBasicMaterial({ color: 0x0000ff })
  const line = new THREE.Line(geometry, material)
  scene.add(line)
}

</script>

<template>
  <div></div>
</template>

<style >
* {
  padding: 0;
  margin: 0;
}

canvas {
  width: 100vw;
  height: 100vh;
  position: fixed;
  left: 0;
  top: 0;
}
</style>
