# Presentation HTML Template

All presentations are built as self-contained HTML files. This template provides the slide framework, navigation, animations, charting, diagram support, and speaker notes.

## File Naming

Each presentation lives under a **project** folder (kebab-case slug) with a **`presentation`** subfolder:
- `Output/<project-name>/presentation/index.html`
- `Output/<project-name>/presentation/exec-summary.html` (optional)

## How to Use

1. Build the presentation using the base structure and component patterns below
2. Preview in browser canvas during creation
3. When finalised, save to `Output/<project-name>/presentation/index.html`
4. Open with `open` (macOS) to verify in the default browser
5. For sharing, push the folder to the presentations GitHub Pages repo

## Opening the File

```bash
open [your-workspace-path] user-Workspace/Output/<project-name>/presentation/index.html
```

## Base HTML Structure

Every presentation file follows this structure. Copy the full `<style>` and `<script>` blocks as-is, then customise the slides.

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>{{TITLE}}</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
<style>
  :root {
    --bg: #0a0a0a;
    --bg-slide: #0a0a0a;
    --text-primary: #ffffff;
    --text-secondary: #a0a0a0;
    --text-muted: #555555;
    --accent: #00d4ff;
    --accent-dim: #0a1a2e;
    --positive: #4ade80;
    --negative: #f87171;
    --warning: #fbbf24;
    --border: #1e1e1e;
    --card-bg: #141414;
    --font-display: 'Inter', -apple-system, BlinkMacSystemFont, sans-serif;
    --font-body: 'Inter', -apple-system, BlinkMacSystemFont, sans-serif;
  }

  * { margin: 0; padding: 0; box-sizing: border-box; }

  html, body {
    width: 100%;
    height: 100%;
    overflow: hidden;
    background: var(--bg);
    font-family: var(--font-body);
    color: var(--text-primary);
    -webkit-font-smoothing: antialiased;
  }

  /* ── Slide system ── */

  .deck {
    width: 100%;
    height: 100%;
    position: relative;
  }

  .slide {
    position: absolute;
    top: 0; left: 0;
    width: 100%;
    height: 100%;
    display: flex;
    flex-direction: column;
    justify-content: center;
    padding: 60px 100px;
    opacity: 0;
    pointer-events: none;
    transition: opacity 0.5s ease;
    background: var(--bg-slide);
  }

  .slide.active {
    opacity: 1;
    pointer-events: all;
  }

  .slide.active [data-animate] {
    animation-play-state: running;
  }

  /* ── Typography ── */

  .slide h1 {
    font-family: var(--font-display);
    font-size: 52px;
    font-weight: 700;
    letter-spacing: -1.5px;
    line-height: 1.1;
    color: var(--text-primary);
    margin-bottom: 24px;
  }

  .slide h2 {
    font-family: var(--font-display);
    font-size: 38px;
    font-weight: 600;
    letter-spacing: -0.5px;
    line-height: 1.2;
    color: var(--text-primary);
    margin-bottom: 20px;
  }

  .slide h3 {
    font-size: 13px;
    font-weight: 600;
    text-transform: uppercase;
    letter-spacing: 2px;
    color: var(--accent);
    margin-bottom: 16px;
  }

  .slide p {
    font-size: 20px;
    line-height: 1.65;
    color: var(--text-secondary);
    max-width: 720px;
  }

  .slide ul {
    list-style: none;
    padding: 0;
  }

  .slide ul li {
    font-size: 20px;
    line-height: 1.5;
    color: var(--text-secondary);
    padding: 10px 0 10px 28px;
    position: relative;
  }

  .slide ul li::before {
    content: '';
    position: absolute;
    left: 0;
    top: 18px;
    width: 8px;
    height: 8px;
    background: var(--accent);
    border-radius: 50%;
  }

  /* ── Utility classes ── */

  .accent { color: var(--accent); }
  .muted { color: var(--text-muted); font-size: 14px; }
  .source { color: var(--text-muted); font-size: 12px; margin-top: 16px; }
  .center { text-align: center; align-items: center; }

  /* ── Big stat ── */

  .stat {
    font-size: 80px;
    font-weight: 700;
    color: var(--accent);
    line-height: 1;
    margin-bottom: 12px;
    font-variant-numeric: tabular-nums;
  }

  .stat-label {
    font-size: 18px;
    color: var(--text-secondary);
  }

  .stat-group {
    display: flex;
    gap: 64px;
    margin-top: 24px;
  }

  /* ── Chart container ── */

  .chart-container {
    position: relative;
    width: 100%;
    max-width: 680px;
    height: 340px;
    margin-top: 24px;
  }

  .chart-container.wide {
    max-width: 840px;
    height: 400px;
  }

  /* ── Diagram container ── */

  .diagram {
    width: 100%;
    max-width: 760px;
    margin-top: 24px;
  }

  .diagram svg {
    width: 100%;
    height: auto;
  }

  /* ── Card grid ── */

  .card-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
    gap: 16px;
    margin-top: 24px;
    width: 100%;
  }

  .card-grid.cols-2 { grid-template-columns: 1fr 1fr; }
  .card-grid.cols-3 { grid-template-columns: 1fr 1fr 1fr; }

  .card {
    background: var(--card-bg);
    border: 1px solid var(--border);
    border-radius: 12px;
    padding: 28px;
  }

  .card h4 {
    font-size: 16px;
    font-weight: 600;
    color: var(--text-primary);
    margin-bottom: 8px;
  }

  .card p {
    font-size: 15px;
    color: var(--text-secondary);
    line-height: 1.55;
  }

  .card .card-stat {
    font-size: 32px;
    font-weight: 700;
    color: var(--accent);
    margin-bottom: 4px;
  }

  /* ── Two-column layout ── */

  .two-col {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 56px;
    align-items: center;
    width: 100%;
  }

  /* ── Metric row (inline stats) ── */

  .metric-row {
    display: flex;
    gap: 40px;
    margin-top: 24px;
  }

  .metric-item {
    text-align: center;
  }

  .metric-value {
    font-size: 36px;
    font-weight: 700;
    color: var(--text-primary);
  }

  .metric-label {
    font-size: 13px;
    color: var(--text-muted);
    margin-top: 4px;
  }

  .metric-change {
    font-size: 13px;
    font-weight: 500;
    margin-top: 4px;
  }

  .metric-change.up { color: var(--positive); }
  .metric-change.down { color: var(--negative); }
  .metric-change.flat { color: var(--text-muted); }

  /* ── Table ── */

  .slide table {
    width: 100%;
    max-width: 720px;
    border-collapse: collapse;
    font-size: 15px;
    margin-top: 20px;
  }

  .slide th {
    text-align: left;
    font-size: 12px;
    font-weight: 600;
    color: var(--text-muted);
    text-transform: uppercase;
    letter-spacing: 0.5px;
    padding: 10px 16px;
    border-bottom: 1px solid var(--border);
  }

  .slide td {
    padding: 12px 16px;
    border-bottom: 1px solid #111;
    color: var(--text-secondary);
  }

  /* ── Quote / callout ── */

  .callout {
    border-left: 3px solid var(--accent);
    padding: 16px 24px;
    margin-top: 24px;
    background: var(--accent-dim);
    border-radius: 0 8px 8px 0;
    max-width: 640px;
  }

  .callout p {
    font-size: 18px;
    font-style: italic;
    color: var(--text-primary);
  }

  /* ── Animations ── */

  [data-animate="fade-up"] {
    opacity: 0;
    transform: translateY(24px);
    animation: fadeUp 0.6s ease forwards;
    animation-play-state: paused;
  }

  [data-animate="fade-in"] {
    opacity: 0;
    animation: fadeIn 0.5s ease forwards;
    animation-play-state: paused;
  }

  [data-animate="scale-in"] {
    opacity: 0;
    transform: scale(0.92);
    animation: scaleIn 0.5s ease forwards;
    animation-play-state: paused;
  }

  [data-animate="slide-left"] {
    opacity: 0;
    transform: translateX(30px);
    animation: slideLeft 0.6s ease forwards;
    animation-play-state: paused;
  }

  @keyframes fadeUp {
    to { opacity: 1; transform: translateY(0); }
  }
  @keyframes fadeIn {
    to { opacity: 1; }
  }
  @keyframes scaleIn {
    to { opacity: 1; transform: scale(1); }
  }
  @keyframes slideLeft {
    to { opacity: 1; transform: translateX(0); }
  }

  /* Stagger delays (use data-delay="1" through "6") */
  [data-delay="1"] { animation-delay: 0.15s; }
  [data-delay="2"] { animation-delay: 0.3s; }
  [data-delay="3"] { animation-delay: 0.45s; }
  [data-delay="4"] { animation-delay: 0.6s; }
  [data-delay="5"] { animation-delay: 0.75s; }
  [data-delay="6"] { animation-delay: 0.9s; }

  /* ── Progress bar ── */

  .progress-bar {
    position: fixed;
    bottom: 0; left: 0;
    height: 3px;
    background: var(--accent);
    transition: width 0.4s ease;
    z-index: 100;
    opacity: 0.7;
  }

  /* ── Slide counter ── */

  .slide-counter {
    position: fixed;
    bottom: 14px; right: 24px;
    font-size: 13px;
    color: var(--text-muted);
    z-index: 100;
    font-variant-numeric: tabular-nums;
    opacity: 0.6;
  }

  /* ── Speaker notes panel ── */

  .notes-panel {
    position: fixed;
    bottom: 0; left: 0;
    width: 100%;
    background: rgba(0, 0, 0, 0.94);
    border-top: 1px solid var(--border);
    padding: 20px 48px;
    max-height: 30vh;
    overflow-y: auto;
    transform: translateY(100%);
    transition: transform 0.3s ease;
    z-index: 200;
    font-size: 15px;
    line-height: 1.65;
    color: var(--text-secondary);
  }

  .notes-panel.visible {
    transform: translateY(0);
  }

  .notes-label {
    font-size: 11px;
    font-weight: 600;
    text-transform: uppercase;
    letter-spacing: 1px;
    color: var(--accent);
    margin-bottom: 8px;
  }

  .slide aside.notes {
    display: none;
  }

  /* ── Title slide aurora ── */

  .slide-hero {
    overflow: hidden;
  }

  .slide-hero .aurora {
    position: absolute;
    inset: 0;
    z-index: 0;
    overflow: hidden;
    pointer-events: none;
  }

  .slide-hero .aurora-blob {
    position: absolute;
    border-radius: 50%;
    filter: blur(120px);
    opacity: 0;
    animation: auroraPulse 8s ease-in-out infinite, auroraDrift 20s ease-in-out infinite;
  }

  .slide-hero.active .aurora-blob {
    opacity: 1;
  }

  .slide-hero .aurora-blob:nth-child(1) {
    width: 600px; height: 600px;
    background: radial-gradient(circle, rgba(0, 212, 255, 0.35) 0%, transparent 70%);
    top: -15%; left: -10%;
    animation-delay: 0s;
  }

  .slide-hero .aurora-blob:nth-child(2) {
    width: 500px; height: 500px;
    background: radial-gradient(circle, rgba(139, 92, 246, 0.3) 0%, transparent 70%);
    top: 30%; right: -8%;
    animation-delay: 2s;
    animation-duration: 10s, 24s;
  }

  .slide-hero .aurora-blob:nth-child(3) {
    width: 450px; height: 450px;
    background: radial-gradient(circle, rgba(6, 182, 212, 0.25) 0%, transparent 70%);
    bottom: -10%; left: 25%;
    animation-delay: 4s;
    animation-duration: 12s, 18s;
  }

  .slide-hero .aurora-blob:nth-child(4) {
    width: 350px; height: 350px;
    background: radial-gradient(circle, rgba(236, 72, 153, 0.2) 0%, transparent 70%);
    top: 10%; right: 30%;
    animation-delay: 3s;
    animation-duration: 9s, 22s;
  }

  .slide-hero .aurora-blob:nth-child(5) {
    width: 400px; height: 400px;
    background: radial-gradient(circle, rgba(59, 130, 246, 0.25) 0%, transparent 70%);
    bottom: 5%; right: 10%;
    animation-delay: 1.5s;
    animation-duration: 11s, 16s;
  }

  @keyframes auroraPulse {
    0%, 100% { opacity: 0.3; transform: scale(1); }
    50% { opacity: 0.7; transform: scale(1.15); }
  }

  @keyframes auroraDrift {
    0% { translate: 0 0; }
    25% { translate: 60px -40px; }
    50% { translate: -30px 50px; }
    75% { translate: 40px 20px; }
    100% { translate: 0 0; }
  }

  .slide-hero .particles-canvas {
    position: absolute;
    inset: 0;
    z-index: 1;
    pointer-events: none;
  }

  .slide-hero h3,
  .slide-hero h1,
  .slide-hero p,
  .slide-hero img {
    position: relative;
    z-index: 2;
  }

  /* ── Print / PDF ── */

  @media print {
    .slide { position: relative; opacity: 1; pointer-events: all; page-break-after: always; }
    .progress-bar, .slide-counter, .notes-panel { display: none; }
    [data-animate] { opacity: 1 !important; transform: none !important; animation: none !important; }
    .aurora, .particles-canvas { display: none; }
  }
