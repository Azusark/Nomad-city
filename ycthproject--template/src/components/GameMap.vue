<template>
  <div class="game-map">
    <div class="map-grid">
      <!-- 图片块 -->
      <div 
        v-for="(block, index) in imageBlocks" 
        :key="'img-' + index"
        class="map-block image-block"
        :style="{
          left: `${block.x}%`,
          top: `${block.y}%`
        }"
        @click="openImagePicker(block)"
      >
        <img v-if="block.image" :src="block.image" alt="resource" class="block-content" />
        <div v-else class="block-placeholder">+</div>
      </div>

      <!-- 颜色块 -->
      <div 
        v-for="(block, index) in colorBlocks" 
        :key="'color-' + index"
        class="map-block color-block"
        :style="{
          left: `${block.x}%`,
          top: `${block.y}%`,
          backgroundColor: block.color || 'transparent'
        }"
        @click="openColorPicker(block)"
      >
        <div v-if="!block.color" class="block-placeholder">+</div>
      </div>
    </div>

    <!-- 图片选择器 -->
    <div v-if="showImagePicker" class="picker image-picker">
      <div class="picker-options">
        <div 
          v-for="(image, index) in images" 
          :key="index"
          class="picker-option"
          @click="selectImage(image)"
        >
          <img :src="image" :alt="'资源' + (index + 1)" />
        </div>
      </div>
    </div>

  </div>
</template>

<script setup>
import { ref, reactive } from 'vue'

// 图片资源
const images = [
  '/image/tu.png',
  '/image/铁.png',
  '/image/源石.png',
  '/image/至纯源石.png',
  '/image/双倍土.png'
];


// 图片块数据
const imageBlocks = reactive([
  { name: 'F-01', x: 31.5, y: 19.5, image: null },
  { name: 'F-02', x: 32.9, y: 39.9, image: null },
  { name: 'F-03', x: 21.2, y: 34.0, image: null },
  { name: 'E-01', x: 44.8, y: 60.0, image: null },
  { name: 'E-02', x: 30.0, y: 61.2, image: null },
  { name: 'E-03', x: 21.5, y: 55.4, image: null },
  { name: 'D-01', x: 56.2, y: 14.1, image: null },
  { name: 'D-02', x: 44.4, y: 15.5, image: null },
  { name: 'D-03', x: 45.1, y: 38.9, image: null },
  { name: 'C-01', x: 60.7, y: 83.5, image: null },
  { name: 'C-02', x: 45.7, y: 79.4, image: null },
  { name: 'C-03', x: 25.8, y: 80.1, image: null },
  { name: 'B-01', x: 70.7, y: 59.9, image: null },
  { name: 'B-02', x: 62.6, y: 67.0, image: null },
  { name: 'B-03', x: 54.8, y: 57.5, image: null },
  { name: 'A-01', x: 70.3, y: 16.1, image: null },
  { name: 'A-02', x: 67.5, y: 38.1, image: null },
  { name: 'A-03', x: 57.9, y: 31.5, image: null }
])



// 选择器状态
const showImagePicker = ref(false)
const showColorPicker = ref(false)
const currentBlock = ref(null)

// 打开图片选择器
const openImagePicker = (block) => {
  currentBlock.value = block
  showImagePicker.value = true
  showColorPicker.value = false // 确保另一个选择器关闭
}

// 选择图片
const selectImage = (image) => {
  if (currentBlock.value) {
    currentBlock.value.image = image
  }
  showImagePicker.value = false
}


</script>

<style scoped>
.game-map {
  position: relative;
  width: 120vw;
  height: 120vh;
  background-image: url('/image/map.jpg');
  background-size: contain;
  background-repeat: no-repeat;
  background-position: center;
  margin: auto;
  top: 500px;
  border-radius: 8px;
  overflow: hidden;
  background-color: #1f1e33;
}

.map-grid {
  position: relative;
  width: 100%;
  height: 100%;
}

.map-block {
  position: absolute;
  border: 2px solid rgba(255, 255, 255, 0.3);
  cursor: pointer;
  display: flex;
  justify-content: center;
  align-items: center;
  transition: all 0.3s ease;
}

.image-block {
  width: 60px;
  height: 60px;
}


.map-block:hover {
  border-color: rgba(255, 255, 255, 0.8);
  box-shadow: 0 0 10px rgba(255, 255, 255, 0.3);
}

.block-content {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.block-placeholder {
  color: white;
  font-size: 24px;
  opacity: 0.7;
}

.picker {
  position: fixed;
  padding: 20px;
  background: white;
  border-radius: 12px;
  box-shadow: 0 0 20px rgba(0, 0, 0, 0.3);
  z-index: 1000;
}

.image-picker {
  left:   2000px;
  bottom: 2300px;
}


.picker-options {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 10px;
}

.picker-option {
  width: 50px;
  height: 50px;
  border-radius: 8px;
  cursor: pointer;
  transition: transform 0.2s;
  border: 1px solid #ddd;
  overflow: hidden;
}

.picker-option:hover {
  transform: scale(1.1);
}

.picker-option img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}
</style> 