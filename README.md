# Radek-plinko
<!--
  Plinko Casino — single-file HTML/CSS/JS
  Přepsáno z původního slotu na hru ve stylu Plinko.
  Vlož tento soubor jako `index.html` do repozitáře a publikuj přes GitHub Pages.

  Funkce:
  - Plinko deska nakreslená v canvasu
  - Fyzika koule (jednoduchá integrace + odrazy od kolíků)
  - Sloty dole s multiplikátory
  - Kredity, sázka, drop/auto
  - Responzivní, jednoduché a snadno rozšířitelné
-->

<!doctype html>
<html lang="cs">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Plinko — Hrací automat (Plinko styl)</title>
  <meta name="description" content="Jednoduchý Plinko — vlož na GitHub Pages" />
  <style>
    :root{ --bg:#071025; --panel:#0f1724; --accent:#ffbf00; --muted:#9aa4b2; color-scheme: dark; }
    *{box-sizing:border-box}
    body{margin:0; font-family:Inter, system-ui, -apple-system, 'Segoe UI', Roboto, Arial; background:linear-gradient(180deg,#041024,#07142a); color:#e6eef6; display:flex; align-items:center; justify-content:center; padding:28px;}
    .frame{width:980px; max-width:96vw; display:grid; grid-template-columns:1fr 300px; gap:20px;}
    .panel{background:linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01)); padding:16px; border-radius:12px}
    h1{margin:0 0 6px; font-size:20px}
    p.lead{color:var(--muted); margin:0 0 12px}
    canvas{width:100%; height:640px; display:block; background:linear-gradient(180deg,#083046,#042033); border-radius:8px; box-shadow:0 14px 40px rgba(2,6,23,0.6)}

    .controls{display:flex; gap:8px; margin-top:10px; align-items:center}
    .btn{background:var(--accent); border:none; padding:10px 14px; border-radius:8px; cursor:pointer; font-weight:700; color:#071025}
    .btn.ghost{background:transparent; color:var(--muted); border:1px solid rgba(255,255,255,0.04)}
    .sidebar{display:flex; flex-direction:column; gap:8px}
    .stat{display:flex; justify-content:space-between; color:var(--muted)}
    .slots{display:flex; gap:6px; margin-top:12px}
    .slot{flex:1; padding:8px; text-align:center; border-radius:6px; background:rgba(255,255,255,0.03)}
    .slot .mult{font-weight:800}
    footer{color:var(--muted); margin-top:10px}

    @media (max-width:920px){ .frame{grid-template-columns:1fr} canvas{height:520px} }
  </style>
</head>
<body>
  <div class="frame">
    <div class="panel">
      <h1>Plinko — Casino styl</h1>
      <p class="lead">Hodíš kulkou do plinka, ta se odráží od kolíků a dopadne do jednoho ze slotů dole. Každý slot má svůj multiplikátor.</p>

      <canvas id="board" width="800" height="640" role="img" aria-label="Plinko deska"></canvas>

      <div class="controls">
        <button id="dropBtn" class="btn">POUŠTĚT</button>
        <button id="autoBtn" class="btn ghost">AUTO</button>
        <label style="margin-left:auto;color:var(--muted)">Sázka
          <select id="bet" style="margin-left:8px">
            <option value="1">1</option>
            <option value="2">2</option>
            <option value="5">5</option>
            <option value="10">10</option>
          </select>
        </label>
      </div>
    </div>

    <aside class="panel sidebar" aria-labelledby="status">
      <div id="status"><strong>Stav</strong></div>
      <div class="stat"><span>Bank</span><span id="bank">100</span></div>
      <div class="stat"><span>Sázka</span><span id="betDisplay">1</span></div>
      <div class="stat"><span>Poslední výhra</span><span id="lastWin">0</span></div>

      <div style="height:1px;background:rgba(255,255,255,0.03);margin:8px 0"></div>
      <div><strong>Sloty</strong></div>
      <div class="slots" id="slots"></div>

      <div style="height:1px;background:rgba(255,255,255,0.03);margin:8px 0"></div>
      <div style="color:var(--muted);font-size:13px">Tip: klikni kamkoliv na horní okraj plochy pro umístění dropu.</div>
      <footer>Verze: Plinko demo — upravitelné</footer>
    </aside>
  </div>

  <script>
    // --- Config ---
    const SLOT_COUNT = 9;
    const PEG_ROWS = 9; // number of peg rows
    const PEG_SPACING = 64; // px
    const PEG_RADIUS = 6;
    const BALL_RADIUS = 10;
    const GRAVITY = 1200; // px/s^2
    const FRICTION = 0.995;

    const canvas = document.getElementById('board');
    const ctx = canvas.getContext('2d');
    let W = canvas.width, H = canvas.height;

    // UI
    const dropBtn = document.getElementById('dropBtn');
    const autoBtn = document.getElementById('autoBtn');
    const betSelect = document.getElementById('bet');
    const bankEl = document.getElementById('bank');
    const betDisplay = document.getElementById('betDisplay');
    const lastWinEl = document.getElementById('lastWin');
    const slotsEl = document.getElementById('slots');

    let bank = 100;
    let auto = false;
    let balls = []; // active balls

    // create slot multipliers (example: outer slots higher)
    const multipliers = [];
    for(let i=0;i<SLOT_COUNT;i++){
      const center = (SLOT_COUNT-1)/2;
      const dist = Math.abs(i-center);
      // example curve: center low, edges high
      multipliers.push(Math.max(1, Math.round((1 + dist*0.8)*10)/10));
    }

    // render slot UI
    function renderSlotUI(){
      slotsEl.innerHTML='';
      for(let i=0;i<SLOT_COUNT;i++){
        const el = document.createElement('div'); el.className='slot';
        el.innerHTML = `<div>Slot ${i+1}</div><div class='mult'>x ${multipliers[i]}</div>`;
        slotsEl.appendChild(el);
      }
    }
    renderSlotUI();

    function resize(){
      // maintain logical size but scale canvas for crispness
      const ratio = window.devicePixelRatio || 1;
      W = Math.min(1100, Math.max(600, Math.floor(window.innerWidth*0.6)));
      H = Math.floor(W * 0.8);
      canvas.width = W * ratio; canvas.height = H * ratio; canvas.style.width = W + 'px'; canvas.style.height = H + 'px';
      ctx.setTransform(ratio,0,0,ratio,0,0);
    }
    resize(); window.addEventListener('resize', resize);

    // Build peg positions in triangular grid
    let pegs = [];
    function buildPegs(){
      pegs = [];
      const top = 80;
      const left = 60;
      const cols = Math.floor((W - left*2) / PEG_SPACING) + 1;
      for(let row=0; row<PEG_ROWS; row++){
        const y = top + row * (PEG_SPACING * 0.8);
        const offset = (row%2===0) ? 0 : PEG_SPACING/2;
        for(let col=0; col<cols; col++){
          const x = left + offset + col*PEG_SPACING;
          if(x > 40 && x < W-40) pegs.push({x,y});
        }
      }
    }
    buildPegs();

    // Slot boundaries
    function slotForX(x){
      const slotWidth = (W - 120) / SLOT_COUNT;
      const left = 60;
      let idx = Math.floor((x - left) / slotWidth);
      if(idx<0) idx=0; if(idx>=SLOT_COUNT) idx=SLOT_COUNT-1; return idx;
    }

    // Ball class
    class Ball{
      constructor(x){ this.x = x; this.y = 40; this.vx = (Math.random()-0.5)*80; this.vy = 0; this.radius = BALL_RADIUS; this.alive = true; }
      step(dt){
        this.vy += GRAVITY * dt;
        this.x += this.vx * dt;
        this.y += this.vy * dt;
        this.vx *= FRICTION;

        // collisions with pegs
        for(const p of pegs){
          const dx = this.x - p.x, dy = this.y - p.y;
          const dist = Math.hypot(dx,dy);
          const minDist = this.radius + PEG_RADIUS;
          if(dist < minDist && dist>0){
            // push out
            const nx = dx / dist, ny = dy / dist;
            const overlap = minDist - dist;
            this.x += nx * overlap;
            this.y += ny * overlap;
            // reflect velocity
            const vDotN = this.vx*nx + this.vy*ny;
            this.vx -= 1.8 * vDotN * nx;
            this.vy -= 1.8 * vDotN * ny;
          }
        }

        // walls
        if(this.x < 40){ this.x = 40; this.vx = Math.abs(this.vx)*0.6; }
        if(this.x > W-40){ this.x = W-40; this.vx = -Math.abs(this.vx)*0.6; }

        // bottom
        if(this.y > H - 100){
          this.alive = false;
          // determine slot
          const idx = slotForX(this.x);
          this.slot = idx;
        }
      }
      draw(ctx){ ctx.beginPath(); ctx.fillStyle='#ffd166'; ctx.arc(this.x,this.y,this.radius,0,Math.PI*2); ctx.fill(); ctx.strokeStyle='rgba(0,0,0,0.2)'; ctx.stroke(); }
    }

    // Simulation loop
    let last = performance.now();
    function loop(now){
      const dt = Math.min(0.03, (now - last)/1000); last = now;
      // step balls
      for(const b of balls) if(b.alive) b.step(dt);
      // handle finished balls
      for(const b of balls.filter(b=>!b.alive && !b.settled)){ b.settled = true; onBallSettled(b); }

      draw();
      requestAnimationFrame(loop);
    }
    requestAnimationFrame(loop);

    function draw(){
      // clear
      ctx.clearRect(0,0,W,H);
      // background gradient is via canvas already set in CSS; paint subtle overlay
      // draw pegs
      for(const p of pegs){ ctx.beginPath(); ctx.fillStyle='rgba(255,255,255,0.06)'; ctx.arc(p.x,p.y,PEG_RADIUS,0,Math.PI*2); ctx.fill(); }

      // draw slots area
      const slotTop = H - 100; const left = 60; const slotWidth = (W - 120) / SLOT_COUNT;
      // dividers
      ctx.fillStyle='rgba(255,255,255,0.02)'; ctx.fillRect(left, slotTop, W-120, 6);
      for(let i=0;i<SLOT_COUNT;i++){
        const x = left + i*slotWidth;
        ctx.fillStyle='rgba(255,255,255,0.03)'; ctx.fillRect(x, slotTop, 4, 60);
        // label
        ctx.fillStyle='#e6eef6'; ctx.font='14px Inter,Arial'; ctx.textAlign='center';
        ctx.fillText('x' + multipliers[i], x + slotWidth/2, slotTop + 42);
      }

      // draw balls
      for(const b of balls) b.draw(ctx);
    }

    // When ball settles in slot
    function onBallSettled(ball){
      const idx = ball.slot; const mult = multipliers[idx];
      const bet = Number(betSelect.value);
      const payout = Math.round(bet * mult);
      bank += payout;
      lastWinEl.textContent = payout;
      bankEl.textContent = bank;
      // show small flash near slot (simple)
      flashSlot(idx, payout);
    }

    function flashSlot(idx, payout){
      // draw a quick highlight on slot area
      const left = 60; const slotWidth = (W - 120) / SLOT_COUNT; const x = left + idx*slotWidth;
      ctx.save(); ctx.fillStyle='rgba(255,191,0,0.14)'; ctx.fillRect(x, H-100, slotWidth, 60); ctx.restore();
      // keep for short time by redrawing (no complex animation)
      setTimeout(()=>{},400);
    }

    // Controls
    dropBtn.addEventListener('click', ()=>{ attemptDrop(W/2); });
    canvas.addEventListener('click', (e)=>{
      // map click to canvas coords
      const rect = canvas.getBoundingClientRect(); const x = (e.clientX - rect.left) * (canvas.width / rect.width) / (window.devicePixelRatio || 1);
      attemptDrop(x);
    });

    autoBtn.addEventListener('click', ()=>{
      auto = !auto; autoBtn.textContent = auto ? 'STOP' : 'AUTO';
      if(auto) autoDropLoop();
    });

    betSelect.addEventListener('change', ()=>{ betDisplay.textContent = betSelect.value; });

    function attemptDrop(x){
      const bet = Number(betSelect.value);
      if(bank < bet){ alert('Nedostatek kreditu'); return; }
      bank -= bet; bankEl.textContent = bank;
      const ratio = window.devicePixelRatio || 1;
      const canvasX = x; // already in canvas space
      const b = new Ball(canvasX);
      balls.push(b);
    }

    async function autoDropLoop(){
      while(auto){
        attemptDrop(Math.random() * (W - 120) + 60);
        // wait until at least one ball settled or small delay
        await new Promise(r=>setTimeout(r, 450));
        // stop if bank insufficient
        if(bank < Number(betSelect.value)){ auto = false; autoBtn.textContent='AUTO'; break; }
      }
    }

    // initial UI
    bankEl.textContent = bank; betDisplay.textContent = betSelect.value;

    // rebuild pegs on resize
    window.addEventListener('resize', ()=>{ buildPegs(); });
  </script>
</body>
</html>
