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
  if (navMesh) {
    const currentRegion = navMesh.getRegionForPoint(vehicle.position)
    if (currentRegion) {
      const distance = currentRegion.distanceToPoint(vehicle.position)
      vehicle.position.y -= distance
    }
  }
  entityManager.update(delta);
  renderer.render(scene, camera);
  requestAnimationFrame(animate);
});

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
let plane
loader.load("./model/modelMap.gltf", gltf => {
  plane = gltf.scene
  plane.traverse(child => {
    if (child.isMesh) {
      child.receiveShadow = true
      child.castShadow = true
    }
  })
  scene.add(plane)
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


// 点击将目标移动到点击的位置
const ndc = new THREE.Vector2()
const raycaster = new THREE.Raycaster()
window.addEventListener('pointerdown', (event) => {
  ndc.x = (event.clientX / window.innerWidth) * 2 - 1;
  ndc.y = - (event.clientY / window.innerHeight) * 2 + 1;
  raycaster.setFromCamera(ndc, camera);
  const intersects = raycaster.intersectObject(plane);
  if (intersects.length > 0) {
    const point = intersects[0].point
    target.position.set(point.x, point.y, point.z)
    let from = vehicle.position
    let to = point
    // 根据导航网格获取路径
    const path = navMesh.findPath(from, to)

    const path1 = new YUKA.Path()
    for (let item of path) {
      path1.add(new YUKA.Vector3(item.x, item.y, item.z))
    }
    showPathLine(path1)

    vehicle.steering.clear()
    // 跟随路径
    // const followPathBehavior = new YUKA.FollowPathBehavior(path1)
    // followPathBehavior.weight = 10
    // vehicle.steering.add(followPathBehavior)

    const onPathBehavior = new YUKA.OnPathBehavior(path1, 0.1, 0.1);
    onPathBehavior.weight = 10;
    vehicle.steering.add(onPathBehavior);

    const arriveBehavior = new YUKA.ArriveBehavior(to, 3, 0.1);
    arriveBehavior.weight = 1;
    vehicle.steering.add(arriveBehavior);
  }
})
let line
function showPathLine(path) {
  if (line) {
    scene.remove(line)
    line.geometry.dispose()
    line.material.dispose()
  }
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
  line = new THREE.Line(geometry, material)
  scene.add(line)
}

// 网格加载器
const navMeshLoader = new YUKA.NavMeshLoader()
// 加载网格
let navMesh
navMeshLoader.load("./model/modelMap.gltf").then((navigationMesh) => {
  navMesh = navigationMesh
})
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
