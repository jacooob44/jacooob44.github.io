---
title: Matches
---

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
  const KEY_MATCHES  = 'matches.items';
  const KEY_PROFILES = 'profiles.list';
  const KEY_ME       = 'profile.initials'; // valgfrit prefill

  const listEl  = document.getElementById('list');
  const p1      = document.getElementById('p1');
  const p2      = document.getElementById('p2');
  const dateIn  = document.getElementById('date');
  const score   = document.getElementById('score');
  const winner  = document.getElementById('winner');
  const betType = document.getElementById('betType');
  const amount  = document.getElementById('amount');
  const addBtn  = document.getElementById('add');

  const up6  = s => (s||'').toUpperCase().slice(0,6).replace(/[^A-Z0-9]/g,'');
  const load = (k, fb) => JSON.parse(localStorage.getItem(k) || JSON.stringify(fb));
  const save = (k, v) => localStorage.setItem(k, JSON.stringify(v));

  function ensureProfiles(...initials){
    const arr = load(KEY_PROFILES, []);
    let changed = false;
    initials.forEach(i=>{
      const v = up6(i);
      if (v && !arr.some(p=>p.i===v)){ arr.push({i:v}); changed = true; }
    });
    if (changed) save(KEY_PROFILES, arr);
  }

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

  function render(){
    const items = load(KEY_MATCHES, []);
    listEl.innerHTML = '';
    if (!items.length){
      const li = document.createElement('li');
      li.className = 'item';
      li.innerHTML = '<span class="meta">Ingen matches endnu. Tilføj din første ovenfor.</span>';
      listEl.appendChild(li);
      return;
    }
    items.forEach((m, idx)=>{
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
      del.addEventListener('click', ()=>{
        const arr = load(KEY_MATCHES, []);
        arr.splice(idx,1);
        save(KEY_MATCHES, arr);
        render();
      });
      right.appendChild(del);
      li.append(left, right);
      listEl.appendChild(li);
    });
  }

  addBtn.addEventListener('click', ()=>{
    const a = up6(p1.value);
    const b = up6(p2.value);
    const t = betType.value;
    const amt = Number(amount.value);

    if (!a || !b) return;
    if (!t || !(t === 'booster' || t === 'money')) return;
    if (!Number.isFinite(amt) || amt < 1 || amt > 5000) return;

    const when = combineDateWithNow(dateIn.value);
    const item = {
      p1: a,
      p2: b,
      when,
      score: (score.value || '').trim(),
      winner: winner.value, // 'p1' | 'p2' | ''
      bet: { type: t, amount: amt }
    };
    const arr = load(KEY_MATCHES, []);
    arr.unshift(item);
    save(KEY_MATCHES, arr);

    // auto-tilføj profiler
    ensureProfiles(a, b);

    // reset felter
    p1.value=''; p2.value='';
    dateIn.value=''; score.value=''; winner.value='';
    betType.value=''; amount.value='';
    render();
  });

  const me = localStorage.getItem(KEY_ME);
  if (me) p1.value = up6(me);

  render();
})();
</script>
