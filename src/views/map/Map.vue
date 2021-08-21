/* eslint-disable prettier/prettier */
<template>
  <div class="map">
    <div id="content"></div>
  </div>
</template>

<script>
import * as THREE from "three";
import { MapControls } from "three/examples/jsm/controls/OrbitControls";
import * as GEOLIB from "geolib";
// const api =
//   "https://gistcdn.githack.com/isjeffcom/a611e99aa888534f67cc2f6273a8d594/raw/9dbb086197c344c860217826c59d8a70d33dcb54/gistfile1.txt";
export default {
  name: "Map",
  data() {
    return {
      scene: null,
      camera: null,
      renderer: null,
      controls: null,
      center: [113.6786766, 34.7706888],
      // center: [-3.188822, 55.943686],

      MAT_BUILDING: null,
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

      // let geometry = new THREE.BoxGeometry(1, 1, 1);
      // let material = new THREE.MeshBasicMaterial({ color: 0x00ff00 });
      // let cube = new THREE.Mesh(geometry, material);
      // this.scene.add(cube);

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
      this.MAT_BUILDING = new THREE.MeshPhongMaterial(); // MeshPhongMaterial 白模反光材质

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

      // fetch(api).then((res) => {
      //   res.json().then((data) => {
      //     this.LoadBuilding(data);
      //   });
      // });
    },

    LoadBuilding(data) {
      let features = data.features;

      for (let i = 0; i < features.length; i++) {
        let fel = features[i];
        if (!fel["properties"]) return;

        if (fel.properties["building"]) {
          this.addBuilding(
            fel.geometry.coordinates,
            fel.properties,
            fel.properties["building:levels"]
          );
        }
      }
    },

    addBuilding(data, info, height = 1) {
      // data 点阵数据；info 建筑数据； height 高度
      // console.log(data, info, height);
      height = height ? height : 1;

      for (let i = 0; i < data.length; i++) {
        let el = data[i];

        let shape = this.genShape(el, this.center);
        let geometry = this.genGeometry(shape, {
          curveSegments: 1,
          depth: 0.05 * height,
          bevelEnabled: false,
        });

        geometry.rotateX(Math.PI / 2);
        geometry.rotateZ(Math.PI);

        let mesh = new THREE.Mesh(geometry, this.MAT_BUILDING);
        this.scene.add(mesh);
      }
    },
    genShape(points, center) {
      let shape = new THREE.Shape();

      for (let i = 0; i < points.length; i++) {
        let elp = points[i];
        elp = this.GPSRelativePosition(elp, center);

        if (i == 0) {
          shape.moveTo(elp[0], elp[1]);
        } else {
          shape.lineTo(elp[0], elp[1]);
        }
      }
      return shape;
    },
    genGeometry(shape, config) {
      let geometry = new THREE.ExtrudeBufferGeometry(shape, config);
      geometry.computeBoundingBox();

      return geometry;
    },
    GPSRelativePosition(objPosi, centerPosi) {
      // Get GPS distance
      let dis = GEOLIB.getDistance(objPosi, centerPosi);

      // Get bearing angle
      let bearing = GEOLIB.getRhumbLineBearing(objPosi, centerPosi);

      // Calculate X by centerPosi.x + distance * cos(rad)
      let x = centerPosi[0] + dis * Math.cos((bearing * Math.PI) / 180);

      // Calculate Y by centerPosi.y + distance * sin(rad)
      let y = centerPosi[1] + dis * Math.sin((bearing * Math.PI) / 180);

      // Reverse X (it work)
      return [-x / 100, y / 100];
    },
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
