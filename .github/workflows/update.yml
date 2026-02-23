"""
update_exhibitions.py  —  runs monthly via GitHub Actions
Calls the Claude API to research current Spanish art exhibitions,
then regenerates index.html with fresh data.
"""

import anthropic
import json
import re
from datetime import datetime
from pathlib import Path

MODEL      = "claude-sonnet-4-6"
TODAY      = datetime.now().strftime("%B %d, %Y")
MONTH_YEAR = datetime.now().strftime("%B %Y")

RESEARCH_PROMPT = f"""
Today is {TODAY}.

Compile a comprehensive list of current and upcoming art exhibitions in Spain —
covering museums, cultural centres, commercial galleries AND art fairs.

Include only exhibitions that are currently open OR opening in the next 60 days.

For each exhibition return a JSON object with:
  title        (string) — exhibition name
  artist       (string) — main artist(s) or "Varios artistas"
  type         (string) — one of: Pintura | Fotografía | Arte Contemporáneo |
                          Escultura | Arte Conceptual | Multidisciplinar |
                          Retrospectiva | Arte Digital | Dibujo
  vtype        (string) — one of: Museo | Galería | Centro Cultural | Feria | Espacio
  city         (string) — Spanish city name
  venue        (string) — venue name
  address      (string) — street address
  open         (string) — opening date YYYY-MM-DD
  close        (string) — closing date YYYY-MM-DD
  description  (string) — 1-2 sentence description in Spanish

Return ONLY a valid JSON array, no markdown fences, no comments.
Include at least 35 exhibitions across multiple Spanish cities.
"""

