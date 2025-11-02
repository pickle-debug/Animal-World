<template>
  <div class="level-three">
    <h2>Show & Tell!</h2>
    <!-- 右上角查看上传记录按钮 -->
    <button class="view-uploads-btn" @click="showUploadsModal = true">📋 查看上传记录</button>
    <div class="stage">
      <div class="animal-picker">
        <button
          v-for="a in animals"
          :key="a.key"
          :class="['pick', { selected: selected === a.key }]"
          @click="selectAnimal(a.key)"
        >{{ a.emoji }}</button>
        <!-- 气泡框选项 -->
        <button
          :class="['pick', 'bubble-option', { selected: selected === 'bubble' }]"
          @click="selectAnimal('bubble')"
        >
          <div class="bubble-animation">💭</div>
        </button>
      </div>
      
      <div class="center-view">
        <transition name="emoji-change" mode="out-in">
          <div :key="displayEmoji" class="chosen">{{ displayEmoji }}</div>
        </transition>
      </div>
      
      <div class="prompt-section">
        <div class="input-line">
          <span class="label">My favorite animal is </span>
          <input 
            v-model="animalInput"
            type="text"
            class="animal-input"
            placeholder="__________"
            @blur="validateAnimalInput($event)"
            @focus="clearError($event)"
          />
          <span class="label">.</span>
        </div>
        <div class="input-line">
          <span class="label">It's </span>
          <div 
            class="blank-input"
            @drop.prevent="onDrop($event, 0)"
            @dragover.prevent
            @click="clearBlank(0)"
          >
            <span v-if="blanks[0]" class="filled-word">{{ blanks[0] }}</span>
            <input 
              v-else
              v-model="blanks[0]"
              type="text"
              placeholder="__________"
            />
          </div>
          <span class="label"> / It has </span>
          <div 
            class="blank-input"
            @drop.prevent="onDrop($event, 1)"
            @dragover.prevent
            @click="clearBlank(1)"
          >
            <span v-if="blanks[1]" class="filled-word">{{ blanks[1] }}</span>
            <input 
              v-else
              v-model="blanks[1]"
              type="text"
              placeholder="__________"
            />
          </div>
          <span class="label"> / It likes </span>
          <div 
            class="blank-input"
            @drop.prevent="onDrop($event, 2)"
            @dragover.prevent
            @click="clearBlank(2)"
          >
            <span v-if="blanks[2]" class="filled-word">{{ blanks[2] }}</span>
            <input 
              v-else
              v-model="blanks[2]"
              type="text"
              placeholder="__________"
            />
          </div>
          <span class="label">.</span>
        </div>
      </div>
      
      <!-- 单词区域 -->
      <div class="words-section">
        <div class="word-bank">
          <div 
            v-for="word in wordBank" 
            :key="word" 
            class="word-chip"
            draggable="true"
            @dragstart="onDragStart($event, word)"
          >
            {{ word }}
          </div>
        </div>
      </div>
      <div class="controls">
        <button class="example-btn" @click="playExample" :disabled="playing || selected === 'bubble'">
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
        <button 
          v-if="recordedAudioUrl" 
          class="upload-btn" 
          @click="showUploadModal = true"
          :disabled="uploading"
        >
          <span v-if="!uploading">☁️ 上传录音</span>
          <span v-else>📤 上传中...</span>
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

    <!-- 上传录音弹窗 -->
    <div v-if="showUploadModal" class="upload-modal" @click="closeUploadModal">
      <div class="upload-content" @click.stop>
        <h3>☁️ 上传录音</h3>
        <div class="upload-form">
          <div class="form-group">
            <label>请输入您的姓名：</label>
            <input 
              v-model="uploaderName" 
              type="text" 
              class="name-input"
              placeholder="例如：张三"
              @keyup.enter="confirmUpload"
            />
          </div>
          <div class="upload-info">
            <p>录音文件名：{{ uploadFileName }}</p>
          </div>
          <div class="modal-buttons">
            <button class="confirm-upload-btn" @click="confirmUpload" :disabled="!uploaderName.trim() || uploading">
              <span v-if="!uploading">确定上传</span>
              <span v-else>上传中...</span>
            </button>
            <button class="cancel-upload-btn" @click="closeUploadModal" :disabled="uploading">取消</button>
          </div>
        </div>
      </div>
    </div>

    <!-- 查看上传记录弹窗 -->
    <div v-if="showUploadsModal" class="uploads-modal" @click="closeUploadsModal">
      <div class="uploads-content" @click.stop>
        <h3>📋 上传记录</h3>
        <div class="uploads-list-container">
          <div v-if="loadingUploads" class="loading">加载中...</div>
          <div v-else-if="uploadedAudios.length === 0" class="empty-list">暂无上传记录</div>
          <div v-else class="uploads-list">
            <div 
              v-for="(audio, index) in uploadedAudios" 
              :key="index" 
              class="upload-item"
            >
              <div class="upload-item-info">
                <span class="upload-name">{{ audio.name }}</span>
                <span class="upload-time">{{ audio.time }}</span>
              </div>
              <audio :src="audio.url" controls class="upload-audio"></audio>
            </div>
          </div>
        </div>
        <div class="modal-buttons">
          <button class="download-all-btn" @click="downloadAllAsZip" :disabled="uploadedAudios.length === 0 || downloading">
            <span v-if="!downloading">📦 下载所有音频 (ZIP)</span>
            <span v-else>📦 打包中...</span>
          </button>
          <button class="refresh-btn" @click="loadUploadedAudios" :disabled="loadingUploads">🔄 刷新</button>
          <button class="close-modal-btn" @click="closeUploadsModal">关闭</button>
        </div>
      </div>
    </div>

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
import { S3Client, PutObjectCommand, ListObjectsV2Command, GetObjectCommand } from '@aws-sdk/client-s3'
import JSZip from 'jszip'
import * as lamejs from '@breezystack/lamejs'

