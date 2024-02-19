<script setup lang="ts">
import { computed, onMounted, ref, watch } from 'vue';
import { PerspectiveCamera, Scene, WebGLRenderer } from 'three';
import { useWindowSize } from '@vueuse/core';

let renderer: WebGLRenderer;
let camera: PerspectiveCamera;
const experience = ref<HTMLCanvasElement | null>(null);
const scene = new Scene();

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

watch(aspectRatio, updateRenderer);
watch(aspectRatio, updateCamera);

camera = new PerspectiveCamera(75, aspectRatio.value, 0.1, 1000);
camera.position.z = 5;
scene.add(camera);

const loop = () => {
    renderer.render(scene, camera);
    requestAnimationFrame(loop);
}

onMounted(() => {
    renderer = new WebGLRenderer({
        canvas: experience.value as unknown as HTMLCanvasElement,
        antialias: true
    });

    updateRenderer();
    updateCamera();

    loop();
});


</script>
<template>
  <canvas ref="experience" />
</template>