HTML_TEMPLATE = """<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Exposiciones en España — MONTH_YEAR</title>
<link href="https://fonts.googleapis.com/css2?family=Playfair+Display:ital,wght@0,400;0,700;0,900;1,400&family=DM+Sans:wght@300;400;500&display=swap" rel="stylesheet">
<style>
:root{--bg:#0d0b09;--card:#1e1a14;--border:#2e2820;--gold:#c9a84c;--gold-light:#e8c97a;--cream:#f0e8d8;--muted:#8a7f70;--red:#c94c4c;--teal:#4c9c8a;--blue:#4c7ac9;--violet:#9c4cc9;--orange:#c97a4c;--rose:#c94c7a;--green:#6a9c4c}
*{box-sizing:border-box;margin:0;padding:0}
body{background:var(--bg);color:var(--cream);font-family:'DM Sans',sans-serif;font-weight:300;min-height:100vh;overflow-x:hidden}
body::before{content:'';position:fixed;inset:0;opacity:.04;pointer-events:none;z-index:0;background-image:url("data:image/svg+xml,%3Csvg viewBox='0 0 200 200' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.9' numOctaves='4'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23n)'/%3E%3C/svg%3E")}
header{position:relative;padding:80px 60px 60px;border-bottom:1px solid var(--border);overflow:hidden}
header::after{content:'ARTE';position:absolute;right:-20px;top:-30px;font-family:'Playfair Display',serif;font-size:280px;font-weight:900;color:rgba(201,168,76,.04);letter-spacing:-20px;pointer-events:none;user-select:none}
.header-label{font-size:11px;letter-spacing:4px;text-transform:uppercase;color:var(--gold);margin-bottom:16px}
h1{font-family:'Playfair Display',serif;font-size:clamp(40px,6vw,80px);font-weight:900;line-height:1;color:var(--cream);max-width:700px}
h1 em{font-style:italic;color:var(--gold)}
.header-sub{margin-top:20px;font-size:15px;color:var(--muted);max-width:520px;line-height:1.7}
.updated{margin-top:10px;font-size:11px;color:var(--muted);letter-spacing:1px}
.header-stats{display:flex;gap:40px;margin-top:32px;flex-wrap:wrap}
.stat{display:flex;flex-direction:column;gap:4px}
.stat-num{font-family:'Playfair Display',serif;font-size:36px;font-weight:700;color:var(--gold-light);line-height:1}
.stat-label{font-size:11px;letter-spacing:2px;text-transform:uppercase;color:var(--muted)}
.filters-section{position:sticky;top:0;z-index:100;background:rgba(13,11,9,.97);backdrop-filter:blur(14px);border-bottom:1px solid var(--border);padding:14px 60px;display:flex;flex-direction:column;gap:8px}
.filter-row{display:flex;align-items:center;gap:10px;flex-wrap:wrap}
.filter-label{font-size:10px;letter-spacing:3px;text-transform:uppercase;color:var(--muted);white-space:nowrap;min-width:60px}
.filter-group{display:flex;gap:5px;flex-wrap:wrap}
.filter-btn{padding:5px 11px;border:1px solid var(--border);background:transparent;color:var(--muted);font-family:'DM Sans',sans-serif;font-size:11px;cursor:pointer;transition:all .18s;border-radius:2px;white-space:nowrap}
.filter-btn:hover{border-color:var(--gold);color:var(--gold)}
.filter-btn.active{background:var(--gold);border-color:var(--gold);color:#0d0b09;font-weight:500}
.vtype-btn[data-value="Galería"].active{background:var(--rose);border-color:var(--rose);color:#fff}
.vtype-btn[data-value="Museo"].active{background:var(--blue);border-color:var(--blue);color:#fff}
.vtype-btn[data-value="Centro Cultural"].active{background:var(--teal);border-color:var(--teal);color:#fff}
.vtype-btn[data-value="Feria"].active{background:var(--violet);border-color:var(--violet);color:#fff}
.vtype-btn[data-value="Espacio"].active{background:var(--orange);border-color:var(--orange);color:#fff}
.ml-auto{margin-left:auto}
.search-wrap{position:relative}
.search-wrap input{background:var(--card);border:1px solid var(--border);color:var(--cream);padding:7px 14px 7px 32px;font-family:'DM Sans',sans-serif;font-size:12px;width:210px;border-radius:2px;outline:none;transition:border-color .2s}
.search-wrap input:focus{border-color:var(--gold)}
.search-wrap input::placeholder{color:var(--muted)}
.search-wrap svg{position:absolute;left:9px;top:50%;transform:translateY(-50%);color:var(--muted);width:13px;height:13px}
.grid-section{padding:36px 60px 80px;position:relative;z-index:1}
.results-bar{font-size:11px;color:var(--muted);letter-spacing:1px;margin-bottom:28px;text-transform:uppercase;display:flex;align-items:center;gap:24px;flex-wrap:wrap}
.results-bar .count{color:var(--gold);font-weight:500;font-size:13px}
.legend{display:flex;gap:14px;flex-wrap:wrap}
.legend-item{display:flex;align-items:center;gap:5px;font-size:10px;letter-spacing:1px}
.legend-dot{width:7px;height:7px;border-radius:50%}
.grid{display:grid;grid-template-columns:repeat(auto-fill,minmax(330px,1fr));gap:2px}
.card{background:var(--card);border:1px solid var(--border);padding:26px 28px 22px;transition:transform .25s,box-shadow .25s;position:relative;overflow:hidden;animation:fadeUp .45s ease both}
.card[data-vtype="Galería"]{background:#201419;border-color:#352028}
.card[data-vtype="Feria"]{background:#19151f;border-color:#2a2035}
@keyframes fadeUp{from{opacity:0;transform:translateY(16px)}to{opacity:1;transform:translateY(0)}}
.card::before{content:'';position:absolute;top:0;left:0;right:0;height:2px}
.card:hover{transform:translateY(-3px);box-shadow:0 10px 40px rgba(0,0,0,.5)}
.card[data-type="Pintura"]::before{background:var(--gold)}
.card[data-type="Fotografía"]::before{background:var(--teal)}
.card[data-type="Arte Contemporáneo"]::before{background:var(--violet)}
.card[data-type="Arte Digital"]::before{background:var(--blue)}
.card[data-type="Escultura"]::before{background:var(--orange)}
.card[data-type="Arte Conceptual"]::before{background:var(--red)}
.card[data-type="Multidisciplinar"]::before{background:var(--muted)}
.card[data-type="Retrospectiva"]::before{background:var(--gold-light)}
.card[data-type="Dibujo"]::before{background:var(--green)}
.vtype-badge{position:absolute;top:13px;right:13px;font-size:9px;letter-spacing:1.5px;text-transform:uppercase;padding:3px 7px;border-radius:2px}
.card-type{font-size:9px;letter-spacing:2.5px;text-transform:uppercase;margin-bottom:10px;display:inline-flex;align-items:center;gap:5px;padding-right:80px}
.card-type-dot{width:5px;height:5px;border-radius:50%;flex-shrink:0}
.card-title{font-family:'Playfair Display',serif;font-size:18px;font-weight:700;line-height:1.25;color:var(--cream);margin-bottom:5px}
.card-artist{font-size:12px;color:var(--gold-light);font-style:italic;margin-bottom:13px}
.card-description{font-size:12px;color:var(--muted);line-height:1.65;margin-bottom:18px}
.card-meta{display:flex;flex-direction:column;gap:6px;padding-top:16px;border-top:1px solid var(--border)}
.meta-row{display:flex;align-items:flex-start;gap:8px;font-size:11.5px}
.meta-icon{color:var(--gold);width:12px;flex-shrink:0;margin-top:1px}
.meta-val{color:var(--cream);line-height:1.4}
.meta-sub{color:var(--muted);font-size:10.5px}
.dates-badge{display:inline-block;margin-top:12px;padding:3px 9px;font-size:10px;letter-spacing:1px;text-transform:uppercase}
.dates-badge.open{background:rgba(201,168,76,.08);border:1px solid rgba(201,168,76,.2);color:var(--gold)}
.dates-badge.ending-soon{background:rgba(201,76,76,.1);border:1px solid rgba(201,76,76,.3);color:var(--red)}
.dates-badge.upcoming{background:rgba(76,156,138,.1);border:1px solid rgba(76,156,138,.3);color:var(--teal)}
.no-results{grid-column:1/-1;text-align:center;padding:80px 20px;color:var(--muted)}
.no-results h3{font-family:'Playfair Display',serif;font-size:28px;margin-bottom:12px;color:var(--cream)}
footer{border-top:1px solid var(--border);padding:26px 60px;font-size:12px;color:var(--muted);display:flex;justify-content:space-between;align-items:center;flex-wrap:wrap;gap:10px;position:relative;z-index:1}
.footer-brand{font-family:'Playfair Display',serif;color:var(--gold);font-size:16px}
@media(max-width:768px){header,.grid-section,footer{padding-left:20px;padding-right:20px}.filters-section{padding:12px 20px}.search-wrap input,.ml-auto{width:100%}header::after{font-size:100px}}
</style>
</head>
<body>
<header>
  <div class="header-label">Agenda Cultural · España MONTH_YEAR</div>
  <h1>Exposiciones<br><em>de Arte</em></h1>
  <p class="header-sub">Museos, centros culturales, galerías privadas y ferias de arte en España.</p>
  <p class="updated">Actualizado automáticamente · TODAY</p>
  <div class="header-stats">
    <div class="stat"><span class="stat-num" id="total-count">—</span><span class="stat-label">Exposiciones</span></div>
    <div class="stat"><span class="stat-num" id="city-count">—</span><span class="stat-label">Ciudades</span></div>
  </div>
</header>
<section class="filters-section">
  <div class="filter-row">
    <span class="filter-label">Espacio</span>
    <div class="filter-group">
      <button class="filter-btn vtype-btn active" data-filter="vtype" data-value="all">Todos</button>
      <button class="filter-btn vtype-btn" data-filter="vtype" data-value="Museo">Museo</button>
      <button class="filter-btn vtype-btn" data-filter="vtype" data-value="Galería">Galería</button>
      <button class="filter-btn vtype-btn" data-filter="vtype" data-value="Centro Cultural">Centro Cultural</button>
      <button class="filter-btn vtype-btn" data-filter="vtype" data-value="Feria">Feria</button>
      <button class="filter-btn vtype-btn" data-filter="vtype" data-value="Espacio">Espacio</button>
    </div>
    <div class="ml-auto">
      <div class="search-wrap">
        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="11" cy="11" r="8"/><path d="M21 21l-4.35-4.35"/></svg>
        <input type="text" id="search" placeholder="Buscar…">
      </div>
    </div>
  </div>
  <div class="filter-row">
    <span class="filter-label">Tipo arte</span>
    <div class="filter-group">
      <button class="filter-btn active" data-filter="type" data-value="all">Todos</button>
      <button class="filter-btn" data-filter="type" data-value="Pintura">Pintura</button>
      <button class="filter-btn" data-filter="type" data-value="Fotografía">Fotografía</button>
      <button class="filter-btn" data-filter="type" data-value="Arte Contemporáneo">Contemporáneo</button>
      <button class="filter-btn" data-filter="type" data-value="Escultura">Escultura</button>
      <button class="filter-btn" data-filter="type" data-value="Arte Conceptual">Conceptual</button>
      <button class="filter-btn" data-filter="type" data-value="Multidisciplinar">Multidisciplinar</button>
      <button class="filter-btn" data-filter="type" data-value="Retrospectiva">Retrospectiva</button>
      <button class="filter-btn" data-filter="type" data-value="Arte Digital">Digital</button>
      <button class="filter-btn" data-filter="type" data-value="Dibujo">Dibujo</button>
    </div>
  </div>
  <div class="filter-row">
    <span class="filter-label">Ciudad</span>
    <div class="filter-group" id="city-filters">
      <button class="filter-btn active" data-filter="city" data-value="all">Todas</button>
    </div>
  </div>
</section>
<section class="grid-section">
  <div class="results-bar">
    <span>Mostrando <span class="count" id="shown-count">0</span> de <span class="count" id="total-count2">0</span> exposiciones</span>
    <div class="legend">
      <div class="legend-item"><div class="legend-dot" style="background:var(--blue)"></div>Museo</div>
      <div class="legend-item"><div class="legend-dot" style="background:var(--rose)"></div>Galería</div>
      <div class="legend-item"><div class="legend-dot" style="background:var(--teal)"></div>Centro Cultural</div>
      <div class="legend-item"><div class="legend-dot" style="background:var(--violet)"></div>Feria</div>
      <div class="legend-item"><div class="legend-dot" style="background:var(--orange)"></div>Espacio</div>
    </div>
  </div>
  <div class="grid" id="grid"></div>
</section>
<footer>
  <span class="footer-brand">Exposiciones España</span>
  <span>Auto-actualizado mensualmente · MONTH_YEAR</span>
</footer>
<script>
const typeColors={'Pintura':'#c9a84c','Fotografía':'#4c9c8a','Arte Contemporáneo':'#9c4cc9','Arte Digital':'#4c7ac9','Escultura':'#c97a4c','Arte Conceptual':'#c94c4c','Multidisciplinar':'#8a7f70','Retrospectiva':'#e8c97a','Dibujo':'#6a9c4c'};
const vtypeBadgeStyle={'Galería':'background:rgba(201,76,122,0.15);color:#c94c7a;border:1px solid rgba(201,76,122,0.3)','Museo':'background:rgba(76,122,201,0.15);color:#4c7ac9;border:1px solid rgba(76,122,201,0.3)','Centro Cultural':'background:rgba(76,156,138,0.15);color:#4c9c8a;border:1px solid rgba(76,156,138,0.3)','Feria':'background:rgba(156,76,201,0.15);color:#9c4cc9;border:1px solid rgba(156,76,201,0.3)','Espacio':'background:rgba(201,122,76,0.15);color:#c97a4c;border:1px solid rgba(201,122,76,0.3)'};
const exhibitions=EXHIBITIONS_DATA;
let activeType='all',activeCity='all',activeVtype='all',searchQuery='';
function getStatus(o,c){const now=new Date(),od=new Date(o),cd=new Date(c),d=Math.ceil((cd-now)/864e5);if(now<od)return{label:'Próximamente',cls:'upcoming'};if(d<=21)return{label:`Cierra en ${d} día${d!==1?'s':''}`,cls:'ending-soon'};return{label:'En cartel',cls:'open'}}
function fmt(s){return new Date(s).toLocaleDateString('es-ES',{day:'numeric',month:'short',year:'numeric'})}
function buildCities(){const cities=[...new Set(exhibitions.map(e=>e.city))].sort(),w=document.getElementById('city-filters');cities.forEach(c=>{const b=document.createElement('button');b.className='filter-btn';b.dataset.filter='city';b.dataset.value=c;b.textContent=c;w.appendChild(b)})}
function render(){const q=searchQuery.toLowerCase(),g=document.getElementById('grid');const f=exhibitions.filter(e=>(activeType==='all'||e.type===activeType)&&(activeCity==='all'||e.city===activeCity)&&(activeVtype==='all'||e.vtype===activeVtype)&&(!q||[e.title,e.artist,e.venue,e.city,e.description].some(s=>s.toLowerCase().includes(q))));document.getElementById('shown-count').textContent=f.length;document.getElementById('total-count').textContent=document.getElementById('total-count2').textContent=exhibitions.length;document.getElementById('city-count').textContent=[...new Set(exhibitions.map(e=>e.city))].length;if(!f.length){g.innerHTML='<div class="no-results"><h3>Sin resultados</h3><p>Prueba con otros filtros.</p></div>';return}g.innerHTML=f.map((e,i)=>{const st=getStatus(e.open,e.close),dot=typeColors[e.type]||'#8a7f70',bs=vtypeBadgeStyle[e.vtype]||'';return`<div class="card" data-type="${e.type}" data-vtype="${e.vtype}" style="animation-delay:${Math.min(i*22,400)}ms"><span class="vtype-badge" style="${bs}">${e.vtype}</span><div class="card-type" style="color:${dot}"><span class="card-type-dot" style="background:${dot}"></span>${e.type}</div><div class="card-title">${e.title}</div><div class="card-artist">${e.artist}</div><div class="card-description">${e.description}</div><div class="card-meta"><div class="meta-row"><svg class="meta-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M12 22s-8-4.5-8-11.8A8 8 0 0 1 12 2a8 8 0 0 1 8 8.2c0 7.3-8 11.8-8 11.8z"/><circle cx="12" cy="10" r="3"/></svg><span><span class="meta-val">${e.venue}</span><br><span class="meta-sub">${e.address} · ${e.city}</span></span></div><div class="meta-row"><svg class="meta-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><rect x="3" y="4" width="18" height="18" rx="2"/><path d="M16 2v4M8 2v4M3 10h18"/></svg><span class="meta-val">${fmt(e.open)} — ${fmt(e.close)}</span></div></div><span class="dates-badge ${st.cls}">${st.label}</span></div>`}).join('')}
document.addEventListener('click',ev=>{const b=ev.target.closest('.filter-btn');if(!b)return;const f=b.dataset.filter,v=b.dataset.value;document.querySelectorAll(`.filter-btn[data-filter="${f}"]`).forEach(x=>x.classList.remove('active'));b.classList.add('active');if(f==='type')activeType=v;else if(f==='city')activeCity=v;else if(f==='vtype')activeVtype=v;render()});
document.getElementById('search').addEventListener('input',e=>{searchQuery=e.target.value;render()});
buildCities();render();
</script>
</body>
</html>"""


