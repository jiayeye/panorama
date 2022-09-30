<template>
  <div>
    <div id="textDiv">
      <text id="errorText" v-if="showErrorInfo">加载失败，请检查资源</text>
    </div>
    <canvas id="threeCanvas" ref="threeCanvas"></canvas>
    <progress
      id="progress"
      v-if="showProgress"
      :value="progressValue"
      max="100"
    ></progress>
  </div>
</template>

<script>
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";
import { FBXLoader } from "three/examples/jsm/loaders/FBXLoader";

let camera, scene, renderer, controls, tickId, scale;

export default {
  props: {
    modelUrl: {
      type: String,
      required: true,
    },
  },
  data() {
    return {
      showErrorInfo: false,
      showProgress: false,
      progressValue: 0,
    };
  },
  mounted() {
    this.initScene(this.modelUrl);
  },
  methods: {
    initScene(modelUrl) {
      scale = 1;
      // reset
      this.destroy();

      // 相机far
      const cameraMaxDistance = 12000;

      // 添加透视相机，设置fov、初始位置
      camera = new THREE.PerspectiveCamera(
        45,
        window.innerWidth / window.innerHeight,
        30,
        cameraMaxDistance
      );
      
      camera.position.set(0, 0, 100);

      // 添加场景及颜色
      scene = new THREE.Scene();
      scene.background = new THREE.Color(0xffffff);

      // 添加光源
      const hemiLight = new THREE.HemisphereLight(0xffffff, 0x000000);
      hemiLight.position.set(0, 2000, 0);
      scene.add(hemiLight);

      // 设置直射光
      const dirLight = new THREE.DirectionalLight(0xdddddd);
      dirLight.castShadow = false;
      dirLight.intensity = 0.6;
      scene.add(dirLight);

      // 添加WebGLRenderer，设置size
      renderer = new THREE.WebGLRenderer({
        canvas: this.$refs.threeCanvas,
        antialias: true,
      });
      renderer.setPixelRatio(window.devicePixelRatio);
      renderer.setSize(window.innerWidth * scale, window.innerHeight * scale);
      renderer.shadowMap.enabled = false;

      // 添加相机控制器
      controls = new OrbitControls(camera, renderer.domElement);
      controls.autoRotate = true;
      controls.autoRotateSpeed = 1.5;
      controls.enablePan = false;
      controls.target.set(0, 0, 0);

      // 加载进度manager
      const manager = new THREE.LoadingManager();
      const loader = new FBXLoader(manager);
      // 除湿机
      loader.load(
        'https://syn-yf-design-tool.oss-cn-beijing.aliyuncs.com/save_fbx/AJ0261M07-模型.fbx',
        (object) => {
          object.traverse((child) => {
            if (child.isMesh) {
              child.castShadow = false;
              child.receiveShadow = false;
            }
          });
          // 设置object position
          object.position.set(0, -900, -2000);

          // 添加object到场景里
          scene.add(object);
        }
      );

      // 体脂秤
      loader.load(
        'https://syn-yf-design-tool.oss-cn-beijing.aliyuncs.com/save_fbx/JM03F400Y-模型.fbx',
        (object) => {
          object.traverse((child) => {
            if (child.isMesh) {
              child.castShadow = false;
              child.receiveShadow = false;
            }
          });
          // 设置object position
          object.position.set(700, -930, -2300);

          // 添加object到场景里
          scene.add(object);
        }
      );

        // 电视
        loader.load(
        'https://syn-yf-design-tool.oss-cn-beijing.aliyuncs.com/save_fbx/DH1UN0A02-模型1.fbx',
        (object) => {
          object.traverse((child) => {
            if (child.isMesh) {
              child.castShadow = false;
              child.receiveShadow = false;
            }
          });
          // 设置object position
          object.position.set(1760, 95, -1790);

          // 添加object到场景里
          scene.add(object);
        }
      );

      // px = right
      // nx = left
      // py = top
      // ny = bottom
      // pz = front
      // nz = back
      const textures = [];
      const px = new THREE.TextureLoader().load("https://syn-yf-design-tool.oss-cn-beijing.aliyuncs.com/panorama/px.jpg");
      const nx = new THREE.TextureLoader().load("https://syn-yf-design-tool.oss-cn-beijing.aliyuncs.com/panorama/nx.jpg");
      const py = new THREE.TextureLoader().load("https://syn-yf-design-tool.oss-cn-beijing.aliyuncs.com/panorama/py.jpg");
      const ny = new THREE.TextureLoader().load("https://syn-yf-design-tool.oss-cn-beijing.aliyuncs.com/panorama/ny.jpg");
      const pz = new THREE.TextureLoader().load("https://syn-yf-design-tool.oss-cn-beijing.aliyuncs.com/panorama/pz.jpg");
      const nz = new THREE.TextureLoader().load("https://syn-yf-design-tool.oss-cn-beijing.aliyuncs.com/panorama/nz.jpg");
      textures.push(px);
      textures.push(nx);
      textures.push(py);
      textures.push(ny);
      textures.push(pz);
      textures.push(nz);
      const materials = [];
      for (let i = 0; i < 6; i++) {
        materials.push(new THREE.MeshBasicMaterial({ map: textures[i] }));
      }
      const skyBox = new THREE.Mesh(new THREE.BoxGeometry(5000, 5000, 5000), materials);
      skyBox.geometry.scale(1, 1, -1);
      scene.add(skyBox);

      // 监听窗口reszie事件
      window.addEventListener("resize", this.onWindowResize);
      // 监听窗口reszie事件
      window.addEventListener("mousedown", this.onMouseDown);
      // 设置tick
      this.animate();
    },

    // 加载进度
    onProgress(xhr) {
      if (xhr.lengthComputable) {
        const percentComplete = (xhr.loaded / xhr.total) * 100;
        this.progressValue = Math.round(percentComplete, 2);
        // console.log("model " + Math.round(percentComplete, 2) + "% downloaded");
      }
    },

    // 加载失败回掉
    onError(error) {
      console.log(error.message);
      this.showProgress = false;
      this.$refs.threeCanvas.hidden = true;
      this.showErrorInfo = true;
    },

    // 重置窗口大小
    onWindowResize() {
      camera.aspect = window.innerWidth / window.innerHeight;
      // 更新相机投影矩阵
      camera.updateProjectionMatrix();
      renderer.setSize(window.innerWidth * scale, window.innerHeight * scale);
    },

    // 鼠标点击事件
    onMouseDown() {
      controls.autoRotate = false;
    },

    // 每帧调用
    animate() {
      // 获取callback handler
      tickId = requestAnimationFrame(this.animate);

      // 更新control状态
      controls.update();
      // 每帧渲染
      renderer.render(scene, camera);
    },
    // 清空场景
    destroy() {
      // 使用handler取消每帧调用
      cancelAnimationFrame(tickId);
      // 移除resize监听
      window.removeEventListener("resize", this.onWindowResize);
      // 移除mouseDown监听
      window.removeEventListener("mousedown", this.onMouseDown);
      if (renderer) {
        renderer.domElement.addEventListener("dblclick", null, false); //remove listener to render
        renderer.forceContextLoss();
      }
      renderer = null;
      scene = null;
      camera = null;
      controls = null;
    },
  },

  beforeDestroy() {
    console.log("beforeDestroy");
    // 清除场景
    this.destroy();
  },
};
</script>

<style scoped>
#three {
  width: 100%;
  height: 100%;
  position: fixed;
  left: 0;
  top: 0;
}

#progress {
  width: 40%;
  height: 2rem;
  position: fixed;
  left: 30%;
  top: 47%;
}

#textDiv {
  position: fixed;
  width: 100%;
  height: 40px;
  line-height: 40px;
  text-align: center;
  top: 46%;
}
</style>
