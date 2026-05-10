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
            --orange-status: #f59e0b;
        }
        * { box-sizing: border-box; }
        
        body { 
            font-family: 'Kanit', sans-serif; 
            background-color: #f8fafc; 
            margin: 0; 
            padding: 10px; 
            color: #333; 
        }
        .container { 
            max-width: 750px; 
            margin: auto; 
            background: white;
            padding: 20px;
            border-radius: 10px;
            width: 100%;
        }
        
        .header { 
            display: flex; 
            align-items: center; 
            gap: 15px; 
            margin-bottom: 30px; 
        }
        .logo-img { width: 80px; height: 80px; object-fit: contain; }
        .school-info h2 { margin: 0; font-size: 20px; color: #333; }
        .school-info p { margin: 0; font-size: 15px; color: #555; }

        .search-area { 
            margin-bottom: 30px; 
            display: flex; 
            flex-wrap: wrap; 
            gap: 10px; 
            border-bottom: 1px solid #eee; 
            padding-bottom: 25px; 
        }
        input#nameInput { 
            flex: 1; min-width: 200px; padding: 12px; border: 1px solid #ccc; border-radius: 8px; 
            font-family: 'Kanit'; font-size: 16px; outline: none; 
        }
        button.btn-search { 
            background: var(--main-blue); color: white; border: none; 
            padding: 12px 30px; border-radius: 8px; cursor: pointer; font-size: 16px; 
            width: 100%; 
        }

        .info-box { 
            border: 2px solid var(--border-blue); 
            border-radius: 8px; 
            padding: 15px; 
            margin-bottom: 25px; 
            background-color: #fff;
        }
        .info-line { font-size: 17px; margin-bottom: 8px; display: flex; flex-wrap: wrap; }
        .label { display: inline-block; width: 130px; font-weight: 500; color: #666; }

        h3.title { font-size: 17px; color: #333; margin: 30px 0 15px; font-weight: 500; }
        h3.title span { font-size: 13px; font-weight: 300; color: #666; display: block; }

        .fee-card { 
            border: 2px solid var(--border-blue); 
            border-radius: 8px; 
            padding: 15px; 
            display: flex; 
            justify-content: space-between; 
            align-items: center; 
            margin-bottom: 10px; 
            flex-wrap: wrap;
            gap: 10px;
        }
        .fee-name { font-size: 17px; font-weight: 500; flex: 1; min-width: 150px; }

        .boarding-box { border: 2px solid var(--border-blue); border-radius: 8px; overflow: hidden; background: #fdfdfd; }
        .boarding-header { padding: 12px 15px; border-bottom: 1px solid #ddd; font-size: 16px; font-weight: 500; background: var(--light-blue); }
        .month-grid { display: grid; grid-template-columns: repeat(3, 1fr); text-align: center; border-bottom: 1px solid #ddd; }
        .month-item { padding: 10px 5px; border-right: 1px solid #eee; border-bottom: 1px solid #eee; }
        .month-label { font-weight: 500; margin-bottom: 5px; font-size: 13px; color: #444; }
        .month-amount { font-size: 11px; color: #666; margin-top: 5px; font-weight: bold; }
        
        .status-badge { padding: 2px 6px; border-radius: 4px; font-size: 11px; border: 1px solid; display: inline-block; }
        .paid { border-color: #22c55e; color: #15803d; background: #f0fdf4; } 
        .unpaid { border-color: #ef4444; color: #b91c1c; background: #fef2f2; } 
        .pending { border-color: #f59e0b; color: #92400e; background: #fffbeb; } 
        .not-reached { border-color: #3b82f6; color: #1e40af; background: #eff6ff; } 

        .total-section { padding: 15px; display: flex; justify-content: space-between; font-weight: 600; font-size: 17px; background: #fff; flex-wrap: wrap; }

        .footer-action { display: flex; flex-direction: column; gap: 10px; margin-top: 10px; margin-bottom: 25px; }
        .btn-upload { border: 2px solid var(--pink-btn); color: #be185d; background: white; padding: 12px; border-radius: 6px; cursor: pointer; font-size: 15px; font-weight: 500; width: 100%; }
        
        .wait-status { text-align: center; color: var(--orange-status); width: 100%; display: none; }
        .wait-status b { font-size: 16px; display: block; }
        .wait-status span { font-size: 12px; color: #777; }

        .payment-btn-box { 
            background: var(--light-blue); border: 2px solid var(--border-blue); 
            padding: 15px; text-align: center; border-radius: 8px; font-size: 18px; font-weight: 600; cursor: pointer; margin-top: 10px;
        }

        .bottom-info { text-align: center; margin-top: 40px; color: #666; font-size: 12px; line-height: 1.6; }

        @media (min-width: 600px) {
            .logo-img { width: 100px; height: 100px; }
            .school-info h2 { font-size: 24px; }
            .btn-search { width: auto; }
            .month-grid { grid-template-columns: repeat(6, 1fr); }
            .footer-action { flex-direction: row; justify-content: space-between; align-items: center; }
            .btn-upload { width: 45%; }
            .wait-status { width: 50%; text-align: right; }
            .info-box { padding: 25px; }
        }
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
        </div>

        <h3 class="title">รายงานประวัติและยอดค้างชำระ <span>( Payment History & Outstanding Balance )</span></h3>

        <div class="fee-card">
            <div class="fee-name">ค่าธรรมเนียมหอพัก 1/2569 <br><span style="font-weight:300; font-size:13px;">( Dormitory Fee )</span></div>
            <div style="text-align:right">
                สถานะ <span id="resFeeStatus"></span><br>
                จำนวนเงิน <b id="resFeeAmt" style="font-size:18px; color: #333;"></b>
            </div>
        </div>

        <div class="footer-action">
            <button class="btn-upload" onclick="triggerUpload('dorm')">แนบสลิปจ่ายค่าธรรมเนียมหอพัก</button>
            <div class="wait-status" id="waitDorm">
                <b id="dormWaitTitle">รอตรวจสอบสถานะการจ่าย</b>
                <span id="dormWaitDetail">โดยปกติแล้วรอตรวจสอบสถานะ 7 วันทำการ</span>
            </div>
        </div>

        <div class="boarding-box">
            <div class="boarding-header">ค่าอาหารรายเดือน 1/2569 <span style="font-size:13px; font-weight:300;">( Boarding Fee )</span></div>
            <div class="month-grid">
                <div class="month-item"><div class="month-label">พ.ค.</div><div id="m1"></div><div class="month-amount" id="a1"></div></div>
                <div class="month-item"><div class="month-label">มิ.ย.</div><div id="m2"></div><div class="month-amount" id="a2"></div></div>
                <div class="month-item"><div class="month-label">ก.ค.</div><div id="m3"></div><div class="month-amount" id="a3"></div></div>
                <div class="month-item"><div class="month-label">ส.ค.</div><div id="m4"></div><div class="month-amount" id="a4"></div></div>
                <div class="month-item"><div class="month-label">ก.ย.</div><div id="m5"></div><div class="month-amount" id="a5"></div></div>
                <div class="month-item"><div class="month-label">ต.ค.</div><div id="m6"></div><div class="month-amount" id="a6"></div></div>
            </div>
            <div class="total-section">
                <span style="font-size:15px;">ยอดที่ต้องชำระทั้งหมด <br><span style="font-weight:300; color:#777; font-size:12px">( Total Outstanding Balance )</span></span>
                <span id="resTotal" style="color: #333;">0 บาท</span>
            </div>
        </div>

        <div class="footer-action">
            <button class="btn-upload" onclick="triggerUpload('food')">แนบสลิปการจ่ายค่าอาหาร</button>
            <div class="wait-status" id="waitMeal">
                <b id="mealWaitTitle">รอตรวจสอบสถานะการจ่าย</b>
                <span id="mealWaitDetail">โดยปกติแล้วรอตรวจสอบสถานะ 7 วันทำการ</span>
            </div>
        </div>

        <div class="payment-btn-box" onclick="showPaymentInfo()">
            ช่องทางการชำระเงิน
        </div>
    </div>

    <input type="file" id="fileIn" style="display:none" onchange="handleUpload()">

    <div class="bottom-info">
        พบปัญหาหรือมีข้อสงสัยโปรดติดต่อ 078 - 789 - 6789<br>
        “ เรียนดี ประพฤติเด่น เน้นคุณภาพ ซึมซาบคุุณธรรม ถูกสัมพันธ์ชุมชน ”
    </div>
</div>

<script>
    const firebaseConfig = { databaseURL: "https://schoolmonny-e6c5e-default-rtdb.asia-southeast1.firebasedatabase.app" };
    firebase.initializeApp(firebaseConfig);
    const db = firebase.database();
    
    // เปลี่ยน URL นี้ให้เป็น Web App URL ของคุณที่ได้จากการ Deploy
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
                // ค้นหาทั้ง "ชื่อ-นามสกุล" และ "ชื่อ-สกุล"
                const match = data.find(s => (s["ชื่อ_นามสกุล"] === name || s["ชื่อ_สกุล"] === name || s["ชื่อ-นามสกุล"] === name || s["ชื่อ-สกุล"] === name));
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
        document.getElementById('resName').innerText = s["ชื่อ_นามสกุล"] || s["ชื่อ_สกุล"] || s["ชื่อ-นามสกุล"] || s["ชื่อ-สกุล"];
        document.getElementById('resProg').innerText = prog;

        // ยอดหอพัก
        const fStat = s["ค่าธรรมเนียมหอพัก_สถานะ"] || "ค้างชำระ";
        const fAmt = parseFloat(String(s["ยอดค้างค่าธรรมเนียม"] || 0).replace(/,/g, '')) || 0;
        document.getElementById('resFeeStatus').innerHTML = `<span class="status-badge ${getStatClass(fStat)}">${fStat}</span>`;
        document.getElementById('resFeeAmt').innerText = fAmt.toLocaleString();

        // คำนวณยอดรวมค่าอาหาร
        let ยอดรวมทั้งหมด = 0;
        const รายการเดือน = [
            { สถานะ: "พฤษภาคม", ยอดเงิน: "ยอดที่ค้าง_พฤษภาคม_" },
            { สถานะ: "มิถุนายน", ยอดเงิน: "ยอดที่ค้าง_มิถุนายน_" },
            { สถานะ: "กรกฎาคม", ยอดเงิน: "ยอดที่ค้าง_กรกฎาคม_" },
            { สถานะ: "สิงหาคม", ยอดเงิน: "ยอดที่ค้าง_สิงหาคม_" },
            { สถานะ: "กันยายน", ยอดเงิน: "ยอดที่ค้าง_กันยายน_" },
            { สถานะ: "ตุลาคม", ยอดเงิน: "ยอดที่ค้าง_ตุลาคม_" }
        ];

        รายการเดือน.forEach((เดือน, i) => {
            const สถานะปัจจุบัน = s[เดือน.สถานะ] || "ยังไม่ถึงกำหนด";
            // ตรวจสอบชื่อ Key ให้ตรงกับที่ Firebase Clean (เปลี่ยน ( ) เป็น _ )
            const keyAmt = เดือน.ยอดเงิน;
            const ค่าดิบ = s[keyAmt] !== undefined ? String(s[keyAmt]) : "0";
            const จำนวนเงิน = parseFloat(ค่าดิบ.replace(/,/g, '')) || 0;
            
            document.getElementById(`m${i+1}`).innerHTML = `<span class="status-badge ${getStatClass(สถานะปัจจุบัน)}">${สถานะปัจจุบัน}</span>`;
            document.getElementById(`a${i+1}`).innerText = จำนวนเงิน.toLocaleString() + " บาท";
            
            if (สถานะปัจจุบัน.includes('ค้างชำระ') || สถานะปัจจุบัน.includes('รอตรวจสอบ') || สถานะปัจจุบัน.includes('ไม่สำเร็จ')) {
                ยอดรวมทั้งหมด += จำนวนเงิน;
            }
        });

        document.getElementById('resTotal').innerText = ยอดรวมทั้งหมด.toLocaleString() + " บาท";
        
        จัดการข้อความอธิบาย(s["สถานะการตรวจสอบหอพัก"], 'waitDorm', 'dormWaitTitle', 'dormWaitDetail');
        จัดการข้อความอธิบาย(s["สถานะการตรวจสอบค่าอาหาร"], 'waitMeal', 'mealWaitTitle', 'mealWaitDetail');
    }

    function จัดการข้อความอธิบาย(สถานะจากแผ่นงาน, ไอดีกล่อง, ไอดีหัวข้อ, ไอดีรายละเอียด) {
        const กล่อง = document.getElementById(ไอดีกล่อง);
        const หัวข้อ = document.getElementById(ไอดีหัวข้อ);
        const รายละเอียด = document.getElementById(ไอดีรายละเอียด);

        if (สถานะจากแผ่นงาน && สถานะจากแผ่นงาน !== "-" && สถานะจากแผ่นงาน !== "") {
            กล่อง.style.display = 'block';
            หัวข้อ.innerText = สถานะจากแผ่นงาน;
            const คลาส = getStatClass(สถานะจากแผ่นงาน);

            if (คลาส === 'paid') {
                กล่อง.style.color = '#15803d'; 
                รายละเอียด.innerText = "ขอบคุณสำหรับการแนบสลิปการโอนเพื่อยืนยันหลักฐาน";
            } else if (คลาส === 'unpaid') {
                กล่อง.style.color = '#b91c1c'; 
                รายละเอียด.innerText = "โปรดตรวจสอบสลิปให้ถูกต้องและแนบกลับมาอีกครั้ง";
            } else {
                กล่อง.style.color = '#f59e0b'; 
                รายละเอียด.innerText = "โดยปกติแล้วรอตรวจสอบสถานะ 7 วันทำการ";
            }
        } else {
            กล่อง.style.display = 'none';
        }
    }

    function getStatClass(stat) {
        if (!stat) return 'pending';
        if (stat.includes('โอนสำเร็จ') || stat.includes('ชำระแล้ว')) return 'paid';
        if (stat.includes('โอนไม่สำเร็จ') || stat.includes('ค้างชำระ')) return 'unpaid';
        if (stat.includes('รอตรวจสอบ')) return 'pending';
        if (stat.includes('ยังไม่ถึงกำหนด')) return 'not-reached';
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
            text: 'กรุณาอย่าปิดหน้าจอ',
            allowOutsideClick: false, 
            didOpen: () => Swal.showLoading() 
        });

        const reader = new FileReader();
        reader.onload = function(e) {
            // ปรับ Payload ให้สอดคล้องกับตัวแปรใน Google Apps Script (doPost)
            const payload = {
                studentName: userData["ชื่อ_นามสกุล"] || userData["ชื่อ_สกุล"] || userData["ชื่อ-นามสกุล"] || userData["ชื่อ-สกุล"],
                dormType: userSheetName, // เปลี่ยนจาก sheetName เป็น dormType
                type: uploadTarget,      // เปลี่ยนจาก paymentType เป็น type
                base64: e.target.result, // เปลี่ยนจาก fileData เป็น base64
                mimeType: file.type,
                fileName: `${uploadTarget}_${Date.now()}.png`
            };

            fetch(SCRIPT_URL, { 
                method: "POST", 
                body: JSON.stringify(payload) 
            })
            .then(res => res.json()) // รับค่าเป็น JSON
            .then(data => {
                if (data.result === "success") {
                    Swal.fire('สำเร็จ!', 'แนบสลิปเรียบร้อย ระบบจะตรวจสอบใน 7 วัน', 'success').then(() => location.reload());
                } else { 
                    throw new Error(data.message || "เกิดข้อผิดพลาดในการบันทึก"); 
                }
            })
            .catch(err => { 
                Swal.fire('ผิดพลาด', 'ไม่สามารถส่งข้อมูลได้: ' + err.message, 'error'); 
            });
        };
        reader.readAsDataURL(file);
    }

    function showPaymentInfo() {
        Swal.fire({
            title: 'ช่องทางการชำระเงิน',
            html: `<div style="text-align:center;">
                    <p>ธนาคารกสิกรไทย<br>เลขบัญชี: 123-x-xxxxx-x<br>โรงเรียนอุทยานศึกษากระบี่</p>
                    <img src="https://i.postimg.cc/wBmf1KRW/att-AB9D1Bakym-D8jp-GMVkk-V5n-39QJO5MFVpd7DBp27Jc0.jpg" style="width:100%; max-width:250px; border-radius:10px;">
                   </div>`,
            showCloseButton: true, showConfirmButton: false
        });
    }
</script>
</body>
</html>
