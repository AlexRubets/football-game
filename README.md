<!DOCTYPE html>
<html lang="uk">
<head>
  <meta charset="UTF-8" />
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
    #explanation {
      margin-bottom: 20px;
      font-weight: bold;
      font-size: 18px;
      color: #333;
    }
    #champion {
      font-size: 24px;
      color: blue;
      margin-top: 20px;
      font-weight: bold;
    }
  </style>
</head>
<body>
  <h1>⚽ УПЛ + Маяк Ромни</h1>

  <div id="explanation">
    Гра триває у два кола: домашній та виїзний матч проти кожного суперника.<br>
    За перемогу – 3 бали, за нічию – 1 бал.<br>
    Натисни «Автосимуляція туру», щоб зіграти всі матчі туру одразу.
  </div>

  <div id="matchArea">
    <select id="team1"></select>
    <select id="team2"></select>
    <button onclick="playMatch()">Зіграти матч</button>
    <button onclick="autoSimulateRound()">Автосимуляція туру</button>
    <h2 id="matchResult">Результат буде тут</h2>
    <div id="goalAnimation"></div>
    <div id="goalScorers"></div>
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

  <div id="champion"></div>
  <button onclick="resetSeason()">Очистити сезон</button>


<script>
  // Команди з прикладами складів (короткі для прикладу)
  const squads = {
    "Динамо": ["Бущан", "Кендзьора", "Шапаренко", "Циганков", "Ванат"],
    "Шахтар": ["Різник", "Бондар", "Матвієнко", "Судаков", "Зубков"],
    "Кривбас": ["Слісенко", "Квеквескірі", "Павлюк", "Кожушко", "Банада"],
    "Полісся": ["Іванюк", "Петров", "Голенко", "Сухомлин", "Бондаренко"],
    "Олександрія": ["Леоненко", "Ткачук", "Демченко", "Захарченко", "Іванов"],
    "Зоря": ["Пастух", "Стеценко", "Лисенко", "Мельник", "Петренко"],
    "Дніпро-1": ["Рябко", "Пархоменко", "Дорошенко", "Кравченко", "Мельник"],
    "Ворскла": ["Коваль", "Гришко", "Бондаренко", "Ісаєнко", "Панасенко"],
    "Оболонь": ["Марченко", "Данилюк", "Гнатюк", "Федоренко", "Прокопенко"],
    "Чорноморець": ["Іваненко", "Грищук", "Коваленко", "Лебідь", "Семенюк"],
    "Металіст 1925": ["Шевченко", "Дудник", "Козаченко", "Михайленко", "Романюк"],
    "Рух": ["Кириленко", "Мельник", "Кравченко", "Костенко", "Олійник"],
    "Колос": ["Демченко", "Захарченко", "Ковальчук", "Петренко", "Сидоренко"],
    "Верес": ["Грицай", "Коваленко", "Сергієнко", "Пархоменко", "Данилюк"],
    "Інгулець": ["Коваль", "Павленко", "Іванов", "Савченко", "Костенко"],
    "Лівий Берег": ["Петренко", "Гончарук", "Кириленко", "Романюк", "Кравченко"],
    "ЛНЗ": ["Шевченко", "Гнатюк", "Козаченко", "Ісаєнко", "Романенко"],
    "Маяк Ромни": [
      "Дмитро Лісний", "Іван Рябенький", "Євген Мурайкін", "Євген Логвин", "Сергій Гринько",
      "Денис Гальченко", "Владислав Ільїн", "Дмитро Лісовий", "Євген Школяренко", "Олександр Рубець", "Віктор Даценко"
    ]
  };

  const teams = Object.keys(squads);

  // Ініціалізація таблиці
  let standings = {};
  teams.forEach(t => standings[t] = { W:0, D:0, L:0, GF:0, GA:0, P:0, played: 0 });

  // Генерація календаря (два кола)
  let calendar = [];
  for(let i=0; i<teams.length; i++) {
    for(let j=0; j<teams.length; j++) {
      if(i !== j) {
        calendar.push({home: teams[i], away: teams[j], played: false});
      }
    }
  }
  shuffleArray(calendar);

  // Календар для показу (по турах)
  const matchesPerRound = teams.length / 2;
  const roundsCount = calendar.length / matchesPerRound;

  // Заповнюємо селекти для вибору вручну
  const team1Select = document.getElementById('team1');
  const team2Select = document.getElementById('team2');
  teams.forEach(t => {
    team1Select.add(new Option(t, t));
    team2Select.add(new Option(t, t));
  });

  // Поточний тур для автосимуляції
  let currentRound = 0;

  // Функції
  function playMatch(homeTeam, awayTeam, manual=true) {
    if(manual) {
      homeTeam = team1Select.value;
      awayTeam = team2Select.value;
      if(homeTeam === awayTeam) {
        alert('Оберіть різні команди!');
        return;
      }
    }

    // Голі - випадкові з складу
    const homeGoals = Math.floor(Math.random()*5);
    const awayGoals = Math.floor(Math.random()*5);

    // Автори голів
    const homeScorers = generateScorers(homeTeam, homeGoals);
    const awayScorers = generateScorers(awayTeam, awayGoals);

    animateGoal(homeGoals + awayGoals);
    showFans(homeTeam);

    const resText = `${homeTeam} ${homeGoals} : ${awayGoals} ${awayTeam}`;
    document.getElementById('matchResult').textContent = resText;
    document.getElementById('goalScorers').innerHTML =
      `<b>Голи:</b><br>` +
      `<span style="color:blue">${homeTeam}:</span> ${homeScorers.join(', ') || 'Немає'}<br>` +
      `<span style="color:red">${awayTeam}:</span> ${awayScorers.join(', ') || 'Немає'}`;

    updateStandings(homeTeam, awayTeam, homeGoals, awayGoals);

    if(manual) renderTable();

    // Відмічаємо в календарі, якщо автосимуляція
    if(!manual) {
      const matchIndex = calendar.findIndex(m => m.home === homeTeam && m.away === awayTeam && !m.played);
      if(matchIndex !== -1) calendar[matchIndex].played = true;
    }
  }

  function generateScorers(team, goals) {
    const players = squads[team];
    let scorers = [];
    for(let i=0; i<goals; i++) {
      scorers.push(players[Math.floor(Math.random()*players.length)]);
    }
    return scorers;
  }

  function animateGoal(count) {
    const anim = document.getElementById('goalAnimation');
    anim.innerHTML = '⚽'.repeat(count);
    anim.style.animation = 'goalEffect 1s infinite';
    setTimeout(() => anim.style.animation = 'none', 2000);
  }

  function showFans(goalTeam) {
    const fans = document.getElementById('fanZone');
    fans.textContent = `🎉 Гол за ${goalTeam}! 🎉`;
    fans.style.display = 'block';
    setTimeout(() => fans.style.display = 'none', 2000);
  }

  function updateStandings(t1, t2, s1, s2) {
    standings[t1].GF += s1;
    standings[t1].GA += s2;
    standings[t2].GF += s2;
    standings[t2].GA += s1;

    standings[t1].played++;
    standings[t2].played++;

    if(s1 > s2) {
      standings[t1].W++;
      standings[t2].L++;
      standings[t1].P += 3;
    } else if(s1 < s2) {
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
    const tbody = document.querySelector('#leagueTable tbody');
    tbody.innerHTML = '';
    const sorted = Object.entries(standings).sort((a,b) => b[1].P - a[1].P || (b[1].GF - b[1].GA) - (a[1].GF - a[1].GA));
    sorted.forEach(([team, s]) => {
      const row = document.createElement('tr');
      row.innerHTML = `<td>${team}</td>
        <td>${s.played}</td>
        <td>${s.W}</td>
        <td>${s.D}</td>
        <td>${s.L}</td>
        <td>${s.GF}:${s.GA}</td>
        <td>${s.P}</td>`;
      tbody.appendChild(row);
    });
  }

  function renderCalendar() {
    const list = document.getElementById('calendarList');
    list.innerHTML = '';
    // Відобразити матчі за поточний тур
    const matchesThisRound = calendar.slice(currentRound*matchesPerRound, (currentRound+1)*matchesPerRound);
    matchesThisRound.forEach(m => {
      const li = document.createElement('li');
      li.textContent = `${m.home} vs ${m.away}` + (m.played ? ' ✅' : '');
      list.appendChild(li);
    });
  }

  function shuffleArray(array) {
    for(let i = array.length - 1; i > 0; i--) {
      const j = Math.floor(Math.random() * (i + 1));
      [array[i], array[j]] = [array[j], array[i]];
    }
  }

  // Автосимуляція туру
  function autoSimulateRound() {
    if(currentRound >= roundsCount) {
      alert('Сезон завершено! Переглянь таблицю нижче.');
      showChampion();
      return;
    }
    const matchesThisRound = calendar.slice(currentRound*matchesPerRound, (currentRound+1)*matchesPerRound);
    for(const m of matchesThisRound) {
      playMatch(m.home, m.away, false);
    }
    currentRound++;
    renderCalendar();
    renderTable();
    if(currentRound === roundsCount) showChampion();
  }

  // Показати чемпіона сезону
  function showChampion() {
    const champDiv = document.getElementById('champion');
    const sorted = Object.entries(standings).sort((a,b) => b[1].P - a[1].P || (b[1].GF - b[1].GA) - (a[1].GF - a[1].GA));
    const champ = sorted[0][0];
    champDiv.textContent = `🏆 Чемпіон сезону: ${champ} 🏆`;
  }

  // Початковий рендер
  renderTable();
  renderCalendar();

</script>
</body>
</html>
