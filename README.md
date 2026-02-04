
<html lang="ka">
<head>
<meta charset="UTF-8">
<title>ექთნების მორიგეობა</title>

<style>
body { font-family: sans-serif; padding: 20px; }
table { border-collapse: collapse; width: 100%; margin-top: 20px; }
th, td { border: 1px solid #555; padding: 4px; text-align: center; }
.planned-row { background: #f5f5f5; }
.real-row { background: #cfcfcf; font-weight: bold; color: #1b5e20; }
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

<br><br>

<input id="nurseName" placeholder="სახელი და გვარი">
<button onclick="addNurse()">ექთნის დამატება</button>
<button onclick="autoGenerate()">ყოველ მე-4 დღეზე შევსება</button>

<div id="tableContainer"></div>

<section>
<h3>რეალურად ნამუშევარი საათების დაფიქსირება</h3>
<select id="attNurse"></select>
<select id="attMonth"></select>
<input type="number" id="attDay" min="1" placeholder="რიცხვი">
<select id="attHours">
  <option value="8">8</option>
  <option value="16">16</option>
  <option value="24">24</option>
</select>
<button onclick="addAttendance()">დაფიქსირება</button>
</section>

<script>
const months = ["იანვარი","თებერვალი","მარტი","აპრილი","მაისი","ივნისი","ივლისი","აგვისტო","სექტემბერი","ოქტომბერი","ნოემბერი","დეკემბერი"];

let data = JSON.parse(localStorage.getItem("nurseShiftData") || `{
  "nurses": [],
  "months": {}
}`);

function daysInMonth(m){ return new Date(2025, m+1, 0).getDate(); }

function initMonth(m){
  if(!data.months[m]){
    data.months[m] = { schedule:{}, real:{} };
    data.nurses.forEach(n=>{
      data.months[m].schedule[n.id] = {};
      data.months[m].real[n.id] = {};
    });
  }
}

function save(){ localStorage.setItem("nurseShiftData", JSON.stringify(data)); }

function addNurse(){
  const name = nurseName.value.trim();
  if(!name) return;
  const id = Date.now();
  data.nurses.push({id,name});

  for(let m=0;m<12;m++){
    initMonth(m);
    data.months[m].schedule[id] = {};
    data.months[m].real[id] = {};
  }

  nurseName.value="";
  save();
  render();
}

function autoGenerate(){
  const m = Number(monthSelect.value);
  initMonth(m);
  data.nurses.forEach(n=>{
    const firstDay = Object.keys(data.months[m].schedule[n.id])[0];
    if(!firstDay) return;
    const val = data.months[m].schedule[n.id][firstDay];
    for(let d=Number(firstDay)+4; d<=daysInMonth(m); d+=4){
      data.months[m].schedule[n.id][d] = val;
    }
  });
  save(); render();
}

function render(){
  const m = Number(monthSelect.value);
  initMonth(m);

  let html = "<table><tr><th>ექთანი</th>";
  for(let d=1; d<=daysInMonth(m); d++) html+=`<th>${d}</th>`;
  html+="</tr>";

  data.nurses.forEach(n=>{
    html+=`<tr class="planned-row"><td>${n.name}</td>`;
    for(let d=1; d<=daysInMonth(m); d++){
      const v=data.months[m].schedule[n.id][d]||"";
      html+=`<td>
        <select onchange="data.months[${m}].schedule[${n.id}][${d}]=Number(this.value);save();render();">
          <option value=""></option>
          <option value="8" ${v==8?"selected":""}>8</option>
          <option value="16" ${v==16?"selected":""}>16</option>
          <option value="24" ${v==24?"selected":""}>24</option>
        </select>
      </td>`;
    }
    html+="</tr>";

    html+=`<tr class="real-row"><td>რეალური</td>`;
    for(let d=1; d<=daysInMonth(m); d++){
      html+=`<td>${data.months[m].real[n.id][d]||""}</td>`;
    }
    html+="</tr>";
  });

  html+="</table>";
  tableContainer.innerHTML = html;

  attNurse.innerHTML = data.nurses.map(n=>`<option value="${n.id}">${n.name}</option>`).join("");
  attMonth.innerHTML = months.map((mo,i)=>`<option value="${i}">${mo}</option>`).join("");
}

function addAttendance(){
  const n=attNurse.value;
  const m=Number(attMonth.value);
  const d=attDay.value;
  const h=Number(attHours.value);
  initMonth(m);
  data.months[m].real[n][d]=(data.months[m].real[n][d]||0)+h;
  save(); render();
}

render();
</script>

</body>
</html>