</style>

<!-- Chart.js (interactive charts with tooltips, click, zoom) -->
<script src="https://cdn.jsdelivr.net/npm/chart.js@4/dist/chart.umd.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/chartjs-plugin-zoom@2/dist/chartjs-plugin-zoom.umd.min.js"></script>
</head>
<body>

<div class="deck" id="deck">

  <!-- ─── SLIDE 1: Title (with aurora + particles) ─── -->
  <section class="slide slide-hero active" data-slide="1">
    <div class="aurora">
      <div class="aurora-blob"></div>
      <div class="aurora-blob"></div>
      <div class="aurora-blob"></div>
      <div class="aurora-blob"></div>
      <div class="aurora-blob"></div>
    </div>
    <canvas class="particles-canvas" id="particlesCanvas"></canvas>
    <h3 data-animate="fade-in">{{SECTION_LABEL}}</h3>
    <h1 data-animate="fade-up" data-delay="1">{{TITLE}}</h1>
    <p data-animate="fade-up" data-delay="2">{{SUBTITLE_OR_DATE}}</p>
    <aside class="notes">{{SPEAKER_NOTES}}</aside>
  </section>

  <!-- ─── SLIDE N: Content ─── -->
  <section class="slide" data-slide="2">
    <h2 data-animate="fade-up">{{SLIDE_TITLE}}</h2>
    <ul>
      <li data-animate="fade-up" data-delay="1">{{BULLET_1}}</li>
      <li data-animate="fade-up" data-delay="2">{{BULLET_2}}</li>
      <li data-animate="fade-up" data-delay="3">{{BULLET_3}}</li>
    </ul>
    <aside class="notes">{{SPEAKER_NOTES}}</aside>
  </section>

  <!-- Add more slides as needed -->

