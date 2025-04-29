<template>
  <div class="game-map">
    <div class="map-grid">
      <!-- 图片块 -->
      <div 
        v-for="(block, index) in imageBlocks" 
        :key="'img-' + index"
        class="map-block"
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
        class="map-block"
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

    <!-- 颜色选择器 -->
    <div v-if="showColorPicker" class="picker color-picker">
      <div class="picker-options">
        <div 
          v-for="color in colors" 
          :key="color"
          class="picker-option"
          :style="{ backgroundColor: color }"
          @click="selectColor(color)"
        ></div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive } from 'vue'

// 图片资源
const images = [
  '../assets/image/土.png',
  '../assets/image/铁.png',
  '../assets/image/源石.png',
  '../assets/image/至纯源石.png',
  '../assets/image/双倍土.png'
]

// 颜色选项
const colors = ['#FF6B6B', '#FFD93D', '#4ECDC4', '#95E1D3']

// 图片块数据
const imageBlocks = reactive([
  { x: 20, y: 20, image: null },
  { x: 40, y: 20, image: null }
])

// 颜色块数据
const colorBlocks = reactive([
  { x: 60, y: 20, color: null },
  { x: 80, y: 20, color: null }
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

// 打开颜色选择器
const openColorPicker = (block) => {
  currentBlock.value = block
  showColorPicker.value = true
  showImagePicker.value = false // 确保另一个选择器关闭
}

// 选择颜色
const selectColor = (color) => {
  if (currentBlock.value) {
    currentBlock.value.color = color
  }
  showColorPicker.value = false
}
</script>

<style scoped>
.game-map {
  position: relative;
  width: 90vmin;
  height: 90vmin;
  background-image: url('../assets/image/map.jpg');
  background-size: contain;
  background-repeat: no-repeat;
  background-position: center;
  margin: auto;
  border-radius: 8px;
  overflow: hidden;
  background-color: #1a1a1a;
}

.map-grid {
  position: relative;
  width: 100%;
  height: 100%;
}

.map-block {
  position: absolute;
  width: 60px;
  height: 60px;
  border: 2px solid rgba(255, 255, 255, 0.3);
  cursor: pointer;
  display: flex;
  justify-content: center;
  align-items: center;
  transition: all 0.3s ease;
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
  left: 20px;
  bottom: 20px;
}

.color-picker {
  right: 20px;
  bottom: 20px;
}

.picker-options {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 10px;
}

.picker-option {
  width: 60px;
  height: 60px;
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