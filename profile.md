---
layout: default
title: Profile
---

<div class="card" style="display:grid; gap:14px;">
  <h1>Profiles</h1>
  <h2>Create profile</h2>
  <div class="form-row">
    <input class="input" id="newProfile" placeholder="Initials (op til 6 tegn, fx MKR123)" maxlength="6">
    <button class="btn" id="addProfile" type="button">Save</button>
  </div>
  <p class="meta">Profiler deles via Supabase. Kun A–Z og 0–9 er tilladt (op til 6 tegn).</p>
  <hr class="sep">
  <h2>All profiles</h2>
  <div id="profilesList" style="display:grid; gap:10px;"></div>
</div>

<div class="card" id="profileDetail" style="display:none; gap:14px;">
  <h1 id="detailTitle">Profile</h1>
  <div style="display:flex; align-items:center; gap:14px; flex-wrap:wrap;">
    <div class="avatar" id="detailAvatar">??</div>
    <p class="meta" id="detailInfo"></p>
  </div>
  <hr class="sep">
  <h2>Last 10 matches</h2>
  <ul class="list" id="detailMatches"></ul>
  <hr class="sep">
  <h2>Booster / Money ledger</h2>
  <div style="display:grid; gap:12px; grid-template-columns: repeat(auto-fit,minmax(260px,1fr));">
    <div class="card" style="padding:16px;">
      <h2 style="margin-bottom:8px;">De skylder mig</h2>
      <ul class="list" id="ledgerOwedToMe"></ul>
    </div>
    <div class="card" style="padding:16px;">
      <h2 style="margin-bottom:8px;">Jeg skylder</h2>
      <ul class="list" id="ledgerIOwe"></ul>
    </div>
  </div>
</div>

