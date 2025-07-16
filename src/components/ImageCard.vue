<template>
  <div 
    class="image-card"
    v-for="card in cards"
    :key="card.id"
    :style="{
      transform: `translate(${card.x}px, ${card.y}px)`,
      width: width + 'px',
      height: height + 'px',
      zIndex: card.zIndex,
      transition: card.isDragging ? 'none' : 'transform 0.1s ease-out'
    }"
    @mousedown="startDrag(card, $event)"
    @touchstart.prevent="startDrag(card, $event)"
    @mousedown.stop="startDrag(card, $event)"  
    @touchstart.stop.prevent="startDrag(card, $event)"  
  >
    <img 
      :src="imageSrc" 
      :style="{
        width: '100%',
        height: '100%',
        objectFit: 'cover',
        pointerEvents: 'none'
      }"
    />
  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount,inject } from 'vue'

const socket = inject('gameSocket')
const playerId = inject('playerId')

const props = defineProps({
  imageSrc: {
    type: String,
    required: true
  },
  width: {
    type: Number,
    default: 120
  },
  height: {
    type: Number,
    default: 80
  },
  count: {
    type: Number,
    default: 1
  },
  zIndex: {
    type: Number,
    default: 0
  },
  initialPosition: {
    type: Function,
    default: (i) => ({
      x: i * 30,
      y: i * 30
    })
  }
})

const cards = ref([])
let currentCard = null
const dragListeners = []

function initCards() {
  cards.value = Array.from({ length: props.count }, (_, i) => {
    const pos = props.initialPosition(i)
    return {
      id: `card-${i}`,
      x: pos.x,
      y: pos.y,
      zIndex: props.zIndex,
      isDragging: false
    }
  })
}

function startDrag(card, event) {
  currentCard = card
  currentCard.isDragging = true
  currentCard.zIndex = 9999
  
  const isTouch = event.type === 'touchstart'
  const clientX = isTouch ? event.touches[0].clientX : event.clientX
  const clientY = isTouch ? event.touches[0].clientY : event.clientY
  
  const startX = clientX
  const startY = clientY
  const startPosX = currentCard.x
  const startPosY = currentCard.y

  const moveHandler = (e) => {
    if (!currentCard) return
    
    const currentX = e.clientX || e.touches[0].clientX
    const currentY = e.clientY || e.touches[0].clientY
    
    currentCard.x = startPosX + (currentX - startX)
    currentCard.y = startPosY + (currentY - startY)

     // 同步给服务器
    if (socket && socket.readyState === WebSocket.OPEN) {
    socket.send(JSON.stringify({
      type: 'move',
      player: playerId,
      id: currentCard.id,
      x: currentCard.x,
      y: currentCard.y
    }))
  }


  }

  const endHandler = () => {
    if (currentCard) {
      currentCard.isDragging = false
      currentCard.zIndex = props.zIndex
      currentCard = null
    }
    cleanupListeners()
  }

  const events = [
    ['mousemove', moveHandler],
    ['touchmove', moveHandler],
    ['mouseup', endHandler],
    ['touchend', endHandler],
    ['mouseleave', endHandler]
  ]

  events.forEach(([type, handler]) => {
    document.addEventListener(type, handler)
    dragListeners.push({ type, handler })
  })
}

function cleanupListeners() {
  dragListeners.forEach(({ type, handler }) => {
    document.removeEventListener(type, handler)
  })
  dragListeners.length = 0
}

onMounted(() => {
  initCards()
})

onBeforeUnmount(() => {
  cleanupListeners()
})
</script>

<style scoped>
.image-card {
  position: absolute;
  top: 0;
  left: 0;
  background-color: white;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.2);
  cursor: grab;
  user-select: none;
  overflow: hidden;
  will-change: transform;
}

.image-card:active {
  cursor: grabbing;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.3);
}
</style>