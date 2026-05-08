
<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ระบบตรวจสอบการชำระเงินหอพัก</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
    <link href="https://fonts.googleapis.com/css2?family=Kanit:wght@300;400;600&display=swap" rel="stylesheet">
    <script src="https://cdn.jsdelivr.net/npm/sweetalert2@11"></script>
    <style>
        body { font-family: 'Kanit', sans-serif; background-color: #f0f2f5; color: #333; }
        .container { max-width: 800px; }
        .card { border-radius: 20px; border: none; box-shadow: 0 10px 25px rgba(0,0,0,0.08); transition: 0.3s; }
        .header-section { background: linear-gradient(135deg, #1e3c72 0%, #2a5298 100%); color: white; padding: 40px 20px; border-radius: 0 0 30px 30px; margin-bottom: 30px; }
        .status-badge { padding: 8px 16px; border-radius: 50px; font-weight: 600; font-size: 0.85em; display: inline-block; }
        .ชำระแล้ว { background-color: #d1fae5; color: #065f46; }
        .ค้างชำระ { background-color: #fee2e2; color: #991b1b; }
        .รอตรวจสอบ { background-color: #fef3c7; color: #92400e; }
        .ยังไม่ถึงกำหนด { background-color: #f3f4f6; color: #374151; }
        .btn-custom { border-radius: 12px; padding: 10px 25px; font-weight: 600; }
        .payment-box { background-color: white; border: 1px solid #e5e7eb; border-radius: 15px; padding: 20px; margin-top: 15px; }
        #logo { max-height: 120px; margin-bottom: 15px; }
        .qr-img { max-width: 220px; border: 5px solid white; border-radius: 10px; box-shadow: 0 4px 10px rgba(0,0,0,0.1); }
    </style>
</head>
<body>

<div class="header-section text-center">
    <img src="https://via.placeholder.com/150x150?text=LOGO" id="logo" alt="Logo">
    <h2 class="fw-bold">ระบบตรวจสอบสถานะหอพัก</h2>
    <p class="opacity-75">โรงเรียน/สถาบัน...</p>
</div>

<div class="container mb-5">
    <div class="card p-4 mb-4">
        <div class="row g-3">
            <div class="col-md-9">
                <input type="text" id="searchInput" class="form-control form-control-lg text-center" placeholder="พิมพ์ ชื่อ-นามสกุล ของนักเรียน">
            </div>
            <div class="col-md-3">
                <button class="btn btn-primary btn-lg w-100 btn-custom" onclick="searchData()">ค้นหา</button>
            </div>
        </div>
    </div>

    <div id="resultArea" style="display: none;">
        <div class="card p-4">
            <div class="d-flex justify-content-between align-items-center mb-3">
                <h4 id="displayName" class="mb-0 text-primary fw-bold">...</h4>
                <span id="displayClass" class="badge bg-secondary">ม...</span>
            </div>
            <p class="text-muted mb-4">สังกัด: <span id="displayDorm" class="fw-bold text-dark">...</span></p>
            
            <div class="row">
                <div class="col-md-6 mb-3">
                    <div class="payment-box">
                        <h6 class="fw-bold border-bottom pb-2">ค่าธรรมเนียมหอพัก</h6>
                        <div id="dormStatus" class="status-badge mb-2 mt-2">...</div>
                        <p class="mb-2">ค้างชำระ: <span id="dormAmount" class="text-danger fw-bold">0</span> บาท</p>
                        <hr>
                        <input type="file" id="dormFile" class="form-control form-control-sm mb-2" accept="image/*">
                        <button class="btn btn-outline-primary btn-sm w-100" onclick="uploadSlip('dorm')">แนบสลิปค่าหอ</button>
                        <div id="dormWaitText" class="mt-2 text-muted small" style="display:none;">⚠️ รอตรวจสอบสถานะ 7 วันทำการ</div>
                    </div>
                </div>

                <div class="col-md-6 mb-3">
                    <div class="payment-box h-100">
                        <h6 class="fw-bold border-bottom pb-2">ค่าอาหารรายเดือน</h6>
                        <div id="mealStatus" class="status-badge mb-2 mt-2">...</div>
                        <p class="mb-2">ยอดรวมค้างชำระ: <span id="totalMealAmount" class="text-danger fw-bold">0</span> บาท</p>
                        <hr>
                        <input type="file" id="mealFile" class="form-control form-control-sm mb-2" accept="image/*">
                        <button class="btn btn-outline-primary btn-sm w-100" onclick="uploadSlip('meal')">แนบสลิปค่าอาหาร</button>
                        <div id="mealWaitText" class="mt-2 text-muted small" style="display:none;">⚠️ รอตรวจสอบสถานะ 7 วันทำการ</div>
                    </div>
                </div>
            </div>

            <div class="mt-4 p-4 bg-light rounded-4 text-center">
                <h5 class="fw-bold mb-3">💰 ช่องทางการชำระเงิน</h5>
                <p class="mb-2">ธนาคาร: <strong>กรุงไทย</strong></p>
                <p class="mb-3">เลขบัญชี: <strong>xxx-x-xxxxx-x</strong></p>
                <img src="https://via.placeholder.com/200x200?text=PROMPTPAY+QR" class="qr-img" alt="Payment QR">
            </div>
        </div>
    </div>
</div>

<script type="module">
    // ตั้งค่า Firebase (ใช้ค่าของคุณ)
    import { initializeApp } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-app.js";
    import { getDatabase, ref, onValue } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-database.js";

    const firebaseConfig = {
        apiKey: "AIzaSyA50WXTVWCZeCBYj_jVJU0a7tb8WsY2FNE",
        authDomain: "schoolmonny-e6c5e.firebaseapp.com",
        databaseURL: "https://schoolmonny-e6c5e-default-rtdb.asia-southeast1.firebasedatabase.app",
        projectId: "schoolmonny-e6c5e",
        storageBucket: "schoolmonny-e6c5e.firebasestorage.app",
        appId: "1:45471802660:web:eb03c6239fb33bbf3b3fb1"
    };

    const app = initializeApp(firebaseConfig);
    const db = getDatabase(app);

    // URL ของ Google Apps Script ที่คุณ Deploy เป็น Web App
    const scriptUrl = "https://script.google.com/macros/s/AKfycbzYSM7NNA5psMwEh16nAvBP66hnHdJ0ebKz0EVmyfpEyWDEgsqqmQnQ_4MEi2pRU0fM/exec";

    window.searchData = function() {
        const name = document.getElementById('searchInput').value.trim();
        if(!name) return Swal.fire('กรุณา!', 'กรอกชื่อ-นามสกุลเพื่อค้นหา', 'warning');

        const dorms = ["หอพักญะมาอะห์ชาย", "หอพักญะมาอะห์หญิง", "หอพักกีฬา", "หอพักฮาฟิซอัลกุรอ่าน"];
        let isFound = false;

        dorms.forEach(dorm => {
            const dbRef = ref(db, dorm);
            onValue(dbRef, (snapshot) => {
                const data = snapshot.val();
                for (let key in data) {
                    if (data[key]['ชื่อ-นามสกุล'] === name) {
                        displayInfo(data[key], dorm);
                        isFound = true;
                    }
                }
            });
        });
        
        setTimeout(() => { if(!isFound) Swal.fire('ไม่พบข้อมูล', 'โปรดตรวจสอบชื่อ-นามสกุลอีกครั้ง', 'error'); }, 1500);
    }

    function displayInfo(user, dormType) {
        document.getElementById('resultArea').style.display = 'block';
        document.getElementById('displayName').innerText = user['ชื่อ-นามสกุล'];
        document.getElementById('displayClass').innerText = "ม." + user['ระดับชั้นมัธยมศึกษาปีที่'];
        document.getElementById('displayDorm').innerText = dormType;
        
        // จัดการสถานะค่าหอ
        const dStatus = user['ค่าธรรมเนียมหอพัก สถานะ'];
        document.getElementById('dormStatus').innerText = dStatus;
        document.getElementById('dormStatus').className = 'status-badge ' + (dStatus || '');
        document.getElementById('dormAmount').innerText = user['ยอดค้างค่าธรรมเนียม'] || 0;

        // จัดการค่าอาหาร
        const mStatus = user['สถานะการตรวจสอบ'] || 'ยังไม่ชำระ';
        document.getElementById('mealStatus').innerText = mStatus;
        document.getElementById('mealStatus').className = 'status-badge ' + (mStatus === 'ชำระแล้ว' ? 'ชำระแล้ว' : 'รอตรวจสอบ');
        document.getElementById('totalMealAmount').innerText = user['ยอดรวมค้างชำระ ( รวมห้าค่าอาหารรายเดือน )'] || 0;

        // แสดงข้อความ "รอตรวจสอบ"
        const isWaiting = user['สถานะการตรวจสอบ'] === "รอตรวจสอบ";
        document.getElementById('dormWaitText').style.display = isWaiting ? 'block' : 'none';
        document.getElementById('mealWaitText').style.display = isWaiting ? 'block' : 'none';
    }

    window.uploadSlip = function(type) {
        const fileInput = document.getElementById(type + 'File');
        const file = fileInput.files[0];
        const studentName = document.getElementById('searchInput').value;
        const sheetName = document.getElementById('displayDorm').innerText;

        if (!file) return Swal.fire('แจ้งเตือน', 'กรุณาเลือกไฟล์สลิป', 'info');

        const reader = new FileReader();
        reader.readAsDataURL(file);
        reader.onload = async () => {
            Swal.fire({ title: 'กำลังอัปโหลด...', allowOutsideClick: false, didOpen: () => Swal.showLoading() });

            const payload = {
                sheetName: sheetName,
                studentName: studentName,
                paymentType: type,
                fileData: reader.result,
                fileType: file.type,
                fileName: `${type}_${studentName}.jpg`
            };

            fetch(scriptUrl, { method: "POST", body: JSON.stringify(payload) })
            .then(() => {
                Swal.fire('สำเร็จ!', 'แนบสลิปเรียบร้อย ระบบจะตรวจสอบภายใน 7 วัน', 'success').then(() => location.reload());
            })
            .catch(() => Swal.fire('ผิดพลาด', 'ไม่สามารถส่งข้อมูลได้', 'error'));
        };
    }
</script>
</body>
</html>
