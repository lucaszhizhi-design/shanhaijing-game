# 山海经守护神兽测试游戏 实现计划

> **面向 AI 代理的工作者：** 必需子技能：使用 superpowers:subagent-driven-development（推荐）或 superpowers:executing-plans 逐任务实现此计划。步骤使用复选框（`- [ ]`）语法来跟踪进度。

**目标：** 构建一个纯前端网页游戏，小学三年级课堂输入姓名→答5道趣味题→AI匹配山海经守护神兽，炫酷游戏风视觉，大屏投影优化。

**架构：** 单 index.html 文件内嵌 CSS+JS，5个游戏阶段通过页面切换实现，Canvas 粒子特效增强视觉，8只神兽素材（图片+视频）从 assets 目录加载，JS 内置题库数据随机抽取。

**技术栈：** HTML5 + CSS3 + vanilla JS + Canvas API + HTML5 Video

---

## 文件结构

| 文件 | 职责 |
|------|------|
| `index.html` | 游戏唯一入口，内嵌全部 CSS 和 JS，5个阶段的页面切换、动画控制、题库数据、匹配算法 |
| `assets/beasts/*.png` | 8只神兽静态高清图片，结果页和图鉴页展示 |
| `assets/beasts/*.mp4` | 8只神兽3-5秒亮相短视频，结果页播放 |

index.html 内部的逻辑分区（非物理拆分，保持单文件）：
- `<style>` — 全局样式 + 5个阶段的页面样式 + 动画定义
- `<script>` — 数据层（神兽定义、题库）→ 状态管理 → 匹配算法 → 页面渲染函数 → Canvas粒子引擎 → 事件绑定

---

### 任务 1：项目骨架 + 数据层

**文件：**
- 创建：`index.html`
- 创建：`assets/beasts/` 目录

- [ ] **步骤 1：创建 assets 目录结构**

```bash
mkdir -p assets/beasts
```

- [ ] **步骤 2：创建 index.html 骨架，内嵌数据层**

创建 `index.html`，包含完整的 HTML 结构、CSS 基础样式、以及 JS 数据层（神兽定义、题库、维度定义）。

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>测测你的山海经守护神兽</title>
<style>
/* 全局基础样式 */
* { margin: 0; padding: 0; box-sizing: border-box; }
body {
  font-family: 'Microsoft YaHei', sans-serif;
  background: #0a0a1a;
  color: #fff;
  overflow: hidden;
  width: 100vw;
  height: 100vh;
}
.page { display: none; width: 100%; height: 100%; position: absolute; top: 0; left: 0; }
.page.active { display: flex; flex-direction: column; align-items: center; justify-content: center; }
/* 后续任务逐步添加各页面样式 */
</style>
</head>
<body>
<canvas id="particleCanvas"></canvas>

<!-- 开场页 -->
<div id="startPage" class="page active">
  <!-- 任务2填充 -->
</div>

<!-- AI生成动画页 -->
<div id="aiPage" class="page">
  <!-- 任务3填充 -->
</div>

<!-- 答题页 -->
<div id="quizPage" class="page">
  <!-- 任务4填充 -->
</div>

<!-- 匹配动画页 -->
<div id="matchPage" class="page">
  <!-- 任务5填充 -->
</div>

<!-- 结果页 -->
<div id="resultPage" class="page">
  <!-- 任务6填充 -->
</div>

<!-- 图鉴页 -->
<div id="galleryPage" class="page">
  <!-- 任务7填充 -->
</div>

<script>
// ===== 数据层 =====
const DIMENSIONS = ['勇气', '智慧', '创意', '友善'];

const BEASTS = [
  { id: 'qilin', name: '麒麟', dimension: '勇气', index: 0,
    tags: ['王者风范', '勇猛无畏', '天生领袖'],
    message: '你有勇气和担当，遇困难从不退缩，麒麟会陪你勇往直前！',
    image: 'assets/beasts/qilin.png', video: 'assets/beasts/qilin.mp4' },
  { id: 'suanni', name: '狻猊', dimension: '勇气', index: 1,
    tags: ['坚毅守护', '忠诚勇敢', '默默坚守'],
    message: '你虽不张扬，但关键时刻总能挺身而出，狻猊会默默守护你！',
    image: 'assets/beasts/suanni.png', video: 'assets/beasts/suanni.mp4' },
  { id: 'baize', name: '白泽', dimension: '智慧', index: 0,
    tags: ['全知全解', '聪明睿智', '洞察真相'],
    message: '你好奇心强什么都想知道，白泽会帮你找到每一个答案！',
    image: 'assets/beasts/baize.png', video: 'assets/beasts/baize.mp4' },
  { id: 'zhulong', name: '烛龙', dimension: '智慧', index: 1,
    tags: ['照亮黑暗', '深邃思考', '目光如炬'],
    message: '你善于深入思考看透本质，烛龙会照亮你前行的方向！',
    image: 'assets/beasts/zhulong.png', video: 'assets/beasts/zhulong.mp4' },
  { id: 'qingniao', name: '青鸟', dimension: '创意', index: 0,
    tags: ['灵动飞翔', '想象无限', '梦想使者'],
    message: '你的想象力总能飞到意想不到的地方，青鸟会带你飞向梦想！',
    image: 'assets/beasts/qingniao.png', video: 'assets/beasts/qingniao.mp4' },
  { id: 'jiutailhu', name: '九尾狐', dimension: '创意', index: 1,
    tags: ['机敏灵动', '聪明百变', '魅力十足'],
    message: '你既聪明又有创意总能想出绝妙好点子，九尾狐会陪你灵光闪现！',
    image: 'assets/beasts/jiutailhu.png', video: 'assets/beasts/jiutailhu.mp4' },
  { id: 'dangkang', name: '当康', dimension: '友善', index: 0,
    tags: ['温和守护', '善良可靠', '平安使者'],
    message: '你让身边人感到安心温暖，当康会守护你和大家平安快乐！',
    image: 'assets/beasts/dangkang.png', video: 'assets/beasts/dangkang.mp4' },
  { id: 'lushu', name: '鹿蜀', dimension: '友善', index: 1,
    tags: ['温暖相伴', '善解人意', '快乐源泉'],
    message: '你总能给别人带来快乐像阳光一样温暖，鹿蜀会永远陪伴你！',
    image: 'assets/beasts/lushu.png', video: 'assets/beasts/lushu.mp4' }
];

