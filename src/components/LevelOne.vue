<template>
  <div class="level-one">
    <h2>Match & Meet!</h2>
    <div class="top-grid" ref="topGrid">
      <div
        v-for="(animal, idx) in animals"
        :key="animal.key"
        :ref="'slot-' + idx"
        class="animal-slot"
        :class="{ 'drop-target': dragOverIndex === idx, 'locked-correct': submitted && isCorrect(animal.key), 'locked-wrong': submitted && !isCorrect(animal.key) && selectedWordByAnimal[animal.key] }"
        @dragover.prevent="onDragOver(idx)"
        @dragleave.prevent="onDragLeave"
        @drop.prevent="onDrop(animal.key)"
      >
        <div class="animal-img" :aria-label="animal.name">
          {{ animal.emoji }}
        </div>
        <transition name="snap">
          <div 
            v-if="selectedWordByAnimal[animal.key]" 
            class="attached-word"
            draggable="true"
            @dragstart="onDragStartFromSlot($event, animal.key)"
            @touchstart="onTouchStartFromSlot($event, animal.key)"
            @touchmove.prevent="onTouchMove"
            @touchend="onTouchEnd"
          >
            {{ selectedWordByAnimal[animal.key] }}
          </div>
        </transition>
        <div v-if="submitted" class="result-mark">
          <span v-if="isCorrect(animal.key)" class="ok">✔</span>
          <span v-else-if="selectedWordByAnimal[animal.key]" class="err">✖</span>
        </div>
      </div>
    </div>

    <div class="cards" ref="cardsArea" :class="{ 'magnet-active': isNearCardsArea }">
      <div
        v-for="card in shuffledCards"
        :key="card"
        class="word-card"
        draggable="true"
        @dragstart="onDragStart(card)"
        @touchstart="onTouchStart($event, card)"
        @touchmove.prevent="onTouchMove"
        @touchend="onTouchEnd"
      >
        {{ card }}
      </div>
    </div>

    <div class="button-group">
      <button v-if="!submitted" class="submit-btn" @click="onSubmit">提交</button>
      <template v-else>
        <button class="retry-btn" @click="onRetry">重来</button>
        <button v-if="allCorrect" class="next-btn" @click="onNext">下一关</button>
      </template>
    </div>

    <transition name="celebrate">
      <div v-if="submitted && allCorrect" class="celebration">
        Great Job! 🎉
      </div>
    </transition>
    
    <!-- 触摸拖动的视觉反馈元素 -->
    <div
      v-if="touchDragging"
      class="touch-drag-ghost"
      :style="{ left: touchGhostX + 'px', top: touchGhostY + 'px' }"
    >
      {{ draggingWord }}
    </div>
    
    <audio ref="cheerAudio">
      <source src="https://assets.mixkit.co/active_storage/sfx/2018/2018-preview.mp3" type="audio/mpeg" />
    </audio>
  </div>
</template>

