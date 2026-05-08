<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SCHOOL MONEY - ระบบตรวจสอบค่าใช้จ่าย</title>
    <link href="https://fonts.googleapis.com/css2?family=Kanit:wght@300;400;500;600&display=swap" rel="stylesheet">
    <script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-app.js"></script>
    <script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-database.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/sweetalert2@11"></script>
    <style>
        :root { --main-blue: #1e3a8a; --pink-accent: #ec4899; }
        body { font-family: 'Kanit', sans-serif; background-color: #f8fafc; margin: 0; padding: 20px; }
        .container { max-width: 800px; margin: auto; background: white; padding: 30px; border-radius: 20px; box-shadow: 0 10px 30px rgba(0,0,0,0.1); }
        .header { display: flex; align-items: center; gap: 20px; margin-bottom: 30px; }
        .logo { width: 80px; height: 80px; background: #eee; border-radius: 50%; display: flex; align-items: center; justify-content: center; overflow: hidden; }
        .logo img { width: 100%; height: 100%; object-fit: cover; }
        
        .search-box { display: flex; gap: 10px; margin-bottom: 30px; }
        input { flex: 1; padding: 15px; border: 2px solid #e2e8f0; border-radius: 12px; font-size: 16px; outline: none; }
        button.btn-search { background: var(--main-blue); color: white; border: none; padding: 0 30px; border-radius: 12px; cursor: pointer; }

        .profile-card { border: 2.5px solid var(--main-blue); border-radius: 15px; padding: 20px; margin-bottom: 25px; }
        .info-row { font-size: 18px; margin-bottom: 8px; display: flex; }
        .info-label { width: 160px; font-weight: 600; color: #64748b; }

        .fee-card { border: 2.5px solid var(--main-blue); border-radius: 15px; padding: 20px; margin-bottom: 20px; }
        .status-badge { padding: 4px 12px; border-radius: 6px; font-size: 14px; font-weight: 500; border: 1px solid; }
        .ชำระแล้ว { background: #f0fdf4; color: #166534; border-color: #bbf7d0; }
        .ค้างชำระ { background: #fef2f2; color: #991b1b; border-color: #fecaca; }
        .รอตรวจสอบ { background: #fffbeb; color: #92400e; border-color: #fef3c7; }
        .ยังไม่ถึงกำหนด { background: #f1f5f9; color: #475569; border-color: #e2e8f0; }

        .month-grid { display: grid; grid-template-columns: repeat(3, 1fr); gap: 10px; margin-top: 15px; }
        .month-item { background: #f8fafc; padding: 10px; border-radius: 8px; text-align: center; border: 1px solid #e2e8f0; }

        .btn-upload { width: 100%; background: white; color: #be185d; border: 2px solid var(--pink-accent); padding: 12px; border-radius: 10px; font-weight: 600; cursor: pointer; margin-top: 15px; transition: 0.3s; }
        .btn-upload:hover { background: #fdf2f8; }
        
        .wait-notice { color: #f59e0b; background: #fffbeb; padding: 10px; border-radius: 8px; margin-top: 10px; font-size: 14px; text-align: center; border: 1px solid #fef3c7; }
        
        .payment-info { background: #f1f5f9; padding: 20px; border-radius: 15px; text-align: center; margin-top: 30px; }
        .qr-placeholder { width: 200px; height: 200px; background: white; margin: 15px auto; border-radius: 10px; display: flex; align-items: center; justify-content: center; border: 1px solid #ddd; }
    </style>
</head>
<body>

<div class="container">
    <div class="header">
        <div class="logo"><img src="YOUR_LOGO_URL" id="schoolLogo"></div>
        <div>
            <h2 style="margin:0; color:var(--main-blue);">SCHOOL MONEY</h2>
            <p style="margin:0; color:#64748b;">ระบบตรวจสอบค่าใช้จ่ายนักเรียน</p>
        </div>
    </div>

    <div class="search-box">
        <input type="text" id="searchInput" placeholder="กรอกชื่อ-นามสกุล เพื่อค้นหา">
        <button class="btn-search" onclick="searchStudent()">ค้นหา</button>
    </div>

    <div id="resultArea" style="display:none;">
        <div class="profile-card">
            <div class="info-row"><span class="info-label">ชื่อ-นามสกุล :</span> <span id="pName"></span></div>
            <div class="info-row"><span class="info-label">โปรแกรมหอพัก :</span> <span id="pProg"></span></div>
            <div class="info-row"><span class="info-label">ระดับชั้น :</span> ม. <span id="pLevel"></span></div>
        </div>

        <h3 style="color:var(--main-blue);">1. ค่าธรรมเนียมหอพัก</h3>
        <div class="fee-card">
            <div style="display:flex; justify-content:space-between; align-items:center;">
                <span>สถานะการชำระ:</span>
                <span id="dormStatus" class="status-badge"></span>
            </div>
            <div style="margin-top:10px; font-size:18px; font-weight:600; text-align:right;">
                ยอดค้างชำระ: <span id="dormAmount" style="color:#ef4444;"></span> บาท
            </div>
            <button class="btn-upload" onclick="openUpload('dorm')">แนบสลิปค่าธรรมเนียมหอพัก</button>
            <div id="waitDorm" class="wait-notice" style="display:none;">
                <b>รอตรวจสอบสถานะการจ่าย</b><br>โดยปกติแล้วรอตรวจสอบสถานะ 7 วันทำการ
            </div>
        </div>

        <h3 style="color:var(--main-blue);">2. ค่าอาหารรายเดือน</h3>
        <div class="fee-card">
            <div class="month-grid" id="monthGrid"></div>
            <div style="margin-top:20px; font-size:20px; font-weight:700; border-top:2px solid #f1f5f9; padding-top:15px; display:flex; justify-content:space-between;">
                <span>ยอดรวมที่ต้องชำระ:</span>
                <span id="totalAmount" style="color:#ef4444;"></span>
            </div>
            <button class="btn-upload" onclick="openUpload('meal')">แนบสลิปค่าอาหารรายเดือน</button>
            <div id="waitMeal" class="wait-notice" style="display:none;">
                <b>รอตรวจสอบสถานะการจ่าย</b><br>โดยปกติแล้วรอตรวจสอบสถานะ 7 วันทำการ
            </div>
        </div>

        <div class="payment-info">
            <h4 style="margin:0;">ช่องทางการชำระเงิน</h4>
            <p style="margin:5px 0;">ธนาคารกสิกรไทย: <b>XXX-X-XXXXX-X</b></p>
            <div class="qr-placeholder">ใส่รูป QR Code ตรงนี้</div>
        </div>
    </div>
</div>

<input type="file" id="fileInput" style="display:none" onchange="handleFile(event)">

<script>
    const firebaseConfig = {
        databaseURL: "https://schoolmonny-e6c5e-default-rtdb.asia-southeast1.firebasedatabase.app"
    };
    firebase.initializeApp(firebaseConfig);
    const db = firebase.database();
    const SCRIPT_URL = "ใส่ URL ของ APPS SCRIPT ที่ก๊อปมา";

    let currentStudent = null;
    let currentSheet = "";
    let uploadType = "";

    async function searchStudent() {
        const name = document.getElementById('searchInput').value.trim();
        if(!name) return;
        
        const tabs = ["หอพักญะมาอะห์ชาย", "หอพักญะมาอะห์หญิง", "หอพักกีฬา", "หอพักฮาฟิซอัลกุรอ่าน"];
        let found = false;

        Swal.fire({ title: 'กำลังค้นหา...', didOpen: () => Swal.showLoading() });

        for (const tab of tabs) {
            const snap = await db.ref(tab).once('value');
            const data = snap.val();
            if(data) {
                const student = data.find(s => s["ชื่อ-นามสกุล"] === name);
                if(student) {
                    currentStudent = student;
                    currentSheet = tab;
                    displayResult(student, tab);
                    found = true;
                    break;
                }
            }
        }
        Swal.close();
        if(!found) Swal.fire('ไม่พบข้อมูล', 'กรุณาตรวจสอบชื่อ-นามสกุลให้ถูกต้อง', 'error');
    }

    function displayResult(s, tab) {
        document.getElementById('resultArea').style.display = 'block';
        document.getElementById('pName').innerText = s["ชื่อ-นามสกุล"];
        document.getElementById('pProg').innerText = tab;
        document.getElementById('pLevel').innerText = s["ระดับชั้นมัธยมศึกษาปีที่"];
        
        const dStat = s["ค่าธรรมเนียมหอพัก สถานะ"];
        document.getElementById('dormStatus').innerText = dStat;
        document.getElementById('dormStatus').className = 'status-badge ' + dStat;
        document.getElementById('dormAmount').innerText = s["ยอดค้างค่าธรรมเนียม"] || 0;
        
        // จัดการสถานะรอตรวจสอบ
        const isWaiting = s["สถานะการตรวจสอบ"] === "รอตรวจสอบ";
        document.getElementById('waitDorm').style.display = isWaiting ? 'block' : 'none';
        document.getElementById('waitMeal').style.display = isWaiting ? 'block' : 'none';

        const months = ["พฤษภาคม", "มิถุนายน", "กรกฎาคม", "สิงหาคม", "กันยายน", "ตุลาคม"];
        let monthHtml = "";
        months.forEach(m => {
            monthHtml += `<div class="month-item"><div>${m}</div><span class="status-badge ${s[m]}">${s[m]}</span></div>`;
        });
        document.getElementById('monthGrid').innerHTML = monthHtml;
        document.getElementById('totalAmount').innerText = (s["ยอดรวมค้างชำระ ( รวมห้าค่าอาหารรายเดือน )"] || 0).toLocaleString() + " บาท";
    }

    function openUpload(type) {
        uploadType = type;
        document.getElementById('fileInput').click();
    }

    function handleFile(e) {
        const file = e.target.files[0];
        if(!file) return;
        
        const reader = new FileReader();
        reader.onload = function(event) {
            Swal.fire({ title: 'กำลังอัปโหลดสลิป...', didOpen: () => Swal.showLoading() });
            
            const payload = {
                studentName: currentStudent["ชื่อ-นามสกุล"],
                sheetName: currentSheet,
                paymentType: uploadType,
                fileData: event.target.result,
                fileName: `${uploadType}_${Date.now()}.png`,
                fileType: file.type
            };

            fetch(SCRIPT_URL, { method: 'POST', body: JSON.stringify(payload), mode: 'no-cors' })
            .then(() => {
                Swal.fire('สำเร็จ!', 'แนบสลิปเรียบร้อย ระบบจะตรวจสอบใน 7 วัน', 'success').then(() => location.reload());
            });
        };
        reader.readAsDataURL(file);
    }
</script>
</body>
</html>