<script>
(async function(){
  const listEl = document.getElementById('profilesList');
  const detailCard = document.getElementById('profileDetail');
  const title = document.getElementById('detailTitle');
  const avatar = document.getElementById('detailAvatar');
  const info = document.getElementById('detailInfo');
  const lastList = document.getElementById('detailMatches');
  const owedToMe = document.getElementById('ledgerOwedToMe');
  const iOwe     = document.getElementById('ledgerIOwe');

  function fitAvatar(el, text){
    const len = (text||'').length;
    let size = 28;
    if (len >= 6) size = 16;
    else if (len === 5) size = 18;
    else if (len === 4) size = 20;
    else if (len === 3) size = 22;
    else size = 28;
    el.style.fontSize = size + 'px';
  }
  function fmtWhen(ts){
    const d = new Date(ts);
    const pad = n=> String(n).padStart(2,'0');
    return `${d.getFullYear()}-${pad(d.getMonth()+1)}-${pad(d.getDate())} ${pad(d.getHours())}:${pad(d.getMinutes())}:${pad(d.getSeconds())}`;
  }

  async function fetchProfiles(){
    const { data, error } = await sb.from('profiles').select('initials').order('created_at', { ascending:false });
    if (error) { console.error(error); return []; }
    return data.map(r => ({ i: r.initials }));
  }
  async function createProfile(initials){
    const i = up6(initials);
    if (!i) return;
    const { error } = await sb.from('profiles').insert({ initials: i });
    if (error && error.code !== '23505') console.error(error);
    await renderProfiles();
  }
  async function deleteProfile(initials){
    await sb.from('profiles').delete().eq('initials', up6(initials));
    await renderProfiles();
    if (title.dataset.u === up6(initials)) detailCard.style.display = 'none';
  }
  async function fetchMatchesFor(initials, limit=null){
    const i = up6(initials);
    let q = sb.from('matches')
      .select('id,p1,p2,when_ts,score,winner,bet_type,amount')
      .or(`p1.eq.${i},p2.eq.${i}`)
      .order('when_ts', { ascending:false });
    if (limit) q = q.limit(limit);
    const { data, error } = await q;
    if (error) { console.error(error); return []; }
    return data.map(r => ({
      id: r.id, p1: r.p1, p2: r.p2,
      when: fmtWhen(r.when_ts),
      score: r.score, winner: r.winner,
      bet: { type: r.bet_type, amount: r.amount }
    }));
  }

  document.getElementById('addProfile').addEventListener('click', async ()=>{
    const inp = document.getElementById('newProfile');
    await createProfile(inp.value);
    inp.value = '';
  });

  async function renderProfiles(){
    const arr = await fetchProfiles();
    listEl.innerHTML = '';
    if (!arr.length){
      const empty = document.createElement('div');
      empty.className = 'item';
      empty.innerHTML = '<span class="meta">Ingen profiler endnu. Opret ovenfor.</span>';
      listEl.appendChild(empty);
      return;
    }
    arr.forEach(({ i })=>{
      const row = document.createElement('div');
      row.className = 'item';
      const left = document.createElement('div');
      left.style.display='flex';
      left.style.alignItems='center';
      left.style.gap='10px';
      const av = document.createElement('div'); 
      av.className = 'avatar'; 
      av.textContent = i;
      fitAvatar(av, i);
      const txt = document.createElement('div');
      txt.innerHTML = `<strong>${i}</strong><div class="meta">Open for profile & matches</div>`;
      left.append(av, txt);
      const open = document.createElement('button');
      open.className = 'btn';
      open.textContent = 'Open';
      open.addEventListener('click', ()=> showProfileDetail(i));
      const del = document.createElement('button');
      del.className = 'btn ghost';
      del.textContent = 'Delete';
      del.addEventListener('click', ()=> deleteProfile(i));
      const right = document.createElement('div');
      right.style.display='flex';
      right.style.gap='8px';
      right.append(open, del);
      row.append(left, right);
      listEl.appendChild(row);
    });
  }

  async function showProfileDetail(initials){
    const u = up6(initials);
    detailCard.style.display = 'grid';
    title.textContent = `Profile: ${u}`;
    title.dataset.u = u;
    avatar.textContent = u;
    fitAvatar(avatar, u);
    info.textContent = 'Seneste 10 kampe, samt netto booster/money-gæld mod hver modstander.';
    const last10 = await fetchMatchesFor(u, 10);
    lastList.innerHTML = '';
    if (!last10.length){
      const li = document.createElement('li');
      li.className = 'item';
      li.innerHTML = '<span class="meta">Ingen kampe endnu.</span>';
      lastList.appendChild(li);
    } else {
      last10.forEach(m=>{
        const li = document.createElement('li'); li.className='item';
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
        li.append(left);
        lastList.appendChild(li);
           });
    }

    // Ledger (brug alle kampe for u, ikke kun 10)
    const all = await fetchMatchesFor(u, null);
    const ledger = {}; // opp => { booster: net, money: net }
    all.forEach(m=>{
      const type = (m.bet && (m.bet.type==='booster' || m.bet.type==='money')) ? m.bet.type : null;
      const amt = Number(m.bet?.amount || 0);
      if (!type || amt <= 0) return;

      const meIsP1 = up6(m.p1) === u;
      const opp = meIsP1 ? up6(m.p2) : up6(m.p1);
      if (!ledger[opp]) ledger[opp] = { booster: 0, money: 0 };

      if (m.winner === 'p1'){
        if (meIsP1) ledger[opp][type] += amt;  // de skylder mig
        else        ledger[opp][type] -= amt;  // jeg skylder dem
      } else if (m.winner === 'p2'){
        if (meIsP1) ledger[opp][type] -= amt;  // jeg skylder dem
        else        ledger[opp][type] += amt;  // de skylder mig
      }
    });

    // Tøm og genopbyg listerne
    owedToMe.innerHTML = '';
    iOwe.innerHTML = '';

    const opps = Object.keys(ledger);
    if (!opps.length){
      const a = document.createElement('li'); a.className='item';
      a.innerHTML = '<span class="meta">Ingen gæld registreret.</span>';
      owedToMe.appendChild(a);
      const b = document.createElement('li'); b.className='item';
      b.innerHTML = '<span class="meta">Ingen gæld registreret.</span>';
      iOwe.appendChild(b);
      return;
    }

    opps.forEach(opp=>{
      const { booster, money } = ledger[opp];

      // De skylder mig (positive værdier)
      if ((booster||0) > 0 || (money||0) > 0){
        const li = document.createElement('li'); li.className='item';
        const parts = [];
        if (booster > 0) parts.push(`Booster × ${booster}`);
        if (money   > 0) parts.push(`Money: ${money}`);
        li.innerHTML = `<strong>${opp}</strong><div class="meta">${parts.join(' • ') || '—'}</div>`;
        owedToMe.appendChild(li);
      }

      // Jeg skylder (negative værdier)
      if ((booster||0) < 0 || (money||0) < 0){
        const li = document.createElement('li'); li.className='item';
        const parts = [];
        if (booster < 0) parts.push(`Booster × ${Math.abs(booster)}`);
        if (money   < 0) parts.push(`Money: ${Math.abs(money)}`);
        li.innerHTML = `<strong>${opp}</strong><div class="meta">${parts.join(' • ') || '—'}</div>`;
        iOwe.appendChild(li);
      }
    });

    // Hvis en af listerne endte tom, vis “ingen…”
    if (!owedToMe.children.length){
      const li = document.createElement('li'); li.className='item';
      li.innerHTML = '<span class="meta">Ingen gæld registreret.</span>';
      owedToMe.appendChild(li);
    }
    if (!iOwe.children.length){
      const li = document.createElement('li'); li.className='item';
      li.innerHTML = '<span class="meta">Ingen gæld registreret.</span>';
      iOwe.appendChild(li);
    }
  }

  // Første render af profil-listen
  await renderProfiles();
})();
</script>
