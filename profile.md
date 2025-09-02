---
title: Profile
---

<div class="card" style="display:grid; gap:14px;">
  <h1>Profiles</h1>

  <!-- ÉT sted at oprette profil -->
  <h2>Create profile</h2>
  <div class="form-row">
    <input class="input" id="newProfile" placeholder="Initials (op til 6 tegn, fx MKR123)" maxlength="6">
    <button class="btn" id="addProfile" type="button">Save</button>
  </div>
  <p class="meta">Profiler gemmes lokalt i din browser. Kun A–Z og 0–9 er tilladt.</p>

  <hr class="sep">

  <!-- Liste over alle profiler -->
  <h2>All profiles</h2>
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

  <h2>Booster ledger</h2>
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
(function(){
  const KEY_PROFILES = 'profiles.list'; // [{ i: "ABC123" }]
  const KEY_MATCHES  = 'matches.items'; // [{ p1, p2, when, score, winner, bet:{type,amount} }]
  const up6  = s => (s||'').toUpperCase().slice(0,6).replace(/[^A-Z0-9]/g,'');
  const load = (k, fb) => JSON.parse(localStorage.getItem(k) || JSON.stringify(fb));
  const save = (k, v) => localStorage.setItem(k, JSON.stringify(v));

  // Avatar fit
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

  // Opret profil
  const newProfile = document.getElementById('newProfile');
  document.getElementById('addProfile').addEventListener('click', ()=>{
    const v = up6(newProfile.value);
    if (!v) return;
    const arr = load(KEY_PROFILES, []);
    if (!arr.some(p=>p.i===v)) {
      arr.unshift({ i: v });
      localStorage.setItem('profiles.list', JSON.stringify(arr));
      renderProfiles();
    }
    newProfile.value = ''; // reset
  });

  // Liste over profiler
  const listEl = document.getElementById('profilesList');
  function renderProfiles(){
    const arr = load(KEY_PROFILES, []);
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
      del.addEventListener('click', ()=>{
        const next = load(KEY_PROFILES, []).filter(p => p.i !== i);
        localStorage.setItem('profiles.list', JSON.stringify(next));
        renderProfiles();
        // hvis man sletter den viste profil, så skjul detalje
        const title = document.getElementById('detailTitle');
        if (title.dataset.u === i) {
          document.getElementById('profileDetail').style.display = 'none';
        }
      });

      const right = document.createElement('div');
      right.style.display='flex';
      right.style.gap='8px';
      right.append(open, del);

      row.append(left, right);
      listEl.appendChild(row);
    });
  }
  renderProfiles();

  // Vis profil-detaljer nederst
  function showProfileDetail(initials){
    const u = up6(initials);
    const detailCard = document.getElementById('profileDetail');
    const title = document.getElementById('detailTitle');
    const avatar = document.getElementById('detailAvatar');
    const info = document.getElementById('detailInfo');
    const list = document.getElementById('detailMatches');
    const owedToMe = document.getElementById('ledgerOwedToMe');
    const iOwe     = document.getElementById('ledgerIOwe');

    detailCard.style.display = 'grid';
    title.textContent = `Profile: ${u}`;
    title.dataset.u = u;
    avatar.textContent = u;
    fitAvatar(avatar, u);
    info.textContent = 'Seneste 10 kampe, samt netto booster/money-gæld mod hver modstander.';

    // hent kampe for u
    const all = load(KEY_MATCHES, []);
    const mine = all.filter(m => up6(m.p1) === u || up6(m.p2) === u);

    // Seneste 10
    list.innerHTML = '';
    const last10 = mine.slice(0, 10);
    if (!last10.length){
      const li = document.createElement('li');
      li.className = 'item';
      li.innerHTML = '<span class="meta">Ingen kampe endnu.</span>';
      list.appendChild(li);
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
        list.appendChild(li);
      });
    }

    // Ledger: netto pr. modstander og pr. bet-type
    // + tal = modstander skylder mig; - tal = jeg skylder modstanderen
    const ledger = {}; // { OPP: { booster: net, money: net } }
    mine.forEach(m=>{
      if (!m.bet || !m.bet.type || !m.bet.amount) return;
      const type = (m.bet.type === 'booster' ? 'booster' : (m.bet.type === 'money' ? 'money' : null));
      if (!type) return;
      const amt = Number(m.bet.amount) || 0;
      if (amt <= 0) return;

      const meIsP1 = up6(m.p1) === u;
      const opp = meIsP1 ? up6(m.p2) : up6(m.p1);

      if (!ledger[opp]) ledger[opp] = { booster: 0, money: 0 };

      if (m.winner === 'p1'){
        // p1 vandt
        if (meIsP1) ledger[opp][type] += amt;  // de skylder mig
        else        ledger[opp][type] -= amt;  // jeg skylder dem
      } else if (m.winner === 'p2'){
        // p2 vandt
        if (meIsP1) ledger[opp][type] -= amt;  // jeg skylder dem
        else        ledger[opp][type] += amt;  // de skylder mig
      } 
      // hvis winner tom => ingen gældsændring
    });

    // udfyld to lister
    owedToMe.innerHTML = '';
    iOwe.innerHTML = '';

    const opponents = Object.keys(ledger);
    if (!opponents.length){
      const a = document.createElement('li'); a.className='item';
      a.innerHTML = '<span class="meta">Ingen gæld registreret.</span>';
      owedToMe.appendChild(a);
      const b = document.createElement('li'); b.className='item';
      b.innerHTML = '<span class="meta">Ingen gæld registreret.</span>';
      iOwe.appendChild(b);
      return;
    }

    opponents.forEach(opp=>{
      const { booster, money } = ledger[opp];
      // “De skylder mig”
      if ((booster||0) > 0 || (money||0) > 0){
        const li = document.createElement('li'); li.className='item';
        const parts = [];
        if (booster > 0) parts.push(`Booster × ${booster}`);
        if (money   > 0) parts.push(`Money: ${money}`);
        li.innerHTML = `<strong>${opp}</strong><div class="meta">${parts.join(' • ')}</div>`;
        owedToMe.appendChild(li);
      }
      // “Jeg skylder”
      if ((booster||0) < 0 || (money||0) < 0){
        const li = document.createElement('li'); li.className='item';
        const parts = [];
        if (booster < 0) parts.push(`Booster × ${Math.abs(booster)}`);
        if (money   < 0) parts.push(`Money: ${Math.abs(money)}`);
        li.innerHTML = `<strong>${opp}</strong><div class="meta">${parts.join(' • ')}</div>`;
        iOwe.appendChild(li);
      }
    });

    // hvis listerne blev tomme, vis “ingen…”
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
})();
</script>
