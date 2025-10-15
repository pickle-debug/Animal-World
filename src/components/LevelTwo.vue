<template>
  <div class="level-two">
    <h2>神秘动物猜猜看 (Who Am I?)</h2>
    <div class="question-mark">?</div>
    <div class="clues">
      <div v-for="(line, i) in visibleClues" :key="i" class="clue-line">{{ line }}</div>
    </div>
    <div class="options">
      <button
        v-for="opt in currentRound.options"
        :key="opt"
        class="opt-btn"
        :disabled="answered"
        @click="onChoose(opt)"
      >{{ opt }}</button>
    </div>
    <div class="toolbar">
      <button class="hint-btn" :disabled="hintUsed || answered" @click="showHint">提示</button>
      <div class="timer" :class="{ urgent: secondsLeft <= 5 }">{{ secondsLeft }}s</div>
      <button class="skip-btn" @click="skipRound">换一题</button>
    </div>

    <transition name="reveal">
      <div v-if="answered && isCorrectChoice" class="reveal">
        <div class="animal-img">{{ currentRound.emoji }}</div>
        <div class="answer-word">{{ currentRound.answer }}</div>
        <ul class="key-points">
          <li v-for="(c, i) in currentRound.clues" :key="i">{{ c }}</li>
        </ul>
        <div class="correct-msg">Correct!</div>
        <button class="next-btn" @click="nextRound">下一题</button>
      </div>
    </transition>

    <div v-if="answered && !isCorrectChoice" class="try-again">Try again!</div>

    <div v-if="finished" class="finished">
      四题完成！
      <button class="restart-btn" @click="restart">再来一轮</button>
    </div>
  </div>
</template>

<script>
export default {
  name: 'LevelTwo',
  data() {
    return {
      pool: [
        {
          answer: 'elephant',
          emoji: '🐘',
          clues: ['I have a long nose.', 'I am very big.']
        },
        {
          answer: 'giraffe',
          emoji: '🦒',
          clues: ['I have a long neck.', 'I am very tall.']
        },
        {
          answer: 'zebra',
          emoji: '🦓',
          clues: ['I am black and white.', 'I look like a horse.']
        },
        {
          answer: 'monkey',
          emoji: '🐒',
          clues: ['I can climb trees.', 'I like bananas.']
        }
      ],
      currentRound: { answer: '', emoji: '', clues: [], options: [] },
      clueIndex: 1,
      secondsLeft: 30,
      timerId: null,
      answered: false,
      chosen: '',
      hintUsed: false,
      order: [],
      roundIndex: 0,
      finished: false
    }
  },
  computed: {
    visibleClues() {
      return this.currentRound.clues.slice(0, this.clueIndex)
    },
    isCorrectChoice() {
      return this.chosen === this.currentRound.answer
    }
  },
  methods: {
    startRound() {
      // pick one by order/roundIndex
      const currentAnswer = this.order[this.roundIndex]
      const item = this.pool.find(x => x.answer === currentAnswer)
      const others = this.pool.filter(x => x.answer !== item.answer)
      const distractors = others.sort(() => Math.random() - 0.5).slice(0, 3).map(x => x.answer)
      const options = [...distractors, item.answer].sort(() => Math.random() - 0.5)
      this.currentRound = { answer: item.answer, emoji: item.emoji, clues: [...item.clues], options }
      this.clueIndex = 1
      this.secondsLeft = 30
      this.answered = false
      this.chosen = ''
      this.hintUsed = false
      this.startTimer()
    },
    buildOrder() {
      this.order = this.pool.map(x => x.answer).sort(() => Math.random() - 0.5)
      this.roundIndex = 0
      this.finished = false
    },
    startTimer() {
      if (this.timerId) clearInterval(this.timerId)
      this.timerId = setInterval(() => {
        if (this.secondsLeft > 0) {
          this.secondsLeft--
          // progressively reveal up to 3 clues
          if (this.secondsLeft % 10 === 0 && this.clueIndex < Math.min(3, this.currentRound.clues.length)) {
            this.clueIndex++
          }
        } else {
          clearInterval(this.timerId)
          this.timerId = null
          this.answered = true
        }
      }, 1000)
    },
    onChoose(opt) {
      if (this.answered) return
      this.chosen = opt
      this.answered = true
      if (this.timerId) {
        clearInterval(this.timerId)
        this.timerId = null
      }
      if (!this.isCorrectChoice) {
        // small shake effect
        this.$el.classList.remove('shake')
        requestAnimationFrame(() => this.$el.classList.add('shake'))
      }
    },
    showHint() {
      if (this.hintUsed) return
      if (this.clueIndex < Math.min(3, this.currentRound.clues.length)) this.clueIndex++
      this.hintUsed = true
    },
    nextRound() {
      if (!this.isCorrectChoice) return
      if (this.roundIndex < this.order.length - 1) {
        this.roundIndex++
        this.startRound()
      } else {
        this.finished = true
      }
    },
    skipRound() {
      if (this.roundIndex < this.order.length - 1) {
        this.roundIndex++
        this.startRound()
      } else {
        this.finished = true
      }
    },
    restart() {
      this.buildOrder()
      this.startRound()
    }
  },
  mounted() {
    this.buildOrder()
    this.startRound()
  },
  beforeUnmount() {
    if (this.timerId) clearInterval(this.timerId)
  }
}
</script>

<style scoped>
.level-two { display: flex; flex-direction: column; align-items: center; gap: 12px; }
.question-mark { font-size: 72px; margin-top: 8px; }
.clues { min-height: 80px; }
.clue-line { margin: 4px 0; }
.options { display: flex; gap: 10px; flex-wrap: wrap; justify-content: center; }
.opt-btn { padding: 8px 12px; border-radius: 8px; border: 1px solid #ddd; background: #fff; cursor: pointer; }
.opt-btn:disabled { opacity: 0.6; cursor: not-allowed; }
.toolbar { display: flex; align-items: center; gap: 12px; }
.hint-btn { padding: 6px 12px; border: none; background: #ffcc00; border-radius: 6px; cursor: pointer; }
.timer { font-weight: bold; }
.timer.urgent { color: #e74c3c; }
.reveal { display: flex; flex-direction: column; align-items: center; gap: 8px; }
.animal-img { font-size: 64px; }
.answer-word { font-size: 20px; color: #2c3e50; }
.key-points { text-align: left; }
.try-again { color: #e67e22; font-weight: bold; }
.shake { animation: shake 0.4s ease; }
@keyframes shake { 0%,100%{transform:translateX(0)} 25%{transform:translateX(-6px)} 75%{transform:translateX(6px)} }
</style>