<script>
export default {
  name: 'LevelOne',
  data() {
    return {
      animals: [
        { key: 'lion', nameZh: '狮子', emoji: '🦁' },
        { key: 'elephant', nameZh: '大象', emoji: '🐘' },
        { key: 'giraffe', nameZh: '长颈鹿', emoji: '🦒' },
        { key: 'zebra', nameZh: '斑马', emoji: '🦓' },
        { key: 'monkey', nameZh: '猴子', emoji: '🐒' },
        { key: 'kangaroo', nameZh: '袋鼠', emoji: '🦘' }
      ],
      selectedWordByAnimal: {},
      draggingWord: null,
      dragOverIndex: null,
      submitted: false,
      // 触摸相关状态
      touchDragging: false,
      touchGhostX: 0,
      touchGhostY: 0,
      touchStartX: 0,
      touchStartY: 0,
      isNearCardsArea: false
    }
  },
  computed: {
    cards() {
      return this.animals.map(a => a.key)
    },
    shuffledCards() {
      // shuffle each render based on current assignments: show only unassigned + assigned pile at bottom
      const unassigned = this.cards.filter(c => !Object.values(this.selectedWordByAnimal).includes(c))
      return [...unassigned].sort(() => Math.random() - 0.5)
    },
    allCorrect() {
      return this.animals.every(a => this.isCorrect(a.key))
    }
  },
  methods: {
    onDragStart(card) {
      this.draggingWord = card
      this.submitted = false
    },
    onDragStartFromSlot(event, animalKey) {
      // 从槽位拖动标签
      const word = this.selectedWordByAnimal[animalKey]
      if (!word) return
      this.draggingWord = word
      this.submitted = false
      // 立即从槽位移除
      delete this.selectedWordByAnimal[animalKey]
    },
    onDragOver(idx) {
      this.dragOverIndex = idx
    },
    onDragLeave() {
      this.dragOverIndex = null
    },
    onDrop(animalKey) {
      if (!this.draggingWord) return
      // Remove previous binding of this word
      Object.keys(this.selectedWordByAnimal).forEach(key => {
        if (this.selectedWordByAnimal[key] === this.draggingWord) delete this.selectedWordByAnimal[key]
      })
      // Assign to animal (Vue 3 supports reactive property adds on Objects)
      this.selectedWordByAnimal[animalKey] = this.draggingWord
      this.draggingWord = null
      this.dragOverIndex = null
    },
    // 触摸事件处理
    onTouchStart(event, card) {
      this.touchDragging = true
      this.draggingWord = card
      this.submitted = false
      
      const touch = event.touches[0]
      this.touchStartX = touch.clientX
      this.touchStartY = touch.clientY
      this.touchGhostX = touch.clientX - 40 // 居中偏移
      this.touchGhostY = touch.clientY - 20
    },
    onTouchStartFromSlot(event, animalKey) {
      // 从槽位触摸拖动标签
      const word = this.selectedWordByAnimal[animalKey]
      if (!word) return
      
      this.touchDragging = true
      this.draggingWord = word
      this.submitted = false
      
      // 立即从槽位移除
      delete this.selectedWordByAnimal[animalKey]
      
      const touch = event.touches[0]
      this.touchStartX = touch.clientX
      this.touchStartY = touch.clientY
      this.touchGhostX = touch.clientX - 40
      this.touchGhostY = touch.clientY - 20
    },
    onTouchMove(event) {
      if (!this.touchDragging) return
      
      const touch = event.touches[0]
      this.touchGhostX = touch.clientX - 40
      this.touchGhostY = touch.clientY - 20
      
      // 检测是否在某个动物槽位上
      const hoveredSlotIndex = this.getSlotIndexAtPoint(touch.clientX, touch.clientY)
      this.dragOverIndex = hoveredSlotIndex
      
      // 检测是否靠近卡片区域（磁吸提示）
      this.isNearCardsArea = this.isInCardsArea(touch.clientX, touch.clientY)
    },
    onTouchEnd(event) {
      if (!this.touchDragging) return
      
      const touch = event.changedTouches[0]
      const x = touch.clientX
      const y = touch.clientY
      
      // 检查是否在卡片区域（磁吸效果）
      const inCardsArea = this.isInCardsArea(x, y)
      
      // 检查是否在动物槽位
      const slotIndex = this.getSlotIndexAtPoint(x, y)
      
      if (slotIndex !== null && slotIndex >= 0 && slotIndex < this.animals.length) {
        // 放到动物槽位
        const animalKey = this.animals[slotIndex].key
        this.onDrop(animalKey)
      } else if (inCardsArea) {
        // 磁吸回卡片区域 - 不放到任何动物上，标签会自动回到底部
        // 不需要额外处理，因为 shuffledCards 会自动包含未分配的卡片
      } else {
        // 没有放到任何地方，也不在卡片区域，标签会自动回到底部
      }
      
      this.touchDragging = false
      this.draggingWord = null
      this.dragOverIndex = null
      this.isNearCardsArea = false
    },
    getSlotIndexAtPoint(x, y) {
      // 遍历所有动物槽位，检查坐标是否在其中
      for (let i = 0; i < this.animals.length; i++) {
        const slotRef = this.$refs['slot-' + i]
        if (slotRef && slotRef[0]) {
          const rect = slotRef[0].getBoundingClientRect()
          if (x >= rect.left && x <= rect.right && y >= rect.top && y <= rect.bottom) {
            return i
          }
        }
      }
      return null
    },
    isInCardsArea(x, y) {
      // 检查是否在卡片区域内（带扩展磁吸范围）
      const cardsArea = this.$refs.cardsArea
      if (!cardsArea) return false
      
      const rect = cardsArea.getBoundingClientRect()
      const magnetRange = 80 // 磁吸范围（像素）
      
      return (
        x >= rect.left - magnetRange &&
        x <= rect.right + magnetRange &&
        y >= rect.top - magnetRange &&
        y <= rect.bottom + magnetRange
      )
    },
    isCorrect(animalKey) {
      return this.selectedWordByAnimal[animalKey] === animalKey
    },
    onSubmit() {
      this.submitted = true
      if (this.allCorrect) {
        // play cheer
        const a = this.$refs.cheerAudio
        if (a && a.play) a.play().catch(() => {})
        // trigger bounce animation by toggling class on root
        this.$el.classList.remove('bounce-all')
        // next tick to reflow
        requestAnimationFrame(() => {
          this.$el.classList.add('bounce-all')
        })
      } else {
        // Flash wrong ones via CSS class
        this.$el.classList.remove('flash-wrong')
        requestAnimationFrame(() => {
          this.$el.classList.add('flash-wrong')
        })
      }
    },
    onRetry() {
      // 重置游戏状态
      this.selectedWordByAnimal = {}
      this.submitted = false
      this.draggingWord = null
      this.dragOverIndex = null
    },
    onNext() {
      // 通知父组件切换到下一关
      this.$emit('next-level')
    }
  }
}
</script>