</div>

<!-- Progress bar -->
<div class="progress-bar" id="progress"></div>

<!-- Slide counter -->
<div class="slide-counter" id="counter"></div>

<!-- Speaker notes panel -->
<div class="notes-panel" id="notesPanel">
  <div class="notes-label">Speaker Notes</div>
  <div id="notesContent"></div>
</div>

<script>
(() => {
  const deck = document.getElementById('deck');
  const slides = deck.querySelectorAll('.slide');
  const progress = document.getElementById('progress');
  const counter = document.getElementById('counter');
  const notesPanel = document.getElementById('notesPanel');
  const notesContent = document.getElementById('notesContent');
  let current = 0;
  let notesVisible = false;

  function resetAnimations(slide) {
    slide.querySelectorAll('[data-animate]').forEach(el => {
      el.style.animation = 'none';
      el.offsetHeight;
      el.style.animation = '';
    });
  }

  function goTo(index) {
    if (index < 0 || index >= slides.length) return;

    slides[current].classList.remove('active');
    resetAnimations(slides[current]);

    current = index;
    slides[current].classList.add('active');

    progress.style.width = ((current + 1) / slides.length * 100) + '%';
    counter.textContent = (current + 1) + ' / ' + slides.length;

    const notes = slides[current].querySelector('aside.notes');
    notesContent.textContent = notes ? notes.textContent : '';

    // Trigger chart animations if the slide has charts
    const chartCanvases = slides[current].querySelectorAll('canvas[data-chart]');
    chartCanvases.forEach(canvas => {
      const chart = Chart.getChart(canvas);
      if (chart) chart.update('active');
    });
  }

  document.addEventListener('keydown', (e) => {
    switch (e.key) {
      case 'ArrowRight':
      case 'ArrowDown':
        e.preventDefault();
        goTo(current + 1);
        break;
      case 'ArrowLeft':
      case 'ArrowUp':
        e.preventDefault();
        goTo(current - 1);
        break;
      case 'n':
      case 'N':
        notesVisible = !notesVisible;
        notesPanel.classList.toggle('visible', notesVisible);
        break;
    }
  });

  goTo(0);
})();
</script>

