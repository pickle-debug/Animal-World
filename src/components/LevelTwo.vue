<template>
  <div class="level-two" :class="{ 'shake-screen': shakeScreen }">
    <h2>Who Am I?</h2>
    
    <div class="question-mark" v-if="!answered || !isCorrectChoice">?</div>
    <div class="animal-reveal" v-else>
      <div class="animal-img">{{ currentRound.emoji }}</div>
      <div class="animal-name">{{ currentRound.answer }}</div>
    </div>
    
    <div class="clues">
      <div v-for="(line, i) in displayedClues" :key="i" class="clue-line">{{ line }}</div>
    </div>
    <div class="options">
      <button
        v-for="opt in currentRound.options"
        :key="opt"
        :class="['opt-btn', { 'wrong-choice': wrongChoices.includes(opt) }]"
        :disabled="wrongChoices.includes(opt) || (answered && isCorrectChoice)"
        @click="onChoose(opt)"
      >{{ opt }}</button>
    </div>
    <div class="toolbar">
      <button class="hint-btn" :disabled="hintUsed || answered" @click="showHint">提示</button>
      <div class="progress-badge">{{ correctCount }}/6</div>
      <div class="timer" :class="{ urgent: secondsLeft <= 5 }">{{ secondsLeft }}s</div>
      <button class="skip-btn" @click="skipRound">换一题</button>
    </div>

    <!-- 答对时的 Correct! 提示 -->
    <transition name="correct-appear">
      <div v-if="answered && isCorrectChoice && showConfetti" class="correct-overlay">
        <div class="correct-text">Correct!</div>
      </div>
    </transition>

    <!-- 撒花动画 -->
    <div v-if="showConfetti" class="confetti-container">
      <div v-for="n in 30" :key="n" class="confetti" :style="confettiStyle()"></div>
    </div>

    <!-- 下一题按钮 -->
    <div v-if="answered && isCorrectChoice && !finished" class="next-round-section">
      <button class="next-btn" @click="nextRound">下一题</button>
    </div>

    <!-- 完成后的下一关按钮 -->
    <div v-if="finished" class="finished">
      <button class="next-level-btn" @click="goToNextLevel">下一关 →</button>
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
          answer: 'monkey',
          emoji: '🐒',
          clues: ['I can climb trees.', 'I like bananas.']
        },
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
          answer: 'tiger',
          emoji: '🐯',
          clues: ['I have orange and black stripes.', 'I am a big cat.']
        },
        {
          answer: 'lion',
          emoji: '🦁',
          clues: ['I am the king of the jungle.', 'I have a big mane.']
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
      finished: false,
      wrongChoices: [],
      correctCount: 0,
      showConfetti: false
    }
  },
  computed: {
    visibleClues() {
      return this.currentRound.clues.slice(0, this.clueIndex)
    },
    displayedClues() {
      // 答对时显示所有线索，否则显示可见的线索
      if (this.answered && this.isCorrectChoice) {
        return this.currentRound.clues
      }
      return this.visibleClues
    },
    isCorrectChoice() {
      return this.chosen === this.currentRound.answer
    },
    shakeScreen() {
      return !this.isCorrectChoice && this.wrongChoices.length > 0
    }
  },
  methods: {
    startRound() {
      // pick one by order/roundIndex
      const currentAnswer = this.order[this.roundIndex]
      const item = this.pool.find(x => x.answer === currentAnswer)
      // 使用所有动物作为选项
      const options = this.pool.map(x => x.answer).sort(() => Math.random() - 0.5)
      this.currentRound = { answer: item.answer, emoji: item.emoji, clues: [...item.clues], options }
      this.clueIndex = 1
      this.secondsLeft = 30
      this.answered = false
      this.chosen = ''
      this.hintUsed = false
      this.wrongChoices = []
      this.showConfetti = false
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
      if (this.wrongChoices.includes(opt)) return
      
      this.chosen = opt
      
      // 检查是否正确
      if (opt === this.currentRound.answer) {
        // 答对了
        this.answered = true
        this.correctCount++
        if (this.timerId) {
          clearInterval(this.timerId)
          this.timerId = null
        }
        
        // 显示撒花动画
        this.showConfetti = true
        setTimeout(() => {
          this.showConfetti = false
        }, 2000)
      } else {
        // 答错了
        this.wrongChoices.push(opt)
        
        // 震动反馈（如果支持）
        if (navigator.vibrate) {
          navigator.vibrate(200)
        }
        
        // 界面变红摇晃
        this.$el.classList.add('flash-red')
        setTimeout(() => {
          this.$el.classList.remove('flash-red')
        }, 600)
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
      this.correctCount = 0
      this.buildOrder()
      this.startRound()
    },
    goToNextLevel() {
      // 发送事件到父组件切换到第三关
      this.$emit('next-level')
    },
    confettiStyle() {
      const colors = ['#ff6b6b', '#4ecdc4', '#45b7d1', '#f9ca24', '#6c5ce7', '#a29bfe']
      const randomColor = colors[Math.floor(Math.random() * colors.length)]
      const randomLeft = Math.random() * 100
      const randomDelay = Math.random() * 0.5
      const randomDuration = 2 + Math.random() * 2
      
      return {
        left: randomLeft + '%',
        backgroundColor: randomColor,
        animationDelay: randomDelay + 's',
        animationDuration: randomDuration + 's'
      }
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
.level-two { 
  display: flex; 
  flex-direction: column; 
  align-items: center; 
  gap: 12px;
  position: relative;
  transition: background-color 0.3s ease;
}

.question-mark { 
  font-size: 96px; 
  margin-top: 8px;
  animation: shake-question 0.5s ease;
}
.level-two.shake-screen .question-mark {
  animation: shake-question 0.5s ease infinite;
}
@keyframes shake-question {
  0%, 100% { transform: rotate(0deg); }
  25% { transform: rotate(-10deg); }
  75% { transform: rotate(10deg); }
}

/* 动物显示 */
.animal-reveal {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
  animation: reveal-fade 0.5s ease;
}
.animal-reveal .animal-img {
  font-size: 96px;
}
.animal-reveal .animal-name {
  font-size: 24px;
  font-weight: bold;
  color: #42b983;
  text-transform: capitalize;
}
@keyframes reveal-fade {
  from {
    opacity: 0;
    transform: scale(0.8);
  }
  to {
    opacity: 1;
    transform: scale(1);
  }
}

.clues { 
  min-height: 80px;
  text-align: center;
}
.clue-line { 
  margin: 4px 0;
  font-size: 16px;
  color: #555;
}
.options { 
  display: flex; 
  gap: 10px; 
  flex-wrap: wrap; 
  justify-content: center; 
  max-width: 600px;
  margin: 0 auto;
}
.opt-btn { 
  padding: 10px 16px; 
  border-radius: 8px; 
  border: 1px solid #ddd; 
  background: #fff; 
  cursor: pointer;
  font-size: 16px;
  transition: all 0.2s ease;
  min-width: 120px;
}
.opt-btn:hover:not(:disabled) {
  background: #f0f9ff;
  border-color: #42b983;
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}
.opt-btn:disabled { 
  opacity: 0.6; 
  cursor: not-allowed; 
}
.opt-btn.wrong-choice {
  background: #ccc !important;
  color: #888 !important;
  border-color: #999 !important;
  cursor: not-allowed !important;
}

/* 界面变红效果 */
.level-two.flash-red {
  animation: flash-red 0.6s ease;
}
@keyframes flash-red {
  0%, 100% { background-color: transparent; }
  50% { background-color: rgba(231, 76, 60, 0.2); }
}

.toolbar { 
  display: flex; 
  align-items: center; 
  gap: 12px;
  justify-content: center;
}
.hint-btn { 
  padding: 6px 12px; 
  border: none; 
  background: #ffcc00; 
  border-radius: 6px; 
  cursor: pointer;
  transition: all 0.2s ease;
}
.hint-btn:hover:not(:disabled) {
  background: #ffd700;
  transform: translateY(-2px);
}
.hint-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}
.progress-badge {
  padding: 4px 12px;
  background: #42b983;
  color: white;
  border-radius: 12px;
  font-weight: bold;
  font-size: 14px;
  box-shadow: 0 2px 4px rgba(66, 185, 131, 0.3);
}
.skip-btn {
  padding: 6px 12px;
  border: 1px solid #ddd;
  background: #fff;
  border-radius: 6px;
  cursor: pointer;
  transition: all 0.2s ease;
}
.skip-btn:hover {
  background: #f5f5f5;
  transform: translateY(-2px);
}
.timer { 
  font-weight: bold;
  min-width: 45px;
  text-align: center;
}
.timer.urgent { color: #e74c3c; }

/* Correct! 提示 */
.correct-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  display: flex;
  justify-content: center;
  align-items: center;
  pointer-events: none;
  z-index: 1000;
}
.correct-text {
  font-size: 72px;
  font-weight: bold;
  color: #42b983;
  text-shadow: 3px 3px 6px rgba(0, 0, 0, 0.2);
  animation: correct-bounce 0.6s ease;
}
@keyframes correct-bounce {
  0%, 100% { transform: scale(1); }
  50% { transform: scale(1.2); }
}
.correct-appear-enter-active,
.correct-appear-leave-active {
  transition: opacity 0.5s ease;
}
.correct-appear-enter-from,
.correct-appear-leave-to {
  opacity: 0;
}

/* 撒花动画 */
.confetti-container {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  pointer-events: none;
  z-index: 999;
}
.confetti {
  position: absolute;
  width: 10px;
  height: 10px;
  top: -10px;
  animation: confetti-fall 3s linear forwards;
}
@keyframes confetti-fall {
  to {
    transform: translateY(100vh) rotate(360deg);
    opacity: 0;
  }
}

/* 按钮样式 */
.next-round-section {
  margin-top: 20px;
}
.next-btn {
  padding: 12px 24px;
  border: none;
  background: #42b983;
  color: #fff;
  border-radius: 8px;
  font-size: 18px;
  cursor: pointer;
  transition: all 0.3s ease;
}
.next-btn:hover {
  background: #38a372;
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(66, 185, 131, 0.3);
}

.finished {
  margin-top: 30px;
}
.next-level-btn {
  padding: 14px 32px;
  border: none;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: #fff;
  border-radius: 12px;
  font-size: 20px;
  font-weight: bold;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 4px 15px rgba(102, 126, 234, 0.4);
}
.next-level-btn:hover {
  transform: translateY(-3px);
  box-shadow: 0 6px 20px rgba(102, 126, 234, 0.6);
}
</style>

