---
title: Profile
---

<div class="card" style="display:grid; gap:14px;">
  <h1>Profiles</h1>

  <!-- Din egen profil -->
  <div style="display:flex; align-items:center; gap:14px; flex-wrap:wrap;">
    <div class="avatar" id="avatar">??</div>
    <div>
      <h2 id="initialsLabel">Your initials</h2>
      <div class="form-row">
        <input class="input" id="initialsInput" placeholder="Fx JACOB1" maxlength="6">
        <button class="btn" id="saveInitials" type="button">Save</button>
        <button class="btn ghost" id="clearInitials" type="button">Clear</button>
      </div>
      <p class="meta" style="margin-top:8px;">Gemmes i din browser (localStorage). Op til 6 tegn (A–Z, 0–9).</p>
    </div>
  </div>

  <hr class="sep">

  <!-- Tilføj kollega-profil -->
  <h2>Add colleague</h2>
  <div class="form-row">
    <input class="input" id="newProfile" placeholder="Initials (fx MKR123)" maxlength="6">
    <button class="btn" id="addProfile" type="button">Add profile</button>
  </div>

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
  const KEY_ME = 'profile.initials';    // din egen profil (forfylder p1 på Matches)
  const KEY_PROFILES = 'profiles.list'; // [{ i: "ABC123" }]
  const KEY_MATCHES = 'matches.items';  // [{ p1, p2, when }]
  const qs = sel => document.querySelector(sel);
  const byId = id => document.getElementById(id);
  const load = (k, fb) => JSON.parse(localStorage.getItem(k) || JSON.stringify(fb));
  const save = (k, v) => localStorage.setItem(k, JSON.stringify(v));
  const up6 = s => (s||'').toUpperCase().slice(0,6).replace(/[^A-Z0-9]/g,'');

  // --- Din egen profil (øverst) ---
  const input  = byId('initialsInput');
  const avatar = byId('avatar');
  const label  = byId('initialsLabel');

  function renderMe(val){
    const txt = up6(val) || '??';
    avatar.textContent = txt;
    label.textContent = txt !== '??' ? `Your initials: ${txt}` : 'Your initials';
  }

  const savedMe = localStorage.getItem(KEY_ME);
  if (savedMe) { input.value = savedMe; renderMe(savedMe); } else { renderMe(null); }

  byId('saveInitials').addEventListener('click', ()=>{
    const v = up6(input.value);
    if (!v) return;
    localStorage.setItem(KEY_ME, v);
    renderMe(v);
    // ensure din egen profil findes i listen
    const arr = load(KEY_PROFILES, []);
    if (!arr.some(p => p.i === v)) { arr.unshift({ i: v }); save(KEY_PROFILES, arr); }
    renderProfiles();
  });

  byId('clearInitials').addEventListener('click', ()=>{
    localStorage.removeItem(KEY_ME);
    input.value = '';
    renderMe(null);
  });

  // --- Tilføj kollega-profil ---
  const newProfile = byId('newProfile');
  byId('addProfile').addEventListener('click', ()=>{
    const v = up6(newProfile.value);
    if (!v) return;
    const arr = load(KEY_PROFILES, []);
    if (!arr.some(p => p.i === v)) {
      arr.unshift({ i: v });
      save(KEY_PROFILES, arr);
      renderProfiles();
      newProfile.value = '';
    }
  });

  // --- Liste over alle profiler (div container, så vi må bruge div.item) ---
  const listEl = byId('profilesList');
  function renderProfiles(){
    const arr = load(KEY_PROFILES, []);
    listEl.innerHTML = '';
    if (!arr.length){
      const empty = document.createElement('div');
      empty.className = 'item';
      empty.innerHTML = '<span class="meta">Ingen profiler endnu. Tilføj ovenfor.</span>';
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
      // behold nuværende sti, tilføj query param ?u=INITIALS
      const u = new URL(location.href);
      u.searchParams.set('u', i);
      open.href = u.toString();

      row.append(left, open);
      listEl.appendChild(row);
    });
  }
  renderProfiles();

  // --- Profil-detaljevisning via ?u=XXXX ---
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
    byId('detailInfo').textContent   = 'Viser alle kampe hvor profilen er tagget (p1 eller p2).';

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
        left.innerHTML = `<strong>${up6(m.p1)}</strong> vs <strong>${up6(m.p2)}</strong><div class="meta">${m.when || 'No date'}</div>`;
        li.append(left);
        list.appendChild(li);
      });
    }
  }
})();
</script>
