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
function saveGameState() {
  localStorage.setItem('leagueStandings', JSON.stringify(standings));
  localStorage.setItem('calendarData', JSON.stringify(calendar));
  localStorage.setItem('currentRound', currentRound);
}

function loadGameState() {
  const savedTable = localStorage.getItem('leagueStandings');
  const savedCalendar = localStorage.getItem('calendarData');
  const savedRound = localStorage.getItem('currentRound');
  if (savedTable && savedCalendar && savedRound !== null) {
    standings = JSON.parse(savedTable);
    calendar = JSON.parse(savedCalendar);
    currentRound = parseInt(savedRound);
    renderTable();
    renderCalendar();
    if (currentRound >= roundsCount) showChampion();
  }
}

function resetSeason() {
  if (confirm("Ви впевнені, що хочете очистити сезон?")) {
    localStorage.clear();
    location.reload();
  }
}
saveGameState();
loadGameState();
# Підготовка JSON-структури, яка допоможе в реалізації ігрової логіки на фронтенді
# Ми створимо структури для УПЛ, Першої ліги, та шаблони для статистики гравців

import json

upl_teams = [
    "Динамо", "Шахтар", "Кривбас", "Полісся", "Олександрія", "Зоря", "Ворскла",
    "Чорноморець", "Рух", "Колос", "Оболонь", "ЛНЗ", "Верес", "Маяк Ромни", "Лівий Берег", "Інгулець"
]

first_league_teams = [
    "Металіст", "Дніпро", "Кудрівка", "Епіцентр", "Суми", "Маріуполь",
    "Волинь", "Арсенал Київ", "Минай", "ЮКСА", "Буковина", "Чернігів",
    "Металург Запоріжжя", "Нива Тернопіль", "Полтава", "Прикарпаття"
]

# Приклад шаблонів гравців з кількістю голів та "сухими матчами"
players_stats = {
    "Динамо": [
        {"name": "Артем Бєсєдін", "goals": 0},
        {"name": "Георгій Бущан", "cleanSheets": 0}
    ],
    "Шахтар": [
        {"name": "Данило Сікан", "goals": 0},
        {"name": "Анатолій Трубін", "cleanSheets": 0}
    ],
    "Маяк Ромни": [
        {"name": "Олександр Рубець", "goals": 0},
        {"name": "Дмитро Лісний", "cleanSheets": 0}
    ],
    # Інші команди аналогічно…
}

# Формування фінального JSON-об'єкта
structure = {
    "uplTeams": upl_teams,
    "firstLeagueTeams": first_league_teams,
    "playerStats": players_stats
}

# Перетворення на JSON
json_output = json.dumps(structure, ensure_ascii=False, indent=2)
json_output[:1000]  # Показати перші 1000 символів для перевірки
# Додамо повний JavaScript-код у HTML-файл: автосимуляція сезону, таблиці УПЛ та Першої Ліги,
# генерація результатів, статистика бомбардирів/воротарів, автоматичне підвищення/виліт

# Спочатку базові змінні, потім структура standings, функції рендеру та симуляції

