<template>
  <div class="container">
    <div class="controls">
      <div class="control-group">
        <label>Высота двери</label>
        <input
            v-model="doorHeight"
            max="10"
            min="1"
            step="0.1"
            type="range"
            @input="updateDoor"
        >
      </div>
      <div class="control-group">
        <label>Ширина двери</label>
        <input
            v-model="doorWidth"
            max="5"
            min="1"
            step="0.1"
            type="range"
            @input="updateDoor"
        >
      </div>
    </div>
    <div ref="element"></div>
  </div>
</template>

<script lang="ts" setup>
import {onBeforeUnmount, onMounted, ref, type Ref} from 'vue';
import * as THREE from 'three';
import {OrbitControls} from 'three/examples/jsm/controls/OrbitControls.js';
import {GLTFLoader} from 'three/addons/loaders/GLTFLoader.js';
import {RGBELoader} from 'three/examples/jsm/loaders/RGBELoader.js';

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

let canvasWidth = window.innerWidth;
let canvasHeight = window.innerHeight;

const onWindowResize = () => {
  canvasHeight = window.innerHeight;
  canvasWidth = window.innerWidth;

  if (objects.camera) {
    objects.camera.aspect = canvasWidth / canvasHeight;
    objects.camera.updateProjectionMatrix();
  }

  if (objects.renderer) {
    objects.renderer.setSize(canvasWidth, canvasHeight);
  }
}

const addScene = async () => {
  objects.scene = new THREE.Scene();
  objects.camera = new THREE.PerspectiveCamera(75, canvasWidth / canvasHeight, 0.1, 1000);
  objects.renderer = new THREE.WebGLRenderer({
    antialias: true,
    powerPreference: "high-performance"
  });

  const hdrLoader = new RGBELoader();
  const hdrTexture = await hdrLoader.loadAsync(`${window.location.pathname}/background.hdr`);
  hdrTexture.mapping = THREE.EquirectangularReflectionMapping;

  const floorTexture = new THREE.TextureLoader().load(`${window.location.pathname}/floor.jpg`);
  floorTexture.wrapS = THREE.RepeatWrapping;
  floorTexture.wrapT = THREE.RepeatWrapping;
  floorTexture.repeat.set(20, 20);

  const floorMaterial = new THREE.MeshStandardMaterial({
    map: floorTexture,
    roughness: 0.8,
    metalness: 0.2
  });

  const floor = new THREE.Mesh(
      new THREE.CircleGeometry(50, 64),
      floorMaterial
  );
  floor.rotation.x = -Math.PI / 2;
  floor.receiveShadow = true;
  objects.scene.add(floor);

  const ambientLight = new THREE.AmbientLight(0xffffff, 0.5);
  objects.scene.add(ambientLight);

  const directionalLight = new THREE.DirectionalLight(0xffffff, 1);
  directionalLight.position.set(0, 5, 5);
  directionalLight.castShadow = true;
  directionalLight.shadow.mapSize.width = 4096;
  directionalLight.shadow.mapSize.height = 4096;
  directionalLight.shadow.camera.near = 0.5;
  directionalLight.shadow.camera.far = 50;
  directionalLight.shadow.camera.left = -20;
  directionalLight.shadow.camera.right = 20;
  directionalLight.shadow.camera.top = 20;
  directionalLight.shadow.camera.bottom = -20;
  objects.scene.add(directionalLight);

  objects.scene.environment = hdrTexture;
  objects.scene.background = hdrTexture;

  if (objects.renderer && element.value) {
    objects.renderer.setSize(canvasWidth, canvasHeight);
    objects.renderer.shadowMap.enabled = true;
    objects.renderer.shadowMap.type = THREE.PCFSoftShadowMap;
    element.value.appendChild(objects.renderer.domElement);

    objects.renderer.setAnimationLoop(renderCamera);
  }

  if (objects.camera) {
    objects.camera.position.z = 10;
    objects.camera.position.y = 10;
  }

  if (objects.camera && objects.renderer) {
    objects.controls = new OrbitControls(objects.camera, objects.renderer.domElement);
    configureControls();
  }

  const textureLoader = new THREE.TextureLoader();
  textureLoader.load(`${window.location.pathname}/wood.png`, (texture: THREE.Texture) => {
    addDoor(texture);
    addFigures();
    animate();
  });

  window.addEventListener('resize', onWindowResize)
};

const addDoor = (texture: THREE.Texture): void => {
  if (!objects.scene) return;

  const geometry = new THREE.BoxGeometry(
      doorWidth.value,
      doorHeight.value,
      0.3
  );
  const material = new THREE.MeshBasicMaterial({map: texture});
  objects.door = new THREE.Mesh(geometry, material);
  objects.door.position.set(3, 3, 0);
  objects.door.castShadow = true;
  objects.door.receiveShadow = true;
  objects.scene.add(objects.door);
};

const addFigures = (): void => {
  if (!objects.scene) return;

  const sphere = new THREE.Mesh(
      new THREE.SphereGeometry(1, 20, 20),
      new THREE.MeshBasicMaterial({color: "red"})
  );
  sphere.castShadow = true;
  sphere.receiveShadow = false;
  sphere.position.set(-2, 3, 0);
  sphere.castShadow = true;
  sphere.receiveShadow = true;

  const cube = new THREE.Mesh(
      new THREE.BoxGeometry(1, 1, 1),
      new THREE.MeshBasicMaterial({color: "purple"})
  );
  cube.castShadow = true;
  cube.receiveShadow = false;
  cube.position.set(0, 3, 0);
  cube.castShadow = true;
  cube.receiveShadow = true;


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
  textureLoader.load(`${window.location.pathname}/wood.png`, (texture: THREE.Texture) => {
    objects.door = new THREE.Mesh(
        newGeometry,
        new THREE.MeshBasicMaterial({map: texture})
    );
    objects.door.position.set(3, -2, 0);
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

  objects.controls?.update(delta);
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
