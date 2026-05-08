<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SCHOOL MONEY - USK</title>
    <link href="https://fonts.googleapis.com/css2?family=Kanit:wght@300;400;500;600&display=swap" rel="stylesheet">
    <script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-app.js"></script>
    <script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-database.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/sweetalert2@11"></script>
    <style>
        :root { --main-blue: #1e3a8a; --pink-btn: #ec4899; }
        body { font-family: 'Kanit', sans-serif; background: #f3f4f6; padding: 15px; margin: 0; }
        .container { max-width: 600px; margin: auto; background: white; border-radius: 20px; overflow: hidden; box-shadow: 0 10px 25px rgba(0,0,0,0.1); }
        .header { background: white; padding: 25px; display: flex; align-items: center; gap: 15px; border-bottom: 1px solid #eee; }
        .logo { width: 70px; height: 70px; border-radius: 50%; object-fit: contain; }
        .search-area { padding: 25px; }
        input { width: 100%; padding: 15px; border: 2px solid #ddd; border-radius: 12px; font-size: 16px; box-sizing: border-box; }
        .btn-search { width: 100%; background: var(--main-blue); color: white; border: none; padding: 15px; border-radius: 12px; margin-top: 10px; cursor: pointer; font-size: 16px; font-weight: 600; }
        
        .result-card { padding: 20px; display: none; }
        .info-box { border: 2.5px solid var(--main-blue); border-radius: 15px; padding: 20px; margin-bottom: 20px; }
        .fee-card { border: 2.5px solid var(--main-blue); border-radius: 15px; padding: 20px; margin-bottom: 20px; position: relative; }
        .status-badge { display: inline-block; padding: 5px 12px; border-radius: 6px; font-size: 13px; font-weight: 500; border: 1px solid; }
        .ชำระแล้ว { border-color: #22c55e; color: #166534; background: #f0fdf4; }
        .ค้างชำระ { border-color: #ef4444; color: #991b1b; background: #fef2f2; }
        .รอตรวจสอบ { border-color: #f59e0b; color: #92400e; background: #fffbeb; }
        .ยังไม่ถึงกำหนด { border-color: #94a3b8; color: #475569; background: #f8fafc; }

        .month-grid { display: grid; grid-template-columns: repeat(3, 1fr); gap: 10px; margin: 15px 0; }
        .month-item { background: #f8fafc; padding: 10px; border-radius: 8px; text-align: center; border: 1px solid #e2e8f0; }
        
        .btn-upload { background: white; border: 2px solid var(--pink-btn); color: var(--pink-btn); width: 100%; padding: 12px; border-radius: 10px; font-weight: 600; cursor: pointer; margin-top: 10px; }
        .wait-notice { color: #f59e0b; font-size: 13px; margin-top: 8px; font-weight: 600; text-align: right; }
        
        .payment-info { background: #f1f5f9; padding: 20px; border-radius: 15px; text-align: center; margin-top: 20px; }
        .qr-img { width: 200px; height: 200px; margin: 10px 0; border: 1px solid #ccc; background: white; }
    </style>
</head>
<body>

<div class="container">
    <div class="header">
        <img src="https://i.postimg.cc/FzPbqZ7n/IMG-7790.png" class="logo">
        <div>
            <h2 style="margin:0; color:var(--main-blue);">SCHOOL MONEY</h2>
            <p style="margin:0; color:#666; font-size:14px;">ระบบแจ้งจ่ายค่าใช้จ่ายโรงเรียนอุทยานศึกษากระบี่</p>
        </div>
    </div>

    <div class="search-area">
        <input type="text" id="nameInput" placeholder="กรอกชื่อ-นามสกุล เพื่อค้นหา">
        <button class="btn-search" onclick="search()">ค้นหา</button>
    </div>

    <div class="result-card" id="resultArea">
        <div class="info-box">
            <p><b>ชื่อ-นามสกุล:</b> <span id="rName"></span></p>
            <p><b>โปรแกรมหอพัก:</b> <span id="rProg"></span></p>
            <p><b>ระดับชั้น:</b> ม. <span id="rLevel"></span></p>
        </div>

        <h3>1. ค่าธรรมเนียมหอพัก</h3>
        <div class="fee-card">
            <div style="display:flex; justify-content:space-between">
                <span>สถานะ: <span id="dormStatus"></span></span>
                <span>ค้างชำระ: <b id="dormAmt" style="color:red"></b> บาท</span>
            </div>
            <button class="btn-upload" onclick="triggerUpload('dorm')">แนบสลิปค่าหอพัก</button>
            <div id="waitDorm" class="wait-notice">รอตรวจสอบสถานะ 7 วันทำการ</div>
        </div>

        <h3>2. ค่าอาหารรายเดือน</h3>
        <div class="fee-card">
            <div class="month-grid" id="monthArea"></div>
            <div style="text-align:right; border-top:1px solid #ddd; padding-top:10px; font-size:18px">
                รวมยอดค้าง: <b id="totalAmt" style="color:red"></b> บาท
            </div>
            <button class="btn-upload" onclick="triggerUpload('meal')">แนบสลิปค่าอาหาร</button>
            <div id="waitMeal" class="wait-notice">รอตรวจสอบสถานะ 7 วันทำการ</div>
        </div>

        <div class="payment-info">
            <b>ช่องทางการชำระเงิน</b><br>
            ธนาคาร: กสิกรไทย | เลขบัญชี: 000-0-00000-0<br>
            ชื่อบัญชี: โรงเรียนอุทยานศึกษากระบี่
            <div class="qr-img">QR CODE</div>
        </div>
    </div>
</div>

<input type="file" id="fileIn" style="display:none" onchange="handleFile(event)">

<script>
    // --- ใส่ FIREBASE CONFIG ของคุณที่นี่ ---
    const firebaseConfig = {
        apiKey: "AIzaSyA50WXTVWCZeCBYj_jVJU0a7tb8WsY2FNE",
        authDomain: "schoolmonny-e6c5e.firebaseapp.com",
        databaseURL: "https://schoolmonny-e6c5e-default-rtdb.asia-southeast1.firebasedatabase.app",
        projectId: "schoolmonny-e6c5e",
        storageBucket: "schoolmonny-e6c5e.firebasestorage.app",
        messagingSenderId: "45471802660",
        appId: "1:45471802660:web:eb03c6239fb33bbf3b3fb1"
    };
    firebase.initializeApp(firebaseConfig);
    const db = firebase.database();
    const GAS_URL = "ใส่_URL_APPS_SCRIPT_ที่ก๊อปมา";

    let currentStudent = null;
    let currentTab = "";
    let currentUploadType = "";

    async function search() {
        const name = document.getElementById('nameInput').value.trim();
        if(!name) return;
        Swal.fire({ title: 'กำลังค้นหา...', didOpen: () => Swal.showLoading() });
        
        const tabs = ["หอพักญะมาอะห์ชาย", "หอพักญะมาอะห์หญิง", "หอพักกีฬา", "หอพักฮาฟิซอัลกุรอ่าน"];
        let found = false;

        for (let tab of tabs) {
            const snap = await db.ref(tab).once('value');
            const data = snap.val();
            if(data) {
                const match = data.find(s => s["ชื่อ-นามสกุล"] === name);
                if(match) {
                    currentStudent = match;
                    currentTab = tab;
                    render(match, tab);
                    found = true; break;
                }
            }
        }
        Swal.close();
        if(!found) Swal.fire('ไม่พบข้อมูล', 'โปรดตรวจสอบชื่อ-นามสกุล', 'error');
    }

    function render(s, tab) {
        document.getElementById('resultArea').style.display = 'block';
        document.getElementById('rName').innerText = s["ชื่อ-นามสกุล"];
        document.getElementById('rProg').innerText = tab;
        document.getElementById('rLevel').innerText = s["ระดับชั้นมัธยมศึกษาปีที่"];
        
        const dStat = s["ค่าธรรมเนียมหอพัก สถานะ"] || "ค้างชำระ";
        document.getElementById('dormStatus').innerHTML = `<span class="status-badge ${dStat}">${dStat}</span>`;
        document.getElementById('dormAmt').innerText = s["ยอดค้างค่าธรรมเนียม"] || "0";

        const months = ["พฤษภาคม", "มิถุนายน", "กรกฎาคม", "สิงหาคม", "กันยายน", "ตุลาคม"];
        let mHtml = "";
        months.forEach(m => {
            const stat = s[m] || "ยังไม่ถึงกำหนด";
            mHtml += `<div class="month-item"><small>${m}</small><br><span class="status-badge ${stat}">${stat}</span></div>`;
        });
        document.getElementById('monthArea').innerHTML = mHtml;
        document.getElementById('totalAmt').innerText = s["ยอดรวมค้างชำระ ( รวมห้าค่าอาหารรายเดือน )"] || "0";

        // แสดงสถานะรอตรวจสอบ
        const isWaiting = s["สถานะการตรวจสอบ"] && s["สถานะการตรวจสอบ"].includes("รอตรวจสอบ");
        document.getElementById('waitDorm').style.display = isWaiting ? 'block' : 'none';
        document.getElementById('waitMeal').style.display = isWaiting ? 'block' : 'none';
        if (isWaiting) {
            document.getElementById('waitDorm').innerText = s["สถานะการตรวจสอบ"];
            document.getElementById('waitMeal').innerText = s["สถานะการตรวจสอบ"];
        }
    }

    function triggerUpload(type) {
        currentUploadType = type;
        document.getElementById('fileIn').click();
    }

    function handleFile(e) {
        const file = e.target.files[0];
        if(!file) return;
        const reader = new FileReader();
        reader.onload = function(evt) {
            Swal.fire({ title: 'กำลังส่งข้อมูล...', allowOutsideClick: false, didOpen: () => Swal.showLoading() });
            const payload = {
                studentName: currentStudent["ชื่อ-นามสกุล"],
                sheetName: currentTab,
                type: currentUploadType,
                fileData: evt.target.result,
                fileName: `SLIP_${Date.now()}.png`,
                fileType: file.type
            };
            fetch(GAS_URL, { method: "POST", body: JSON.stringify(payload), mode: "no-cors" })
            .then(() => {
                Swal.fire('สำเร็จ!', 'แนบสลิปแล้ว ระบบจะตรวจสอบใน 7 วัน', 'success').then(() => location.reload());
            });
        };
        reader.readAsDataURL(file);
    }
</script>
</body>
</html>
