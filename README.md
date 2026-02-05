<!doctype html>
<html lang="es">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>San Valentín 💖</title>
<style>
  :root{
    --bg1: #ff5f9e;
    --bg2: #b5179e;
    --card-bg: rgba(0,0,0,0.35);
    --btn-bg: #ffccd5;
    --btn-color: #800f2f;
  }

  html,body{
    height:100%;
    margin:0;
    font-family: Arial, Helvetica, sans-serif;
    -webkit-font-smoothing:antialiased;
    -moz-osx-font-smoothing:grayscale;
  }

  body {
    display:flex;
    align-items:center;
    justify-content:center;
    background: radial-gradient(circle, var(--bg1), var(--bg2));
    color: white;
    overflow:hidden;
    text-align:center;
  }

  canvas {
    position: absolute;
    inset: 0;
    z-index: 1;
    pointer-events: none;
  }

  .card {
    position: relative;
    z-index: 2;
    background: var(--card-bg);
    padding: 28px 36px;
    border-radius: 22px;
    box-shadow: 0 10px 40px rgba(0,0,0,0.35), 0 0 30px rgba(255,255,255,0.08) inset;
    max-width: 420px;
    width: calc(100% - 48px);
  }

  h1 {
    margin: 0 0 18px 0;
    font-size: 1.6rem;
    line-height: 1.2;
  }

  .controls {
    display:flex;
    gap:12px;
    justify-content:center;
    align-items:center;
    margin-top:6px;
  }

  button {
    font-size: 1rem;
    padding: 10px 22px;
    border: none;
    border-radius: 36px;
    cursor: pointer;
    background: var(--btn-bg);
    color: var(--btn-color);
    font-weight:700;
    transition: transform .14s ease, box-shadow .14s ease;
    box-shadow: 0 6px 18px rgba(0,0,0,0.2);
  }

  button:hover { transform: translateY(-3px) scale(1.03); }
  button:active { transform: translateY(0); }

  button:focus {
    outline: 3px solid rgba(255,255,255,0.18);
    outline-offset: 4px;
  }

  .small {
    font-size: .9rem;
    padding: 8px 14px;
    border-radius: 999px;
    background: rgba(255,255,255,0.12);
    color: #fff;
    border: 1px solid rgba(255,255,255,0.06);
  }

  .sr-only {
    position:absolute !important;
    height:1px; width:1px;
    overflow:hidden; clip:rect(1px,1px,1px,1px);
    white-space:nowrap;
  }

  @media (max-width:420px){
    .card { padding: 20px; border-radius: 18px; }
    h1 { font-size: 1.3rem; }
  }
</style>
</head>
<body>

<canvas id="fireworks" aria-hidden="true"></canvas>

<div class="card" role="region" aria-label="Tarjeta de San Valentín">
  <h1>
    Angie Zamata 💖<br>
    ¿Quieres ser mi San Valentín?
  </h1>

  <div class="controls">
    <button id="toggleBtn" aria-pressed="false">Sí 💘</button>
    <button id="stopBtn" class="small" aria-pressed="false">Detener</button>
  </div>

  <p class="sr-only" id="status">Fuegos artificiales inactivos.</p>
</div>

<audio id="music" src="morat.mp3" loop preload="auto"></audio>

