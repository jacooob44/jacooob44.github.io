---
layout: default
title: Profile
---

<!-- API + JSONP helper -->
<script>
  /* ← Sæt din Apps Script Web App URL (slutter på /exec) */
  const API = "https://script.google.com/macros/s/AKfycbzRYfQAeVvWgr4ZlPYP93oqrolkgRTjwydDATrRrFD19Cn1xJTSsydwJ09YoZz0O_Po8g/exec";

  // Lille badge så vi kan se at JS kører og hvilken API der bruges
  window.addEventListener('DOMContentLoaded', () => {
    const b = document.createElement('div');
    b.className = 'meta';
    b.style.margin = '8px 0';
    b.textContent = 'HELD OG LYKKE DERUDE - FLERE OPDATERINGER KOMMER LØBENDE :)';
    document.querySelector('main')?.prepend(b);
  });

  // JSONP helper (ingen CORS)
  function jsonp(url){
    return new Promise((resolve, reject) => {
      const cb = 'cb_' + Math.random().toString(36).slice(2);
      const s = document.createElement('script');
      window[cb] = (data) => { resolve(data); delete window[cb]; s.remove(); };
      s.onerror = (e) => { reject(e); delete window[cb]; s.remove(); };
      // cache-buster
      s.src = url + (url.includes('?') ? '&' : '?') + 'callback=' + cb + '&_=' + Date.now();
      document.head.appendChild(s);
    });
  }

  const up6 = s => (s||'').toUpperCase().replace(/[^A-Z0-9]/g,'').slice(0,6);

  // API wrapper
  const api = {
    async listProfiles(){
      const res = await jsonp(`${API}?action=list_profiles`);
      console.log('[API] listProfiles ->', res);
      if (!res?.ok) return [];
      // Tål både objekter og rene strenge
      return (res.data || []).map(x => {
        if (typeof x === 'string') return { initials: up6(x) };
        if (x && typeof x === 'object' && 'initials' in x) return { initials: up6(x.initials) };
        return null;
      }).filter(Boolean);
    },
    addProfile(initials){
      const qs = new URLSearchParams({ action:'add_profile', initials: up6(initials) });
      return jsonp(`${API}?${qs.toString()}`); // {ok:true}
    },
    deleteProfile(initials){
      const qs = new URLSearchParams({ action:'delete_profile', initials: up6(initials) });
      return jsonp(`${API}?${qs.toString()}`); // {ok:true}
    },
    async listMatchesFor(initials, limit){
      const qs = new URLSearchParams({ action:'list_matches', initials: up6(initials), limit: limit?String(limit):'' });
      const res = await jsonp(`${API}?${qs.toString()}`);
      console.log('[API] listMatchesFor ->', res);
      return res?.ok ? (res.data || []) : [];
    }
  };
</script>

<div class="card" style="display:grid; gap:14px;">
  <h1>Profiler</h1>

  <h2>Opret Konsulent</h2>
  <div class="form-row">
    <input class="input" id="newProfile" placeholder="Norlys Initialer" maxlength="6">
    <button class="btn" id="addProfile" type="button">Save</button>
  </div>
  <p class="meta">Profiler ses nedenfor...</p>

  <hr class="sep">

  <h2>Profiler!</h2>
  <div id="profilesList" style="display:grid; gap:10px;"></div>
</div>

<!-- Profil-detaljer nederst på siden -->
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

  <h2>GÆLD</h2>
  <div style="display:grid; gap:12px; grid-template-columns: repeat(auto-fit,minmax(260px,1fr));">
    <div class="card" style="padding:16px;">
      <h2 style="margin-bottom:8px;">De skylder!</h2>
      <ul class="list" id="ledgerOwedToMe"></ul>
    </div>
    <div class="card" style="padding:16px;">
      <h2 style="margin-bottom:8px;">Jeg skylder!</h2>
      <ul class="list" id="ledgerIOwe"></ul>
    </div>
  </div>
</div>

