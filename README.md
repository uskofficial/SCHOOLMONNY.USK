<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SCHOOL MONEY - ระบบแจ้งจ่ายค่าใช้จ่าย</title>
    <link href="https://fonts.googleapis.com/css2?family=Kanit:wght@300;400;500;600&display=swap" rel="stylesheet">
    <script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-app.js"></script>
    <script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-database.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/sweetalert2@11"></script>

    <style>
        :root { 
            --main-blue: #1e3a8a; 
            --border-blue: #1e3a8a; 
            --pink-btn: #ec4899; 
            --light-blue: #eff6ff;
        }
        body { 
            font-family: 'Kanit', sans-serif; 
            background-color: #f8fafc; 
            margin: 0; 
            padding: 20px; 
            color: #333; 
        }
        .container { 
            max-width: 750px; 
            margin: auto; 
            background: white;
            padding: 20px;
            border-radius: 10px;
        }
        
        /* Header */
        .header { 
            display: flex; 
            align-items: center; 
            gap: 20px; 
            margin-bottom: 30px; 
        }
        .logo-img { width: 100px; height: 100px; object-fit: contain; }
        .school-info h2 { margin: 0; font-size: 24px; color: #333; }
        .school-info p { margin: 0; font-size: 18px; color: #555; }

        /* Search */
        .search-area { 
            margin-bottom: 30px; 
            display: flex; 
            gap: 10px; 
            border-bottom: 1px solid #eee; 
            padding-bottom: 25px; 
        }
        input#nameInput { 
            flex: 1; padding: 12px; border: 1px solid #ccc; border-radius: 8px; 
            font-family: 'Kanit'; font-size: 16px; outline: none; 
        }
        button.btn-search { 
            background: var(--main-blue); color: white; border: none; 
            padding: 0 30px; border-radius: 8px; cursor: pointer; font-size: 16px; 
        }

        /* Profile Box */
        .info-box { 
            border: 2px solid var(--border-blue); 
            border-radius: 8px; 
            padding: 25px; 
            margin-bottom: 25px; 
            background-color: #fff;
        }
        .info-line { font-size: 20px; margin-bottom: 12px; }
        .label { display: inline-block; width: 160px; font-weight: 500; }

        h3.title { font-size: 19px; color: #333; margin: 30px 0 15px; font-weight: 500; }
        h3.title span { font-size: 14px; font-weight: 300; color: #666; }

        /* Fee Cards */
        .fee-card { 
            border: 2px solid var(--border-blue); 
            border-radius: 8px; 
            padding: 15px 20px; 
            display: flex; 
            justify-content: space-between; 
            align-items: center; 
            margin-bottom: 10px; 
        }
        .fee-name { font-size: 19px; font-weight: 500; }

        /* Boarding Grid */
        .boarding-box { border: 2px solid var(--border-blue); border-radius: 8px; overflow: hidden; background: #fdfdfd; }
        .boarding-header { padding: 12px 20px; border-bottom: 1px solid #ddd; font-size: 18px; font-weight: 500; background: var(--light-blue); }
        .month-grid { display: grid; grid-template-columns: repeat(6, 1fr); text-align: center; border-bottom: 1px solid #ddd; }
        .month-item { padding: 15px 5px; border-right: 1px solid #eee; }
        .month-item:last-child { border-right: none; }
        .month-label { font-weight: 500; margin-bottom: 10px; font-size: 14px; color: #444; }
        .month-amount { font-size: 12px; color: #666; margin-top: 5px; }
        
        /* Status Badges */
        .status-badge { padding: 2px 8px; border-radius: 4px; font-size: 13px; border: 1px solid; display: inline-block; }
        .paid { border-color: #22c55e; color: #15803d; background: #f0fdf4; } /* ชำระแล้ว */
        .unpaid { border-color: #ef4444; color: #b91c1c; background: #fef2f2; } /* ค้างชำระ */
        .pending { border-color: #f59e0b; color: #92400e; background: #fffbeb; } /* รอตรวจสอบ */

        .total-section { padding: 15px 20px; display: flex; justify-content: space-between; font-weight: 600; font-size: 19px; background: #fff; }

        /* Buttons & Wait Text */
        .footer-action { display: flex; justify-content: space-between; align-items: center; margin-top: 15px; margin-bottom: 25px; }
        .btn-upload { border: 2px solid var(--pink-btn); color: #be185d; background: white; padding: 10px 20px; border-radius: 6px; cursor: pointer; font-size: 16px; font-weight: 500; width: 45%; }
        .wait-status { text-align: right; color: #f59e0b; width: 50%; display: none; }
        .wait-status b { font-size: 18px; display: block; }
        .wait-status span { font-size: 13px; color: #777; }

        /* Payment Channel */
        .payment-btn-box { 
            background: var(--light-blue); border: 2px solid var(--border-blue); 
            padding: 15px; text-align: center; border-radius: 8px; font-size: 20px; font-weight: 600; cursor: pointer;
        }

        .bottom-info { text-align: center; margin-top: 50px; color: #666; font-size: 14px; line-height: 1.6; }
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

    <div id="reportArea" style="display:none;">
        <div class="info-box">
            <div class="info-line"><span class="label">ชื่อ - สกุล :</span> <span id="resName"></span></div>
            <div class="info-line"><span class="label">โปรแกรมหอพัก :</span> <span id="resProg"></span></div>
            <div class="info-line"><span class="label">ระดับชั้น :</span> <span id="resLevel" style="font-weight: 500;"></span></div>
        </div>

        <h3 class="title">รายงานประวัติและยอดค้างชำระ <span>( Payment History & Outstanding Balance )</span></h3>

        <div class="fee-card">
            <div class="fee-name">ค่าธรรมเนียมหอพัก 1/2569 <br><span style="font-weight:300; font-size:14px;">( Dormitory Fee )</span></div>
            <div style="text-align:right">
                สถานะ <span id="resFeeStatus"></span><br>
                จำนวนเงิน <b id="resFeeAmt" style="font-size:18px; color: #333;"></b>
            </div>
        </div>

        <div class="footer-action">
            <button class="btn-upload" onclick="triggerUpload('dorm')">แนบสลิปการจ่ายค่าธรรมเนียมหอพัก</button>
            <div class="wait-status" id="waitDorm">
                <b>รอตรวจสอบสถานะการจ่าย</b>
                <span>โดยปกติแล้วรอตรวจสอบสถานะ 7 วันทำการ</span>
            </div>
        </div>

        <div class="boarding-box">
            <div class="boarding-header">ค่าอาหารรายเดือน 1/2569 <span style="font-size:14px; font-weight:300;">( Boarding Fee )</span></div>
            <div class="month-grid">
                <div class="month-item"><div class="month-label">พฤษภาคม</div><div id="m1"></div><div class="month-amount" id="a1"></div></div>
                <div class="month-item"><div class="month-label">มิถุนายน</div><div id="m2"></div><div class="month-amount" id="a2"></div></div>
                <div class="month-item"><div class="month-label">กรกฎาคม</div><div id="m3"></div><div class="month-amount" id="a3"></div></div>
                <div class="month-item"><div class="month-label">สิงหาคม</div><div id="m4"></div><div class="month-amount" id="a4"></div></div>
                <div class="month-item"><div class="month-label">กันยายน</div><div id="m5"></div><div class="month-amount" id="a5"></div></div>
                <div class="month-item"><div class="month-label">ตุลาคม</div><div id="m6"></div><div class="month-amount" id="a6"></div></div>
            </div>
            <div class="total-section">
                <span style="font-size:16px;">ยอดที่ต้องชำระทั้งหมด <br><span style="font-weight:300; color:#777; font-size:13px">( Total Outstanding Balance )</span></span>
                <span id="resTotal" style="color: #333;"></span>
            </div>
        </div>

        <div class="footer-action">
            <button class="btn-upload" onclick="triggerUpload('meal')">แนบสลิปการจ่ายค่าอาหาร</button>
            <div class="wait-status" id="waitMeal">
                <b>รอตรวจสอบสถานะการจ่าย</b>
                <span>โดยปกติแล้วรอตรวจสอบสถานะ 7 วันทำการ</span>
            </div>
        </div>

        <div class="payment-btn-box" onclick="showPaymentInfo()">
            ช่องทางการชำระเงิน
        </div>
    </div>

    <input type="file" id="fileIn" style="display:none" onchange="handleUpload()">

    <div class="bottom-info">
        พบปัญหาหรือมีข้อสงสัยโปรดติดต่อ 078 - 789 - 6789<br>
        “ เรียนดี ประพฤติเด่น เน้นคุณภาพ ซึมซาบคุุณธรรม ถูกสัมพันธ์ชุมชน “
    </div>
</div>

<script>
    // --- ตั้งค่า Firebase ---
    const firebaseConfig = { 
        databaseURL: "https://schoolmonny-e6c5e-default-rtdb.asia-southeast1.firebasedatabase.app" 
    };
    firebase.initializeApp(firebaseConfig);
    const db = firebase.database();
    
    // Web App URL จาก Apps Script
    const SCRIPT_URL = "https://script.google.com/macros/s/AKfycbzYSM7NNA5psMwEh16nAvBP66hnHdJ0ebKz0EVmyfpEyWDEgsqqmQnQ_4MEi2pRU0fM/exec";

    let userData = null;
    let userSheetName = "";
    let uploadTarget = "";

    async function doSearch() {
        const name = document.getElementById('nameInput').value.trim();
        if(!name) return;

        Swal.fire({ title: 'กำลังค้นหา...', allowOutsideClick: false, didOpen: () => Swal.showLoading() });

        const programs = ["หอพักญะมาอะห์ชาย", "หอพักญะมาอะห์หญิง", "หอพักกีฬา", "หอพักฮาฟิซอัลกุรอ่าน"];
        let found = false;

        for (let p of programs) {
            const snap = await db.ref(p).once('value');
            const data = snap.val();
            if(data) {
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
        Swal.close();
        if(!found) Swal.fire('ไม่พบข้อมูล', 'โปรดตรวจสอบชื่อ-นามสกุลอีกครั้ง', 'error');
    }

    function renderReport(s, prog) {
        document.getElementById('reportArea').style.display = 'block';
        document.getElementById('resName').innerText = s["ชื่อ-นามสกุล"] || s["ชื่อ-สกุล"];
        document.getElementById('resProg').innerText = prog;
        
        // แก้ไขระดับชั้นตามที่ต้องการ
        const levelData = s["ระดับชั้นมัธยมศึกษาปีที่"] || "-";
        document.getElementById('resLevel').innerText = "มัธยมศึกษาปีที่ " + levelData;

        // จัดการสถานะและยอดเงิน
        const fStat = s["ค่าธรรมเนียมหอพัก สถานะ"] || "ค้างชำระ";
        document.getElementById('resFeeStatus').innerHTML = `<span class="status-badge ${getStatClass(fStat)}">${fStat}</span>`;
        document.getElementById('resFeeAmt').innerText = (s["ยอดค้างค่าธรรมเนียม"] || 0).toLocaleString();

        const months = ["พฤษภาคม", "มิถุนายน", "กรกฎาคม", "สิงหาคม", "กันยายน", "ตุลาคม"];
        months.forEach((m, i) => {
            const mStat = s[m] || "ยังไม่ถึงกำหนด";
            document.getElementById(`m${i+1}`).innerHTML = `<span class="status-badge ${getStatClass(mStat)}">${mStat}</span>`;
            // ดึงยอดที่ค้างรายเดือน (F, H, J, L, N, P)
            const amtKeys = ["ยอดที่ค้าง (พ.ค.)", "ยอดที่ค้าง (มิ.ย.)", "ยอดที่ค้าง (ก.ค.)", "ยอดที่ค้าง (ส.ค.)", "ยอดที่ค้าง (ก.ย.)", "ยอดที่ค้าง (ต.ค.)"];
            document.getElementById(`a${i+1}`).innerText = (s[amtKeys[i]] || 0).toLocaleString() + " บาท";
        });

        document.getElementById('resTotal').innerText = (s["ยอดรวมค้างชำระ ( รวมห้าค่าอาหารรายเดือน )"] || 0).toLocaleString() + " บาท";
        
        // แสดงข้อความ "รอตรวจสอบ" ตามเงื่อนไขคอลัมน์ R
        const isWaiting = s["สถานะการตรวจสอบ"] === "รอตรวจสอบ";
        document.getElementById('waitDorm').style.display = isWaiting ? 'block' : 'none';
        document.getElementById('waitMeal').style.display = isWaiting ? 'block' : 'none';
    }

    function getStatClass(stat) {
        if (!stat) return 'pending';
        if (stat.includes('ชำระแล้ว')) return 'paid';
        if (stat.includes('ค้างชำระ')) return 'unpaid';
        return 'pending';
    }

    function triggerUpload(target) {
        uploadTarget = target;
        document.getElementById('fileIn').click();
    }

    function handleUpload() {
        const file = document.getElementById('fileIn').files[0];
        if(!file) return;

        Swal.fire({
            title: 'กำลังส่งข้อมูล...',
            text: 'ระบบกำลังบันทึกสลิปลงฐานข้อมูล',
            allowOutsideClick: false,
            didOpen: () => Swal.showLoading()
        });

        const reader = new FileReader();
        reader.onload = function(e) {
            const payload = {
                studentName: userData["ชื่อ-นามสกุล"] || userData["ชื่อ-สกุล"],
                sheetName: userSheetName,
                paymentType: uploadTarget, // 'dorm' หรือ 'meal'
                fileData: e.target.result,
                fileType: file.type,
                fileName: `${uploadTarget}_${Date.now()}.png`
            };

            fetch(SCRIPT_URL, {
                method: "POST",
                body: JSON.stringify(payload)
            }).then(() => {
                Swal.fire('สำเร็จ!', 'แนบสลิปเรียบร้อย ระบบจะตรวจสอบภายใน 7 วัน', 'success').then(() => location.reload());
            }).catch(err => {
                Swal.fire('ผิดพลาด', 'ไม่สามารถส่งข้อมูลได้', 'error');
            });
        };
        reader.readAsDataURL(file);
    }

    function showPaymentInfo() {
        Swal.fire({
            title: 'ช่องทางการชำระเงิน',
            html: `
                <div style="text-align:center;">
                    <p>ธนาคารกรุงไทย<br>เลขบัญชี: 982-x-xxxxx-x<br>ชื่อบัญชี: โรงเรียนอุทยานศึกษากระบี่</p>
                    <img src="https://i.postimg.cc/wBmf1KRW/att-AB9D1Bakym-D8jp-GMVkk-V5n-39QJO5MFVpd7DBp27Jc0.jpg" style="width:200px; border-radius:10px;">
                </div>
            `,
            showCloseButton: true,
            showConfirmButton: false
        });
    }
</script>
</body>
</html>