// 题库：每组5题，前4题对应4维度，第5题综合细分
// 每题选项带维度加分
const QUESTION_BANK = [
  [
    {
      dimension: '勇气',
      text: '你在一片神秘森林里遇到了一只巨大的怪兽挡住了路，你会？',
      options: [
        { text: '拿起木棍，勇敢地冲上前！', scores: {勇气:3} },
        { text: '仔细观察怪兽，找到它的弱点', scores: {智慧:2, 勇气:1} },
        { text: '叫上好朋友一起想办法', scores: {友善:2, 创意:1} }
      ]
    },
    {
      dimension: '智慧',
      text: '你发现了一本古老的谜题书，上面写着一道谁也解不开的谜题，你会？',
      options: [
        { text: '认真思考，一步步推理找出答案', scores: {智慧:3} },
        { text: '大胆猜一个答案试试看', scores: {勇气:2, 创意:1} },
        { text: '去找老师或同学一起讨论', scores: {友善:2, 智慧:1} }
      ]
    },
    {
      dimension: '创意',
      text: '学校要举办才艺表演，你最想表演什么？',
      options: [
        { text: '自己编一个神奇的故事讲给大家听', scores: {创意:3} },
        { text: '表演一段勇敢的武术动作', scores: {勇气:2, 创意:1} },
        { text: '和好朋友一起合唱一首温暖的歌曲', scores: {友善:2, 智慧:1} }
      ]
    },
    {
      dimension: '友善',
      text: '你的好朋友今天心情不好，你会怎么做？',
      options: [
        { text: '陪在他身边，听他说心事', scores: {友善:3} },
        { text: '帮他分析问题，找到解决办法', scores: {智慧:2, 友善:1} },
        { text: '想一个好玩的游戏逗他开心', scores: {创意:2, 友善:1} }
      ]
    },
    {
      dimension: '综合细分',
      text: '如果你能拥有一种超能力，你最想要？',
      options: [] // 动态生成，根据主维度填入对应2只神兽的选项
    }
  ],
  [
    {
      dimension: '勇气',
      text: '你和同学们在探险时发现了一个黑漆漆的洞穴，你会？',
      options: [
        { text: '第一个走进洞穴探一探！', scores: {勇气:3} },
        { text: '先在洞口观察里面的声音和光亮', scores: {智慧:2, 勇气:1} },
        { text: '拉着好朋友的手一起进去', scores: {友善:2, 创意:1} }
      ]
    },
    {
      dimension: '智慧',
      text: '你在书里看到了一句很深的话理解不了，你会？',
      options: [
        { text: '反复琢磨直到想明白为止', scores: {智慧:3} },
        { text: '用画画的方式把意思表达出来', scores: {创意:2, 智慧:1} },
        { text: '去问懂的人帮我解释', scores: {友善:2, 勇气:1} }
      ]
    },
    {
      dimension: '创意',
      text: '下雨天不能出去玩，你在家里会做什么？',
      options: [
        { text: '用纸箱和彩笔搭一个奇幻城堡', scores: {创意:3} },
        { text: '读一本探险故事书', scores: {勇气:2, 创意:1} },
        { text: '给家人做一张温暖的贺卡', scores: {友善:2, 智慧:1} }
      ]
    },
    {
      dimension: '友善',
      text: '班上来了一个新同学看起来很孤单，你会？',
      options: [
        { text: '主动走过去跟他聊天做朋友', scores: {友善:3} },
        { text: '邀请他一起参加游戏', scores: {创意:2, 友善:1} },
        { text: '帮他熟悉学校的环境和规则', scores: {智慧:2, 友善:1} }
      ]
    },
    {
      dimension: '综合细分',
      text: '你最想成为什么样的人？',
      options: [] // 动态生成
    }
  ],
  [
    {
      dimension: '勇气',
      text: '运动会上你要参加一个很难的项目，你会怎么准备？',
      options: [
        { text: '拼命练习，一定要拿到好成绩！', scores: {勇气:3} },
        { text: '研究技巧找最聪明的训练方法', scores: {智慧:2, 勇气:1} },
        { text: '找小伙伴一起练习互相鼓励', scores: {友善:2, 创意:1} }
      ]
    },
    {
      dimension: '智慧',
      text: '你在博物馆看到了一件奇妙的展品，你会？',
      options: [
        { text: '仔细阅读旁边的介绍弄明白原理', scores: {智慧:3} },
        { text: '想象这件展品在古代是怎么用的', scores: {创意:2, 智慧:1} },
        { text: '叫同学们一起看大家一起讨论', scores: {友善:2, 勇气:1} }
      ]
    },
    {
      dimension: '创意',
      text: '如果让你设计一个全新的游戏，你会做成什么样子？',
      options: [
        { text: '一个充满魔法和奇幻世界的冒险游戏', scores: {创意:3} },
        { text: '一个需要勇敢闯关的挑战游戏', scores: {勇气:2, 创意:1} },
        { text: '一个可以和朋友一起玩的合作游戏', scores: {友善:2, 智慧:1} }
      ]
    },
    {
      dimension: '友善',
      text: '下课时有人不小心撞到你，你会？',
      options: [
        { text: '微笑说没关系，问他有没有受伤', scores: {友善:3} },
        { text: '冷静想想是不是自己站的位置不对', scores: {智慧:2, 友善:1} },
        { text: '开玩笑说没事，化解尴尬气氛', scores: {创意:2, 勇气:1} }
      ]
    },
    {
      dimension: '综合细分',
      text: '如果你能变成一只动物，你想变成？',
      options: [] // 动态生成
    }
  ]
];