<!-- Particles animation for title slide -->
<script>
(() => {
  const canvas = document.getElementById('particlesCanvas');
  if (!canvas) return;
  const ctx = canvas.getContext('2d');
  let w, h, particles, connDist = 140;

  const colours = [
    'rgba(0, 212, 255, ',
    'rgba(139, 92, 246, ',
    'rgba(59, 130, 246, ',
    'rgba(236, 72, 153, ',
    'rgba(6, 182, 212, ',
  ];

  function resize() {
    const rect = canvas.parentElement.getBoundingClientRect();
    w = canvas.width = rect.width;
    h = canvas.height = rect.height;
  }

  function createParticles() {
    const count = Math.floor((w * h) / 12000);
    particles = [];
    for (let i = 0; i < count; i++) {
      particles.push({
        x: Math.random() * w,
        y: Math.random() * h,
        vx: (Math.random() - 0.5) * 0.4,
        vy: (Math.random() - 0.5) * 0.4,
        r: Math.random() * 2 + 1,
        col: colours[Math.floor(Math.random() * colours.length)],
        alpha: Math.random() * 0.5 + 0.3,
        pulse: Math.random() * Math.PI * 2,
        pulseSpeed: Math.random() * 0.02 + 0.005,
      });
    }
  }

  function draw() {
    ctx.clearRect(0, 0, w, h);
    for (let i = 0; i < particles.length; i++) {
      const p = particles[i];
      p.x += p.vx;
      p.y += p.vy;
      p.pulse += p.pulseSpeed;
      if (p.x < -10) p.x = w + 10;
      if (p.x > w + 10) p.x = -10;
      if (p.y < -10) p.y = h + 10;
      if (p.y > h + 10) p.y = -10;
      const glow = p.alpha + Math.sin(p.pulse) * 0.15;
      for (let j = i + 1; j < particles.length; j++) {
        const q = particles[j];
        const dx = p.x - q.x;
        const dy = p.y - q.y;
        const dist = Math.sqrt(dx * dx + dy * dy);
        if (dist < connDist) {
          const lineAlpha = (1 - dist / connDist) * 0.15;
          ctx.beginPath();
          ctx.moveTo(p.x, p.y);
          ctx.lineTo(q.x, q.y);
          ctx.strokeStyle = p.col + lineAlpha + ')';
          ctx.lineWidth = 0.5;
          ctx.stroke();
        }
      }
      ctx.beginPath();
      ctx.arc(p.x, p.y, p.r, 0, Math.PI * 2);
      ctx.fillStyle = p.col + glow + ')';
      ctx.fill();
    }
    requestAnimationFrame(draw);
  }

  resize();
  createParticles();
  draw();
  window.addEventListener('resize', () => { resize(); createParticles(); });
})();
</script>