<script>
  const canvas = document.getElementById('fireworks');
  const ctx = canvas.getContext('2d');
  let dpr = Math.max(1, window.devicePixelRatio || 1);
  function resize(){
    dpr = Math.max(1, window.devicePixelRatio || 1);
    const w = window.innerWidth;
    const h = window.innerHeight;
    canvas.style.width = w + 'px';
    canvas.style.height = h + 'px';
    canvas.width = Math.floor(w * dpr);
    canvas.height = Math.floor(h * dpr);
    ctx.setTransform(dpr, 0, 0, dpr, 0, 0);
  }
  window.addEventListener('resize', resize);
  resize();

  let particles = [];
  let intervalId = null;
  let running = false;
  function rand(min, max){ return Math.random()*(max-min)+min; }

  function drawHeart(x, y, r, color, alpha){
    ctx.save();
    ctx.translate(x, y);
    ctx.scale(r/20, r/20);
    ctx.globalAlpha = alpha;
    ctx.fillStyle = color;
    ctx.beginPath();
    ctx.moveTo(0, -10);
    ctx.bezierCurveTo(12, -30, 40, -18, 0, 20);
    ctx.bezierCurveTo(-40, -18, -12, -30, 0, -10);
    ctx.closePath();
    ctx.fill();
    ctx.restore();
    ctx.globalAlpha = 1;
  }

  function createFirework(x, y){
    const hue = Math.floor(rand(0, 360));
    const count = Math.floor(rand(80, 140));
    for (let i=0;i<count;i++){
      const speed = rand(1.5, 6);
      const angle = rand(0, Math.PI*2);
      const life = rand(60, 120);
      const radius = rand(1, 4);
      const isHeart = Math.random() < 0.12;
      particles.push({
        x, y,
        vx: Math.cos(angle)*speed,
        vy: Math.sin(angle)*speed,
        radius,
        life,
        age: 0,
        alpha: 1,
        color: `hsl(${hue + rand(-20,20)}, 90%, ${rand(45,65)}%)`,
        isHeart
      });
    }
  }

  function updateAndDraw(){
    ctx.clearRect(0,0,canvas.width,canvas.height);
    particles = particles.filter(p => {
      p.x += p.vx;
      p.y += p.vy + 0.02 * p.age;
      p.vx *= 0.996;
      p.vy *= 0.996;
      p.age++;
      p.alpha = Math.max(0, 1 - p.age/p.life);

      if (p.isHeart) {
        drawHeart(p.x, p.y, Math.max(2, p.radius*4), p.color, p.alpha);
      } else {
        ctx.globalAlpha = p.alpha;
        ctx.fillStyle = p.color;
        ctx.beginPath();
        ctx.arc(p.x, p.y, Math.max(0.8, p.radius), 0, Math.PI*2);
        ctx.fill();
        ctx.globalAlpha = 1;
      }

      return p.age < p.life;
    });
    requestAnimationFrame(updateAndDraw);
  }

  function startFireworks(){
    if (running) return;
    running = true;
    createFirework(window.innerWidth/2, window.innerHeight/2 - 60);
    intervalId = setInterval(() => {
      createFirework(rand(80, window.innerWidth-80), rand(80, window.innerHeight-120));
      if (Math.random() < 0.14) createFirework(rand(80, window.innerWidth-80), rand(80, window.innerHeight-120));
    }, 420);
    document.getElementById('status').textContent = 'Fuegos artificiales activos.';
  }

  function stopFireworks(){
    if (!running) return;
    running = false;
    clearInterval(intervalId);
    intervalId = null;
    document.getElementById('status').textContent = 'Fuegos artificiales detenidos.';
  }

  const audio = document.getElementById('music');
  const toggleBtn = document.getElementById('toggleBtn');
  const stopBtn = document.getElementById('stopBtn');

  function setToggleState(on){
    toggleBtn.setAttribute('aria-pressed', String(!!on));
    toggleBtn.textContent = on ? 'Sí 💘 (Activado)' : 'Sí 💘';
  }

  toggleBtn.addEventListener('click', async () => {
    if (!running) {
      try { await audio.play(); } catch(e){ console.warn('Audio play failed:', e); }
      startFireworks();
      setToggleState(true);
      stopBtn.setAttribute('aria-pressed', 'false');
    } else {
      await audio.pause();
      stopFireworks();
      setToggleState(false);
      stopBtn.setAttribute('aria-pressed', 'true');
    }
  });

  stopBtn.addEventListener('click', async () => {
    await audio.pause();
    stopFireworks();
    setToggleState(false);
    stopBtn.setAttribute('aria-pressed', 'true');
  });

  toggleBtn.addEventListener('keydown', (e) => {
    if (e.key === 'Enter' || e.key === ' ') {
      e.preventDefault();
      toggleBtn.click();
    }
  });

  requestAnimationFrame(updateAndDraw);
  window.addEventListener('pointerdown', (ev) => { createFirework(ev.clientX, ev.clientY); });
</script>

</body>
</html>

