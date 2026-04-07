<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>ScoreLock Cricket - Admin Dashboard</title>
<style>
@import url('https://fonts.googleapis.com/css2?family=Roboto:wght@400;500;700&display=swap');

/* Body & Fonts */
body { font-family:'Roboto', sans-serif; margin:0; padding:0; background: linear-gradient(135deg,#1e3d59,#f5f0e1); min-height:100vh; display:flex; justify-content:center; align-items:center; }

/* Frame Login */
.frame { display:flex; flex-direction:row; background: rgba(255,255,255,0.95); border-radius:20px; box-shadow:0 15px 40px rgba(0,0,0,0.3); overflow:hidden; max-width:900px; width:90%; }
.frame-left { flex:1; background:url('https://images.unsplash.com/photo-1597927211644-17f38c9b6f6b?auto=format&fit=crop&w=800&q=80') no-repeat center center; background-size:cover; position:relative; color:#fff; padding:30px; }
.frame-left::after { content:''; position:absolute; inset:0; background:rgba(30,61,89,0.6); border-radius:20px 0 0 20px; }
.frame-left h2, .frame-left p { position:relative; }
.frame-left h2 { font-size:2em; margin-bottom:15px; }
.frame-left p { font-size:1em; }

.frame-right { flex:1; padding:50px 40px; display:flex; flex-direction:column; justify-content:center; }
.frame-right h2 { color:#1e3d59; margin-bottom:25px; text-align:center; }
.frame-right label { margin-top:15px; font-weight:500; }
.frame-right input { padding:12px; margin-top:5px; border-radius:8px; border:1px solid #ccc; font-size:1em; width:100%; box-sizing:border-box; }
.frame-right button { margin-top:25px; padding:12px; background:#ff6f3c; color:#fff; font-weight:bold; border:none; border-radius:10px; cursor:pointer; transition:0.3s; }
.frame-right button:hover { background:#ff4f00; }
.frame-right .back { background:#1e3d59; margin-top:10px; }
.frame-right .back:hover { background:#163350; }

/* Dashboard */
#dashboard { display:none; width:90%; max-width:1200px; margin:20px auto; color:#1e3d59; }
#dashboard h2 { text-align:center; }
.card { background: rgba(255,255,255,0.95); border-radius: 15px; padding:20px; margin-bottom:25px; box-shadow:0 10px 25px rgba(0,0,0,0.2); }
.flex-row { display:flex; flex-wrap:wrap; gap:25px; }
.flex-row .card { flex:1 1 45%; }
label, input, select, button { display:block; width:100%; margin-top:5px; padding:10px; font-size:1em; border-radius:8px; border:1px solid #ccc; }
button { cursor:pointer; }
#liveScore { font-weight:bold; font-size:1.2em; padding:15px; background:#1e3d59; color:#fff; border-radius:10px; text-align:center; }

@media(max-width:768px){ .frame { flex-direction:column; border-radius:20px; } .frame-left { height:200px; border-radius:20px 20px 0 0; } .frame-left::after { border-radius:20px 20px 0 0; } .flex-row .card { flex:1 1 100%; } }
</style>
</head>
<body>

<!-- Login Frame -->
<div class="frame" id="loginFrame">
    <div class="frame-left">
        <h2>Welcome to ScoreLock Cricket</h2>
        <p>Manage teams, players, matches, and scores efficiently.<br>Admin login provides full access.</p>
    </div>
    <div class="frame-right">
        <h2>Admin Sign-In</h2>
        <label>Username:</label>
        <input type="text" id="adminUser" placeholder="Enter username">
        <label>Password:</label>
        <input type="password" id="adminPass" placeholder="Enter password">
        <button onclick="adminLogin()">Sign In</button>
    </div>
</div>

<!-- Dashboard Section -->
<div id="dashboard">
    <h2>Admin Dashboard - ScoreLock Cricket</h2>
    
    <div class="flex-row">
        <!-- Team & Player Setup -->
        <div class="card">
            <h3>Team & Player Setup</h3>
            <label>Team Name:</label>
            <input type="text" id="teamName" required>
            <label>Player Name:</label>
            <input type="text" id="playerName" required>
            <label>Mobile Number (+91):</label>
            <input type="tel" id="mobile" placeholder="10 digits" pattern="\d{10}" maxlength="10" required>
            <label>Aadhaar Number:</label>
            <input type="text" id="aadhaar" placeholder="12 digits" pattern="\d{12}" maxlength="12" required>
            <button onclick="addPlayer()">Add Player</button>
        </div>

        <!-- Match Setup -->
        <div class="card">
            <h3>Match Setup</h3>
            <label>Team A:</label>
            <select id="teamA"></select>
            <label>Team B:</label>
            <select id="teamB"></select>
            <label>Toss Winner:</label>
            <select id="tossWinner"></select>
            <label>Toss Decision:</label>
            <select id="tossDecision">
                <option value="bat">Bat</option>
                <option value="bowl">Bowl</option>
            </select>
            <button onclick="startMatch()">Start Match</button>
        </div>
    </div>

    <!-- Player List -->
    <div class="card">
        <h3>Player List</h3>
        <table id="playerTable">
            <thead>
                <tr><th>Team</th><th>Player</th><th>Mobile</th><th>Aadhaar</th></tr>
            </thead>
            <tbody></tbody>
        </table>
    </div>

    <!-- Score Update -->
    <div class="card">
        <h3>Score Update</h3>
        <label>Batsman:</label>
        <select id="batsman"></select>
        <label>Bowler:</label>
        <select id="bowler"></select>
        <label>Runs:</label>
        <input type="number" id="runs" min="0">
        <label>Wicket:</label>
        <select id="wicket"><option value="no">No</option><option value="yes">Yes</option></select>
        <button onclick="updateScore()">Update Score</button>
    </div>

    <div class="card">
        <h3>Live Score</h3>
        <div id="liveScore">No match started.</div>
    </div>
</div>

<script>
// Users stored in localStorage
let users = JSON.parse(localStorage.getItem('users')) || [{username:'admin', password:'admin123', role:'admin'}];
localStorage.setItem('users', JSON.stringify(users));

function adminLogin(){
    const user = document.getElementById('adminUser').value.trim();
    const pass = document.getElementById('adminPass').value.trim();
    users = JSON.parse(localStorage.getItem('users'));
    const found = users.find(u=>u.username===user && u.password===pass && u.role==='admin');
    if(found){
        alert("Login successful!");
        document.getElementById('loginFrame').style.display='none';
        document.getElementById('dashboard').style.display='block';
    } else {
        alert("Invalid credentials!");
    }
}

// Cricket Dashboard Logic
const teams = {};
const players = [];
let currentMatch = null;
let lock = false;

function addPlayer() {
    const team = document.getElementById('teamName').value.trim();
    const name = document.getElementById('playerName').value.trim();
    const mobile = document.getElementById('mobile').value.trim();
    const aadhaar = document.getElementById('aadhaar').value.trim();
    if(players.some(p => p.name===name || p.mobile===mobile || p.aadhaar===aadhaar)){
        alert("Duplicate player details not allowed!");
        return;
    }
    if(!teams[team]) teams[team]=[];
    const player={team,name,mobile,aadhaar};
    players.push(player);
    teams[team].push(player);

    const tbody=document.getElementById('playerTable').querySelector('tbody');
    const row=tbody.insertRow();
    row.insertCell(0).innerText=team;
    row.insertCell(1).innerText=name;
    row.insertCell(2).innerText=mobile;
    row.insertCell(3).innerText=aadhaar;

    updateTeamSelects();
}

function updateTeamSelects(){
    const teamASelect=document.getElementById('teamA');
    const teamBSelect=document.getElementById('teamB');
    const tossWinner=document.getElementById('tossWinner');
    [teamASelect,teamBSelect,tossWinner].forEach(sel=>{
        sel.innerHTML='<option value="">--Select--</option>';
        Object.keys(teams).forEach(t=>{
            const option=document.createElement('option'); option.value=t; option.text=t;
            sel.appendChild(option);
        });
    });
}

function startMatch(){
    const teamA=document.getElementById('teamA').value;
    const teamB=document.getElementById('teamB').value;
    const tossWinner=document.getElementById('tossWinner').value;
    if(!teamA || !teamB || teamA===teamB){ alert("Select two different teams!"); return; }
    currentMatch={teamA,teamB,tossWinner,score:[],editor:null};
    alert(`Match started between ${teamA} and ${teamB}. Toss winner: ${tossWinner}`);
    document.getElementById('liveScore').innerText=`Match: ${teamA} vs ${teamB} | Toss won by ${tossWinner}`;
    updatePlayerSelects();
}

function updatePlayerSelects(){
    const batsmanSelect=document.getElementById('batsman');
    const bowlerSelect=document.getElementById('bowler');
    [batsmanSelect,bowlerSelect].forEach(sel=>sel.innerHTML='');
    players.forEach(p=>{
        const option1=document.createElement('option'); option1.value=p.name; option1.text=p.name;
        const option2=document.createElement('option'); option2.value=p.name; option2.text=p.name;
        batsmanSelect.appendChild(option1);
        bowlerSelect.appendChild(option2);
    });
}

function updateScore(){
    if(!currentMatch){ alert("Start a match first!"); return; }
    if(lock){ alert("Another user is updating score!"); return; }
    lock=true;
    const batsman=document.getElementById('batsman').value;
    const bowler=document.getElementById('bowler').value;
    const runs=parseInt(document.getElementById('runs').value);
    const wicket=document.getElementById('wicket').value;
    currentMatch.score.push({batsman,bowler,runs,wicket});
    document.getElementById('liveScore').innerText=`Score Updated: ${JSON.stringify(currentMatch.score)}`;
    lock=false;
}
</script>

</body>
</html>