</body>
</html>
```

## Slide Components

Below are the HTML patterns for different slide types. Combine and adapt as needed.

---

### Title Slide (with aurora + particles)

Every presentation's first slide uses the `slide-hero` class, which adds the animated aurora background and floating particle constellation. This is the user's signature opening.

```html
<section class="slide slide-hero" data-slide="N">
  <div class="aurora">
    <div class="aurora-blob"></div>
    <div class="aurora-blob"></div>
    <div class="aurora-blob"></div>
    <div class="aurora-blob"></div>
    <div class="aurora-blob"></div>
  </div>
  <canvas class="particles-canvas" id="particlesCanvas"></canvas>
  <h3 data-animate="fade-in">SECTION LABEL</h3>
  <h1 data-animate="fade-up" data-delay="1">Main Title Here</h1>
  <p data-animate="fade-up" data-delay="2">Subtitle or date or context line</p>
  <aside class="notes">Speaker notes go here.</aside>
</section>
```

The aurora uses five colour blobs (cyan, purple, teal, pink, blue) that pulse and drift. The particles canvas draws floating dots with connecting lines. Both are scoped to `.slide-hero` only and hidden in print. The particles script is included at the bottom of the base HTML structure.

### Content Slide (headline + bullets)

```html
<section class="slide" data-slide="N">
  <h2 data-animate="fade-up">Slide Headline</h2>
  <ul>
    <li data-animate="fade-up" data-delay="1">First point, concise and specific</li>
    <li data-animate="fade-up" data-delay="2">Second point with <span class="accent">highlighted term</span></li>
    <li data-animate="fade-up" data-delay="3">Third point with supporting detail</li>
  </ul>
  <aside class="notes">Context for the presenter.</aside>
