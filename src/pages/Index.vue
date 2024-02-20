<script setup lang="ts">
import { computed, onMounted, ref, watch } from 'vue';
import { AmbientLight, Box3, Color, Mesh, MeshPhysicalMaterial, PerspectiveCamera, Scene, Vector3, WebGLRenderer } from 'three';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
import { STLLoader } from 'three/examples/jsm/loaders/STLLoader'
import { useWindowSize } from '@vueuse/core';

let renderer: WebGLRenderer;
let camera: PerspectiveCamera;
let controls: OrbitControls;
let loadedMesh: Mesh;
const experience = ref<HTMLCanvasElement | null>(null);
const fileRef = ref<HTMLInputElement | null>(null);
const scene = new Scene();
const loader = new STLLoader();
const light = new AmbientLight(0x404040);

const { width, height } = useWindowSize();

const aspectRatio = computed(() => width.value / height.value)

function updateRenderer() {
    renderer.setSize(width.value, height.value);
    renderer.setPixelRatio(window.devicePixelRatio);
}

function updateCamera() {
    camera.aspect = aspectRatio.value;
    camera.updateProjectionMatrix;
}


function handleFileUpload() {
    const fileLoaded = fileRef?.value?.files?.[0];

    if(!fileLoaded) return;

    let reader = new FileReader();
    reader.onload = (event) => {
        if (event?.target?.result) loadSTLModel(event?.target?.result as string);
    }

    reader.readAsDataURL(fileLoaded);
}
function loadSTLModel(fileBase64: string) {
    loader.load(
        fileBase64,
        (geometry) => {
/*             const boxGeometry = new BoxGeometry(
                1, 1, 1
            ); */

            const material = new MeshPhysicalMaterial({
                color: 0xb2ffc8,
                metalness: 0.25,
                roughness: 0.1,
                opacity: 1.0,
                transparent: false,
                transmission: 0.99,
                clearcoat: 1.0,
                clearcoatRoughness: 0.25
            });
            if (loadedMesh) scene.remove(loadedMesh);
            /* const mesh = new Mesh(boxGeometry, new MeshBasicMaterial({color: 0x09FF95})); */
            loadedMesh = new Mesh(geometry, material);
            /* const mesh = new Mesh(geometry, new MeshLambertMaterial({color: 0x0ff00})); */
            /*loadedMesh.scale.set(0.1, 0.1, 0.1);
             loadedMesh.position.set(-10, -2, 0);*/ 
            loadedMesh.rotation.x = Math.PI / 2;

            const box = new Box3();
            box.setFromObject(loadedMesh);

            const size = new Vector3();
            box.getSize(size);

            const center = new Vector3();
            box.getCenter(center);
        
            const scaleTemp = new Vector3().copy({x:1,y:1, z:1}).divide(size);
            const scale = Math.min(scaleTemp.x, Math.min(scaleTemp.y, scaleTemp.z));
            loadedMesh.scale.setScalar(scale);

            loadedMesh.position.sub(center.multiplyScalar(scale));

            scene.add(loadedMesh);

            loop();

        },
        (xhr) => {
            console.log('loading')
        },
        (error) => {
            console.log(error)
        }
    )
}

watch(aspectRatio, updateRenderer);
watch(aspectRatio, updateCamera);

camera = new PerspectiveCamera(75, aspectRatio.value, 0.1, 1000);
camera.position.z = 5;
scene.add(camera);
light.position.set(20, 20, 20)
scene.add(light);
scene.background = new Color(0x4c61d4);
scene.traverse(object => {
    object.frustumCulled = false;
});

const loop = () => {
    controls.update();
    renderer.render(scene, camera);
    requestAnimationFrame(loop);
}

onMounted(() => {
    renderer = new WebGLRenderer({
        canvas: experience.value as unknown as HTMLCanvasElement,
        antialias: true
    });
    controls = new OrbitControls(camera, renderer.domElement)
    controls.enableDamping = true;

    updateRenderer();
    updateCamera();

    loop();
});


</script>
<template>
    <div class="w-screen absolute text-white text-center top-1/4" style="transform: translateY(-50%);">
		<h1 class="font-mono font-bold text-3xl tracking-wide">Iru Hernandez</h1>
		<p class="font-exo font-bold text-6xl">Vue 3 + ThreeJS Visualizer</p>
        <button v-if="fileRef" @click="fileRef.click()" class="mt-2 bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-4 rounded inline-flex items-center font-mono font-bold text-xl tracking-wide">Load STL model</button>
        <input ref="fileRef" type="file" class="hidden" @change="handleFileUpload()">
	</div>
  <canvas ref="experience" />
</template>