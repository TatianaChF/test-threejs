<template>
  <div class="container">
    <div class="controls">
      <div class="control-group">
        <label>Высота двери</label>
        <input
            type="range"
            v-model="doorHeight"
            min="1"
            max="10"
            step="0.1"
            @input="updateDoor"
        >
      </div>
      <div class="control-group">
        <label>Ширина двери</label>
        <input
            type="range"
            v-model="doorWidth"
            min="1"
            max="5"
            step="0.1"
            @input="updateDoor"
        >
      </div>
    </div>
    <div ref="element"></div>
  </div>
</template>

<script setup lang="ts">
import {ref, onMounted, onBeforeUnmount, type Ref, computed} from 'vue';
import * as THREE from 'three';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';
import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js';

type SceneObjects = {
  scene: THREE.Scene | null;
  camera: THREE.PerspectiveCamera | null;
  renderer: THREE.WebGLRenderer | null;
  door: THREE.Mesh | null;
  controls: OrbitControls | null;
  light: THREE.PointLight | null;
  loader: GLTFLoader | null;
}

const element: Ref<HTMLElement | null> = ref(null);
const doorHeight = ref<number>(5);
const doorWidth = ref<number>(3);
const objects: SceneObjects = {
  scene: null,
  camera: null,
  renderer: null,
  door: null,
  controls: null,
  light: null,
  loader: null
};
let clock = new THREE.Clock();

const addScene = async () => {
  objects.loader = new GLTFLoader();
  objects.scene = new THREE.Scene();

  if (objects.loader && objects.scene) {
    const glt = await objects.loader.loadAsync("/background.glb");
    glt.scene.scale.set(1.3, 1.3, 1.3);
    objects.scene.add(glt.scene);
  }

  objects.camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
  objects.renderer = new THREE.WebGLRenderer({ antialias: true });

  objects.scene.background = new THREE.Color("white");

  if (objects.renderer && element.value) {
    objects.renderer.setSize(window.innerWidth, window.innerHeight);
    element.value.appendChild(objects.renderer.domElement);

    objects.renderer.setAnimationLoop(renderCamera);
  }

  if (objects.camera) {
    objects.camera.position.z = 10;
  }

  if (objects.camera && objects.renderer) {
    objects.controls = new OrbitControls(objects.camera, objects.renderer.domElement);
    configureControls();
  }

  const textureLoader = new THREE.TextureLoader();
  textureLoader.load('/wood.png', (texture: THREE.Texture) => {
    addDoor(texture);
    addFigures();
    animate();
  });

  objects.light = new THREE.PointLight( 0xfffff, 100, 1000 );
  objects.light.position.set( 0, 10, 4);
  objects.light.castShadow = true;
  objects.scene.add( objects.light );

  objects.light.shadow.mapSize.width = 512;
  objects.light.shadow.mapSize.height = 512;
  objects.light.shadow.camera.near = 0.5;
  objects.light.shadow.camera.far = 500;

};

const addDoor = (texture: THREE.Texture): void => {
  if (!objects.scene) return;

  const geometry = new THREE.BoxGeometry(
      doorWidth.value,
      doorHeight.value,
      0.3
  );
  const material = new THREE.MeshBasicMaterial({ map: texture });
  objects.door = new THREE.Mesh(geometry, material);
  objects.door.position.set(3, 2, 0);
  objects.scene.add(objects.door);
};

const addFigures = (): void => {
  if (!objects.scene) return;

  const sphere = new THREE.Mesh(
      new THREE.SphereGeometry(1, 20, 20),
      new THREE.MeshBasicMaterial({ color: "red" })
  );
  sphere.castShadow = true;
  sphere.receiveShadow = false;
  sphere.position.set(-2, 1, 0);

  const cube = new THREE.Mesh(
      new THREE.BoxGeometry(1, 1, 1),
      new THREE.MeshBasicMaterial({ color: "purple" })
  );
  cube.castShadow = true;
  cube.receiveShadow = false;
  cube.position.set(0,1,0);

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
    objects.door.position.set(3, 3, 0);
    objects.door.castShadow = true;
    objects.door.receiveShadow = false;
    objects.scene?.add(objects.door);
  });
};

const configureControls = (): void => {
  if (!objects.controls) return;

  objects.controls.enableDamping = true;
  objects.controls.dampingFactor = 0.05;
  objects.controls.rotateSpeed = 0.5;
  objects.controls.minDistance = 5;
  objects.controls.maxDistance = 15;
  objects.controls.enableZoom = true;
};

const animate = (): void => {
  requestAnimationFrame(animate);

  if (objects.controls) {
    objects.controls.update();
  }

  if (objects.renderer && objects.scene && objects.camera) {
    objects.renderer.render(objects.scene, objects.camera);
  }
};

const renderCamera = () => {
  const delta = clock.getDelta();

  objects.controls.update( delta );
}

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

.controls {
  position: absolute;
  top: 20px;
  left: 20px;
  z-index: 100;
  background: rgba(255, 255, 255, 0.9);
  padding: 15px;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

.control-group {
  margin-bottom: 15px;
}

.control-group:last-child {
  margin-bottom: 0;
}

input[type="range"] {
  margin-top: 8px;
  width: 200px;
  display: block;
}

label {
  display: block;
  margin-bottom: 5px;
  font-family: Arial, sans-serif;
  color: #333;
  font-size: 14px;
}
</style>