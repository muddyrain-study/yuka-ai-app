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
plane.position.y = 0
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
const loader = new GLTFLoader();
const dracoLoader = new DRACOLoader();
dracoLoader.setDecoderPath("./draco/");
loader.setDRACOLoader(dracoLoader);
loader.load("./model/car.gltf", function (gltf) {
  // console.log(gltf);
  // 创建群体随机行走行为
  const wanderBehavior = new YUKA.WanderBehavior(3);

  // 设置整齐群体转向
  const alignmentBehavior = new YUKA.AlignmentBehavior();
  alignmentBehavior.weight = 5;

  // 设置聚集行为
  const cohesionBehavior = new YUKA.CohesionBehavior();
  cohesionBehavior.weight = 5;

  // 设置分离行为
  const separationBehavior = new YUKA.SeparationBehavior();
  separationBehavior.weight = 0.5;

  for (let i = 0; i < 40; i++) {
    // 创建yuka的车辆
    const vehicle = new YUKA.Vehicle();
    vehicle.position.set(Math.random() * 20 - 10, 0, Math.random() * 20 - 10);
    vehicle.rotation.fromEuler(0, Math.random() * Math.PI, 0);
    vehicle.maxSpeed = 3;
    const car = gltf.scene.clone();
    car.children[0].rotation.y = Math.PI / 2;
    car.children[0].scale.set(0.2, 0.2, 0.2);
    scene.add(car);
    vehicle.setRenderComponent(car, callback);
    entityManager.add(vehicle);

    vehicle.steering.add(wanderBehavior);
    vehicle.steering.add(alignmentBehavior);
    vehicle.steering.add(cohesionBehavior);
    vehicle.steering.add(separationBehavior);
  }
});

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
// entityManager.add(vehicle);
entityManager.add(target);


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
