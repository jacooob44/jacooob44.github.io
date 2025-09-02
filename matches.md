---
title: Matches
---

# Matches
<form id="matchForm">
  <input type="text" id="player1" placeholder="Player 1 Initials" required>
  <input type="text" id="player2" placeholder="Player 2 Initials" required>
  <button type="submit">Add Match</button>
</form>

<ul id="matchesList"></ul>

<script>
  const form = document.getElementById('matchForm');
  const list = document.getElementById('matchesList');

  form.addEventListener('submit', function(e) {
    e.preventDefault();
    const p1 = document.getElementById('player1').value;
    const p2 = document.getElementById('player2').value;

    const li = document.createElement('li');
    li.textContent = `${p1} vs ${p2}`;
    list.appendChild(li);

    form.reset();
  });
</script>
