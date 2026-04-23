
<html lang="ja">
<head>
<meta charset="UTF-8">
<title>筋トレタイマー</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<style>
svg {
  margin-bottom: 20px;
}

body {
  margin: 0;
  background: #111;
  color: white;
  font-family: Arial;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 100vh;
}

/* 編集ボタン（右上） */
#editBtn {
  position: absolute;
  top: 10px;
  right: 10px;
  font-size: 12px;
  padding: 6px 10px;
  background: #10b981;
}

/* 時間表示 */



#time {
  font-size: 80px;
  margin-bottom: 10px;
}

#status {
  font-size: 20px;
  margin-bottom: 20px;
  color: #38bdf8;
}

/* ボタン共通 */
button {
  font-size: 18px;
  padding: 12px 20px;
  margin: 5px;
  border: none;
  border-radius: 10px;
}

button {
  transition: all 0.1s ease;
  box-shadow: 0 4px 0 rgba(0,0,0,0.4);
}

button:active {
  transform: translateY(4px);
  box-shadow: 0 1px 0 rgba(0,0,0,0.4);
}

/* 色 */
.start { background: #2563eb; color: white; }
.stop { background: #f59e0b; }
.reset { background: #dc2626; }

/* レイアウト */
#subBtns {
  display: flex;
  gap: 10px;
  margin-top: 20px;
}
</style>
</head>

<body>

<!-- 編集 -->
<button id="editBtn" onclick="clickSound();editSettings()">編集</button>

<svg width="150" height="150">
  <circle cx="75" cy="75" r="65"
    stroke="#333" stroke-width="10" fill="none"/>
    
  <circle id="progress" cx="75" cy="75" r="65"
    stroke="#38bdf8" stroke-width="10" fill="none"
    stroke-dasharray="565"
    stroke-dashoffset="0"
    stroke-linecap="round"
    transform="rotate(-90 75 75)"
</svg>

<div id="time">5</div>
<div id="status">準備中</div>

<!-- スタート -->
<div id="mainBtn">
  <button class="start" onclick="clickSound(); startTimer()">スタート</button>
</div>

<!-- スタート後 -->
<div id="subBtns" style="display:none;">
  <button class="stop" onclick="togglePause()" id="pauseBtn">停止</button>
  <button class="reset" onclick="clickSound();resetTimer()">終了</button>
</div>

<script>
const circle = document.getElementById("progress");

const circumference = 2 * Math.PI * 65;

circle.style.strokeDasharray = circumference;

function updateCircle() {
  let total;

  if (mode === "ready") total = settings.readyTime;
  if (mode === "training") total = settings.trainingTime;
  if (mode === "rest") total = settings.restTime;

  const progress = time / total;
  const offset = circumference * (1 - progress);

  circle.style.strokeDashoffset = offset;
}

let settings = {
  readyTime: 5,
  trainingTime: 20,
  restTime: 10,
  rounds: 8
};

let mode = "ready";
let time = settings.readyTime;
let currentRound = 1;
let timer = null;

let isPaused = false;

let audioCtx = null;

function initAudio() {
  if (!audioCtx) {
    audioCtx = new (window.AudioContext || window.webkitAudioContext)();
  }
}

function playBeep(freq = 800, duration = 0.1) {
  if (!audioCtx) return;
  const osc = audioCtx.createOscillator();
  const gain = audioCtx.createGain();
  osc.frequency.value = freq;
  gain.gain.value = 0.2;
  osc.connect(gain);
  gain.connect(audioCtx.destination);
  osc.start();
  osc.stop(audioCtx.currentTime + duration);
}

function updateDisplay() {
  document.getElementById("time").innerText = time;

  let text = "";
  if (mode === "ready") text = "準備中";
  if (mode === "training") text = "トレーニング";
  if (mode === "rest") text = "休憩";

  document.getElementById("status").innerText =
    text + "（" + currentRound + "/" + settings.rounds + "）";
 updateCircle(); 
}

function startTimer() {
  if (timer) return;

  initAudio();


  // ボタン切り替え
  document.getElementById("mainBtn").style.display = "none";
  document.getElementById("subBtns").style.display = "flex";

  mode = "ready";
  time = settings.readyTime;
  currentRound = 1;
isPaused=false;

 document.getElementById("pauseBtn").innerText = "停止";

  updateDisplay();

startInterval();
}
function startInterval() {
  timer = setInterval(() => {
    time--;
    updateDisplay();

    playBeep(500, 0.05);

    if (mode === "ready" && time <= 3 && time > 0) {
      playBeep(1200, 0.1);
    }

    if (time <= 0) nextPhase();
  }, 1000);
}
 
function nextPhase() {
  playBeep(1500, 0.2);

  if (mode === "ready") {
    mode = "training";
    time = settings.trainingTime;
  } 
  else if (mode === "training") {
    mode = "rest";
    time = settings.restTime;
  } 
  else if (mode === "rest") {
    currentRound++;
    if (currentRound > settings.rounds) {
      clearInterval(timer);
      timer = null;
      alert("終了！");
      return;
    }
    mode = "training";
    time = settings.trainingTime;
  }
}

function stopTimer() {
  clearInterval(timer);
  timer = null;
}

function resetTimer() {
  clearInterval(timer);
  timer = null;

  // ボタン戻す
  document.getElementById("mainBtn").style.display = "block";
  document.getElementById("subBtns").style.display = "none";

  mode = "ready";
  time = settings.readyTime;
  currentRound = 1;
  updateDisplay();
}

function editSettings() {
  settings.readyTime = Number(prompt("準備時間", settings.readyTime));
  settings.trainingTime = Number(prompt("トレ時間", settings.trainingTime));
  settings.restTime = Number(prompt("休憩時間", settings.restTime));
  settings.rounds = Number(prompt("回数", settings.rounds));

  resetTimer();
}

updateDisplay();

function clickSound() {
  const ctx = new (window.AudioContext || window.webkitAudioContext)();
  const osc = ctx.createOscillator();
  const gain = ctx.createGain();

  osc.frequency.value = 1000; // 高めの音
  gain.gain.value = 0.2;

  osc.connect(gain);
  gain.connect(ctx.destination);

  osc.start();
  osc.stop(ctx.currentTime + 0.05);
}


function togglePause() {
  if (!timer && isPaused) {
    // ▶ 再開
    startInterval();
    isPaused = false;

document.body.style.opacity = 1;

    document.getElementById("pauseBtn").innerText = "停止";
    return;
  }

  if (timer) {
    // ⏸ 停止
    clearInterval(timer);
    timer = null;
    isPaused = true;

document.body.style.opacity = 0.82;
    document.getElementById("pauseBtn").innerText = "再開";
  }
}

window.onload = () => {
  const circle = document.getElementById("progress");
};
</script>

</body>
</html>
