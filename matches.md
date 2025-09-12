---
layout: default
title: Matches
---

<script>
  // ← Sæt din Apps Script Web App URL (slutter på /exec)
  const API = "https://script.google.com/macros/s/AKfycbzprP3330YCRKrHfF-kqoR9pIOzkePw_zTyrIfqUWJTvZAjiLMHfkzR8bY4coutv_4srA/exec";

  const up6 = s => (s||'').toUpperCase().replace(/[^A-Z0-9]/g,'').slice(0,6);

  const api = {
    async listMatches() {
      return fetch(`${API}?action=list_matches`)
        .then(r=>r.json()).then(j=>j.data||[]);
    },
    async addMatch({p1,p2,when_ts,score,winner,bet_type,amount}) {
      const qs = new URLSearchParams({
        action:'add_match', p1,p2, when_ts, score: (score||''), winner: (winner||''),
        bet_type, amount: String(amount)
      });
      return fetch(`${API}?${qs.toString()}`)
        .then(r=>r.json());
    },
    async deleteMatch(id) {
      const qs = new URLSearchParams({ action:'delete_match', id });
      return fetch(`${API}?${qs.toString()}`).then(r=>r.json());
    }
  };
</script>

<div class="card">
  <h1>Create Match</h1>
  <div class="form-row" style="margin-top:8px;">
    <input class="input" id="p1" placeholder="Player 1 (fx initialer)" required>
    <input class="input" id="p2" placeholder="Player 2 (fx initialer)" required>
    <input class="input" id="date" type="date" aria-label="Date">
  </div>
  <div class="form-row" style="margin-top:8px;">
    <input class="input" id="score" placeholder="Score (fx 3-2)">
    <select class="input" id="winner" aria-label="Winner">
      <option value="">Winner?</option>
      <option value="p1">Player 1</option>
      <option value="p2">Player 2</option>
    </select>
    <select class="input" id="betType" aria-label="Bet type" required>
      <option value="">Bet type</option>
      <option value="booster">Booster</option>
      <option value="money">Money</option>
    </select>
    <input class="input" id="amount" type="number" min="1" max="5000" placeholder="Amount (1–5000)">
    <button class="btn" id="add" type="button">Add</button>
  </div>
  <hr style="border:0; height:1px; background: var(--border); margin:16px 0;">
  <h2>All Matches</h2>
  <ul class="list" id="list"></ul>
</div>

<script>
(function(){
  const listEl  = document.getElementById('list');
  const p1      = document.getElementById('p1');
  const p2      = document.getElementById('p2');
  const dateIn  = document.getElementById('date');
  const score   = document.getElementById('score');
  const winner  = document.getElementById('winner');
  const betType = document.getElementById('betType');
  const amount  = document.getElementById('amount');

  function nowTimeHHMMSS(){
    const d = new Date();
    const pad = n => String(n).padStart(2,'0');
    return `${pad(d.getHours())}:${pad(d.getMinutes())}:${pad(d.getSeconds())}`;
  }
  function combineDateWithNow(dateStr){
    const d = dateStr ? new Date(dateStr) : new Date();
    const yyyy = d.getFullYear();
    const mm = String(d.getMonth()+1).padStart(2,'0');
    const dd = String(d.getDate()).padStart(2,'0');
    return `${yyyy}-${mm}-${dd} ${nowTimeHHMMSS()}`;
  }
  function fmtWhen(ts){
    const d = new Date(ts);
    const pad = n=> String(n).padStart(2,'0');
    return `${d.getFullYear()}-${pad(d.getMonth()+1)}-${pad(d.getDate())} ${pad(d.getHours())}:${pad(d.getMinutes())}:${pad(d.getSeconds())}`;
  }

  async function render(){
    const items = await api.listMatches();
    listEl.innerHTML = '';
    if (!items.length){
      const li = document.createElement('li');
      li.className = 'item';
      li.innerHTML = '<span class="meta">Ingen matches endnu. Tilføj din første ovenfor.</span>';
      listEl.appendChild(li);
      return;
    }
    items.forEach(m=>{
      const li = document.createElement('li'); li.className = 'item';
      const left = document.createElement('div');
      const wtxt = m.winner === 'p1' ? up6(m.p1) : (m.winner === 'p2' ? up6(m.p2) : '—');
      const betText = m.bet?.type === 'booster'
        ? `Booster × ${m.bet.amount}`
        : (m.bet?.type === 'money' ? `Money: ${m.bet.amount}` : '—');
      left.innerHTML = `
        <div><strong>${up6(m.p1)}</strong> vs <strong>${up6(m.p2)}</strong></div>
        <div class="meta">${m.when || ''}</div>
        <div class="meta">Score: ${m.score || '—'} • Winner: ${wtxt} • Bet: ${betText}</div>
      `;
      const right = document.createElement('div');
      const del = document.createElement('button'); del.className = 'btn ghost'; del.textContent = 'Delete';
      del.addEventListener('click', async ()=>{ await api.deleteMatch(m.id); await render(); });
      right.appendChild(del);
      li.append(left, right);
      listEl.appendChild(li);
    });
  }

  document.getElementById('add').addEventListener('click', async ()=>{
    const a = up6(p1.value), b = up6(p2.value);
    const t = betType.value; const amt = Number(amount.value);
    if (!a || !b) return;
    if (!(t==='booster' || t==='money')) return;
    if (!Number.isFinite(amt) || amt<1 || amt>5000) return;
    const iso = combineDateWithNow(dateIn.value).replace(' ','T'); // "YYYY-MM-DDTHH:mm:ss"
    await api.addMatch({ p1:a, p2:b, when_ts: iso, score: (score.value||'').trim(), winner: winner.value||'', bet_type: t, amount: amt });
    p1.value=''; p2.value=''; dateIn.value=''; score.value=''; winner.value=''; betType.value=''; amount.value='';
    await render();
  });

  render();
})();
</script>
