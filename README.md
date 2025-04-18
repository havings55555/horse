<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <title>경마 게임 - 밸런스 보정</title>
  <style>
    body { font-family: sans-serif; padding: 20px; }

    .track {
      position: relative;
      width: 2000px;
      height: 260px;
      border: 3px solid #000;
      margin-bottom: 20px;
      background-color: #f0f0f0;
      overflow: hidden;
    }

    .horse {
      position: absolute;
      width: 200px;
      height: 40px;
      color: #fff;
      font-weight: bold;
      text-align: center;
      line-height: 40px;
      border-radius: 5px;
    }

    #horse1 { top: 30px; background-color: #e74c3c; }
    #horse2 { top: 110px; background-color: #27ae60; }
    #horse3 { top: 190px; background-color: #2980b9; }
  </style>
</head>
<body>

  <h1>경마 게임 - 밸런스 보정</h1>
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

    const horseWidth = 200;
    const trackWidth = 2000;
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
      horses[0].speed = Math.floor(Math.random() * 251) + 600;  // 600~850
      horses[1].speed = Math.floor(Math.random() * 601) + 400;  // 400~1000

      // 3번 말: 40% 빠름
      if (Math.random() < 0.5) {
        horses[2].speed = Math.floor(Math.random() * 201) + 1000; // 800~1000
      } else {
        horses[2].speed = Math.floor(Math.random() * 101) + 50;  // 50~150
      }
    }

    function moveHorses(timestamp) {
      if (!lastFrame) lastFrame = timestamp;
      const delta = (timestamp - lastFrame) / 1000;
      lastFrame = timestamp;

      horses.forEach(horse => {
        horse.pos += horse.speed * delta;
        document.getElementById(horse.id).style.left = horse.pos + 'px';
      });

      checkWinner();

      if (running) {
        animationId = requestAnimationFrame(moveHorses);
      }
    }

    function checkWinner() {
      for (let horse of horses) {
        if (horse.pos + horseWidth >= trackWidth) {
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
        document.getElementById(horse.id).style.left = '0px';
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