// ===== 状态管理 =====
let gameState = {
  playerName: '',
  scores: {勇气:0, 智慧:0, 创意:0, 友善:0},
  currentQuestion: 0,
  questions: [],
  matchedBeast: null
};

// ===== 匹配算法 =====
function calculateMatch() {
  // 找最高分维度
  let maxScore = 0;
  let maxDimension = '';
  for (const dim of DIMENSIONS) {
    if (gameState.scores[dim] > maxScore) {
      maxScore = gameState.scores[dim];
      maxDimension = dim;
    }
  }
  // 第5题选择决定 index (0 或 1)
  const beastIndex = gameState.fifthChoiceIndex || 0;
  // 找对应神兽
  gameState.matchedBeast = BEASTS.find(
    b => b.dimension === maxDimension && b.index === beastIndex
  );
}

// 动态生成第5题选项
function generateFifthQuestion() {
  let maxDimension = '';
  let maxScore = 0;
  for (const dim of DIMENSIONS) {
    if (gameState.scores[dim] > maxScore) {
      maxScore = gameState.scores[dim];
      maxDimension = dim;
    }
  }
  const beastsInDim = BEASTS.filter(b => b.dimension === maxDimension);
  const fifthQ = gameState.questions[4];
  fifthQ.options = [
    { text: `成为${beastsInDim[0].tags[0]}的${beastsInDim[0].name}`, scores: {}, beastIndex: 0 },
    { text: `成为${beastsInDim[1].tags[0]}的${beastsInDim[1].name}`, scores: {}, beastIndex: 1 }
  ];
}

// ===== 页面切换 =====
function showPage(pageId) {
  document.querySelectorAll('.page').forEach(p => p.classList.remove('active'));
  document.getElementById(pageId).classList.add('active');
}

// ===== 游戏启动 =====
function startGame(name) {
  gameState = {
    playerName: name,
    scores: {勇气:0, 智慧:0, 创意:0, 友善:0},
    currentQuestion: 0,
    questions: [],
    matchedBeast: null,
    fifthChoiceIndex: 0
  };
  // 随机抽取一组题目
  const groupIndex = Math.floor(Math.random() * QUESTION_BANK.length);
  gameState.questions = QUESTION_BANK[groupIndex].map(q => ({...q, options: q.options.map(o => ({...o}))}));
  showPage('aiPage');
  // 3秒后切换到答题页
  setTimeout(() => {
    showPage('quizPage');
    renderQuestion(0);
  }, 3000);
}

// ===== 答题逻辑 =====
function selectOption(option) {
  // 累加维度得分
  if (option.scores) {
    for (const [dim, val] of Object.entries(option.scores)) {
      gameState.scores[dim] += val;
    }
  }
  // 第5题记录 beastIndex
  if (option.beastIndex !== undefined) {
    gameState.fifthChoiceIndex = option.beastIndex;
  }
  gameState.currentQuestion++;
  if (gameState.currentQuestion < 5) {
    if (gameState.currentQuestion === 4) {
      generateFifthQuestion();
    }
    renderQuestion(gameState.currentQuestion);
  } else {
    calculateMatch();
    showPage('matchPage');
    setTimeout(() => {
      showPage('resultPage');
      renderResult();
    }, 3000);
  }
}

function renderQuestion(index) { /* 任务4填充 */ }
function renderResult() { /* 任务6填充 */ }
function renderGallery() { /* 任务7填充 */ }

</script>
</body>
</html>
```

- [ ] **步骤 3：浏览器打开 index.html，确认页面骨架正确加载**

运行：浏览器直接打开 `index.html`
预期：黑底页面，能看到 JS 数据正确（console.log 验证 BEASTS 和 QUESTION_BANK）

- [ ] **步骤 4：Git commit**

```bash
git add index.html
git commit -m "feat: 游戏骨架和数据层 — 神兽定义、题库、匹配算法、页面切换"
```

---

### 任务 2：开场页

**文件：**
- 修改：`index.html`（startPage 区块 + CSS）

- [ ] **步骤 1：添加开场页 HTML 内容**

将 `startPage` div 填充为：

```html
<div id="startPage" class="page active">
  <h1 class="main-title">测测你的<br>山海经守护神兽</h1>
  <p class="start-subtitle">AI将为你生成专属题目，匹配属于你的神兽</p>
  <div class="name-input-area">
    <input type="text" id="nameInput" class="name-input" placeholder="请输入你的名字" maxlength="10">
    <button id="startBtn" class="start-btn" onclick="onStartClick()">开始测试</button>
  </div>
