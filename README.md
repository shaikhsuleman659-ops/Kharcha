<!DOCTYPE html>
<html lang="hi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Roj Ka Kharcha</title>
    <style>
        :root { --primary: #2563eb; --bg: #f8fafc; --text: #1e293b; }
        body { font-family: -apple-system, sans-serif; background: var(--bg); color: var(--text); padding: 15px; margin: 0; }
        .container { max-width: 500px; margin: auto; }
        
        /* Dashboard */
        .dashboard { background: linear-gradient(135deg, #2563eb, #1d4ed8); color: white; padding: 25px; border-radius: 20px; box-shadow: 0 10px 20px rgba(37,99,235,0.2); margin-bottom: 20px; text-align: center; }
        .dashboard h1 { margin: 0; font-size: 32px; }
        .dashboard p { opacity: 0.9; margin: 5px 0 0; }

        /* Input Form */
        .card { background: white; padding: 20px; border-radius: 15px; box-shadow: 0 2px 10px rgba(0,0,0,0.05); margin-bottom: 15px; }
        input, select, textarea { width: 100%; padding: 12px; margin: 8px 0; border: 1px solid #e2e8f0; border-radius: 10px; box-sizing: border-box; font-size: 16px; }
        
        .btn { width: 100%; padding: 14px; border: none; border-radius: 10px; font-weight: bold; font-size: 16px; cursor: pointer; transition: 0.2s; }
        .btn-add { background: #10b981; color: white; margin-top: 5px; }
        .btn-import { background: #64748b; color: white; margin-top: 10px; }
        
        /* Expense List */
        .date-section { margin-top: 20px; font-weight: bold; color: #64748b; border-bottom: 2px solid #e2e8f0; padding-bottom: 5px; margin-bottom: 10px; display: flex; justify-content: space-between; }
        .expense-item { background: white; padding: 12px 15px; border-radius: 10px; margin-bottom: 8px; display: flex; justify-content: space-between; align-items: center; border: 1px solid #f1f5f9; }
        .exp-info b { display: block; color: #334155; }
        .exp-info small { color: #94a3b8; font-size: 12px; }
        .exp-amt { font-weight: bold; color: #ef4444; }
        
        #import-area { display: none; }
    </style>
</head>
<body>

<div class="container">
    <div class="dashboard">
        <p>Kul Kharcha (Total)</p>
        <h1 id="total-val">₹0</h1>
    </div>

    <div class="card">
        <h3 style="margin-top:0">Naya Kharcha</h3>
        <input type="date" id="new-date">
        <input type="text" id="new-item" placeholder="Item ka naam (e.g. Doodh)">
        <input type="number" id="new-amt" placeholder="Amount ₹">
        <button class="btn btn-add" onclick="addNew()">Add Karein</button>
        <button class="btn btn-import" onclick="document.getElementById('import-area').style.display='block'">Purana Data Paste Karein</button>
    </div>

    <div class="card" id="import-area">
        <h3>Paste Old Data</h3>
        <textarea id="bulk-data" rows="6" placeholder="Apna purana text yahan paste karein..."></textarea>
        <button class="btn btn-add" onclick="processImport()">Data Import Karein</button>
    </div>

    <div id="history"></div>
</div>

<script>
    let expenses = JSON.parse(localStorage.getItem('kharcha_v5')) || [];
    document.getElementById('new-date').valueAsDate = new Date();

    function render() {
        const historyDiv = document.getElementById('history');
        const totalDiv = document.getElementById('total-val');
        historyDiv.innerHTML = '';
        let grandTotal = 0;

        // Sorting by date
        expenses.sort((a, b) => new Date(b.date) - new Date(a.date));

        const grouped = expenses.reduce((acc, obj) => {
            const d = obj.date;
            if (!acc[d]) acc[d] = [];
            acc[d].push(obj);
            return acc;
        }, {});

        for (let date in grouped) {
            let dayTotal = 0;
            let dateTitle = new Date(date).toLocaleDateString('hi-IN', { day:'numeric', month:'short', year:'numeric'});
            let sectionHtml = `<div class="date-section"><span>${dateTitle}</span><span id="day-total-${date}"></span></div>`;
            
            grouped[date].forEach(item => {
                dayTotal += parseFloat(item.amount);
                grandTotal += parseFloat(item.amount);
                sectionHtml += `
                    <div class="expense-item">
                        <div class="exp-info"><b>${item.item}</b></div>
                        <div class="exp-amt">₹${item.amount}</div>
                    </div>`;
            });

            historyDiv.innerHTML += sectionHtml;
            document.getElementById(`day-total-${date}`).innerText = "₹" + dayTotal;
        }
        totalDiv.innerText = "₹" + grandTotal.toLocaleString('hi-IN');
        localStorage.setItem('kharcha_v5', JSON.stringify(expenses));
    }

    function addNew() {
        const date = document.getElementById('new-date').value;
        const item = document.getElementById('new-item').value;
        const amount = document.getElementById('new-amt').value;
        if(date && item && amount) {
            expenses.push({ date, item, amount });
            document.getElementById('new-item').value = '';
            document.getElementById('new-amt').value = '';
            render();
        }
    }

    function processImport() {
        const text = document.getElementById('bulk-data').value;
        const lines = text.split('\n');
        let currentD = "";

        lines.forEach(line => {
            line = line.trim();
            if(!line) return;

            // Simple date detection logic
            const isDate = line.match(/\d{1,2}\s*(nov|dec|jan|feb)/i);
            if(isDate && !line.toLowerCase().includes('total')) {
                // Convert text date to ISO format
                let day = line.match(/\d+/)[0];
                let month = line.toLowerCase().includes('nov') ? '11' : '12';
                currentD = `2025-${month}-${day.padStart(2, '0')}`;
            } else {
                const amtMatch = line.match(/(\d+(\.\d+)?)/);
                if(amtMatch && !line.toLowerCase().includes('total')) {
                    const amt = amtMatch[0];
                    const name = line.replace(amt, '').replace('=', '').trim() || "Kharcha";
                    if(currentD) expenses.push({ date: currentD, item: name, amount: amt });
                }
            }
        });
        document.getElementById('import-area').style.display = 'none';
        render();
    }

    render();
</script>
</body>
</html>

