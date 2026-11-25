# index.html
Main website interface for Six Number Studio 2.0.

<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<meta name="theme-color" content="#101c32">
<title>Six Number Studio 2.0</title>
<style>
*{box-sizing:border-box}
body{margin:0;background:#edf2f9;color:#15233b;font-family:Arial,sans-serif}
header{background:#101c32;color:white;padding:24px 18px;border-bottom:4px solid #438ee8}
header h1{margin:0;font-size:25px}
header p{color:#b9cce5;font-size:13px}
main{max-width:850px;margin:auto;padding:18px}
nav{display:grid;grid-template-columns:repeat(2,1fr);gap:9px;margin-bottom:18px}
nav button{padding:13px 5px;border:0;border-radius:10px;background:#dce7f5;color:#17365d;font-weight:bold}
nav button.active{background:#246dc1;color:white}
.card{background:white;padding:19px;border-radius:15px;margin-bottom:15px;box-shadow:0 4px 15px #142c4810}
h2{font-size:19px;margin-top:0}
button{cursor:pointer}
.action{background:#236fc4;color:white;border:0;padding:13px 17px;border-radius:10px;font-weight:bold}
input,select,textarea{width:100%;padding:13px;margin:7px 0 13px;border:1px solid #cbd7e6;border-radius:9px;font-size:15px}
.number{background:#101c32;color:white;text-align:center;padding:25px 8px;border-radius:12px;font-size:35px;font-weight:bold;letter-spacing:6px;margin:16px 0}
.page{display:none}.page.show{display:block}
.small{color:#687b96;font-size:13px;line-height:1.5}
.record{padding:12px 0;border-bottom:1px solid #e1e8f0}
footer{text-align:center;padding:25px;color:#718198;font-size:12px}
</style>
</head>
<body>
<header>
<h1>SIX NUMBER STUDIO <span style="color:#68a9ff">2.0</span></h1>
<p>Personal Number & Data Management Studio</p>
</header>
<main>
<nav>
<button class="active" data-page="home">Home</button>
<button data-page="generator">Generator</button>
<button data-page="collections">Collections</button>
<button data-page="history">History</button>
<button data-page="analytics">Analytics</button>
<button data-page="admin">Admin</button>
</nav>

<section id="home" class="page show">
<div class="card">
<h2>Welcome to Six Number Studio</h2>
<p>Your personal workspace for random number generation and historical data management.</p>
<button class="action" onclick="openPage('generator')">Open Generator</button>
</div>
<div class="card">
<h2>Dashboard Overview</h2>
<p>Total Saved Records: <b id="total">0</b></p>
<p>Total Collections: <b id="totalCollections">1</b></p>
</div>
</section>

<section id="generator" class="page">
<div class="card">
<h2>Number Generator</h2>
<label>Generation Mode</label>
<select id="mode" onchange="toggleMode()">
<option value="auto">Automatic Random</option>
<option value="manual">Manual Entry</option>
</select>
<div id="autoBox">
<button class="action" onclick="generate()">Generate Six Digits</button>
</div>
<div id="manualBox" style="display:none">
<input id="manualNumber" maxlength="6" inputmode="numeric" placeholder="Enter six digits">
<button class="action" onclick="manualGenerate()">Use Number</button>
</div>
<div class="number" id="result">000000</div>
<label>Collection</label>
<select id="collectionSelect"><option>General</option></select>
<label>Note</label>
<textarea id="note" placeholder="Optional note"></textarea>
<button class="action" onclick="saveRecord()">Save Record</button>
<p class="small" id="message">Numbers are random and are not predictions.</p>
</div>
</section>

<section id="collections" class="page">
<div class="card">
<h2>My Collections</h2>
<input id="collectionName" placeholder="New collection name">
<button class="action" onclick="addCollection()">Add Collection</button>
<div id="collectionList"></div>
</div>
</section>

<section id="history" class="page">
<div class="card">
<h2>Saved History</h2>
<input id="search" placeholder="Search records..." oninput="renderHistory()">
<div id="historyList"></div>
<button class="action" onclick="exportData()">Export Backup</button>
</div>
</section>

<section id="analytics" class="page">
<div class="card">
<h2>Basic Analytics</h2>
<p>Total records: <b id="analyticsTotal">0</b></p>
<p>Collections: <b id="analyticsCollections">1</b></p>
<p class="small">Statistics are descriptive summaries of saved records, not predictions.</p>
</div>
</section>

<section id="admin" class="page">
<div class="card">
<h2>Admin Settings</h2>
<p class="small">This is an interface preview. Secure admin authentication and Supabase connection will be integrated in the next step.</p>
<label>Project URL</label>
<input placeholder="Supabase Project URL" disabled>
<label>Public API Key</label>
<input placeholder="Configure in next step" disabled>
<p>Admin access: Not connected</p>
</div>
</section>
<footer>Six Number Studio 2.0 · Random generation · Personal data management</footer>
</main>
<script>
let records=JSON.parse(localStorage.getItem('snsRecords')||'[]');
let collections=JSON.parse(localStorage.getItem('snsCollections')||'["General"]');
let current='000000';
function openPage(id){
 document.querySelectorAll('.page').forEach(p=>p.classList.toggle('show',p.id===id));
 document.querySelectorAll('nav button').forEach(b=>b.classList.toggle('active',b.dataset.page===id));
 if(id==='history')renderHistory();
 if(id==='collections')renderCollections();
 updateStats();
}
document.querySelectorAll('nav button').forEach(b=>b.onclick=()=>openPage(b.dataset.page));
function toggleMode(){
 let m=document.getElementById('mode').value;
 document.getElementById('autoBox').style.display=m==='auto'?'block':'none';
 document.getElementById('manualBox').style.display=m==='manual'?'block':'none';
}
function generate(){
 current=Array.from({length:6},()=>Math.floor(Math.random()*10)).join('');
 document.getElementById('result').textContent=current;
 document.getElementById('message').textContent='Random number generated.';
}
function manualGenerate(){
 let n=document.getElementById('manualNumber').value.trim();
 if(!/^\d{6}$/.test(n)){alert('Please enter exactly six digits.');return;}
 current=n;document.getElementById('result').textContent=n;
}
function saveRecord(){
 if(!/^\d{6}$/.test(current)){alert('Generate a number first.');return;}
 records.unshift({number:current,collection:document.getElementById('collectionSelect').value,note:document.getElementById('note').value,date:new Date().toLocaleString()});
 localStorage.setItem('snsRecords',JSON.stringify(records));
 document.getElementById('note').value='';
 document.getElementById('message').textContent='Record saved on this device.';
 updateStats();
}
function addCollection(){
 let n=document.getElementById('collectionName').value.trim();
 if(!n)return;
 if(collections.some(c=>c.toLowerCase()===n.toLowerCase())){alert('Already exists');return;}
 collections.push(n);localStorage.setItem('snsCollections',JSON.stringify(collections));
 document.getElementById('collectionName').value='';
 renderCollections();updateStats();
}
function renderCollections(){
 let list=document.getElementById('collectionList');list.innerHTML='';
 collections.forEach(c=>{let d=document.createElement('div');d.className='record';d.textContent=c;list.appendChild(d)});
 let s=document.getElementById('collectionSelect');let old=s.value;s.innerHTML='';
 collections.forEach(c=>{let o=document.createElement('option');o.textContent=c;s.appendChild(o)});
 if(collections.includes(old))s.value=old;
}
function renderHistory(){
 let q=document.getElementById('search').value.toLowerCase();
 let list=document.getElementById('historyList');list.innerHTML='';
 let filtered=records.filter(r=>(r.number+' '+r.collection+' '+r.note).toLowerCase().includes(q));
 if(!filtered.length){list.textContent='No records found.';return;}
 filtered.forEach(r=>{let d=document.createElement('div');d.className='record';let b=document.createElement('b');b.textContent=r.number;let p=document.createElement('p');p.className='small';p.textContent=r.collection+' · '+r.date+' · '+r.note;d.append(b,p);list.appendChild(d)});
}
function updateStats(){
 document.getElementById('total').textContent=records.length;
 document.getElementById('totalCollections').textContent=collections.length;
 document.getElementById('analyticsTotal').textContent=records.length;
 document.getElementById('analyticsCollections').textContent=collections.length;
}
function exportData(){
 let blob=new Blob([JSON.stringify({records,collections},null,2)],{type:'application/json'});
 let a=document.createElement('a');a.href=URL.createObjectURL(blob);a.download='six-number-studio-backup.json';a.click();
 URL.revokeObjectURL(a.href);
}
renderCollections();updateStats();
</script>
</body>
</html>
