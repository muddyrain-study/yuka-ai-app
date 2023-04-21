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
const loader = new GLTFLoader()
const dracoLoader = new DRACOLoader()
dracoLoader.setDecoderPath('/draco/')
loader.setDRACOLoader(dracoLoader)
let c1, c2

c1 = loader.loadAsync('/model/car.gltf')
c2 = loader.loadAsync('/model/truck.gltf')

Promise.all([c1, c2]).then(res => {

  const truck = res[1].scene
  truck.children[0].rotation.y = Math.PI / 2
  truck.children[0].scale.set(0.25, 0.25, 0.25)

  const vehicle2 = new YUKA.Vehicle()
  vehicle2.position.set(
    Math.random() * 20 - 10,
    0,
    Math.random() * 20 - 10
  )
  vehicle2.setRenderComponent(truck, callback)
  entityManager.add(vehicle2)
  scene.add(truck)



  // 设置卡车追击行为  第二个参数 - 预测转弯
  // const pursuitBehavior = new YUKA.PursuitBehavior(vehicle2)
  // vehicle.steering.add(pursuitBehavior)

  const offsets = [
    new YUKA.Vector3(0, 0, 1),
    new YUKA.Vector3(-1, 0, 1),
    new YUKA.Vector3(1, 0, 1),
    new YUKA.Vector3(-3, 0, 3),
    new YUKA.Vector3(3, 0, 3),
  ]

  for (let i = 0; i < offsets.length; i++) {
    const car = res[0].scene.clone()
    car.children[0].rotation.y = Math.PI / 2
    car.children[0].scale.set(0.25, 0.25, 0.25)
    const vehicle = new YUKA.Vehicle()
    vehicle.maxSpeed = 6
    vehicle.position.set(
      Math.random() * 20 - 10,
      0,
      Math.random() * 20 - 10
    )
    vehicle.setRenderComponent(car, callback)
    entityManager.add(vehicle)
    scene.add(car)
    const offsetPursuitBehavior = new YUKA.OffsetPursuitBehavior(vehicle2, offsets[i])
    vehicle.steering.add(offsetPursuitBehavior)
  }

  // 设置卡车到达目标行为
  const arriveBehavior = new YUKA.ArriveBehavior(target.position)
  vehicle2.steering.add(arriveBehavior)
  vehicle2.maxSpeed = 4

  setInterval(() => {
    target.position.set(Math.random() * 50 - 25, 0, Math.random() * 50 - 25)
  }, 3000);
})

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
