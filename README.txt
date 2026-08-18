<!doctype html>
<html lang="pl">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1,viewport-fit=cover">
<title>OG_Ogrody — Zlecenia</title>
<style>
:root{--bg:#101217;--card:#191c23;--card2:#20242d;--text:#f4f5f7;--muted:#9da3ae;--accent:#7cff00;--danger:#ff5b6e;--line:#303541}
*{box-sizing:border-box}body{margin:0;background:var(--bg);color:var(--text);font-family:-apple-system,BlinkMacSystemFont,"Segoe UI",sans-serif}
header{padding:22px 18px 12px;position:sticky;top:0;background:rgba(16,18,23,.96);backdrop-filter:blur(12px);z-index:5}
h1{margin:0;font-size:25px}.sub{color:var(--muted);font-size:13px;margin-top:4px}
nav{display:flex;gap:8px;padding:10px 14px;position:sticky;top:72px;background:var(--bg);z-index:4}
nav button{flex:1;border:0;border-radius:12px;padding:11px 8px;background:var(--card);color:var(--muted);font-weight:700}
nav button.active{background:var(--accent);color:#101217}
main{padding:8px 14px 100px;max-width:700px;margin:auto}
.card{background:var(--card);border:1px solid var(--line);border-radius:18px;padding:16px;margin-bottom:14px}
label{display:block;font-size:12px;color:var(--muted);margin:10px 0 6px}
input,select{width:100%;border:1px solid var(--line);background:var(--card2);color:var(--text);border-radius:11px;padding:13px;font-size:16px;outline:none}
.grid{display:grid;grid-template-columns:1fr 1fr;gap:10px}
.primary{width:100%;border:0;border-radius:13px;background:var(--accent);color:#101217;font-weight:800;padding:14px;margin-top:14px;font-size:16px}
.secondary{border:1px solid var(--line);background:transparent;color:var(--text);border-radius:10px;padding:9px 12px}
.job{padding:14px 0;border-top:1px solid var(--line)}.job:first-child{border-top:0}
.jobtop{display:flex;justify-content:space-between;gap:10px}.name{font-weight:800}.profit{font-weight:900;color:var(--accent)}
.meta{color:var(--muted);font-size:13px;margin-top:5px;line-height:1.5}
.actions{display:flex;gap:7px;margin-top:9px}.delete{border:0;background:transparent;color:var(--danger);padding:6px}
.statgrid{display:grid;grid-template-columns:1fr 1fr;gap:10px}.stat{background:var(--card2);border-radius:14px;padding:14px}.stat b{display:block;font-size:22px;margin-top:5px}.muted{color:var(--muted);font-size:12px}
.empty{text-align:center;color:var(--muted);padding:30px 10px}
.hidden{display:none!important}
</style>
</head>
<body>
<header><h1>🌿 OG_Ogrody</h1><div class="sub">Zlecenia • przychody • koszty • magazyn</div></header>
<nav>
<button class="active" onclick="showTab('jobs',this)">Zlecenia</button>
<button onclick="showTab('stats',this)">Podsumowanie</button>
<button onclick="showTab('stock',this)">Magazyn</button>
</nav>

<main>
<section id="jobs">
<div class="card">
<h2 style="margin:0 0 8px">➕ Nowe zlecenie</h2>
<label>Imię</label><input id="name" placeholder="np. Marta">
<label>Telefon</label><input id="phone" type="tel" placeholder="np. 500 000 000">
<label>Źródło</label><input id="source" placeholder="np. Facebook, Fixly, OFERTEO">
<label>Usługa</label><input id="service" placeholder="np. obrzeża, trawnik, koszenie">
<div class="grid">
<div><label>Przychód (zł)</label><input id="revenue" type="number" inputmode="decimal" min="0" step="0.01" placeholder="0"></div>
<div><label>Koszta (zł)</label><input id="costs" type="number" inputmode="decimal" min="0" step="0.01" placeholder="0"></div>
</div>
<div class="grid">
<div><label>Paliwo (zł)</label><input id="fuel" type="number" inputmode="decimal" min="0" step="0.01" placeholder="0"></div>
<div><label>Magazyn</label><input id="warehouse" placeholder="np. nasiona / zapas"></div>
</div>
<label>Dochód — liczy się automatycznie</label>
<input id="income" readonly value="0 zł">
<button class="primary" onclick="addJob()">ZAPISZ ZLECENIE</button>
</div>

<div class="card"><h2 style="margin:0">📋 Zlecenia</h2><div id="list"></div></div>
</section>

<section id="stats" class="hidden">
<div class="card"><h2 style="margin:0 0 14px">📊 Podsumowanie</h2><div class="statgrid">
<div class="stat"><span class="muted">Przychód</span><b id="sRevenue">0 zł</b></div>
<div class="stat"><span class="muted">Koszta</span><b id="sCosts">0 zł</b></div>
<div class="stat"><span class="muted">Paliwo</span><b id="sFuel">0 zł</b></div>
<div class="stat"><span class="muted">Dochód</span><b id="sIncome">0 zł</b></div>
<div class="stat"><span class="muted">Zlecenia</span><b id="sJobs">0</b></div>
</div></div>
</section>

<section id="stock" class="hidden">
<div class="card">
<h2 style="margin:0 0 8px">📦 Magazyn</h2>
<p class="muted">Na razie magazyn zapisujemy przy konkretnym zleceniu. Jeśli chcesz, możemy później zrobić osobny magazyn z ilością, ceną zakupu i stanem.</p>
</div>
</section>
</main>

<script>
const KEY='og_ogrody_jobs_v1';
let jobs=JSON.parse(localStorage.getItem(KEY)||'[]');
const money=n=>new Intl.NumberFormat('pl-PL',{style:'currency',currency:'PLN'}).format(Number(n)||0);
function val(id){return document.getElementById(id).value}
function calc(){let r=+val('revenue')||0,c=+val('costs')||0,f=+val('fuel')||0;document.getElementById('income').value=money(r-c-f)}
['revenue','costs','fuel'].forEach(id=>document.getElementById(id).addEventListener('input',calc));
function save(){localStorage.setItem(KEY,JSON.stringify(jobs))}
function addJob(){
 let job={id:Date.now(),name:val('name').trim(),phone:val('phone').trim(),source:val('source').trim(),service:val('service').trim(),revenue:+val('revenue')||0,costs:+val('costs')||0,fuel:+val('fuel')||0,warehouse:val('warehouse').trim(),date:new Date().toLocaleDateString('pl-PL')};
 if(!job.name && !job.service){alert('Wpisz przynajmniej imię lub usługę.');return}
 jobs.unshift(job);save();render();document.querySelectorAll('#jobs input').forEach((x,i)=>{if(x.id!=='income')x.value=''});calc();
}
function del(id){if(confirm('Usunąć zlecenie?')){jobs=jobs.filter(j=>j.id!==id);save();render()}}
function render(){
 let el=document.getElementById('list');
 if(!jobs.length){el.innerHTML='<div class="empty">Brak zleceń. Dodaj pierwsze powyżej.</div>'}
 else el.innerHTML=jobs.map(j=>`<div class="job">
 <div class="jobtop"><div><div class="name">${esc(j.name||'Bez imienia')}</div><div class="meta">${esc(j.service||'')} • ${j.date}</div></div><div class="profit">${money(j.revenue-j.costs-j.fuel)}</div></div>
 <div class="meta">${j.phone?'📞 '+esc(j.phone)+' • ':''}${j.source?'Źródło: '+esc(j.source):''}${j.warehouse?' • 📦 '+esc(j.warehouse):''}<br>Przychód ${money(j.revenue)} • Koszta ${money(j.costs)} • Paliwo ${money(j.fuel)}</div>
 <div class="actions"><button class="delete" onclick="del(${j.id})">Usuń</button></div>
 </div>`).join('');
 let r=jobs.reduce((a,j)=>a+j.revenue,0),c=jobs.reduce((a,j)=>a+j.costs,0),f=jobs.reduce((a,j)=>a+j.fuel,0);
 sRevenue.textContent=money(r);sCosts.textContent=money(c);sFuel.textContent=money(f);sIncome.textContent=money(r-c-f);sJobs.textContent=jobs.length;
}
function esc(s){return String(s).replace(/[&<>"']/g,m=>({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#039;'}[m]))}
function showTab(id,btn){document.querySelectorAll('main>section').forEach(s=>s.classList.add('hidden'));document.getElementById(id).classList.remove('hidden');document.querySelectorAll('nav button').forEach(b=>b.classList.remove('active'));btn.classList.add('active');}
render();calc();
</script>
</body>
</html>