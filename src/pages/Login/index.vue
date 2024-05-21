<script setup>
import { reactive, ref, onMounted } from "vue";
import * as THREE from "three";
import { OrbitControls } from "three/addons/controls/OrbitControls.js";

// 容器
let container,
  // 场景
  scene,
  // 容器宽
  width,
  // 容器高
  height,
  // 容器深度
  depth,
  // 相机
  camera,
  // 渲染器
  renderer,
  // 轨道控制器
  controls,
  // mesh地球
  earth,
  // 相机位置
  zAxisNumber,
  // 星星初始位置
  starsPositionInit,
  //mesh星星
  meshPoints,
  // 星星参数
  parameters,
  // 星星进度
  starProgress,
  // mesh云1
  meshCloud1,
  // mesh云2
  meshCloud2,
  // 云1动画
  renderCloud1,
  // 云2动画
  renderCloud2;
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
const IMAGE_CLOUD = new URL("../../assets/images/cloud.png", import.meta.url)
  .href;

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
  starProgress = starsPositionInit;
  addSceneStar(starsPositionInit);
  meshCloud1 = addCloud(400, 200);
  meshCloud2 = addCloud(200, 100);
  renderCloud1 = initCloudMoving(
    meshCloud1,
    [
      new THREE.Vector3(-width / 10, 0, -depth / 2),
      new THREE.Vector3(-width / 4, height / 8, 0),
      new THREE.Vector3(-width / 4, 0, zAxisNumber),
    ],
    0.0002
  );
  renderCloud2 = initCloudMoving(
    meshCloud2,
    [
      new THREE.Vector3(width / 8, height / 8, -depth / 2),
      new THREE.Vector3(width / 8, height / 8, zAxisNumber),
    ],
    0.0008
  );
  initRender();
  // initOrbitControls();
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
  // 星星运动
  starProgress += 1;
  if (starProgress >= zAxisNumber + depth / 2) {
    starProgress = starsPositionInit;
  } else {
    meshPoints.position.setZ(starProgress);
  }
};

// 添加星云
const addCloud = (width, height) => {
  const geometry = new THREE.PlaneGeometry(width, height);
  const texture = new THREE.TextureLoader().load(IMAGE_CLOUD);
  const material = new THREE.MeshBasicMaterial({
    map: texture,
    blending: THREE.AdditiveBlending,
    transparent: true,
    depthTest: false,
  });
  const cloud = new THREE.Mesh(geometry, material);
  scene.add(cloud);
  return cloud;
};

// 初始化星云运动
const initCloudMoving = (meshCloud, points, speed) => {
  let cloudProgress = 0;
  const curve = new THREE.CatmullRomCurve3(points);
  return () => {
    if (cloudProgress <= 1) {
      cloudProgress += speed;
      // 获取当前位置
      const coord = curve.getPoint(cloudProgress);
      coord?.x && meshCloud.position.set(coord.x, coord.y, coord.z);
    } else {
      cloudProgress = 0;
    }
  };
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

  // controls.update();
  earthRotate();
  renderStarMoving();
  renderCloud1();
  renderCloud2();
  renderer.render(scene, camera);
};

// 登陆表单验证
const formSize = ref("default");
const labelPosition = ref("left");
const ruleFormRef = ref();
const ruleForm = reactive({
  name: "",
  password: "",
});

const rules = reactive({
  name: [
    { required: true, message: "Please input Activity name", trigger: "blur" },
    { min: 3, max: 5, message: "Length should be 3 to 5", trigger: "blur" },
  ],
  password: [
    { required: true, message: "Please input password", trigger: "blur" },
  ],
});

const submitForm = async (formEl) => {
  if (!formEl) return;
  await formEl.validate((valid, fields) => {
    if (valid) {
      console.log("submit!");
    } else {
      console.log("error submit!", fields);
    }
  });
};

const resetForm = (formEl) => {
  if (!formEl) return;
  formEl.resetFields();
};
</script>

<template>
  <div class="login-container">
    <div id="login-three-container"></div>
    <div class="login-ground"></div>
    <div class="login-plane">
      <div class="login-plane-container">
        <h1 class="login-plane-container-title">大数据智能应用系统</h1>
        <el-form
          ref="ruleFormRef"
          style="max-width: 300px"
          :label-position="labelPosition"
          :model="ruleForm"
          :rules="rules"
          label-width="auto"
          class="login-plane-container-from"
          :size="formSize"
          status-icon
        >
          <el-form-item label="用户名" prop="name">
            <el-input
              v-model="ruleForm.name"
              style="width: 240px"
              placeholder="Please input name"
              :input-style="{ color: red }"
            />
          </el-form-item>
          <el-form-item label="密码" prop="password">
            <el-input
              v-model="ruleForm.password"
              style="width: 240px"
              type="password"
              placeholder="Please input password"
              autocomplete="off"
              show-password
            />
          </el-form-item>
          <el-form-item class="login-plane-container-button">
            <el-button
              color="#3d4072"
              :dark="isDark"
              @click="submitForm(ruleFormRef)"
            >
              Create
            </el-button>
            <el-button
              color="#3d4072"
              :dark="isDark"
              @click="resetForm(ruleFormRef)"
              >Reset</el-button
            >
          </el-form-item>
        </el-form>
      </div>
      <img
        src="@/assets/images/login_human.png"
        alt="宇航员"
        class="login-plane-human"
      />
    </div>
  </div>
</template>

<style lang='less' scoped>
@keyframes upDownAnimation {
  0% {
    transform: translateY(0);
  }
  50% {
    transform: translateY(-15px); /* 上移10像素 */
  }
  100% {
    transform: translateY(0);
  }
}

.login-container {
  position: fixed;
  left: 0;
  top: 0;

  #login-three-container {
    width: 100vw;
    height: 100vh;
  }
  .login-ground {
    position: absolute;
    left: 0;
    bottom: 0;
    width: 100%;
    height: 50%;
    background-image: url("@/assets/images/ground.png");
  }
  .login-plane {
    // background-color: #794e4e;
    position: absolute;
    left: 50%;
    top: 50%;
    width: 600px;
    height: 300px;
    transform: translate(-50%, -50%);
    background-image: url("@/assets/images/login_border.png");
    background-size: 100% 100%;
    .login-plane-container {
      margin: 30px;
      // background-color: #fff;
      &-title {
        text-align: center;
        line-height: 70px;
        color: #fff;
        letter-spacing: 15px;
        background-image: url("@/assets/images/login_horizontal_line.png");
        /* 背景图片的位置，确保它在底部 */
        background-position: center bottom;
        /* 背景图片不重复 */
        background-repeat: no-repeat;
      }
      &-from {
        margin: 30px auto 0;
      }
    }
    &-human {
      position: absolute;
      right: -75px;
      top: -88px;
      width: 200px;
      animation: upDownAnimation 5s infinite;
    }
  }
}
</style>
<style lang='less'>
.login-container
  .login-plane
  .login-plane-container
  .login-plane-container-from {
  .el-form-item .el-form-item__label-wrap .el-form-item__label {
    color: #fff;
  }
  .login-plane-container-button .el-form-item__content {
    display: flex;
    justify-content: center;
    align-items: center;
  }
}
</style>
