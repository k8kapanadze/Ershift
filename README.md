<!DOCTYPE html>
<html lang="ka">
<head>
<meta charset="UTF-8">
<title>ექთნების მორიგეობის სისტემა</title>

<style>
body { font-family: sans-serif; padding: 20px; }
table { border-collapse: collapse; margin-top: 15px; }
th, td { border: 1px solid #666; padding: 4px; text-align: center; }
.total-planned { background: #eee; font-weight: bold; }
.total-real { background: #cfd8dc; font-weight: bold; }
.real { font-size: 11px; color: #2e7d32; }
section { margin-top: 25px; padding: 15px; border: 2px solid #333; }

@media print {
  button, section { display: none; }
  select { appearance: none; border: none; }
}
</style>
</head>

<body>

<h2>ექთნების მორიგეობის სისტემა</h2>

<select id="month" onchange="changeMonth()">
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

<button onclick="window.print()">PDF ამობეჭდვა</button>

<br><br>

<input id="nurseName" placeholder="სახელი და გვარი">
<button onclick="addNurse()">დამატება</button>
<button onclick="autoGenerate()">ყოველ მე-4 დღეზე შევსება</button>

<div id="tableContainer"></div>

<section>
<h3>რეალურად ნამუშევარი საათების დაფიქსირება</h3>
<select id="attNurse"></select>
<input id="attDay" type="number" min="1">
<select id="attHours">
  <option value="8">8</option>
  <option value="16">16</option>
  <option value="24">24</option>
</select>
<button onclick="addAttendance()">ასახვა</button>
</section>

<section>
<h3>გაცვლის დაფიქსირება</h3>
<input id="swapDay" type="number" min="1">
<select id="swapFrom"></select>
<select id="swapTo"></select>
<select id="swapHours">
  <option value="8">8</option>
  <option value="16">16</option>
  <option value="24">24</option>
</select>
<button onclick="addSwap()">გაცვლა</button>
</section>

<section>
<h3>გაცვლის ისტორია</h3>
<ul id="swapHistory"></ul>
</section>

<script>
function getDays(month, year = new Date().getFullYear()) {
  return new Date(year, month + 1, 0).getDate();
}

let currentMonth = 0;
let data = JSON.parse(localStorage.getItem("scheduleData") || "{}");

function initMonth() {
  if (!data[currentMonth]) {
    data[currentMonth] = {
      nurses: [],
      schedule: {},
      real: {},
      swaps: []
    };
  }
}

function save() {
  localStorage.setItem("scheduleData", JSON.stringify(data));
}

function changeMonth() {
  currentMonth = Number(month.value);
  initMonth();
  render();
}

function addNurse() {
  const name = nurseName.value.trim();
  if (!name) return;
  const id = Date.now();
  data[currentMonth].nurses.push({ id, name });
  data[currentMonth].schedule[id] = {};
  data[currentMonth].real[id] = {};
  nurseName.value = "";
  save();
  render();
}

function autoGenerate() {
  const m = data[currentMonth];
  m.nurses.forEach(n => {
    const firstDay = Object.keys(m.schedule[n.id])[0] || 1;
    const value = m.schedule[n.id][firstDay];
    if (!value) return;
    for (let d = Number(firstDay) + 4; d <= getDays(currentMonth); d += 4) {
      if (!m.schedule[n.id][d]) {
        m.schedule[n.id][d] = value;
      }
    }
  });
  save();
  render();
}

function addAttendance() {
  const id = attNurse.value;
  const day = attDay.value;
  const hours = Number(attHours.value);
  if (!id || !day) return;
  data[currentMonth].real[id][day] = hours;
  save();
  render();
}

function addSwap() {
  const day = swapDay.value;
  const from = swapFrom.value;
  const to = swapTo.value;
  const hours = Number(swapHours.value);
  if (!day || !from || !to) return;

  const planned = data[currentMonth].schedule[from][day] || 0;
  data[currentMonth].schedule[from][day] = 0;
  data[currentMonth].schedule[to][day] = planned;

  data[currentMonth].real[from][day] = 0;
  data[currentMonth].real[to][day] = hours;

  const fromName = data[currentMonth].nurses.find(n => n.id == from)?.name;
  const toName = data[currentMonth].nurses.find(n => n.id == to)?.name;

  data[currentMonth].swaps.push(
    `დღე ${day}: ${fromName} შეცვალა ${toName} (${hours} სთ)`
  );

  save();
  render();
}

function sum(obj) {
  return Object.values(obj || {}).reduce((a, b) => a + (b || 0), 0);
}

function render() {
  initMonth();
  const m = data[currentMonth];
  let html = "<table><tr><th>ექთანი</th>";

  for (let d = 1; d <= getDays(currentMonth); d++) {
    html += `<th>${d}</th>`;
  }

  html += "<th class='total-planned'>დაგეგმილი</th>";
  html += "<th class='total-real'>რეალური</th></tr>";

  m.nurses.forEach(n => {
    html += `<tr><td>${n.name}</td>`;
    for (let d = 1; d <= getDays(currentMonth); d++) {
      const planned = m.schedule[n.id][d] || "";
      html += `<td>
        <select onchange="data[${currentMonth}].schedule[${n.id}][${d}] = Number(this.value); save(); render();">
          <option value=""></option>
          <option value="8" ${planned==8?"selected":""}>8</option>
          <option value="16" ${planned==16?"selected":""}>16</option>
          <option value="24" ${planned==24?"selected":""}>24</option>
        </select>
        <div class="real">${m.real[n.id][d] || ""}</div>
      </td>`;
    }
    html += `<td class='total-planned'>${sum(m.schedule[n.id])}</td>`;
    html += `<td class='total-real'>${sum(m.real[n.id])}</td></tr>`;
  });

  html += "</table>";
  tableContainer.innerHTML = html;

  attNurse.innerHTML =
  swapFrom.innerHTML =
  swapTo.innerHTML =
    m.nurses.map(n => `<option value="${n.id}">${n.name}</option>`).join("");

  swapHistory.innerHTML =
    m.swaps.map(s => `<li>${s}</li>`).join("");
}

initMonth();
render();
</script>

</body>
</html>
