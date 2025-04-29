<script setup>
import { ref, reactive, onMounted } from 'vue'
import GameMap from '../components/GameMap.vue'

// 游戏状态
const decks = reactive({
  mainDeck: Array.from({ length: 20 }, (_, i) => ({
    id: `main-${i + 1}`,
    color: getRandomColor()
  })),
  deckA: Array.from({ length: 10 }, (_, i) => ({
    id: `deckA-${i + 1}`,
    color: '#FFC0CB' // 粉色
  })),
  deckB: Array.from({ length: 10 }, (_, i) => ({
    id: `deckB-${i + 1}`,
    color: '#ADD8E6' // 浅蓝
  })),
  deckC: Array.from({ length: 10 }, (_, i) => ({
    id: `deckC-${i + 1}`,
    color: '#90EE90' // 浅绿
  }))
})

const cardsInPlay = reactive([])

// 角色系统
const characters = reactive([
  {
    id: 1,
    name: '暴风赤红',
    resources: {
      生命值: 10,
      攻击力: 3,
      防御力: 2
    },
    color: '#FF6B6B'
  },
  {
    id: 2,
    name: '法师',
    resources: {
      魔力: 8,
      智力: 5,
      防御力: 1
    },
    color: '#4ECDC4'
  },
  {
    id: 3,
    name: '游侠',
    resources: {
      敏捷: 7,
      幸运: 4,
      潜行: 3
    },
    color: '#FFD166'
  }
])

// 拖拽状态
const dragState = reactive({
  isDragging: false,
  currentCard: null,
  offsetX: 0,
  offsetY: 0
})

// 辅助函数
function getRandomColor() {
  const colors = ['#FFC0CB', '#ADD8E6', '#90EE90', '#FFD700', '#FFA07A']
  return colors[Math.floor(Math.random() * colors.length)]
}

// 抽卡功能
function drawCard(deckName) {
  if (decks[deckName].length === 0) return
  
  const card = decks[deckName].shift()
  
  cardsInPlay.push({
    ...card,
    x: window.innerWidth / 2 - 50,
    y: window.innerHeight / 2 - 75,
    zIndex: 1
  })
}

// 资源管理
function changeResource(characterIndex, resourceName, delta) {
  const character = characters[characterIndex]
  character.resources[resourceName] = Math.max(0, character.resources[resourceName] + delta)
}

// 拖拽功能
function startDrag(card, event) {
  dragState.isDragging = true
  dragState.currentCard = card
  
  const clientX = event.clientX || event.touches[0].clientX
  const clientY = event.clientY || event.touches[0].clientY
  
  dragState.offsetX = clientX - card.x
  dragState.offsetY = clientY - card.y
  
  // 提高当前卡牌的z-index
  card.zIndex = 100
  
  document.addEventListener('mousemove', handleDrag)
  document.addEventListener('mouseup', stopDrag)
  document.addEventListener('touchmove', handleDrag)
  document.addEventListener('touchend', stopDrag)
}

function handleDrag(event) {
  if (!dragState.isDragging) return
  
  const clientX = event.clientX || event.touches[0].clientX
  const clientY = event.clientY || event.touches[0].clientY
  
  dragState.currentCard.x = clientX - dragState.offsetX
  dragState.currentCard.y = clientY - dragState.offsetY
}

function stopDrag() {
  if (dragState.currentCard) {
    dragState.currentCard.zIndex = 1
  }
  
  dragState.isDragging = false
  dragState.currentCard = null
  
  document.removeEventListener('mousemove', handleDrag)
  document.removeEventListener('mouseup', stopDrag)
  document.removeEventListener('touchmove', handleDrag)
  document.removeEventListener('touchend', stopDrag)
}
</script>

