---
title: Matches
---

<div class="card">
  <h1>Create Match</h1>
  <div class="form-row" style="margin-top:8px;">
    <input class="input" id="p1" placeholder="Player 1 (fx dine initialer)" required>
    <input class="input" id="p2" placeholder="Player 2 (kollega)" required>
    <input class="input" id="when" type="date" aria-label="Date">
    <button class="btn" id="add">Add</button>
  </div>
  <hr class="sep">
  <h2>Upcoming & Past</h2>
  <ul class="list" id="list"></ul>
</div>

<script>
  (function(){
    const KEY = 'matches.items';
    const listEl = document.getElementById('list');
    const p1 = document.getElementById('p1');
    const p2 = document.getElementById('p2');
    const when = document.getElementById('when');
    const add = document.getElementById('add');

    const load = () => JSON.parse(localStorage.getItem(KEY) || '[]');
    const save = (arr) => localStorage.setItem(KEY, JSON.stringify(arr));

    function render(){
      const items = load();
      listEl.innerHTML = '';
      if (!items.length){
        const empty = document.createElement('p');
        empty.className = 'meta';
        empty.textContent = 'Ingen matches endnu. Tilføj din første ovenfor.';
        listEl.appendChild(empty);
        return;
      }
      items.forEach((m, idx)=>{
        const li = document.createElement('li'); li.className = 'item';
        const left = document.createElement('div');
        left.innerHTML = `<strong>${m.p1}</strong> vs <strong>${m.p2}</strong><div class="meta">${m.when || 'No date'}</div>`;
        const right = document.createElement('div');
        const del = document.createElement('button'); del.className = 'btn ghost'; del.textContent = 'Delete';
        del.addEventListener('click', ()=>{
          const arr = load(); arr.splice(idx,1); save(arr); render();
        });
        right.appendChild(del);
        li.append(left,right);
        listEl.appendChild(li);
      });
    }

    add.addEventListener('click', ()=>{
      const a = (p1.value || '').trim().toUpperCase();
      const b = (p2.value || '').trim().toUpperCase();
      if (!a || !b) return;
      const item = { p1:a, p2:b, when: when.value || '' };
      const arr = load(); arr.unshift(item); save(arr); render();
      p1.value=''; p2.value=''; /* keep date */
    });

    // Prefill Player 1 with saved initials
    const initials = localStorage.getItem('profile.initials');
    if (initials) p1.value = initials;

    render();
  })();
</script>
