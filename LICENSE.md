<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
<title>Реальная жизнь</title>
<style>
  * { box-sizing: border-box; margin: 0; padding: 0; -webkit-tap-highlight-color: transparent; }
  body {
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
    background: linear-gradient(135deg, #1a1a2e 0%, #16213e 100%);
    color: #eee;
    min-height: 100vh;
    padding: 10px;
    display: flex;
    justify-content: center;
    align-items: flex-start;
  }
  .game {
    width: 100%;
    max-width: 480px;
    background: rgba(255,255,255,0.05);
    border-radius: 20px;
    padding: 16px;
    box-shadow: 0 8px 32px rgba(0,0,0,0.4);
  }
  h1 {
    text-align: center;
    font-size: 22px;
    margin-bottom: 12px;
    color: #4ade80;
  }
  .stats {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 8px;
    margin-bottom: 14px;
  }
  .stat {
    background: rgba(0,0,0,0.3);
    padding: 8px 10px;
    border-radius: 10px;
    font-size: 13px;
    display: flex;
    justify-content: space-between;
  }
  .stat span:last-child { font-weight: bold; color: #fbbf24; }
  .bar {
    height: 6px;
    background: rgba(255,255,255,0.1);
    border-radius: 3px;
    margin-top: 6px;
    overflow: hidden;
  }
  .bar-fill {
    height: 100%;
    transition: width 0.4s;
  }
  .log {
    background: rgba(0,0,0,0.4);
    border-radius: 10px;
    padding: 10px;
    height: 110px;
    overflow-y: auto;
    font-size: 12px;
    margin-bottom: 12px;
    line-height: 1.5;
  }
  .log p { margin-bottom: 4px; opacity: 0.9; }
  .log .good { color: #4ade80; }
  .log .bad { color: #f87171; }
  .log .neutral { color: #93c5fd; }
  .actions {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 8px;
  }
  button {
    background: linear-gradient(135deg, #4f46e5, #7c3aed);
    color: white;
    border: none;
    padding: 14px 8px;
    border-radius: 12px;
    font-size: 13px;
    font-weight: 600;
    cursor: pointer;
    transition: transform 0.1s, opacity 0.2s;
    font-family: inherit;
  }
  button:active { transform: scale(0.95); }
  button:disabled { opacity: 0.4; cursor: not-allowed; }
  button.work { background: linear-gradient(135deg, #059669, #10b981); }
  button.rest { background: linear-gradient(135deg, #0284c7, #0ea5e9); }
  button.food { background: linear-gradient(135deg, #d97706, #f59e0b); }
  button.social { background: linear-gradient(135deg, #db2777, #ec4899); }
  .overlay {
    position: fixed;
    inset: 0;
    background: rgba(0,0,0,0.85);
    display: none;
    justify-content: center;
    align-items: center;
    padding: 20px;
    z-index: 10;
  }
  .overlay.show { display: flex; }
  .modal {
    background: #1e293b;
    padding: 24px;
    border-radius: 20px;
    text-align: center;
    max-width: 360px;
    width: 100%;
  }
  .modal h2 { margin-bottom: 12px; color: #fbbf24; }
  .modal p { margin-bottom: 16px; line-height: 1.6; font-size: 14px; }
  .modal button { width: 100%; background: linear-gradient(135deg, #4ade80, #22c55e); }
</style>
</head>
<body>
<div class="game">
  <h1>🏙️ Реальная жизнь</h1>

  <div class="stats">
    <div class="stat"><span>💰 Деньги</span><span id="money">500</span></div>
    <div class="stat"><span>🔋 Энергия</span><span id="energy">100</span></div>
    <div class="stat"><span>😊 Настроение</span><span id="mood">100</span></div>
    <div class="stat"><span>📅 День</span><span id="day">1</span></div>
    <div style="grid-column: 1 / -1;">
      <div class="stat" style="flex-direction:column; align-items:stretch; gap:0;">
        <div style="display:flex; justify-content:space-between;">
          <span>⚡ Энергия</span><span id="energyBar">100%</span>
        </div>
        <div class="bar"><div class="bar-fill" id="energyFill" style="width:100%; background:linear-gradient(90deg,#22c55e,#4ade80);"></div></div>
      </div>
    </div>
    <div style="grid-column: 1 / -1;">
      <div class="stat" style="flex-direction:column; align-items:stretch; gap:0;">
        <div style="display:flex; justify-content:space-between;">
          <span>😊 Настроение</span><span id="moodBar">100%</span>
        </div>
        <div class="bar"><div class="bar-fill" id="moodFill" style="width:100%; background:linear-gradient(90deg,#ec4899,#f472b6);"></div></div>
      </div>
    </div>
  </div>

  <div class="log" id="log"></div>

  <div class="actions">
    <button class="work" onclick="work()">💼 Работать (+200💰, -25🔋)</button>
    <button class="food" onclick="eat()">🍔 Поесть (-50💰, +30🔋)</button>
    <button class="rest" onclick="rest()">😴 Отдохнуть (+40🔋, +5😊)</button>
    <button class="social" onclick="social()">🎉 Развлечься (-80💰, +30😊)</button>
  </div>
</div>

<div class="overlay" id="overlay">
  <div class="modal">
    <h2 id="modalTitle">Игра окончена</h2>
    <p id="modalText"></p>
    <button onclick="restart()">Начать заново</button>
  </div>
</div>

<script>
let state = {
  money: 500,
  energy: 100,
  mood: 100,
  day: 1,
  gameOver: false
};

const logEl = document.getElementById('log');

function log(msg, type = 'neutral') {
  const p = document.createElement('p');
  p.className = type;
  p.textContent = msg;
  logEl.appendChild(p);
  logEl.scrollTop = logEl.scrollHeight;
}

function update() {
  document.getElementById('money').textContent = state.money;
  document.getElementById('energy').textContent = state.energy;
  document.getElementById('mood').textContent = state.mood;
  document.getElementById('day').textContent = state.day;

  document.getElementById('energyBar').textContent = state.energy + '%';
  document.getElementById('energyFill').style.width = state.energy + '%';
  document.getElementById('moodBar').textContent = state.mood + '%';
  document.getElementById('moodFill').style.width = state.mood + '%';

  // Цвет энергии
  const ef = document.getElementById('energyFill');
  if (state.energy > 50) ef.style.background = 'linear-gradient(90deg,#22c55e,#4ade80)';
  else if (state.energy > 25) ef.style.background = 'linear-gradient(90deg,#f59e0b,#fbbf24)';
  else ef.style.background = 'linear-gradient(90deg,#ef4444,#f87171)';

  const mf = document.getElementById('moodFill');
  if (state.mood > 50) mf.style.background = 'linear-gradient(90deg,#ec4899,#f472b6)';
  else if (state.mood > 25) mf.style.background = 'linear-gradient(90deg,#f59e0b,#fbbf24)';
  else mf.style.background = 'linear-gradient(90deg,#ef4444,#f87171)';

  // Проверка конца игры
  if (state.money < 0) endGame('Вы утонули в долгах! 💸', `Вы прожили ${state.day} дней. Начните заново и следите за деньгами.`);
  else if (state.energy <= 0) endGame('Вы полностью истощены! 😵', `Вы прожили ${state.day} дней. Не забывайте отдыхать и есть!`);
  else if (state.mood <= 0) endGame('Депрессия одолела вас... 😔', `Вы прожили ${state.day} дней. Развлечения тоже важны!`);
}

function checkDay() {
  // Каждые 4 действия = 1 день
  if (state.day < 30) {
    state.day++;
    state.energy = Math.max(0, state.energy - 5);
    log(`📅 Наступил день ${state.day}. Энергия -5`, 'neutral');
  }
  update();
}

function work() {
  if (state.gameOver) return;
  if (state.energy < 25) { log('❌ Слишком устали для работы! Отдохните.', 'bad'); return; }
  const bonus = Math.random() < 0.15 ? 200 : 0;
  state.money += 200 + bonus;
  state.energy -= 25;
  state.mood -= 5;
  log(`💼 Вы поработали. +${200 + bonus}💰, -25🔋` + (bonus ? ' 🎁 Бонус!' : ''), 'good');
  checkDay();
  update();
}

function eat() {
  if (state.gameOver) return;
  if (state.money < 50) { log('❌ Не хватает денег на еду!', 'bad'); return; }
  state.money -= 50;
  state.energy = Math.min(100, state.energy + 30);
  state.mood += 5;
  log('🍔 Вы поели. -50💰, +30🔋, +5😊', 'good');
  checkDay();
  update();
}

function rest() {
  if (state.gameOver) return;
  state.energy = Math.min(100, state.energy + 40);
  state.mood += 5;
  log('😴 Вы отдохнули. +40🔋, +5😊', 'good');
  checkDay();
  update();
}

function social() {
  if (state.gameOver) return;
  if (state.money < 80) { log('❌ Не хватает денег на развлечения!', 'bad'); return; }
  state.money -= 80;
  state.mood = Math.min(100, state.mood + 30);
  state.energy -= 10;
  log('🎉 Вы хорошо провели время. -80💰, +30😊, -10🔋', 'good');
  checkDay();
  update();
}

function endGame(title, text) {
  state.gameOver = true;
  document.getElementById('modalTitle').textContent = title;
  document.getElementById('modalText').textContent = text;
  document.getElementById('overlay').classList.add('show');
}

function restart() {
  state = { money: 500, energy: 100, mood: 100, day: 1, gameOver: false };
  logEl.innerHTML = '';
  document.getElementById('overlay').classList.remove('show');
  log('🎮 Новая жизнь началась! У вас 500💰.', 'good');
  update();
}

// Старт
log('🎮 Добро пожаловать в «Реальную жизнь»!', 'good');
log('Зарабатывайте деньги, следите за энергией и настроением.', 'neutral');
update();
</script>
</body>
</html>
