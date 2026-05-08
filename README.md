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
        :root { --main-blue: #1e3a8a; --light-blue: #eff6ff; --border: #1e3a8a; }
        body { font-family: 'Kanit', sans-serif; background-color: #fff; margin: 0; padding: 20px; color: #333; }
        .container { max-width: 750px; margin: auto; padding: 20px; border-radius: 10px; }
        
        /* Header */
        .header { display: flex; align-items: center; gap: 20px; margin-bottom: 40px; }
        .logo-box { width: 90px; height: 90px; border: 2px solid var(--main-blue); border-radius: 50%; display: flex; align-items: center; justify-content: center; font-weight: bold; color: var(--main-blue); }
        .school-info h2 { margin: 0; font-size: 26px; color: #444; }
        .school-info p { margin: 0; font-size: 20px; }

        /* Search Section */
        .search-area { margin-bottom: 30px; text-align: center; }
        input { padding: 12px; width: 300px; border: 1.5px solid #ccc; border-radius: 8px; font-family: 'Kanit'; }
        button.btn-search { background: var(--main-blue); color: white; border: none; padding: 12px 25px; border-radius: 8px; cursor: pointer; }

        /* Content Boxes */
        .box { border: 2.5px solid var(--border); border-radius: 5px; padding: 20px; margin-bottom: 25px; }
        .info-grid { font-size: 20px; line-height: 1.8; }
        .label { display: inline-block; width: 160px; font-weight: 500; }

        h3.title { font-size: 20px; color: #444; margin-top: 40px; margin-bottom: 15px; font-weight: 500; }
        h3.title span { font-size: 14px; font-weight: 300; color: #666; }

        /* Table */
        .table-box { border: 2.5px solid var(--border); border-radius: 5px; overflow: hidden; }
        .table-header { padding: 15px 20px; border-bottom: 2px solid #ddd; font-size: 18px; font-weight: 500; }
        .month-row { display: grid; grid-template-columns: repeat(6, 1fr); border-bottom: 1px solid #ddd; text-align: center; }
        .month-item { padding: 15px 5px; border-right: 1px solid #ddd; }
        .month-item:last-child { border-right: none; }
        .month-name { font-weight: 500; margin-bottom: 10px; }
        
        /* Status Badges */
        .status { padding: 2px 8px; border-radius: 4px; font-size: 14px; border: 1px solid; }
        .paid { border-color: #22c55e; color: #15803d; background: #f0fdf4; }
        .unpaid { border-color: #ef4444; color: #b91c1c; background: #fef2f2; }

        .total-row { padding: 20px; display: flex; justify-content: space-between; font-weight: 600; font-size: 20px; }

        /* Footer */
        .footer-action { display: flex; justify-content: space-between; align-items: center; margin-top: 40px; }
        .btn-upload { border: 2.5px solid #ec4899; color: #be185d; background: white; padding: 12px 40px; border-radius: 5px; cursor: pointer; font-size: 18px; font-weight: 500; }
        .wait-text { color: #f59e0b; text-align: right; }
        .wait-text b { font-size: 20px; display: block; }
    </style>
</head>
<body>

<div class="container">
    <div class="header">
        <div class="logo-box">โลโก้</div>
        <div class="school-info">
            <h2>SCHOOL MONEY</h2>
            <p>ระบบแจ้งจ่ายค่าใช้จ่ายโรงเรียนอุทยานศึกษากระบี่</p>
        </div>
    </div>

    <div class="search-area">
        <input type="text" id="nameInput" placeholder="กรอกชื่อ-นามสกุล...">
        <button class="btn-search" onclick="doSearch()">ค้นหา</button>
    </div>

    <div id="resultArea" style="display:none">
        <div class="box info-grid">
            <div><span class="label">ชื่อ - สกุล :</span> <span id="rName"></span></div>
            <div><span class="label">โปรแกรมหอพัก :</span> <span id="rProg"></span></div>
            <div><span class="label">ระดับชั้น :</span> <span id="rLevel"></span></div>
        </div>

        <h3 class="title">รายงานประวัติและยอดค้างชำระ <span>( Payment History & Outstanding Balance )</span></h3>

        <div class="box" style="display:flex; justify-content:space-between; align-items:center">
            <div style="font-size:20px; font-weight:500">ค่าธรรมเนียมหอพัก 1/2569 <span style="font-size:14px; font-weight:300">( Dormitory Fee )</span></div>
            <div style="text-align:right">
                สถานะ <span id="rFeeStat"></span><br>
                <small>จำนวนเงิน</small> <b id="rFeeAmt" style="font-size:18px"></b> บาท
            </div>
        </div>

        <div class="table-box">
            <div class="table-header">ค่าอาหารรายเดือน 1/2569 <span style="font-size:14px; font-weight:300">( Boarding Fee )</span></div>
            <div class="month-row">
                <div class="month-item"><div class="month-name">พฤษภาคม</div><div id="m1"></div></div>
                <div class="month-item"><div class="month-name">มิถุนายน</div><div id="m2"></div></div>
                <div class="month-item"><div class="month-name">กรกฎาคม</div><div id="m3"></div></div>
                <div class="month-item"><div class="month-name">สิงหาคม</div><div id="m4"></div></div>
                <div class="month-item"><div class="month-name">กันยายน</div><div id="m5"></div></div>
                <div class="month-item"><div class="month-name">ตุลาคม</div><div id="m6"></div></div>
            </div>
            <div class="total-row">
                <span>ยอดที่ต้องชำระทั้งหมด <span style="font-weight:300; color:#666; font-size:14px">( Total Outstanding Balance )</span></span>
                <span id="rTotal"></span>
            </div>
        </div>

        <div class="footer-action">
            <button class="btn-upload" onclick="document.getElementById('fileIn').click()">แนบสลิปการจ่าย</button>
            <input type="file" id="fileIn" style="display:none" onchange="uploadSlip()">
            <div class="wait-text" id="waitMsg" style="display:none">
                <b>รอตรวจสอบสถานะการจ่าย</b>
                <span>โดยปกติแล้วรอตรวจสอบสถานะ 7 วันทำการ</span>
            </div>
        </div>
    </div>
</div>

<script>
    const firebaseConfig = { databaseURL: "https://schoolmonny-default-rtdb.asia-southeast1.firebasedatabase.app" };
    firebase.initializeApp(firebaseConfig);
    const db = firebase.database();

    let currentStudent = null;
    let currentSheet = "";

    async function doSearch() {
        const name = document.getElementById('nameInput').value.trim();
        const programs = ["หอพักญะมาอะห์ชาย", "หอพักญะมาอะห์หญิง", "หอพักกีฬา", "หอพักฮาฟิซอัลกุรอ่าน"];
        for (let p of programs) {
            const snap = await db.ref(p).once('value');
            const data = snap.val();
            if(data) {
                const found = data.find(s => s["ชื่อ-สกุล"] === name || s["ชื่อ-นามสกุล"] === name);
                if(found) {
                    currentStudent = found; currentSheet = p;
                    renderData(found, p);
                    return;
                }
            }
        }
        alert("ไม่พบข้อมูล");
    }

    function renderData(s, prog) {
        document.getElementById('resultArea').style.display = 'block';
        document.getElementById('rName').innerText = s["ชื่อ-สกุล"] || s["ชื่อ-นามสกุล"];
        document.getElementById('rProg').innerText = prog;
        document.getElementById('rLevel').innerText = "มัธยมศึกษาปีที่ " + (s["ระดับชั้นมัธยมศึกษาปีที่"] || "-");
        
        const fStat = s["ค่าธรรมเนียมหอพัก สถานะ"] || s["ค่าธรรมเนียมหอพัก-สถานะ"];
        document.getElementById('rFeeStat').innerHTML = `<span class="status ${fStat === 'ชำระแล้ว' ? 'paid' : 'unpaid'}">${fStat}</span>`;
        document.getElementById('rFeeAmt').innerText = s["ยอดค้างค่าธรรมเนียม"] || "0";

        const months = ["พฤษภาคม", "มิถุนายน", "กรกฎาคม", "สิงหาคม", "กันยายน", "ตุลาคม"];
        months.forEach((m, i) => {
            const stat = s[m] || "ค้างชำระ";
            document.getElementById(`m${i+1}`).innerHTML = `<span class="status ${stat === 'ชำระแล้ว' ? 'paid' : 'unpaid'}">${stat}</span>`;
        });

        document.getElementById('rTotal').innerText = (s["ยอดรวมค้างชำระ ( รวมห้าค่าอาหารรายเดือน )"] || 0) + " บาท";
        document.getElementById('waitMsg').style.display = s["สถานะการตรวจสอบ"] === "รอตรวจสอบสถานะการจ่าย" ? "block" : "none";
    }

    function uploadSlip() {
        const file = document.getElementById('fileIn').files[0];
        const reader = new FileReader();
        reader.onload = function(e) {
            const payload = {
                studentName: currentStudent["ชื่อ-สกุล"] || currentStudent["ชื่อ-นามสกุล"],
                sheetName: currentSheet,
                fileName: `SLIP_${Date.now()}.png`,
                fileData: e.target.result,
                fileType: file.type
            };
            fetch("https://script.google.com/macros/s/AKfycbxKxKXWudQyNA8mToTIfz7eu2p4dldwNfIIrQNU0Z3ZouJhiuXHkUkhsrfca-EVeaAz/exec", {
                method: "POST", body: JSON.stringify(payload)
            }).then(() => { alert("ส่งสลิปสำเร็จ"); location.reload(); });
        };
        reader.readAsDataURL(file);
    }
</script>
</body>
</html>
