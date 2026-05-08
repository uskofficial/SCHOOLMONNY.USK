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
        :root { --main-blue: #1e3a8a; --border-blue: #1e3a8a; --pink-btn: #ec4899; }
        body { font-family: 'Kanit', sans-serif; background-color: #fff; margin: 0; padding: 20px; color: #333; }
        .container { max-width: 750px; margin: auto; padding: 10px; }
        
        /* 1. หน้ากรอกชื่อเพื่อค้นหา */
        .header { display: flex; align-items: center; gap: 20px; margin-bottom: 30px; }
        .logo-img { width: 100px; height: 100px; object-fit: contain; }
        .school-info h2 { margin: 0; font-size: 24px; color: #333; }
        .school-info p { margin: 0; font-size: 18px; color: #555; }

        .search-area { margin-bottom: 30px; display: flex; gap: 10px; border-bottom: 1px solid #eee; padding-bottom: 25px; }
        input#nameInput { flex: 1; padding: 12px; border: 1px solid #ccc; border-radius: 8px; font-family: 'Kanit'; font-size: 16px; }
        button.btn-search { background: var(--main-blue); color: white; border: none; padding: 0 30px; border-radius: 8px; cursor: pointer; font-size: 16px; }

        /* 2. ส่วนแสดง ชื่อ สกุล โปรแกรมหอพัก และระดับชั้น */
        .report-section { display: none; animation: fadeIn 0.5s ease; }
        .info-box { border: 2.5px solid var(--border-blue); border-radius: 8px; padding: 20px; margin-bottom: 25px; }
        .info-line { font-size: 20px; margin-bottom: 8px; }
        .label { display: inline-block; width: 150px; font-weight: 500; }

        h3.title { font-size: 19px; color: #333; margin: 30px 0 15px; font-weight: 500; }
        h3.title span { font-size: 14px; font-weight: 300; color: #666; }

        /* 3. สถานะค่าธรรมเนียมหอพัก */
        .fee-card { border: 2.5px solid var(--border-blue); border-radius: 8px; padding: 20px; display: flex; justify-content: space-between; align-items: center; margin-bottom: 20px; }
        .fee-name { font-size: 19px; font-weight: 500; }

        /* 4. สถานะค่าอาหารรายเดือนและการคำนวณยอดรวม */
        .boarding-box { border: 2.5px solid var(--border-blue); border-radius: 8px; overflow: hidden; }
        .boarding-header { padding: 15px 20px; border-bottom: 1px solid #ddd; font-size: 18px; font-weight: 500; }
        .month-grid { display: grid; grid-template-columns: repeat(6, 1fr); text-align: center; border-bottom: 1px solid #ddd; }
        .month-item { padding: 15px 5px; border-right: 1px solid #eee; }
        .month-item:last-child { border-right: none; }
        .month-label { font-weight: 500; margin-bottom: 8px; font-size: 14px; }
        
        .status-badge { padding: 2px 8px; border-radius: 4px; font-size: 13px; border: 1px solid; }
        .paid { border-color: #22c55e; color: #15803d; background: #f0fdf4; }
        .unpaid { border-color: #ef4444; color: #b91c1c; background: #fef2f2; }

        .total-section { padding: 20px; display: flex; justify-content: space-between; font-weight: 600; font-size: 20px; background: #fff; }

        /* 5. ส่วนแนบสลิปและสถานะรอตรวจสอบ */
        .footer-action { display: flex; justify-content: space-between; align-items: center; margin-top: 40px; }
        .btn-upload { border: 2.5px solid var(--pink-btn); color: #be185d; background: white; padding: 12px 35px; border-radius: 6px; cursor: pointer; font-size: 18px; font-weight: 500; transition: 0.3s; }
        .btn-upload:hover { background: #fdf2f8; }
        .wait-status { text-align: right; color: #f59e0b; display: none; }
        .wait-status b { font-size: 20px; display: block; }
        .wait-status span { font-size: 14px; color: #666; }

        @keyframes fadeIn { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }
    </style>
</head>
<body>

<div class="container">
    <div class="header">
        <img src="https://i.postimg.cc/FzPbqZ7n/IMG-7790.png" alt="Logo" class="logo-img">
        <div class="school-info">
            <h2>SCHOOL MONEY</h2>
            <p>ระบบแจ้งจ่ายค่าใช้จ่ายโรงเรียนอุทยานศึกษากระบี่</p>
        </div>
    </div>

    <div class="search-area">
        <input type="text" id="nameInput" placeholder="กรอกชื่อ-นามสกุล นักเรียน">
        <button class="btn-search" onclick="doSearch()">ค้นหา</button>
    </div>

    <div id="loading" style="display:none; text-align:center;">กำลังค้นหาข้อมูล...</div>

    <div id="reportArea" class="report-section">
        <div class="info-box">
            <div class="info-line"><span class="label">ชื่อ - สกุล :</span> <span id="resName"></span></div>
            <div class="info-line"><span class="label">โปรแกรมหอพัก :</span> <span id="resProg"></span></div>
            <div class="info-line"><span class="label">ระดับชั้น :</span> <span id="resLevel"></span></div>
        </div>

        <h3 class="title">รายงานประวัติและยอดค้างชำระ <span>( Payment History & Outstanding Balance )</span></h3>

        <div class="fee-card">
            <div class="fee-name">ค่าธรรมเนียมหอพัก 1/2569 <br><span style="font-weight:300; font-size:14px;">( Dormitory Fee )</span></div>
            <div style="text-align:right">
                สถานะ <span id="resFeeStatus"></span><br>
                จำนวนเงิน <b id="resFeeAmt" style="font-size:18px"></b> บาท
            </div>
        </div>

        <div class="boarding-box">
            <div class="boarding-header">ค่าอาหารรายเดือน 1/2569 <span style="font-size:14px; font-weight:300;">( Boarding Fee )</span></div>
            <div class="month-grid">
                <div class="month-item"><div class="month-label">พฤษภาคม</div><div id="m1"></div></div>
                <div class="month-item"><div class="month-label">มิถุนายน</div><div id="m2"></div></div>
                <div class="month-item"><div class="month-label">กรกฎาคม</div><div id="m3"></div></div>
                <div class="month-item"><div class="month-label">สิงหาคม</div><div id="m4"></div></div>
                <div class="month-item"><div class="month-label">กันยายน</div><div id="m5"></div></div>
                <div class="month-item"><div class="month-label">ตุลาคม</div><div id="m6"></div></div>
            </div>
            <div class="total-section">
                <span style="font-size:16px;">ยอดที่ต้องชำระทั้งหมด <br><span style="font-weight:300; color:#777; font-size:13px">( Total Outstanding Balance )</span></span>
                <span id="resTotal"></span>
            </div>
        </div>

        <div class="footer-action">
            <div>
                <button class="btn-upload" onclick="document.getElementById('fileIn').click()">แนบสลิปการจ่าย</button>
                <input type="file" id="fileIn" style="display:none" multiple onchange="handleUpload()">
            </div>
            
            <div class="wait-status" id="waitMsg">
                <b>รอตรวจสอบสถานะการจ่าย</b>
                <span>โดยปกติแล้วรอตรวจสอบสถานะ 7 วันทำการ</span>
            </div>
        </div>
    </div>
</div>

<script>
    const firebaseConfig = {
        databaseURL: "https://schoolmonny-default-rtdb.asia-southeast1.firebasedatabase.app"
    };
    firebase.initializeApp(firebaseConfig);
    const db = firebase.database();

    let userData = null;
    let userSheetName = "";

    async function doSearch() {
        const name = document.getElementById('nameInput').value.trim();
        if(!name) return;

        document.getElementById('loading').style.display = 'block';
        document.getElementById('reportArea').style.display = 'none';

        const programs = ["หอพักญะมาอะห์ชาย", "หอพักญะมาอะห์หญิง", "หอพักกีฬา", "หอพักฮาฟิซอัลกุรอ่าน"];
        let found = false;

        for (let p of programs) {
            const snap = await db.ref(p).once('value');
            const data = snap.val();
            if(data) {
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
        if(!found) alert("ไม่พบข้อมูลนักเรียนคนนี้");
    }

    function renderReport(s, prog) {
        document.getElementById('reportArea').style.display = 'block';
        document.getElementById('resName').innerText = s["ชื่อ-สกุล"] || s["ชื่อ-นามสกุล"];
        document.getElementById('resProg').innerText = prog;
        document.getElementById('resLevel').innerText = "มัธยมศึกษาปีที่ " + (s["ระดับชั้นมัธยมศึกษาปีที่"] || "-");

        const fStat = s["ค่าธรรมเนียมหอพัก สถานะ"] || s["ค่าธรรมเนียมหอพัก-สถานะ"] || "ค้างชำระ";
        document.getElementById('resFeeStatus').innerHTML = `<span class="status-badge ${fStat === 'ชำระแล้ว' ? 'paid' : 'unpaid'}">${fStat}</span>`;
        document.getElementById('resFeeAmt').innerText = s["ยอดค้างค่าธรรมเนียม"] || "0";

        const months = ["พฤษภาคม", "มิถุนายน", "กรกฎาคม", "สิงหาคม", "กันยายน", "ตุลาคม"];
        months.forEach((m, index) => {
            const stat = s[m] || "ค้างชำระ";
            document.getElementById(`m${index+1}`).innerHTML = `<span class="status-badge ${stat === 'ชำระแล้ว' ? 'paid' : 'unpaid'}">${stat}</span>`;
        });

        document.getElementById('resTotal').innerText = (s["ยอดรวมค้างชำระ ( รวมห้าค่าอาหารรายเดือน )"] || 0) + " บาท";
        
        // เงื่อนไขข้อ 5: แสดงข้อความรอตรวจสอบเฉพาะเมื่อใน Sheet มีคำนี้
        const isWait = s["สถานะการตรวจสอบ"] === "รอตรวจสอบสถานะการจ่าย";
        document.getElementById('waitMsg').style.display = isWait ? 'block' : 'none';
    }

    function handleUpload() {
        const files = document.getElementById('fileIn').files;
        if(files.length === 0) return;

        // วนลูปบันทึกไฟล์ (กรณีเลือกหลายไฟล์)
        Array.from(files).forEach(file => {
            const reader = new FileReader();
            reader.onload = function(e) {
                const payload = {
                    studentName: userData["ชื่อ-สกุล"] || userData["ชื่อ-นามสกุล"],
                    sheetName: userSheetName,
                    fileName: `SLIP_${Date.now()}_${file.name}`,
                    fileData: e.target.result,
                    fileType: file.type
                };

                fetch("https://script.google.com/macros/s/AKfycbxKxKXWudQyNA8mToTIfz7eu2p4dldwNfIIrQNU0Z3ZouJhiuXHkUkhsrfca-EVeaAz/exec", {
                    method: "POST",
                    body: JSON.stringify(payload)
                }).then(() => {
                    alert("แนบสลิปเรียบร้อยแล้ว ระบบกำลังส่งข้อมูลไปตรวจสอบ");
                    location.reload();
                });
            };
            reader.readAsDataURL(file);
        });
    }
</script>
</body>
</html>