</div>
```

- [ ] **步骤 2：添加开场页 CSS**

在 `<style>` 中添加：

```css
.main-title {
  font-size: 4rem;
  text-align: center;
  color: #ffd700;
  text-shadow: 0 0 20px rgba(255,215,0,0.6), 0 0 40px rgba(255,215,0,0.3);
  margin-bottom: 20px;
  line-height: 1.3;
}
.start-subtitle {
  font-size: 1.4rem;
  color: #aaa;
  margin-bottom: 40px;
}
.name-input-area {
  display: flex;
  gap: 15px;
  align-items: center;
}
.name-input {
  font-size: 1.5rem;
  padding: 12px 24px;
  border: 2px solid #ffd700;
  border-radius: 8px;
  background: rgba(255,215,0,0.1);
  color: #fff;
  outline: none;
  width: 240px;
}
.name-input:focus {
  border-color: #ff6b6b;
  box-shadow: 0 0 15px rgba(255,107,107,0.4);
}
.start-btn {
  font-size: 1.5rem;
  padding: 12px 32px;
  border: none;
  border-radius: 8px;
  background: linear-gradient(135deg, #ffd700, #ff6b6b);
  color: #0a0a1a;
  cursor: pointer;
  font-weight: bold;
  transition: transform 0.2s, box-shadow 0.2s;
}
.start-btn:hover {
  transform: scale(1.05);
  box-shadow: 0 0 20px rgba(255,215,0,0.5);
}
```

- [ ] **步骤 3：添加 startBtn 事件处理**

在 `<script>` 中添加：

```javascript
function onStartClick() {
  const name = document.getElementById('nameInput').value.trim();
  if (!name) {
    document.getElementById('nameInput').style.borderColor = '#ff6b6b';
    return;
  }
  startGame(name);
}
```

- [ ] **步骤 4：浏览器测试开场页**

运行：浏览器打开 `index.html`
预期：金色大标题、输入框和按钮正常显示，输入名字点击按钮后切换到 AI 动画页（黑屏，因为 AI 页还没内容）

- [ ] **步骤 5：Git commit**

```bash
git add index.html
git commit -m "feat: 开场页 — 金色标题、姓名输入、开始按钮"
```

---

### 任务 3：AI生成动画页

**文件：**
- 修改：`index.html`（aiPage 区块 + CSS + JS 动画）

- [ ] **步骤 1：添加 AI 动画页 HTML**

将 `aiPage` div 填充为：

```html
<div id="aiPage" class="page">
  <div class="ai-animation">
    <div class="ai-icon">🤖</div>
    <div class="ai-text" id="aiText"></div>
    <div class="ai-progress-bar">
      <div class="ai-progress-fill" id="aiProgressFill"></div>
    </div>
  </div>
</div>
```

- [ ] **步骤 2：添加 AI 动画页 CSS**

```css
.ai-animation {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 20px;
}
.ai-icon {
  font-size: 4rem;
  animation: aiPulse 1s ease-in-out infinite;
}
@keyframes aiPulse {
  0%, 100% { transform: scale(1); opacity: 1; }
  50% { transform: scale(1.1); opacity: 0.8; }
}
.ai-text {
  font-size: 1.6rem;
  color: #4fc3f7;
  min-height: 2rem;
}
.ai-progress-bar {
  width: 300px;
  height: 6px;
  background: rgba(255,255,255,0.1);
  border-radius: 3px;
  overflow: hidden;
}
.ai-progress-fill {
  width: 0%;
  height: 100%;
  background: linear-gradient(90deg, #4fc3f7, #ffd700);
  border-radius: 3px;
  transition: width 0.3s;
}
```

- [ ] **步骤 3：添加 AI 打字机动画 JS**

在 `<script>` 中添加，并修改 `startGame` 函数以触发动画：

```javascript
function playAiAnimation() {
  const textEl = document.getElementById('aiText');
  const progressEl = document.getElementById('aiProgressFill');
  const fullText = `AI正在为${gameState.playerName}生成专属题目...`;
  let charIndex = 0;
  textEl.textContent = '';
  progressEl.style.width = '0%';

  const typeInterval = setInterval(() => {
    if (charIndex < fullText.length) {
      textEl.textContent += fullText[charIndex];
      charIndex++;
      progressEl.style.width = `${(charIndex / fullText.length) * 100}%`;
    } else {
      clearInterval(typeInterval);
    }
  }, 80);
}

// 修改 startGame 函数，在 showPage('aiPage') 后调用动画
// 已有 startGame 中 setTimeout 3秒后切换，只需加上动画调用
```

修改 `startGame` 函数中 `showPage('aiPage')` 后立即添加 `playAiAnimation();`

- [ ] **步骤 4：浏览器测试 AI 动画**

运行：浏览器打开 → 输入名字 → 点开始
预期：机器人图标脉动、打字机逐字显示"AI正在为XXX生成专属题目..."、进度条同步推进、3秒后切换到答题页

- [ ] **步骤 5：Git commit**

```bash
git add index.html
git commit -m "feat: AI生成动画页 — 打字机效果、进度条、机器人图标脉动"
```

---

### 任务 4：答题页

**文件：**
- 修改：`index.html`（quizPage 区块 + CSS + renderQuestion 函数）

- [ ] **步骤 1：添加答题页 HTML**

将 `quizPage` div 填充为：

```html
<div id="quizPage" class="page">
  <div class="quiz-header">
    <div class="quiz-progress" id="quizProgress"></div>
    <div class="quiz-counter" id="quizCounter">第1题 / 共5题</div>
  </div>
  <div class="quiz-question" id="quizQuestion"></div>
  <div class="quiz-options" id="quizOptions"></div>
</div>
```

- [ ] **步骤 2：添加答题页 CSS**

```css
.quiz-header {
  display: flex;
  align-items: center;
  gap: 20px;
  margin-bottom: 30px;
  width: 80%;
}
.quiz-progress {
  width: 100%;
  height: 8px;
  background: rgba(255,255,255,0.1);
  border-radius: 4px;
  overflow: hidden;
}
.quiz-progress-fill {
  height: 100%;
  background: linear-gradient(90deg, #ffd700, #ff6b6b);
  border-radius: 4px;
  transition: width 0.5s ease;
}
.quiz-counter {
  font-size: 1.2rem;
  color: #aaa;
  white-space: nowrap;
}
.quiz-question {
  font-size: 2rem;
  text-align: center;
  max-width: 80%;
  margin-bottom: 40px;
  line-height: 1.4;
  color: #fff;
}
.quiz-options {
  display: flex;
  flex-direction: column;
  gap: 18px;
  width: 70%;
}
.quiz-option {
  font-size: 1.4rem;
  padding: 16px 24px;
  border: 2px solid rgba(255,215,0,0.4);
  border-radius: 12px;
  background: rgba(255,215,0,0.05);
  color: #fff;
  cursor: pointer;
  transition: all 0.3s;
  text-align: center;
}
.quiz-option:hover {
  border-color: #ffd700;
  background: rgba(255,215,0,0.15);
  transform: scale(1.02);
}
.quiz-option.selected {
  border-color: #4fc3f7;
  background: rgba(79,195,247,0.2);
  transform: scale(1.05);
}
```

- [ ] **步骤 3：实现 renderQuestion 函数**

```javascript
function renderQuestion(index) {
  const q = gameState.questions[index];
  const questionEl = document.getElementById('quizQuestion');
  const optionsEl = document.getElementById('quizOptions');
  const counterEl = document.getElementById('quizCounter');

  counterEl.textContent = `第${index + 1}题 / 共5题`;
  questionEl.textContent = q.text;
  optionsEl.innerHTML = '';

  // 更新进度条
  const progressEl = document.getElementById('quizProgress');
  progressEl.innerHTML = `<div class="quiz-progress-fill" style="width:${(index / 5) * 100}%"></div>`;

  const labels = ['A', 'B', 'C'];
  q.options.forEach((opt, i) => {
    const btn = document.createElement('div');
    btn.className = 'quiz-option';
    btn.textContent = `${labels[i]}. ${opt.text}`;
    btn.onclick = () => {
      btn.classList.add('selected');
      // 禁用其他选项
      optionsEl.querySelectorAll('.quiz-option').forEach(o => o.onclick = null);
      // 延迟500ms让用户看到选中效果
      setTimeout(() => selectOption(opt), 500);
    };
    optionsEl.appendChild(btn);
  });
}
```

- [ ] **步骤 4：浏览器测试答题流程**

运行：浏览器打开 → 输入名字 → 完成全部5题
预期：题目逐题出现，进度条推进，选项可点击，选中后有视觉反馈，答完5题后切换到匹配动画页

- [ ] **步骤 5：Git commit**

```bash
git add index.html
git commit -m "feat: 答题页 — 题目渲染、进度条、选项交互、选中反馈"
```

---

### 任务 5：匹配动画页

**文件：**
- 修改：`index.html`（matchPage 区块 + CSS + JS 粒子汇聚效果）

- [ ] **步骤 1：添加匹配动画页 HTML**

将 `matchPage` div 填充为：

```html
<div id="matchPage" class="page">
  <div class="match-animation">
    <div class="match-text" id="matchText">AI正在匹配你的守护神兽...</div>
    <div class="match-orb" id="matchOrb"></div>
  </div>
</div>
```

- [ ] **步骤 2：添加匹配动画页 CSS**

```css
.match-animation {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 30px;
}
.match-text {
  font-size: 1.6rem;
  color: #4fc3f7;
}
.match-orb {
  width: 120px;
  height: 120px;
  border-radius: 50%;
  background: radial-gradient(circle, rgba(255,215,0,0.6), rgba(255,107,107,0.3), transparent);
  animation: orbGlow 1.5s ease-in-out infinite;
  position: relative;
}
@keyframes orbGlow {
  0%, 100% { transform: scale(1); box-shadow: 0 0 30px rgba(255,215,0,0.4); }
  50% { transform: scale(1.3); box-shadow: 0 0 60px rgba(255,215,0,0.8), 0 0 100px rgba(255,107,107,0.4); }
}
```

- [ ] **步骤 3：浏览器测试匹配动画**

运行：浏览器打开 → 完整答题流程 → 答完5题
预期：显示"AI正在匹配..."文字 + 光球脉动动画，3秒后切换到结果页

- [ ] **步骤 4：Git commit**

```bash
git add index.html
git commit -m "feat: 匹配动画页 — 光球脉动、匹配提示文字"
```

---

### 任务 6：结果页

**文件：**
- 修改：`index.html`（resultPage 区块 + CSS + renderResult 函数）

- [ ] **步骤 1：添加结果页 HTML**

将 `resultPage` div 填充为：

```html
<div id="resultPage" class="page">
  <div class="result-container" id="resultContainer"></div>
</div>
```

- [ ] **步骤 2：添加结果页 CSS**

```css
.result-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 20px;
  width: 90%;
  max-width: 700px;
}
.result-name-label {
  font-size: 1.4rem;
  color: #aaa;
}
.result-beast-name {
  font-size: 3.5rem;
  color: #ffd700;
  text-shadow: 0 0 20px rgba(255,215,0,0.6), 0 0 40px rgba(255,215,0,0.3);
  animation: beastNameGlow 2s ease-in-out infinite;
}
@keyframes beastNameGlow {
  0%, 100% { text-shadow: 0 0 20px rgba(255,215,0,0.6), 0 0 40px rgba(255,215,0,0.3); }
  50% { text-shadow: 0 0 30px rgba(255,215,0,0.8), 0 0 60px rgba(255,215,0,0.5), 0 0 80px rgba(255,107,107,0.3); }
}
.result-video-area {
  width: 400px;
  height: 400px;
  border-radius: 16px;
  overflow: hidden;
  border: 3px solid rgba(255,215,0,0.4);
  position: relative;
  background: #111;
}
.result-video-area video,
.result-video-area img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}
.result-tags {
  display: flex;
  gap: 12px;
}
.result-tag {
  font-size: 1.2rem;
  padding: 8px 16px;
  border-radius: 20px;
  background: rgba(255,215,0,0.15);
  border: 1px solid rgba(255,215,0,0.3);
  color: #ffd700;
}
.result-message {
  font-size: 1.4rem;
  text-align: center;
  line-height: 1.6;
  color: #ddd;
  max-width: 80%;
  padding: 20px;
  border-radius: 12px;
  background: rgba(255,255,255,0.05);
}
.result-buttons {
  display: flex;
  gap: 20px;
  margin-top: 20px;
}
.result-btn {
  font-size: 1.2rem;
  padding: 12px 28px;
  border-radius: 8px;
  cursor: pointer;
  font-weight: bold;
  transition: transform 0.2s, box-shadow 0.2s;
}
.result-btn:hover { transform: scale(1.05); }
.result-btn-primary {
  border: none;
  background: linear-gradient(135deg, #ffd700, #ff6b6b);
  color: #0a0a1a;
}
.result-btn-secondary {
  border: 2px solid rgba(255,215,0,0.4);
  background: transparent;
  color: #ffd700;
}
```

- [ ] **步骤 3：实现 renderResult 函数**

```javascript
function renderResult() {
  const beast = gameState.matchedBeast;
  const container = document.getElementById('resultContainer');
  container.innerHTML = `
    <div class="result-name-label">${gameState.playerName}，你的守护神兽是——</div>
    <div class="result-beast-name">${beast.name}</div>
    <div class="result-video-area">
      <video src="${beast.video}" autoplay muted onerror="this.style.display='none';this.nextElementSibling.style.display='block';">
      </video>
      <img src="${beast.image}" alt="${beast.name}" style="display:none;">
    </div>
    <div class="result-tags">
      ${beast.tags.map(t => `<div class="result-tag">${t}</div>`).join('')}
    </div>
    <div class="result-message">${beast.message}</div>
    <div class="result-buttons">
      <button class="result-btn result-btn-primary" onclick="restartGame()">再测一次</button>
      <button class="result-btn result-btn-secondary" onclick="showGallery()">看看其他神兽</button>
    </div>
  `;
  // 视频播放结束后显示图片
  const video = container.querySelector('video');
  if (video) {
    video.onended = () => { video.style.display = 'none'; video.nextElementSibling.style.display = 'block'; };
  }
}
```

- [ ] **步骤 4：添加 restartGame 和 showGallery 函数**

```javascript
function restartGame() {
  showPage('startPage');
  document.getElementById('nameInput').value = '';
}

function showGallery() {
  showPage('galleryPage');
  renderGallery();
}
```

- [ ] **步骤 5：浏览器测试结果页**

运行：浏览器打开 → 完整答题流程 → 到结果页
预期：显示神兽名称（光效）、视频/图片区域（当前无素材会显示兜底图片区）、性格标签、守护寄语、两个按钮

- [ ] **步骤 6：Git commit**

```bash
git add index.html
git commit -m "feat: 结果页 — 神兽名称光效、视频/图片展示、标签、寄语、操作按钮"
```

---

### 任务 7：图鉴页

**文件：**
- 修改：`index.html`（galleryPage 区块 + CSS + renderGallery 函数）

- [ ] **步骤 1：添加图鉴页 HTML**

将 `galleryPage` div 填充为：

```html
<div id="galleryPage" class="page">
  <h2 class="gallery-title">山海经神兽图鉴</h2>
  <div class="gallery-grid" id="galleryGrid"></div>
  <button class="result-btn result-btn-primary" onclick="restartGame()" style="margin-top:30px;">返回首页</button>
</div>
```

- [ ] **步骤 2：添加图鉴页 CSS**

```css
.gallery-title {
  font-size: 2.5rem;
  color: #ffd700;
  text-shadow: 0 0 15px rgba(255,215,0,0.4);
  margin-bottom: 30px;
}
.gallery-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 20px;
  width: 90%;
  max-width: 900px;
}
.gallery-card {
  background: rgba(255,255,255,0.05);
  border: 2px solid rgba(255,215,0,0.2);
  border-radius: 12px;
  padding: 15px;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 10px;
  transition: transform 0.3s, border-color 0.3s;
  cursor: pointer;
}
.gallery-card:hover {
  transform: scale(1.05);
  border-color: #ffd700;
}
.gallery-card.beast-matched {
  border-color: #ffd700;
  box-shadow: 0 0 20px rgba(255,215,0,0.3);
}
.gallery-card-img {
  width: 120px;
  height: 120px;
  border-radius: 8px;
  object-fit: cover;
  background: #222;
}
.gallery-card-name {
  font-size: 1.4rem;
  color: #ffd700;
}
.gallery-card-dim {
  font-size: 0.9rem;
  color: #aaa;
}
```

- [ ] **步骤 3：实现 renderGallery 函数**

```javascript
function renderGallery() {
  const grid = document.getElementById('galleryGrid');
  grid.innerHTML = '';
  BEASTS.forEach(beast => {
    const isMatched = gameState.matchedBeast && gameState.matchedBeast.id === beast.id;
    const card = document.createElement('div');
    card.className = `gallery-card ${isMatched ? 'beast-matched' : ''}`;
    card.innerHTML = `
      <img class="gallery-card-img" src="${beast.image}" alt="${beast.name}"
           onerror="this.src='';this.style.background='#222';this.alt='${beast.name}';">
      <div class="gallery-card-name">${beast.name}</div>
      <div class="gallery-card-dim">${beast.dimension}</div>
    `;
    grid.appendChild(card);
  });
}
```

- [ ] **步骤 4：浏览器测试图鉴页**

运行：完整答题流程 → 结果页 → 点"看看其他神兽"
预期：4×2网格展示8只神兽卡片，匹配的神兽卡片有金色边框高亮

- [ ] **步骤 5：Git commit**

```bash
git add index.html
git commit -m "feat: 图鉴页 — 8只神兽4×2网格、匹配神兽高亮"
```

---

### 任务 8：Canvas 粒子背景引擎

**文件：**
- 修改：`index.html`（Canvas 相关 CSS + JS 粒子系统）

- [ ] **步骤 1：添加 Canvas CSS（全屏覆盖）**

```css
#particleCanvas {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: -1;
  pointer-events: none;
}
```

- [ ] **步骤 2：实现粒子引擎 JS**

在 `<script>` 中添加完整的粒子系统：

```javascript
// ===== Canvas 粒子引擎 =====
const canvas = document.getElementById('particleCanvas');
const ctx = canvas.getContext('2d');
let particles = [];
let particleMode = 'starfield'; // starfield | datalflow | converge

