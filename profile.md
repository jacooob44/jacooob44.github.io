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
  <p class="meta">Profiler gemmes lokalt i din browser. Kun A–Z og 0–9 tilladt.</p>

  <hr class="sep">

  <!-- Liste over alle profiler -->
  <h2>All profiles</h2>
  <div id="profilesList" style="display:grid; gap:10px;"></div>
</div>

<!-- Profil-detaljevisning via ?u=XXXX -->
<div class="card" id="profileDetail" style="display:none; gap:10px;">
  <h1 id="detailTitle">Profile</h1>
  <div style="display:flex; align-items:center; gap:14px; flex-wrap:wrap;">
    <div class="avatar" id="detailAvatar">??</div>
    <p class="meta" id="detailInfo"></p>
  </div>
  <hr class="sep">
  <h2>Matches for this profile</h2>
  <ul class="list" id="detailMatches"></ul>
</div>

<script>
(function(){
  const KEY_PROFILES = 'profiles.list'; // [{ i: "ABC123" }]
  const KEY_MATCHES  = 'matches.items'; // [{ p1, p2, when, score, winner, bet }]
  const KEY_ME       = 'profile.initials'; // valgfrit: gem din "egen" til prefill på matches

  const byId = id => document.getElementById(id);
  const load = (k, fb) => JSON.parse(localStorage.getItem(k) || JSON.stringify(fb));
  const save = (k, v) => localStorage.setItem(k, JSON.stringify(v));
  const up6  = s => (s||'').toUpperCase().slice(0,6).replace(/[^A-Z0-9]/g,'');

  // Opret profil (ét sted)
  const newProfile = byId('newProfile');
  byId('addProfile').addEventListener('click', ()=>{
    const v = up6(newProfile.value);
    if (!v) return;
    const arr = load(KEY_PROFILES, []);
    if (!arr.some(p=>p.i===v)) {
      arr.unshift({ i: v });
      save(KEY_PROFILES, arr);
      // første profil kan gemmes som "min" for prefill på matches
      if (!localStorage.getItem(KEY_ME)) localStorage.setItem(KEY_ME, v);
      renderProfiles();
    }
    // reset & ryd feltet
    newProfile.value = '';
  });

  // Liste over profiler
  const listEl = byId('profilesList');
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

      const av = document.createElement('div'); av.className = 'avatar'; av.textContent = i;
      const txt = document.createElement('div');
      txt.innerHTML = `<strong>${i}</strong><div class="meta">Klik for profil & kampe</div>`;
      left.append(av, txt);

      const open = document.createElement('a');
      open.className = 'btn';
      open.textContent = 'Open';
      const u = new URL(location.href);
      u.searchParams.set('u', i);
      open.href = u.toString();

      row.append(left, open);
      listEl.appendChild(row);
    });
  }
  renderProfiles();

  // Profil-detaljevisning
  function getParam(name){
    const url = new URL(location.href);
    return url.searchParams.get(name);
  }
  const uParam = up6(getParam('u') || '');
  if (uParam){
    const detailCard = byId('profileDetail');
    detailCard.style.display = 'grid';
    byId('detailTitle').textContent  = `Profile: ${uParam}`;
    byId('detailAvatar').textContent = uParam;
    byId('detailInfo').textContent   = 'Kampe hvor profilen er tagget (p1 eller p2).';

    const all = load(KEY_MATCHES, []);
    const mine = all.filter(m => up6(m.p1) === uParam || up6(m.p2) === uParam);

    const list = byId('detailMatches');
    list.innerHTML = '';

    if (!mine.length){
      const li = document.createElement('li');
      li.className = 'item';
      li.innerHTML = '<span class="meta">Ingen kampe endnu.</span>';
      list.appendChild(li);
    } else {
      mine.forEach(m=>{
        const li = document.createElement('li'); li.className='item';
        const left = document.createElement('div');
        const wtxt = m.winner === 'p1' ? up6(m.p1) : (m.winner === 'p2' ? up6(m.p2) : '—');
        left.innerHTML = `
          <div><strong>${up6(m.p1)}</strong> vs <strong>${up6(m.p2)}</strong></div>
          <div class="meta">${m.when || ''}</div>
          <div class="meta">Score: ${m.score || '—'} • Winner: ${wtxt} • Bet: ${m.bet || '—'}</div>
        `;
        li.append(left);
        list.appendChild(li);
      });
    }
  }
})();
</script>
