<script setup>
import { ref, reactive, onMounted, onBeforeUnmount, provide } from 'vue'
import GameMap from '../components/GameMap.vue'
import ImageCard from '../components/ImageCard.vue'

const socket = new WebSocket('ws://140.246.21.37:8888') // ← 用你的后端地址端口
const playerId = Math.random().toString(36).slice(2, 10)

provide('gameSocket', socket) // ✅ 提供 socket 给子组件用
provide('playerId', playerId) // ✅ 提供 playerId（可选）

// 如果你这里有 cards，也可以定义并 provide 出去
// provide('cards', cards)



socket.onopen = () => {
  console.log("[WebSocket] 连接成功");
  // 连接成功后主动发送一条测试消息
  socket.send(JSON.stringify({ type: "test", message: "Hello from client" }));
};




onMounted(() => {
  socket.onmessage = (event) => {
    try {
      const data = JSON.parse(event.data);
      
      // ====== 调试代码放在这里 ======
      if (true) { 
        console.log("收到任意数据：", data); 
      }
      // =============================

      if (data.type === 'move') {
        console.log('收到移动数据：');
      }
    } catch (error) {
      console.error("JSON 解析失败:", error, "原始数据:", event.data);
    }
  };
});



// 创建卡组的工具函数
function createDeck({ length, prefix, props }) {
  return Array.from({ length }, (_, i) => ({
    id: `${prefix}-${i + 1}`,
    ...props(i)
  }))
}

// 游戏状态
// 修改主卡组为四个颜色子卡组
const decks = reactive({
  
  mainDeckRed: createDeck({
    length: 50,
    prefix: 'main-red',
    props: () => ({ color: '#FF6B6B', type: 'colorBlock' })
  }),
  mainDeckBlue: createDeck({
    length: 50,
    prefix: 'main-blue',
    props: () => ({ color: '#4ECDC4', type: 'colorBlock' })
  }),
  mainDeckYellow: createDeck({
    length: 50,
    prefix: 'main-yellow',
    props: () => ({ color: '#FFD166', type: 'colorBlock' })
  }),
  mainDeckGreen: createDeck({
    length: 50,
    prefix: 'main-green',
    props: () => ({ color: '#90EE90', type: 'colorBlock' })
  }),
  deckA: createDeck({
    length: 11,
    prefix: 'deckA',
    props: (i) => ({ 
      image: `/image/operators/split-${i + 1}.png`,
      type: 'operator'
    })
  }),
  deckB: createDeck({
    length: 7,
    prefix: 'deckB',
    props: (i) => ({
      image: `/image/enterprises/split-${i + 1}.png`,
      type: 'enterprise'
    })
  }),
  deckC: createDeck({
    length: 44,
    prefix: 'buildings',
    props: (i) => ({
      image: `/image/buildings/split-${i + 1}.png`,
      type: 'buildings', 
      width: 240,
      height: 340
    })
  }),

  eventDeckA: createDeck({
    length: 6,
    prefix: 'eventA',
    props: (i) => ({
      image: `image/greenevents/split-${i + 1}.png`
    })
  }),
  eventDeckB: createDeck({
    length: 11,
    prefix: 'eventB',
    props: (i) => ({
      image: `image/yellowevents/split-${i + 1}.png`
    })
  }),
  eventDeckC: createDeck({
    length: 7,
    prefix: 'eventC',
    props: (i) => ({
      image: `image/redevents/split-${i + 1}.png`
    })
  }),
  deckD: createDeck({
    length: 18,
    prefix: 'buildingsstyle',
    props: (i) => ({
      image: `/image/buildingsstyle/split-${i + 1}.png`,
      type: 'buildingsstyle', // Ensure type property exists

    })
  }),
  deckE: createDeck({
    length: 7,
    prefix: 'enterprise',
    props: (i) => ({
      image: `/image/enterprises/split-${i + 1}.png`,
      type: 'enterprise', // Ensure type property exists
      width: 325,
      height: 925
    })
  })
})

const cardsInPlay = reactive([])