<template>
  <div class="game-container">
    <!-- 角色资源面板 -->
    <div class="character-panel">
      <div 
        class="character-card" 
        v-for="(character, index) in characters" 
        :key="character.id"
        :style="{ backgroundColor: character.color }"
      >
        <h3>{{ character.name }}</h3>
        <div class="resource" v-for="(value, name) in character.resources" :key="name">
          <span>{{ name }}: {{ value }}</span>
          <div class="resource-controls">
            <button @click.stop="changeResource(index, name, 1)">+</button>
            <button @click.stop="changeResource(index, name, -1)">-</button>
          </div>
        </div>
      </div>
    </div>
    
    <!-- 游戏地图 -->
    <div class="map-container">
      <GameMap />
    </div>
    
    <!-- 游戏主区域 -->
    <div class="play-area">
      <!-- 放置的卡牌 -->
      <div 
        class="card" 
        v-for="card in cardsInPlay" 
        :key="card.id"
        :style="{
          left: card.x + 'px',
          top: card.y + 'px',
          backgroundColor: card.color,
          zIndex: card.zIndex
        }"
        @mousedown="startDrag(card, $event)"
        @touchstart="startDrag(card, $event)"
      >
        卡牌 {{ card.id.split('-')[1] }}
      </div>
    </div>
    
    <!-- 卡组区域 -->
    <div class="deck-area">
      <!-- 主卡组 (右上角) -->
      <div 
        class="deck main-deck" 
        @click="drawCard('mainDeck')"
      >
        <div v-if="decks.mainDeck.length === 0" class="empty-deck">
          空
        </div>
        <div v-else class="deck-content">
          主卡组<br>{{ decks.mainDeck.length }}
        </div>
      </div>
      
      <!-- 副卡组 (右侧) -->
      <div class="sub-decks">
        <div 
          class="deck sub-deck" 
          v-for="(deck, name) in { deckA: 'A', deckB: 'B', deckC: 'C' }" 
          :key="name"
          @click="drawCard(name)"
        >
          <div v-if="decks[name].length === 0" class="empty-deck">
            空
          </div>
          <div v-else class="deck-content">
            卡组{{ deck }}<br>{{ decks[name].length }}
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.game-container {
  position: relative;
  width: 100vw;
  height: 100vh;
  overflow: hidden;
  background-color: #f5f5f5;
}

.play-area {
  position: absolute;
  width: 100%;
  height: 100%;
  z-index: 1;
  pointer-events: none;
}

/* 卡牌样式 */
.card {
  pointer-events: auto;
  width: 100px;
  height: 150px;
  background-color: white;
  border-radius: 8px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
  display: flex;
  justify-content: center;
  align-items: center;
  position: absolute;
  cursor: grab;
  user-select: none;
  transition: transform 0.1s, box-shadow 0.1s;
  font-weight: bold;
  color: #333;
}

.card:active {
  cursor: grabbing;
  transform: scale(1.05);
  box-shadow: 0 6px 12px rgba(0, 0, 0, 0.3);
}

/* 角色面板 */
.character-panel {
  position: absolute;
  top: 20px;
  left: 20px;
  display: flex;
  gap: 15px;
  z-index: 2;
}

.character-card {
  padding: 15px;
  border-radius: 10px;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
  width: 180px;
  color: white;
}

.character-card h3 {
  margin-top: 0;
  margin-bottom: 10px;
  text-align: center;
  font-size: 1.2em;
}

.resource {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 8px;
  font-size: 0.9em;
}

.resource-controls {
  display: flex;
  gap: 5px;
}

.resource-controls button {
  width: 25px;
  height: 25px;
  border: none;
  border-radius: 50%;
  background-color: rgba(255, 255, 255, 0.2);
  color: white;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
}

.resource-controls button:hover {
  background-color: rgba(255, 255, 255, 0.3);
}

/* 卡组区域 */
.deck-area {
  position: absolute;
  top: 20px;
  right: 20px;
  display: flex;
  flex-direction: column;
  gap: 15px;
  z-index: 2;
}

/* 卡组样式 */
.deck {
  width: 80px;
  height: 120px;
  border-radius: 8px;
  background-color: #333;
  color: white;
  display: flex;
  justify-content: center;
  align-items: center;
  cursor: pointer;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.2);
  transition: transform 0.1s;
}

.deck:hover {
  transform: translateY(-5px);
}

.main-deck {
  background-color: #6a5acd; /* 紫罗兰色 */
}

.sub-decks {
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.sub-deck {
  background-color: #4682b4; /* 钢蓝色 */
}

.deck-content {
  text-align: center;
  font-size: 0.8em;
  padding: 5px;
}

.empty-deck {
  opacity: 0.7;
  font-size: 0.9em;
}

.map-container {
  position: absolute;
  top: 70%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 120%;
  height: 120%;
  z-index: 0;
}
</style>