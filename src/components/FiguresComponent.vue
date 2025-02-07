<script setup lang="ts">
  import * as THREE from "three";
  import {onMounted, ref} from "vue";

  const element = ref<HTMLCanvasElement | null>(null);
  const scene = new THREE.Scene();
  const camera = new THREE.PerspectiveCamera(
      75,
      window.innerWidth / window.innerHeight,
      0.1,
      1000
  );
  camera.position.z = 5;
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

  const group = new THREE.Group();
  group.add(sphere, cube);
  scene.add(group);

  onMounted(() => {
    const renderer = new THREE.WebGLRenderer({
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
