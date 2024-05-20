<script setup>
import { h, onMounted } from "vue";
import * as THREE from "three";
import { OrbitControls } from "three/addons/controls/OrbitControls.js";

// 容器 场景
let container,
  scene,
  width,
  height,
  depth,
  camera,
  renderer,
  controls,
  earth,
  zAxisNumber,
  starsPositionInit,
  meshPoints,
  parameters;
// 加载图片
const IMAGE_SKY = new URL("../../assets/images/sky.png", import.meta.url).href;
const IMAGE_EARTH = new URL("../../assets/images/earth_bg.png", import.meta.url)
  .href;
const IMAGE_STAR1 = new URL(
  "../../assets/images/starflake1.png",
  import.meta.url
).href;
const IMAGE_STAR2 = new URL(
  "../../assets/images/starflake2.png",
  import.meta.url
).href;

onMounted(() => {
  container = document.getElementById("login-three-container");
  width = container.clientWidth;
  height = container.clientHeight;
  depth = 1400;
  console.log("size", width, height, depth);
  // console.log(container);
  initScene();
  initSceneBackground();
  addEarth();
  initLight();
  initCamera();
  starsPositionInit = -(zAxisNumber + depth / 2);
  addSceneStar(starsPositionInit);
  initRender();
  initOrbitControls();
  animate();
});

// 初始化场景
const initScene = () => {
  scene = new THREE.Scene();
  // 添加雾的效果
  // scene.fog = new THREE.Fog(0x000000, 10, 10000);
  // const axesHelper = new THREE.AxesHelper(10000);
  // scene.add(axesHelper);
};

// 添加背景
const initSceneBackground = () => {
  // console.log("IMAGE_SKY", IMAGE_SKY);
  new THREE.TextureLoader().load(IMAGE_SKY, (texture) => {
    // console.log("texture", texture);
    // 创建立方体
    const geometry = new THREE.BoxGeometry(width, height, depth);
    // 材质
    const material = new THREE.MeshBasicMaterial({
      map: texture,
      side: THREE.BackSide,
    });
    // 网格
    const mesh = new THREE.Mesh(geometry, material);
    // console.log(mesh, geometry);
    // 添加到场景中
    scene.add(mesh);
  });
};

// 添加地球
const addEarth = () => {
  // console.log("IMAGE_EARTH", IMAGE_EARTH);
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

// 添加场景星星
const addSceneStar = (initZposition) => {
  // 创建几何体基类
  const geometry = new THREE.BufferGeometry();
  // 星星位置的坐标
  const vertices = [];
  // 创建纹理
  const texture1 = new THREE.TextureLoader().load(IMAGE_STAR1);
  const texture2 = new THREE.TextureLoader().load(IMAGE_STAR2);
  // 声明点的参数
  parameters = [
    [[0.6, 100, 0.75], texture1, 50],
    [[0, 0, 1], texture2, 20],
  ];
  // 创建1500个星星
  for (let index = 0; index < 1500; index++) {
    const x = THREE.MathUtils.randFloatSpread(width);
    const y = THREE.MathUtils.randFloat(0, height / 2);
    const z = THREE.MathUtils.randFloat(-depth / 2, zAxisNumber);
    vertices.push(x, y, z);
  }
  // console.log(vertices);
  geometry.setAttribute(
    "position",
    new THREE.Float32BufferAttribute(vertices, 3)
  );

  // 设置2个定点组，形成两个材质
  geometry.addGroup(0, 750, 0);
  geometry.addGroup(750, 1499, 1);

  // 创建两种不同的材质
  const materials = [];
  for (let index = 0; index < parameters.length; index++) {
    const element = parameters[index];
    // 颜色
    const color = element[0];
    // 纹理
    const texture = element[1];
    // 点的大小
    const size = element[2];
    materials[index] = new THREE.PointsMaterial({
      size,
      map: texture,
      blending: THREE.AdditiveBlending,
      transparent: true,
    });
    //  设置颜色HSL
    materials[index].color.setHSL(...color);
  }

  // 创建点网格
  meshPoints = new THREE.Points(geometry, materials);
  // meshPoints.rotation.x = Math.random() * 0.2 - 0.15;
  // meshPoints.rotation.y = Math.random() * 0.2 - 0.15;
  // meshPoints.rotation.z = Math.random() * 0.2 - 0.15;
  meshPoints.position.setZ(initZposition);
  console.log("meshPoints", geometry, meshPoints);

  scene.add(meshPoints);
};

// 星星的运动
const renderStarMoving = () => {
  // 星星的颜色交替变化
  const time = Date.now() * 0.00005;
  for (let index = 0; index < parameters.length; index++) {
    const color = parameters[index][0];
    const h = ((360 * (color[0] + time)) % 360) / 360;
    meshPoints?.material?.[index]?.color?.setHSL(
      color[0],
      color[1],
      parseFloat(h.toFixed(2))
    );
    // materials[index].color.setHSL(color[0], color[1], parseFloat(h.toFixed(2)));
  }
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
  const fov = 15;
  const ratio = 180 / fov;
  // 相机距离立方体的距离
  const distance = width / 2 / Math.tan(Math.PI / ratio);
  // 相机在z轴上的距离
  zAxisNumber = Math.floor(distance - depth / 2);
  console.log("distance", distance, "zAxisNumber", zAxisNumber);
  camera = new THREE.PerspectiveCamera(fov, width / height, 1, 30000);
  // 设置相机位置
  camera.position.set(0, 0, zAxisNumber);
  // 看向原点
  camera.lookAt(0, 0, 0);
  // console.log(camera);
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
  renderStarMoving();
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
