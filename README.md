<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <title>경마 게임</title>
  <style>
    body { font-family: sans-serif; padding: 20px; }
    .track {
      position: relative;
      width: 100%;
      height: 200px;
      border: 2px solid #000;
      margin-bottom: 20px;
    }
    .horse {
      position: absolute;
      width: 60px;
      height: 40px;
      background-color: lightblue;
      border: 2px solid #333;
      border-radius: 5px;
      text-align: center;
      line-height: 40px;
    }
    .horse:nth-child(1) { top: 20px; background-color: #f77; }
    .horse:nth-child(2) { top: 80px; background-color: #7f7; }
    .horse:nth-child(3) { top: 140px; background-color: #77f; }
  </style>
</head>
<body>

  <h1>경마 게임</h1>
  <button onclick="startRace()">시작</button>

  <div class="track" id="track">
    <div class="horse" id="horse1">1번</div>
    <div class="horse" id="horse2">2번</div>
    <div class="horse" id="horse3">3번</div>
  </div>

  <div id="result"></div>
  <div id="stats"></div>

  <script>
    const horses = [
      { id: 'horse1', name: '1번 말', pos: 0, wins: 0 },
      { id: 'horse2', name: '2번 말', pos: 0, wins: 0 },
      { id: 'horse3', name: '3번 말', pos: 0, wins: 0 }
    ];

    const finishLine = 900;
    let raceInterval = null;
    let running = false;

    function loadStats() {
      horses.forEach(horse => {
        const storedWins = localStorage.getItem(horse.id);
        if (storedWins) horse.wins = parseInt(storedWins);
      });
    }

    function saveStats() {
      horses.forEach(horse => {
        localStorage.setItem(horse.id, horse.wins);
      });
    }

    function updatePositions() {
      horses.forEach(horse => {
        const move = Math.random() * 10 + 1; // 1 ~ 11
        horse.pos += move;
        document.getElementById(horse.id).style.left = horse.pos + 'px';
      });
    }

    function checkWinner() {
      for (let horse of horses) {
        if (horse.pos >= finishLine) {
          running = false;
          clearInterval(raceInterval);
          horse.wins++;
          saveStats();
          showResult(horse.name);
          return;
        }
      }
    }

    function showResult(winnerName) {
      document.getElementById('result').innerHTML = `<h2>우승: ${winnerName}</h2>`;
      let totalWins = horses.reduce((acc, h) => acc + h.wins, 0);
      let statsHTML = '<h3>승률</h3><ul>';
      horses.forEach(h => {
        const percent = totalWins ? ((h.wins / totalWins) * 100).toFixed(1) : 0;
        statsHTML += `<li>${h.name}: ${h.wins}승 (${percent}%)</li>`;
      });
      statsHTML += '</ul>';
      document.getElementById('stats').innerHTML = statsHTML;
    }

    function resetRace() {
      horses.forEach(horse => {
        horse.pos = 0;
        document.getElementById(horse.id).style.left = '0px';
      });
      document.getElementById('result').innerHTML = '';
    }

    function startRace() {
      if (running) return;
      resetRace();
      loadStats();
      running = true;
      raceInterval = setInterval(() => {
        updatePositions();
        checkWinner();
      }, 50);
    }
  </script>

</body>
</html>
