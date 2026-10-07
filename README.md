<!DOCTYPE html>
<html lang="kk">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
<title>Робот және Бағдарламашы — алгоритм құру ойыны</title>
<style>
  :root{
    --bg:#fdf1dd; --card:#ffffff; --text:#2b2a28; --muted:#7a756c;
    --purple:#7c5cff; --teal:#0d9488; --red:#e0473e; --good:#16a34a; --bad:#dc2626;
    --border:#f0d9b5; --shadow:0 6px 20px rgba(43,42,40,.08);
    box-sizing:border-box;
    padding-top:env(safe-area-inset-top,0px);
    padding-bottom:env(safe-area-inset-bottom,0px);
  }
  *{box-sizing:border-box;}
  body{margin:0;background:var(--bg);color:var(--text);font-family:Georgia, 'Times New Roman', serif;}
  .wrap{max-width:780px;margin:0 auto;padding:32px 18px 60px;}
  header{text-align:center;margin-bottom:20px;}
  header h1{font-size:clamp(24px,5.5vw,36px);margin:0 0 6px;}
  header p{color:var(--muted);margin:0;font-size:16px;}
  .score{
    text-align:center;background:var(--card);border:2px solid var(--border);border-radius:18px;
    padding:14px;margin-bottom:22px;font-size:18px;box-shadow:var(--shadow);
  }
  .score b{color:var(--purple);font-size:22px;}
  .scene{background:var(--card);border-radius:24px;padding:32px 24px;box-shadow:var(--shadow);margin-bottom:22px;}
  .cond{
    background:#f3ecff;border-radius:18px;padding:20px;text-align:center;margin-bottom:22px;
  }
  .cond .e{font-size:60px;}
  .cond p{font-size:22px;font-weight:700;margin:6px 0 0;}
  .branch-row{display:flex;gap:18px;}
  @media (max-width:480px){.branch-row{flex-direction:column;}}
  .branch{flex:1;border-radius:18px;padding:18px;text-align:center;}
  .branch.yes{background:#e3f8f5;}
  .branch.no{background:#fde7e5;}
  .branch .label{font-weight:700;font-size:18px;margin-bottom:10px;}
  .branch.yes .label{color:var(--teal);}
  .branch.no .label{color:var(--red);}
  .pickbtn{
    width:100%;background:#ffffff;border:3px solid var(--border);border-radius:14px;
    padding:14px 8px;font-family:inherit;font-size:17px;cursor:pointer;margin-bottom:8px;
    display:flex;align-items:center;justify-content:center;gap:8px;transition:.15s;
  }
  .pickbtn .e{font-size:28px;}
  .pickbtn:hover{border-color:var(--purple);}
  .pickbtn.correct{border-color:var(--good);background:#eafbf1;}
  .pickbtn.wrong{border-color:var(--bad);background:#fdecea;}
  .pickbtn:disabled{cursor:default;}
  .feedback{text-align:center;font-size:19px;font-weight:700;min-height:28px;margin-top:16px;}
  .feedback.ok{color:var(--good);}
  .feedback.no{color:var(--bad);}
  .next{
    display:block;margin:18px auto 0;padding:12px 28px;border:none;border-radius:999px;
    background:var(--purple);color:#fff;font-size:17px;font-weight:700;cursor:pointer;
  }
  .next:disabled{opacity:.4;cursor:default;}
  .done{text-align:center;padding:40px 20px;}
  .done .big{font-size:80px;}
  .restart{
    margin-top:18px;padding:12px 26px;border:none;border-radius:999px;background:var(--teal);
    color:#fff;font-size:16px;font-weight:700;cursor:pointer;
  }
</style>
</head>
<body>
<div class="wrap">
  <header>
    <h1>🧑‍🤝‍🧑🤖 Робот және Бағдарламашы</h1>
    <p>Шартты оқы: «Иә» және «Жоқ» жолына дұрыс әрекетті тап</p>
  </header>
  <div class="score">Ұпай: <b id="score">0</b> / <span id="total">5</span></div>
  <div id="game"></div>
</div>

<script>
const rounds = [
  { e:"🍽️", cond:"Егер аш болсаң?",
    yes:[{e:"🍲",t:"Тамақтан",ok:true},{e:"😴",t:"Ұйықта",ok:false}],
    no:[{e:"🎮",t:"Ойна",ok:true},{e:"🍲",t:"Тамақтан",ok:false}] },
  { e:"😴", cond:"Егер ұйқың келсе?",
    yes:[{e:"🛏️",t:"Жат та ұйықта",ok:true},{e:"🏃",t:"Жүгір",ok:false}],
    no:[{e:"📚",t:"Сабақ оқы",ok:true},{e:"🛏️",t:"Жат",ok:false}] },
  { e:"🏠", cond:"Егер үй жұмысы бітсе?",
    yes:[{e:"🎮",t:"Ойнауға болады",ok:true},{e:"✏️",t:"Жаза бер",ok:false}],
    no:[{e:"✏️",t:"Жалғастыр",ok:true},{e:"🎮",t:"Ойна",ok:false}] },
  { e:"🚦", cond:"Егер бағдаршам қызыл болса?",
    yes:[{e:"🛑",t:"Тоқта",ok:true},{e:"🚶",t:"Жүгір",ok:false}],
    no:[{e:"🚶",t:"Абайлап өт",ok:true},{e:"🛑",t:"Тоқтай бер",ok:false}] },
  { e:"🌡️", cond:"Егер денең қызса (температура)?",
    yes:[{e:"👨‍⚕️",t:"Үлкенге айт",ok:true},{e:"🎮",t:"Ойна",ok:false}],
    no:[{e:"🏫",t:"Сабаққа бара бер",ok:true},{e:"👨‍⚕️",t:"Дәрігерге жүгір",ok:false}] },
];

let idx = 0, score = 0, correctCount = 0;
document.getElementById('total').textContent = rounds.length * 2;

function renderBranch(container, opts, roundScore){
  container.innerHTML = opts.map((o,i)=>`<button class="pickbtn" data-i="${i}"><span class="e">${o.e}</span>${o.t}</button>`).join('');
  const btns = container.querySelectorAll('.pickbtn');
  btns.forEach(btn=>{
    btn.onclick = ()=>{
      btns.forEach(b=>b.disabled = true);
      const chosen = opts[Number(btn.dataset.i)];
      if(chosen.ok){
        btn.classList.add('correct');
        score++; document.getElementById('score').textContent = score;
      } else {
        btn.classList.add('wrong');
        btns.forEach(b=>{ if(opts[Number(b.dataset.i)].ok) b.classList.add('correct'); });
      }
      roundScore.done++;
      checkRoundDone(roundScore);
    };
  });
}

function checkRoundDone(roundScore){
  if(roundScore.done >= 2){
    document.getElementById('nextBtn').disabled = false;
  }
}

function render(){
  const g = document.getElementById('game');
  if(idx >= rounds.length){
    g.innerHTML = `
      <div class="done scene">
        <div class="big">🏆</div>
        <h2>Ойын бітті!</h2>
        <p>Нәтиже: ${score} / ${rounds.length*2}</p>
        <button class="restart" id="restart">Қайта ойнау</button>
      </div>`;
    document.getElementById('restart').onclick = ()=>{ idx=0; score=0; document.getElementById('score').textContent=score; render(); };
    return;
  }
  const r = rounds[idx];
  g.innerHTML = `
    <div class="scene">
      <div class="cond"><div class="e">${r.e}</div><p>${r.cond}</p></div>
      <div class="branch-row">
        <div class="branch yes"><div class="label">Иә ✓ → онда...</div><div id="yesOpts"></div></div>
        <div class="branch no"><div class="label">Жоқ ✗ → әйтпесе...</div><div id="noOpts"></div></div>
      </div>
      <button class="next" id="nextBtn" disabled>Келесі →</button>
    </div>`;
  const roundScore = { done: 0 };
  renderBranch(document.getElementById('yesOpts'), r.yes, roundScore);
  renderBranch(document.getElementById('noOpts'), r.no, roundScore);
  document.getElementById('nextBtn').onclick = ()=>{ idx++; render(); };
}
render();
</script>
</body>
</html>
