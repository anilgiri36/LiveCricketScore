<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>ScoreLock Cricket</title>
<style>
@import url('https://fonts.googleapis.com/css2?family=Roboto:wght@400;500;700&display=swap');

/* General Body & Font */
body { font-family: 'Roboto', sans-serif; margin:0; padding:0; background: linear-gradient(135deg,#1e3d59,#f5f0e1); min-height:100vh; }
header { background: rgba(30,61,89,0.95); color: #fff; text-align:center; padding:20px; font-size:2em; font-weight:bold; box-shadow:0 5px 15px rgba(0,0,0,0.3); }

/* Cards */
.card { background: rgba(255,255,255,0.95); border-radius: 15px; padding: 20px; margin-bottom: 25px; box-shadow: 0 10px 25px rgba(0,0,0,0.2); transition: transform 0.3s, box-shadow 0.3s; }
.card:hover { transform: translateY(-5px); box-shadow:0 15px 35px rgba(0,0,0,0.3); }
.card h2 { margin-top:0; color:#1e3d59; border-bottom:2px solid #1e3d59; padding-bottom:5px; }

label { display:block; margin:10px 0 5px; font-weight:500; }
input, select, button { width:100%; padding:12px; margin-bottom:12px; border-radius:8px; border:1px solid #ccc; font-size:1em; box-sizing:border-box; }
button { background:#ff6f3c; color:#fff; border:none; font-weight:bold; cursor:pointer; transition:0.3s; }
button:hover { background:#ff4f00; }

table { width:100%; border-collapse: collapse; margin-top:10px; background:#fff; border-radius:10px; overflow:hidden; }
th, td { padding:12px; text-align:left; border-bottom:1px solid #ddd; }
th { background:#1e3d59; color:#fff; }

.flex-row { display:flex; flex-wrap:wrap; gap:25px; }
.flex-row .card { flex:1 1 45%; }

#liveScore { font-weight:bold; font-size:1.2em; padding:15px; background:#1e3d59; color:#fff; border-radius:10px; text-align:center; }

@media(max-width:768px){ .flex-row .card { flex:1 1 100%; } }

/* Section visibility */
section { display:none; }
section.active { display:block; }
</style>
</head>
<body>

<header>ScoreLock Cricket</header>

<div class="container">

<!-- Welcome Section -->
<section id="welcome" class="active card">
    <h2>Welcome</h2>
    <button onclick="showSection('adminLogin')">Admin Sign-In</button>
    <button onclick="showSection('createUser')">Create User</button>
    <button onclick="showSection('dashboard')">Go to Dashboard</button>
</section>

<!-- Admin Login Section -->
<section id="adminLogin" class="card">
    <h2>Admin Sign-In</h2>
    <label>Username:</label>
    <input type="text" id="adminUser" placeholder="Enter username">
    <label>Password:</label>
    <input type="password" id="adminPass" placeholder="Enter password">
    <button onclick="adminLogin()">Sign In</button>
    <button onclick="showSection('welcome')">Back</button>
</section>

<!-- Create User Section -->
<section id="createUser" class="card">
    <h2>Create User</h2>
    <label>Username:</label>
    <input type="text" id="newUser" placeholder="Enter username">
    <label>Password:</label>
    <input type="password" id="newPass" placeholder="Enter password">
    <label>Role:</label>
    <select id="role">
        <option value="player">Player</option>
        <option value="admin">Admin</option>
    </select>
    <button onclick="createUser()">Create User</button>
    <button onclick="showSection('welcome')">Back</button>
</section>

<!-- Dashboard Section -->
<section id="dashboard" class="card">
    <h2>Team & Match Setup</h2>
    <div class="flex-row">
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

    <div class="card">
        <h3>Player List</h3>
        <table id="playerTable">
            <thead>
                <tr>
                    <th>Team</th>
                    <th>Player</th>
                    <th>Mobile</th>
                    <th>Aadhaar</th>
                </tr>
            </thead>
            <tbody></tbody>
        </table>
    </div>

    <div class="card">
        <h3>Score Update</h3>
        <label>Batsman:</label>
        <select id="batsman"></select>
        <label>Bowler:</label>
        <select id="bowler"></select>
        <label>Runs:</label>
        <input type="number" id="runs" min="0">
        <label>Wicket:</label>
        <select id="wicket">
            <option value="no">No</option>
            <option value="yes">Yes</option>
        </select>
        <button onclick="updateScore()">Update Score</button>
    </div>

    <div class="card">
        <h3>Live Score</h3>
        <div id="liveScore">No match started.</div>
    </div>
</section>

</div>

<script>
// Section Navigation
function showSection(id){
    document.querySelectorAll('section').forEach(sec => sec.classList.remove('active'));
    document.getElementById(id).classList.add('active');
}

// Initialize Users in localStorage
let users = JSON.parse(localStorage.getItem('users')) || [{username:'admin', password:'admin123', role:'admin'}];
localStorage.setItem('users', JSON.stringify(users));

// Admin Login
function adminLogin(){
    const user = document.getElementById('adminUser').value.trim();
    const pass = document.getElementById('adminPass').value.trim();
    users = JSON.parse(localStorage.getItem('users'));
    const found = users.find(u=>u.username===user && u.password===pass && u.role==='admin');
    if(found){ alert('Login Successful!'); showSection('dashboard'); }
    else{ alert('Invalid credentials!'); }
}

// Create User
function createUser(){
    const username = document.getElementById('newUser').value.trim();
    const password = document.getElementById('newPass').value.trim();
    const role = document.getElementById('role').value;
    users = JSON.parse(localStorage.getItem('users'));

    if(users.some(u=>u.username===username)){ alert('Username already exists!'); return; }

    users.push({username,password,role});
    localStorage.setItem('users', JSON.stringify(users));
    alert(`User ${username} created successfully!`);
    document.getElementById('newUser').value='';
    document.getElementById('newPass').value='';
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