// 角色系统
const characters = reactive([
  {
    id: 1,
    name: '小红',
    resources: {
      钱: 0,
      土: 0,
      铁: 0,
      源石: 0,
      至纯源石:0,
      总得分: 0
    },
    color: '#FF6B6B',
    
  },
  {
    id: 2,
    name: '小蓝',
    resources: {
      钱: 0,
      土: 0,
      铁: 0,
      源石: 0,
      至纯源石:0,
      总得分: 0
    },
    color: '#4ECDC4'
  },
  {
    id: 3,
    name: '小黄',
    resources: {
      钱: 0,
      土: 0,
      铁: 0,
      源石: 0,
      至纯源石:0,
      总得分: 0
    },
    color: '#FFD166'
  },
  {
    id: 4,
    name: '小绿',
    resources: {
      钱: 0,
      土: 0,
      铁: 0,
      源石: 0,
      至纯源石:0,
      总得分: 0
    },
    color: '#90EE90'
  },
])

// 拖动状态
const position = reactive({ x: 0, y: 0 })
const isDragging = ref(false)
const dragStart = reactive({ x: 0, y: 0 })

// 卡牌拖动状态
const cardDragState = reactive({
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

// 抽卡功能  计算出生创造点
function drawCard(deckName) {
  if (decks[deckName].length === 0) return
  
  const randomIndex = Math.floor(Math.random() * decks[deckName].length)
  const card = decks[deckName].splice(randomIndex, 1)[0]
  
  cardsInPlay.push({
    ...card,
    x: window.innerWidth / 2 + 2600,
    y: window.innerHeight / 2 - 3200,
    zIndex: 1,
    'data-type': card.type // 添加类型属性以应用正确的样式
  })
}

// 资源管理
function changeResource(characterIndex, resourceName, delta) {
  const character = characters[characterIndex]
  character.resources[resourceName] = Math.max(0, character.resources[resourceName] + delta)
}

// 桌面拖动处理函数
function startDrag(event) {
  const target = event.target;

  isDragging.value = true;
  dragStart.x = event.clientX - position.x;
  dragStart.y = event.clientY - position.y;
}


function onDrag(event) {
  if (!isDragging.value) return
  
  position.x = event.clientX - dragStart.x
  position.y = event.clientY - dragStart.y
}

function stopDrag() {
  isDragging.value = false
}



// 卡牌拖动处理函数
function startCardDrag(card, event) {
  event.stopPropagation()
  event.preventDefault() // Prevent default behavior (especially important for touch events)
  
  // Add boundary check to prevent triggering desktop drag
  if (event.target.closest('.card')) {
    isDragging.value = false
  }

  cardDragState.isDragging = true
  cardDragState.currentCard = card
  
  const clientX = event.clientX || event.touches[0].clientX
  const clientY = event.clientY || event.touches[0].clientY
  
  cardDragState.offsetX = clientX - card.x
  cardDragState.offsetY = clientY - card.y
  
  card.zIndex = 100
  
  document.addEventListener('mousemove', handleCardDrag)
  document.addEventListener('mouseup', stopCardDrag)
  document.addEventListener('touchmove', handleCardDrag)
  document.addEventListener('touchend', stopCardDrag)
}

function handleCardDrag(event) {
  if (!cardDragState.isDragging) return
  
  const clientX = event.clientX || event.touches[0].clientX
  const clientY = event.clientY || event.touches[0].clientY
  
  cardDragState.currentCard.x = clientX - cardDragState.offsetX
  cardDragState.currentCard.y = clientY - cardDragState.offsetY
}

function stopCardDrag() {
  if (cardDragState.currentCard) {
    cardDragState.currentCard.zIndex = 1
  }
  
  cardDragState.isDragging = false
  cardDragState.currentCard = null
  
  document.removeEventListener('mousemove', handleCardDrag)
  document.removeEventListener('mouseup', stopCardDrag)
  document.removeEventListener('touchmove', handleCardDrag)
  document.removeEventListener('touchend', stopCardDrag)
}

const subDeckSettings = ref({
  eventDeckA: {
    name: '事件卡组A',
    capacity: 10,
    imagePath: '/image/greenevents/split-'
  },
  eventDeckB: {
    name: '事件卡组B',
    capacity: 10,
    imagePath: '/image/yellowevents/split-'
  },
  eventDeckC: {
    name: '事件卡组C',
    capacity: 10,
    imagePath: '/image/redevents/split-'
  }
})
</script>

<template>
  <div class="game-container" 
       @mousedown="startDrag"
       @mousemove="onDrag"
       @mouseup="stopDrag"
       @mouseleave="stopDrag"
       :style="{
         transform: `translate(${position.x}px, ${position.y}px)`,
         cursor: isDragging ? 'grabbing' : 'grab'
       }">
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
            <button  @click.stop="changeResource(index, name, 1)" style="color: black;">+</button>
            <button  @click.stop="changeResource(index, name, -1)" style="color: black;">-</button>
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
      <!-- 放置的卡牌  圣堂 -->

    <ImageCard
      image-src="/image/core.png"
      :width="240"
      :height="340"
      :count="4"
      :z-index="1"
      :initial-position="(i) => ({ x: 1000 + i * 30, y: -1000 })"
    />
    
    <ImageCard
      image-src="/image/board.png"
      :width="800"
      :height="1400"
      :count="4"
      :z-index="0"
      :initial-position="(i) => ({ x: 1000 + i * 30, y: -1000 })"
    />

    <ImageCard
      image-src="/image/highway.jpg"
      :width="120"
      :height="40"
      :count="4"
      :z-index="1"
      :initial-position="(i) => ({ x: 1000 + i * 30, y: -1000 })"
    />
    <ImageCard
      image-src="/image/yellowlong.png"
      :width="240"
      :height="340"
      :count="1"
      :z-index="2"
      :initial-position="(i) => ({ x: 1000 + i * 30, y: -1000 })"
    />
    <ImageCard
      image-src="/image/redlong.png"
      :width="240"
      :height="340"
      :count="1"
      :z-index="2"
      :initial-position="(i) => ({ x: 1000 + i * 30, y: -1000 })"
    />
    <ImageCard
      image-src="/image/bluelong.png"
      :width="240"
      :height="340"
      :count="1"
      :z-index="2"
      :initial-position="(i) => ({ x: 1000 + i * 30, y: -1000 })"
    />
    <ImageCard
      image-src="/image/redcity.png"
      :width="50"
      :height="90"
      :count="1"
      :z-index="3"
      :initial-position="(i) => ({ x: 1000 + i * 30, y: -1000 })"
    />
    <ImageCard
      image-src="/image/bluecity.png"
      :width="50"
      :height="90"
      :count="1"
      :z-index="3"
      :initial-position="(i) => ({ x: 1000 + i * 30, y: -1000 })"
    />
    <ImageCard
      image-src="/image/greencity.png"
      :width="50"
      :height="90"
      :count="1"
      :z-index="3"
      :initial-position="(i) => ({ x: 1000 + i * 30, y: -1000 })"
    />
    <ImageCard
      image-src="/image/yellowcity.png"
      :width="50"
      :height="90"
      :count="1"
      :z-index="3"
      :initial-position="(i) => ({ x: 1000 + i * 30, y: -1000 })"
    />
    <ImageCard
      image-src="/image/xianshou.png"
      :width="80"
      :height="140"
      :count="1"
      :z-index="2"
      :initial-position="(i) => ({ x: 1000 + i * 30, y: -1000 })"
    />

      <div 
        class="card" 
        v-for="card in cardsInPlay" 
        :key="card.id"
        :data-type="card.type"
        :style="{
          left: card.x + 'px',
          top: card.y + 'px',
          backgroundColor: card.color,
          backgroundImage: card.image ? `url(${card.image})` : 'none',
          zIndex: card.zIndex
        }"
        @mousedown="startCardDrag(card, $event)"
        @touchstart="startCardDrag(card, $event)"
      >
        <span v-if="!card.image">卡牌 {{ card.id.split('-')[1] }}</span>
      </div>
    </div>
    
    <!-- 卡组区域 -->
    <div class="deck-area">

  <!-- 主卡组 (四色卡组) -->
