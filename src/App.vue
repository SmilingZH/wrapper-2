<script setup lang="ts">
import { ref } from 'vue';
import GalleryItem from './components/GalleryItem.vue';

const isFullscreen = ref(false);

const toggleFullscreen = async () => {
  if (!document.fullscreenElement) {
    try {
      await document.documentElement.requestFullscreen();
      isFullscreen.value = true;
    } catch (err) {
      console.error('Error attempting to enable full-screen mode:', err);
    }
  } else {
    if (document.exitFullscreen) {
      try {
        await document.exitFullscreen();
        isFullscreen.value = false;
      } catch (err) {
        console.error('Error attempting to exit full-screen mode:', err);
      }
    }
  }
};

const images = Array.from({ length: 9 }, (_, i) => {
  const number = String(i + 1).padStart(4, '0');
  return `/src/assets/works/${number}.png`;
});
</script>

<template>
  <button class="landscape-prompt" @click="toggleFullscreen">
    {{ isFullscreen ? '退出全屏' : '横屏看体验更好哦~~' }}
  </button>

  <div class="wrapper">
    <div class="prompt-text">点击图片后如果模型位置不对再点两下图片就好啦~~</div>
    <div class="items">
      <GalleryItem
        v-for="(image, index) in images"
        :key="index"
        :image-url="image"
      />
    </div>
  </div>
</template>