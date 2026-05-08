<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SCHOOL MONEY - ระบบแจ้งค่าใช้จ่าย</title>
    <link href="https://fonts.googleapis.com/css2?family=Kanit:wght@300;400;500;600&display=swap" rel="stylesheet">
    <script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-app.js"></script>
    <script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-database.js"></script>
    <style>
        :root { --primary: #1e3a8a; --border: #cbd5e1; --text: #334155; --bg: #f8fafc; }
        body { font-family: 'Kanit', sans-serif; background-color: #f1f5f9; margin: 0; padding: 20px; color: var(--text); }
        .container { max-width: 700px; margin: auto; background: white; padding: 40px; border-radius: 8px; box-shadow: 0 4px 6px -1px rgb(0 0 0 / 0.1); }
        
        /* Header */
        .header { display: flex; align-items: center; gap: 20px; margin-bottom: 30px; }
        .logo-circle { width: 80px; height: 80px; border: 2px solid var(--primary); border-radius: 50%; display: flex; align-items: center; justify-content: center; font-weight: bold; color: var(--primary); }
        .school-name h2 { margin: 0; color: var(--primary); font-size: 24px; }
        .school-name p { margin: 5px 0 0; font-size: 18px; }

        /* Search Section */
        .search-box { display: flex; gap: 10px; margin-bottom: 30px; border-bottom: 2px solid #eee; padding-bottom: 20px; }
        input { flex: 1; padding: 12px; border: 1px solid var(--border); border-radius: 8px; font-size: 16px; font-family: 'Kanit'; }
        button.btn-search { background: var(--primary); color: white; border: none; padding: 12px 25px; border-radius: 8px; cursor: pointer; font-weight: 500; }

        /* Info Card Section */
        .report-section { display: none; animation: fadeIn 0.5s ease; }
        .info-card { border: 2px solid var(--primary); border-radius: 8px; padding: 20px; margin-bottom: 25px; line-height: 1.8; }
        .info-card div { font-size: 18px; }
        .label { display: inline-block; width: 150px; font-weight: 500; }

        h3.section-title { font-size: 20px; margin: 30px 0 15px; color: #444; }
        h3.section-title span { font-size: 14px; font-weight: 300; color: #777; }

        /* Dorm Fee Card */
        .dorm-fee-card { border: 2px solid var(--primary); border-radius: 8px; padding: 15px 20px; display: flex; justify-content: space-between; align-items: center; margin-bottom: 25px; }
        .fee-title { font-weight: 500; font-size: 18px; }
        .fee-details { display: flex; gap: 30px; align-items: center; }

        /* Boarding Fee Table */
        .boarding-card { border: 2px solid var(--primary); border-radius: 8px; padding: 0; overflow: hidden; margin-bottom: 25px; }
        .boarding-title { padding: 15px 20px; border-bottom: 1px solid #eee; font-weight: 500; font-size: 18px; }
        .month-grid { display: grid; grid-template-columns: repeat(6, 1fr); border-bottom: 1px solid #eee; }
        .month-item { padding: 15px 5px; text-align: center; border-right: 1px solid #eee; }
        .month-item:last-child { border-right: none; }
        .month-name { font-weight: 500; margin-bottom: 10px; }
        
        /* Status Badges */
        .status { padding: 3px 12px; border-radius: 4px; font-size: 14px; font-weight: 400; border: 1px solid; }
        .paid { border-color: #22c55e; color: #166534; background: #f0fdf4; }
        .unpaid { border-color: #ef4444; color: #991b1b; background: #fef2f2; }

        .total-row { padding: 20px; display: flex; justify-content: space-between; font-size: 18px; font-weight: 600; }

        /* Footer Buttons */
        .footer-action { display: flex; justify-content: space-between; align-items: center; margin-top: 40px; }
        .btn-upload { border: 2px solid #ec4899; color: #be185d; background: white; padding: 12px 40px; border-radius: 4px; cursor: pointer; font-size: 16px; font-weight: 500; }
        .wait-text { color: #f59e0b; text-align: right; }
        .wait-text strong { display: block; font-size: 18px; }
        .wait-text span { font-size: 14px; color: #666; }

        .bottom-quote { text-align: center; margin-top: 60px; color: #666; font-size: 14px; }

        @keyframes fadeIn { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }
    </style>
</head>
<body>

<div class="container">
    <div class="header">
        <div class="logo-circle">โลโก้</div>
        <div class="school-name">
            <h2>SCHOOL MONEY</h2>
            <p>ระบบแจ้งจ่ายค่าใช้จ่ายโรงเรียนอุทยานศึกษากระบี่</p>
        </div>
    </div>

    <div class="search-box">
        <input type="text" id="nameInput" placeholder="กรอกชื่อ-นามสกุล นักเรียน...">
        <button class="btn-search" onclick="doSearch()">ค้นหา</button>
    </div>

    <div id="loading" style="display:none; text-align:center;">กำลังค้นหา...</div>

    <div id="reportArea" class="report-section">
        <div class="info-card">
            <div><span class="label">ชื่อ - สกุล :</span> <span id="resName"></span></div>
            <div><span class="label">โปรแกรมหอพัก :</span> <span id="resProg"></span></div>
            <div><span class="label">ระดับชั้น :</span> <span id="resLevel"></span></div>
        </div>

        <h3 class="section-title">รายงานประวัติและยอดค้างชำระ <span>( Payment History & Outstanding Balance )</span></h3>

        <div class="dorm-fee-card">
            <div class="fee-title">ค่าธรรมเนียมหอพัก 1/2569 <span>( Dormitory Fee )</span></div>
            <div class="fee-details">
                <div>สถานะ <span id="resFeeStatus"></span></div>
                <div style="text-align:right;">จำนวนเงิน<br><strong id="resFeeAmt"></strong> บาท</div>
            </div>
        </div>

        <div class="boarding-card">
            <div class="boarding-title">ค่าอาหารรายเดือน 1/2569 <span>( Boarding Fee )</span></div>
            <div class="month-grid">
                <div class="month-item"><div class="month-name">พฤษภาคม</div><div id="m1"></div></div>
                <div class="month-item"><div class="month-name">มิถุนายน</div><div id="m2"></div></div>
                <div class="month-item"><div class="month-name">กรกฎาคม</div><div id="m3"></div></div>
                <div class="month-item"><div class="month-name">สิงหาคม</div><div id="m4"></div></div>
                <div class="month-item"><div class="month-name">กันยายน</div><div id="m5"></div></div>
                <div class="month-item"><div class="month-name">ตุลาคม</div><div id="m6"></div></div>
            </div>
            <div class="total-row">
                <span>ยอดที่ต้องชำระทั้งหมด <span>( Total Outstanding Balance )</span></span>
                <span id="resTotal"></span>
            </div>
        </div>

        <div class="footer-action">
            <button class="btn-upload" onclick="document.getElementById('fileIn').click()">แนบสลิปการจ่าย</button>
            <input type="file" id="fileIn" style="display:none" onchange="handleUpload()">
            
            <div class="wait-text">
                <strong>รอตรวจสอบสถานะการจ่าย</strong>
                <span>โดยปกติแล้วรอตรวจสอบสถานะ 7 วันทำการ</span>
            </div>
        </div>
    </div>

    <div class="bottom-quote">
        พบปัญหาหรือมีข้อสงสัยโปรดติดต่อ 078 - 789 - 6789<br>
        “ เรียนดี ประพฤติเด่น เน้นคุณภาพ ซึมซาบคุุณธรรม ถูกสัมพันธ์ชุมชน “
    </div>
</div>

<script>
    // --- ตั้งค่า Firebase ---
    const firebaseConfig = {
        apiKey: "AIzaSyDo7qG7z3z0-Sq2iFEr9POsdLcg4mB66sU",
        authDomain: "schoolmonny.firebaseapp.com",
        databaseURL: "https://schoolmonny-default-rtdb.asia-southeast1.firebasedatabase.app",
        projectId: "schoolmonny",
        appId: "1:629430537095:web:d8bfda78ed827660fae122"
    };
    firebase.initializeApp(firebaseConfig);
    const db = firebase.database();

    let userData = null;
    let userSheetName = "";

    async function doSearch() {
        const name = document.getElementById('nameInput').value.trim();
        if(!name) return alert("กรุณาระบุชื่อนักเรียน");

        document.getElementById('loading').style.display = 'block';
        document.getElementById('reportArea').style.display = 'none';

        const programs = ["หอพักญะมาอะห์ชาย", "หอพักญะมาอะห์หญิง", "หอพักกีฬา", "หอพักฮาฟิซอัลกุรอ่าน"];
        let found = false;

        for (let p of programs) {
            const snap = await db.ref(p).once('value');
            const data = snap.val();
            if(data) {
                // ค้นหาข้อมูลจาก "ชื่อ-สกุล" ใน Firebase ของคุณ
                const match = data.find(s => (s["ชื่อ-สกุล"] === name || s["ชื่อ-นามสกุล"] === name));
                if(match) {
                    userData = match;
                    userSheetName = p;
                    renderReport(match, p);
                    found = true;
                    break;
                }
            }
        }
        document.getElementById('loading').style.display = 'none';
        if(!found) alert("ไม่พบข้อมูลนักเรียน กรุณาตรวจสอบการสะกดชื่อ-นามสกุล");
    }

    function renderReport(s, prog) {
        document.getElementById('reportArea').style.display = 'block';
        document.getElementById('resName').innerText = s["ชื่อ-สกุล"] || s["ชื่อ-นามสกุล"];
        document.getElementById('resProg').innerText = prog;
        document.getElementById('resLevel').innerText = "มัธยมศึกษาปีที่ " + (s["ระดับชั้นมัธยมศึกษาปีที่"] || "-");

        // Dorm Fee
        const feeStatus = s["ค่าธรรมเนียมหอพัก-สถานะ"] || s["ค่าธรรมเนียมหอพัก สถานะ"];
        document.getElementById('resFeeStatus').innerHTML = `<span class="status ${feeStatus === 'ชำระแล้ว' ? 'paid' : 'unpaid'}">${feeStatus}</span>`;
        document.getElementById('resFeeAmt').innerText = s["ยอดค้างค่าธรรมเนียม"] || "0";

        // Boarding Fee Grid (เช็ครายเดือน)
        const months = ["พฤษภาคม", "มิถุนายน", "กรกฎาคม", "สิงหาคม", "กันยายน", "ตุลาคม"];
        months.forEach((m, index) => {
            const status = s[m] || "ค้างชำระ";
            document.getElementById(`m${index+1}`).innerHTML = `<span class="status ${status === 'ชำระแล้ว' ? 'paid' : 'unpaid'}">${status}</span>`;
        });

        document.getElementById('resTotal').innerText = (s["ยอดรวมค้างชำระ ( รวมห้าค่าอาหารรายเดือน )"] || 0) + " บาท";
    }

    function handleUpload() {
        const file = document.getElementById('fileIn').files[0];
        if(!file) return;

        const reader = new FileReader();
        reader.onload = function(e) {
            const payload = {
                studentName: userData["ชื่อ-สกุล"] || userData["ชื่อ-นามสกุล"],
                sheetName: userSheetName,
                fileName: `SLIP_${Date.now()}.png`,
                fileData: e.target.result,
                fileType: file.type
            };

            const WEB_APP_URL = "https://script.google.com/macros/s/AKfycbxKxKXWudQyNA8mToTIfz7eu2p4dldwNfIIrQNU0Z3ZouJhiuXHkUkhsrfca-EVeaAz/exec";

            fetch(WEB_APP_URL, {
                method: "POST",
                body: JSON.stringify(payload)
            }).then(() => {
                alert("ระบบได้รับสลิปแล้ว กำลังรอการตรวจสอบ");
                location.reload();
            });
        };
        reader.readAsDataURL(file);
    }
</script>
</body>
</html>
