<script setup lang="ts">
  import * as THREE from "three";
  import {ref} from "vue";

  const element = ref<HTMLCanvasElement | null>(null);
  const scene = new THREE.Scene();
  const camera = new THREE.PerspectiveCamera(
      75,
      window.innerWidth / window.innerHeight,
      0.1,
      1000
  );
  scene.add(camera);

  const renderer = new THREE.WebGLRenderer({
    canvas: element.value as unknown as HTMLCanvasElement,
    antialias: true
  });
  renderer.setSize(window.innerWidth, window.innerHeight);
  renderer.render(scene, camera);
</script>

<template>
  <canvas ref="element"/>
</template>

<style scoped>

</style>
