<script setup lang="ts">
import * as THREE from "three";
import {onMounted, ref} from "vue";
import woodUrl from '/wood.png';

const element = ref<HTMLCanvasElement | null>(null);
const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(
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

let renderer = new THREE.WebGLRenderer();

const group = new THREE.Group();
group.add(sphere, cube);
scene.add(group);

new THREE.TextureLoader().load(
    woodUrl,
    function (texture) {
      const material = new THREE.MeshBasicMaterial({ map: texture });
      const door = new THREE.Mesh(
          new THREE.BoxGeometry(3,5,0.3,),
          material
      )
      door.position.set(3,0,0);
      scene.add(cube);
      group.add(door);
      renderer.render(scene, camera);
    }
);

// const light = new THREE.DirectionalLight('white', 100);
// light.position.set(10, 10, 10);
// scene.add(light);

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
  <canvas ref="element"/>
</template>

<style scoped>

</style>