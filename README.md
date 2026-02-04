
<html lang="ka">
<head>
<meta charset="UTF-8">
<title>ექთნების მორიგეობა</title>

<style>
body { font-family: sans-serif; padding:20px; }
table { border-collapse: collapse; width:100%; margin-top:15px; }
th, td { border:1px solid #555; padding:4px; text-align:center; }

.planned-row { background:#f5f5f5; }
.real-row { background:#d0d0d0; }

.total-plan { background:#e6e6e6; font-weight:bold; }
.total-real { background:#f2bcbc; color:darkred; font-weight:bold; }

section { margin-top:25px; border:2px solid #333; padding:15px; }

@media print {
  button, input, select, section { display:none; }
}
</style>
</head>

<body>

<h2>ექთნების მორიგეობის სისტემა</h2>

<select id="monthSelect"></select>
<button onclick="window.print()">PDF ბეჭდვა / შენახვა</button>

<br><br>

<input id="nurseName" placeholder="სახელი და გვარი">
<button onclick="addNurse()">ექთნის დამატება</button>
<button onclick="autoFill()">ყოველ მე-4 დღეზე შევსება</button>

<div id="tableContainer"></div>

<section>
<h3>რეალურად შესრულებული საათების დაფიქსირება</h3>
<select id="realNurse"></select>
<select id="realMonth"></select>
<input type="number" id="realDay" min="1" placeholder="რიცხვი">
<select id="realHours">
  <option value="8">8 სთ</option>
  <option value="16">16 სთ</option>
  <option value="24">24 სთ</option>
</select>
<button onclick="addReal()">დაფიქსირება</button>
</section>

<section>
<h3>ძებნა – სახელი + თვე</h3>
<select id="searchNurse"></select>
<select id="searchMonth"></select>
<button onclick="searchByNurse()">ძებნა</button>
<div id="result1"></div>
</section>

<section>
<h3>ძებნა – თვე + რიცხვი</h3>
<select id="searchMonth2"></select>
<input type="number" id="searchDay" min="1" placeholder="რიცხვი">
<button onclick="searchByDay()">ძებნა</button>
<div id="result2"></div>
</section>

<script>
const months = ["იანვარი","თებერვალი","მარტი","აპრილი","მაისი","ივნისი","ივლისი","აგვისტო","სექტემბერი","ოქტომბერი","ნოემბერი","დეკემბერი"];

let data = JSON.parse(localStorage.getItem("nurseSchedule") || `{
  "nurses": [],
  "months": {}
}`);

function save(){ localStorage.setItem("nurseSchedule", JSON.stringify(data)); }

function daysInMonth(m){
  return new Date(2025, m+1, 0).getDate();
}

function initMonth(m){
  if(!data.months[m]){
    data.months[m] = { planned:{}, real:{} };
    data.nurses.forEach(n=>{
      data.months[m].planned[n.id] = {};
      data.months[m].real[n.id] = {};
    });
  }
}

function addNurse(){
  const name = nurseName.value.trim();
  if(!name) return;
  const id = Date.now();
  data.nurses.push({id,name});
  Object.keys(data.months).forEach(m=>{
    data.months[m].planned[id] = {};
    data.months[m].real[id] = {};
  });
  nurseName.value="";
  save(); render();
}

function autoFill(){
  const m = monthSelect.value;
  initMonth(m);
  data.nurses.forEach(n=>{
    const days = Object.keys(data.months[m].planned[n.id]);
    if(days.length === 0) return;
    const first = days[0];
    const val = data.months[m].planned[n.id][first];
    for(let d = +first + 4; d <= daysInMonth(m); d += 4){
      data.months[m].planned[n.id][d] = val;
    }
  });
  save(); render();
}

function addReal(){
  const m = realMonth.value;
  const id = realNurse.value;
  const d = realDay.value;
  const h = Number(realHours.value);
  if(!id || !d) return;
  initMonth(m);
  data.months[m].real[id][d] = h;
  save(); render();
}

function sum(o){
  return Object.values(o||{}).reduce((a,b)=>a+(b||0),0);
}

function render(){
  const m = monthSelect.value;
  initMonth(m);

  let html = "<table><tr><th>ექთანი</th>";
  for(let d=1; d<=daysInMonth(m); d++) html+=`<th>${d}</th>`;
  html += "<th>დაგეგმილი</th><th>რეალური</th></tr>";

  data.nurses.forEach(n=>{
    html += `<tr class="planned-row"><td>${n.name}</td>`;
    for(let d=1; d<=daysInMonth(m); d++){
      const v = data.months[m].planned[n.id][d] || "";
      html += `<td>
        <select onchange="data.months[${m}].planned[${n.id}][${d}]=Number(this.value);save();render();">
          <option></option>
          <option value="8" ${v==8?"selected":""}>8</option>
          <option value="16" ${v==16?"selected":""}>16</option>
          <option value="24" ${v==24?"selected":""}>24</option>
        </select>
      </td>`;
    }
    html += `<td class="total-plan">${sum(data.months[m].planned[n.id])}</td>`;
    html += `<td class="total-real">${sum(data.months[m].real[n.id])}</td></tr>`;

    html += `<tr class="real-row"><td>რეალური</td>`;
    for(let d=1; d<=daysInMonth(m); d++){
      html += `<td>${data.months[m].real[n.id][d] || ""}</td>`;
    }
    html += "<td></td><td></td></tr>";
  });

  html += "</table>";
  tableContainer.innerHTML = html;

  const nurseOptions = data.nurses.map(n=>`<option value="${n.id}">${n.name}</option>`).join("");
  realNurse.innerHTML = searchNurse.innerHTML = nurseOptions;
}

function searchByNurse(){
  const m = searchMonth.value;
  const id = searchNurse.value;
  initMonth(m);
  const days = Object.keys(data.months[m].planned[id] || {});
  result1.innerHTML = "მორიგეობის დღეები: " + (days.length ? days.join(", ") : "არ აქვს");
}

function searchByDay(){
  const m = searchMonth2.value;
  const d = searchDay.value;
  initMonth(m);
  const res = data.nurses.filter(n=>data.months[m].planned[n.id][d]).map(n=>n.name);
  result2.innerHTML = "იმ დღეს მორიგეა: " + (res.length ? res.join(", ") : "არავინ");
}

months.forEach((mo,i)=>{
  monthSelect.innerHTML += `<option value="${i}">${mo}</option>`;
  realMonth.innerHTML += `<option value="${i}">${mo}</option>`;
  searchMonth.innerHTML += `<option value="${i}">${mo}</option>`;
  searchMonth2.innerHTML += `<option value="${i}">${mo}</option>`;
});

monthSelect.value = new Date().getMonth();
realMonth.value = monthSelect.value;

render();
</script>

</body>
</html>
