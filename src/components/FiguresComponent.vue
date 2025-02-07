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
      new THREE.MeshBasicMaterial({color: 0x005423}),
  );
  scene.add(sphere);

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