def fetch_exhibitions():
    client = anthropic.Anthropic()
    print(f"[{TODAY}] Requesting data from Claude {MODEL}…")
    msg = client.messages.create(
        model=MODEL,
        max_tokens=8192,
        messages=[{"role": "user", "content": RESEARCH_PROMPT}]
    )
    raw = msg.content[0].text.strip()
    raw = re.sub(r"^```(?:json)?\s*", "", raw, flags=re.MULTILINE)
    raw = re.sub(r"\s*```$", "", raw, flags=re.MULTILINE)
    data = json.loads(raw)
    print(f"  ✓ Got {len(data)} exhibitions")
    return data


def build_html(data):
    html = HTML_TEMPLATE.replace("EXHIBITIONS_DATA", json.dumps(data, ensure_ascii=False))
    html = html.replace("MONTH_YEAR", MONTH_YEAR)
    html = html.replace("TODAY", TODAY)
    return html


if __name__ == "__main__":
    data = fetch_exhibitions()
    Path("exhibitions.json").write_text(json.dumps(data, ensure_ascii=False, indent=2), encoding="utf-8")
    print("  ✓ Saved exhibitions.json")
    Path("index.html").write_text(build_html(data), encoding="utf-8")
    print("  ✓ Saved index.html")
    print("Done! 🎨")
