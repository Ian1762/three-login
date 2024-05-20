<script setup>
import { h, onMounted } from "vue";
import * as THREE from "three";
import { OrbitControls } from "three/addons/controls/OrbitControls.js";

// 容器 场景
let container, scene, width, height, depth, camera, renderer, controls, earth;
// 加载图片
const IMAGE_SKY = new URL("../../assets/images/sky.png", import.meta.url).href;
const IMAGE_EARTH = new URL("../../assets/images/earth_bg.png", import.meta.url)
  .href;

onMounted(() => {
  container = document.getElementById("login-three-container");
  width = container.clientWidth;
  height = container.clientHeight;
  depth = 1400;
  console.log("size", width, height, depth);
  console.log(container);
  initScene();
  initSceneBackground();
  addEarth();
  initLight();
  initCamera();
  initRender();
  initOrbitControls();
  animate();
});

// 初始化场景
const initScene = () => {
  scene = new THREE.Scene();
  // 添加雾的效果
  scene.fog = new THREE.Fog(0x000000, 10, 10000);
  // const axesHelper = new THREE.AxesHelper(10000);
  // scene.add(axesHelper);
};

// 添加背景
const initSceneBackground = () => {
  console.log("IMAGE_SKY", IMAGE_SKY);
  new THREE.TextureLoader().load(IMAGE_SKY, (texture) => {
    console.log("texture", texture);
    // 创建立方体
    const geometry = new THREE.BoxGeometry(width, height, depth);
    // 材质
    const material = new THREE.MeshBasicMaterial({
      map: texture,
      side: THREE.BackSide,
    });
    // 网格
    const mesh = new THREE.Mesh(geometry, material);
    // 添加到场景中
    scene.add(mesh);
  });
};

// 添加地球
const addEarth = () => {
  console.log("IMAGE_EARTH", IMAGE_EARTH);
  new THREE.TextureLoader().load(IMAGE_EARTH, (texture) => {
    const geometry = new THREE.SphereGeometry(50, 64, 32);
    const material = new THREE.MeshPhongMaterial({ map: texture });
    earth = new THREE.Mesh(geometry, material);
    earth.position.set(-400, 200, -200);
    scene.add(earth);
  });
};

// 地球自转
const earthRotate = () => {
  earth?.rotateY(0.001);
};

// 光源
const initLight = () => {
  // 环境光
  const ambientLight = new THREE.AmbientLight(0xffffff);
  scene.add(ambientLight);
  // 电光源
  const pointLight = new THREE.PointLight(0x0655fd, 5, 0);
  pointLight.position.set(0, 100, -200);
  scene.add(pointLight);

  // const sphereSize = 10;
  // const pointLightHelper = new THREE.PointLightHelper(pointLight, sphereSize);
  // scene.add(pointLightHelper);
};

// 初始化相机
const initCamera = () => {
  // 摄像机视锥体垂直视野角度
  const fov = 30;
  const radian = 180 / (fov / 2);
  // 相机距离立方体的距离
  const distance = width / 2 / Math.tan(Math.PI / radian);
  // 相机在z轴上的距离
  const zAxisNumber = Math.floor(distance - depth / 2);
  camera = new THREE.PerspectiveCamera(fov, width / height, 1, 30000);
  console.log("zAxisNumber", zAxisNumber);
  // 设置相机位置
  camera.position.set(0, 0, zAxisNumber);
  // 看向原点
  camera.lookAt(0, 0, 0);
  scene.add(camera);
};

// 初始化渲染器
const initRender = () => {
  renderer = new THREE.WebGLRenderer();
  //  定义渲染器尺寸
  renderer.setSize(width, height);
  container.appendChild(renderer.domElement);
};

// 添加轨道控制器
const initOrbitControls = () => {
  controls = new OrbitControls(camera, renderer.domElement);
  controls.update();
};

// 持续动画
const animate = () => {
  requestAnimationFrame(animate);

  controls.update();
  earthRotate();
  renderer.render(scene, camera);
};
</script>

<template>
  <div id="login-three-container"></div>
</template>

<style lang='less' scoped>
#login-three-container {
  width: 100%;
  height: 100vh;
}
</style>
