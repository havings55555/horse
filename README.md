<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <title>세로 경마 게임</title>
  <style>
    body { font-family: sans-serif; padding: 20px; }

    .track {
      position: relative;
      width: 300px;
      height: 2000px; /* 세로 길이 */
      border: 3px solid #000;
      margin-bottom: 20px;
      background-color: #f0f0f0;
      overflow: hidden;
    }

    .horse {
      position: absolute;
      width: 90px;
      height: 200px;
      color: #fff;
      font-weight: bold;
      text-align: center;
      writing-mode: vertical-rl;
      line-height: 90px;
      border-radius: 8px;
    }

    #horse1 { left: 10px; background-color: #e74c3c; }
    #horse2 { left: 110px; background-color: #27ae60; }
    #horse3 { left: 210px; background-color: #2980b9; }
  </style>
</head>
<body>

  <h1>세로 경마 게임</h1>
  <button onclick="startRace()">시작</button>
  <button onclick="resetStats()">승률 초기화</button>

  <div class="track" id="track">
    <div class="horse" id="horse1">1번 말</div>
    <div class="horse" id="horse2">2번 말</div>
    <div class="horse" id="horse3">3번 말</div>
  </div>

  <div id="result"></div>
  <div id="stats"></div>

  <script>
    const horses = [
      { id: 'horse1', name: '1번 말', pos: 0, wins: 0, speed: 0 },
      { id: 'horse2', name: '2번 말', pos: 0, wins: 0, speed: 0 },
      { id: 'horse3', name: '3번 말', pos: 0, wins: 0, speed: 0 }
    ];

    const horseHeight = 200;
    const trackHeight = 2000;
    let raceInterval = null;
    let animationId = null;
    let lastFrame = null;
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

    function resetStats() {
      horses.forEach(horse => {
        horse.wins = 0;
        localStorage.removeItem(horse.id);
      });
      updateStatsUI();
    }

    function updateStatsUI() {
      let totalWins = horses.reduce((acc, h) => acc + h.wins, 0);
      let statsHTML = '<h3>승률</h3><ul>';
      horses.forEach(h => {
        const percent = totalWins ? ((h.wins / totalWins) * 100).toFixed(1) : 0;
        statsHTML += `<li>${h.name}: ${h.wins}승 (${percent}%)</li>`;
      });
      statsHTML += '</ul>';
      document.getElementById('stats').innerHTML = statsHTML;
    }

    function updateSpeeds() {
      horses[0].speed = Math.floor(Math.random() * 501) + 800;  // 800~1300
      horses[1].speed = Math.floor(Math.random() * 601) + 200;  // 200~800
      if (Math.random() < 0.2) {
        horses[2].speed = Math.floor(Math.random() * 201) + 800; // 800~1000
      } else {
        horses[2].speed = Math.floor(Math.random() * 151) + 50;  // 50~200
      }
    }

    function moveHorses(timestamp) {
      if (!lastFrame) lastFrame = timestamp;
      const delta = (timestamp - lastFrame) / 1000;
      lastFrame = timestamp;

      horses.forEach(horse => {
        horse.pos += horse.speed * delta;
        document.getElementById(horse.id).style.top = horse.pos + 'px';
      });

      checkWinner();

      if (running) {
        animationId = requestAnimationFrame(moveHorses);
      }
    }

    function checkWinner() {
      for (let horse of horses) {
        if (horse.pos + horseHeight >= trackHeight) {
          running = false;
          cancelAnimationFrame(animationId);
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
      updateStatsUI();
    }

    function resetRace() {
      horses.forEach(horse => {
        horse.pos = 0;
        horse.speed = 0;
        document.getElementById(horse.id).style.top = '0px';
      });
      document.getElementById('result').innerHTML = '';
      lastFrame = null;
    }

    function startRace() {
      if (running) return;
      resetRace();
      loadStats();
      updateStatsUI();
      running = true;

      updateSpeeds();
      raceInterval = setInterval(updateSpeeds, 200);
      animationId = requestAnimationFrame(moveHorses);
    }
  </script>

</body>
</html>
