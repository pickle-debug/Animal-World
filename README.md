# 🦁 Animal World - 动物世界互动游戏

一个有趣的英语学习互动游戏，帮助孩子们认识动物及其英文名称。

## ✨ 特性

- 🎮 **三个有趣的关卡**：匹配游戏、猜谜游戏和口语练习
- 📱 **完全支持移动端**：支持触摸拖放、响应式设计
- 🖥️ **桌面端友好**：原生拖放支持
- 🎨 **精美的动画效果**：流畅的交互体验
- 🔊 **音效反馈**：完成时的鼓励音效

## 🎯 游戏内容

### 第一关：Match & Meet!
- 将英文单词拖动到对应的动物上
- 支持鼠标拖放（桌面端）
- 支持触摸拖动（移动端）
- 提交后查看答案

### 第二关：Who Am I?
- 根据线索猜测动物
- 有时间限制的挑战
- 可以使用提示功能

### 第三关：Show & Tell!
- 选择你最喜欢的动物
- 录音练习口语表达
- 自动给予积极反馈

## 📱 移动端支持

项目完全支持移动设备触摸交互！查看 [MOBILE_TOUCH_GUIDE.md](./MOBILE_TOUCH_GUIDE.md) 了解详情。

## 🚀 快速开始

### 安装依赖
```bash
npm install
```

### 开发模式（支持热重载）
```bash
npm run serve
```

### 在手机上测试
1. 启动开发服务器：`npm run serve`
2. 查看终端中的本地 IP 地址（如 `http://192.168.x.x:8080`）
3. 在手机浏览器中访问这个地址

### 构建生产版本
```bash
npm run build
```

### 代码检查和修复
```bash
npm run lint
```

## 🛠️ 技术栈

- **Vue.js 3** - 渐进式 JavaScript 框架
- **HTML5 Drag & Drop API** - 桌面端拖放
- **Touch Events API** - 移动端触摸交互
- **CSS3 Animations** - 流畅的动画效果
- **MediaRecorder API** - 录音功能

## 📂 项目结构

```
Animal-World/
├── public/              # 静态资源
│   └── index.html       # HTML 模板（已优化移动端）
├── src/
│   ├── App.vue          # 主应用组件
│   ├── main.js          # 入口文件
│   └── components/
│       ├── LevelOne.vue    # 第一关：匹配游戏（支持触摸）
│       ├── LevelTwo.vue    # 第二关：猜谜游戏
│       └── LevelThree.vue  # 第三关：口语练习
└── README.md            # 项目说明
```

## 🎨 浏览器兼容性

- ✅ Chrome (Desktop & Mobile)
- ✅ Safari (Desktop & iOS)
- ✅ Firefox (Desktop & Mobile)
- ✅ Edge (Desktop)
- ✅ 所有现代移动浏览器

## 📝 更新日志

### v1.1.0 (最新)
- ✨ 添加完整的移动端触摸支持
- 📱 响应式设计优化
- 🎨 触摸拖动视觉反馈
- 🔧 防止移动端页面缩放

### v1.0.0
- 🎉 初始版本
- 🎮 三个互动关卡
- 🖥️ 桌面端拖放支持

## 🤝 贡献

欢迎提交 Issue 和 Pull Request！

## 📄 许可

MIT License

## 🎓 了解更多

- [Vue.js 文档](https://vuejs.org/)
- [Vue CLI 配置参考](https://cli.vuejs.org/config/)
- [移动端触摸指南](./MOBILE_TOUCH_GUIDE.md)

---

用 ❤️ 制作，让学习变得更有趣！

