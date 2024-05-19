<script setup>
import { onMounted } from "vue";
import * as THREE from "three";

// 容器 场景
let container, scene, width, height, depth;
// 加载图片
const IMAGE_SKY = new URL("../../assets/images/sky.jpg", import.meta.url).href;

onMounted(() => {
  container = document.getElementById("login-three-container");
  width = container.clientWidth;
  height = container.clientHeight;
  depth = 1400;
  console.log(container);
  initScene();
  initCamera();
});

// 初始化场景
const initScene = () => {
  scene = new THREE.Scene();
  // 添加雾的效果
  scene.fog = new THREE.Fog(0x000000, 10, 10000);
};

// 添加背景
const initSceneBackground = () => {
  new THREE.TextureLoader().load("IMAGE_SKY", (texture) => {});
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
};

const initCamera = () => {
  const fov = 15;
  const distance = width / 2 / Math.tan(Math.PI / 12);
  const zAxisNumber = Math.floor(distance - depth / 2);
  const camera = new THREE.PerspectiveCamera(fov, width / height, 1, 30000);
  camera.position.set(0, 0, zAxisNumber);
  camera.lookAt(scene.position);
  scene.add(camera);
};

// 添加光源
const initLight = () => {
  const light = new THREE.PointLight(0xffffff, 1);
  light.position.set(0, 0, 0);
  scene.add(light);
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