function resizeCanvas() {
  canvas.width = window.innerWidth;
  canvas.height = window.innerHeight;
}
window.addEventListener('resize', resizeCanvas);
resizeCanvas();

class Particle {
  constructor() {
    this.reset();
  }
  reset() {
    this.x = Math.random() * canvas.width;
    this.y = Math.random() * canvas.height;
    this.vx = (Math.random() - 0.5) * 0.5;
    this.vy = (Math.random() - 0.5) * 0.5;
    this.size = Math.random() * 2 + 1;
    this.alpha = Math.random() * 0.5 + 0.3;
    this.color = this.randomColor();
    this.life = 1;
  }
  randomColor() {
    const colors = ['#ffd700', '#ff6b6b', '#4fc3f7', '#ab47bc', '#fff'];
    return colors[Math.floor(Math.random() * colors.length)];
  }
  update() {
    if (particleMode === 'converge') {
      // 向中心汇聚
      const cx = canvas.width / 2;
      const cy = canvas.height / 2;
      this.vx += (cx - this.x) * 0.005;
      this.vy += (cy - this.y) * 0.005;
      this.life -= 0.003;
    } else {
      this.x += this.vx;
      this.y += this.vy;
      // 边界反弹
      if (this.x < 0 || this.x > canvas.width) this.vx *= -1;
      if (this.y < 0 || this.y > canvas.height) this.vy *= -1;
    }
  }
  draw() {
    ctx.beginPath();
    ctx.arc(this.x, this.y, this.size, 0, Math.PI * 2);
    ctx.fillStyle = this.color;
    ctx.globalAlpha = this.alpha * this.life;
    ctx.fill();
    // 光晕效果
    ctx.beginPath();
    ctx.arc(this.x, this.y, this.size * 3, 0, Math.PI * 2);
    ctx.fillStyle = this.color;
    ctx.globalAlpha = this.alpha * this.life * 0.1;
    ctx.fill();
  }
}

