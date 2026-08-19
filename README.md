# Uma Drop

点击掉落 Uma 玩偶的交互小站：Canvas + Matter.js 物理堆叠，玩偶全部是 **GIF 动图**逐帧播放（每只相位错开），带稀有度抽卡、语音彩蛋和隐藏菜单。零构建、单文件页面，直接部署 GitHub Pages。

## 本地预览

```bash
node dev/server.js
# 浏览器打开 http://127.0.0.1:8787/（端口被占用时：PowerShell 里先 $env:PORT=8899）
```

> 注意：直接双击 index.html 打开会因为 file:// 协议下 fetch 被浏览器拦截而无法加载 GIF，务必用上面的本地服务器预览。

## 部署到 GitHub Pages

1. 新建仓库（如 `uma`），把本目录全部内容推上去；
2. 仓库 **Settings → Pages** → Source 选 `main` 分支根目录 → Save；
3. 稍等片刻，访问 `https://<用户名>.github.io/uma/`；
4. （可选）绑自定义域名：仓库根目录放一个 `CNAME` 文件（内容为域名一行，如 `uma.example.com`），再到域名 DNS 加一条 CNAME 记录指向 `<用户名>.github.io`。

## 素材与玩法配置

所有可调项集中在 `index.html` 顶部 `CONFIG` 块：

```js
const CONFIG = {
  normalGifs: [...],   // 普通款 GIF 列表（每只随机挑一张，等概率）
  specials: [          // 特殊款：gif + 音频 + 概率（1/N，数字越大越稀有）
    { gif: 'assets/special/special1.gif', audios: ['assets/audio/special1.mp3'], rarity: 250 },
    { gif: 'assets/special/special2.gif', audios: ['assets/audio/special2.mp3'], rarity: 100 },
    { gif: 'assets/special/special3.gif', audios: ['assets/audio/special1.mp3', 'assets/audio/special2.mp3'], rarity: 40 },
  ],
  weiAudio: 'assets/audio/wei.mp3', // 每次点击/连发都放一次 wei
  maxPlushies: 100,   // 默认上限
  holdDelay: 350,     // 按住多久开始连发 ms
  spawnCooldown: 70,  // 连发间隔 ms
  cacheMaxSide: 256,  // GIF 帧缓存最大边长（内存/显存上限，改大更清晰改小更省内存）
  plushSize: [110, 150],       // 玩偶直径范围 px（桌面基准）
  plushScaleDivisor: 800,      // 屏幕宽度 ÷ 此值 = 缩放系数（≤1）
  plushScaleMin: 0.45          // 缩放系数下限（手机上玩偶变小）
};
```

当前素材映射（按你的分类整理）：

| 目录 | 用途 |
|---|---|
| `assets/normal/normal1-4.gif` | 普通款（4 张透明底动图，随机） |
| `assets/special/special1.gif` | 特殊款 1/250，配 `special1.mp3` |
| `assets/special/special3.gif` | 特殊款 1/40，配 `special2.mp3` |
| `assets/audio/wei.mp3` | 每次生成播放 |

想换素材：把新文件放进对应目录，改 `CONFIG` 里的路径即可。特殊 3 想要独立音频：把它 `audios` 数组换成新文件路径。

## 隐藏彩蛋

右下角最角落有一个**隐约可见的“⋯”按钮**，悬停变实，点开后：

- **Unlimited Uma Works** —— 解除 100 只上限（单向开启），显示计数器，镜头随堆高自动缩小、物理墙壁同步外扩；
- **Mobile Gravity Mode** —— 手机摇一摇散射玩偶 + 倾斜手机改变重力方向（支持上下颠倒，以开启瞬间的姿态校准）；
- **Audio** —— 语音开关（默认开）。

## 技术要点

- 页面内置**自写 GIF 解码器**（LZW + disposal 帧合成，零外部依赖），加载时两遍解码：第一遍算全帧透明包围盒，第二遍按包围盒裁剪 + 缩放到 `cacheMaxSide` 缓存，玩偶贴图更紧凑；
- 每只玩偶独立相位（`spawnedAt + random phase` 取模动画时长），堆叠后动画不会同步闪；
- wei 音频复用单实例重触发（点一下就响一次、连发不叠加糊成一团）；特殊音频每次新实例（可重叠、完整播完）；
- 特殊款生成时带一圈星光粒子；
- 加载界面分两阶段进度：下载（字节数）40% + 解码（帧数）60%。

## 目录结构

```
├── index.html          ← 全部代码（HTML/CSS/JS 单文件）
├── favicon.png
├── assets/
│   ├── normal/         ← 普通款 GIF
│   ├── special/        ← 特殊款 GIF
│   └── audio/          ← wei + 特殊音频
└── dev/
    ├── server.js       ← 本地预览服务器
    └── *.js            ← 开发期验证脚本（解码器测试等，不影响线上）
```
