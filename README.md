<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SCHOOL MONEY - ระบบแจ้งค่าใช้จ่าย</title>
    <link href="https://fonts.googleapis.com/css2?family=Kanit:wght@300;400;500;600&display=swap" rel="stylesheet">
    <script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-app.js"></script>
    <script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-database.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/sweetalert2@11"></script>

    <style>
        :root { --main-blue: #1e3a8a; --border-blue: #1e3a8a; --pink-btn: #ec4899; }
        body { font-family: 'Kanit', sans-serif; background-color: #fff; margin: 0; padding: 20px; color: #333; }
        .container { max-width: 750px; margin: auto; padding: 10px; }
        .header { display: flex; align-items: center; gap: 20px; margin-bottom: 30px; }
        .logo-img { width: 100px; height: 100px; object-fit: contain; }
        .school-info h2 { margin: 0; font-size: 24px; color: #333; }
        .school-info p { margin: 0; font-size: 18px; color: #555; }
        .search-area { margin-bottom: 30px; display: flex; gap: 10px; border-bottom: 1px solid #eee; padding-bottom: 25px; }
        input#nameInput { flex: 1; padding: 12px; border: 1px solid #ccc; border-radius: 8px; font-family: 'Kanit'; font-size: 16px; outline: none; }
        button.btn-search { background: var(--main-blue); color: white; border: none; padding: 0 30px; border-radius: 8px; cursor: pointer; font-size: 16px; }
        .report-section { display: none; animation: fadeIn 0.5s ease; }
        .info-box { border: 2.5px solid var(--border-blue); border-radius: 8px; padding: 20px; margin-bottom: 25px; }
        .info-line { font-size: 20px; margin-bottom: 8px; }
        .label { display: inline-block; width: 150px; font-weight: 500; }
        h3.title { font-size: 19px; color: #333; margin: 30px 0 15px; font-weight: 500; }
        .fee-card { border: 2.5px solid var(--border-blue); border-radius: 8px; padding: 20px; display: flex; justify-content: space-between; align-items: center; margin-bottom: 10px; }
        .boarding-box { border: 2.5px solid var(--border-blue); border-radius: 8px; overflow: hidden; }
        .boarding-header { padding: 15px 20px; border-bottom: 1px solid #ddd; font-size: 18px; font-weight: 500; background: #f8fafc; }
        .month-grid { display: grid; grid-template-columns: repeat(3, 1fr); text-align: center; border-bottom: 1px solid #ddd; }
        @media (min-width: 600px) { .month-grid { grid-template-columns: repeat(6, 1fr); } }
        .month-item { padding: 10px 5px; border-right: 1px solid #eee; border-bottom: 1px solid #eee; }
        .month-label { font-weight: 500; margin-bottom: 5px; font-size: 13px; color: #666; }
        .status-badge { padding: 2px 8px; border-radius: 4px; font-size: 12px; border: 1px solid; display: inline-block; }
        .ชำระแล้ว { border-color: #22c55e; color: #15803d; background: #f0fdf4; }
        .ค้างชำระ { border-color: #ef4444; color: #b91c1c; background: #fef2f2; }
        .รอตรวจสอบ { border-color: #f59e0b; color: #92400e; background: #fffbeb; }
        .ยังไม่ถึงกำหนด { border-color: #94a3b8; color: #475569; background: #f1f5f9; }
        .total-section { padding: 20px; display: flex; justify-content: space-between; font-weight: 600; font-size: 20px; background: #fdf2f8; }
        .upload-zone { background: #fafafa; padding: 15px; border-radius: 8px; margin-top: 10px; border: 1px dashed #ccc; text-align: center; }
        .btn-upload-small { background: var(--pink-btn); color: white; border: none; padding: 8px 15px; border-radius: 5px; font-size: 14px; cursor: pointer; margin-top: 10px; }
        .wait-status { text-align: center; color: #f59e0b; margin-top: 20px; display: none; padding: 15px; border: 1px solid #f59e0b; border-radius: 8px; }
        .bottom-info { text-align: center; margin-top: 50px; color: #666; font-size: 14px; line-height: 1.6; }
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

    <div id="loading" style="display:none; text-align:center; padding: 20px;">
        <div class="spinner-border text-primary"></div> กำลังค้นหาข้อมูล...
    </div>

    <div id="reportArea" class="report-section">
        <div class="info-box">
            <div class="info-line"><span class="label">ชื่อ - สกุล :</span> <span id="resName" style="color:var(--main-blue); font-weight:600;"></span></div>
            <div class="info-line"><span class="label">โปรแกรมหอพัก :</span> <span id="resProg"></span></div>
            <div class="info-line"><span class="label">ระดับชั้น :</span> <span id="resLevel"></span></div>
        </div>

        <h3 class="title">รายงานประวัติและยอดค้างชำระ <span>( Payment History )</span></h3>

        <div class="fee-card">
            <div class="fee-name">ค่าธรรมเนียมหอพัก 1/2569 <br><span style="font-weight:300; font-size:14px;">( Dormitory Fee )</span></div>
            <div style="text-align:right">
                สถานะ <span id="resFeeStatus"></span><br>
                จำนวนเงินค้าง <b id="resFeeAmt" style="font-size:18px; color:#ef4444;"></b> บาท
            </div>
        </div>
        <div class="upload-zone">
            <span style="font-size:13px; color:#666;">แนบสลิปค่าหอพัก (ไม่จำกัดขนาดไฟล์):</span><br>
            <input type="file" id="dormFile" accept="image/*" style="font-size:12px;">
            <button class="btn-upload-small" onclick="handleUpload('dorm')">ส่งสลิปค่าหอพัก</button>
        </div>

        <div class="boarding-box" style="margin-top:25px;">
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
                <span style="font-size:16px;">ยอดค้างค่าอาหารรวม <br><span style="font-weight:300; color:#777; font-size:13px">( Total Meal Outstanding )</span></span>
                <span id="resTotal" style="color:#be185d;"></span>
            </div>
        </div>
        <div class="upload-zone">
            <span style="font-size:13px; color:#666;">แนบสลิปค่าอาหารรายเดือน:</span><br>
            <input type="file" id="mealFile" accept="image/*" style="font-size:12px;">
            <button class="btn-upload-small" onclick="handleUpload('meal')">ส่งสลิปค่าอาหาร</button>
        </div>

        <div class="wait-status" id="waitMsg">
            <b style="font-size:20px;">รอตรวจสอบสถานะการจ่าย</b><br>
            <span>โดยปกติแล้วรอตรวจสอบสถานะ 7 วันทำการ</span>
        </div>

        <div style="margin-top:30px; text-align:center; padding:20px; border:1px solid #ddd; border-radius:8px;">
            <h4 style="font-size:18px;">ช่องทางการชำระเงิน</h4>
            <p style="margin:5px 0;">ธนาคารกรุงไทย : <b>982-x-xxxxx-x</b></p>
            <img src="https://i.postimg.cc/wBmf1KRW/att-AB9D1Bakym-D8jp-GMVkk-V5n-39QJO5MFVpd7DBp27Jc0.jpg" style="width:200px; margin-top:10px; border-radius:10px;">
        </div>
    </div>

    <div class="bottom-info">
        พบปัญหาหรือมีข้อสงสัยโปรดติดต่อ 078 - 789 - 6789<br>
        “ เรียนดี ประพฤติเด่น เน้นคุณภาพ ซึมซาบคุุณธรรม ถูกสัมพันธ์ชุมชน “
    </div>
</div>

<script>
    const firebaseConfig = { 
        databaseURL: "https://schoolmonny-e6c5e-default-rtdb.asia-southeast1.firebasedatabase.app" 
    };
    firebase.initializeApp(firebaseConfig);
    const db = firebase.database();
    
    // URL Web App จาก Apps Script ของคุณ
    const SCRIPT_URL = "https://script.google.com/macros/s/AKfycbzYSM7NNA5psMwEh16nAvBP66hnHdJ0ebKz0EVmyfpEyWDEgsqqmQnQ_4MEi2pRU0fM/exec";

    let userData = null;
    let userSheetName = "";

    async function doSearch() {
        const name = document.getElementById('nameInput').value.trim();
        if(!name) return Swal.fire('กรุณา', 'กรอกชื่อ-นามสกุลนักเรียน', 'warning');

        document.getElementById('loading').style.display = 'block';
        document.getElementById('reportArea').style.display = 'none';

        const programs = ["หอพักญะมาอะห์ชาย", "หอพักญะมาอะห์หญิง", "หอพักกีฬา", "หอพักฮาฟิซอัลกุรอ่าน"];
        let found = false;

        for (let p of programs) {
            const snap = await db.ref(p).once('value');
            const data = snap.val();
            if(data) {
                // ค้นหาแบบยืดหยุ่น (เผื่อ Header ใน Google Sheet ต่างกันเล็กน้อย)
                const match = data.find(s => (s["ชื่อ-นามสกุล"] === name || s["ชื่อ-สกุล"] === name));
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
        if(!found) Swal.fire('ไม่พบข้อมูล', 'โปรดตรวจสอบชื่อ-นามสกุลให้ถูกต้อง', 'error');
    }

    function renderReport(s, prog) {
        document.getElementById('reportArea').style.display = 'block';
        document.getElementById('resName').innerText = s["ชื่อ-นามสกุล"] || s["ชื่อ-สกุล"];
        document.getElementById('resProg').innerText = prog;
        document.getElementById('resLevel').innerText = "มัธยมศึกษาปีที่ " + (s["ระดับชั้นมัธยมศึกษาปีที่"] || "-");

        // ค่าธรรมเนียมหอพัก
        const fStat = s["ค่าธรรมเนียมหอพัก สถานะ"] || "ยังไม่ถึงกำหนด";
        document.getElementById('resFeeStatus').innerHTML = `<span class="status-badge ${fStat}">${fStat}</span>`;
        document.getElementById('resFeeAmt').innerText = s["ยอดค้างค่าธรรมเนียม"] || "0";

        // ค่าอาหารรายเดือน
        const months = ["พฤษภาคม", "มิถุนายน", "กรกฎาคม", "สิงหาคม", "กันยายน", "ตุลาคม"];
        months.forEach((m, i) => {
            const mStat = s[m] || "ยังไม่ถึงกำหนด";
            document.getElementById(`m${i+1}`).innerHTML = `<span class="status-badge ${mStat}">${mStat}</span>`;
        });

        document.getElementById('resTotal').innerText = (s["ยอดรวมค้างชำระ ( รวมห้าค่าอาหารรายเดือน )"] || 0) + " บาท";
        
        // แสดง "รอตรวจสอบ" ถ้าคอลัมน์ R (สถานะการตรวจสอบ) ใน Sheet มีค่านี้
        const checkStatus = s["สถานะการตรวจสอบ"];
        document.getElementById('waitMsg').style.display = checkStatus === "รอตรวจสอบ" ? 'block' : 'none';
    }

    function handleUpload(paymentType) {
        const fileInput = document.getElementById(paymentType + 'File');
        const file = fileInput.files[0];
        if(!file) return Swal.fire('แจ้งเตือน', 'กรุณาเลือกไฟล์สลิปก่อนส่ง', 'info');

        Swal.fire({
            title: 'กำลังส่งข้อมูล...',
            text: 'ระบบกำลังบันทึกสลิปลง Google Sheet',
            allowOutsideClick: false,
            didOpen: () => Swal.showLoading()
        });

        const reader = new FileReader();
        reader.onload = function(e) {
            const payload = {
                studentName: userData["ชื่อ-นามสกุล"] || userData["ชื่อ-สกุล"],
                sheetName: userSheetName,
                paymentType: paymentType, // "dorm" หรือ "meal"
                fileName: `${paymentType}_${Date.now()}.png`,
                fileData: e.target.result,
                fileType: file.type
            };

            fetch(SCRIPT_URL, {
                method: "POST",
                mode: "no-cors", // จำเป็นสำหรับ Apps Script
                body: JSON.stringify(payload)
            }).then(() => {
                Swal.fire('สำเร็จ!', 'แนบสลิปเรียบร้อย สถานะจะอัปเดตเป็นรอตรวจสอบภายใน 7 วัน', 'success')
                .then(() => location.reload());
            }).catch(err => {
                Swal.fire('เกิดข้อผิดพลาด', 'ไม่สามารถส่งไฟล์ได้ โปรดลองอีกครั้ง', 'error');
            });
        };
        reader.readAsDataURL(file);
    }
</script>
</body>
</html>
