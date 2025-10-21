<template>
  <div class="level-one">
    <h2>Match & Meet!</h2>
    <div class="top-grid">
      <div
        v-for="(animal, idx) in animals"
        :key="animal.key"
        class="animal-slot"
        :class="{ 'drop-target': dragOverIndex === idx, 'locked-correct': submitted && isCorrect(animal.key), 'locked-wrong': submitted && !isCorrect(animal.key) && selectedWordByAnimal[animal.key] }"
        @dragover.prevent="onDragOver(idx)"
        @dragleave.prevent="onDragLeave"
        @drop.prevent="onDrop(animal.key)"
      >
        <div class="animal-img" :aria-label="animal.name">
          {{ animal.emoji }}
        </div>
        <div class="animal-name">{{ animal.nameZh }}</div>
        <transition name="snap">
          <div v-if="selectedWordByAnimal[animal.key]" class="attached-word">
            {{ selectedWordByAnimal[animal.key] }}
          </div>
        </transition>
        <div v-if="submitted" class="result-mark">
          <span v-if="isCorrect(animal.key)" class="ok">✔</span>
          <span v-else-if="selectedWordByAnimal[animal.key]" class="err">✖</span>
        </div>
      </div>
    </div>

    <div class="cards">
      <div
        v-for="card in shuffledCards"
        :key="card"
        class="word-card"
        draggable="true"
        @dragstart="onDragStart(card)"
      >
        {{ card }}
      </div>
    </div>

    <button class="submit-btn" @click="onSubmit">提交</button>

    <transition name="celebrate">
      <div v-if="submitted && allCorrect" class="celebration">
        Great Job! 🎉
      </div>
    </transition>
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
      submitted: false
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
  font-size: 48px;
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
}
.word-card {
  user-select: none;
  padding: 8px 12px;
  background: #f0f9ff;
  border: 1px solid #b6e0fe;
  border-radius: 8px;
  cursor: grab;
}
.submit-btn {
  padding: 10px 18px;
  border: none;
  background: #42b983;
  color: #fff;
  border-radius: 8px;
  cursor: pointer;
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
</style>

