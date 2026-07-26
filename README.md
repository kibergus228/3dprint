<html lang="uk">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>ХРОМ.ДРУК // калькулятор вартості 3D-друку</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link href="https://fonts.googleapis.com/css2?family=Orbitron:wght@500;700;900&family=Share+Tech+Mono&display=swap" rel="stylesheet">
<style>
  :root{
    --bg:#05070b;
    --cyan:#00fff2;
    --magenta:#ff2ea6;
    --green:#39ff88;
    --text:#d7e2ea;
    --muted:#6a7a89;
    --border: rgba(0,255,242,0.22);
    --panel-w: 150px;
  }

  *{box-sizing:border-box;}
  html,body{margin:0;padding:0;}
  body{
    background:var(--bg);
    color:var(--text);
    font-family:'Share Tech Mono', monospace;
    min-height:100vh;
    overflow-x:hidden;
    position:relative;
  }

  /* ---------- side circuit panels ---------- */
  .side-panel{
    position:fixed;
    top:0; bottom:0;
    width:var(--panel-w);
    z-index:0;
    background:#000;
    border-color:var(--border);
  }
  .side-left{ left:0; border-right:1px solid var(--border); }
  .side-right{ right:0; border-left:1px solid var(--border); transform:scaleX(-1); }
  .side-panel svg{ width:100%; height:100%; display:block; }

  .led{ animation:blink 3.2s ease-in-out infinite; }
  .d2{ animation-delay:.6s; }
  .d3{ animation-delay:1.3s; }
  .d4{ animation-delay:2s; }
  @keyframes blink{ 0%,100%{opacity:1} 50%{opacity:.2} }

  /* ---------- scanline overlay ---------- */
  .scanlines{
    position:fixed; inset:0; pointer-events:none; z-index:60;
    background:repeating-linear-gradient(to bottom, rgba(255,255,255,.025) 0px, rgba(255,255,255,.025) 1px, transparent 2px, transparent 4px);
    mix-blend-mode:overlay;
  }

  /* ---------- layout ---------- */
  .page{
    position:relative; z-index:1;
    margin:0 var(--panel-w);
  }

  header{
    display:flex; align-items:center; justify-content:space-between;
    padding:22px 40px;
    border-bottom:1px solid var(--border);
    position:sticky; top:0; background:rgba(5,7,11,.85); backdrop-filter:blur(6px);
    z-index:10;
  }
  .logo{
    font-family:'Orbitron',sans-serif; font-weight:900; letter-spacing:2px;
    color:var(--cyan); font-size:1.15rem; text-shadow:0 0 8px rgba(0,255,242,.6);
  }
  .logo span{ color:var(--magenta); text-shadow:0 0 8px rgba(255,46,166,.6); }
  nav{ display:flex; gap:28px; }
  nav a{
    color:var(--muted); text-decoration:none; font-size:.8rem; letter-spacing:1px;
    text-transform:uppercase; transition:color .2s, text-shadow .2s;
  }
  nav a:hover{ color:var(--cyan); text-shadow:0 0 6px rgba(0,255,242,.7); }

  /* ---------- hero ---------- */
  .hero{
    padding:110px 40px 80px; text-align:center;
    display:flex; flex-direction:column; align-items:center; gap:22px;
  }
  .eyebrow{
    font-size:.75rem; letter-spacing:4px; color:var(--green);
    text-shadow:0 0 6px rgba(57,255,136,.6);
  }
  .glitch{
    font-family:'Orbitron',sans-serif; font-weight:900;
    font-size:clamp(2.2rem, 6.5vw, 4.6rem);
    line-height:1.05; margin:0; position:relative; color:var(--text);
    letter-spacing:2px;
  }
  .glitch::before, .glitch::after{
    content:attr(data-text); position:absolute; left:0; top:0; width:100%;
    overflow:hidden; opacity:.7;
  }
  .glitch::before{ color:var(--cyan); animation:glitchTop 4.5s infinite linear; clip-path:inset(0 0 55% 0); }
  .glitch::after{ color:var(--magenta); animation:glitchBottom 5.5s infinite linear; clip-path:inset(55% 0 0 0); }
  @keyframes glitchTop{
    0%,88%,100%{ transform:translate(0,0); }
    90%{ transform:translate(-3px,-1px); }
    92%{ transform:translate(3px,1px); }
    94%{ transform:translate(-2px,0); }
  }
  @keyframes glitchBottom{
    0%,90%,100%{ transform:translate(0,0); }
    92%{ transform:translate(3px,1px); }
    94%{ transform:translate(-3px,-1px); }
    96%{ transform:translate(2px,0); }
  }
  .hero p{ max-width:560px; color:var(--muted); font-size:.95rem; line-height:1.6; }

  /* ---------- calculator ---------- */
  .calc-section{ padding:20px 40px 90px; max-width:760px; margin:0 auto; }
  .panel{
    border:1px solid var(--border); background:rgba(0,255,242,.03);
    padding:34px 32px; position:relative;
  }
  .panel::before{
    content:"МОДУЛЬ РОЗРАХУНКУ"; position:absolute; top:-11px; left:24px;
    background:var(--bg); padding:0 10px; font-size:.68rem; letter-spacing:3px; color:var(--cyan);
  }
  .field{ margin-bottom:22px; }
  .field label{
    display:block; font-size:.72rem; letter-spacing:1.5px; text-transform:uppercase;
    color:var(--muted); margin-bottom:8px;
  }
  .field select, .field input{
    width:100%; background:#000; border:1px solid var(--border); color:var(--cyan);
    font-family:'Share Tech Mono',monospace; font-size:.95rem; padding:12px 14px;
    outline:none; transition:border-color .2s, box-shadow .2s;
  }
  .field select:focus, .field input:focus{
    border-color:var(--cyan); box-shadow:0 0 10px rgba(0,255,242,.3);
  }
  .row2{ display:grid; grid-template-columns:1fr 1fr; gap:20px; }
  .hint{ font-size:.7rem; color:var(--muted); margin-top:6px; }

  .calc-btn{
    width:100%; margin-top:6px; padding:15px; background:transparent;
    border:1px solid var(--magenta); color:var(--magenta); font-family:'Share Tech Mono',monospace;
    letter-spacing:2px; text-transform:uppercase; font-size:.85rem; cursor:pointer;
    box-shadow:0 0 12px rgba(255,46,166,.2) inset; transition:all .25s;
  }
  .calc-btn:hover{ background:var(--magenta); color:#000; box-shadow:0 0 24px rgba(255,46,166,.6); }

  /* ---------- electricity toggle ---------- */
  .toggle-field{ margin-bottom:8px; }
  .toggle-label{
    display:flex; align-items:center; gap:12px; cursor:pointer;
    font-size:.82rem; letter-spacing:.5px; color:var(--text); user-select:none;
  }
  .toggle-label input{ display:none; }
  .toggle-switch{
    position:relative; width:42px; height:22px; flex-shrink:0;
    border:1px solid var(--border); background:#000; transition:box-shadow .2s;
  }
  .toggle-switch::after{
    content:""; position:absolute; top:2px; left:2px; width:16px; height:16px;
    background:var(--muted); transition:transform .25s, background .25s, box-shadow .25s;
  }
  .toggle-label input:checked + .toggle-switch{ border-color:var(--green); box-shadow:0 0 10px rgba(57,255,136,.4); }
  .toggle-label input:checked + .toggle-switch::after{
    transform:translateX(20px); background:var(--green); box-shadow:0 0 8px rgba(57,255,136,.8);
  }
  .elec-fields{
    max-height:0; overflow:hidden; opacity:0;
    transition:max-height .35s ease, opacity .25s ease, margin .25s ease;
    margin-bottom:0;
  }
  .elec-fields.open{ max-height:260px; opacity:1; margin-bottom:22px; }
  .elec-line{ display:none; }
  .elec-line.show{ display:flex; }

  .result{
    margin-top:28px; border-top:1px dashed var(--border); padding-top:24px;
    display:none;
  }
  .result.show{ display:block; }
  .result .line{ display:flex; justify-content:space-between; font-size:.85rem; color:var(--muted); margin-bottom:8px; }
  .result .total{
    display:flex; justify-content:space-between; align-items:baseline; margin-top:14px;
    font-family:'Orbitron',sans-serif; font-size:1.5rem; color:var(--green);
    text-shadow:0 0 10px rgba(57,255,136,.5);
  }
  .result .total span:last-child{ font-size:1.7rem; }

  /* ---------- materials info ---------- */
  .materials{ padding:20px 40px 100px; }
  .section-title{
    font-family:'Orbitron',sans-serif; font-weight:700; letter-spacing:2px;
    text-align:center; color:var(--text); font-size:1.3rem; margin:0 0 8px;
  }
  .section-sub{ text-align:center; color:var(--muted); font-size:.82rem; margin-bottom:44px; }
  .grid{
    display:grid; grid-template-columns:repeat(auto-fit, minmax(210px,1fr));
    gap:22px; max-width:1000px; margin:0 auto;
  }
  .card{
    border:1px solid var(--border); padding:22px 20px; background:linear-gradient(180deg, rgba(0,255,242,.04), transparent);
    transition:transform .25s, box-shadow .25s;
  }
  .card:hover{ transform:translateY(-4px); box-shadow:0 0 22px rgba(0,255,242,.18); border-color:var(--cyan); }
  .card .tag{ font-size:.65rem; letter-spacing:2px; color:var(--magenta); }
  .card h3{ font-family:'Orbitron',sans-serif; font-size:.95rem; color:var(--text); margin:10px 0 10px; letter-spacing:1px; }
  .card p{ color:var(--muted); font-size:.8rem; line-height:1.6; }

  /* ---------- footer ---------- */
  footer{
    border-top:1px solid var(--border); padding:20px 40px;
    display:flex; flex-wrap:wrap; gap:20px; justify-content:space-between;
    font-size:.72rem; color:var(--muted); letter-spacing:1px;
  }
  .dot{ display:inline-block; width:7px; height:7px; border-radius:50%; background:var(--green);
    box-shadow:0 0 8px var(--green); margin-right:6px; animation:blink 2s ease-in-out infinite; }

  @media (max-width:1100px){ :root{ --panel-w:80px; } }
  @media (max-width:700px){
    :root{ --panel-w:0px; }
    .side-panel{ display:none; }
    header, .hero, .calc-section, .materials, footer{ padding-left:22px; padding-right:22px; }
    .row2{ grid-template-columns:1fr; }
  }
</style>
</head>
<body>

  <div class="side-panel side-left">
    <svg xmlns="http://www.w3.org/2000/svg">
      <defs>
        <filter id="glowL" x="-50%" y="-50%" width="200%" height="200%">
          <feGaussianBlur stdDeviation="2.1" result="b"/>
          <feMerge><feMergeNode in="b"/><feMergeNode in="SourceGraphic"/></feMerge>
        </filter>
        <pattern id="circuitL" width="150" height="280" patternUnits="userSpaceOnUse">
          <rect width="150" height="280" fill="#000"/>
          <g stroke="#00fff2" stroke-width="1.4" fill="none" opacity="0.5" filter="url(#glowL)">
            <path d="M22,0 L22,58 L75,58 L75,150"/>
            <path d="M118,0 L118,42 L95,42 L95,118 L128,118 L128,280"/>
            <path d="M42,150 L42,205 L14,205 L14,280"/>
            <path d="M0,95 L34,95"/>
            <path d="M105,190 L150,190"/>
            <path d="M60,230 L60,280"/>
          </g>
          <g fill="#ff2ea6" filter="url(#glowL)">
            <circle class="led" cx="75" cy="58" r="4"/>
            <circle class="led d3" cx="42" cy="150" r="3.6"/>
          </g>
          <g fill="#39ff88" filter="url(#glowL)">
            <circle class="led d2" cx="95" cy="118" r="3.4"/>
            <circle class="led d4" cx="14" cy="205" r="3"/>
            <circle class="led d2" cx="60" cy="230" r="3"/>
          </g>
        </pattern>
      </defs>
      <rect width="100%" height="100%" fill="url(#circuitL)"/>
    </svg>
  </div>

  <div class="side-panel side-right">
    <svg xmlns="http://www.w3.org/2000/svg">
      <defs>
        <filter id="glowR" x="-50%" y="-50%" width="200%" height="200%">
          <feGaussianBlur stdDeviation="2.1" result="b"/>
          <feMerge><feMergeNode in="b"/><feMergeNode in="SourceGraphic"/></feMerge>
        </filter>
        <pattern id="circuitR" width="150" height="280" patternUnits="userSpaceOnUse">
          <rect width="150" height="280" fill="#000"/>
          <g stroke="#00fff2" stroke-width="1.4" fill="none" opacity="0.5" filter="url(#glowR)">
            <path d="M22,0 L22,58 L75,58 L75,150"/>
            <path d="M118,0 L118,42 L95,42 L95,118 L128,118 L128,280"/>
            <path d="M42,150 L42,205 L14,205 L14,280"/>
            <path d="M0,95 L34,95"/>
            <path d="M105,190 L150,190"/>
            <path d="M60,230 L60,280"/>
          </g>
          <g fill="#ff2ea6" filter="url(#glowR)">
            <circle class="led" cx="75" cy="58" r="4"/>
            <circle class="led d3" cx="42" cy="150" r="3.6"/>
          </g>
          <g fill="#39ff88" filter="url(#glowR)">
            <circle class="led d2" cx="95" cy="118" r="3.4"/>
            <circle class="led d4" cx="14" cy="205" r="3"/>
            <circle class="led d2" cx="60" cy="230" r="3"/>
          </g>
        </pattern>
      </defs>
      <rect width="100%" height="100%" fill="url(#circuitR)"/>
    </svg>
  </div>

  <div class="scanlines"></div>

  <div class="page">

    <header>
      <div class="logo">ХРОМ<span>.ДРУК</span></div>
      <nav>
        <a href="#calc">Розрахунок</a>
        <a href="#materials">Матеріали</a>
      </nav>
    </header>

    <main>
      <section class="hero">
        <div class="eyebrow">ВУЗОЛ 7 // МОДУЛЬ АДИТИВНОГО ВИРОБНИЦТВА</div>
        <h1 class="glitch" data-text="ХРОМ.ДРУК">ХРОМ.ДРУК</h1>
        <p>Обери тип пластику, вкажи вагу виробу та ціну за кілограм — модуль порахує точну вартість друку за лічені секунди.</p>
      </section>

      <section class="calc-section" id="calc">
        <div class="panel">
          <div class="field">
            <label for="material">Тип пластика</label>
            <select id="material">
              <option value="700" data-gpm="2.98">PLA — базовий, простий у друці</option>
              <option value="750" data-gpm="2.50">ABS — міцний, термостійкий</option>
              <option value="850" data-gpm="3.06">PETG — стійкий до вологи й ударів</option>
              <option value="1200" data-gpm="2.91">TPU — гнучкий, еластичний</option>
              <option value="1600" data-gpm="2.74">Nylon — інженерний, зносостійкий</option>
              <option value="1500" data-gpm="2.65">Resin — фотополімер, висока деталізація</option>
            </select>
            <div class="hint">розрахунок ваги — для нитки Ø1.75 мм</div>
          </div>

          <div class="row2">
            <div class="field">
              <label for="weight">Довжина нитки (метри)</label>
              <input type="number" id="weight" placeholder="напр. 15" min="0" step="0.01">
            </div>
            <div class="field">
              <label for="price">Ціна за 1 кг (₴)</label>
              <input type="number" id="price" min="0" step="1">
              <div class="hint">підставляється автоматично, можна змінити вручну</div>
            </div>
          </div>

          <div class="field toggle-field">
            <label class="toggle-label">
              <input type="checkbox" id="elecToggle">
              <span class="toggle-switch"></span>
              <span>Врахувати вартість електроенергії</span>
            </label>
          </div>

          <div class="elec-fields" id="elecFields">
            <div class="row2">
              <div class="field">
                <label for="power">Потужність принтера (Вт)</label>
                <input type="number" id="power" value="150" min="0" step="1">
              </div>
              <div class="field">
                <label for="hours">Час друку (годин)</label>
                <input type="number" id="hours" placeholder="напр. 4.5" min="0" step="0.1">
              </div>
            </div>
            <div class="field">
              <label for="tariff">Тариф за 1 кВт·год (₴)</label>
              <input type="number" id="tariff" value="4.32" min="0" step="0.01">
            </div>
          </div>

          <button class="calc-btn" id="calcBtn">Розрахувати вартість</button>

          <div class="result" id="result">
            <div class="line"><span>Матеріал</span><span id="resMaterial">—</span></div>
            <div class="line"><span>Довжина нитки</span><span id="resWeight">—</span></div>
            <div class="line"><span>Розрахункова вага</span><span id="resGrams">—</span></div>
            <div class="line"><span>Ціна за кг</span><span id="resPrice">—</span></div>
            <div class="line elec-line" id="elecLine"><span>Електроенергія</span><span id="resElec">—</span></div>
            <div class="total"><span>Вартість виробу</span><span id="resTotal">0 ₴</span></div>
          </div>
        </div>
      </section>

      <section class="materials" id="materials">
        <h2 class="section-title">ДОВІДКА ПО МАТЕРІАЛАХ</h2>
        <p class="section-sub">орієнтовні властивості для вибору пластика</p>
        <div class="grid">
          <div class="card">
            <div class="tag">PLA</div>
            <h3>Базовий</h3>
            <p>Легко друкується, мінімум деформацій. Підходить для прототипів і декору.</p>
          </div>
          <div class="card">
            <div class="tag">ABS</div>
            <h3>Міцний</h3>
            <p>Витримує нагрів і механічні навантаження. Потребує закритої камери друку.</p>
          </div>
          <div class="card">
            <div class="tag">PETG</div>
            <h3>Стійкий</h3>
            <p>Гарний баланс міцності й простоти друку. Не боїться вологи.</p>
          </div>
          <div class="card">
            <div class="tag">TPU</div>
            <h3>Гнучкий</h3>
            <p>Еластичний матеріал для чохлів, прокладок і амортизуючих деталей.</p>
          </div>
          <div class="card">
            <div class="tag">NYLON</div>
            <h3>Інженерний</h3>
            <p>Висока зносостійкість, підходить для функціональних механізмів.</p>
          </div>
          <div class="card">
            <div class="tag">RESIN</div>
            <h3>Деталізований</h3>
            <p>Фотополімер для мініатюр і ювелірних форм із високою точністю.</p>
          </div>
        </div>
      </section>
    </main>

    <footer>
      <div><span class="dot"></span>МОДУЛЬ: АКТИВНИЙ</div>
      <div>РОЗРАХУНКІВ ВИКОНАНО: 2 847</div>
      <div>ТОЧНІСТЬ: 100%</div>
      <div>© ХРОМ.ДРУК</div>
    </footer>

  </div>

<script>
  const materialSelect = document.getElementById('material');
  const priceInput = document.getElementById('price');
  const weightInput = document.getElementById('weight');
  const calcBtn = document.getElementById('calcBtn');
  const result = document.getElementById('result');

  function syncPrice(){
    priceInput.value = materialSelect.value;
  }
  materialSelect.addEventListener('change', syncPrice);
  syncPrice();

  const elecToggle = document.getElementById('elecToggle');
  const elecFields = document.getElementById('elecFields');
  const elecLine = document.getElementById('elecLine');
  const powerInput = document.getElementById('power');
  const hoursInput = document.getElementById('hours');
  const tariffInput = document.getElementById('tariff');

  elecToggle.addEventListener('change', () => {
    elecFields.classList.toggle('open', elecToggle.checked);
    elecLine.classList.toggle('show', elecToggle.checked);
  });

  calcBtn.addEventListener('click', () => {
    const length = parseFloat(weightInput.value);
    const pricePerKg = parseFloat(priceInput.value);
    const selectedOption = materialSelect.options[materialSelect.selectedIndex];
    const materialName = selectedOption.text.split(' — ')[0];
    const gramsPerMeter = parseFloat(selectedOption.dataset.gpm);

    if(!length || length <= 0 || !pricePerKg || pricePerKg <= 0){
      weightInput.style.borderColor = '#ff2ea6';
      return;
    }
    weightInput.style.borderColor = '';

    const grams = length * gramsPerMeter;
    let total = (grams / 1000) * pricePerKg;
    let elecCost = 0;

    if(elecToggle.checked){
      const power = parseFloat(powerInput.value) || 0;
      const hours = parseFloat(hoursInput.value) || 0;
      const tariff = parseFloat(tariffInput.value) || 0;
      elecCost = (power / 1000) * hours * tariff;
      total += elecCost;
      document.getElementById('resElec').textContent = elecCost.toFixed(2) + ' ₴';
    }

    document.getElementById('resMaterial').textContent = materialName;
    document.getElementById('resWeight').textContent = length + ' м';
    document.getElementById('resGrams').textContent = '≈ ' + grams.toFixed(1) + ' г';
    document.getElementById('resPrice').textContent = pricePerKg + ' ₴/кг';
    document.getElementById('resTotal').textContent = total.toFixed(2) + ' ₴';
    result.classList.add('show');
  });
</script>

</body>
</html>