</section>
```

### Big Stat Slide

```html
<section class="slide" data-slide="N">
  <h3 data-animate="fade-in">KEY METRIC</h3>
  <div class="stat" data-animate="scale-in" data-delay="1">+42%</div>
  <div class="stat-label" data-animate="fade-up" data-delay="2">Campaign open rate since Q1</div>
  <aside class="notes">This stat is up from 29% in January.</aside>
</section>
```

### Multiple Stats

```html
<section class="slide" data-slide="N">
  <h2 data-animate="fade-up">Key Numbers</h2>
  <div class="stat-group">
    <div data-animate="scale-in" data-delay="1">
      <div class="stat">769</div>
      <div class="stat-label">Paying accounts</div>
    </div>
    <div data-animate="scale-in" data-delay="2">
      <div class="stat">235</div>
      <div class="stat-label">Product/day</div>
    </div>
    <div data-animate="scale-in" data-delay="3">
      <div class="stat">97%</div>
      <div class="stat-label">Delivery rate</div>
    </div>
  </div>
  <aside class="notes">All metrics trending up week over week.</aside>
</section>
```

### Two-Column Layout

Use for content + visual, or two parallel ideas:

```html
<section class="slide" data-slide="N">
  <div class="two-col">
    <div>
      <h2 data-animate="fade-up">Left Column</h2>
      <p data-animate="fade-up" data-delay="1">Text or bullets on the left, visual on the right.</p>
    </div>
    <div data-animate="slide-left" data-delay="2">
      <div class="chart-container">
        <canvas id="chartN" data-chart></canvas>
      </div>
    </div>
  </div>
  <aside class="notes">The chart shows weekly trend data.</aside>
</section>
```

### Card Grid

```html
<section class="slide" data-slide="N">
  <h2 data-animate="fade-up">Three Pillars</h2>
  <div class="card-grid cols-3">
    <div class="card" data-animate="fade-up" data-delay="1">
      <h4>Integration</h4>
      <p>Deep connectivity with monday CRM and Work Management</p>
    </div>
    <div class="card" data-animate="fade-up" data-delay="2">
      <h4>AI-First</h4>
      <p>Intelligent automation at the core of every feature</p>
    </div>
    <div class="card" data-animate="fade-up" data-delay="3">
      <h4>Simplicity</h4>
      <p>Built for marketers, not marketing technologists</p>
    </div>
  </div>
  <aside class="notes">These are our three strategic pillars for H2.</aside>
</section>
```

### Metric Row

```html
<section class="slide" data-slide="N">
  <h2 data-animate="fade-up">Weekly Performance</h2>
  <div class="metric-row" data-animate="fade-up" data-delay="1">
    <div class="metric-item">
      <div class="metric-value">1,247</div>
      <div class="metric-label">Active users</div>
      <div class="metric-change up">+8% WoW</div>
    </div>
    <div class="metric-item">
      <div class="metric-value">89%</div>
      <div class="metric-label">Activation</div>
      <div class="metric-change up">+3pp</div>
    </div>
    <div class="metric-item">
      <div class="metric-value">4.2%</div>
      <div class="metric-label">Churn</div>
      <div class="metric-change down">+0.5pp</div>
    </div>
  </div>
  <aside class="notes">Churn uptick driven by seasonal trial expiry.</aside>
