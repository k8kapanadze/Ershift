<!DOCTYPE html>
<html lang="ka">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>NurseFlow OS - Minimal Edition</title>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;600;800&display=swap" rel="stylesheet">
    <style>
        :root {
            --accent: #2563eb;
            --text: #1e293b;
            --shadow: 0 8px 20px rgba(0,0,0,0.05);
            --border: #e2e8f0;
        }

        * { margin: 0; padding: 0; box-sizing: border-box; font-family: 'Inter', sans-serif; }

        body {
            background: #fff;
            min-height: 100vh;
            padding: 15px;
            color: var(--text);
        }

        .container { max-width: 800px; margin: 0 auto; }

        /* Cards */
        .glass-card {
            background: #fff;
            border: 1px solid var(--border);
            border-radius: 20px;
            box-shadow: var(--shadow);
            padding: 20px;
            margin-bottom: 20px;
        }

        .header { text-align: center; margin-bottom: 10px; }
        .header h1 {
            font-size: 1.5rem;
            font-weight: 800;
            color: var(--accent);
            text-transform: uppercase;
            letter-spacing: 1px;
        }

        .control-grid {
            display: grid;
            grid-template-columns: repeat(2, 1fr);
            gap: 12px;
            margin-bottom: 15px;
        }

        .field { display: flex; flex-direction: column; gap: 5px; }
        label {
            font-size: 0.65rem;
            font-weight: 700;
            color: #64748b;
            text-transform: uppercase;
        }

        input, select {
            background: #f9fafb;
            border: 1px solid var(--border);
            border-radius: 12px;
            padding: 12px;
            font-size: 1rem;
            outline: none;
            transition: 0.3s;
        }

        input:focus, select:focus { border-color: var(--accent); }

        .btn {
            border-radius: 12px;
            border: none;
            padding: 14px;
            font-weight: 700;
            cursor: pointer;
            transition: 0.2s;
            box-shadow: 0 4px 0 rgba(0,0,0,0.05);
        }
        .btn:active { transform: translateY(2px); box-shadow: none; }

        .btn-primary { background: var(--accent); color: #fff; width: 100%; }
        .btn-secondary { background: #fff; color: var(--text); border: 1px solid var(--border); }
        .btn-danger { background: #fee2e2; color: #ef4444; border: 1px solid #fca5a5; width: 100%; margin-top: 10px; }

        /* Table */
        .table-wrap { border-radius: 20px; overflow: hidden; box-shadow: var(--shadow); background: #fff; border: 1px solid var(--border);}
        table { width: 100%; border-collapse: collapse; }
        th { background: #f3f4f6; padding: 15px; font-size: 0.7rem; color: #64748b; border-bottom: 1px solid var(--border); }
        td { padding: 10px; text-align: center; border-bottom: 1px solid #f3f4f6; }

        .day-num { font-weight: 700; color: #475569; width: 60px; }

        .plan-select {
            width: 70px;
            padding: 6px;
            border-radius: 8px;
            border: 1px solid var(--border);
            background: #f9fafb;
            font-weight: 700;
            text-align: center;
        }

        .real-val {
            background: #f0f9ff;
            color: var(--accent);
            font-weight: 700;
            border-radius: 10px;
            padding: 8px;
            cursor: pointer;
            display: inline-block;
            min-width: 45px;
        }

        .totals {
            display: flex;
            justify-content: space-around;
            padding: 20px;
            background: #f9fafb;
            border-top: 1px solid var(--border);
        }
        .total-item { text-align: center; }
        .t-label { font-size: 0.65rem; font-weight: 700; color: #64748b; }
        .t-num { display: block; font-size: 1.2rem; font-weight: 800; color: var(--accent); }

        /* Modal */
        .modal { display: none; position: fixed; inset: 0; background: rgba(15, 23, 42, 0.1); z-index: 1000; }
        .modal-content { background: #fff; max-width: 400px; margin: 80px auto; padding: 30px; border-radius: 20px; box-shadow: var(--shadow); }
        textarea { width: 100%; border-radius: 12px; padding: 12px; border: 1px solid var(--border); margin: 15px 0; font-size: 1rem; }

        @media print { .no-print { display: none !important; } }
    </style>
</head>
<body>
<div class="container">
    <div class="header">
        <h1 id="nurseHeading">ექთნის გრაფიკი</h1>
    </div>

    <div class="glass-card no-print">
        <div class="field" style="margin-bottom: 15px;">
            <label>ექთნის სახელი</label>
            <input type="text" id="nurseName" placeholder="შეიყვანეთ სახელი..." oninput="updateHeading()">
        </div>

        <div class="control-grid">
            <div class="field">
                <label>თვე</label>
                <select id="mSelect" onchange="render()"></select>
            </div>
            <div class="field">
                <label>წელი</label>
                <select id="ySelect" onchange="render()"></select>
            </div>
        </div>

        <div class="control-grid">
            <div class="field">
                <label>დღე (რეალური)</label>
                <input type="number" id="realDay" placeholder="1-31">
            </div>
            <div class="field">
                <label>საათი</label>
                <select id="realHours">
                    <option value="24">24</option>
                    <option value="16">16</option>
                    <option value="8">8</option>
                    <option value="0">0</option>
                </select>
            </div>
        </div>

        <div style="display: grid; grid-template-columns: 1fr 1fr; gap: 10px;">
            <button class="btn btn-primary" onclick="saveReal()">დაფიქსირება</button>
            <button class="btn btn-secondary" onclick="fillFuture()">წლის შევსება</button>
        </div>

        <button class="btn btn-danger" onclick="clearFromNow()">გეგმის წაშლა ამ თვიდან წლის ბოლომდე</button>
        <button class="btn btn-secondary" style="width:100%; margin-top:10px;" onclick="window.print()">ბეჭდვა</button>
    </div>

    <div class="table-wrap">
        <table>
            <thead>
                <tr>
                    <th class="day-num">#</th>
                    <th>გეგმა</th>
                    <th>რეალური</th>
                </tr>
            </thead>
            <tbody id="bTable"></tbody>
        </table>
        <div class="totals">
            <div class="total-item">
                <span class="t-label">გეგმიური</span>
                <span id="totalPlan" class="t-num">0</span>
            </div>
            <div class="total-item">
                <span class="t-label">ნამუშევარი</span>
                <span id="totalReal" class="t-num" style="color: #10b981;">0</span>
            </div>
        </div>
    </div>
</div>

<div id="jModal" class="modal">
    <div class="modal-content">
        <h3 id="jTitle"></h3>
        <p style="font-size: 0.8rem; color: #64748b; margin-top: 5px;">რა ისწავლეთ? / როგორ გაერთეთ?</p>
        <textarea id="jText" rows="5"></textarea>
        <div style="display: flex; gap: 10px;">
            <button class="btn btn-secondary" style="flex:1" onclick="closeM()">დახურვა</button>
            <button class="btn btn-primary" style="flex:1" onclick="saveJ()">შენახვა</button>
        </div>
    </div>
</div>

<script>
    let state = JSON.parse(localStorage.getItem('nurse_os_v10')) || { name: "", shifts: {} };
    const months = ["იანვარი","თებერვალი","მარტი","აპრილი","მაისი","ივნისი","ივლისი","აგვისტო","სექტემბერი","ოქტომბერი","ნოემბერი","დეკემბერი"];
    let activeKey = null;

    function init() {
        const ms = document.getElementById('mSelect');
        const ys = document.getElementById('ySelect');
        months.forEach((m,i)=>ms.add(new Option(m,i)));
        for(let y=2026;y<=2030;y++) ys.add(new Option(y,y));
        ms.value = new Date().getMonth();
        ys.value = new Date().getFullYear();
        document.getElementById('nurseName').value = state.name || "";
        updateHeading(); render();
    }

    function updateHeading() {
        const n = document.getElementById('nurseName').value;
        state.name = n;
        document.getElementById('nurseHeading').innerText = n ? `${n}-ის გრაფიკი` : "ექთნის გრაფიკი";
        save();
    }

    function render() {
        const m = parseInt(document.getElementById('mSelect').value);
        const y = parseInt(document.getElementById('ySelect').value);
        const days = new Date(y,m+1,0).getDate();
        let bHTML="", tPlan=0, tReal=0;

        for(let d=1;d<=days;d++){
            const k=`d_${y}_${m}_${d}`;
            const pVal=state.shifts[k]?.plan||"";
            const rVal=state.shifts[k]?.real||"-";
            tPlan+=parseInt(pVal)||0;
            tReal+=parseInt(rVal)||0;

            bHTML+=`
            <tr>
                <td class="day-num">${d}</td>
                <td>
                    <select class="plan-select" onchange="upd('${k}','plan',this.value)">
                        <option value=""></option>
                        <option value="8" ${pVal=='8'?'selected':''}>8</option>
                        <option value="16" ${pVal=='16'?'selected':''}>16</option>
                        <option value="24" ${pVal=='24'?'selected':''}>24</option>
                    </select>
                </td>
                <td><div class="real-val" onclick="openM('${k}',${d})">${rVal}</div></td>
            </tr>`;
        }
        document.getElementById('bTable').innerHTML=bHTML;
        document.getElementById('totalPlan').innerText=tPlan;
        document.getElementById('totalReal').innerText=tReal;
    }

    function saveReal() {
        const d=document.getElementById('realDay').value;
        const h=document.getElementById('realHours').value;
        const m=document.getElementById('mSelect').value;
        const y=document.getElementById('ySelect').value;
        if(!d || d<1 || d>31) return alert("მიუთითეთ რიცხვი");
        const k=`d_${y}_${m}_${d}`;
        if(!state.shifts[k]) state.shifts[k]={};
        state.shifts[k].real=h;
        save(); render();
    }

    function clearFromNow() {
        const currentM=parseInt(document.getElementById('mSelect').value);
        const currentY=document.getElementById('ySelect').value;
        if(confirm(`${months[currentM]} თვიდან წლის ბოლომდე ყველა გეგმა წაიშლება. დარწმუნებული ხართ?`)){
            for(let m=currentM;m<=11;m++){
                for(let d=1;d<=31;d++){
                    const k=`d_${currentY}_${m}_${d}`;
                    if(state.shifts[k]) state.shifts[k].plan="";
                }
            }
            save(); render();
        }
    }

    function fillFuture() {
        const m=parseInt(document.getElementById('mSelect').value);
        const y=parseInt(document.getElementById('ySelect').value);
        let startD=null, hrs=24;
        for(let d=1;d<=31;d++){
            if(state.shifts[`d_${y}_${m}_${d}`]?.plan){
                startD=new Date(y,m,d);
                hrs=state.shifts[`d_${y}_${m}_${d}`].plan;
                break;
            }
        }
        if(startD){
            let c=new Date(startD);
            while(c.getFullYear()<=2030){
                const k=`d_${c.getFullYear()}_${c.getMonth()}_${c.getDate()}`;
                if(!state.shifts[k]) state.shifts[k]={};
                state.shifts[k].plan=hrs;
                c.setDate(c.getDate()+4);
            }
            save(); render(); alert("გრაფიკი შეივსო 2030 წლამდე!");
        }else{
            alert("ჯერ მონიშნეთ ერთ-ერთი დღის გეგმა (8,16,24) ცხრილში!");
        }
    }

    function upd(k,f,v){ if(!state.shifts[k]) state.shifts[k]={}; state.shifts[k][f]=v; save(); render(); }
    function openM(k,d){ activeKey=k; document.getElementById('jTitle').innerText=`${d} რიცხვის ჟურნალი`; document.getElementById('jText').value=state.shifts[k]?.note||""; document.getElementById('jModal').style.display='block'; }
    function saveJ(){ if(!state.shifts[activeKey]) state.shifts[activeKey]={}; state.shifts[activeKey].note=document.getElementById('jText').value; save(); render(); closeM(); }
    function closeM(){ document.getElementById('jModal').style.display='none'; }
    function save(){ localStorage.setItem('nurse_os_v10',JSON.stringify(state)); }

    init();
</script>
</body>
</html>
