<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ScoreLock Cricket - Live Scoring</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 20px; background: #f4f4f4; }
        h1 { text-align: center; }
        form { background: #fff; padding: 20px; margin-bottom: 20px; border-radius: 5px; }
        input, select, button { padding: 8px; margin: 5px 0; width: 100%; }
        table { width: 100%; border-collapse: collapse; margin-top: 10px; }
        table, th, td { border: 1px solid #ccc; }
        th, td { padding: 8px; text-align: left; }
        .locked { background: #eee; }
    </style>
</head>
<body>
    <h1>ScoreLock Cricket - Live Scoring (Prototype)</h1>

    <!-- Admin / Team Selection -->
    <form id="teamForm">
        <h2>Team Setup</h2>
        <label>Team Name:</label>
        <input type="text" id="teamName" required>
        
        <label>Player Name:</label>
        <input type="text" id="playerName" required>
        
        <label>Mobile Number (+91):</label>
        <input type="text" id="mobile" placeholder="10 digits" pattern="\d{10}" required>
        
        <label>Aadhaar Number:</label>
        <input type="text" id="aadhaar" placeholder="12 digits" pattern="\d{12}" required>
        
        <button type="button" onclick="addPlayer()">Add Player</button>
    </form>

    <!-- Player List Table -->
    <table id="playerTable">
        <thead>
            <tr>
                <th>Team</th>
                <th>Player</th>
                <th>Mobile</th>
                <th>Aadhaar</th>
            </tr>
        </thead>
        <tbody>
        </tbody>
    </table>

    <!-- Match Section -->
    <form id="matchForm">
        <h2>Match Setup</h2>
        <label>Team A:</label>
        <select id="teamA"></select>
        
        <label>Team B:</label>
        <select id="teamB"></select>
        
        <label>Toss Winner:</label>
        <select id="tossWinner">
            <option value="">--Select--</option>
        </select>
        
        <label>Toss Decision:</label>
        <select id="tossDecision">
            <option value="bat">Bat</option>
            <option value="bowl">Bowl</option>
        </select>
        
        <button type="button" onclick="startMatch()">Start Match</button>
    </form>

    <!-- Score Update -->
    <form id="scoreForm">
        <h2>Ball-by-Ball Score Update</h2>
        <label>Batsman:</label>
        <select id="batsman"></select>

        <label>Bowler:</label>
        <select id="bowler"></select>

        <label>Runs:</label>
        <input type="number" id="runs" min="0" required>

        <label>Wicket:</label>
        <select id="wicket">
            <option value="no">No</option>
            <option value="yes">Yes</option>
        </select>

        <button type="button" onclick="updateScore()">Update Score</button>
    </form>

    <h2>Live Score</h2>
    <div id="liveScore">No match started.</div>

    <script>
        const teams = {};
        const players = [];
        let currentMatch = null;
        let lock = false;

        function addPlayer() {
            const team = document.getElementById('teamName').value.trim();
            const name = document.getElementById('playerName').value.trim();
            const mobile = document.getElementById('mobile').value.trim();
            const aadhaar = document.getElementById('aadhaar').value.trim();

            // Validate duplicates
            if (players.some(p => p.name === name || p.mobile === mobile || p.aadhaar === aadhaar)) {
                alert("Duplicate player details not allowed!");
                return;
            }

            if (!teams[team]) teams[team] = [];
            const player = { team, name, mobile, aadhaar };
            players.push(player);
            teams[team].push(player);

            // Update player table
            const tbody = document.getElementById('playerTable').querySelector('tbody');
            const row = tbody.insertRow();
            row.insertCell(0).innerText = team;
            row.insertCell(1).innerText = name;
            row.insertCell(2).innerText = mobile;
            row.insertCell(3).innerText = aadhaar;

            updateTeamSelects();
        }

        function updateTeamSelects() {
            const teamASelect = document.getElementById('teamA');
            const teamBSelect = document.getElementById('teamB');
            [teamASelect, teamBSelect].forEach(sel => {
                sel.innerHTML = '<option value="">--Select--</option>';
                Object.keys(teams).forEach(t => {
                    const option = document.createElement('option');
                    option.value = t; option.text = t;
                    sel.appendChild(option);
                });
            });
        }

        function startMatch() {
            const teamA = document.getElementById('teamA').value;
            const teamB = document.getElementById('teamB').value;
            const tossWinner = document.getElementById('tossWinner').value || teamA;

            if (!teamA || !teamB || teamA === teamB) { alert("Select two different teams!"); return; }

            currentMatch = { teamA, teamB, tossWinner, score: [], editor: null };
            alert(`Match started between ${teamA} and ${teamB}. Toss winner: ${tossWinner}`);
            document.getElementById('liveScore').innerText = `Match: ${teamA} vs ${teamB} | Toss won by ${tossWinner}`;

            updatePlayerSelects();
        }

        function updatePlayerSelects() {
            const batsmanSelect = document.getElementById('batsman');
            const bowlerSelect = document.getElementById('bowler');
            [batsmanSelect, bowlerSelect].forEach(sel => sel.innerHTML = '');
            players.forEach(p => {
                const option1 = document.createElement('option'); option1.value = p.name; option1.text = p.name;
                const option2 = document.createElement('option'); option2.value = p.name; option2.text = p.name;
                batsmanSelect.appendChild(option1);
                bowlerSelect.appendChild(option2);
            });
        }

        function updateScore() {
            if (!currentMatch) { alert("Start a match first!"); return; }
            if (lock) { alert("Another user is updating score!"); return; }

            lock = true;
            const batsman = document.getElementById('batsman').value;
            const bowler = document.getElementById('bowler').value;
            const runs = parseInt(document.getElementById('runs').value);
            const wicket = document.getElementById('wicket').value;

            currentMatch.score.push({ batsman, bowler, runs, wicket });
            document.getElementById('liveScore').innerText = `Score Updated: ${JSON.stringify(currentMatch.score)}`;

            lock = false;
        }
    </script>
</body>
</html>
