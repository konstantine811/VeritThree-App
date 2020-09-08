<template>
  <div>
    <div class="scene" ref="scene" @keydown="keyDown($event)"></div>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
import { TransformControls } from "three/examples/jsm/controls/TransformControls";
import { DragControls } from "three/examples/jsm/controls/DragControls";

@Component
export default class ThreeCanvas extends Vue {
  private camera!: THREE.PerspectiveCamera;
  private renderer: THREE.WebGLRenderer = new THREE.WebGLRenderer();
  private scene: THREE.Scene = new THREE.Scene();
  private orbit!: OrbitControls;
  private control!: TransformControls;
  private raycaster!: THREE.Raycaster;
  private mouse = new THREE.Vector2();
  private clientWidth!: number;
  private clientHeight!: number;
  mounted() {
    const el = this.$refs.scene as Element;

    this.clientWidth = el.clientWidth;
    this.clientHeight = el.clientHeight;
    this.camera = new THREE.PerspectiveCamera(
      50,
      el.clientWidth / el.clientHeight,
      0.01,
      30000
    );
    this.camera.position.set(1000, 500, 1000);
    this.camera.lookAt(0, 200, 0);

    this.renderer.setSize(el.clientWidth, el.clientHeight);
    this.renderer.setClearColor(0xffffff, 1);
    el.appendChild(this.renderer.domElement);

    this.scene.add(new THREE.GridHelper(1000, 10));

    this.orbit = new OrbitControls(this.camera, this.renderer.domElement);
    this.orbit.update();

    this.control = new TransformControls(this.camera, this.renderer.domElement);

    const geometry = new THREE.BoxGeometry(200, 200, 200);
    const material = new THREE.MeshBasicMaterial({ color: 0xcc0f00 });
    const cube = new THREE.Mesh(geometry, material);

    this.scene.add(cube);

    const geometry2 = new THREE.BoxGeometry(200, 200, 200);
    const material2 = new THREE.MeshBasicMaterial({ color: 0x00ff00 });
    const cube2 = new THREE.Mesh(geometry2, material2);
    cube2.position.x = 100;
    this.scene.add(cube2);

    this.scene.add(this.control);

    this.control.addEventListener("dragging-changed", (event) => {
      this.orbit.enabled = !event.value;
    });

    const dragControls = new DragControls(
      [cube, cube2],
      this.camera,
      this.renderer.domElement
    );
    dragControls.enabled = true;
    dragControls.addEventListener("hoveron", (event) => {
      this.control.attach(event.object);
    });

    const animate = () => {
      requestAnimationFrame(animate);
      this.renderer.render(this.scene, this.camera);
    };
    animate();
  }

  keyDown(e: KeyboardEvent) {
    switch (e.code) {
      case "KeyR":
        this.control.setMode("rotate");
        break;
      case "KeyG":
        this.control.setMode("translate");
        break;
      case "KeyS":
        this.control.setMode("scale");
        break;
    }
  }
}
</script>

<style lang="scss" scoped>
.scene {
  height: calc(100vh - 90px);
  width: 100%;
  border-radius: 22px;
  border: 1px solid #747474;
  overflow: hidden;
}
</style>
