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
        body { font-family: 'Kanit', sans-serif; background-color: #fff; margin: 0; padding: 20px; color: #333; }
        .container { max-width: 750px; margin: auto; padding: 10px; }
        .header { display: flex; align-items: center; gap: 20px; margin-bottom: 30px; }
        .logo-img { width: 100px; height: 100px; object-fit: contain; }
        .search-area { margin-bottom: 30px; display: flex; gap: 10px; border-bottom: 1px solid #eee; padding-bottom: 25px; }
        input#nameInput { flex: 1; padding: 12px; border: 1.5px solid #ccc; border-radius: 8px; font-size: 16px; outline: none; }
        .btn-search { background: var(--main-blue); color: white; border: none; padding: 0 25px; border-radius: 8px; cursor: pointer; }
        .info-box { border: 2.5px solid var(--main-blue); border-radius: 8px; padding: 20px; margin-bottom: 25px; }
        .info-line { font-size: 20px; margin-bottom: 8px; }
        .label { display: inline-block; width: 150px; font-weight: 500; }
        .fee-card { border: 2.5px solid var(--main-blue); border-radius: 8px; padding: 20px; display: flex; justify-content: space-between; align-items: center; margin-bottom: 20px; }
        .boarding-box { border: 2.5px solid var(--main-blue); border-radius: 8px; overflow: hidden; }
        .month-grid { display: grid; grid-template-columns: repeat(6, 1fr); text-align: center; border-bottom: 1px solid #ddd; }
        .month-item { padding: 15px 5px; border-right: 1px solid #eee; }
        .status-badge { padding: 2px 8px; border-radius: 4px; font-size: 13px; border: 1px solid; }
        .ชำระแล้ว { border-color: #22c55e; color: #15803d; background: #f0fdf4; }
        .ค้างชำระ { border-color: #ef4444; color: #b91c1c; background: #fef2f2; }
        .รอตรวจสอบ { border-color: #f59e0b; color: #92400e; background: #fffbeb; }
        .total-section { padding: 20px; display: flex; justify-content: space-between; font-weight: 600; font-size: 20px; }
        .wait-status { color: #f59e0b; text-align: right; margin-top: 10px; display: none; }
        .btn-upload { border: 2.5px solid var(--pink-btn); color: #be185d; background: white; padding: 10px 20px; border-radius: 6px; cursor: pointer; font-weight: 500; }
    </style>
</head>
<body>

<div class="container">
    <div class="header">
        <img src="https://i.postimg.cc/FzPbqZ7n/IMG-7790.png" class="logo-img">
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

    <div id="reportArea" style="display:none;">
        <div class="info-box">
            <div class="info-line"><span class="label">ชื่อ - สกุล :</span> <span id="resName"></span></div>
            <div class="info-line"><span class="label">โปรแกรมหอพัก :</span> <span id="resProg"></span></div>
            <div class="info-line"><span class="label">ระดับชั้น :</span> <span id="resLevel"></span></div>
        </div>

        <div class="fee-card">
            <div>ค่าธรรมเนียมหอพัก 1/2569</div>
            <div style="text-align:right">สถานะ <span id="resFeeStatus"></span><br>จำนวนเงิน <b id="resFeeAmt"></b> บาท</div>
        </div>

        <div class="boarding-box">
            <div style="padding:15px; border-bottom:1px solid #ddd;">ค่าอาหารรายเดือน 1/2569</div>
            <div class="month-grid" id="monthGrid">
                </div>
            <div class="total-section">
                <span>ยอดที่ต้องชำระทั้งหมด</span>
                <span id="resTotal"></span>
            </div>
        </div>

        <div style="margin-top:20px; display:flex; justify-content:space-between; align-items:center;">
            <button class="btn-upload" onclick="triggerUpload()">แนบสลิปการจ่าย</button>
            <div id="waitMsg" class="wait-status"><b>รอตรวจสอบสถานะการจ่าย</b><br><small>ปกติใช้เวลา 7 วันทำการ</small></div>
        </div>
        <input type="file" id="fileIn" style="display:none" onchange="handleUpload()">
    </div>
</div>

<script>
    const firebaseConfig = { databaseURL: "https://schoolmonny-e6c5e-default-rtdb.asia-southeast1.firebasedatabase.app" };
    firebase.initializeApp(firebaseConfig);
    const db = firebase.database();

    let userData = null;
    let userSheet = "";

    async function doSearch() {
        const name = document.getElementById('nameInput').value.trim();
        if(!name) return;
        document.getElementById('loading').style.display = 'block';
        document.getElementById('reportArea').style.display = 'none';

        const progs = ["หอพักญะมาอะห์ชาย", "หอพักญะมาอะห์หญิง", "หอพักกีฬา", "หอพักฮาฟิซอัลกุรอ่าน"];
        let found = false;

        for (let p of progs) {
            const snap = await db.ref(p).once('value');
            const data = snap.val();
            if(data) {
                const match = data.find(s => s["ชื่อ-นามสกุล"] === name);
                if(match) {
                    userData = match; userSheet = p;
                    render(match, p);
                    found = true; break;
                }
            }
        }
        document.getElementById('loading').style.display = 'none';
        if(!found) Swal.fire('ไม่พบข้อมูล', 'โปรดตรวจสอบชื่อ-นามสกุล', 'error');
    }

    function render(s, p) {
        document.getElementById('reportArea').style.display = 'block';
        document.getElementById('resName').innerText = s["ชื่อ-นามสกุล"];
        document.getElementById('resProg').innerText = p;
        document.getElementById('resLevel').innerText = s["ระดับชั้นมัธยมศึกษาปีที่"];

        const fStat = s["ค่าธรรมเนียมหอพัก-สถานะ"] || "ค้างชำระ";
        document.getElementById('resFeeStatus').innerHTML = `<span class="status-badge ${fStat}">${fStat}</span>`;
        document.getElementById('resFeeAmt').innerText = s["ยอดค้างค่าธรรมเนียม"] || "0";

        const months = ["พฤษภาคม", "มิถุนายน", "กรกฎาคม", "สิงหาคม", "กันยายน", "ตุลาคม"];
        let mHtml = "";
        months.forEach(m => {
            const stat = s[m] || "ยังไม่ถึงกำหนด";
            mHtml += `<div class="month-item"><div style="font-size:12px;color:#666;">${m}</div><div class="status-badge ${stat}">${stat}</div></div>`;
        });
        document.getElementById('monthGrid').innerHTML = mHtml;
        document.getElementById('resTotal').innerText = (s["ยอดรวมค้างชำระ ( รวมห้าค่าอาหารรายเดือน )"] || 0) + " บาท";
        document.getElementById('waitMsg').style.display = s["สถานะการตรวจสอบ"] === "รอตรวจสอบ" ? 'block' : 'none';
    }

    function triggerUpload() { document.getElementById('fileIn').click(); }

    function handleUpload() {
        const file = document.getElementById('fileIn').files[0];
        if(!file) return;
        const reader = new FileReader();
        reader.onload = function(e) {
            Swal.fire({title:'กำลังส่ง...', didOpen:()=>Swal.showLoading()});
            const payload = {
                studentName: userData["ชื่อ-นามสกุล"],
                sheetName: userSheet,
                fileData: e.target.result,
                fileName: `SLIP_${Date.now()}.png`,
                fileType: file.type
            };
            fetch("https://script.google.com/macros/s/AKfycbzYSM7NNA5psMwEh16nAvBP66hnHdJ0ebKz0EVmyfpEyWDEgsqqmQnQ_4MEi2pRU0fM/exec", {
                method:"POST", body: JSON.stringify(payload)
            }).then(() => {
                Swal.fire('สำเร็จ', 'บันทึกสลิปแล้ว', 'success').then(()=>location.reload());
            });
        };
        reader.readAsDataURL(file);
    }
</script>
</body>
</html>
