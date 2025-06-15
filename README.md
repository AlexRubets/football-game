<!DOCTYPE html>
<html lang="uk">
<head>
  <meta charset="UTF-8">
  <title>Футбольна гра: УПЛ + Маяк Ромни</title>
  <style>
    body {
      font-family: 'Arial';
      background: #f0f8ff;
      text-align: center;
      padding: 20px;
    }
    select, button {
      padding: 10px;
      font-size: 16px;
      margin: 10px;
    }
    #leagueTable {
      width: 95%;
      margin: auto;
      border-collapse: collapse;
    }
    #leagueTable th, #leagueTable td {
      padding: 8px;
      border: 1px solid #000;
    }
    #goalAnimation {
      font-size: 40px;
      color: red;
      animation: none;
    }
    @keyframes goalEffect {
      0% { transform: scale(1); opacity: 1; }
      50% { transform: scale(1.5); opacity: 0.5; }
      100% { transform: scale(1); opacity: 1; }
    }
    #fanZone {
      font-size: 28px;
      color: green;
      display: none;
      margin-bottom: 20px;
    }
  </style>
</head>
<body>
  <h1>⚽ УПЛ + Маяк Ромни</h1>

  <div id="matchArea">
    <select id="team1"></select>
    <select id="team2"></select>
    <button onclick="playMatch()">Зіграти матч</button>
    <h2 id="matchResult">Результат буде тут</h2>
    <div id="goalAnimation"></div>
  </div>

  <div id="fanZone">🎉 Ура! Гол! 🎉</div>

  <h2>Турнірна таблиця</h2>
  <table id="leagueTable">
    <thead>
      <tr><th>Команда</th><th>І</th><th>В</th><th>Н</th><th>П</th><th>ЗМ:ПМ</th><th>Очки</th></tr>
    </thead>
    <tbody></tbody>
  </table>

  <h2>Календар матчів</h2>
  <ul id="calendarList"></ul>

  <script>
    const teams = [
      "Динамо", "Шахтар", "Кривбас", "Полісся", "Олександрія",
      "Зоря", "Дніпро-1", "Ворскла", "Оболонь", "Чорноморець",
      "Металіст 1925", "Рух", "Колос", "Верес", "Інгулець",
      "Лівий Берег", "ЛНЗ", "Маяк Ромни"
    ];

    let standings = {};
    teams.forEach(t => standings[t] = { W: 0, D: 0, L: 0, GF: 0, GA: 0, P: 0 });

    const calendar = [];
    for (let i = 0; i < teams.length; i++) {
      for (let j = i + 1; j < teams.length; j++) {
        calendar.push({ home: teams[i], away: teams[j] });
      }
    }
    calendar.sort(() => Math.random() - 0.5);

    const team1Select = document.getElementById("team1");
    const team2Select = document.getElementById("team2");
    teams.forEach(t => {
      team1Select.add(new Option(t, t));
      team2Select.add(new Option(t, t));
    });

    function playMatch() {
      const team1 = team1Select.value;
      const team2 = team2Select.value;
      if (team1 === team2) {
        alert("Оберіть різні команди!");
        return;
      }

      const score1 = Math.floor(Math.random() * 5);
      const score2 = Math.floor(Math.random() * 5);

      animateGoal(score1 + score2);
      showFans(team1);

      document.getElementById("matchResult").textContent = `${team1} ${score1} : ${score2} ${team2}`;

      updateStandings(team1, team2, score1, score2);
      renderTable();
    }

    function animateGoal(count) {
      const anim = document.getElementById("goalAnimation");
      anim.innerHTML = "⚽".repeat(count);
      anim.style.animation = "goalEffect 1s infinite";
      setTimeout(() => anim.style.animation = "none", 2000);
    }

    function showFans(goalTeam) {
      const fans = document.getElementById("fanZone");
      fans.textContent = `🎉 Гол за ${goalTeam}! 🎉`;
      fans.style.display = "block";
      setTimeout(() => fans.style.display = "none", 2000);
    }

    function updateStandings(t1, t2, s1, s2) {
      standings[t1].GF += s1;
      standings[t1].GA += s2;
      standings[t2].GF += s2;
      standings[t2].GA += s1;

      if (s1 > s2) {
        standings[t1].W++;
        standings[t2].L++;
        standings[t1].P += 3;
      } else if (s1 < s2) {
        standings[t2].W++;
        standings[t1].L++;
        standings[t2].P += 3;
      } else {
        standings[t1].D++;
        standings[t2].D++;
        standings[t1].P++;
        standings[t2].P++;
      }
    }

    function renderTable() {
      const tbody = document.querySelector("#leagueTable tbody");
      tbody.innerHTML = "";

      const sorted = Object.entries(standings).sort((a, b) => b[1].P - a[1].P);
      sorted.forEach(([team, s]) => {
        const row = document.createElement("tr");
        row.innerHTML = `<td>${team}</td>
          <td>${s.W + s.D + s.L}</td>
          <td>${s.W}</td>
          <td>${s.D}</td>
          <td>${s.L}</td>
          <td>${s.GF}:${s.GA}</td>
          <td>${s.P}</td>`;
        tbody.appendChild(row);
      });
    }

    function renderCalendar() {
      const list = document.getElementById("calendarList");
      calendar.forEach(match => {
        const li = document.createElement("li");
        li.textContent = `${match.home} vs ${match.away}`;
        list.appendChild(li);
      });
    }

    renderTable();
    renderCalendar();
  </script>
</body>
</html>
