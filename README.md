<!DOCTYPE html>
<html lang="ka">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<title>ექთნების მორიგეობის სისტემა</title>

<style>

body{
font-family:system-ui;
margin:0;
background:#f4f6fb;
}

header{
background:white;
padding:12px;
display:flex;
flex-wrap:wrap;
gap:8px;
box-shadow:0 2px 8px rgba(0,0,0,0.05);
}

h1{
font-size:18px;
margin-right:auto;
}

input,select,button{
padding:6px 10px;
border-radius:6px;
border:1px solid #ddd;
}

button{
background:#4f6df5;
color:white;
border:none;
cursor:pointer;
}

table{
border-collapse:collapse;
width:100%;
background:white;
margin-top:10px;
}

th,td{
border:1px solid #eee;
padding:5px;
text-align:center;
font-size:13px;
}

th.date{
writing-mode:vertical-rl;
transform:rotate(180deg);
font-size:11px;
padding:8px 2px;
}

.plan{background:white;}
.real{background:#f2f2f2;}

.sum{
font-weight:bold;
background:#fafafa;
}

.menu{
cursor:pointer;
}

.modal{
display:none;
position:fixed;
top:0;
left:0;
width:100%;
height:100%;
background:rgba(0,0,0,0.4);
}

.modal-box{
background:white;
padding:20px;
width:300px;
margin:10% auto;
border-radius:10px;
}

textarea{
width:100%;
margin-bottom:6px;
}

</style>
</head>

<body>

<header>

<h1>ექთნების მორიგეობა</h1>

<input id="nurseInput" placeholder="ექთნის სახელი">
<button onclick="addNurse()">დამატება</button>

<select id="month"></select>
<select id="year"></select>

<button onclick="generate()">განახლება</button>
<button onclick="autoFill()">მე-4 დღე</button>

<button onclick="print()">ბეჭდვა</button>

</header>


<header>

<select id="realNurse"></select>

<input id="realDay" type="number" placeholder="დღე">

<select id="realHours">
<option>8</option>
<option>16</option>
<option>24</option>
</select>

<button onclick="setReal()">რეალური საათი</button>

</header>


<table id="table"></table>


<div id="modal" class="modal">

<div class="modal-box">

<h3>დღის ჩანაწერი</h3>

<textarea id="exp" placeholder="სამუშაო გამოცდილება"></textarea>
<textarea id="note" placeholder="პირადი დღიური"></textarea>

<button onclick="saveJournal()">შენახვა</button>
<button onclick="closeModal()">დახურვა</button>

</div>

</div>


<script>

let nurses=JSON.parse(localStorage.nurses||"[]")
let journal=JSON.parse(localStorage.journal||"{}")

let activeCell=null

const month=document.getElementById("month")
const year=document.getElementById("year")

for(let i=1;i<=12;i++)
month.innerHTML+=`<option value="${i}">${i}</option>`

for(let i=2024;i<=2030;i++)
year.innerHTML+=`<option value="${i}">${i}</option>`

month.value=new Date().getMonth()+1
year.value=new Date().getFullYear()


function save(){

localStorage.nurses=JSON.stringify(nurses)
localStorage.journal=JSON.stringify(journal)

}


function addNurse(){

let name=document.getElementById("nurseInput").value.trim()

if(!name)return

if(nurses.includes(name)){
alert("ეს ექთანი უკვე არსებობს")
return
}

nurses.push(name)

save()

generate()

}


function removeNurse(name){

if(confirm("წავშალო ექთანი?")){

nurses=nurses.filter(n=>n!==name)

save()

generate()

}

}


function generate(){

let days=new Date(year.value,month.value,0).getDate()

let html="<tr><th>ექთანი</th>"

for(let d=1;d<=days;d++)
html+=`<th class="date">${d}</th>`

html+="<th>ჯამი</th></tr>"

nurses.forEach(n=>{

html+=`<tr class="plan">

<td>${n}
<span class="menu" onclick="removeNurse('${n}')">⋮</span>
</td>`

for(let d=1;d<=days;d++){

html+=`<td>

<select onchange="sumRow(this)">
<option></option>
<option>8</option>
<option>16</option>
<option>24</option>
</select>

</td>`

}

html+=`<td class="sum">0</td></tr>`


html+=`<tr class="real">

<td>რეალური</td>`

for(let d=1;d<=days;d++){

html+=`<td onclick="openJournal(this)"></td>`

}

html+=`<td class="sum">0</td></tr>`

})

document.getElementById("table").innerHTML=html

document.getElementById("realNurse").innerHTML=
nurses.map(n=>`<option>${n}</option>`).join("")

}


function sumRow(el){

let row=el.closest("tr")

let sum=0

row.querySelectorAll("select").forEach(s=>{
sum+=Number(s.value||0)
})

row.querySelector(".sum").innerText=sum

}


function autoFill(){

document.querySelectorAll(".plan").forEach(row=>{

let selects=row.querySelectorAll("select")

selects.forEach((s,i)=>{

if(s.value){

for(let j=i+4;j<selects.length;j+=4)
selects[j].value=s.value

}

})

sumRow(selects[0])

})

}


function setReal(){

let nurse=document.getElementById("realNurse").value
let day=parseInt(document.getElementById("realDay").value)
let hours=document.getElementById("realHours").value

document.querySelectorAll(".plan").forEach(r=>{

if(r.children[0].innerText.includes(nurse)){

let real=r.nextElementSibling

real.children[day].innerText=hours

}

})

}


function openJournal(cell){

activeCell=cell

document.getElementById("modal").style.display="block"

}


function saveJournal(){

journal[Date.now()]={
exp:document.getElementById("exp").value,
note:document.getElementById("note").value
}

activeCell.innerText="📝"

save()

closeModal()

}


function closeModal(){

document.getElementById("modal").style.display="none"

}


function print(){

window.print()

}


generate()

</script>

</body>
</html>
