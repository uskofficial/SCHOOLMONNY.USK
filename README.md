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

        .status-badge { padding: 2px 8px; border-radius: 4px; font-size: 13px; font-weight: 500; display: inline-block; border: 1px solid; }
        .paid { color: #15803d; border-color: #22c55e; background: #f0fdf4; } 
        .unpaid { color: #b91c1c; border-color: #ef4444; background: #fef2f2; } 
        .pending { color: #d97706; border-color: #f59e0b; background: #fffbeb; } 
        .not-reached { color: #2563eb; border-color: #60a5fa; background: #eff6ff; } 

        .boarding-box { border: 2px solid var(--border-blue); border-radius: 8px; overflow: hidden; background: #fdfdfd; }
        .boarding-header { padding: 12px 15px; border-bottom: 1px solid #ddd; font-size: 16px; font-weight: 500; background: var(--light-blue); }
        .month-grid { display: grid; grid-template-columns: repeat(3, 1fr); text-align: center; border-bottom: 1px solid #ddd; }
        .month-item { padding: 10px 5px; border-right: 1px solid #eee; border-bottom: 1px solid #eee; }
        .month-label { font-weight: 500; margin-bottom: 5px; font-size: 13px; color: #444; }
        .month-amount { font-size: 11px; color: #666; margin-top: 5px; font-weight: bold; }
        
        .total-section { padding: 15px; display: flex; justify-content: space-between; font-weight: 600; font-size: 17px; background: #fff; flex-wrap: wrap; }

        .footer-action { 
            display: flex; 
            justify-content: space-between; 
            align-items: center; 
            gap: 15px; 
            margin-top: 10px; 
            margin-bottom: 25px; 
            flex-wrap: wrap;
        }
        .btn-upload { 
            border: 2px solid var(--pink-btn); 
            color: #be185d; 
            background: white; 
            padding: 12px; 
            border-radius: 8px; 
            cursor: pointer; 
            font-size: 16px; 
            font-weight: 500; 
            flex: 1;
            min-width: 250px;
        }
        
        .wait-status { 
            text-align: right; 
            flex: 1;
            min-width: 200px;
            display: none; 
        }
        .wait-status b { font-size: 24px; display: block; margin-bottom: 2px; }
        .wait-status span { font-size: 14px; color: #666; display: block; }

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
                <b id="dormWaitTitle"></b>
                <span id="dormWaitDetail"></span>
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
                <b id="mealWaitTitle"></b>
                <span id="mealWaitDetail"></span>
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
                const match = data.find(s => (s["ชื่อ-นามสกุล"] === name || s["ชื่อ-สกุล"] === name || s["ชื่อ_นามสกุล"] === name));
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
        document.getElementById('resName').innerText = s["ชื่อ-นามสกุล"] || s["ชื่อ-สกุล"] || s["ชื่อ_นามสกุล"];
        document.getElementById('resProg').innerText = prog;

        const fStat = s["ค่าธรรมเนียมหอพัก-สถานะ"] || s["ค่าธรรมเนียมหอพัก_สถานะ"] || "ค้างชำระ";
        const fAmt = parseFloat(String(s["ยอดค้างค่าธรรมเนียม"] || 0).replace(/,/g, '')) || 0;
        
        document.getElementById('resFeeStatus').innerHTML = `<span class="status-badge ${getStatClass(fStat)}">${fStat}</span>`;
        document.getElementById('resFeeAmt').innerText = fAmt.toLocaleString();

        let ยอดรวมทั้งหมด = 0;
        const รายการเดือน = [
            { key: "พฤษภาคม", amtKey: "ยอดที่ค้าง_พฤษภาคม_" },
            { key: "มิถุนายน", amtKey: "ยอดที่ค้าง_มิถุนายน_" },
            { key: "กรกฎาคม", amtKey: "ยอดที่ค้าง_กรกฎาคม_" },
            { key: "สิงหาคม", amtKey: "ยอดที่ค้าง_สิงหาคม_" },
            { key: "กันยายน", amtKey: "ยอดที่ค้าง_กันยายน_" },
            { key: "ตุลาคม", amtKey: "ยอดที่ค้าง_ตุลาคม_" }
        ];

        รายการเดือน.forEach((เดือน, i) => {
            const สถานะปัจจุบัน = s[เดือน.key] || "ยังไม่ถึงกำหนด";
            const keyAmt = s[เดือน.amtKey] || s[`ยอดที่ค้าง(${เดือน.key})`] || 0;
            const จำนวนเงิน = parseFloat(String(keyAmt).replace(/,/g, '')) || 0;
            
            document.getElementById(`m${i+1}`).innerHTML = `<span class="status-badge ${getStatClass(สถานะปัจจุบัน)}">${สถานะปัจจุบัน}</span>`;
            document.getElementById(`a${i+1}`).innerText = จำนวนเงิน.toLocaleString() + " บาท";
            
            if (สถานะปัจจุบัน.includes('ค้างชำระ') || สถานะปัจจุบัน.includes('รอตรวจสอบ') || สถานะปัจจุบัน.includes('ไม่สำเร็จ')) {
                ยอดรวมทั้งหมด += จำนวนเงิน;
            }
        });

        const totalAmt = parseFloat(String(s["ยอดรวมค้างชำระ"] || 0).replace(/,/g, '')) || ยอดรวมทั้งหมด;
        document.getElementById('resTotal').innerText = totalAmt.toLocaleString() + " บาท";
        
        // ดึงสถานะตรวจสอบตามคีย์ใน Firebase ของคุณ (IMG_9089.jpg)
        จัดการข้อความอธิบาย(s["สถานะการตรวจสอบค่าหอพัก"], 'waitDorm', 'dormWaitTitle', 'dormWaitDetail');
        จัดการข้อความอธิบาย(s["สถานะการตรวจสอบค่าอาหาร"], 'waitMeal', 'mealWaitTitle', 'mealWaitDetail');
    }

    function จัดการข้อความอธิบาย(สถานะ, ไอดีกล่อง, ไอดีหัวข้อ, ไอดีรายละเอียด) {
        const กล่อง = document.getElementById(ไอดีกล่อง);
        const หัวข้อ = document.getElementById(ไอดีหัวข้อ);
        const รายละเอียด = document.getElementById(ไอดีรายละเอียด);

        if (สถานะ === "รอตรวจสอบการโอน") {
            กล่อง.style.display = 'block';
            หัวข้อ.innerText = "รอตรวจสอบการโอน";
            หัวข้อ.style.color = '#d97706'; // สีส้ม
            รายละเอียด.innerText = "โดยปกติแล้วรอตรวจสอบสถานะ 7 วันทำการ";
        } 
        else if (สถานะ === "การโอนสำเร็จ") {
            กล่อง.style.display = 'block';
            หัวข้อ.innerText = "การโอนสำเร็จ";
            หัวข้อ.style.color = '#15803d'; // สีเขียว
            รายละเอียด.innerText = "ขอบคุณสำหรับการหลักฐานการโอน";
        } 
        else if (สถานะ === "การโอนไม่สำเร็จ") {
            กล่อง.style.display = 'block';
            หัวข้อ.innerText = "การโอนไม่สำเร็จ";
            หัวข้อ.style.color = '#b91c1c'; // สีแดง
            รายละเอียด.innerText = "โปรดตรวจสอบสลิปให้ถูกต้องและแนบกลับมาอีกครั้ง";
        } 
        else {
            // หากไม่ใช่ 3 สถานะที่กำหนด ให้หายไป
            กล่อง.style.display = 'none';
        }
    }

    function getStatClass(stat) {
        if (!stat) return 'unpaid';
        if (stat.includes('ชำระแล้ว') || stat.includes('สำเร็จ')) return 'paid';
        if (stat.includes('ค้างชำระ') || stat.includes('ไม่สำเร็จ')) return 'unpaid';
        if (stat.includes('รอตรวจสอบ')) return 'pending';
        if (stat.includes('ยังไม่ถึงกำหนด')) return 'not-reached';
        return 'unpaid';
    }

    function triggerUpload(target) {
        uploadTarget = target;
        document.getElementById('fileIn').click();
    }

    function handleUpload() {
        const file = document.getElementById('fileIn').files[0];
        if(!file) return;
        
        Swal.fire({ title: 'กำลังส่งข้อมูล...', allowOutsideClick: false, didOpen: () => Swal.showLoading() });
        
        const reader = new FileReader();
        reader.onload = function(e) {
            const payload = {
                studentName: userData["ชื่อ-นามสกุล"] || userData["ชื่อ-สกุล"] || userData["ชื่อ_นามสกุล"],
                dormType: userSheetName,
                type: uploadTarget,
                base64: e.target.result.split(',')[1],
                mimeType: file.type,
                fileName: `${uploadTarget}_${Date.now()}.png`
            };
            
            fetch(SCRIPT_URL, { 
                method: "POST", 
                body: JSON.stringify(payload) 
            })
            .then(res => res.json())
            .then(res => {
                if (res.result === "success") {
                    Swal.fire('สำเร็จ!', 'แนบสลิปเรียบร้อย ระบบจะตรวจสอบใน 7 วัน', 'success').then(() => location.reload());
                } else { 
                    throw new Error(res.message); 
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
