<!DOCTYPE html>
<html lang="he" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ניחוש תוצאות הבחירות 2026</title>
    <style>
        :root {
            --bg-color: #f8f9fa;
            --card-bg: #ffffff;
            --text-color: #212529;
            --border-color: #dee2e6;
            --primary-color: #0d6efd;
            --success-color: #198754;
            --danger-color: #dc3545;
        }

        body {
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Arial, sans-serif;
            background-color: var(--bg-color);
            color: var(--text-color);
            margin: 0;
            padding: 15px;
            display: flex;
            justify-content: center;
        }

        .container {
            width: 100%;
            max-width: 800px;
            background: var(--card-bg);
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 2px 8px rgba(0,0,0,0.05);
            border: 1px solid var(--border-color);
        }

        h1, h2, h3 {
            text-align: center;
            margin-bottom: 15px;
            font-weight: 600;
        }

        .form-group {
            margin-bottom: 15px;
        }

        label {
            display: block;
            margin-bottom: 6px;
            font-weight: 500;
        }

        input[type="text"], input[type="password"], input[type="number"] {
            width: 100%;
            padding: 10px;
            border: 1px solid var(--border-color);
            border-radius: 4px;
            box-sizing: border-box;
            font-size: 16px;
        }

        .parties-grid {
            display: grid;
            grid-template-columns: repeat(auto-fill, minmax(160px, 1fr));
            gap: 12px;
            margin-bottom: 20px;
        }

        .party-card {
            padding: 10px;
            border: 1px solid var(--border-color);
            border-radius: 4px;
            background: #fafafa;
        }

        .party-card label {
            font-size: 13px;
        }

        .counter-box {
            position: sticky;
            top: 10px;
            background: #e9ecef;
            padding: 12px;
            border-radius: 4px;
            text-align: center;
            font-size: 18px;
            font-weight: bold;
            margin-bottom: 20px;
            z-index: 10;
        }

        .counter-box.valid { color: var(--success-color); }
        .counter-box.invalid { color: var(--danger-color); }

        button {
            width: 100%;
            padding: 12px;
            background-color: var(--primary-color);
            color: white;
            border: none;
            border-radius: 4px;
            font-size: 16px;
            cursor: pointer;
            font-weight: 600;
        }

        button:hover { opacity: 0.9; }

        .results-box {
            margin-top: 25px;
            padding: 15px;
            background: #f1f3f5;
            border-radius: 4px;
            display: none;
        }

        .blocks-summary {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(130px, 1fr));
            gap: 10px;
            margin-top: 15px;
        }

        .block-card {
            background: white;
            padding: 10px;
            border-radius: 4px;
            text-align: center;
            border: 1px solid var(--border-color);
        }

        .admin-section {
            margin-top: 40px;
            border-top: 2px solid var(--border-color);
            padding-top: 20px;
        }

        table {
            width: 100%;
            border-collapse: collapse;
            margin-top: 15px;
            font-size: 13px;
        }

        th, td {
            border: 1px solid var(--border-color);
            padding: 6px;
            text-align: right;
        }

        th { background-color: #f1f3f5; }

        .admin-login {
            display: flex;
            gap: 10px;
        }

        .admin-login input { flex: 1; }
        .admin-login button { width: auto; }
    </style>
</head>
<body>

<div class="container">
    <h1>ניחוש בחירות 2026</h1>

    <div id="counterBox" class="counter-box invalid">
        סה"כ מנדטים: <span id="totalSeats">0</span> / 120
    </div>

    <form id="predictionForm">
        <div class="form-group">
            <label for="userName">שם פרטי:</label>
            <input type="text" id="userName" required placeholder="הכנס את שמך">
        </div>

        <h3>חלוקת מנדטים (בחירות 2026):</h3>
        <div class="parties-grid" id="partiesGrid"></div>

        <button type="submit">שלח ניחוש</button>
    </form>

    <div id="resultsBox" class="results-box">
        <h3>חלוקה לגושים לפי הניחוש שלך:</h3>
        <div class="blocks-summary" id="blocksSummary"></div>
    </div>

    <div class="admin-section">
        <h2>אזור מנהל מערכת</h2>
        <div id="adminAuth" class="admin-login">
            <input type="password" id="adminPassword" placeholder="סיסמת מנהל">
            <button type="button" onclick="loginAdmin()">התחבר</button>
        </div>

        <div id="adminPanel" style="display: none;">
            <h3>כל ההצבעות שנשמרו</h3>
            <div style="overflow-x: auto;">
                <table id="votesTable">
                    <thead>
                        <tr id="tableHeader">
                            <th>שם</th>
                            <th>זמן</th>
                        </tr>
                    </thead>
                    <tbody id="tableBody"></tbody>
                </table>
            </div>
            <button type="button" onclick="clearData()" style="margin-top: 15px; background-color: var(--danger-color);">מחק את כל הנתונים</button>
        </div>
    </div>
</div>

<script>
    // מפלגות בחירות 2026 העדכניות + המפלגות שביקשת
    const parties = [
        { id: 'yashar', name: 'ישר! (איזנקוט)', block: 'גוש השינוי / אופוזיציה' },
        { id: 'likud', name: 'הליכוד', block: 'גוש הימין / קואליציה נוכחית' },
        { id: 'beiyachad', name: 'ביחד (בנט-לפיד)', block: 'גוש השינוי / אופוזיציה' },
        { id: 'democrats', name: 'הדמוקרטים', block: 'גוש השינוי / אופוזיציה' },
        { id: 'utj', name: 'יהדות התורה', block: 'גוש הימין / קואליציה נוכחית' },
        { id: 'shas', name: 'ש"ס', block: 'גוש הימין / קואליציה נוכחית' },
        { id: 'israelBeiteinu', name: 'ישראל ביתנו', block: 'גוש השינוי / אופוזיציה' },
        { id: 'otzma', name: 'עוצמה יהודית', block: 'גוש הימין / קואליציה נוכחית' },
        { id: 'zionism', name: 'הציונות הדתית וזהות', block: 'גוש הימין / קואליציה נוכחית' },
        { id: 'raam', name: 'רע"ם', block: 'מפלגות ערביות' },
        { id: 'hadashTaal', name: 'חד"ש-תע"ל', block: 'מפלגות ערביות' },
        { id: 'jointList', name: 'הרשימה המשותפת', block: 'מפלגות ערביות' },
        { id: 'vinter', name: 'עמך ישראל (עופר וינטר)', block: 'מפלגות חדשות / שונות' },
        { id: 'yoazlica', name: 'יועז הנדל וירון זליכה', block: 'מפלגות חדשות / שונות' },
        { id: 'noam', name: 'נועם', block: 'גוש הימין / קואליציה נוכחית' },
        { id: 'harediPublic', name: 'הציבור החרדי', block: 'מפלגות חדשות / שונות' },
        { id: 'blackColor', name: 'צבע שחור', block: 'מפלגות חדשות / שונות' }
    ];

    const partiesGrid = document.getElementById('partiesGrid');
    parties.forEach(party => {
        const div = document.createElement('div');
        div.className = 'party-card';
        div.innerHTML = `
            <label for="${party.id}">${party.name}</label>
            <input type="number" id="${party.id}" name="${party.id}" min="0" max="120" value="0" class="party-input">
        `;
        partiesGrid.appendChild(div);
    });

    const partyInputs = document.querySelectorAll('.party-input');
    const totalSeatsEl = document.getElementById('totalSeats');
    const counterBox = document.getElementById('counterBox');

    function calculateTotal() {
        let sum = 0;
        partyInputs.forEach(input => {
            sum += parseInt(input.value) || 0;
        });
        totalSeatsEl.textContent = sum;

        if (sum === 120) {
            counterBox.className = 'counter-box valid';
        } else {
            counterBox.className = 'counter-box invalid';
        }
        return sum;
    }

    partyInputs.forEach(input => {
        input.addEventListener('input', calculateTotal);
    });

    document.getElementById('predictionForm').addEventListener('submit', function(e) {
        e.preventDefault();
        
        const total = calculateTotal();
        if (total !== 120) {
            alert(`סך המנדטים חייב להיות בדיוק 120. כרגע הזינות ${total} מנדטים.`);
            return;
        }

        const userName = document.getElementById('userName').value.trim();
        if (!userName) {
            alert('אנא הזן שם פרטי.');
            return;
        }

        const currentPrediction = {
            name: userName,
            date: new Date().toLocaleString('he-IL'),
            votes: {},
            blocks: {}
        };

        parties.forEach(party => {
            const count = parseInt(document.getElementById(party.id).value) || 0;
            currentPrediction.votes[party.id] = count;

            if (!currentPrediction.blocks[party.block]) {
                currentPrediction.blocks[party.block] = 0;
            }
            currentPrediction.blocks[party.block] += count;
        });

        const existingVotes = JSON.parse(localStorage.getItem('electionPredictions2026') || '[]');
        existingVotes.push(currentPrediction);
        localStorage.setItem('electionPredictions2026', JSON.stringify(existingVotes));

        displayUserBlocks(currentPrediction.blocks);
        alert('הניחוש שלך נשמר בהצלחה!');

        if (document.getElementById('adminPanel').style.display === 'block') {
            loadAdminTable();
        }
    });

    function displayUserBlocks(blocks) {
        const resultsBox = document.getElementById('resultsBox');
        const blocksSummary = document.getElementById('blocksSummary');
        blocksSummary.innerHTML = '';

        for (const [blockName, count] of Object.entries(blocks)) {
            const blockCard = document.createElement('div');
            blockCard.className = 'block-card';
            blockCard.innerHTML = `
                <strong>${blockName}</strong>
                <div style="font-size: 18px; margin-top: 5px;">${count} מנדטים</div>
            `;
            blocksSummary.appendChild(blockCard);
        }

        resultsBox.style.display = 'block';
    }

    function loginAdmin() {
        const pass = document.getElementById('adminPassword').value;
        if (pass === '10101010') {
            document.getElementById('adminAuth').style.display = 'none';
            document.getElementById('adminPanel').style.display = 'block';
            loadAdminTable();
        } else {
            alert('סיסמה שגויה!');
        }
    }

    function loadAdminTable() {
        const votesData = JSON.parse(localStorage.getItem('electionPredictions2026') || '[]');
        const tableHeader = document.getElementById('tableHeader');
        const tableBody = document.getElementById('tableBody');

        tableHeader.innerHTML = '<th>שם</th><th>זמן</th>';
        parties.forEach(party => {
            const th = document.createElement('th');
            th.textContent = party.name;
            tableHeader.appendChild(th);
        });

        tableBody.innerHTML = '';
        votesData.forEach(row => {
            const tr = document.createElement('tr');
            tr.innerHTML = `<td>${row.name}</td><td>${row.date}</td>`;
            
            parties.forEach(party => {
                const td = document.createElement('td');
                td.textContent = row.votes[party.id] || 0;
                tr.appendChild(td);
            });

            tableBody.appendChild(tr);
        });
    }

    function clearData() {
        if (confirm('האם למחוק את כל ההצבעות שנשמרו בטלפון?')) {
            localStorage.removeItem('electionPredictions2026');
            loadAdminTable();
        }
    }
</script>

</body>
</html>
