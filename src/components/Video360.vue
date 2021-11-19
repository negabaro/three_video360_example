<template>
  <div>
    <div>
      <label for="videourl">URL</label>
      <input v-model="videoUrl" id="videourl" width="500px" /><button
        @click="changeURL()"
      >
        Change video
      </button>
      <button @click="playVideo()" v-bind:disabled="isplaying">Play</button>
      <button @click="pauseVideo()" v-bind:disabled="!isplaying">Pause</button>
      <button @click="stopVideo()">Stop</button>
    </div>
    <div id="video360player"></div>
  </div>
</template>

<script>
import * as THREE from "three";
import * as controls from "three-orbit-controls";

export default {
  name: "video360",
  data() {
    const width = 600; //window.innerWidth,
    const height = 400; // window.innerHeight;
    const scene = new THREE.Scene();
    const geometry = new THREE.SphereGeometry(5, 60, 40);
    const video360 = document.createElement("video");
    video360.id = "video360";
    const texture = new THREE.VideoTexture(video360);
    const material = new THREE.MeshBasicMaterial({ map: texture });
    const sphere = null;
    const camera = new THREE.PerspectiveCamera(75, width / height, 1, 1000);
    const renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });

    return {
      width,
      height,
      scene,
      geometry,
      video360,
      texture,
      material,
      sphere,
      camera,
      renderer,

      videoUrl: require("./360_sample.mp4"),
      isplaying: false,
      currenttime: 0,
      controls: null,
    };
  },
  mounted() {
    (async () => {
      await this.initialize();
    })();
  },
  methods: {
    async initialize() {
      THREE.ImageUtils.crossOrigin = "anonymous";
      THREE.crossOrigin = "anonymous";
      // geometry
      this.geometry.scale(-100, 100, 100);
      // video
      this.video360.setAttribute("crossorigin", "anonymous");
      this.video360.width = this.width;
      this.video360.height = this.height;
      this.video360.src = this.videoUrl;
      this.video360.load();
      // texture
      this.texture.minFilter = THREE.LinearFilter;
      // sphere
      this.sphere = new THREE.Mesh(this.geometry, this.material);
      this.scene.add(this.sphere);
      // camera
      this.camera.position.set(0, 0, 0.1);
      this.camera.lookAt(this.sphere.position);
      // renderer
      this.renderer.setSize(this.width, this.height);
      this.renderer.setClearColor({ color: 0x000000 });

      // video player
      document
        .getElementById("video360player")
        .appendChild(this.renderer.domElement);
      this.renderer.render(this.scene, this.camera);
      // 3d mouse controls
      const OrbitControls = controls(THREE);
      this.controls = new OrbitControls(this.camera);

      this.render();
    },
    render() {
      requestAnimationFrame(this.render);
      this.renderer.render(this.scene, this.camera);
      this.controls.update();
      this.currenttime = this.video360.currentTime;
    },
    async playVideo() {
      if (!this.video360.src) {
        alert("video url not found");
        return;
      }
      if (this.video360.paused) {
        try {
          this.video360.play();
          this.isplaying = true;
        } catch (e) {
          console.log(e);
          alert("error occured.");
        }
      }
    },
    async pauseVideo() {
      this.video360.pause();
      this.isplaying = false;
    },
    async stopVideo() {
      this.video360.pause();
      this.isplaying = false;
      this.video360.currentTime = 0;
      this.controls.activate;
    },
    async changeURL() {
      console.log("change", this.videoUrl);
      this.video360.src = this.videoUrl;
      this.video360.load();
      this.stopVideo();
    },
  },
};
</script>