// 初始化粒子
function initParticles(count) {
  particles = [];
  for (let i = 0; i < count; i++) {
    particles.push(new Particle());
  }
}
initParticles(80);

// 动画循环
function animateParticles() {
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  particles.forEach(p => {
    p.update();
    p.draw();
  });
  // 汇聚模式下，移除生命结束的粒子
  if (particleMode === 'converge') {
    particles = particles.filter(p => p.life > 0);
  }
  requestAnimationFrame(animateParticles);
}
animateParticles();

// 切换粒子模式
function setParticleMode(mode) {
  particleMode = mode;
  if (mode === 'converge') {
    // 重置粒子位置为散布状态
    initParticles(150);
    particles.forEach(p => {
      p.x = Math.random() * canvas.width;
      p.y = Math.random() * canvas.height;
      p.vx = (Math.random() - 0.5) * 2;
      p.vy = (Math.random() - 0.5) * 2;
      p.life = 1;
    });
  }
}
```

- [ ] **步骤 3：在页面切换时联动粒子模式**

修改 `showPage` 函数：

```javascript
function showPage(pageId) {
  document.querySelectorAll('.page').forEach(p => p.classList.remove('active'));
  document.getElementById(pageId).classList.add('active');
  // 联动粒子模式
  if (pageId === 'startPage') setParticleMode('starfield');
  else if (pageId === 'aiPage') setParticleMode('dataflow');
  else if (pageId === 'quizPage') setParticleMode('starfield');
  else if (pageId === 'matchPage') setParticleMode('converge');
  else if (pageId === 'resultPage') setParticleMode('starfield');
  else if (pageId === 'galleryPage') setParticleMode('starfield');
}
```

- [ ] **步骤 4：浏览器测试粒子效果**

运行：浏览器打开 → 逐步走完流程
预期：开场页星场粒子飘动 → AI页粒子流动 → 答题页星场 → 匹配页粒子向中心汇聚 → 结果页恢复星场

- [ ] **步骤 5：Git commit**

```bash
git add index.html
git commit -m "feat: Canvas粒子引擎 — 星场/数据流/汇聚模式、页面联动切换"
```

---

### 任务 9：占位素材 + 完整流程验证

**文件：**
- 创建：`assets/beasts/` 中8只神兽的占位图片

- [ ] **步骤 1：生成占位图片（用 Canvas 绘制简单图标）**

创建一个 `tools/generate-placeholders.html` 辅助工具，用于生成8只神兽的占位图片：

```html
<!DOCTYPE html>
<html>
<head><meta charset="UTF-8"><title>生成占位素材</title></head>
<body>
<h2>点击下方按钮生成占位神兽图片</h2>
<div id="outputs"></div>
<script>
const beasts = ['qilin','suanni','baize','zhulong','qingniao','jiutailhu','dangkang','lushu'];
const colors = ['#ffd700','#ff6b6b','#4fc3f7','#ab47bc','#66bb6a','#e91e63','#8d6e63','#81d4fa'];
const outputs = document.getElementById('outputs');