<div class="main-deck-group">
  <div
    class="deck main-deck"
    v-for="(label, name) in {
      mainDeckRed: '红色',
      mainDeckBlue: '蓝色',
      mainDeckYellow: '黄色',
      mainDeckGreen: '绿色'
    }"
    :key="name"
    @click="drawCard(name)"
    :style="{ backgroundColor: decks[name][0]?.color || '#747377' }"
  >
    <div v-if="decks[name].length === 0" class="empty-deck">
      空
    </div>
    <div v-else class="deck-content">
      {{ label }}卡组<br>{{ decks[name].length }}
    </div>
  </div>
</div>

      
      <!-- 副卡组 (右侧) -->
      <div class="sub-decks">
        <div 
          class="deck sub-deck" 
          v-for="(deck, name) in { deckA: '干员', deckB: '企业板', deckC: '建筑',deckD:'建筑样式' }" 
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

      <!-- 事件卡组 -->
      <div class="event-decks">
        <div 
          class="deck event-deck" 
          v-for="(deck, name) in { eventDeckA: 'A', eventDeckB: 'B', eventDeckC: 'C' }" 
          :key="name"
          @click="drawCard(name)"
        >
          <div v-if="decks[name].length === 0" class="empty-deck">
            空
          </div>

          
          <div v-else class="deck-content">
            事件卡组{{ deck }}<br>{{ decks[name].length }}
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.game-container {
  position: relative;
  width: 350vw;  /* 扩大容器尺寸 */
  height: 350vh;
  background-color: #1f1e33;
  transition: transform 0.1s ease;
  will-change: transform;
  user-select: none;
}

