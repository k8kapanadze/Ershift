
<html lang="ka">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ChronoCare ER - Smart Scheduler</title>
    <style>
        :root { --primary: #2c3e50; --secondary: #27ae60; --accent: #e74c3c; --bg: #f8f9fa; }
        body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; margin: 0; background: var(--bg); font-size: 13px; }
        .header { background: var(--primary); color: white; padding: 15px 20px; box-shadow: 0 2px 10px rgba(0,0,0,0.2); }
        .toolbar { display: flex; flex-wrap: wrap; gap: 15px; padding: 20px; background: white; border-bottom: 1px solid #ddd; position: sticky; top: 0; z-index: 100; }
        
        .search-group, .action-group { display: flex; align-items: center; gap: 8px; }
        input, select, button { padding: 8px 12px; border: 1px solid #ccc; border-radius: 5px; outline: none; }
        button { cursor: pointer; background: var(--secondary); color: white; border: none; font-weight: bold; transition: 0.3s; }
        button:hover { opacity: 0.8; }
        .btn-gen { background: #2980b9; }

        .table-container { overflow: auto; height: calc(100vh - 180px); padding: 0 20px; }
        table { border-collapse: collapse; background: white; width: 100%; box-shadow: 0 0 20px rgba(0,0,0,0.05); }
        th, td { border: 1px solid #dfe6e9; padding: 4px; text-align: center; min-width: 60px; }
        th { background: #f1f2f6; position: sticky; top: 0; z-index: 10; }
        .name-col { position: sticky; left: 0; background: #fff; z-index: 11; min-width: 180px; text-align: left; font-weight: bold; }
        
        .shift-cell { display: flex; flex-direction: column; gap: 2px; }
        .plan { color: #2980b9; font-size: 10px; border-bottom: 1px dashed #eee; }
        .fact { font-weight: bold; color: #2c3e50; }
        .total-col { background: #e8f8f5; font-weight: bold; color: var(--secondary); }
        
        .nurse-row:nth-child(even) .name-col { background: #f9f9f9; }
        .nurse-row:hover { background: #f1f2f6; }
        .highlight-day { background: #fff9c4 !important; }
    </style>
</head>
<body>

<div class="header">
    <h1>✨ ChronoCare ER <span style="font-size: 14px; font-weight: normal; opacity: 0.8;">| 51 Nurses Management System</span></h1>
</div>

<div class="toolbar">
    <div class="search-group">
        <strong>🔍 ძებნა:</strong>
        <input type="text" id="searchName" placeholder="ექთნის გვარი..." onkeyup="filterSystem()">
        <input type="number" id="searchDate" min="1" max="31" placeholder="თარიღი..." oninput="filterSystem()">
    </div>
    
    <div class="action-group">
        <strong>⚙️ მოქმედება:</strong>
        <button class="btn-gen" onclick="generateAutoSchedule()">ავტ. შევსება (ყოველ მე-4 დღეს)</button>
        <button onclick="saveData()" style="background: #8e44ad;">💾 შენახვა</button>
        <button onclick="resetData()" style="background: var(--accent);">🔄 გასუფთავება</button>
    </div>
</div>

<div class="table-container">
    <table id="mainTable">
        <thead>
            <tr id="topHeader">
                <th class="name-col">პერსონალი (51)</th>
                <th class="total-col">ჯამი (სთ)</th>
            </tr>
        </thead>
        <tbody id="tableBody"></tbody>
    </table>
</div>

<script>
    const NURSE_COUNT = 51;
    const DAYS = 31;
    let nurseData = JSON.parse(localStorage.getItem('chronoCareData')) || [];

    // მონაცემების ინიციალიზაცია თუ ცარიელია
    if (nurseData.length === 0) {
        for (let i = 1; i <= NURSE_COUNT; i++) {
            nurseData.push({
                id: i,
                name: `ექთანი ${i}`,
                shifts: Array(DAYS).fill({ plan: 0, fact: 0, actualWorker: "" })
            });
        }
    }

    function renderTable() {
        const header = document.getElementById('topHeader');
        const body = document.getElementById('tableBody');
        
        // თარიღების დახატვა
        if (header.children.length <= 2) {
            for (let d = 1; d <= DAYS; d++) {
                let th = document.createElement('th');
                th.innerText = d;
                header.insertBefore(th, header.lastElementChild);
            }
        }

        body.innerHTML = '';
        nurseData.forEach((nurse, nIdx) => {
            let tr = document.createElement('tr');
            tr.className = 'nurse-row';
            let totalHours = 0;

            let nameTd = `<td class="name-col"><input type="text" value="${nurse.name}" onchange="updateName(${nIdx}, this.value)"></td>`;
            let daysHtml = '';

            nurse.shifts.forEach((shift, dIdx) => {
                // თუ ფაქტიური მუშაკი სხვაა, საათები მასთან უნდა წავიდეს (ამ ლოგიკას ჯამში ვითვლით)
                totalHours += Number(shift.fact);
                
                daysHtml += `<td>
                    <div class="shift-cell">
                        <select class="plan" onchange="updateShift(${nIdx}, ${dIdx}, 'plan', this.value)">
                            <option value="0" ${shift.plan==0?'selected':''}>-</option>
                            <option value="8" ${shift.plan==8?'selected':''}>8</option>
                            <option value="16" ${shift.plan==16?'selected':''}>16</option>
                            <option value="24" ${shift.plan==24?'selected':''}>24</option>
                        </select>
                        <select class="fact" onchange="updateShift(${nIdx}, ${dIdx}, 'fact', this.value)">
                            <option value="0" ${shift.fact==0?'selected':''}>0</option>
                            <option value="8" ${shift.fact==8?'selected':''}>8</option>
                            <option value="16" ${shift.fact==16?'selected':''}>16</option>
                            <option value="24" ${shift.fact==24?'selected':''}>24</option>
                        </select>
                    </div>
                </td>`;
            });

            tr.innerHTML = nameTd + daysHtml + `<td class="total-col">${totalHours}</td>`;
            body.appendChild(tr);
        });
    }

    function updateName(nIdx, val) { nurseData[nIdx].name = val; }
    
    function updateShift(nIdx, dIdx, type, val) {
        nurseData[nIdx].shifts[dIdx][type] = parseInt(val);
        renderTable(); // განახლება ჯამების დასათვლელად
    }

    function generateAutoSchedule() {
        if(!confirm("გსურთ ყოველ მე-4 დღეს გეგმიური მორიგეობების დაგენერირება?")) return;
        nurseData.forEach((nurse, index) => {
            let startDay = index % 4; // რომ ყველა ერთ დღეს არ იყოს მორიგე
            for (let d = 0; d < DAYS; d++) {
                if ((d % 4) === startDay) {
                    nurse.shifts[d].plan = 24;
                    nurse.shifts[d].fact = 24; // თავიდან ფაქტი გეგმას მიჰყვება
                }
            }
        });
        renderTable();
    }

    function filterSystem() {
        let nameInp = document.getElementById('searchName').value.toLowerCase();
        let dateInp = document.getElementById('searchDate').value;
        let rows = document.querySelectorAll('.nurse-row');

        rows.forEach((row, idx) => {
            let name = nurseData[idx].name.toLowerCase();
            let show = name.includes(nameInp);
            row.style.display = show ? "" : "none";
            
            // თარიღის ჰაილაითი
            let cells = row.querySelectorAll('td');
            cells.forEach(c => c.classList.remove('highlight-day'));
            if(dateInp && dateInp > 0 && dateInp <= DAYS) {
                cells[dateInp].classList.add('highlight-day');
            }
        });
    }

    function saveData() {
        localStorage.setItem('chronoCareData', JSON.stringify(nurseData));
        alert("მონაცემები წარმატებით შეინახა!");
    }

    function resetData() {
        if(confirm("დარწმუნებული ხართ? სრულიად წაიშლება მონაცემები!")) {
            localStorage.removeItem('chronoCareData');
            location.reload();
        }
    }

    window.onload = renderTable;
</script>

</body>
</html>
