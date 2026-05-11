<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SCHOOL MONEY - ระบบแจ้งจ่ายค่าใช้จ่าย</title>
    <link href="https://fonts.googleapis.com/css2?family=Kanit:wght@300;400;500;600&display=swap" rel="stylesheet">
    <script src="https://cdn.jsdelivr.net/npm/sweetalert2@11"></script>
    <script src="https://unpkg.com/@lottiefiles/dotlottie-wc@0.9.14/dist/dotlottie-wc.js" type="module"></script>

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

        /* --- Loading Overlay --- */
        #loadingOverlay {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(255, 255, 255, 0.98);
            display: none;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            z-index: 9999;
            padding: 20px;
        }
        #loadingOverlay svg { width: 100%; max-width: 600px; height: auto; }
        .school-text {
            fill: none;
            stroke: #1a237e;
            stroke-width: 1.5;
            font-size: 65px;
            font-weight: 900;
            text-transform: uppercase;
            letter-spacing: 1px;
        }
        .dash-animation { animation: dashArray 4s ease-in-out infinite, dashOffset 4s linear infinite; }
        @keyframes dashArray {
            0% { stroke-dasharray: 0 1 500 0; }
            50% { stroke-dasharray: 0 500 1 0; }
            100% { stroke-dasharray: 500 1 0 0; }
        }
        @keyframes dashOffset {
            0% { stroke-dashoffset: 1000; }
            100% { stroke-dashoffset: 0; }
        }

        /* --- Not Found Popup --- */
        #notFoundOverlay {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0, 0, 0, 0.85);
            backdrop-filter: blur(10px);
            display: none;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            z-index: 10000;
            padding: 20px;
            text-align: center;
        }
        .not-found-content {
            max-width: 500px;
            width: 100%;
            display: flex;
            flex-direction: column;
            align-items: center;
            animation: fadeInScale 0.4s ease-out;
        }
        @keyframes fadeInScale {
            from { transform: scale(0.9); opacity: 0; }
            to { transform: scale(1); opacity: 1; }
        }
        .not-found-content h2 { color: #ffffff; font-size: 32px; margin: 10px 0; font-weight: 600; }
        .not-found-content p { color: #e0e0e0; font-size: 18px; margin-bottom: 25px; line-height: 1.5; }
        .btn-close-notfound {
            background: var(--main-blue);
            color: #ffffff;
            border: none;
            padding: 12px 40px;
            border-radius: 50px;
            font-family: 'Kanit';
            font-size: 18px;
            cursor: pointer;
            box-shadow: 0 4px 15px rgba(0, 0, 0, 0.3);
        }

        .container { 
            max-width: 750px; 
            margin: auto; 
            background: white;
            padding: 20px;
            border-radius: 10px;
            width: 100%;
            box-shadow: 0 4px 6px -1px rgb(0 0 0 / 0.1);
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
            width: 100%; font-weight: 500;
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

        h3.title { font-size: 17px; color: #333; margin: 30px 0 5px; font-weight: 500; }
        h3.title span { font-size: 13px; font-weight: 300; color: #666; display: block; }

        .current-date-time {
            text-align: right;
            font-size: 14px;
            color: #1e3a8a;
            margin-bottom: 8px;
            font-weight: 400;
        }

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
            transition: 0.2s;
        }
        .btn-upload:hover { background: #fff1f2; }
        
        .wait-status { 
            text-align: right; 
            flex: 1;
            min-width: 200px;
            display: none; 
        }
        .wait-status b { font-size: 18px; display: block; margin-bottom: 2px; }
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
            .school-text { font-size: 65px; }
        }
        @media (max-width: 599px) {
            .school-text { font-size: 50px; }
            dotlottie-wc { width: 280px !important; height: 280px !important; }
        }
    </style>
</head>
<body>

<div id="loadingOverlay">
    <svg viewBox="0 0 1000 250">
        <text x="50%" y="50%" dy=".35em" text-anchor="middle" class="school-text dash-animation">
            UTTAYANSUKSAKRABI
        </text>
        <text x="50%" y="80%" dy=".35em" text-anchor="middle" class="school-text dash-animation" style="font-size: 60px;">
            SCHOOL
        </text>
    </svg>
    <div style="margin-top: 20px; color: #1a237e; letter-spacing: 2px; font-weight: 600; font-size: 18px;">กำลังค้นหาข้อมูล...</div>
</div>

<div id="notFoundOverlay">
    <div class="not-found-content">
        <dotlottie-wc src="https://lottie.host/63822361-e4de-4fc5-8b6f-a78c2d586b16/CV7OyOVb3x.lottie" style="width: 300px; height: 300px;" autoplay loop></dotlottie-wc>
        <h2>ไม่พบข้อมูลนักเรียน</h2>
        <p>ขออภัยครับ ไม่พบชื่อนี้ในระบบ<br>กรุณาตรวจสอบตัวสะกดใหม่อีกครั้ง</p>
        <button class="btn-close-notfound" onclick="closeNotFound()">ลองใหม่อีกครั้ง</button>
    </div>
</div>

<div class="container">
    <div class="header">
        <img src="https://i.postimg.cc/FzPbqZ7n/IMG-7790.png" alt="Logo" class="logo-img">
        <div class="school-info">
            <h2>SCHOOL MONEY</h2>
            <p>ระบบแจ้งจ่ายค่าใช้จ่ายโรงเรียนอุทยานศึกษากระบี่</p>
        </div>
    </div>

    <div id="searchArea" class="search-area">
        <input type="text" id="nameInput" placeholder="กรอกชื่อ-นามสกุล นักเรียน">
        <button class="btn-search" onclick="doSearch()">ค้นหา</button>
    </div>

    <div id="reportArea" style="display:none;">
        <div class="info-box">
            <div class="info-line"><span class="label">ชื่อ - สกุล :</span> <span id="resName"></span></div>
            <div class="info-line"><span class="label">โปรแกรมหอพัก :</span> <span id="resProg"></span></div>
        </div>

        <h3 class="title">รายงานประวัติและยอดค้างชำระ <span>( Payment History & Outstanding Balance )</span></h3>

        <div id="realTimeClock" class="current-date-time"></div>

        <div class="fee-card">
            <div class="fee-name">ค่าธรรมเนียมหอพัก 1/2569 <br><span style="font-weight:300; font-size:13px;">( Dormitory Fee )</span></div>
            <div style="text-align:right">
                สถานะ <span id="resFeeStatus"></span><br>
                จำนวนเงิน <b id="resFeeAmt" style="font-size:18px; color: #333;"></b>
            </div>
        </div>

        <div class="footer-action">
            <button id="btnDorm" class="btn-upload" onclick="triggerUpload('dorm')">แนบสลิปจ่ายค่าธรรมเนียมหอพัก</button>
            <div class="wait-status" id="waitDorm">
                <b id="waitDormTitle"></b>
                <span id="waitDormSub"></span>
            </div>
        </div>

        <div class="boarding-box">
            <div class="boarding-header">ค่าอาหารรายเดือน 1/2569 <span style="font-size:13px; font-weight:300;">( Boarding Fee )</span></div>
            <div id="monthGrid" class="month-grid"></div>
            <div class="total-section">
                <span style="font-size:15px;">ยอดที่ต้องชำระทั้งหมด <br><span style="font-weight:300; color:#777; font-size:12px">( เฉพาะยอดค้างค่าอาหารรายเดือน )</span></span>
                <span id="resTotal" style="color: #b91c1c; font-size: 22px;">0 บาท</span>
            </div>
        </div>

        <div class="footer-action">
            <button id="btnFood" class="btn-upload" onclick="triggerUpload('food')">แนบสลิปการจ่ายค่าอาหาร</button>
            <div class="wait-status" id="waitFood">
                <b id="waitFoodTitle"></b>
                <span id="waitFoodSub"></span>
            </div>
        </div>

        <div class="payment-btn-box" onclick="showPaymentInfo()">
            ช่องทางการชำระเงิน
        </div>

        <div style="text-align: center; margin-top: 20px;">
            <button onclick="location.reload()" style="background: none; border: none; color: #666; text-decoration: underline; cursor: pointer; font-size: 14px;">กลับหน้าค้นหา</button>
        </div>
    </div>

    <input type="file" id="fileIn" style="display:none" onchange="handleFile()">

    <div class="bottom-info">
        พบปัญหาหรือมีข้อสงสัยโปรดติดต่อ 078 - 789 - 6789<br>
        “ เรียนดี ประพฤติเด่น เน้นคุณภาพ ซึมซาบคุุณธรรม ถูกสัมพันธ์ชุมชน ”
    </div>
</div>

<script>
    const WEB_APP_URL = "https://script.google.com/macros/s/AKfycbz9TLqzzDDzCddSwyYEzCnpyOLU69pd6EErKgiJWYMownXOceaiIbbu0yJ6lBQJmosu/exec";

    let currentData = null;
    let targetType = "";
    let clockInterval = null;

    // ฟังก์ชันรันเวลา Real-time
    function startClock() {
        if (clockInterval) clearInterval(clockInterval);
        clockInterval = setInterval(() => {
            const now = new Date();
            const dateStr = now.toLocaleDateString('th-TH', { year: 'numeric', month: 'long', day: 'numeric' });
            const timeStr = now.toLocaleTimeString('th-TH');
            const clockEl = document.getElementById('realTimeClock');
            if (clockEl) clockEl.innerText = `ข้อมูล ณ วันที่: ${dateStr} | เวลา: ${timeStr} น.`;
        }, 1000);
    }

    async function doSearch() {
        const name = document.getElementById('nameInput').value.trim();
        if(!name) return;
        
        document.getElementById('loadingOverlay').style.display = 'flex';
        
        try {
            const res = await fetch(`${WEB_APP_URL}?action=search&name=${encodeURIComponent(name)}`);
            const result = await res.json();
            
            document.getElementById('loadingOverlay').style.display = 'none';

            if (result.found) {
                currentData = result;
                renderReport(result.data, result.sheetName);
                startClock();
            } else {
                showNotFound();
            }
        } catch (e) {
            document.getElementById('loadingOverlay').style.display = 'none';
            Swal.fire('ผิดพลาด', 'ไม่สามารถเชื่อมต่อฐานข้อมูลได้', 'error');
        }
    }

    function showNotFound() {
        document.getElementById('notFoundOverlay').style.display = 'flex';
    }

    function closeNotFound() {
        document.getElementById('notFoundOverlay').style.display = 'none';
    }

    function renderReport(d, sheet) {
        document.getElementById('searchArea').style.display = 'none';
        document.getElementById('reportArea').style.display = 'block';
        
        document.getElementById('resName').innerText = d["ชื่อ-นามสกุล"];
        document.getElementById('resProg').innerText = sheet;

        const dormAmt = Number(d["ยอดค้างค่าธรรมเนียม"] || 0);
        document.getElementById('resFeeStatus').innerHTML = `<span class="status-badge ${getBadgeClass(d["ค่าธรรมเนียมหอพัก-สถานะ"] || "ค้างชำระ")}">${d["ค่าธรรมเนียมหอพัก-สถานะ"] || "ค้างชำระ"}</span>`;
        document.getElementById('resFeeAmt').innerText = dormAmt.toLocaleString();
        
        toggleVerifyUI('dorm', d["สถานะการตรวจสอบค่าหอพัก"]);

        let totalFoodOnly = 0;
        const months = ["พฤษภาคม", "มิถุนายน", "กรกฎาคม", "สิงหาคม", "กันยายน", "ตุลาคม"];
        let gridHTML = "";
        
        months.forEach(m => {
            const stat = d[m] || "ยังไม่ถึงกำหนด";
            const amt = Number(d[`ยอดที่ค้าง(${m})`] || 0);
            if (stat.includes("ค้างชำระ")) { totalFoodOnly += amt; }

            gridHTML += `
                <div class="month-item">
                    <div class="month-label">${m.substring(0, 3)}</div>
                    <span class="status-badge ${getBadgeClass(stat)}">${stat}</span>
                    <div class="month-amount">${amt.toLocaleString()} บ.</div>
                </div>`;
        });
        
        document.getElementById('monthGrid').innerHTML = gridHTML;
        document.getElementById('resTotal').innerText = totalFoodOnly.toLocaleString() + " บาท";
        
        toggleVerifyUI('food', d["สถานะการตรวจสอบค่าอาหาร"]);
    }

    function getBadgeClass(s) {
        if(s.includes("ชำระแล้ว") || s.includes("สำเร็จ")) return "paid";
        if(s.includes("ค้างชำระ") || s.includes("ไม่สำเร็จ")) return "unpaid";
        if(s.includes("รอตรวจสอบ")) return "pending";
        return "not-reached";
    }

    function toggleVerifyUI(type, status) {
        const btn = document.getElementById(type === 'dorm' ? 'btnDorm' : 'btnFood');
        const wait = document.getElementById(type === 'dorm' ? 'waitDorm' : 'waitFood');
        const waitTitle = document.getElementById(type === 'dorm' ? 'waitDormTitle' : 'waitFoodTitle');
        const waitSub = document.getElementById(type === 'dorm' ? 'waitDormSub' : 'waitFoodSub');
        
        // ปุ่มแนบสลิปแสดงตลอดเวลา
        btn.style.display = 'block';

        // ถ้าแอดมินไม่ได้กรอก (null หรือว่าง) ให้ซ่อนข้อความสถานะ
        if (!status || status.trim() === "") {
            wait.style.display = 'none';
            return;
        }

        wait.style.display = 'block';

        if (status.includes("รอตรวจสอบ")) {
            waitTitle.innerText = "รอตรวจสอบ";
            waitTitle.style.color = "#d97706"; // สีส้ม
            waitSub.innerText = "โดยปกติแล้วจะตรวจสอบภายใน 7 วันทำการ";
        } else if (status.includes("การโอนสำเร็จ")) {
            waitTitle.innerText = "การโอนสำเร็จ";
            waitTitle.style.color = "#15803d"; // สีเขียว
            waitSub.innerText = "ขอบคุณสำหรับการแนบหลักฐานยืนยัน";
        } else if (status.includes("การโอนไม่สำเร็จ")) {
            waitTitle.innerText = "การโอนไม่สำเร็จ";
            waitTitle.style.color = "#b91c1c"; // สีแดง
            waitSub.innerText = "โปรดตรวจสอบสลิปให้ถูกต้องและส่งกลับมาอีกครั้ง";
        } else {
            // กรณีมีข้อความอื่นๆ
            waitTitle.innerText = status;
            waitTitle.style.color = "#333";
            waitSub.innerText = "";
        }
    }

    function triggerUpload(t) {
        targetType = t;
        document.getElementById('fileIn').click();
    }

    function handleFile() {
        const file = document.getElementById('fileIn').files[0];
        if(!file) return;
        const reader = new FileReader();
        reader.onload = async (e) => {
            Swal.fire({
                background: 'transparent',
                allowOutsideClick: false,
                showConfirmButton: false,
                html: `
                    <div style="display: flex; flex-direction: column; align-items: center; justify-content: center; text-align: center; font-family: 'Kanit', sans-serif;">
                        <dotlottie-wc src="https://lottie.host/f8ece05c-73c6-418c-8667-fb0e71af5f00/AIqGPZ1ivr.lottie" style="width: 300px; height: 300px;" autoplay loop></dotlottie-wc>
                        <h1 style="color: #ffffff; font-size: 32px; font-weight: 600; margin: 0; text-shadow: 2px 2px 8px rgba(0,0,0,0.8);">กำลังอัปโหลดสลิป...</h1>
                        <p style="color: #ffffff; font-size: 18px; margin-top: 5px; text-shadow: 1px 1px 4px rgba(0,0,0,0.8);">กรุณารอสักครู่ ห้ามปิดหน้าต่างนี้</p>
                    </div>
                `
            });

            const payload = {
                action: targetType === 'dorm' ? 'uploadDormSlip' : 'uploadFoodSlip',
                sheetName: currentData.sheetName,
                rowIndex: currentData.rowIndex,
                studentName: currentData.data["ชื่อ-นามสกุล"],
                type: targetType,
                image: e.target.result
            };
            try {
                const res = await fetch(WEB_APP_URL, { method: "POST", body: JSON.stringify(payload) });
                const result = await res.json();
                if(result.success) {
                    Swal.fire({
                        background: 'transparent',
                        showConfirmButton: true,
                        confirmButtonText: 'รับทราบ',
                        confirmButtonColor: '#1e3a8a',
                        allowOutsideClick: false,
                        html: `
                            <div style="display: flex; flex-direction: column; align-items: center; justify-content: center; text-align: center; font-family: 'Kanit', sans-serif;">
                                <dotlottie-wc src="https://lottie.host/463d8a69-81c7-4850-881c-fd73eb6b3904/3SjgwFnZ11.lottie" style="width: 300px; height: 300px;" autoplay loop></dotlottie-wc>
                                <h1 style="color: #ffffff; font-size: 38px; font-weight: 600; margin: 0; text-shadow: 3px 3px 10px rgba(0,0,0,0.8);">แนบสลิปสำเร็จ!</h1>
                                <p style="color: #ffffff; font-size: 22px; font-weight: 400; margin-top: 10px; text-shadow: 2px 2px 6px rgba(0,0,0,0.8);">ระบบได้รับข้อมูลเรียบร้อยแล้ว<br><b>โปรดรอตรวจสอบสถานะ 7 วันทำการ</b></p>
                            </div>
                        `
                    }).then(() => location.reload());
                } else {
                    Swal.fire({
                        background: 'transparent',
                        showConfirmButton: true,
                        confirmButtonText: 'ลองใหม่อีกครั้ง',
                        confirmButtonColor: '#b91c1c',
                        allowOutsideClick: false,
                        html: `
                            <div style="display: flex; flex-direction: column; align-items: center; justify-content: center; text-align: center; font-family: 'Kanit', sans-serif;">
                                <dotlottie-wc src="https://lottie.host/d5c3641f-2d14-4a40-96bd-576c3e85e166/E64C7QsOzk.lottie" style="width: 300px; height: 300px;" autoplay loop></dotlottie-wc>
                                <h1 style="color: #ffffff; font-size: 38px; font-weight: 600; margin: 0; text-shadow: 3px 3px 10px rgba(0,0,0,0.8);">อัปโหลดไม่สำเร็จ</h1>
                                <p style="color: #ffffff; font-size: 22px; font-weight: 400; margin-top: 10px; text-shadow: 2px 2px 6px rgba(0,0,0,0.8);">${result.message || 'เกิดข้อผิดพลาด'}</p>
                            </div>
                        `
                    });
                }
            } catch (err) {
                Swal.fire({
                    background: 'transparent',
                    showConfirmButton: true,
                    confirmButtonText: 'ตกลง',
                    confirmButtonColor: '#b91c1c',
                    html: `
                        <div style="display: flex; flex-direction: column; align-items: center; justify-content: center; text-align: center; font-family: 'Kanit', sans-serif;">
                            <dotlottie-wc src="https://lottie.host/d5c3641f-2d14-4a40-96bd-576c3e85e166/E64C7QsOzk.lottie" style="width: 300px; height: 300px;" autoplay loop></dotlottie-wc>
                            <h1 style="color: #ffffff; font-size: 38px; font-weight: 600; margin: 0; text-shadow: 3px 3px 10px rgba(0,0,0,0.8);">เกิดข้อผิดพลาด</h1>
                            <p style="color: #ffffff; font-size: 22px; font-weight: 400; margin-top: 10px; text-shadow: 2px 2px 6px rgba(0,0,0,0.8);">ไม่สามารถเชื่อมต่อเซิร์ฟเวอร์ได้</p>
                        </div>
                    `
                });
            }
        };
        reader.readAsDataURL(file);
    }

    function showPaymentInfo() {
        Swal.fire({
            title: 'ช่องทางการชำระเงิน',
            html: `
                <div style="text-align:center;">
                    <p style="font-size:16px; margin-bottom:10px;">ธนาคารกสิกรไทย (K-Bank)<br>เลขบัญชี: 123-x-xxxxx-x<br>ชื่อบัญชี: โรงเรียนอุทยานศึกษากระบี่</p>
                    <img src="https://i.postimg.cc/kGQgnL8J/att-AB9D1Bakym-D8jp-GMVkk-V5n-39QJO5MFVpd7DBp27Jc0.jpg" style="width:100%; max-width:280px; border-radius:10px; box-shadow: 0 4px 10px rgba(0,0,0,0.1);">
                    <p style="font-size:12px; color:#666; margin-top:10px;">* โปรดเก็บสลิปเพื่อนำมาแนบแจ้งในระบบ</p>
                </div>`,
            showCloseButton: true, showConfirmButton: false
        });
    }
</script>

</body>
</html>
