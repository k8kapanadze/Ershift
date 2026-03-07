<!DOCTYPE html>
<html lang="ka">

<head>

<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<title>CMC Shift</title>

<script src="https://www.gstatic.com/firebasejs/9.22.0/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.22.0/firebase-database-compat.js"></script>

<style>

body{
font-family:system-ui;
margin:0;
background:#f2f4f7;
}

header{
background:#001f3f;
color:white;
padding:15px;
font-size:18px;
text-align:center;
}

.container{
padding:15px;
}

.controls{
display:flex;
gap:10px;
flex-wrap:wrap;
margin-bottom:15px;
}

input,select,button{
padding:10px;
border-radius:8px;
border:1px solid #ccc;
font-size:14px;
}

button{
background:#001f3f;
color:white;
border:none;
}

.nurse-card{

background:white;
border-radius:12px;
padding:15px;
margin-bottom:15px;

box-shadow:0 2px 10px rgba(0,0,0,0.1);

}

.nurse-title{
font-weight:bold;
font-size:16px;
margin-bottom:10px;
display:flex;
justify-content:space-between;
}

.days{

display:flex;
overflow-x:auto;
gap:8px;

}

.day{

min-width:70px;
background:#f7f7f7;
border-radius:8px;
padding:6px;
text-align:center;

}

.day select{

width:100%;

}

.real{

margin-top:5px;
background:#eaf3ff;
padding:5px;
border-radius:6px;
font-size:12px;
cursor:pointer;

}

.weekend{
background:#ffeaea;
}

</style>

</head>

<body>

<header>
CMC Shift Manager
</header>

<div class="container">

<div class="controls">

<input id="nurseInp" placeholder="ექთანის სახელი">

<button onclick="addNurse()">დამატება</button>

<select id="month"></select>

<input id="year" type="number">

</div>

<div id="app"></div>

</div>

<script>

const firebaseConfig={
apiKey:"YOURKEY",
authDomain:"YOURDOMAIN",
databaseURL:"YOURDB",
projectId:"YOURID"
};

firebase.initializeApp(firebaseConfig);

const db=firebase.database();

let nurses=[];
let schedule={};

const months=["იან","თებ","მარ","აპრ","მაი","ივნ","ივლ","აგვ","სექ","ოქტ","ნოე","დეკ"];

const monthSelect=document.getElementById("month");

months.forEach((m,i)=>{
monthSelect.innerHTML+=`<option value="${i}">${m}</option>`;
});

monthSelect.value=new Date().getMonth();

document.getElementById("year").value=new Date().getFullYear();

db.ref().on("value",snap=>{

const data=snap.val()||{};

nurses=data.nurses||[];
schedule=data.schedule||{};

render();

});

function addNurse(){

const name=document.getElementById("nurseInp").value.trim();

if(!name)return;

if(nurses.includes(name))return alert("არსებობს");

nurses.push(name);

db.ref("nurses").set(nurses);

document.getElementById("nurseInp").value="";

}

function render(){

const m=parseInt(monthSelect.value);
const y=parseInt(document.getElementById("year").value);

const days=new Date(y,m+1,0).getDate();

let html="";

nurses.forEach(n=>{

html+=`<div class="nurse-card">

<div class="nurse-title">

${n}

<span onclick="deleteNurse('${n}')" style="cursor:pointer">❌</span>

</div>

<div class="days">`;

for(let d=1;d<=days;d++){

const key=`${y}-${m}-${n}`;

const p=(schedule[key]&&schedule[key][d])||0;

const wk=[0,6].includes(new Date(y,m,d).getDay())?"weekend":"";

html+=`

<div class="day ${wk}">

<div>${d}</div>

<select onchange="setPlan('${n}',${d},this.value)">

<option value="0">-</option>
<option value="8">8</option>
<option value="16">16</option>
<option value="24">24</option>

</select>

<div class="real" onclick="setReal('${n}',${d})">

${p||"რეალური"}

</div>

</div>

`;

}

html+="</div></div>";

});

document.getElementById("app").innerHTML=html;

}

function setPlan(n,d,v){

const m=monthSelect.value;
const y=document.getElementById("year").value;

db.ref(`schedule/${y}-${m}-${n}/${d}`).set(parseInt(v));

}

function setReal(n,d){

const val=prompt("რეალური საათი");

if(!val)return;

const m=monthSelect.value;
const y=document.getElementById("year").value;

db.ref(`schedule/${y}-${m}-${n}/${d}`).set(parseInt(val));

}

function deleteNurse(n){

if(!confirm("წავშალოთ?"))return;

nurses=nurses.filter(x=>x!==n);

db.ref("nurses").set(nurses);

}

</script>

</body>
</html>