<style scoped>
.level-one {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 16px;
}
.top-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 12px;
  width: 100%;
  max-width: 720px;
}
.animal-slot {
  position: relative;
  border: 2px dashed #ccc;
  border-radius: 12px;
  padding: 12px;
  min-height: 120px;
  background: #fafafa;
  transition: box-shadow .2s ease, transform .2s ease, border-color .2s ease;
}
.animal-slot.drop-target {
  box-shadow: 0 0 0 3px #a0e7e5 inset;
  border-color: #30b0ad;
}
.animal-img {
  font-size: 72px;
}
.animal-name {
  margin-top: 6px;
  color: #666;
}
.attached-word {
  margin-top: 8px;
  padding: 4px 10px;
  background: #fff;
  border: 1px solid #ddd;
  border-radius: 8px;
  display: inline-block;
  cursor: grab;
  user-select: none;
  touch-action: none;
  transition: transform 0.2s ease, box-shadow 0.2s ease;
}

.attached-word:hover {
  transform: scale(1.05);
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.15);
}

.attached-word:active {
  cursor: grabbing;
}
.result-mark {
  position: absolute;
  right: 8px;
  top: 8px;
  font-size: 20px;
}
.ok { color: #2ecc71; }
.err { color: #e74c3c; }

.cards {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  justify-content: center;
  max-width: 720px;
  padding: 16px;
  border-radius: 12px;
  transition: all 0.3s ease;
}

.cards.magnet-active {
  background: rgba(66, 185, 131, 0.1);
  box-shadow: 0 0 0 3px rgba(66, 185, 131, 0.3) inset;
  transform: scale(1.02);
}
.word-card {
  user-select: none;
  padding: 8px 12px;
  background: #f0f9ff;
  border: 1px solid #b6e0fe;
  border-radius: 8px;
  cursor: grab;
  touch-action: none; /* 防止触摸时的默认行为 */
}
.touch-drag-ghost {
  position: fixed;
  pointer-events: none;
  user-select: none;
  padding: 8px 12px;
  background: #f0f9ff;
  border: 2px solid #42b983;
  border-radius: 8px;
  z-index: 1000;
  opacity: 0.9;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
  transform: scale(1.1);
}
.button-group {
  display: flex;
  gap: 12px;
  justify-content: center;
  align-items: center;
}

.submit-btn, .retry-btn, .next-btn {
  padding: 10px 18px;
  border: none;
  color: #fff;
  border-radius: 8px;
  cursor: pointer;
  font-size: 16px;
  transition: all 0.3s ease;
}

.submit-btn {
  background: #42b983;
}

.submit-btn:hover {
  background: #38a372;
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(66, 185, 131, 0.3);
}

.retry-btn {
  background: #e67e22;
}

.retry-btn:hover {
  background: #d35400;
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(230, 126, 34, 0.3);
}

.next-btn {
  background: #3498db;
}

.next-btn:hover {
  background: #2980b9;
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(52, 152, 219, 0.3);
}

.celebration {
  font-size: 24px;
  color: #ff8c00;
  animation: happy 0.8s ease both;
}
@keyframes happy {
  0% { transform: scale(0.8); opacity: 0; }
  100% { transform: scale(1); opacity: 1; }
}
.bounce-all .animal-slot, .bounce-all .word-card {
  animation: bounce 0.8s ease;
}
@keyframes bounce {
  0%, 100% { transform: translateY(0); }
  30% { transform: translateY(-8px); }
  60% { transform: translateY(-4px); }
}
.flash-wrong .locked-wrong {
  animation: flash 0.6s ease 0s 2;
}
@keyframes flash {
  0%, 100% { box-shadow: 0 0 0 0 rgba(231,76,60,0); }
  50% { box-shadow: 0 0 0 3px rgba(231,76,60,0.5) inset; }
}

/* 移动端响应式优化 */
@media (max-width: 768px) {
  .top-grid {
    grid-template-columns: repeat(2, 1fr);
    gap: 8px;
    max-width: 100%;
  }
  
  .animal-slot {
    min-height: 100px;
    padding: 8px;
  }
  
  .animal-img {
    font-size: 56px;
  }
  
  .cards {
    gap: 8px;
    max-width: 100%;
  }
  
  .word-card {
    padding: 10px 14px;
    font-size: 16px;
  }
  
  .touch-drag-ghost {
    padding: 10px 14px;
    font-size: 16px;
  }
  
  h2 {
    font-size: 24px;
  }
}

@media (max-width: 480px) {
  .animal-img {
    font-size: 48px;
  }
  
  .animal-slot {
    min-height: 90px;
    padding: 6px;
  }
}
</style>

