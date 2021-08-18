<template>
  <div class="map">
    <div id="content"></div>
  </div>
</template>

<script>
import * as THREE from "three";
import { MapControls } from "three/examples/jsm/controls/OrbitControls";
export default {
  name: "Map",
  data() {
    return {
      scene: null,
      camera: null,
      renderer: null,
      controls: null,
    };
  },
  methods: {
    Awake() {
      let cont = document.getElementById("content");

      // 初始化场景
      this.scene = new THREE.Scene();
      this.scene.background = new THREE.Color(0x222222);

      // 初始化摄像机
      this.camera = new THREE.PerspectiveCamera(
        45,
        window.clientWidth / window.clientHeight,
        1,
        100
      );
      this.camera.position.set(8, 4, 0);

      // 初始化灯光
      let light0 = new THREE.AmbientLight(0xfafafa, 0.25);
      let light1 = new THREE.PointLight(0xfafafa, 0.4);
      light1.position.set(200, 90, 40);
      let light2 = new THREE.PointLight(0xfafafa, 0.4);
      light2.position.set(200, 90, -40);

      this.scene.add(light0);
      this.scene.add(light1);
      this.scene.add(light2);

      let gh = new THREE.GridHelper(
        60,
        160,
        new THREE.Color(0x555555),
        new THREE.Color(0x333333)
      );
      this.scene.add(gh);

      let geometry = new THREE.BoxGeometry(1, 1, 1);
      let material = new THREE.MeshBasicMaterial({ color: 0x00ff00 });
      let cube = new THREE.Mesh(geometry, material);
      this.scene.add(cube);

      // 初始化渲染器
      this.renderer = new THREE.WebGLRenderer({ antialias: true });
      this.renderer.setPixelRatio(window.devicePixelRatio);
      this.renderer.setSize(window.innerWidth, window.innerHeight);

      cont.appendChild(this.renderer.domElement);

      // 初始化控制器
      this.controls = new MapControls(this.camera, this.renderer.domElement);
      this.controls.enableDamping = true;
      this.controls.dampingFactor = 0.25;
      this.controls.screenSpacePanning = false;
      this.controls.maxDistance = 800;

      this.controls.update();

      this.Update();

      this.GetGeoJson();
    },
    Update() {
      requestAnimationFrame(this.Update);
      this.renderer.render(this.scene, this.camera);
      this.controls.update();
    },

    GetGeoJson() {
      fetch("./assets/zhengzhou.geojson").then((res) => {
        res.json().then((data) => {
          this.LoadBuilding(data);
        });
      });
    },

    LoadBuilding(data) {
      let features = data.features;
      for (let i = 0; i < features.length; i++) {
        let fel = features[i];
        if (!fel["properties"]) return;
        if (fel.properties["building"]) {
          this.addBuilding();
        }
      }
    },

    addBuilding() {
      
    }
  },
  mounted() {
    this.Awake();
    let that = this;
    window.addEventListener("resize", onWindowResize, false);
    function onWindowResize() {
      that.camera.aspect = window.innerWidth / window.innerHeight;
      that.camera.updateProjectionMatrix();
      that.renderer.setSize(window.innerWidth, window.innerHeight);
    }
    onWindowResize();
  },
};
</script>

<style>
.map,
#content {
  width: 100%;
  height: 100%;
}
</style>