</section>
```

### Chart Slide (Chart.js)

The chart is initialised in a `<script>` block at the bottom of the HTML. Charts are interactive: hover for tooltips, scroll to zoom, click to toggle datasets.

```html
<section class="slide" data-slide="N">
  <h2 data-animate="fade-up">Growth Trajectory</h2>
  <div class="chart-container wide" data-animate="scale-in" data-delay="1">
    <canvas id="growthChart" data-chart></canvas>
  </div>
  <p class="source" data-animate="fade-in" data-delay="2">Source: Internal analytics, March 2026</p>
  <aside class="notes">The inflection point in Feb correlates with the AI launch.</aside>
</section>
```

Chart initialisation (add before the closing `</body>` tag, inside the main `<script>` block or in a separate one):

```javascript
// Example: Line chart with zoom
new Chart(document.getElementById('growthChart'), {
  type: 'line',
  data: {
    labels: ['Jan', 'Feb', 'Mar', 'Apr', 'May'],
    datasets: [{
      label: 'Paying accounts',
      data: [680, 710, 745, 769, null],
      borderColor: '#00d4ff',
      backgroundColor: 'rgba(0, 212, 255, 0.08)',
      fill: true,
      tension: 0.3,
      pointRadius: 4,
      pointHoverRadius: 7,
    }]
  },
  options: {
    responsive: true,
    maintainAspectRatio: false,
    animation: { duration: 800, easing: 'easeOutQuart' },
    plugins: {
      legend: { display: false },
      tooltip: {
        backgroundColor: '#141414',
        titleColor: '#fff',
        bodyColor: '#a0a0a0',
        borderColor: '#1e1e1e',
        borderWidth: 1,
        padding: 12,
        cornerRadius: 8,
      },
      zoom: {
        zoom: {
          wheel: { enabled: true },
          pinch: { enabled: true },
          mode: 'x',
        },
        pan: { enabled: true, mode: 'x' },
      },
    },
    scales: {
      x: {
        ticks: { color: '#555' },
        grid: { color: '#1a1a1a' },
      },
      y: {
        ticks: { color: '#555' },
        grid: { color: '#1a1a1a' },
      },
    },
  },
});
```

### Diagram Slide (SVG)

For flowcharts, user journeys, and architecture diagrams. Describe the diagram to the agent and it will generate the SVG.

```html
<section class="slide" data-slide="N">
  <h2 data-animate="fade-up">User Journey</h2>
  <div class="diagram" data-animate="fade-in" data-delay="1">
    <svg viewBox="0 0 800 300" xmlns="http://www.w3.org/2000/svg">
      <!-- SVG content generated from description -->
    </svg>
  </div>
  <aside class="notes">Simplified journey, excludes edge cases.</aside>
</section>
```

### Callout / Quote

```html
<section class="slide" data-slide="N">
  <h2 data-animate="fade-up">What Users Are Saying</h2>
  <div class="callout" data-animate="slide-left" data-delay="1">
    <p>"The integration with our CRM changed how we think about campaign targeting."</p>
  </div>
  <p class="muted" data-animate="fade-in" data-delay="2">Head of Marketing, Mid-market SaaS</p>
  <aside class="notes">From the March user interview batch.</aside>
</section>
```

### Table Slide

```html
<section class="slide" data-slide="N">
  <h2 data-animate="fade-up">Competitive Landscape</h2>
  <table data-animate="fade-up" data-delay="1">
    <thead>
      <tr><th>Feature</th><th>Us</th><th>Mailchimp</th><th>HubSpot</th></tr>
    </thead>
    <tbody>
      <tr><td>CRM integration</td><td class="accent">Native</td><td>API only</td><td>Native</td></tr>
      <tr><td>AI content</td><td class="accent">Built-in</td><td>Add-on</td><td>Beta</td></tr>
      <tr><td>Pricing (SMB)</td><td class="accent">Included</td><td>Separate</td><td>$800/mo+</td></tr>
    </tbody>
  </table>
  <aside class="notes">Our pricing advantage is significant at the SMB tier.</aside>
