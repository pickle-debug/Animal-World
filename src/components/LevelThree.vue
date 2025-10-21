<template>
  <div class="level-three">
    <h2>Show & Tell!</h2>
    <div class="stage">
      <div class="animal-picker">
        <button
          v-for="a in animals"
          :key="a.key"
          :class="['pick', { selected: selected === a.key }]"
          @click="selectAnimal(a.key)"
        >{{ a.emoji }}</button>
      </div>
      <div class="prompt">
        <div>My favorite animal is ____ .</div>
        <div>It's ____ (big/small/cute).</div>
      </div>
      <div class="center-view">
        <div class="chosen">{{ currentEmoji }}</div>
      </div>
      <button class="mic" @click="toggleRecord">
        <span v-if="!recording">🎤 开始</span>
        <span v-else>⏹ 结束</span>
      </button>
      <div class="tips">点击麦克风后将提示“准备，开始！”。完成后自动给予积极反馈。</div>
    </div>

    <transition name="celebrate">
      <div v-if="showPraise" class="praise">
        Excellent! 🏅
      </div>
    </transition>
    <div class="fireworks" v-if="showPraise">
      <div class="spark" v-for="n in 12" :key="n" :style="sparkStyle(n)"></div>
    </div>
    <audio ref="cheerAudio">
      <source src="https://assets.mixkit.co/active_storage/sfx/1392/1392-preview.mp3" type="audio/mpeg" />
    </audio>
  </div>
</template>

<script>
export default {
  name: 'LevelThree',
  data() {
    return {
      animals: [
        { key: 'lion', emoji: '🦁' },
        { key: 'elephant', emoji: '🐘' },
        { key: 'giraffe', emoji: '🦒' },
        { key: 'zebra', emoji: '🦓' },
        { key: 'monkey', emoji: '🐒' },
        { key: 'kangaroo', emoji: '🦘' }
      ],
      selected: 'lion',
      recording: false,
      mediaRecorder: null,
      showPraise: false
    }
  },
  computed: {
    currentEmoji() {
      const f = this.animals.find(a => a.key === this.selected)
      return f ? f.emoji : '❓'
    }
  },
  methods: {
    selectAnimal(k) { this.selected = k },
    async toggleRecord() {
      if (!this.recording) {
        // start
        try {
          alert('准备，开始！')
          const stream = await navigator.mediaDevices.getUserMedia({ audio: true })
          this.mediaRecorder = new MediaRecorder(stream)
          this.mediaRecorder.start()
          this.recording = true
        } catch (e) {
          console.error(e)
          alert('需要麦克风权限才能录音')
        }
      } else {
        // stop
        if (this.mediaRecorder) {
          this.mediaRecorder.stop()
          this.mediaRecorder.stream.getTracks().forEach(t => t.stop())
          this.mediaRecorder = null
        }
        this.recording = false
        this.praise()
      }
    },
    praise() {
      this.showPraise = true
      const a = this.$refs.cheerAudio
      if (a && a.play) a.play().catch(() => {})
      setTimeout(() => { this.showPraise = false }, 1500)
    },
    sparkStyle(n) {
      const angle = (360 / 12) * (n - 1)
      return {
        transform: `rotate(${angle}deg) translateY(-40px)`
      }
    }
  }
}
</script>

<style scoped>
.level-three { display: flex; flex-direction: column; align-items: center; gap: 12px; }
.stage { position: relative; width: 100%; max-width: 720px; background: linear-gradient(180deg,#111 0%,#333 40%,#222 100%); color: #fff; border-radius: 16px; padding: 16px; box-shadow: 0 8px 24px rgba(0,0,0,0.35); }
.animal-picker { display: flex; gap: 8px; justify-content: center; flex-wrap: wrap; }
.pick { width: 40px; height: 40px; border-radius: 8px; border: 1px solid #555; background: #222; color: #fff; cursor: pointer; }
.pick.selected { outline: 2px solid #42b983; }
.prompt { text-align: center; margin: 10px 0; color: #ffd; }
.center-view { display: flex; justify-content: center; align-items: center; height: 180px; }
.chosen { font-size: 96px; }
.mic { padding: 10px 16px; border: none; background: #ff4081; color: #fff; border-radius: 10px; cursor: pointer; }
.tips { margin-top: 8px; color: #ddd; font-size: 12px; }
.praise { position: absolute; top: 16px; right: 16px; background: #ffc107; color: #333; padding: 8px 12px; border-radius: 10px; font-weight: bold; }
.celebrate-enter-active, .celebrate-leave-active { transition: opacity .3s, transform .3s; }
.celebrate-enter-from, .celebrate-leave-to { opacity: 0; transform: scale(0.9); }
.fireworks { position: absolute; left: 50%; top: 50%; width: 0; height: 0; }
.spark { position: absolute; left: -2px; top: -2px; width: 4px; height: 12px; background: #ffea00; border-radius: 2px; animation: burst 0.8s ease forwards; }
@keyframes burst { 0% { opacity: 0; transform: scale(0.5); } 100% { opacity: 1; transform: scale(1); } }
</style>

