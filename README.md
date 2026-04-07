<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Cricket Manager</title>
<style>
body { font-family: Arial, sans-serif; background:#f4f6f8; margin:0; padding:0; }
header { background:#1e3d59; color:white; padding:20px; text-align:center; }
section { display:none; padding:20px; max-width:1000px; margin:20px auto; background:white; border-radius:10px; box-shadow:0 5px 15px rgba(0,0,0,0.1); }
h2,h3 { color:#1e3d59; }
label { display:block; margin-top:10px; }
input, select, button { display:block; margin-top:5px; padding:10px; width:100%; max-width:400px; border-radius:5px; border:1px solid #ccc; }
button { background:#ff6f3c; color:white; border:none; cursor:pointer; margin-top:10px; }
button:hover { background:#ff4f00; }
table { width:100%; border-collapse:collapse; margin-top:10px; }
th, td { border:1px solid #ccc; padding:8px; text-align:center; }
#scoreboard { margin-top:15px; padding:10px; background:#1e3d59; color:white; font-weight:bold; text-align:center; border-radius:5px; }
.deleteBtn { background:red; padding:5px 10px; }
.deleteBtn:hover { background:#b30000; }
</style>
</head>
<body>

<header>
<h1>Cricket Manager</h1>
<p>Admin Full Access | Single User Score Lock</p>
</header>

<!-- Home -->
<section id="home" style="display:block;">
<h2>Welcome</h2>
<p>Manage your cricket teams, players, and match scores efficiently.</p>
<button onclick="showSection('adminLogin')">Admin Sign-In</button>
<button onclick="showSection('userLogin')">User Sign-In</button>
</section>

<!-- Admin Login -->
<section id="adminLogin">
<h2>Admin Login</h2>
<label>Username:</label><input type="text" id="adminUser">
<label>Password:</label><input type="password" id="adminPass">
<button onclick="adminLogin()">Sign In</button>
<button onclick="showSection('home')">Back</button>
</section>

<!-- Admin Dashboard -->
<section id="adminDashboard">
<h2>Admin Dashboard</h2>

<h3>Create User</h3>
<label>Username:</label><input type="text" id="newUsername">
<label>Password:</label><input type="password" id="newUserPass">
<button onclick="createUser()">Create User</button>

<h3>Users</h3>
<ul id="userList"></ul>

<h3>Players</h3>
<table>
<thead><tr><th>Name</th><th>Mobile</th><th>Aadhaar</th><th>Address</th><th>Team</th><th>Action</th></tr></thead>
<tbody id="playerTable"></tbody>
</table>

<h3>Teams</h3>
<table>
<thead><tr><th>Team</th><th>Players</th><th>Action</th></tr></thead>
<tbody id="teamTable"></tbody>
</table>

<h3>Ball-by-Ball Score</h3>
<label>Batsman:</label><select id="batsmanSelect"></select>
<label>Bowler:</label><select id="bowlerSelect"></select>
<label>Runs:</label><input type="number" id="runsInput" min="0">
<label>Wicket:</label><select id="wicketInput"><option>No</option><option>Yes</option></select>
<button onclick="updateScore(true)">Update Score (Admin)</button>
<div id="scoreboard">Score: 0</div>
<button onclick="logout()">Logout</button>
</section>

<!-- User Login -->
<section id="userLogin">
<h2>User Login</h2>
<label>Username:</label><input type="text" id="userName">
<label>Password:</label><input type="password" id="userPass">
<button onclick="userLogin()">Sign In</button>
<button onclick="showSection('home')">Back</button>
</section>

<!-- User Dashboard -->
<section id="userDashboard">
<h2>User Dashboard</h2>

<h3>Add Player</h3>
<label>Name:</label><input type="text" id="playerName">
<label>Mobile (10 digits):</label><input type="tel" id="playerMobile" maxlength="10">
<label>Aadhaar (12 digits):</label><input type="text" id="playerAadhaar" maxlength="12">
<label>Address:</label><input type="text" id="playerAddress">
<button onclick="addPlayer()">Add Player</button>

<h3>Players</h3>
<table>
<thead><tr><th>Name</th><th>Mobile</th><th>Aadhaar</th><th>Address</th><th>Team</th></tr></thead>
<tbody id="playerTableUser"></tbody>
</table>

<h3>Form Team (12 Members)</h3>
<label>Team Name:</label><input type="text" id="teamName">
<label>Select Players:</label>
<select id="teamPlayers" multiple size="12"></select>
<button onclick="formTeam()">Create Team</button>

<h3>Ball-by-Ball Score</h3>
<label>Batsman:</label><select id="batsmanSelectUser"></select>
<label>Bowler:</label><select id="bowlerSelectUser"></select>
<label>Runs:</label><input type="number" id="runsInputUser" min="0">
<label>Wicket:</label><select id="wicketInputUser"><option>No</option><option>Yes</option></select>
<button onclick="updateScore(false)">Update Score</button>
<div id="scoreboardUser">Score: 0</div>

<button onclick="logout()">Logout</button>
</section>

<script>
function showSection(id){
    document.querySelectorAll('section').forEach(s=>s.style.display='none');
    document.getElementById(id).style.display='block';
}

// Admin
let admin={username:'admin', password:'admin123'};
let users=JSON.parse(localStorage.getItem('users'))||[];
let currentUser=null;

// Data
let players=[], teams={}, matchScore=[], lock=false;

// Admin Functions
function adminLogin(){
    const u=document.getElementById('adminUser').value;
    const p=document.getElementById('adminPass').value;
    if(u===admin.username && p===admin.password){ showSection('adminDashboard'); renderUsers(); renderPlayers(true); renderTeams(); refreshSelects(true);}
    else alert('Invalid admin credentials');
}

function createUser(){
    const uname=document.getElementById('newUsername').value;
    const pass=document.getElementById('newUserPass').value;
    if(users.find(u=>u.username===uname)){ alert('User exists'); return; }
    users.push({username:uname,password:pass});
    localStorage.setItem('users', JSON.stringify(users));
    renderUsers();
}

function renderUsers(){
    const ul=document.getElementById('userList'); ul.innerHTML='';
    users.forEach((u,i)=>{
        const li=document.createElement('li');
        li.textContent=u.username+' ';
        const del=document.createElement('button'); del.textContent='Delete'; del.className='deleteBtn';
        del.onclick=()=>{ users.splice(i,1); localStorage.setItem('users', JSON.stringify(users)); renderUsers();}
        li.appendChild(del); ul.appendChild(li);
    });
}

// User login
function userLogin(){
    const uname=document.getElementById('userName').value;
    const pass=document.getElementById('userPass').value;
    const found=users.find(u=>u.username===uname && u.password===pass);
    if(found){ currentUser=found; showSection('userDashboard'); renderPlayers(false); refreshSelects(false);}
    else alert('Invalid credentials');
}

function logout(){ currentUser=null; showSection('home'); }

// Players
function addPlayer(){
    const isAdmin = document.getElementById('adminDashboard').style.display==='block';
    const name = isAdmin ? prompt("Player Name") : document.getElementById('playerName').value.trim();
    const mobile = isAdmin ? prompt("Mobile") : document.getElementById('playerMobile').value.trim();
    const aadhaar = isAdmin ? prompt("Aadhaar") : document.getElementById('playerAadhaar').value.trim();
    const address = isAdmin ? prompt("Address") : document.getElementById('playerAddress').value.trim();

    if(!name || !mobile || !aadhaar || !address){ alert('Fill all'); return; }
    if(mobile.length!==10 || isNaN(mobile)) { alert('Invalid mobile'); return; }
    if(aadhaar.length!==12 || isNaN(aadhaar)) { alert('Invalid Aadhaar'); return; }
    if(players.find(p=>p.aadhaar===aadhaar || p.mobile===mobile)){ alert('Duplicate player'); return; }

    players.push({name,mobile,aadhaar,address,team:''});
    renderPlayers(isAdmin);
    refreshSelects(isAdmin);
}

function renderPlayers(isAdmin){
    const tbody = isAdmin ? document.getElementById('playerTable') : document.getElementById('playerTableUser');
    tbody.innerHTML='';
    players.forEach((p,i)=>{
        const row=tbody.insertRow();
        row.insertCell(0).innerText=p.name;
        row.insertCell(1).innerText=p.mobile;
        row.insertCell(2).innerText=p.aadhaar;
        row.insertCell(3).innerText=p.address;
        row.insertCell(4).innerText=p.team;
        if(isAdmin){
            const actions=row.insertCell(5);
            const del=document.createElement('button'); del.textContent='Delete'; del.className='deleteBtn';
            del.onclick=()=>{ players.splice(i,1); renderPlayers(true); refreshSelects(true);}
            actions.appendChild(del);
        }
    });
}

// Teams
function formTeam(){
    const teamName=document.getElementById('teamName').value.trim();
    const selected=Array.from(document.getElementById('teamPlayers').selectedOptions);
    if(selected.length!==12){ alert('Select exactly 12 players'); return; }
    if(teams[teamName]){ alert('Team exists'); return; }

    const teamMembers=[];
    for(let opt of selected){
        const player=players.find(p=>p.name===opt.value);
        if(player.team){ alert(`${player.name} already in a team`); return; }
        player.team=teamName; 
        teamMembers.push(player.name);
    }
    teams[teamName]=teamMembers;
    renderTeams();
    renderPlayers(true);
    refreshSelects(true);
}

function renderTeams(){
    const tbody=document.getElementById('teamTable'); tbody.innerHTML='';
    for(let t in teams){
        const row=tbody.insertRow();
        row.insertCell(0).innerText=t;
        row.insertCell(1).innerText=teams[t].join(', ');
        const actions=row.insertCell(2);
        const del=document.createElement('button'); del.textContent='Delete'; del.className='deleteBtn';
        del.onclick=()=>{ teams[t].forEach(pn=>{ players.find(p=>p.name===pn).team=''; }); delete teams[t]; renderTeams(); renderPlayers(true); refreshSelects(true);}
        actions.appendChild(del);
    }
}

// Score
function refreshSelects(isAdmin){
    const batsman = isAdmin ? document.getElementById('batsmanSelect') : document.getElementById('batsmanSelectUser');
    const bowler = isAdmin ? document.getElementById('bowlerSelect') : document.getElementById('bowlerSelectUser');
    const teamSelect = document.getElementById('teamPlayers');

    batsman.innerHTML = '';
    bowler.innerHTML = '';
    teamSelect.innerHTML = '';

    players.forEach(p=>{
        if(p.team){
            batsman.innerHTML += `<option>${p.name}</option>`;
            bowler.innerHTML += `<option>${p.name}</option>`;
        } else {
            teamSelect.innerHTML += `<option value="${p.name}">${p.name}</option>`;
        }
    });
}

function updateScore(isAdmin){
    const batsman = isAdmin ? document.getElementById('batsmanSelect').value : document.getElementById('batsmanSelectUser').value;
    const bowler = isAdmin ? document.getElementById('bowlerSelect').value : document.getElementById('bowlerSelectUser').value;
    const runs = parseInt(isAdmin ? prompt("Runs?") : document.getElementById(isAdmin?'runsInput':'runsInputUser').value);
    const wicket = isAdmin ? prompt("Wicket Yes/No?") : document.getElementById(isAdmin?'wicketInput':'wicketInputUser').value;

    if(!batsman || !bowler || isNaN(runs)){ alert('Fill all'); return; }

    if(!isAdmin && lock){ alert('Another user updating score!'); return; }
    if(!isAdmin) lock=true;

    matchScore.push({batsman,bowler,runs,wicket});
    let total = matchScore.reduce((a,v)=>a+v.runs,0);
    const board = isAdmin?document.getElementById('scoreboard'):document.getElementById('scoreboardUser');
    board.innerText=`Score: ${total}`;

    if(!isAdmin) lock=false;
}
</script>
</body>
</html>