</section>
```

---

## Chart Styling Defaults

All charts should follow this colour palette and styling:

| Element | Value |
|---------|-------|
| Primary dataset | `#00d4ff` (accent) with `rgba(0, 212, 255, 0.08)` fill |
| Secondary dataset | `#c084fc` (purple) |
| Tertiary dataset | `#4ade80` (green) |
| Negative/alert | `#f87171` (red) |
| Grid lines | `#1a1a1a` |
| Tick labels | `#555555` |
| Tooltip background | `#141414` with `#1e1e1e` border |
| Font | Inherit from page (Inter) |
| Animation | 800ms, easeOutQuart |

Enable zoom and pan on all charts by default. Include `chartjs-plugin-zoom` in the head.

For bar charts, use `borderRadius: 4` for rounded corners.
For doughnut/pie charts, use `cutout: '65%'` for a clean ring.

## Diagram Generation

When a slide needs a diagram (flowchart, journey map, architecture, process):

1. The agent receives a description from the user (e.g. "show the user flow from signup to first campaign")
2. The agent generates an SVG with these styling rules:
   - Background: transparent (inherits slide background)
   - Node fill: `#141414` with `#1e1e1e` border
   - Node text: `#e5e5e5`
   - Arrows/lines: `#555555` with arrowheads
   - Highlighted nodes: `#0a1a2e` fill with `#00d4ff` border
   - Labels on arrows: `#888888`, 12px
   - Rounded corners on nodes: 8px

Alternatively, Mermaid can be used for quick diagrams. Include the Mermaid CDN and use `<pre class="mermaid">` blocks.

## Speaker Notes

- Embedded as `<aside class="notes">` inside each slide section
- Hidden by default (CSS `display: none`)
- Press `N` to toggle the notes panel at the bottom of the screen
- The notes panel shows notes for the current slide only
- When sharing the URL, recipients see slides only (notes are hidden)

## Navigation

| Key | Action |
|-----|--------|
| `→` or `↓` | Next slide |
| `←` or `↑` | Previous slide |
| `N` | Toggle speaker notes |

## Design Principles

- **Black, minimal.** Dark backgrounds, thin borders, generous spacing
- **Cyan (#00d4ff) is the primary accent.** Highlights, active elements, key data
- **One idea per slide.** Two ideas = two slides
- **Data speaks visually.** Charts over tables, annotate the insight in cyan
- **Hierarchy is clarity.** Large headline, then supporting detail, then source (small, muted)
- **Easy on the eye.** Dark background reduces fatigue. Let whitespace breathe
- **No decoration for decoration's sake.** Every visual element carries meaning

## Exec Summary (PDF-ready)

When requested, generate a condensed 1-page exec summary as a separate HTML file (`exec-summary.html`). Structure:

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>{{TITLE}} — Executive Summary</title>
<style>
  /* Use the routine HTML template styles (skills/system/routine-html-template.md) */
  /* Single page, print-optimised */
  @media print {
    body { max-width: 100%; padding: 20px; }
  }
</style>
</head>
<body>
  <div class="header">
    <h1>{{TITLE}}</h1>
    <div class="subtitle">Executive Summary · {{DATE}}</div>
  </div>

  <div class="section">
    <div class="section-title">Key Takeaways</div>
    <!-- 3-5 bullet points summarising the core message -->
  </div>

  <div class="section">
    <div class="section-title">Key Metric</div>
    <!-- One chart or metric grid showing the most important data point -->
  </div>

  <div class="section">
    <div class="section-title">Recommendation / Next Steps</div>
    <!-- 2-3 action items -->
  </div>

  <div class="footer">{{DATE}} · {{AUTHOR}}</div>
</body>
</html>
```

The exec summary should:
- Pull the most important data point and 3-5 key messages from the presentation
- Be reviewed by the user before being finalised
- Use the routine HTML template CSS for consistency
- Print cleanly to a single A4 page
