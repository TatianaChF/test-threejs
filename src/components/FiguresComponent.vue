<template>
  <div class="container">
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
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount } from 'vue';
import * as THREE from 'three';

type SceneObjects = {
  scene: THREE.Scene | null;
  camera: THREE.PerspectiveCamera | null;
  renderer: THREE.WebGLRenderer | null;
  door: THREE.Mesh | null;
}

const element = ref<HTMLElement | null>(null);
const doorHeight = ref<number>(5);
const doorWidth = ref<number>(3);
const objects: SceneObjects = {
  scene: null,
  camera: null,
  renderer: null,
  door: null
};

const addScene = (): void => {
  objects.scene = new THREE.Scene();
  objects.camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
  objects.renderer = new THREE.WebGLRenderer({ antialias: true });

  objects.scene.background = new THREE.Color("skyblue");

  if (objects.renderer && element.value) {
    objects.renderer.setSize(window.innerWidth, window.innerHeight);
    element.value.appendChild(objects.renderer.domElement);
  }

  if (objects.camera) {
    objects.camera.position.z = 5;
  }

  const textureLoader = new THREE.TextureLoader();
  textureLoader.load('/wood.png', (texture: THREE.Texture) => {
    addDoor(texture);
    addFigures();
    animate();
  });
};

const addDoor = (texture: THREE.Texture): void => {
  if (!objects.scene) return;

  const geometry = new THREE.BoxGeometry(
      doorWidth.value,
      doorHeight.value,
      0.1
  );
  const material = new THREE.MeshBasicMaterial({ map: texture });
  objects.door = new THREE.Mesh(geometry, material);
  objects.door.position.set(3, 0, 0);
  objects.scene.add(objects.door);
};

const addFigures = (): void => {
  if (!objects.scene) return;

  const sphere = new THREE.Mesh(
      new THREE.SphereGeometry(1, 32, 32),
      new THREE.MeshBasicMaterial({ color: 0xff0000 })
  );
  sphere.position.set(-2, 0, 0);

  const cube = new THREE.Mesh(
      new THREE.BoxGeometry(1, 1, 1),
      new THREE.MeshBasicMaterial({ color: 0x800080 })
  );

  objects.scene.add(sphere, cube);
};

const updateDoor = (): void => {
  if (!objects.door || !objects.scene) return;

  objects.door.geometry.dispose();
  objects.scene.remove(objects.door);

  const newGeometry = new THREE.BoxGeometry(
      doorWidth.value,
      doorHeight.value,
      0.1
  );
  const textureLoader = new THREE.TextureLoader();
  textureLoader.load('/wood.png', (texture: THREE.Texture) => {
    objects.door = new THREE.Mesh(
        newGeometry,
        new THREE.MeshBasicMaterial({ map: texture })
    );
    objects.door.position.set(3, 0, 0);
    objects.scene?.add(objects.door);
  });
};

const animate = (): void => {
  requestAnimationFrame(animate);

  if (objects.renderer && objects.scene && objects.camera) {
    objects.renderer.render(objects.scene, objects.camera);
  }
};

onMounted(() => {
  addScene();
});

onBeforeUnmount(() => {
  if (objects.renderer) {
    objects.renderer.dispose();
  }
  if (objects.door) {
    objects.door.geometry.dispose();
    if (Array.isArray(objects.door.material)) {
      objects.door.material.forEach(m => m.dispose());
    } else {
      (objects.door.material as THREE.Material).dispose();
    }
  }
});
</script>

<style scoped>
.container {
  position: relative;
  height: 100vh;
}
</style>