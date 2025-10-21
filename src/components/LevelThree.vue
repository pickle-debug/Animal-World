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
        <div v-for="(line, i) in currentPromptLines" :key="i" class="prompt-line">
          {{ line }}
        </div>
      </div>
      <div class="center-view">
        <div class="chosen-wrapper">
          <div class="chosen">{{ currentEmoji }}</div>
          <transition name="thinking-fade">
            <div v-if="recording" class="thinking-bubble">
              💭
            </div>
          </transition>
        </div>
      </div>
      <div class="controls">
        <button class="example-btn" @click="playExample" :disabled="playing">
          <span v-if="!playing">🔊 播放示例</span>
          <span v-else>🔊 播放中...</span>
        </button>
        <button 
          :class="['mic', { 'recording': recording }]"
          @click="onRecordClick"
          @mousedown="onRecordMouseDown"
          @mouseup="onRecordMouseUp"
          @mouseleave="onRecordMouseUp"
          @touchstart="onRecordTouchStart"
          @touchend="onRecordTouchEnd"
          @touchcancel="onRecordTouchEnd"
        >
          <span v-if="!recording">🎤 点按/长按录音</span>
          <span v-else>⏹ {{ recordingTimeText }}</span>
        </button>
        <button 
          v-if="recordedAudioUrl" 
          class="play-recorded-btn" 
          @click="playRecordedAudio"
        >
          <span v-if="!isPlayingRecorded">▶️ 播放我的录音</span>
          <span v-else>⏸ 暂停</span>
        </button>
      </div>
      <div class="tips">
        💡 先点击"播放示例"听一遍<br>
        🎤 长按录音按钮录到手松开（最多10秒）
      </div>
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
    <audio ref="exampleAudio" @ended="onAudioEnded">
      <source :src="currentAudioUrl" type="audio/mpeg" />
    </audio>
    <audio ref="recordedAudio" @ended="onRecordedAudioEnded"></audio>

    <!-- 权限指导弹窗 -->
    <div v-if="showPermissionGuide" class="permission-modal" @click="closePermissionGuide">
      <div class="permission-content" @click.stop>
        <h3>🎤 需要麦克风权限</h3>
        <p>为了录制你的声音，我们需要访问你的麦克风。</p>
        
        <div class="browser-guide">
          <div class="guide-section">
            <h4>Chrome 浏览器：</h4>
            <ol>
              <li>点击地址栏左侧的 🔒 或 ⓘ 图标</li>
              <li>找到"麦克风"选项</li>
              <li>选择"允许"</li>
              <li>刷新页面后重试</li>
            </ol>
          </div>
          
          <div class="guide-section">
            <h4>Safari 浏览器：</h4>
            <ol>
              <li>打开 Safari 菜单 → 设置</li>
              <li>点击"网站"标签</li>
              <li>找到"麦克风"</li>
              <li>为本网站选择"允许"</li>
            </ol>
          </div>
        </div>
        
        <div class="modal-buttons">
          <button class="retry-permission-btn" @click="retryPermission">重新尝试</button>
          <button class="close-modal-btn" @click="closePermissionGuide">关闭</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'LevelThree',
  data() {
    return {
      animals: [
        { 
          key: 'tiger', 
          emoji: '🐯',
          sentence: 'My favorite animal is tiger. It is strong.',
          blank: 'My favorite animal is ____. It is ____.',
          audioUrl: 'https://funzoor2.heself.com/resources/tiger.mp3'
        },
        { 
          key: 'monkey', 
          emoji: '🐒',
          sentence: 'My favorite animal is monkey. It is very clever.',
          blank: 'My favorite animal is ____. It is very ____.',
          audioUrl: 'https://funzoor2.heself.com/resources/monkey.mp3'
        },
        { 
          key: 'fish', 
          emoji: '🐟',
          sentence: 'My favorite animal is fish. It can swim.',
          blank: 'My favorite animal is ____. It can ____.',
          audioUrl: 'https://funzoor2.heself.com/resources/fish.mp3'
        },
        { 
          key: 'lion', 
          emoji: '🦁',
          sentence: 'My favorite animal is lion. It is brave.',
          blank: 'My favorite animal is ____. It is ____.',
          audioUrl: 'https://funzoor2.heself.com/resources/lion.mp3'
        },
        { 
          key: 'bird', 
          emoji: '🐦',
          sentence: 'My favorite animal is bird. It can fly.',
          blank: 'My favorite animal is ____. It can ____.',
          audioUrl: 'https://funzoor2.heself.com/resources/bird.mp3'
        },
        { 
          key: 'rabbit', 
          emoji: '🐰',
          sentence: 'My favorite animal is rabbit. It is cute.',
          blank: 'My favorite animal is ____. It is ____.',
          audioUrl: 'https://funzoor2.heself.com/resources/rabbit.mp3'
        },
        { 
          key: 'deer', 
          emoji: '🦌',
          sentence: 'My favorite animal is deer. It is beautiful.',
          blank: 'My favorite animal is ____. It is ____.',
          audioUrl: 'https://funzoor2.heself.com/resources/deer.mp3'
        },
        { 
          key: 'panda', 
          emoji: '🐼',
          sentence: 'My favorite animal is panda. It likes bamboo.',
          blank: 'My favorite animal is ____. It likes ____.',
          audioUrl: 'https://funzoor2.heself.com/resources/panda.mp3'
        }
      ],
      selected: 'tiger',
      recording: false,
      mediaRecorder: null,
      showPraise: false,
      playing: false,
      recordingTime: 0,
      recordingTimer: null,
      maxRecordTime: 10,
      isLongPress: false,
      longPressTimer: null,
      recordedAudioUrl: null,
      recordedAudioBlob: null,
      isPlayingRecorded: false,
      hasPermission: null,
      showPermissionGuide: false
    }
  },
  computed: {
    currentEmoji() {
      const f = this.animals.find(a => a.key === this.selected)
      return f ? f.emoji : '❓'
    },
    currentAudioUrl() {
      const f = this.animals.find(a => a.key === this.selected)
      return f ? f.audioUrl : ''
    },
    currentPromptLines() {
      const f = this.animals.find(a => a.key === this.selected)
      if (!f || !f.blank) return []
      return f.blank.split('. ').map(line => line.trim()).filter(line => line)
    },
    recordingTimeText() {
      return `${this.recordingTime}s / ${this.maxRecordTime}s`
    }
  },
  methods: {
    selectAnimal(k) { 
      this.selected = k 
      // 停止当前播放的音频
      if (this.playing) {
        const audio = this.$refs.exampleAudio
        if (audio) {
          audio.pause()
          audio.currentTime = 0
        }
        this.playing = false
      }
    },
    playExample() {
      const audio = this.$refs.exampleAudio
      if (!audio) return
      
      try {
        this.playing = true
        audio.load() // 重新加载音频源
        audio.play().catch(err => {
          console.error('播放失败:', err)
          this.playing = false
          alert('音频播放失败，请检查网络连接')
        })
      } catch (e) {
        console.error('播放错误:', e)
        this.playing = false
      }
    },
    onAudioEnded() {
      this.playing = false
    },
    // 点击录音按钮（用于手动停止）
    onRecordClick() {
      // 如果正在录音，停止录音
      if (this.recording) {
        this.stopRecording()
      }
    },
    
    // 鼠标按下（桌面端长按）
    onRecordMouseDown(event) {
      if (this.recording) return
      event.preventDefault()
      this.isLongPress = true
      this.longPressTimer = setTimeout(() => {
        this.startRecording()
      }, 200) // 200ms后判定为长按
    },
    
    // 鼠标抬起（桌面端）
    onRecordMouseUp() {
      if (this.longPressTimer) {
        clearTimeout(this.longPressTimer)
        this.longPressTimer = null
      }
      
      // 如果是长按录音，松开时停止
      if (this.isLongPress && this.recording) {
        this.stopRecording()
      }
      this.isLongPress = false
    },
    
    // 触摸开始（移动端）
    onRecordTouchStart(event) {
      if (this.recording) return
      event.preventDefault()
      this.isLongPress = true
      this.longPressTimer = setTimeout(() => {
        this.startRecording()
      }, 200)
    },
    
    // 触摸结束（移动端）
    onRecordTouchEnd() {
      if (this.longPressTimer) {
        clearTimeout(this.longPressTimer)
        this.longPressTimer = null
      }
      
      // 如果还没开始录音，说明是短按，开始10秒录音
      if (!this.recording && !this.isLongPress) {
        this.startRecording()
      }
      // 如果是长按录音，松开时停止
      else if (this.isLongPress && this.recording) {
        this.stopRecording()
      }
      this.isLongPress = false
    },
    
    async startRecording() {
      try {
        const stream = await navigator.mediaDevices.getUserMedia({ audio: true })
        this.hasPermission = true
        this.mediaRecorder = new MediaRecorder(stream)
        
        const audioChunks = []
        this.mediaRecorder.ondataavailable = (event) => {
          audioChunks.push(event.data)
        }
        
        this.mediaRecorder.onstop = () => {
          // 释放之前的录音URL
          if (this.recordedAudioUrl) {
            URL.revokeObjectURL(this.recordedAudioUrl)
          }
          
          // 创建新的录音
          const audioBlob = new Blob(audioChunks, { type: 'audio/webm' })
          this.recordedAudioBlob = audioBlob
          this.recordedAudioUrl = URL.createObjectURL(audioBlob)
          
          console.log('录音完成', audioBlob)
        }
        
        this.mediaRecorder.start()
        this.recording = true
        this.recordingTime = 0
        
        // 开始计时
        this.recordingTimer = setInterval(() => {
          this.recordingTime++
          
          // 达到最大录音时长，自动停止
          if (this.recordingTime >= this.maxRecordTime) {
            this.stopRecording()
          }
        }, 1000)
        
      } catch (e) {
        console.error('麦克风权限错误:', e)
        this.hasPermission = false
        this.showPermissionGuide = true
      }
    },
    
    stopRecording() {
      if (!this.recording) return
      
      // 停止计时
      if (this.recordingTimer) {
        clearInterval(this.recordingTimer)
        this.recordingTimer = null
      }
      
      // 停止录音
      if (this.mediaRecorder && this.mediaRecorder.state !== 'inactive') {
        this.mediaRecorder.stop()
        this.mediaRecorder.stream.getTracks().forEach(t => t.stop())
      }
      
      this.recording = false
      this.recordingTime = 0
      this.mediaRecorder = null
      
      // 不再自动播放音效
      // this.praise()
    },
    praise() {
      this.showPraise = true
      const a = this.$refs.cheerAudio
      if (a && a.play) a.play().catch(() => {})
      setTimeout(() => { this.showPraise = false }, 1500)
    },
    playRecordedAudio() {
      const audio = this.$refs.recordedAudio
      if (!audio || !this.recordedAudioUrl) return
      
      try {
        // 如果正在播放，则暂停
        if (this.isPlayingRecorded && !audio.paused) {
          audio.pause()
          this.isPlayingRecorded = false
          return
        }
        
        // 如果是暂停状态，则继续播放
        if (audio.paused && audio.currentTime > 0) {
          audio.play().catch(err => {
            console.error('播放录音失败:', err)
            this.isPlayingRecorded = false
            alert('播放失败，请重试')
          })
          this.isPlayingRecorded = true
          return
        }
        
        // 首次播放
        this.isPlayingRecorded = true
        audio.src = this.recordedAudioUrl
        audio.play().catch(err => {
          console.error('播放录音失败:', err)
          this.isPlayingRecorded = false
          alert('播放失败，请重试')
        })
      } catch (e) {
        console.error('播放错误:', e)
        this.isPlayingRecorded = false
      }
    },
    onRecordedAudioEnded() {
      this.isPlayingRecorded = false
    },
    closePermissionGuide() {
      this.showPermissionGuide = false
    },
    retryPermission() {
      this.showPermissionGuide = false
      // 稍微延迟一下再尝试，给用户时间去设置
      setTimeout(() => {
        this.startRecording()
      }, 300)
    },
    sparkStyle(n) {
      const angle = (360 / 12) * (n - 1)
      return {
        transform: `rotate(${angle}deg) translateY(-40px)`
      }
    }
  },
  beforeUnmount() {
    // 清理定时器
    if (this.recordingTimer) {
      clearInterval(this.recordingTimer)
    }
    if (this.longPressTimer) {
      clearTimeout(this.longPressTimer)
    }
    // 停止录音
    if (this.mediaRecorder && this.mediaRecorder.state !== 'inactive') {
      this.mediaRecorder.stop()
      this.mediaRecorder.stream.getTracks().forEach(t => t.stop())
    }
    // 清理录音URL
    if (this.recordedAudioUrl) {
      URL.revokeObjectURL(this.recordedAudioUrl)
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
.prompt { 
  text-align: center; 
  margin: 10px 0; 
  color: #ffd; 
  min-height: 50px;
}
.prompt-line {
  margin: 4px 0;
  font-size: 18px;
  line-height: 1.5;
}
.center-view { 
  display: flex; 
  justify-content: center; 
  align-items: center; 
  height: 180px; 
}
.chosen-wrapper {
  position: relative;
  display: inline-block;
}
.chosen { 
  font-size: 96px; 
}
.thinking-bubble {
  position: absolute;
  top: -10px;
  right: -20px;
  font-size: 48px;
  animation: thinking-float 2s ease-in-out infinite;
}
@keyframes thinking-float {
  0%, 100% { 
    transform: translateY(0px) scale(1);
    opacity: 1;
  }
  50% { 
    transform: translateY(-10px) scale(1.1);
    opacity: 0.8;
  }
}
.thinking-fade-enter-active,
.thinking-fade-leave-active {
  transition: opacity 0.3s ease, transform 0.3s ease;
}
.thinking-fade-enter-from,
.thinking-fade-leave-to {
  opacity: 0;
  transform: scale(0.5);
}
.controls { 
  display: flex; 
  gap: 12px; 
  justify-content: center; 
  align-items: center;
  flex-wrap: wrap;
}
.example-btn, .mic, .play-recorded-btn { 
  padding: 10px 16px; 
  border: none; 
  color: #fff; 
  border-radius: 10px; 
  cursor: pointer;
  font-size: 14px;
  transition: all 0.3s ease;
  min-width: 120px;
}
.example-btn {
  background: #42b983;
}
.example-btn:hover:not(:disabled) {
  background: #38a372;
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(66, 185, 131, 0.3);
}
.example-btn:disabled {
  background: #7dd4b4;
  cursor: not-allowed;
  opacity: 0.7;
}
.mic { 
  background: #ff4081;
  user-select: none;
  -webkit-user-select: none;
}
.mic:hover {
  background: #e91e63;
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(255, 64, 129, 0.3);
}
.mic:active {
  transform: scale(0.95);
}
.recording {
  animation: pulse 1.5s ease-in-out infinite;
}
@keyframes pulse {
  0%, 100% { 
    box-shadow: 0 0 0 0 rgba(255, 64, 129, 0.7);
  }
  50% { 
    box-shadow: 0 0 0 10px rgba(255, 64, 129, 0);
  }
}
.play-recorded-btn {
  background: #9c27b0;
}
.play-recorded-btn:hover {
  background: #7b1fa2;
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(156, 39, 176, 0.3);
}
.tips { 
  margin-top: 8px; 
  color: #ddd; 
  font-size: 13px; 
  line-height: 1.8;
  text-align: center;
}
.praise { position: absolute; top: 16px; right: 16px; background: #ffc107; color: #333; padding: 8px 12px; border-radius: 10px; font-weight: bold; }
.celebrate-enter-active, .celebrate-leave-active { transition: opacity .3s, transform .3s; }
.celebrate-enter-from, .celebrate-leave-to { opacity: 0; transform: scale(0.9); }
.fireworks { position: absolute; left: 50%; top: 50%; width: 0; height: 0; }
.spark { position: absolute; left: -2px; top: -2px; width: 4px; height: 12px; background: #ffea00; border-radius: 2px; animation: burst 0.8s ease forwards; }
@keyframes burst { 0% { opacity: 0; transform: scale(0.5); } 100% { opacity: 1; transform: scale(1); } }

/* 权限指导弹窗 */
.permission-modal {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.7);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 2000;
  padding: 20px;
}

.permission-content {
  background: white;
  border-radius: 16px;
  padding: 30px;
  max-width: 600px;
  width: 100%;
  max-height: 80vh;
  overflow-y: auto;
  box-shadow: 0 10px 40px rgba(0, 0, 0, 0.3);
}

.permission-content h3 {
  margin: 0 0 15px 0;
  color: #2c3e50;
  font-size: 24px;
  text-align: center;
}

.permission-content > p {
  color: #666;
  text-align: center;
  margin-bottom: 25px;
  font-size: 16px;
}

.browser-guide {
  display: flex;
  flex-direction: column;
  gap: 20px;
  margin-bottom: 25px;
}

.guide-section {
  background: #f8f9fa;
  padding: 15px;
  border-radius: 10px;
  border-left: 4px solid #42b983;
}

.guide-section h4 {
  margin: 0 0 10px 0;
  color: #2c3e50;
  font-size: 16px;
}

.guide-section ol {
  margin: 0;
  padding-left: 20px;
  color: #555;
  line-height: 1.8;
}

.guide-section li {
  margin-bottom: 5px;
}

.modal-buttons {
  display: flex;
  gap: 12px;
  justify-content: center;
}

.retry-permission-btn,
.close-modal-btn {
  padding: 10px 24px;
  border: none;
  border-radius: 8px;
  font-size: 16px;
  cursor: pointer;
  transition: all 0.3s ease;
}

.retry-permission-btn {
  background: #42b983;
  color: white;
}

.retry-permission-btn:hover {
  background: #38a372;
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(66, 185, 131, 0.3);
}

.close-modal-btn {
  background: #e0e0e0;
  color: #666;
}

.close-modal-btn:hover {
  background: #d0d0d0;
  transform: translateY(-2px);
}

/* 移动端适配 */
@media (max-width: 768px) {
  .permission-content {
    padding: 20px;
    max-height: 90vh;
  }
  
  .browser-guide {
    gap: 15px;
  }
  
  .guide-section {
    padding: 12px;
  }
  
  .guide-section h4 {
    font-size: 15px;
  }
  
  .guide-section ol {
    font-size: 14px;
  }
  
  .modal-buttons {
    flex-direction: column;
  }
  
  .retry-permission-btn,
  .close-modal-btn {
    width: 100%;
  }
}
</style>