beasts.forEach((id, i) => {
  const c = document.createElement('canvas');
  c.width = 400; c.height = 400;
  const ctx = c.getContext('2d');
  // 渐变背景
  const grad = ctx.createRadialGradient(200,200,50,200,200,200);
  grad.addColorStop(0, colors[i]);
  grad.addColorStop(1, '#0a0a1a');
  ctx.fillStyle = grad;
  ctx.fillRect(0,0,400,400);
  // 神兽名文字
  const names = ['麒麟','狻猊','白泽','烛龙','青鸟','九尾狐','当康','鹿蜀'];
  ctx.fillStyle = '#fff';
  ctx.font = 'bold 48px Microsoft YaHei';
  ctx.textAlign = 'center';
  ctx.fillText(names[i], 200, 210);
  // 光圈
  ctx.beginPath();
  ctx.arc(200,200,120,0,Math.PI*2);
  ctx.strokeStyle = colors[i];
  ctx.lineWidth = 3;
  ctx.stroke();

  const div = document.createElement('div');
  div.style.margin = '10px';
  const img = document.createElement('img');
  img.src = c.toDataURL('image/png');
  img.style.width = '200px';
  div.appendChild(document.createTextNode(names[i] + ': '));
  div.appendChild(img);
  outputs.appendChild(div);

  // 提供下载（手动保存到 assets/beasts/ 目录）
  const link = document.createElement('a');
  link.download = `${id}.png`;
  link.href = c.toDataURL('image/png');
  link.textContent = '下载 ' + names[i];
  div.appendChild(link);
});
</script>
</body>
</html>
```

- [ ] **步骤 2：浏览器打开占位生成工具，下载8张图片到 assets/beasts/**

运行：浏览器打开 `tools/generate-placeholders.html`，逐个点击下载链接，保存到 `assets/beasts/` 目录

- [ ] **步骤 3：完整流程测试 — 验证8只神兽都能被匹配**

测试矩阵（选择各维度主导答案）：

| 测试 | 第1题 | 第2题 | 第3题 | 第4题 | 第5题 | 预期神兽 |
|------|-------|-------|-------|-------|-------|---------|
| 1 | A(勇气3) | A(智慧3) | A(创意3) | A(友善3) | 需看最高维度 | 视最高分 |
| 2 | A(勇气3) | B(智慧2) | B(勇气2) | B(友善2) | A(麒麟) | 麒麟 |
| 3 | A(勇气3) | B(智慧2) | B(勇气2) | B(友善2) | B(狻猊) | 狻猊 |
| 4 | C(友善2) | A(智慧3) | C(友善2) | A(友善3) | A(白泽) | 白泽 |

逐一测试，确认8只神兽都能通过特定答题组合被匹配到。

- [ ] **步骤 4：验证"再测一次"和"看看其他神兽"功能**

- 点"再测一次" → 返回开场页，输入框清空，可以重新玩
- 点"看看其他神兽" → 图鉴页展示8只，匹配的神兽高亮
- 图鉴页点"返回首页" → 回到开场页

- [ ] **步骤 5：Git commit**

```bash
git add index.html tools/ assets/
git commit -m "feat: 占位素材生成工具 + 完整流程验证"
```

---

### 任务 10：视觉打磨 + 投影仪适配

**文件：**
- 修改：`index.html`（全局样式调优）

- [ ] **步骤 1：优化投影仪适配样式**

在全局 CSS 中添加投影仪优化：

```css
/* 投影仪优化：大字体、高对比度、避免纯黑背景 */
body {
  background: #0d0d2b; /* 微亮深蓝而非纯黑，投影仪更可见 */
}
.main-title { font-size: 4.5rem; } /* 投影仪上更大 */
.quiz-question { font-size: 2.2rem; }
.quiz-option { font-size: 1.5rem; }
.result-beast-name { font-size: 4rem; }
.result-message { font-size: 1.5rem; }
```

- [ ] **步骤 2：添加页面切换过渡动画**

```css
.page {
  opacity: 0;
  transition: opacity 0.5s ease;
}
.page.active {
  opacity: 1;
}
```

- [ ] **步骤 3：添加答题选项选中后的动画效果**

```css
.quiz-option.selected {
  border-color: #4fc3f7;
  background: rgba(79,195,247,0.2);
  transform: scale(1.05);
  animation: optionSelected 0.5s ease;
}
@keyframes optionSelected {
  0% { transform: scale(1); }
  50% { transform: scale(1.08); }
  100% { transform: scale(1.05); }
}
```

- [ ] **步骤 4：在投影仪或大屏上实际测试**

在全屏模式（F11）下测试，确认：
- 字体在投影仪上清晰可读
- 颜色对比度足够
- 粒子效果可见但不干扰内容
- 流程完整无卡顿

- [ ] **步骤 5：Git commit**

```bash
git add index.html
git commit -m "feat: 投影仪适配优化 + 过渡动画 + 选中效果打磨"
```

---

## 规格覆盖度自检

| 规格章节 | 覆盖任务 |
|----------|---------|
| 游戏流程5阶段 | 任务2-6，每个阶段独立任务 |
| 性格维度×神兽体系 | 任务1数据层 |
| 题目设计框架 | 任务1题库 + 任务4答题页 |
| 匹配算法 | 任务1匹配函数 + 任务4第5题动态生成 |
| 结果页内容 | 任务6结果页 |
| 项目结构 | 任务1骨架 + 任务9素材 |
| 视觉风格 | 任务8粒子引擎 + 任务10打磨 |
| 技术架构 | 全部任务均为纯前端单文件实现 |
| 验证方式 | 任务9完整验证 |

无遗漏。无占位符。类型一致（BEASTS.id 在数据层和渲染层一致使用）。