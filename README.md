# Radek-website0
<!--
  Slot Machine - Single-file HTML/CSS/JS
  Drop this file into a GitHub repo as `index.html` and publish with GitHub Pages (or open locally).

  Features:
  - Pure front-end: no external assets (uses emoji for symbols)
  - 3 reels, animations, random results
  - Credits, bet size, spin / autoplay, simple payouts
  - Accessible controls and keyboard support

  To use on GitHub pages: create a repo, add this file as `index.html`, then enable Pages in repo settings.
-->

<!doctype html>
<html lang="cs">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Hrací automat — Demo</title>
  <meta name="description" content="Jednoduchý front-end hrací automat (slot machine) — upravitelné, vhodné na GitHub Pages" />
  <style>
    :root{
      --bg:#0b1020;
      --card:#0f1724;
      --accent:#ffbf00;
      --muted:#9aa4b2;
      --glass: rgba(255,255,255,0.03);
      --win:#28c76f;
      font-family: Inter, system-ui, -apple-system, 'Segoe UI', Roboto, 'Helvetica Neue', Arial;
    }
    *{box-sizing:border-box}
    html,body{height:100%}
    body{
      margin:0; background: linear-gradient(180deg,#071025 0%, #081426 60%); color:#e6eef6; display:flex; align-items:center; justify-content:center; padding:32px;
    }

    .frame{
      width:980px; max-width:96vw; background:linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01)); border-radius:16px; padding:28px; box-shadow:0 10px 40px rgba(2,6,23,0.6); display:grid; grid-template-columns:1fr 320px; gap:24px;
    }

    .left{padding:18px}
    h1{margin:0 0 6px; font-size:22px; letter-spacing:0.6px}
    p.lead{color:var(--muted); margin-top:0; margin-bottom:18px}

    /* Slot cabinet */
    .cabinet{background: linear-gradient(180deg,#0b2740, #05202b); border-radius:12px; padding:18px; box-shadow: inset 0 -6px 20px rgba(0,0,0,0.6), 0 6px 22px rgba(2,6,23,0.6);}

    .reels{display:flex; gap:12px; justify-content:center; align-items:center;}
    .reel{
      width:160px; height:160px; background:var(--glass); border-radius:12px; overflow:hidden; position:relative; border:2px solid rgba(255,255,255,0.04);
      display:flex; align-items:center; justify-content:center; box-shadow:0 6px 12px rgba(2,6,23,0.45) inset;
    }

    /* The strip that moves inside the reel */
    .strip{position:absolute; top:0; left:0; right:0;}
    .symbol{height:160px; display:flex; align-items:center; justify-content:center; font-size:56px; user-select:none}
    .symbol .label{display:block; font-size:12px; color:var(--muted); margin-top:6px}

    /* Window mask */
    .mask{position:absolute; inset:0; pointer-events:none; box-shadow: inset 0 30px 30px rgba(0,0,0,0.6);}

    /* Controls */
    .controls{margin-top:16px; display:flex; gap:10px; align-items:center}
    .btn{background:linear-gradient(180deg,#ffcf4a,#ffb800); border:none; padding:10px 16px; border-radius:10px; cursor:pointer; font-weight:700; color:#0b1020; box-shadow:0 8px 20px rgba(255,184,0,0.12);}
    .btn.secondary{background:transparent; border:1px solid rgba(255,255,255,0.06); color:var(--muted); font-weight:600}

    /* Sidebar */
    .sidebar{padding:18px; background:linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01)); border-radius:12px}
    .stat{display:flex; justify-content:space-between; margin-bottom:10px; color:var(--muted)}
    .big{font-size:20px; font-weight:700}

    /* Win flash */
    .flash{
      position:absolute; inset:0; display:flex; align-items:center; justify-content:center; font-size:28px; font-weight:800; color:var(--bg); background:var(--accent); border-radius:12px; opacity:0; transform:scale(0.8); transition:all .35s ease;
    }
    .flash.show{opacity:1; transform:scale(1)}

    footer{margin-top:14px; color:var(--muted); font-size:13px}

    /* Mobile */
    @media (max-width:880px){
      .frame{grid-template-columns:1fr;}
      .reel{width:110px; height:110px}
      .symbol{font-size:40px}
    }
  </style>
</head>
<body>
  <div class="frame" role="application" aria-label="Hrací automat">
    <div class="left">
      <h1>Hrací automat — Demo</h1>
      <p class="lead">Jednoduchý, responzivní slot machine. Uprav si symboly, výplaty nebo animace a publikuj na GitHub Pages.</p>

      <div class="cabinet" aria-hidden="false">
        <div style="position:relative">
          <div class="reels" id="reels" aria-live="polite">
            <!-- Three reels -->
            <div class="reel" data-index="0">
              <div class="strip" aria-hidden="true"></div>
              <div class="mask"></div>
            </div>
            <div class="reel" data-index="1">
              <div class="strip" aria-hidden="true"></div>
              <div class="mask"></div>
            </div>
            <div class="reel" data-index="2">
              <div class="strip" aria-hidden="true"></div>
              <div class="mask"></div>
            </div>
          </div>

          <div class="flash" id="flash">VÝHRA!</div>
        </div>

        <div class="controls">
          <button class="btn" id="spinBtn">SPIN</button>
          <button class="btn secondary" id="autoBtn">AUTO</button>
          <div style="margin-left:auto; display:flex; gap:8px; align-items:center">
            <label for="bet" style="color:var(--muted); font-size:13px">Sázka</label>
            <select id="bet" aria-label="Sázka">
              <option value="1">1</option>
              <option value="2">2</option>
              <option value="5">5</option>
              <option value="10">10</option>
            </select>
          </div>
        </div>

      </div>

      <footer>Tip: symboly jsou emoji — můžeš je nahradit obrázky nebo SVG pro reálnější vzhled.</footer>
    </div>

    <aside class="sidebar" aria-labelledby="status">
      <div id="status" style="margin-bottom:8px"><strong>Stav</strong></div>
      <div class="stat"><span>Bank</span><span id="bank" class="big">100</span></div>
      <div class="stat"><span>Sázka</span><span id="betDisplay">1</span></div>
      <div class="stat"><span>Poslední výhra</span><span id="lastWin">0</span></div>
      <div style="height:1px; background:rgba(255,255,255,0.03); margin:10px 0"></div>
      <div><strong>Výplaty</strong></div>
      <ul style="margin:8px 0 0 16px; color:var(--muted); line-height:1.6">
        <li>3 stejné = 10× sázka</li>
        <li>2 stejné = 2× sázka</li>
        <li>Jackpot (••• 🍋🍒⭐) = 50×</li>
      </ul>
      <div style="height:1px; background:rgba(255,255,255,0.03); margin:10px 0"></div>
      <div style="font-size:13px; color:var(--muted)">Keyboard: <kbd>Space</kbd> spin</div>
    </aside>
  </div>

  <script>
    // Symbols: you can replace with image tags or SVG
    const SYMBOLS = [
      {key:'cherry', emoji:'🍒'},
      {key:'lemon', emoji:'🍋'},
      {key:'seven', emoji:'7️⃣'},
      {key:'star', emoji:'⭐'},
      {key:'bar', emoji:'🔶'},
      {key:'diamond', emoji:'💎'}
    ];

    // DOM
    const reelsEl = document.getElementById('reels');
    const spinBtn = document.getElementById('spinBtn');
    const autoBtn = document.getElementById('autoBtn');
    const betSelect = document.getElementById('bet');
    const bankEl = document.getElementById('bank');
    const betDisplay = document.getElementById('betDisplay');
    const lastWinEl = document.getElementById('lastWin');
    const flash = document.getElementById('flash');

    let bank = 100;
    let auto = false;
    let spinning = false;

    // Build strips
    const reelEls = Array.from(document.querySelectorAll('.reel'));
    function buildReels(){
      reelEls.forEach((reel, idx)=>{
        const strip = reel.querySelector('.strip');
        strip.innerHTML = '';
        // make a repeated list so animation can loop
        const sequence = [];
        for(let r=0;r<6;r++) sequence.push(...SYMBOLS);
        // add a final visible row that will stop in center
        sequence.push(...SYMBOLS);
        sequence.forEach(s=>{
          const sym = document.createElement('div');
          sym.className='symbol';
          sym.innerHTML = `<div style="text-align:center">${s.emoji}<div class=\"label\">${s.key}</div></div>`;
          strip.appendChild(sym);
        });
        // set initial translate
        strip.style.transition='none';
        strip.style.transform='translateY(0px)';
      });
    }

    buildReels();

    function randInt(n){ return Math.floor(Math.random()*n); }

    function getRandomResult(){
      // return array of 3 indices into SYMBOLS
      return [randInt(SYMBOLS.length), randInt(SYMBOLS.length), randInt(SYMBOLS.length)];
    }

    function symbolAt(index){ return SYMBOLS[index]; }

    function calculatePayout(result, bet){
      // result = array of keys
      const a = result[0], b = result[1], c = result[2];
      // all equal
      if(a===b && b===c){
        if(a==='lemon' && b==='lemon' && c==='lemon') return 50 * bet; // rare jackpot for demo
        return 10 * bet;
      }
      // two equal
      if(a===b || a===c || b===c) return 2 * bet;
      return 0;
    }

    function showFlash(text='VÝHRA!'){
      flash.textContent = text;
      flash.classList.add('show');
      setTimeout(()=> flash.classList.remove('show'), 1100);
    }

    async function spin(){
      if(spinning) return;
      const bet = Number(betSelect.value);
      if(bank < bet){ alert('Nedostatek kreditu'); return; }
      spinning = true;
      bank -= bet; updateUI();

      spinBtn.disabled = true;

      // random final symbols
      const finalIdx = getRandomResult();
      // for each reel animate to final position with different durations and easing
      const promises = reelEls.map((reel, i)=>{
        return new Promise(resolve=>{
          const strip = reel.querySelector('.strip');
          // compute index in strip where chosen symbol appears (choose one of repeated symbols for nicer animation)
          const choices = Array.from(strip.children).map((c,idx)=>({idx, key: c.querySelector('.label').textContent}));
          // find locations of desired key
          const desired = SYMBOLS[finalIdx[i]].key;
          const positions = choices.filter(c=>c.key===desired).map(x=>x.idx);
          // pick a position deep in the strip to simulate long roll
          const pos = positions[Math.floor(Math.random()*positions.length)];
          const symbolHeight = reel.clientHeight; // 160 or 110
          const targetY = -pos * symbolHeight + symbolHeight*0; // center row index 0

          // randomize duration and delay
          const duration = 900 + i*250 + Math.floor(Math.random()*400);
          // animate
          requestAnimationFrame(()=>{
            strip.style.transition = `transform ${duration}ms cubic-bezier(.17,.67,.35,1)`;
            strip.style.transform = `translateY(${targetY}px)`;
          });

          setTimeout(()=>{ resolve(); }, duration + 30);
        });
      });

      // wait for all reels
      await Promise.all(promises);

      // compute payout
      const keys = finalIdx.map(i=>SYMBOLS[i].key);
      const payout = calculatePayout(keys, bet);
      bank += payout;
      lastWinEl.textContent = payout;
      updateUI();

      if(payout>0){ showFlash('VÝHRA! ' + payout); }

      spinning = false;
      spinBtn.disabled = false;

      // auto mode continue
      if(auto) setTimeout(()=>{ if(!spinning) spin(); }, 350);
    }

    function updateUI(){
      bankEl.textContent = bank;
      betDisplay.textContent = betSelect.value;
    }

    // events
    spinBtn.addEventListener('click', spin);
    betSelect.addEventListener('change', ()=>{ betDisplay.textContent = betSelect.value; });
    autoBtn.addEventListener('click', ()=>{
      auto = !auto;
      autoBtn.textContent = auto? 'STOP' : 'AUTO';
      if(auto && !spinning) spin();
    });

    // keyboard support
    window.addEventListener('keydown', e=>{ if(e.code==='Space'){ e.preventDefault(); spin(); } });

    // small touch: reset to top if window resizes (recalculate symbol height)
    window.addEventListener('resize', ()=>{ buildReels(); });

    // Accessibility: announce results (simple)
    const live = document.createElement('div'); live.setAttribute('aria-live','polite'); live.style.position='absolute'; live.style.left='-9999px'; document.body.appendChild(live);
    const origSpin = spinBtn.onclick;
    function announceResult(){
      live.textContent = `Stav: bank ${bank}. Poslední výhra ${lastWinEl.textContent}.`;
    }
    // patch to announce after spin ends
    const origSpinFunc = spin;
    // wrap spin with announcement call at end

    // initial UI
    updateUI();
  </script>
</body>
</html>
