<script setup lang="ts">
import * as THREE from "three";
import {onMounted, ref} from "vue";
import woodUrl from '/wood.png';

const element = ref<HTMLCanvasElement | null>(null);
const doorHeight = ref<number>(5);
const doorWidth = ref<number>(3);
let scene;
let camera;
let renderer = new THREE.WebGLRenderer();
let door: THREE.Mesh<THREE.BoxGeometry, THREE.MeshBasicMaterial, THREE.Object3DEventMap>;
scene = new THREE.Scene();
camera = new THREE.PerspectiveCamera(
    75,
    window.innerWidth / window.innerHeight,
    0.1,
    1000
);
camera.position.z = 5;
scene.background = new THREE.Color("skyblue");
scene.add(camera);

const sphere = new THREE.Mesh(
    new THREE.SphereGeometry(1, 20, 20),
    new THREE.MeshBasicMaterial({color: "red"}),
);
sphere.position.set(-2,0,0);

const cube = new THREE.Mesh(
    new THREE.BoxGeometry(1,1,1,),
    new THREE.MeshBasicMaterial({color: "purple"}),
);
cube.position.set(0,0,0);

new THREE.TextureLoader().load(
    woodUrl,
    function (texture) {
      const material = new THREE.MeshBasicMaterial({ map: texture });
      door = new THREE.Mesh(
          new THREE.BoxGeometry(doorWidth.value,doorHeight.value,0.3,),
          material
      )
      door.position.set(3,0,0);
      scene.add(door)
      renderer.render(scene, camera);
    }
);

const updateDoor = () => {
  door.geometry.dispose();
  scene.remove(door);

  const newDoorGeometry = new THREE.BoxGeometry(doorWidth.value, doorHeight.value, 0.3);
  door = new THREE.Mesh(newDoorGeometry, door.material);
  door.position.set(3, 0, 0);
  scene.add(door);
}

onMounted(() => {
  renderer = new THREE.WebGLRenderer({
    canvas: element.value as unknown as HTMLCanvasElement,
    antialias: true
  });
  renderer.setSize(window.innerWidth, window.innerHeight);
  renderer.render(scene, camera);
})
</script>

<template>
  <div>
    <input
        v-model="doorHeight"
        @input="updateDoor"
        type="range"
        min="1"
        max="10"
        step="0.1"/>
    <input
        v-model="doorWidth"
        @input="updateDoor"
        type="range"
        min="1"
        max="10"
        step="0.1"/>
  </div>
  <canvas ref="element"/>
</template>

<style scoped>

</style>