script_code = """
let uplTeams = [...Array(16)].map((_, i) => ({
  name: ["Динамо", "Шахтар", "Кривбас", "Полісся", "Олександрія", "Зоря", "Ворскла",
         "Чорноморець", "Рух", "Колос", "Оболонь", "ЛНЗ", "Верес", "Маяк Ромни", "Лівий Берег", "Інгулець"][i],
  pts: 0, gf: 0, ga: 0, wins: 0, draws: 0, losses: 0
}));

let firstLeagueTeams = [...Array(16)].map((_, i) => ({
  name: ["Металіст", "Дніпро", "Кудрівка", "Епіцентр", "Суми", "Маріуполь", "Волинь",
         "Арсенал Київ", "Минай", "ЮКСА", "Буковина", "Чернігів", "Металург Запоріжжя",
         "Нива Тернопіль", "Полтава", "Прикарпаття"][i],
  pts: 0, gf: 0, ga: 0
}));

let round = 0, maxRounds = 30;
let topScorers = {}, cleanSheets = {};

function simulateMatch(teamA, teamB, isUpl = true) {
  let goalsA = Math.floor(Math.random() * 4);
  let goalsB = Math.floor(Math.random() * 4);
  if (isUpl) updateTable(teamA, teamB, goalsA, goalsB, uplTeams);
  else updateTable(teamA, teamB, goalsA, goalsB, firstLeagueTeams);
  return `${teamA.name} ${goalsA} - ${goalsB} ${teamB.name}`;
}

function updateTable(a, b, gA, gB, table) {
  a.gf += gA; a.ga += gB;
  b.gf += gB; b.ga += gA;
  if (gA > gB) { a.pts += 3; a.wins++; b.losses++; }
  else if (gA < gB) { b.pts += 3; b.wins++; a.losses++; }
  else { a.pts += 1; b.pts += 1; a.draws++; b.draws++; }

  // Статистика гравців
  topScorers[a.name] = (topScorers[a.name] || 0) + gA;
  topScorers[b.name] = (topScorers[b.name] || 0) + gB;
  if (gB === 0) cleanSheets[a.name] = (cleanSheets[a.name] || 0) + 1;
  if (gA === 0) cleanSheets[b.name] = (cleanSheets[b.name] || 0) + 1;
}

function renderTable(teams, el, isUpl = true) {
  teams.sort((a, b) => b.pts - a.pts || (b.gf - b.ga) - (a.gf - a.ga));
  let html = "<table><tr><th>М</th><th>Команда</th><th>І</th><th>В</th><th>Н</th><th>П</th><th>ЗМ</th><th>ПМ</th><th>О</th></tr>";
  teams.forEach((t, i) => {
    let cls = "";
    if (isUpl) {
      if (i < 2) cls = "ucl";
      else if (i < 4) cls = "uel";
      else if (i === 4) cls = "conf";
      else if (i > 13) cls = "relegated";
    } else {
      if (i === 0) cls = "promo1";
      else if (i === 1) cls = "promo2";
    }
    html += `<tr class='${cls}'><td>${i + 1}</td><td>${t.name}</td><td>${t.wins + t.draws + t.losses}</td><td>${t.wins}</td><td>${t.draws}</td><td>${t.losses}</td><td>${t.gf}</td><td>${t.ga}</td><td>${t.pts}</td></tr>`;
  });
  html += "</table>";
  document.getElementById(el).innerHTML = html;
}

function simulateRound() {
  let results = "";
  for (let i = 0; i < uplTeams.length; i += 2) {
    results += simulateMatch(uplTeams[i], uplTeams[i + 1]) + "<br>";
  }
  for (let i = 0; i < firstLeagueTeams.length; i += 2) {
    simulateMatch(firstLeagueTeams[i], firstLeagueTeams[i + 1], false);
  }
  round++;
  document.getElementById("roundResults").innerHTML = results;
  renderTable(uplTeams, "uplTable");
  renderTable(firstLeagueTeams, "firstLeagueTable");
  if (round === maxRounds) endSeason();
}

function autoSimulateSeason() {
  while (round < maxRounds) simulateRound();
}

function endSeason() {
  uplTeams.sort((a, b) => b.pts - a.pts || (b.gf - b.ga) - (a.gf - a.ga));
  firstLeagueTeams.sort((a, b) => b.pts - a.pts);
  document.getElementById("champion").textContent = uplTeams[0].name;

  let topScorer = Object.entries(topScorers).sort((a, b) => b[1] - a[1])[0];
  let topKeeper = Object.entries(cleanSheets).sort((a, b) => b[1] - a[1])[0];
  document.getElementById("topScorer").textContent = topScorer ? topScorer[0] + " (" + topScorer[1] + " голів)" : "—";
  document.getElementById("topKeeper").textContent = topKeeper ? topKeeper[0] + " (" + topKeeper[1] + " сухих матчів)" : "—";

  // Виліт / підвищення
  let relegated = uplTeams.slice(-2);
  let promoted = firstLeagueTeams.slice(0, 2);
  uplTeams.splice(-2, 2, ...promoted.map(t => ({...t, pts: 0, gf: 0, ga: 0, wins: 0, draws: 0, losses: 0})));
  firstLeagueTeams.splice(0, 2, ...relegated.map(t => ({...t, pts: 0, gf: 0, ga: 0})));
}

function resetSeason() {
  location.reload();
}

window.onload = () => {
  renderTable(uplTeams, "uplTable");
  renderTable(firstLeagueTeams, "firstLeagueTable");
};
"""

# Додамо код у HTML-файл
full_html_path = "/mnt/data/futbolna-liga-ukrainy-full.html"

with open("/mnt/data/futbolna-liga-ukrainy.html", "r", encoding="utf-8") as f:
    base_html = f.read()

# Вставити скрипт перед </script>
final_html = base_html.replace("// ... ТУТ ДАЛІ ПІДІДЕ УВЕСЬ JAVASCRIPT-КОД ГРИ ...", script_code)

with open(full_html_path, "w", encoding="utf-8") as f:
    f.write(final_html)

full_html_path
