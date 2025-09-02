---
title: Matches
---

<div class="card">
  <h1>Create Match</h1>
  <div class="form-row" style="margin-top:8px;">
    <input class="input" id="p1" placeholder="Player 1 (fx dine initialer)" required>
    <input class="input" id="p2" placeholder="Player 2 (kollega)" required>
    <input class="input" id="when" type="date" aria-label="Date">
    <button class="btn" id="add" type="button">Add</button>
  </div>
  <hr style="border:0; height:1px; background: var(--border); margin:16px 0;">
  <h2>Upcoming & Past</h2>
  <ul class="list" id="list"></ul>
</div>

<script>
(function(){
  const KEY = 'matches.items';
  const KEY_ME = 'profile.initials';
  const listEl = document.getElementById('list');
  const p1 = document.getElementById('p1');
  const p2 = document.getElementById('p2');
  const when = document.getElementById('when');
  const add = document.getElementById('add');
  const up6 = (s)=> (s||'').toUpperCase().slice(0,6).replace(/[^A-Z0-9]/g,'');

  const load = () => JSON.parse(localStorage.getItem(KEY) || '[]');
  const save = (arr) => localStorage.setItem(KEY, JSON.stringify(arr));

  function render(){
    const items = load();
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
      left.innerHTML = `<strong>${up6(m.p1)}</strong> vs <strong>${up6(m.p2)}</strong><div class="meta">${m.when || 'No date'}</div>`;
      const del = document.createElement('button'); del.className = 'btn ghost'; del.textContent = 'Delete';
      del.addEventListener('click', ()=>{
        const arr = load(); arr.splice(idx,1); save(arr); render();
      });
      li.append(left, del);
      listEl.appendChild(li);
    });
  }

  add.addEventListener('click', ()=>{
    const a = up6(p1.value);
    const b = up6(p2.value);
    if (!a || !b) return;
    const item = { p1:a, p2:b, when: when.value || '' };
    const arr = load(); arr.unshift(item); save(arr); render();
    p1.value=''; p2.value='';
  });

  // Prefill Player 1 med dine gemte initialer (op til 6)
  const me = localStorage.getItem(KEY_ME);
  if (me) p1.value = me.toUpperCase().slice(0,6);

  render();
})();
</script>
