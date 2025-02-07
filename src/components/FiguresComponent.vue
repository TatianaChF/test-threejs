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

<script setup lang="ts">
import * as THREE from "three";
import {onBeforeMount, onMounted, ref} from "vue";
import woodUrl from '/wood.png';

type SceneObjects = {
  scene: THREE.Scene | null;
  camera: THREE.PerspectiveCamera | null;
  renderer: THREE.WebGLRenderer | null;
  door: THREE.Mesh | null;
}

const element = ref<HTMLCanvasElement | null>(null);
const doorHeight = ref<number>(5);
const doorWidth = ref<number>(3);
const objects: SceneObjects = {
  scene: null,
  camera: null,
  renderer: null,
  door: null
};

const addScene = () => {
  objects.scene = new THREE.Scene();
  objects.camera = new THREE.PerspectiveCamera(
      75,
      window.innerWidth / window.innerHeight,
      0.1,
      1000
  );

  if (objects.renderer) {
    objects.renderer.setSize(window.innerWidth, window.innerHeight);
    objects.renderer = new THREE.WebGLRenderer({
      canvas: element.value as unknown as HTMLCanvasElement,
      antialias: true
    });
  }

  if (objects.camera) {
    objects.camera.position.z = 5;
  }

  objects.scene.background = new THREE.Color("skyblue");
  //scene.add(camera);

  new THREE.TextureLoader().load(
      woodUrl,
      function (texture: THREE.Texture) {
        addFigures();
        addDoor(texture);
        //objects.renderer.render(scene, camera);
      }
  );
}

const addFigures = () => {
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

  scene.add(sphere, cube);
}

const addDoor = (texture: THREE.Texture) => {
  const material = new THREE.MeshBasicMaterial({ map: texture });
  door = new THREE.Mesh(
      new THREE.BoxGeometry(doorWidth.value,doorHeight.value,0.3,),
      material
  )
  door.position.set(3,0,0);
  scene.add(door);
}

const updateDoor = () => {
  if (!door) return;

  door.geometry.dispose();
  scene.remove(door);

  const newDoorGeometry = new THREE.BoxGeometry(doorWidth.value, doorHeight.value, 0.3);
  door = new THREE.Mesh(newDoorGeometry, door.material);
  door.position.set(3, 0, 0);
  scene.add(door);
}

onMounted(addScene);
onBeforeMount(() => {
  scene?.dispose();
  renderer?.dispose();
})

</script>

<style scoped>

</style>