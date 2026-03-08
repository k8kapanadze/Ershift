<!DOCTYPE html>
<html lang="ka">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<title>Nurse Shift Manager</title>

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
padding:6px;
text-align:center;
font-size:13px;
}

.day{
background:#fafafa;
font-weight:bold;
}

.sumRow{
background:#f2f2f2;
font-weight:bold;
}

.menu{
cursor:pointer;
}

</style>
</head>

<body>

<header>

<h1>ექთნების მორიგეობა</h1>

<input id="nurseInput" placeholder="სახელი გვარი">

<button onclick="addNurse()">დამატება</button>

<select id="month"></select>
<select id="year"></select>

<button onclick="generate()">თვე</button>

<button onclick="autoFillYear()">მე-4 დღე (წელი)</button>

<button onclick="print()">ბეჭდვა</button>

</header>

<table id="table"></table>

<script>

let nurses=JSON.parse(localStorage.nurses||"[]")
let shifts=JSON.parse(localStorage.shifts||"{}")

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
localStorage.shifts=JSON.stringify(shifts)

}

function addNurse(){

let name=nurseInput.value.trim()

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

if(confirm("წავშალო?")){

nurses=nurses.filter(n=>n!==name)

save()

generate()

}

}

function generate(){

let days=new Date(year.value,month.value,0).getDate()

let html="<tr><th>დღე</th>"

nurses.forEach(n=>{
html+=`<th>${n} <span class="menu" onclick="removeNurse('${n}')">⋮</span></th>`
})

html+="</tr>"

for(let d=1;d<=days;d++){

html+=`<tr><td class="day">${d}</td>`

nurses.forEach((n,i)=>{

let key=`${year.value}-${month.value}-${d}-${i}`

let value=shifts[key]||""

html+=`
<td>

<select onchange="saveShift(${d},${i},this.value)">

<option value=""></option>
<option value="8" ${value==8?"selected":""}>8</option>
<option value="16" ${value==16?"selected":""}>16</option>
<option value="24" ${value==24?"selected":""}>24</option>

</select>

</td>
`

})

html+="</tr>"

}

html+=`<tr class="sumRow"><td>ჯამი</td>`

nurses.forEach((n,i)=>{

let sum=0

Object.keys(shifts).forEach(k=>{

let parts=k.split("-")

if(parts[0]==year.value && parts[1]==month.value && parts[3]==i){

sum+=Number(shifts[k])

}

})

html+=`<td>${sum}</td>`

})

html+="</tr>"

table.innerHTML=html

}

function saveShift(day,index,value){

let key=`${year.value}-${month.value}-${day}-${index}`

if(value)shifts[key]=value
else delete shifts[key]

save()

generate()

}

function autoFillYear(){

let y=Number(year.value)

nurses.forEach((n,i)=>{

for(let m=1;m<=12;m++){

let days=new Date(y,m,0).getDate()

for(let d=1;d<=days;d++){

let key=`${y}-${m}-${d}-${i}`

if(shifts[key]){

let val=shifts[key]

for(let k=4;k<365;k+=4){

let date=new Date(y,m-1,d)

date.setDate(date.getDate()+k)

if(date.getFullYear()!=y)break

let nk=`${date.getFullYear()}-${date.getMonth()+1}-${date.getDate()}-${i}`

shifts[nk]=val

}

}

}

}

})

save()

generate()

}

function print(){

window.print()

}

generate()

</script>

</body>
</html>