<script>
(function(){
  // DOM refs
  const listEl     = document.getElementById('profilesList');
  const detailCard = document.getElementById('profileDetail');
  const title      = document.getElementById('detailTitle');
  const avatar     = document.getElementById('detailAvatar');
  const info       = document.getElementById('detailInfo');
  const lastList   = document.getElementById('detailMatches');
  const owedToMe   = document.getElementById('ledgerOwedToMe');
  const iOwe       = document.getElementById('ledgerIOwe');

  // Helpers
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

  // Create profile
  document.getElementById('addProfile').addEventListener('click', async ()=>{
    const inp = document.getElementById('newProfile');
    const i = up6(inp.value);
    if (!i) return;
    await api.addProfile(i);
    inp.value = '';
    await renderProfiles();
  });

  // Render alle profiler
  async function renderProfiles(){
    const arr = await api.listProfiles(); // [{initials:'…'}]
    listEl.innerHTML = '';

    if (!arr.length){
      const empty = document.createElement('div');
      empty.className = 'item';
      empty.innerHTML = '<span class="meta">Ingen profiler endnu. Opret ovenfor.</span>';
      listEl.appendChild(empty);

      // DEBUG: vis rå data fra API (hjælper hvis noget stadig er off)
      const dbg = document.createElement('pre');
      dbg.style.background = '#111'; dbg.style.color = '#0f0';
      dbg.style.padding = '8px'; dbg.style.marginTop = '8px'; dbg.style.overflow = 'auto';
      jsonp(API + '?action=list_profiles').then(raw=>{
        dbg.textContent = 'DEBUG listProfiles råt:\n' + JSON.stringify(raw, null, 2);
      });
      listEl.appendChild(dbg);
      return;
    }

    arr.forEach(({ initials:i })=>{
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
      txt.innerHTML = `<strong>${i}</strong><div class="meta">Åben profil for data..</div>`;
      left.append(av, txt);

      const open = document.createElement('button');
      open.className = 'btn';
      open.textContent = 'Open';
      open.addEventListener('click', ()=> showProfileDetail(i));

      const del = document.createElement('button');
      del.className = 'btn ghost';
      del.textContent = 'Delete';
      del.addEventListener('click', async ()=>{
        await api.deleteProfile(i);
        await renderProfiles();
        if ((title.dataset.u||'') === i) detailCard.style.display='none';
      });

      const right = document.createElement('div');
      right.style.display='flex'; right.style.gap='8px';
      right.append(open, del);

      row.append(left, right);
      listEl.appendChild(row);
    });
  }

  // Vis profil-detaljer
  async function showProfileDetail(initials){
    const u = up6(initials);
    detailCard.style.display = 'grid';
    title.textContent = `Norlys Konsulent: ${u}`;
    title.dataset.u = u;
    avatar.textContent = u;
    fitAvatar(avatar, u);
    info.textContent = 'Nedenfor ses seneste 10 kampe, plus konsulentens gæld... #ROFUS';

    const last10 = await api.listMatchesFor(u, 10);
    lastList.innerHTML = '';
    if (!last10.length){
      const li = document.createElement('li'); li.className = 'item';
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

    // Ledger (brug ALLE kampe for u)
    const all = await api.listMatchesFor(u);
    const ledger = {}; // opp => { booster: net, money: net }
    all.forEach(m=>{
      const type = (m.bet && (m.bet.type==='booster' || m.bet.type==='money')) ? m.bet.type : null;
      const amt = Number(m.bet?.amount || 0);
      if (!type || amt <= 0) return;
      const meIsP1 = up6(m.p1) === u;
      const opp = meIsP1 ? up6(m.p2) : up6(m.p1);
      if (!ledger[opp]) ledger[opp] = { booster: 0, money: 0 };
      if (m.winner === 'p1'){
        if (meIsP1) ledger[opp][type] += amt; else ledger[opp][type] -= amt;
      } else if (m.winner === 'p2'){
        if (meIsP1) ledger[opp][type] -= amt; else ledger[opp][type] += amt;
      }
    });

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
      if ((booster||0) > 0 || (money||0) > 0){
        const li = document.createElement('li'); li.className='item';
        const parts = [];
        if (booster > 0) parts.push(`Booster × ${booster}`);
        if (money   > 0) parts.push(`Money: ${money}`);
        li.innerHTML = `<strong>${opp}</strong><div class="meta">${parts.join(' • ') || '—'}</div>`;
        owedToMe.appendChild(li);
      }
      if ((booster||0) < 0 || (money||0) < 0){
        const li = document.createElement('li'); li.className='item';
        const parts = [];
        if (booster < 0) parts.push(`Booster × ${Math.abs(booster)}`);
        if (money   < 0) parts.push(`Money: ${Math.abs(money)}`);
        li.innerHTML = `<strong>${opp}</strong><div class="meta">${parts.join(' • ') || '—'}</div>`;
        iOwe.appendChild(li);
      }
    });

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

  // Første render
  renderProfiles();

})();
</script>
