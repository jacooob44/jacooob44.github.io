---
layout: default
title: Matches
---

<!-- Supabase init -->
<script src="https://unpkg.com/@supabase/supabase-js@2/dist/umd/supabase.min.js"></script>
<script>
  if (!window.sb) {
    var SUPABASE_URL = "https://wmuvougpavpoybuvkvgq.supabase.co";
    var SUPABASE_ANON_KEY = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6IndtdXZvdWdwYXZwb3lidXZrdmdxIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NTY4MjEzMTIsImV4cCI6MjA3MjM5NzMxMn0.gBS-5DmvVXRdeYtGhax76J52u1-9JCGZXjwFd31IxbY";
    window.sb = supabase.createClient(SUPABASE_URL, SUPABASE_ANON_KEY);
  }
  if (!window.up6) {
    window.up6 = s => (s||'').toUpperCase().slice(0,6).replace(/[^A-Z0-9]/g,'');
  }
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
(async function(){
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

  async function upsertProfile(i){
    const initials = up6(i);
    if (!initials) return;
    await sb.from('profiles').insert({ initials }).select().single().catch(()=>{});
  }

  async function createMatch(){
    const a = up6(p1.value);
    const b = up6(p2.value);
    const t = betType.value;
    const amt = Number(amount.value);

    if (!a || !b) return;
    if (!t || !(t === 'booster' || t === 'money')) return;
    if (!Number.isFinite(amt) || amt < 1 || amt > 5000) return;

    const whenLocal = combineDateWithNow(dateIn.value);
    const iso = whenLocal.replace(' ', 'T');

    const row = {
      p1: a, p2: b,
      when_ts: iso,
      score: (score.value || '').trim() || null,
      winner: winner.value || null,
      bet_type: t,
      amount: amt
    };
    const { error } = await sb.from('matches').insert(row);
    if (!error){
      await Promise.all([upsertProfile(a), upsertProfile(b)]);
      await render();
      p1.value=''; p2.value=''; dateIn.value=''; score.value=''; winner.value=''; betType.value=''; amount.value='';
    } else {
      console.error(error);
    }
  }

  async function fetchMatches(){
    const { data, error } = await sb.from('matches').select('id,p1,p2,when_ts,score,winner,bet_type,amount').order('when_ts', { ascending:false });
    if (error) { console.error(error); return []; }
    return data.map(r => ({
      id: r.id, p1: r.p1, p2: r.p2,
      when: fmtWhen(r.when_ts),
      score: r.score, winner: r.winner,
      bet: { type: r.bet_type, amount: r.amount }
    }));
  }

  async function deleteMatch(id){
    await sb.from('matches').delete().eq('id', id);
    await render();
  }

  async function render(){
    const items = await fetchMatches();
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
      del.addEventListener('click', ()=> deleteMatch(m.id));
      right.appendChild(del);
      li.append(left, right);
      listEl.appendChild(li);
    });
  }

  document.getElementById('add').addEventListener('click', createMatch);
  await render();
})();
</script>