export default {
  name: 'LevelThree',
  data() {
    return {
      animals: [
        { 
          key: 'tiger', 
          emoji: '🐯',
          sentence: 'My favorite animal is tiger. It is strong.',
          blank1: 'My favorite animal is __________.',
          blank2: "It's __________ / It has __________ / It likes __________.",
          audioUrl: 'https://funzoor2.heself.com/resources/tiger.mp3'
        },
        { 
          key: 'monkey', 
          emoji: '🐒',
          sentence: 'My favorite animal is monkey. It is very clever.',
          blank1: 'My favorite animal is __________.',
          blank2: "It's __________ / It has __________ / It likes __________.",
          audioUrl: 'https://funzoor2.heself.com/resources/monkey.mp3'
        },
        { 
          key: 'fish', 
          emoji: '🐟',
          sentence: 'My favorite animal is fish. It can swim.',
          blank1: 'My favorite animal is __________.',
          blank2: "It's __________ / It has __________ / It likes __________.",
          audioUrl: 'https://funzoor2.heself.com/resources/fish.mp3'
        },
        { 
          key: 'lion', 
          emoji: '🦁',
          sentence: 'My favorite animal is lion. It is brave.',
          blank1: 'My favorite animal is __________.',
          blank2: "It's __________ / It has __________ / It likes __________.",
          audioUrl: 'https://funzoor2.heself.com/resources/lion.mp3'
        },
        { 
          key: 'bird', 
          emoji: '🐦',
          sentence: 'My favorite animal is bird. It can fly.',
          blank1: 'My favorite animal is __________.',
          blank2: "It's __________ / It has __________ / It likes __________.",
          audioUrl: 'https://funzoor2.heself.com/resources/bird.mp3'
        },
        { 
          key: 'rabbit', 
          emoji: '🐰',
          sentence: 'My favorite animal is rabbit. It is cute.',
          blank1: 'My favorite animal is __________.',
          blank2: "It's __________ / It has __________ / It likes __________.",
          audioUrl: 'https://funzoor2.heself.com/resources/rabbit.mp3'
        },
        { 
          key: 'deer', 
          emoji: '🦌',
          sentence: 'My favorite animal is deer. It is beautiful.',
          blank1: 'My favorite animal is __________.',
          blank2: "It's __________ / It has __________ / It likes __________.",
          audioUrl: 'https://funzoor2.heself.com/resources/deer.mp3'
        },
        { 
          key: 'panda', 
          emoji: '🐼',
          sentence: 'My favorite animal is panda. It likes bamboo.',
          blank1: 'My favorite animal is __________.',
          blank2: "It's __________ / It has __________ / It likes __________.",
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
      showPermissionGuide: false,
      wordBank: [
        'big', 'small', 'cute', 'clever', 'strong',
        'red', 'yellow', 'white', 'blue', 'black',
        'eyes', 'ears', 'mouth', 'nose', 'hair', 'leg', 'head', 'long', 'short'
      ],
      animalInput: '',
      blanks: ['', '', ''],
      draggingWord: null,
      animalEmojiMap: {
        'tiger': '🐯',
        'monkey': '🐒',
        'fish': '🐟',
        'lion': '🦁',
        'bird': '🐦',
        'rabbit': '🐰',
        'deer': '🦌',
        'panda': '🐼'
      },
      // 上传相关
      showUploadModal: false,
      uploaderName: '',
      uploading: false,
      showUploadsModal: false,
      uploadedAudios: [],
      loadingUploads: false,
      downloading: false,
      // R2配置 - 请替换为您的实际配置
      r2Config: {
        endpoint: 'https://cbbca4b929b2a4a0d3618894ed8f15be.r2.cloudflarestorage.com', // 您的 R2 Endpoint URL，格式: 'https://xxxxxxxxxxxx.r2.cloudflarestorage.com'
        // 获取方式：Cloudflare 控制台 → R2 → 存储桶设置 → S3 API → 查看 Endpoint
        accessKeyId: '7b01b369d533d8d412dafd2556b5b865', // 您的 R2 Access Key ID（创建 token 时显示的，格式类似长字符串）
        // 获取方式：创建 token 时与 Secret Access Key 一起显示的 Access Key ID
        secretAccessKey: 'c1f9a8260abb6d8e8f9277a7ca88f1be651b703a2212ba0ad0902899eed06af6', // 您的 R2 Secret Access Key
        bucketName: 'funzoo', // 您的存储桶名称
        region: 'auto' // R2 使用 'auto'
      }
    }
  },
  computed: {
    currentEmoji() {
      const f = this.animals.find(a => a.key === this.selected)
      return f ? f.emoji : '❓'
    },
    currentAudioUrl() {
      // 气泡框没有音频
      if (this.selected === 'bubble') return ''
      const f = this.animals.find(a => a.key === this.selected)
      return f ? f.audioUrl : ''
    },
    currentPrompt1() {
      const f = this.animals.find(a => a.key === this.selected)
      return f ? f.blank1 : ''
    },
    currentPrompt2() {
      const f = this.animals.find(a => a.key === this.selected)
      return f ? f.blank2 : ''
    },
    recordingTimeText() {
      return `${this.recordingTime}s / ${this.maxRecordTime}s`
    },
    displayEmoji() {
      // 如果选择的是气泡框
      if (this.selected === 'bubble') {
        // 检查用户输入的动物名是否正确
        const inputLower = this.animalInput.toLowerCase().trim()
        if (this.animalEmojiMap[inputLower]) {
          return this.animalEmojiMap[inputLower]
        }
        return '💭'
      }
      return this.currentEmoji
    },
    uploadFileName() {
      // 这个方法现在在 confirmUpload 中动态生成文件名
      // 保留这个方法用于显示预览文件名
      if (!this.recordedAudioBlob || !this.uploaderName.trim()) return 'recording.webm'
      const timestamp = Date.now()
      const sanitizedName = this.uploaderName.trim().replace(/[^a-zA-Z0-9\u4e00-\u9fa5_-]/g, '_')
      // 默认显示为 webm（实际可能会转换为 mp3）
      return `${sanitizedName}_${timestamp}.webm`
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
    onDragStart(event, word) {
      this.draggingWord = word
      event.dataTransfer.effectAllowed = 'move'
    },
    onDrop(event, blankIndex) {
      if (!this.draggingWord) return
      // 将单词放到对应的空白处
      this.blanks[blankIndex] = this.draggingWord
      this.draggingWord = null
    },
    validateAnimalInput(event) {
      // 如果是气泡框选项，不需要验证
      if (this.selected === 'bubble') {
        return
      }
      
      // 如果输入为空，不验证
      if (!this.animalInput.trim()) {
        return
      }
      
      const inputLower = this.animalInput.toLowerCase().trim()
      const isValid = this.animals.some(a => a.key === inputLower)
      
      if (!isValid) {
        // 输入错误 - 只影响输入框
        const inputEl = event?.target
        if (inputEl) {
          inputEl.classList.add('error')
        }
        
        // 震动反馈
        if (navigator.vibrate) {
          navigator.vibrate(200)
        }
        
        // 2秒后清除错误状态
        setTimeout(() => {
          if (inputEl) {
            inputEl.classList.remove('error')
          }
          this.animalInput = ''
        }, 2000)
      }
    },
    clearError(event) {
      // 清除输入框的错误样式
      const inputEl = event?.target
      if (inputEl) {
        inputEl.classList.remove('error')
      }
    },
    clearBlank(index) {
      // 点击已填充的单词可以清除
      if (this.blanks[index]) {
        this.blanks[index] = ''
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
    },
    closeUploadModal() {
      if (!this.uploading) {
        this.showUploadModal = false
        this.uploaderName = ''
      }
    },
    // 将 WebM 音频转换为 MP3
    async convertWebmToMp3(webmBlob) {
      try {
        // 创建 AudioContext
        const audioContext = new (window.AudioContext || window.webkitAudioContext)()
        const arrayBuffer = await webmBlob.arrayBuffer()
        const audioBuffer = await audioContext.decodeAudioData(arrayBuffer)
        
        // 获取 PCM 数据
        const sampleRate = audioBuffer.sampleRate
        const leftChannel = audioBuffer.getChannelData(0)
        const rightChannel = audioBuffer.numberOfChannels > 1 ? audioBuffer.getChannelData(1) : leftChannel
        
        // 转换为 16-bit PCM
        const samples = leftChannel.length
        const pcmLeft = new Int16Array(samples)
        const pcmRight = new Int16Array(samples)
        
        for (let i = 0; i < samples; i++) {
          pcmLeft[i] = Math.max(-32768, Math.min(32767, leftChannel[i] * 32768))
          if (audioBuffer.numberOfChannels > 1) {
            pcmRight[i] = Math.max(-32768, Math.min(32767, rightChannel[i] * 32768))
          } else {
            pcmRight[i] = pcmLeft[i]
          }
        }
        
        // 使用 lamejs 编码为 MP3
        const mp3encoder = new lamejs.Mp3Encoder(1, sampleRate, 128) // 单声道，128kbps
        const sampleBlockSize = 1152
        const mp3Data = []
        
        for (let i = 0; i < samples; i += sampleBlockSize) {
          const leftChunk = pcmLeft.subarray(i, i + sampleBlockSize)
          const rightChunk = pcmRight.subarray(i, i + sampleBlockSize)
          const mp3buf = mp3encoder.encodeBuffer(leftChunk, rightChunk)
          if (mp3buf.length > 0) {
            mp3Data.push(mp3buf)
          }
        }
        
        // 完成编码
        const mp3buf = mp3encoder.flush()
        if (mp3buf.length > 0) {
          mp3Data.push(mp3buf)
        }
        
        // 合并所有 MP3 数据
        const mp3Blob = new Blob(mp3Data, { type: 'audio/mp3' })
        return mp3Blob
      } catch (error) {
        console.error('转换 MP3 失败:', error)
        throw error
      }
    },
    
    // 使用原生 HTTP (fetch) 上传到 R2（未使用，保留以备将来实现纯 fetch 上传）
    // eslint-disable-next-line no-unused-vars
    async uploadToR2WithFetch(fileBlob, filePath) {
      const endpoint = this.r2Config.endpoint.replace(/^https?:\/\//, '').replace(/\/$/, '')
      const url = `https://${endpoint}/${this.r2Config.bucketName}/${filePath}`
      
      // 使用 fetch 直接上传（需要完整的 AWS Signature V4 实现）
      // 当前使用 AWS SDK，此方法暂未使用
      const response = await fetch(url, {
        method: 'PUT',
        headers: {
          'Content-Type': 'audio/mp3',
          'Authorization': this.getAuthHeader('PUT', `/${this.r2Config.bucketName}/${filePath}`)
        },
        body: fileBlob
      })
      
      if (!response.ok) {
        const errorText = await response.text()
        throw new Error(`上传失败: ${response.status} ${response.statusText} - ${errorText}`)
      }
      
      return response
    },
    
    // 生成 AWS Signature V4 认证头（简化版，未使用，保留以备将来实现纯 fetch 上传）
    // eslint-disable-next-line no-unused-vars
    getAuthHeader(method, path) {
      const accessKeyId = this.r2Config.accessKeyId
      const secretAccessKey = this.r2Config.secretAccessKey
      
      // 注意：这是一个简化版本，实际 AWS Signature V4 更复杂
      // 当前使用 AWS SDK 进行上传，此方法暂未使用
      // 如果要实现纯 fetch 上传，需要完整的签名算法实现
      
      // 使用 btoa 进行 base64 编码（简化方式）
      const credentials = btoa(`${accessKeyId}:${secretAccessKey}`).replace(/=+$/, '')
      
      return `AWS ${credentials}`
    },
    
    async confirmUpload() {
      if (!this.uploaderName.trim() || !this.recordedAudioBlob || this.uploading) return
      
      // 检查录音文件大小
      if (this.recordedAudioBlob.size < 1000) {
        alert('录音文件太小，可能录音失败。请重新录制。')
        return
      }
      
      // 检查R2配置
      if (!this.r2Config.endpoint || !this.r2Config.accessKeyId || !this.r2Config.secretAccessKey || !this.r2Config.bucketName) {
        alert('请先在代码中配置 Cloudflare R2 的访问信息！\n\n请在 LevelThree.vue 的 data() 中设置 r2Config 对象的值。')
        return
      }
      
      this.uploading = true
      try {
        let fileBlob = null
        let contentType = 'audio/webm'
        let fileName = ''
        
        // 步骤1: 尝试转换为 MP3
        try {
          console.log('开始转换音频为 MP3...', '原始文件大小:', this.recordedAudioBlob.size)
          fileBlob = await this.convertWebmToMp3(this.recordedAudioBlob)
          console.log('MP3 转换完成，文件大小:', fileBlob.size)
          contentType = 'audio/mp3'
          
          // 更新文件名
          const timestamp = Date.now()
          const sanitizedName = this.uploaderName.trim().replace(/[^a-zA-Z0-9\u4e00-\u9fa5_-]/g, '_')
          fileName = `${sanitizedName}_${timestamp}.mp3`
        } catch (conversionError) {
          console.warn('MP3 转换失败，使用原始 WebM 格式:', conversionError)
          // 降级：直接使用 WebM
          fileBlob = this.recordedAudioBlob
          contentType = 'audio/webm'
          
          // 更新文件名
          const timestamp = Date.now()
          const sanitizedName = this.uploaderName.trim().replace(/[^a-zA-Z0-9\u4e00-\u9fa5_-]/g, '_')
          fileName = `${sanitizedName}_${timestamp}.webm`
        }
        
        // 步骤2: 上传文件
        const filePath = `records/${fileName}`
        
        console.log('开始上传到 R2...', '文件类型:', contentType, '文件大小:', fileBlob.size)
        
        // 使用 AWS SDK 上传
        const s3Client = new S3Client({
          endpoint: this.r2Config.endpoint,
          region: this.r2Config.region,
          credentials: {
            accessKeyId: this.r2Config.accessKeyId,
            secretAccessKey: this.r2Config.secretAccessKey
          },
          forcePathStyle: true
        })
        
        // 将 Blob 转换为 Uint8Array
        const arrayBuffer = await fileBlob.arrayBuffer()
        const uint8Array = new Uint8Array(arrayBuffer)
        
        const uploadCommand = new PutObjectCommand({
          Bucket: this.r2Config.bucketName,
          Key: filePath,
          Body: uint8Array,
          ContentType: contentType
        })
        
        await s3Client.send(uploadCommand)
        
        alert('上传成功！')
        this.closeUploadModal()
        
        // 刷新上传列表
        if (this.showUploadsModal) {
          this.loadUploadedAudios()
        }
      } catch (error) {
        console.error('上传失败:', error)
        alert('上传失败：' + (error.message || '未知错误'))
      } finally {
        this.uploading = false
      }
    },
    async loadUploadedAudios() {
      if (!this.r2Config.endpoint || !this.r2Config.accessKeyId || !this.r2Config.secretAccessKey || !this.r2Config.bucketName) {
        alert('请先在代码中配置 Cloudflare R2 的访问信息！')
        return
      }
      
      this.loadingUploads = true
      try {
        // 配置 S3Client - R2 需要 forcePathStyle
        const s3Client = new S3Client({
          endpoint: this.r2Config.endpoint,
          region: this.r2Config.region,
          credentials: {
            accessKeyId: this.r2Config.accessKeyId,
            secretAccessKey: this.r2Config.secretAccessKey
          },
          forcePathStyle: true // R2 需要这个配置
        })
        
        const listCommand = new ListObjectsV2Command({
          Bucket: this.r2Config.bucketName,
          Prefix: 'records/' // 只列出 records 文件夹中的文件
        })
        
        const response = await s3Client.send(listCommand)
        
        this.uploadedAudios = (response.Contents || [])
          .filter(item => item.Key && item.Key.startsWith('records/') && (item.Key.endsWith('.webm') || item.Key.endsWith('.mp3')))
          .map(item => {
            // 从文件路径中提取文件名（去掉 records/ 前缀）
            const fileName = item.Key.replace(/^records\//, '')
            // 从文件名提取姓名和时间戳（支持 .webm 和 .mp3）
            const match = fileName.match(/^(.+?)_(\d+)\.(webm|mp3)$/)
            const name = match ? match[1] : fileName
            const timestamp = match ? parseInt(match[2]) : (item.LastModified ? new Date(item.LastModified).getTime() : Date.now())
            const time = new Date(timestamp).toLocaleString('zh-CN')
            const customDomain = 'https://funzoor2.heself.com'
            const url = `${customDomain}/${item.Key}`
            return {
              name,
              time,
              url,
              key: item.Key,
              size: item.Size
            }
          })
          .sort((a, b) => b.time.localeCompare(a.time))
      } catch (error) {
        console.error('加载上传记录失败:', error)
        alert('加载失败：' + (error.message || '未知错误'))
      } finally {
        this.loadingUploads = false
      }
    },
    async downloadAllAsZip() {
      if (this.uploadedAudios.length === 0 || this.downloading) return
      
      this.downloading = true
      try {
        const zip = new JSZip()
        // 配置 S3Client - R2 需要 forcePathStyle
        const s3Client = new S3Client({
          endpoint: this.r2Config.endpoint,
          region: this.r2Config.region,
          credentials: {
            accessKeyId: this.r2Config.accessKeyId,
            secretAccessKey: this.r2Config.secretAccessKey
          },
          forcePathStyle: true // R2 需要这个配置
        })
        
        // 下载所有音频文件并添加到zip
        for (const audio of this.uploadedAudios) {
          try {
            const getCommand = new GetObjectCommand({
              Bucket: this.r2Config.bucketName,
              Key: audio.key
            })
            
            const response = await s3Client.send(getCommand)
            
            // 修复浏览器环境中的流读取问题
            // 在浏览器中，AWS SDK v3 返回的 Body 可能是 ReadableStream 或其他格式
            let arrayBuffer
            try {
              // 方法1: 使用 Response API (浏览器标准方式，最可靠)
              if (response.Body) {
                // 将 Body 包装为 Response 对象，这是浏览器环境中最兼容的方式
                const responseObj = response.Body instanceof Response 
                  ? response.Body 
                  : new Response(response.Body)
                
                arrayBuffer = await responseObj.arrayBuffer()
              } else {
                throw new Error('响应体为空')
              }
            } catch (streamError) {
              console.error('读取流失败，尝试备用方法:', streamError)
              // 备用方法1: 尝试直接使用 transformToByteArray (如果可用)
              if (response.Body && typeof response.Body.transformToByteArray === 'function') {
                try {
                  arrayBuffer = await response.Body.transformToByteArray()
                } catch (e) {
                  console.error('transformToByteArray 失败:', e)
                  throw streamError
                }
              } else {
                throw streamError
              }
            }
            
            // 使用文件名而不是完整路径作为zip中的文件名
            const fileName = audio.key.replace(/^records\//, '')
            zip.file(fileName, arrayBuffer)
          } catch (error) {
            console.error(`下载 ${audio.key} 失败:`, error)
          }
        }
        
        // 生成zip文件
        const blob = await zip.generateAsync({ type: 'blob' })
        const url = URL.createObjectURL(blob)
        const a = document.createElement('a')
        a.href = url
        a.download = `录音合集_${new Date().toISOString().slice(0, 10)}.zip`
        document.body.appendChild(a)
        a.click()
        document.body.removeChild(a)
        URL.revokeObjectURL(url)
        
        alert('打包下载完成！')
      } catch (error) {
        console.error('打包下载失败:', error)
        alert('打包下载失败：' + (error.message || '未知错误'))
      } finally {
        this.downloading = false
      }
    },
    closeUploadsModal() {
      if (!this.loadingUploads && !this.downloading) {
        this.showUploadsModal = false
      }
    }
  },
  watch: {
    showUploadsModal(newVal) {
      if (newVal) {
        this.loadUploadedAudios()
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
.level-three { 
  display: flex; 
  flex-direction: column; 
  align-items: center; 
  gap: 12px; 
  position: relative;
}
.stage { position: relative; width: 100%; max-width: 900px; background: linear-gradient(180deg,#111 0%,#333 40%,#222 100%); color: #fff; border-radius: 16px; padding: 20px; box-shadow: 0 8px 24px rgba(0,0,0,0.35); }
.animal-picker { display: flex; gap: 8px; justify-content: center; flex-wrap: wrap; margin-bottom: 10px; }
.pick { width: 60px; height: 60px; border-radius: 10px; border: 1px solid #555; background: #222; color: #fff; cursor: pointer; font-size: 32px; transition: all 0.2s ease; display: flex; align-items: center; justify-content: center; }
.pick:hover { transform: scale(1.1); }
.pick.selected { outline: 3px solid #42b983; background: #333; }

/* 气泡框选项 */
.bubble-option .bubble-animation {
  animation: bubble-float 2s ease-in-out infinite;
}
@keyframes bubble-float {
  0%, 100% { 
    transform: translateY(0px);
  }
  50% { 
    transform: translateY(-5px);
  }
}

.center-view { 
  display: flex; 
  justify-content: center; 
  align-items: center; 
  height: 160px;
  margin: 15px 0;
}
.chosen { 
  font-size: 120px;
  transition: all 0.3s ease;
}

/* Emoji 变化动画 */
.emoji-change-enter-active,
.emoji-change-leave-active {
  transition: all 0.4s ease;
}
.emoji-change-enter-from {
  opacity: 0;
  transform: scale(0.5) rotate(180deg);
}
.emoji-change-leave-to {
  opacity: 0;
  transform: scale(0.5) rotate(-180deg);
}

/* 提示区域 - 输入框 */
.prompt-section {
  margin: 15px 0;
  width: 100%;
}

/* 确保所有输入框都没有默认边框，除了下划线 */
.prompt-section input {
  -webkit-appearance: none;
  -moz-appearance: none;
  appearance: none;
  border: none !important;
  border-top: none !important;
  border-left: none !important;
  border-right: none !important;
  border-bottom: none !important;
  box-shadow: none !important;
  outline: none !important;
  background-image: none !important;
}
.input-line {
  display: flex;
  align-items: center;
  justify-content: center;
  flex-wrap: wrap;
  gap: 5px;
  margin: 8px 0;
  font-size: 16px;
  color: #ffd;
}
.label {
  white-space: nowrap;
}
.animal-input {
  width: 120px;
  padding: 4px 8px;
  border: none !important;
  border-bottom: 2px solid #ffd !important;
  border-top: none !important;
  border-left: none !important;
  border-right: none !important;
  background: transparent;
  color: #ffd;
  font-size: 16px;
  text-align: center;
  outline: none !important;
  transition: all 0.3s ease;
  box-shadow: none !important;
  -webkit-appearance: none !important;
  -moz-appearance: none !important;
  appearance: none !important;
}
.animal-input:focus {
  border-bottom-color: #42b983;
  background: rgba(66, 185, 131, 0.1);
}
.animal-input::placeholder {
  color: rgba(255, 255, 221, 0.5);
}
.animal-input.error {
  border-bottom-color: #e74c3c;
  background: rgba(231, 76, 60, 0.2);
  animation: shake-input 0.5s ease;
}
@keyframes shake-input {
  0%, 100% { transform: translateX(0); }
  25% { transform: translateX(-5px); }
  75% { transform: translateX(5px); }
}

/* 空白输入框（可拖入） */
.blank-input {
  position: relative;
  min-width: 100px;
  padding: 4px 8px;
  border: none;
  border-bottom: none;
  background: transparent;
  display: inline-block;
  min-height: 26px;
  transition: all 0.3s ease;
}
.blank-input:hover {
  background: rgba(66, 185, 131, 0.1);
}
.blank-input:hover input {
  background: linear-gradient(to bottom, transparent 0%, transparent calc(100% - 2px), #42b983 calc(100% - 2px), #42b983 100%) !important;
}
.blank-input input {
  width: 100%;
  border: none !important;
  border-bottom: none !important;
  border-top: none !important;
  border-left: none !important;
  border-right: none !important;
  background: linear-gradient(to bottom, transparent 0%, transparent calc(100% - 2px), #ffd calc(100% - 2px), #ffd 100%) !important;
  color: #ffd;
  font-size: 16px;
  text-align: center;
  outline: none !important;
  padding: 0;
  box-shadow: none !important;
  -webkit-appearance: none !important;
  -moz-appearance: none !important;
  appearance: none !important;
  transition: background 0.3s ease;
}
.blank-input input::placeholder {
  color: rgba(255, 255, 221, 0.5);
}
.filled-word {
  color: #42b983;
  font-weight: bold;
  display: inline-block;
  padding: 2px 6px;
  background: rgba(66, 185, 131, 0.3);
  border-radius: 4px;
  cursor: pointer;
  transition: all 0.2s ease;
}
.filled-word:hover {
  background: rgba(66, 185, 131, 0.5);
  transform: scale(1.05);
}

/* 单词区域 */
.words-section {
  margin: 15px 0;
  width: 100%;
}
.word-bank {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
  justify-content: center;
  padding: 15px;
  background: rgba(0, 0, 0, 0.3);
  border-radius: 10px;
  border: 1px solid rgba(255, 255, 255, 0.2);
}
.word-chip {
  padding: 6px 12px;
  background: rgba(66, 185, 131, 0.8);
  color: white;
  border-radius: 16px;
  font-size: 14px;
  font-weight: 500;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
  cursor: grab;
  user-select: none;
  transition: all 0.2s ease;
}
.word-chip:hover {
  background: rgba(66, 185, 131, 1);
  transform: translateY(-2px);
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.3);
}
.word-chip:active {
  cursor: grabbing;
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

/* 查看上传记录按钮 */
.view-uploads-btn {
  position: absolute;
  top: 10px;
  right: 10px;
  padding: 8px 16px;
  border: none;
  background: #42b983;
  color: white;
  border-radius: 8px;
  cursor: pointer;
  font-size: 14px;
  transition: all 0.3s ease;
  z-index: 100;
}
.view-uploads-btn:hover {
  background: #38a372;
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(66, 185, 131, 0.3);
}

/* 上传按钮 */
.upload-btn {
  padding: 10px 16px;
  border: none;
  background: #17a2b8;
  color: #fff;
  border-radius: 10px;
  cursor: pointer;
  font-size: 14px;
  transition: all 0.3s ease;
  min-width: 120px;
}
.upload-btn:hover:not(:disabled) {
  background: #138496;
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(23, 162, 184, 0.3);
}
.upload-btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

/* 上传弹窗 */
.upload-modal {
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

.upload-content {
  background: white;
  border-radius: 16px;
  padding: 30px;
  max-width: 500px;
  width: 100%;
  box-shadow: 0 10px 40px rgba(0, 0, 0, 0.3);
}

.upload-content h3 {
  margin: 0 0 20px 0;
  color: #2c3e50;
  font-size: 24px;
  text-align: center;
}

.upload-form {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.form-group {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.form-group label {
  color: #2c3e50;
  font-weight: 500;
  font-size: 16px;
}

.name-input {
  padding: 12px;
  border: 2px solid #ddd;
  border-radius: 8px;
  font-size: 16px;
  outline: none;
  transition: all 0.3s ease;
}

.name-input:focus {
  border-color: #42b983;
  box-shadow: 0 0 0 3px rgba(66, 185, 131, 0.1);
}

.upload-info {
  padding: 12px;
  background: #f8f9fa;
  border-radius: 8px;
  font-size: 14px;
  color: #666;
}

.upload-info p {
  margin: 0;
}

.confirm-upload-btn,
.cancel-upload-btn {
  padding: 10px 24px;
  border: none;
  border-radius: 8px;
  font-size: 16px;
  cursor: pointer;
  transition: all 0.3s ease;
}

.confirm-upload-btn {
  background: #42b983;
  color: white;
}

.confirm-upload-btn:hover:not(:disabled) {
  background: #38a372;
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(66, 185, 131, 0.3);
}

.confirm-upload-btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.cancel-upload-btn {
  background: #e0e0e0;
  color: #666;
}

.cancel-upload-btn:hover:not(:disabled) {
  background: #d0d0d0;
  transform: translateY(-2px);
}

/* 查看上传记录弹窗 */
.uploads-modal {
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

.uploads-content {
  background: white;
  border-radius: 16px;
  padding: 30px;
  max-width: 800px;
  width: 100%;
  max-height: 80vh;
  display: flex;
  flex-direction: column;
  box-shadow: 0 10px 40px rgba(0, 0, 0, 0.3);
}

.uploads-content h3 {
  margin: 0 0 20px 0;
  color: #2c3e50;
  font-size: 24px;
  text-align: center;
}

.uploads-list-container {
  flex: 1;
  overflow-y: auto;
  margin-bottom: 20px;
  min-height: 200px;
  max-height: 400px;
}

.loading,
.empty-list {
  text-align: center;
  padding: 40px;
  color: #999;
  font-size: 16px;
}

.uploads-list {
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.upload-item {
  padding: 15px;
  background: #f8f9fa;
  border-radius: 10px;
  border: 1px solid #e0e0e0;
}

.upload-item-info {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 10px;
}

.upload-name {
  font-weight: bold;
  color: #2c3e50;
  font-size: 16px;
}

.upload-time {
  color: #999;
  font-size: 14px;
}

.upload-audio {
  width: 100%;
  margin-top: 10px;
}

.download-all-btn,
.refresh-btn {
  padding: 10px 24px;
  border: none;
  border-radius: 8px;
  font-size: 16px;
  cursor: pointer;
  transition: all 0.3s ease;
}

.download-all-btn {
  background: #ff9800;
  color: white;
}

.download-all-btn:hover:not(:disabled) {
  background: #f57c00;
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(255, 152, 0, 0.3);
}

.download-all-btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.refresh-btn {
  background: #2196f3;
  color: white;
}

.refresh-btn:hover:not(:disabled) {
  background: #1976d2;
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(33, 150, 243, 0.3);
}

.refresh-btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

/* 移动端适配 */
@media (max-width: 768px) {
  .stage {
    max-width: 100%;
    padding: 15px;
  }
  
  .pick {
    width: 50px;
    height: 50px;
    font-size: 28px;
  }
  
  .chosen {
    font-size: 100px;
  }
  
  .input-line {
    font-size: 14px;
  }
  
  .animal-input,
  .blank-input input {
    font-size: 14px;
  }
  
  .word-chip {
    font-size: 13px;
    padding: 5px 10px;
  }
  
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

@media (max-width: 480px) {
  .chosen {
    font-size: 80px;
  }
  
  .pick {
    width: 45px;
    height: 45px;
    font-size: 24px;
  }
}
</style>

