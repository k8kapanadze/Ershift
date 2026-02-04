
<html lang="ka">
<head>
<meta charset="UTF-8">
<title>ექთნების მორიგეობა</title>

<style>
body { font-family: sans-serif; padding: 20px; }
table { border-collapse: collapse; width: 100%; margin-top: 20px; }
th, td { border: 1px solid #555; padding: 4px; text-align: center; }
.planned { background: #f7f7f7; }
.real-row { background: #d6d6d6; font-weight: bold; }
.real-cell { color: #1b5e20; }
section { margin-top: 25px; padding: 15px; border: 2px solid #333; }
</style>
</head>

<body>

<h2>ექთნების მორიგეობის სისტემა</h2>

<select id="monthSelect" onchange="render()">
  <option value="0">იანვარი</option>
  <option value="1">თებერვალი</option>
  <option value="2">მარტი</option>
  <option value="3">აპრილი</option>
  <option value="4">მაისი</option>
  <option value="5">ივნისი</option>
  <option value="6">ივლისი</option>
  <option value="7">აგვისტო</option>
  <option value="8">სექტემბერი</option>
  <option value="9">ოქტომბერი</option>
  <option value="10">ნოემბერი</option>
  <option value="11">დეკემბერი</option>
</select>

<input id="nurseName" placeholder="სახელი და გვარი">
<button onclick="addNurse()">ექთნის დამატება</button>

<div id="tableContainer"></div>

<section>
<h3>რეალურად ნამუშევარი საათების დაფიქსირება</h3>
<select id="attNurse"></select>
<input type="number" id="attMonth" min="1" max="12" placeholder="თვე">
<input type="number" id="attDay" min="1" placeholder="რიცხვი">
<select id="attHours">
  <option value="8">8</option>
  <option value="16">16</option>
  <option value="24">24</option>
</select>
<button onclick="addAttendance()">დაფიქსირება</button>
</section>

<section>
<h3>გაცვლა (ორი თარიღი)</h3>
<select id="swapFrom"></select>
<input type="number" id="fromMonth" placeholder="თვე">
<input type="number" id="fromDay" placeholder="რიცხვი"><br><br>

<select id="swapTo"></select>
<input type="number" id="toMonth" placeholder="თვე">
<input type="number" id="toDay" placeholder="რიცხვი"><br><br>

<select id="swapHours">
  <option value="8">8</option>
  <option value="16">16</option>
  <option value="24">24</option>
</select>

<button onclick="addSwap()">გაცვლა</button>
</section>

<section>
<h3>ძებნა</h3>
<input id="searchName" placeholder="სახელი და გვარი">
<button onclick="searchByName()">ძებნა სახელით</button><br><br>

<input type="number" id="searchMonth" placeholder="თვე">
<input type="number" id="searchDay" placeholder="რიცხვი">
<button onclick="searchByDay()">ძებნა რიცხვით</button>

<div id="searchResult"></div>
</section>

<script>
let data = JSON.parse(localStorage.getItem("shiftFull") || "{}");
let currentMonth = 0;

function daysInMonth(m){ return new Date(2025, m+1, 0).getDate(); }

function initMonth(m){
  if(!data[m]) data[m]={ nurses:[], schedule:{}, real:{} };
}

function save(){ localStorage.setItem("shiftFull", JSON.stringify(data)); }

function addNurse(){
  const name = nurseName.value.trim();
  if(!name) return;
  const id = Date.now();
  for(let m=0;m<12;m++){
    initMonth(m);
    data[m].nurses.push({id,name});
    data[m].schedule[id]={};
    data[m].real[id]={};
  }
  nurseName.value="";
  save(); render();
}

function render(){
  currentMonth = Number(monthSelect.value);
  initMonth(currentMonth);
  const m = data[currentMonth];
  let html="<table><tr><th>ექთანი</th>";
  for(let d=1;d<=daysInMonth(currentMonth);d++) html+=`<th>${d}</th>`;
  html+="</tr>";

  m.nurses.forEach(n=>{
    html+=`<tr class="planned"><td>${n.name}</td>`;
    for(let d=1;d<=daysInMonth(currentMonth);d++){
      const v=m.schedule[n.id][d]||"";
      html+=`<td>
      <select onchange="data[${currentMonth}].schedule[${n.id}][${d}]=Number(this.value);save();render();">
        <option value=""></option>
        <option value="8" ${v==8?"selected":""}>8</option>
        <option value="16" ${v==16?"selected":""}>16</option>
        <option value="24" ${v==24?"selected":""}>24</option>
      </select></td>`;
    }
    html+="</tr>";

    html+=`<tr class="real-row"><td>რეალური</td>`;
    for(let d=1;d<=daysInMonth(currentMonth);d++){
      html+=`<td class="real-cell">${m.real[n.id][d]||""}</td>`;
    }
    html+="</tr>";
  });

  html+="</table>";
  tableContainer.innerHTML=html;

  attNurse.innerHTML =
  swapFrom.innerHTML =
  swapTo.innerHTML =
    m.nurses.map(n=>`<option value="${n.id}">${n.name}</option>`).join("");
}

function addAttendance(){
  const n=attNurse.value;
  const m=Number(attMonth.value)-1;
  const d=attDay.value;
  const h=Number(attHours.value);
  if(!data[m]) initMonth(m);
  data[m].real[n][d]=(data[m].real[n][d]||0)+h;
  save(); render();
}

function addSwap(){
  const f=swapFrom.value, t=swapTo.value;
  const fm=fromMonth.value-1, tm=toMonth.value-1;
  const fd=fromDay.value, td=toDay.value;
  const h=Number(swapHours.value);

  data[fm].schedule[f][fd]-=h;
  data[tm].schedule[t][td]=(data[tm].schedule[t][td]||0)+h;

  save(); render();
}

function searchByName(){
  const name=searchName.value;
  let res=[];
  for(let m in data){
    data[m].nurses.forEach(n=>{
      if(n.name===name){
        for(let d in data[m].schedule[n.id]){
          res.push(`${Number(m)+1}/${d} – ${data[m].schedule[n.id][d]} სთ`);
        }
      }
    });
  }
  searchResult.innerHTML=res.join("<br>");
}

function searchByDay(){
  const m=searchMonth.value-1;
  const d=searchDay.value;
  let res=[];
  data[m].nurses.forEach(n=>{
    if(data[m].schedule[n.id][d])
      res.push(`${n.name} – ${data[m].schedule[n.id][d
