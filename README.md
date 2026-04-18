<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0"/>
<title>Quiz Timer — Firecracker Edition</title>
<style>
  * { box-sizing: border-box; }
  html, body {
    margin: 0; padding: 0; height: 100%;
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
    color: #f5f5f5;
    overflow: hidden;
    background: radial-gradient(ellipse at top, #1b2a55 0%, #0a0f2c 55%, #03040f 100%);
  }
  .stars {
    position: fixed; inset: 0; z-index: 0; pointer-events: none;
    background-image:
      radial-gradient(1px 1px at 20% 30%, #fff, transparent),
      radial-gradient(1px 1px at 70% 20%, #fff, transparent),
      radial-gradient(1.5px 1.5px at 40% 60%, #fff, transparent),
      radial-gradient(1px 1px at 85% 70%, #fff, transparent),
      radial-gradient(1px 1px at 15% 80%, #fff, transparent),
      radial-gradient(1.5px 1.5px at 55% 85%, #fff, transparent),
      radial-gradient(1px 1px at 90% 40%, #fff, transparent),
      radial-gradient(1px 1px at 30% 15%, #fff, transparent),
      radial-gradient(1px 1px at 60% 45%, #fff, transparent),
      radial-gradient(1.5px 1.5px at 75% 90%, #fff, transparent);
    opacity: 0.75;
    animation: twinkle 4s ease-in-out infinite alternate;
  }
  @keyframes twinkle { from {opacity: 0.4;} to {opacity: 0.9;} }

  .app {
    position: relative; z-index: 2;
    height: 100%;
    display: flex; flex-direction: column;
    align-items: center; justify-content: center;
    gap: 24px;
    padding: 24px;
  }

  h1 {
    margin: 0;
    font-size: clamp(20px, 2.4vw, 28px);
    letter-spacing: 2px;
    text-transform: uppercase;
    color: #ffd76a;
    text-shadow: 0 0 12px rgba(255, 200, 60, 0.35);
  }

  .stage {
    position: relative;
    width: min(720px, 92vw);
    height: clamp(260px, 40vh, 360px);
    display: flex;
    align-items: center;
    justify-content: center;
  }

  /* Firecracker */
  .cracker {
    position: relative;
    width: 120px; height: 220px;
    display: flex; flex-direction: column; align-items: center;
    transition: transform 0.3s ease;
  }
  .cracker.shaking { animation: shake 0.08s linear infinite; }
  @keyframes shake {
    0% { transform: translate(0,0) rotate(0); }
    25% { transform: translate(-2px,1px) rotate(-1deg); }
    50% { transform: translate(2px,-1px) rotate(1deg); }
    75% { transform: translate(-1px,2px) rotate(-1deg); }
    100% { transform: translate(1px,-2px) rotate(1deg); }
  }

  .fuse-wrap {
    position: relative;
    height: 90px;
    display: flex; align-items: flex-end; justify-content: center;
  }
  .fuse {
    width: 4px; height: 100%;
    background: repeating-linear-gradient(
      to bottom,
      #6e4a1c 0 4px,
      #3b2510 4px 8px
    );
    border-radius: 2px;
    position: relative;
    transform-origin: bottom center;
  }
  .spark {
    position: absolute;
    width: 28px; height: 28px;
    top: -6px; left: 50%;
    transform: translate(-50%, -50%);
    pointer-events: none;
    filter: drop-shadow(0 0 10px #ffb347);
  }
  .spark.hidden { display: none; }
  .spark svg { width: 100%; height: 100%; animation: sparkPulse 0.25s ease-in-out infinite alternate; }
  @keyframes sparkPulse {
    from { transform: scale(0.85) rotate(0deg); }
    to   { transform: scale(1.15) rotate(25deg); }
  }

  .body {
    width: 110px; height: 140px;
    background: linear-gradient(180deg, #e53935 0%, #b71c1c 60%, #7a0f0f 100%);
    border-radius: 14px;
    position: relative;
    box-shadow:
      inset -8px 0 14px rgba(0,0,0,0.35),
      inset 8px 0 14px rgba(255,255,255,0.08),
      0 8px 20px rgba(0,0,0,0.4);
  }
  .body::before {
    content: "";
    position: absolute; left: 50%; top: -6px;
    transform: translateX(-50%);
    width: 80px; height: 10px;
    background: #f5c518;
    border-radius: 4px;
    box-shadow: 0 2px 0 #a58606;
  }
  .body::after {
    content: "";
    position: absolute; left: 50%; bottom: -6px;
    transform: translateX(-50%);
    width: 80px; height: 10px;
    background: #f5c518;
    border-radius: 4px;
    box-shadow: 0 -2px 0 #a58606;
  }
  .label {
    position: absolute; left: 0; right: 0; top: 50%;
    transform: translateY(-50%);
    text-align: center;
    color: #fff; font-weight: 700;
    letter-spacing: 2px;
    font-size: 14px;
    text-shadow: 1px 1px 0 rgba(0,0,0,0.5);
  }

  /* Timer display */
  .display {
    font-family: "Segoe UI", monospace;
    font-variant-numeric: tabular-nums;
    font-size: clamp(44px, 7vw, 76px);
    font-weight: 700;
    color: #fff;
    text-shadow: 0 0 18px rgba(255,255,255,0.25);
    letter-spacing: 4px;
    min-width: 280px;
    text-align: center;
  }
  .display.warning { color: #ffb347; text-shadow: 0 0 20px rgba(255,180,80,0.6); }
  .display.danger  { color: #ff5252; text-shadow: 0 0 22px rgba(255,80,80,0.65); animation: pulse 0.8s ease-in-out infinite; }
  @keyframes pulse { 50% { transform: scale(1.05); } }

  /* Controls */
  .controls {
    display: flex; flex-wrap: wrap;
    gap: 10px;
    align-items: center; justify-content: center;
    background: rgba(255,255,255,0.05);
    border: 1px solid rgba(255,255,255,0.1);
    backdrop-filter: blur(6px);
    padding: 14px 18px;
    border-radius: 16px;
    box-shadow: 0 4px 30px rgba(0,0,0,0.35);
  }
  .set-group { display: flex; align-items: center; gap: 6px; }
  .set-group label { font-size: 12px; opacity: 0.75; text-transform: uppercase; letter-spacing: 1px; }
  .set-group input {
    width: 62px;
    padding: 8px 6px;
    text-align: center;
    font-size: 16px;
    font-weight: 600;
    border-radius: 8px;
    border: 1px solid rgba(255,255,255,0.2);
    background: rgba(0,0,0,0.3);
    color: #fff;
    outline: none;
  }
  .set-group input:focus { border-color: #ffd76a; }
  button {
    padding: 10px 18px;
    border: 0;
    border-radius: 10px;
    font-size: 15px;
    font-weight: 700;
    letter-spacing: 0.5px;
    cursor: pointer;
    transition: transform 0.05s ease, filter 0.15s ease, opacity 0.15s ease;
    color: #fff;
    min-width: 92px;
  }
  button:active { transform: translateY(1px); }
  button:disabled { opacity: 0.4; cursor: not-allowed; }
  .btn-set   { background: #455a64; }
  .btn-start { background: linear-gradient(180deg, #43a047, #2e7d32); }
  .btn-stop  { background: linear-gradient(180deg, #1e88e5, #1565c0); }
  .btn-clear { background: linear-gradient(180deg, #e53935, #c62828); }
  button:hover { filter: brightness(1.1); }

  .hint { font-size: 12px; opacity: 0.55; margin-top: -8px; }

  /* Fireworks canvas overlay */
  #fireworks {
    position: fixed; inset: 0;
    z-index: 5;
    pointer-events: none;
  }

  .done-banner {
    position: fixed; top: 10%; left: 50%;
    transform: translateX(-50%) scale(0.8);
    z-index: 6;
    font-size: clamp(36px, 6vw, 64px);
    font-weight: 900;
    letter-spacing: 3px;
    color: #fff;
    text-shadow:
      0 0 12px #ffcb6a,
      0 0 24px #ff6ec7,
      0 0 36px #6ec8ff;
    opacity: 0;
    transition: opacity 0.4s ease, transform 0.4s ease;
    pointer-events: none;
    text-align: center;
  }
  .done-banner.show { opacity: 1; transform: translateX(-50%) scale(1); }

  @media (max-width: 520px) {
    .controls { padding: 12px; }
    button { min-width: 0; padding: 9px 12px; font-size: 14px; }
    .set-group input { width: 54px; }
  }
</style>
</head>
<body>
  <div class="stars"></div>
  <canvas id="fireworks"></canvas>
  <div class="done-banner" id="doneBanner">🎆 TIME'S UP! 🎆</div>

  <div class="app">
    <h1>Quiz Timer</h1>

    <div class="stage">
      <div class="cracker" id="cracker">
        <div class="fuse-wrap">
          <div class="fuse" id="fuse"></div>
          <div class="spark hidden" id="spark">
            <svg viewBox="0 0 64 64">
              <defs>
                <radialGradient id="g" cx="50%" cy="50%" r="50%">
                  <stop offset="0%" stop-color="#fff7c2"/>
                  <stop offset="40%" stop-color="#ffd24d"/>
                  <stop offset="75%" stop-color="#ff7a1a"/>
                  <stop offset="100%" stop-color="#b33400" stop-opacity="0"/>
                </radialGradient>
              </defs>
              <polygon fill="url(#g)" points="32,2 38,22 60,22 42,34 50,56 32,42 14,56 22,34 4,22 26,22"/>
            </svg>
          </div>
        </div>
        <div class="body">
          <div class="label">BOOM</div>
        </div>
      </div>
    </div>

    <div class="display" id="display">00:00</div>

    <div class="controls">
      <div class="set-group">
        <label for="minInput">Min</label>
        <input id="minInput" type="number" min="0" max="180" value="1" />
      </div>
      <div class="set-group">
        <label for="secInput">Sec</label>
        <input id="secInput" type="number" min="0" max="59" value="0" />
      </div>
      <button class="btn-set"   id="btnSet">Set</button>
      <button class="btn-start" id="btnStart">Start</button>
      <button class="btn-stop"  id="btnStop">Stop</button>
      <button class="btn-clear" id="btnClear">Clear</button>
    </div>
    <div class="hint">Tip: set minutes + seconds, hit <b>Set</b>, then <b>Start</b>. Press <b>Space</b> to start/stop.</div>
  </div>

<script>
(() => {
  'use strict';

  // ---------- Timer state ----------
  const el = {
    display: document.getElementById('display'),
    minInput: document.getElementById('minInput'),
    secInput: document.getElementById('secInput'),
    btnSet: document.getElementById('btnSet'),
    btnStart: document.getElementById('btnStart'),
    btnStop: document.getElementById('btnStop'),
    btnClear: document.getElementById('btnClear'),
    fuse: document.getElementById('fuse'),
    spark: document.getElementById('spark'),
    cracker: document.getElementById('cracker'),
    banner: document.getElementById('doneBanner'),
    canvas: document.getElementById('fireworks'),
  };

  let totalMs = 60_000;      // what was set
  let remainingMs = 60_000;  // current remaining
  let running = false;
  let lastTick = null;
  let rafId = null;

  function fmt(ms) {
    if (ms < 0) ms = 0;
    const totalSec = Math.ceil(ms / 1000);
    const m = Math.floor(totalSec / 60);
    const s = totalSec % 60;
    return `${String(m).padStart(2,'0')}:${String(s).padStart(2,'0')}`;
  }

  function render() {
    el.display.textContent = fmt(remainingMs);
    el.display.classList.remove('warning', 'danger');
    if (running && remainingMs <= 10_000) el.display.classList.add('danger');
    else if (running && remainingMs <= 30_000) el.display.classList.add('warning');

    // fuse length shrinks with remaining/total
    const pct = totalMs > 0 ? Math.max(0, remainingMs / totalMs) : 0;
    el.fuse.style.height = (pct * 100) + '%';

    // Spark sits at top of fuse when running
    if (running && remainingMs > 0) {
      el.spark.classList.remove('hidden');
      el.cracker.classList.add('shaking');
    } else {
      el.spark.classList.add('hidden');
      el.cracker.classList.remove('shaking');
    }
  }

  function tick(ts) {
    if (!running) return;
    if (lastTick === null) lastTick = ts;
    const dt = ts - lastTick;
    lastTick = ts;
    remainingMs -= dt;
    if (remainingMs <= 0) {
      remainingMs = 0;
      running = false;
      render();
      boom();
      return;
    }
    render();
    rafId = requestAnimationFrame(tick);
  }

  function setFromInputs() {
    const m = Math.max(0, parseInt(el.minInput.value || '0', 10));
    const s = Math.max(0, Math.min(59, parseInt(el.secInput.value || '0', 10)));
    const ms = (m * 60 + s) * 1000;
    totalMs = ms;
    remainingMs = ms;
    running = false;
    lastTick = null;
    el.banner.classList.remove('show');
    render();
  }

  function start() {
    if (remainingMs <= 0) setFromInputs();
    if (remainingMs <= 0) return;
    if (running) return;
    running = true;
    lastTick = null;
    el.banner.classList.remove('show');
    rafId = requestAnimationFrame(tick);
    render();
  }

  function stop() {
    running = false;
    if (rafId) cancelAnimationFrame(rafId);
    rafId = null;
    render();
  }

  function clear() {
    stop();
    totalMs = 0;
    remainingMs = 0;
    el.banner.classList.remove('show');
    stopFireworks();
    render();
  }

  el.btnSet.addEventListener('click', setFromInputs);
  el.btnStart.addEventListener('click', start);
  el.btnStop.addEventListener('click', stop);
  el.btnClear.addEventListener('click', clear);
  [el.minInput, el.secInput].forEach(i => {
    i.addEventListener('change', () => { if (!running) setFromInputs(); });
  });
  document.addEventListener('keydown', (e) => {
    if (e.target.tagName === 'INPUT') return;
    if (e.code === 'Space') { e.preventDefault(); running ? stop() : start(); }
    if (e.code === 'KeyR')  { clear(); }
  });

  // Initialize
  setFromInputs();

  // ---------- Fireworks ----------
  const cvs = el.canvas;
  const ctx = cvs.getContext('2d');
  let particles = [];
  let fwRaf = null;
  let fwStart = 0;
  let lastBurst = 0;

  function resize() {
    cvs.width = window.innerWidth;
    cvs.height = window.innerHeight;
  }
  resize();
  window.addEventListener('resize', resize);

  function rand(a, b) { return a + Math.random() * (b - a); }

  function spawnBurst(x, y, color) {
    const count = 70 + Math.floor(Math.random() * 40);
    const hue = color ?? Math.floor(Math.random() * 360);
    for (let i = 0; i < count; i++) {
      const angle = (Math.PI * 2 * i) / count + rand(-0.05, 0.05);
      const speed = rand(2, 6.5);
      particles.push({
        x, y,
        vx: Math.cos(angle) * speed,
        vy: Math.sin(angle) * speed,
        life: rand(70, 110),
        age: 0,
        hue: hue + rand(-20, 20),
        size: rand(2, 3.5),
      });
    }
  }

  function spawnTrail(tx, ty) {
    // rising trail that bursts
    let y = cvs.height + 10;
    let x = tx;
    const targetY = ty;
    const stepY = -8;
    const trailParticles = [];
    const interval = setInterval(() => {
      trailParticles.push({
        x: x + rand(-1,1),
        y,
        vx: rand(-0.4, 0.4),
        vy: rand(0.5, 1.5),
        life: 30, age: 0,
        hue: 40, size: 2,
      });
      particles.push(trailParticles[trailParticles.length - 1]);
      y += stepY;
      if (y <= targetY) {
        clearInterval(interval);
        spawnBurst(x, targetY);
      }
    }, 16);
  }

  function fwTick(ts) {
    if (!fwStart) fwStart = ts;
    const elapsed = ts - fwStart;

    // spawn a new rocket every ~500ms for ~6s
    if (ts - lastBurst > 420 && elapsed < 6500) {
      lastBurst = ts;
      const x = rand(cvs.width * 0.15, cvs.width * 0.85);
      const y = rand(cvs.height * 0.15, cvs.height * 0.45);
      spawnTrail(x, y);
    }

    // fade trail (pure black layer with alpha creates motion blur)
    ctx.globalCompositeOperation = 'source-over';
    ctx.fillStyle = 'rgba(3, 4, 15, 0.18)';
    ctx.fillRect(0, 0, cvs.width, cvs.height);

    ctx.globalCompositeOperation = 'lighter';
    for (let i = particles.length - 1; i >= 0; i--) {
      const p = particles[i];
      p.age++;
      p.vy += 0.035; // gravity
      p.vx *= 0.99;
      p.vy *= 0.99;
      p.x += p.vx;
      p.y += p.vy;
      const lifeLeft = 1 - p.age / p.life;
      if (lifeLeft <= 0) { particles.splice(i, 1); continue; }
      ctx.beginPath();
      ctx.fillStyle = `hsla(${p.hue}, 100%, ${55 + 20 * lifeLeft}%, ${lifeLeft})`;
      ctx.arc(p.x, p.y, p.size * (0.6 + lifeLeft * 0.6), 0, Math.PI * 2);
      ctx.fill();
    }

    if (particles.length > 0 || elapsed < 7000) {
      fwRaf = requestAnimationFrame(fwTick);
    } else {
      stopFireworks();
    }
  }

  function stopFireworks() {
    if (fwRaf) cancelAnimationFrame(fwRaf);
    fwRaf = null;
    fwStart = 0;
    lastBurst = 0;
    particles = [];
    ctx.clearRect(0, 0, cvs.width, cvs.height);
  }

  // ---------- Finale "boom" ----------
  function beep(freq, dur, type = 'sine', vol = 0.15) {
    try {
      const AC = window.AudioContext || window.webkitAudioContext;
      if (!AC) return;
      const ac = beep._ac || (beep._ac = new AC());
      const o = ac.createOscillator();
      const g = ac.createGain();
      o.type = type; o.frequency.value = freq;
      g.gain.value = vol;
      o.connect(g).connect(ac.destination);
      o.start();
      g.gain.exponentialRampToValueAtTime(0.0001, ac.currentTime + dur);
      o.stop(ac.currentTime + dur + 0.02);
    } catch (e) { /* ignore audio errors */ }
  }

  function boom() {
    // kick off fireworks
    stopFireworks();
    fwRaf = requestAnimationFrame(fwTick);
    // a few immediate bursts right away so it feels snappy
    spawnBurst(cvs.width * 0.5, cvs.height * 0.35);
    setTimeout(() => spawnBurst(cvs.width * 0.3, cvs.height * 0.28), 250);
    setTimeout(() => spawnBurst(cvs.width * 0.7, cvs.height * 0.3), 450);
    // sound
    beep(120, 0.6, 'sawtooth', 0.25);
    setTimeout(() => beep(80, 0.8, 'triangle', 0.2), 120);
    setTimeout(() => beep(600, 0.15, 'square', 0.08), 300);
    // banner
    el.banner.classList.add('show');
    setTimeout(() => el.banner.classList.remove('show'), 5000);
  }
})();
</script>
</body>
</html>
