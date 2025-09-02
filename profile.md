---
title: Profile
---

<div class="card" style="display:grid; gap:14px; align-items:center; justify-items:start;">
  <h1>Your Profile</h1>
  <div style="display:flex; align-items:center; gap:14px;">
    <div class="avatar" id="avatar">??</div>
    <div>
      <h2 id="initialsLabel">Initials</h2>
      <div class="form-row">
        <input class="input" id="initialsInput" placeholder="Fx JB" maxlength="4">
        <button class="btn" id="saveInitials">Save</button>
        <button class="btn ghost" id="clearInitials">Clear</button>
      </div>
      <p class="meta" style="margin-top:8px;">Initialer gemmes i din browser (localStorage).</p>
    </div>
  </div>
</div>

<script>
  (function(){
    const KEY = 'profile.initials';
    const input = document.getElementById('initialsInput');
    const avatar = document.getElementById('avatar');
    const label = document.getElementById('initialsLabel');

    function render(val){
      const txt = (val || '??').toUpperCase().slice(0,4);
      avatar.textContent = txt || '??';
      label.textContent = txt ? `Initials: ${txt}` : 'Initials';
    }

    const saved = localStorage.getItem(KEY);
    if (saved) { input.value = saved; render(saved); } else { render(null); }

    document.getElementById('saveInitials').addEventListener('click', ()=>{
      const v = (input.value || '').trim().toUpperCase();
      if (!v) return;
      localStorage.setItem(KEY, v);
      render(v);
    });
    document.getElementById('clearInitials').addEventListener('click', ()=>{
      localStorage.removeItem(KEY);
      input.value = '';
      render(null);
    });
  })();
</script>
