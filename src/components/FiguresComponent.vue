<script setup lang="ts">
  import * as THREE from "three";

  const scene = new THREE.Scene();
  const camera = new THREE.PerspectiveCamera(
      75,
      window.innerWidth / window.innerHeight,
      0.1,
      1000
  );
  scene.add(camera);
</script>

<template>

</template>

<style scoped>

</style>
