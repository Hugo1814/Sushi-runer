<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<title>Sushi Runner Sound</title>

<style>
body {
  margin: 0;
  background: #5a0000;
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  font-family: Arial, sans-serif;
}

#game {
  width: 360px;
  height: 560px;
  background: linear-gradient(#a80000, #ff3b3b);
  border-radius: 20px;
  position: relative;
  overflow: hidden;
  box-shadow: 0 0 25px black;
}

#score {
  position: absolute;
  top: 10px;
  left: 10px;
  color: #fff;
  font-weight: bold;
  z-index: 10;
}

#soundBtn {
  position: absolute;
  top: 10px;
  right: 10px;
  z-index: 10;
  border-radius: 10px;
  border: none;
  padding: 6px 10px;
}

/* PLAYER */
#player {
  width: 50px;
  height: 80px;
  position: absolute;
  bottom: 20px;
  left: 155px;
  transition: left 0.15s;
}

.head {
  width: 35px;
  height: 35px;
  background: #ffd1b3;
  border-radius: 50%;
  margin: 0 auto -10px auto;
}

.body {
  width: 100%;
  height: 55px;
  border-radius: 12px;
}

.female .body { background: #ff69b4; }
.male .body { background: #4169e1; }

/* SUSHIS */
.sushi {
  position: absolute;
  top: -60px;
  border-radius: 10px;
  background: white;
  border: 3px solid black;
}

.nigiri { width:45px;height:45px; }
.uramaki { width:38px;height:38px;background:#fff8dc; }
.temaki { width:55px;height:55px;border-radius:50%; }
.especial { width:50px;height:50px;background:gold; }

/* CONTROLES */
#controls {
  position: absolute;
  bottom: 10px;
  width: 100%;
  display: flex;
  justify-content: space-around;
  z-index: 10;
}

button {
  width: 130px;
  height: 42px;
  border: none;
  border-radius: 12px;
  font-size: 18px;
  font-weight: bold;
}
</style>
</head>

<body>

<div id="game">
  <div id="score">Pontuação: 0</div>
  <button id="soundBtn">🔊</button>

  <div id="player" class="female">
    <div class="head"></div>
    <div class="body"></div>
  </div>

  <div id="controls">
    <button onclick="moveLeft()">⬅</button>
    <button onclick="moveRight()">➡</button>
  </div>
</div>

<script>
/* ÁUDIOS (sintéticos via Web Audio API) */
const AudioCtx = new (window.AudioContext || window.webkitAudioContext)();
let soundOn = true;

// Música japonesa (loop simples)
let musicOsc, musicGain;

function startMusic() {
  if (musicOsc) return;
  musicOsc = AudioCtx.createOscillator();
  musicGain = AudioCtx.createGain();
  musicOsc.type = "sine";
  musicOsc.frequency.value = 220;
  musicGain.gain.value = 0.05;
  musicOsc.connect(musicGain).connect(AudioCtx.destination);
  musicOsc.start();
}

function stopMusic() {
  if (musicOsc) {
    musicOsc.stop();
    musicOsc = null;
  }
}

function beep(freq, time=0.1) {
  if (!soundOn) return;
  const o = AudioCtx.createOscillator();
  const g = AudioCtx.createGain();
  o.frequency.value = freq;
  g.gain.value = 0.1;
  o.connect(g).connect(AudioCtx.destination);
  o.start();
  o.stop(AudioCtx.currentTime + time);
}

// Botão som
document.getElementById("soundBtn").onclick = () => {
  soundOn = !soundOn;
  document.getElementById("soundBtn").textContent = soundOn ? "🔊" : "🔇";
  soundOn ? startMusic() : stopMusic();
};

// JOGO
const game = document.getElementById("game");
const player = document.getElementById("player");
const scoreDisplay = document.getElementById("score");

let x = 155, score = 0, gameOver = false, baseSpeed = 3;

// Inicia música ao interagir
document.body.onclick = () => {
  AudioCtx.resume();
  startMusic();
};

function moveLeft() {
  if (x > 10) {
    x -= 45;
    player.style.left = x + "px";
    beep(600);
  }
}

function moveRight() {
  if (x < 300) {
    x += 45;
    player.style.left = x + "px";
    beep(600);
  }
}

function createSushi() {
  if (gameOver) return;

  const sushi = document.createElement("div");
  sushi.className = "sushi";

  let speed = baseSpeed;
  let types = ["nigiri","uramaki","temaki","especial"];
  let type = types[Math.min(Math.floor(score/30),3)];
  sushi.classList.add(type);

  sushi.style.left = Math.random() * 300 + "px";
  game.appendChild(sushi);

  let y = -60;

  const fall = setInterval(() => {
    y += speed;
    sushi.style.top = y + "px";

    const s = sushi.getBoundingClientRect();
    const p = player.getBoundingClientRect();

    if (s.bottom > p.top && s.left < p.right && s.right > p.left) {
      gameOver = true;
      beep(150,0.5);
      stopMusic();
      alert("🍣 Game Over! Pontuação: " + score);
      location.reload();
    }

    if (y > 560) {
      clearInterval(fall);
      sushi.remove();
    }
  }, 20);
}

setInterval(() => {
  if (!gameOver) {
    score++;
    scoreDisplay.textContent = "Pontuação: " + score;
    if (score % 20 === 0) baseSpeed += 0.5;
  }
}, 1000);

setInterval(createSushi, 900);
</script>

</body>
</html># Sushi-runer
Jogue enquanto seu pedido chegar
