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

<script>
(function(){
  const KEY_PROFILES = 'profiles.list'; // [{ i: "ABC123" }]
  const KEY_ME       = 'profile.initials'; // valgfrit: brug første som "min"
  const up6  = s => (s||'').toUpperCase().slice(0,6).replace(/[^A-Z0-9]/g,'');
  const load = (k, fb) => JSON.parse(localStorage.getItem(k) || JSON.stringify(fb));
  const save = (k, v) => localStorage.setItem(k, JSON.stringify(v));

  // Helper: tilpas avatar-fontstørrelse efter længde (2–6)
  function fitAvatar(el, text){
    const len = (text||'').length;
    let size = 28;           // default
    if (len >= 6) size = 16; // 6 tegn
    else if (len === 5) size = 18;
    else if (len === 4) size = 20;
    else if (len === 3) size = 22;
    else if (len <= 2) size = 28;
    el.style.fontSize = size + 'px';
  }

  // Opret profil (ét sted)
  const newProfile = document.getElementById('newProfile');
  document.getElementById('addProfile').addEventListener('click', ()=>{
    const v = up6(newProfile.value);
    if (!v) return;
    const arr = load(KEY_PROFILES, []);
    if (!arr.some(p=>p.i===v)) {
      arr.unshift({ i: v });
      save(KEY_PROFILES, arr);
      if (!localStorage.getItem(KEY_ME)) localStorage.setItem(KEY_ME, v);
      renderProfiles();
    }
    // reset & ryd feltet
    newProfile.value = '';
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

      // Open-knap: ny side (profile-view.html?u=XXXX)
      const open = document.createElement('a');
      open.className = 'btn';
      open.textContent = 'Open';
      open.href = "{{ '/profile-view.html' | relative_url }}" + "?u=" + encodeURIComponent(i);

      // Delete-knap
      const del = document.createElement('button');
      del.className = 'btn ghost';
      del.textContent = 'Delete';
      del.addEventListener('click', ()=>{
        const next = load(KEY_PROFILES, []).filter(p => p.i !== i);
        save(KEY_PROFILES, next);
        // hvis KEY_ME var denne profil, ryd den
        if (localStorage.getItem(KEY_ME) === i) localStorage.removeItem(KEY_ME);
        renderProfiles();
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
})();
</script>
