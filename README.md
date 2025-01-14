# roll-of-the-night
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Roll Of The Night</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            text-align: center;
            margin: 0;
            padding: 0;
        }
        .container {
            padding: 20px;
        }
        input, button {
            margin: 10px;
            padding: 10px;
            font-size: 16px;
        }
        .tabs {
            margin: 20px 0;
        }
        .tab-button {
            padding: 10px 20px;
            margin: 0 5px;
            cursor: pointer;
            background-color: #007BFF;
            color: white;
            border: none;
            border-radius: 5px;
        }
        .tab-button.active {
            background-color: #0056b3;
        }
        .tab-content {
            display: none;
        }
        .tab-content.active {
            display: block;
        }
        .tables {
            display: flex;
            justify-content: center;
            gap: 20px;
            margin-top: 20px;
        }
        table {
            width: auto;
            border-collapse: collapse;
        }
        th, td {
            border: 1px solid #ddd;
            padding: 8px;
            white-space: nowrap;
        }
        th {
            background-color: #f2f2f2;
            text-align: center;
        }
        td {
            text-align: center;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Roll Of The Night</h1>
        <div id="setup">
            <p>Enter players for a lifetime match:</p>
            <input type="text" id="playerName" placeholder="Player Name" />
            <button onclick="addPlayer()">Add Player</button>
            <button onclick="startGame()">Start Game</button>
            <ul id="playerList"></ul>
        </div>
        <div id="game" style="display: none;">
            <p>Current Guesser: <span id="currentGuesser"></span></p>
            <input type="number" id="guess" placeholder="Enter Guess (1-6)" />
            <input type="number" id="rolledNumber" placeholder="Enter Roll (1-6)" />
            <button onclick="submitRound()">Submit</button>

            <div class="tabs">
                <button class="tab-button active" onclick="switchTab('historyTab')">Game History</button>
                <button class="tab-button" onclick="switchTab('statsTab')">Player Stats</button>
            </div>

            <div id="historyTab" class="tab-content active">
                <div class="tables">
                    <div>
                        <h3>Wins This Month</h3>
                        <table id="matchTable">
                            <thead>
                                <tr>
                                    <th>Guesser</th>
                                    <th>Wins</th>
                                </tr>
                            </thead>
                            <tbody></tbody>
                        </table>
                    </div>
                    <div>
                        <h3>Game History</h3>
                        <table id="historyTable">
                            <thead>
                                <tr>
                                    <th>Date</th>
                                    <th>Guesser</th>
                                    <th>Guessed #</th>
                                    <th>Rolled #</th>
                                </tr>
                            </thead>
                            <tbody></tbody>
                        </table>
                    </div>
                </div>
            </div>

            <div id="statsTab" class="tab-content">
                <div id="playerStats"></div>
            </div>
        </div>
    </div>
    <script>
        let players = [];
        let currentGuesserIndex = 0;
        let history = [];
        let stats = {};

        function addPlayer() {
            const name = document.getElementById('playerName').value;
            if (name) {
                players.push(name);
                stats[name] = { guesses: Array(6).fill(0), rolls: Array(6).fill(0), wins: 0 };
                document.getElementById('playerList').innerHTML += `<li>${name}</li>`;
                document.getElementById('playerName').value = '';
            }
        }

        function startGame() {
            if (players.length > 0) {
                document.getElementById('setup').style.display = 'none';
                document.getElementById('game').style.display = 'block';
                document.getElementById('currentGuesser').textContent = players[currentGuesserIndex];
                updatePlayerStats();
            }
        }

        function submitRound() {
            const guess = document.getElementById('guess').value;
            const roll = document.getElementById('rolledNumber').value;

            if (guess && roll) {
                const currentDate = new Date();
                const guessInt = parseInt(guess);
                const rollInt = parseInt(roll);

                history.push({
                    date: currentDate,
                    guesser: players[currentGuesserIndex],
                    guess: guessInt,
                    roll: rollInt
                });

                if (guessInt === rollInt) {
                    stats[players[currentGuesserIndex]].wins++;
                }

                stats[players[currentGuesserIndex]].guesses[guessInt - 1]++;
                stats[players[currentGuesserIndex]].rolls[rollInt - 1]++;

                currentGuesserIndex = (currentGuesserIndex + 1) % players.length;
                document.getElementById('currentGuesser').textContent = players[currentGuesserIndex];
                document.getElementById('guess').value = '';
                document.getElementById('rolledNumber').value = '';

                updateHistory();
                updateMatchTable();
                updatePlayerStats();
            }
        }

        function updateHistory() {
            const historyTable = document.querySelector('#historyTable tbody');
            historyTable.innerHTML = '';
            history.forEach(entry => {
                const row = `<tr>
                    <td>${entry.date.toLocaleDateString('en-US')}</td>
                    <td>${entry.guesser}</td>
                    <td>${entry.guess}</td>
                    <td>${entry.roll}</td>
                </tr>`;
                historyTable.innerHTML += row;
            });
        }

        function updateMatchTable() {
            const matchTable = document.querySelector('#matchTable tbody');
            matchTable.innerHTML = '';

            players.forEach(player => {
                const row = `<tr>
                    <td>${player}</td>
                    <td>${stats[player].wins}</td>
                </tr>`;
                matchTable.innerHTML += row;
            });
        }

        function updatePlayerStats() {
            const playerStatsDiv = document.getElementById('playerStats');
            playerStatsDiv.innerHTML = '';

            players.forEach(player => {
                const statsTable = `<div>
                    <h3>${player}'s Stats</h3>
                    <table>
                        <thead>
                            <tr>
                                <th>Number</th>
                                <th>Guessed</th>
                                <th>Rolled</th>
                            </tr>
                        </thead>
                        <tbody>
                            ${[1, 2, 3, 4, 5, 6].map(num => `
                                <tr>
                                    <td>${num}</td>
                                    <td>${stats[player].guesses[num - 1]}</td>
                                    <td>${stats[player].rolls[num - 1]}</td>
                                </tr>`).join('')}
                        </tbody>
                    </table>
                </div>`;
                playerStatsDiv.innerHTML += statsTable;
            });
        }

        function switchTab(tabId) {
            document.querySelectorAll('.tab-button').forEach(button => {
                button.classList.remove('active');
            });
            document.querySelectorAll('.tab-content').forEach(content => {
                content.classList.remove('active');
            });

            document.querySelector(`#${tabId}`).classList.add('active');
            document.querySelector(`[onclick="switchTab('${tabId}')"]`).classList.add('active');
        }
    </script>
</body>
</html>
