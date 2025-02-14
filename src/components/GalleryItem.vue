<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue';
import * as THREE from 'three';

const props = defineProps<{
  imageUrl: string;
}>();

const itemRef = ref<HTMLDivElement>();
const isZoomed = ref(false);
const canvasAdded = ref(false);

const getRandomColor = () => {
  const letters = '0123456789ABCDEF';
  let color = '0x';
  for (let i = 0; i < 6; i++) {
    color += letters[Math.floor(Math.random() * 16)];
  }
  return parseInt(color, 16);
};

const handleClick = () => {
  if (!isZoomed.value) {
    if (itemRef.value) {
      itemRef.value.style.transform = 'translateZ(calc(var(--index) * 10))';
      itemRef.value.style.filter = 'inherit';
    }
    isZoomed.value = true;
  } else {
    if (!canvasAdded.value && itemRef.value) {
      const canvas = document.createElement('canvas');
      const img = new Image();
      img.src = props.imageUrl;
      
      img.onload = () => {
        if (!itemRef.value) return;
        
        canvas.width = itemRef.value.clientWidth;
        canvas.height = itemRef.value.clientHeight;
        itemRef.value.appendChild(canvas);
        canvasAdded.value = true;

        canvas.style.position = 'absolute';
        canvas.style.top = '0';
        canvas.style.left = '0';
        canvas.style.width = '100%';
        canvas.style.height = '100%';
        canvas.style.zIndex = '1';

        const scene = new THREE.Scene();
        const camera = new THREE.PerspectiveCamera(75, canvas.width / canvas.height, 0.1, 1000);
        const renderer = new THREE.WebGLRenderer({ canvas, alpha: true });
        renderer.setSize(canvas.width, canvas.height);

        const geometry = new THREE.BoxGeometry(10, 10, 10);
        const material = new THREE.MeshBasicMaterial({ color: getRandomColor() });
        const cube = new THREE.Mesh(geometry, material);
        scene.add(cube);

        camera.position.z = 20;

        function animate() {
          requestAnimationFrame(animate);
          cube.rotation.x += 0.01;
          cube.rotation.y += 0.01;
          renderer.render(scene, camera);
        }

        animate();
      };
    } else {
      if (itemRef.value) {
        const canvas = itemRef.value.querySelector('canvas');
        if (canvas) {
          itemRef.value.removeChild(canvas);
          canvasAdded.value = false;
        }
      }
    }
  }
};

const handleOutsideClick = (event: MouseEvent) => {
  if (isZoomed.value && itemRef.value && event.target !== itemRef.value && !itemRef.value.contains(event.target as Node)) {
    if (itemRef.value) {
      itemRef.value.style.transform = '';
      const canvas = itemRef.value.querySelector('canvas');
      if (canvas) {
        itemRef.value.removeChild(canvas);
        canvasAdded.value = false;
      }
    }
    isZoomed.value = false;
  }
};

onMounted(() => {
  document.addEventListener('click', handleOutsideClick);
});

onUnmounted(() => {
  document.removeEventListener('click', handleOutsideClick);
});
</script>

<template>
  <div
    ref="itemRef"
    class="item"
    tabindex="0"
    :style="{ backgroundImage: `url(${imageUrl})` }"
    @click="handleClick"
  ></div>
</template>