.play-area {
  position: relative;
  width: 100%;
  height: 100%;
  z-index: 1;
  min-height: 800px; /* 确保有足够空间显示横向卡牌 */
}

:root {
  --card-bg-color: white;
  --card-border-radius: 8px;
  --card-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
  --card-active-shadow: 0 6px 12px rgba(0, 0, 0, 0.3);
  --card-transition: transform 0.1s, box-shadow 0.1s;
}

/* 卡牌基础样式 */
.card {
  pointer-events: auto;
  background-color: var(--card-bg-color);
  border-radius: var(--card-border-radius);
  box-shadow: var(--card-shadow);
  display: flex;
  justify-content: center;
  align-items: center;
  position: absolute;
  cursor: grab;
  user-select: none;
  transition: var(--card-transition);
  font-weight: bold;
  color: #333;
  z-index: 1;
  background-size: cover;
  background-position: center;
}

/* 纯色方块卡牌样式 */
.play-area .card[data-type="colorBlock"] {
  width: 25px !important;
  height: 25px !important;
  background-color: var(--card-color);
  background-image: none !important;
  font-size: 0 !important;
  color: transparent !important;
  z-index: 9999 !important;
  position: relative;
  pointer-events: auto !important;
}

/* 扩大点击区域 */
.play-area .card[data-type="colorBlock"]::after {
  content: '';
  position: absolute;
  top: -15px;
  left: -15px;
  right: -15px;
  bottom: -15px;
  z-index: 9999;
  pointer-events: auto;
}

/* 确保colorBlock卡牌优先响应事件 */
.play-area .card[data-type="colorBlock"] {
  pointer-events: auto !important;
}

/* 恢复其他卡牌的交互能力 */
.play-area .card:not([data-type="colorBlock"]) {
  pointer-events: auto !important;
}

.play-area .card[data-type="colorBlock"]::before {
  content: none !important;
}

/* Increase selector specificity */
.play-area .card[data-type="operator"] {
  width: 300px !important;
  height: 425px !important;
}

.play-area .card[data-type="buildings"] {
  width: 240px !important;
  height: 340px !important;
}

.play-area .card[data-type="buildingsstyle"] {
  width: 372px !important;
  height: 240px !important;
}

.play-area .card[data-type="enterprise"] {
  width: 240px !important;
  height: 660px !important;
}

/* Add scope limitation to default styles */
.play-area .card:not([data-type]) {
  width: 680px;
  height: 480px;
}

.card:active {
  cursor: grabbing;
  /* 去除缩放和阴影变化 */
  transform: none;
  box-shadow: var(--card-shadow);
}


/* 角色面板 */
.character-panel {
  position: fixed;  /* 保持固定定位 */
  top: 570px;
  left: 1500px;
  display: flex;
  gap: 5px;
  z-index: 2;
}

.character-card {
  padding: 15px;
  border-radius: 10px;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
  width: 50px;
  color: black;
  
}

.character-card h3 {
  margin-top: 0;
  margin-bottom: 10px;
  text-align: center;
  font-size: 1.5em;
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
  position: fixed;
  top: 500px;
  right: 1850px;
  display: flex;
  flex-direction: column;
  gap: 20px;
  z-index: 0;
}

/* 卡组样式 */
.deck {
  width: 280px;
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
.main-deck-group {
  display: flex;
  gap: 10px;
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
  position: relative;  /* 改为相对定位 */
  width: 100%;
  height: 100%;
  z-index: 0;
}

.character-card, .deck {
  font-size: 14px;
}

.character-card {
  width: 180px;
}

.deck {
  width: 80px;
  height: 120px;
}

/* 事件卡组样式 */
.event-decks {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.event-deck {
  background-color: rgba(0, 0, 0, 0.7);
  width: 80px;    /* 与其他卡组按钮统一 */
  height: 120px;
}


</style>