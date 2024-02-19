<script setup lang="ts">
import { onMounted, ref } from 'vue';
import { PerspectiveCamera, Scene, WebGLRenderer } from 'three';

const experience = ref<HTMLCanvasElement | null>(null);
const scene = new Scene();

const camera = new PerspectiveCamera(75, window.innerWidth / innerHeight, 0.1, 1000);
scene.add(camera);

onMounted(() => {
    const renderer = new WebGLRenderer({
        canvas: experience.value as unknown as HTMLCanvasElement,
        antialias: true
    });

    renderer.setSize(window.innerWidth, window.innerHeight);
    renderer.render(scene, camera);
});


</script>
<template>
  <canvas ref="experience" />
</template>