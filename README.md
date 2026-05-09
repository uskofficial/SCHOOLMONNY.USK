var FIREBASE_URL = "https://schoolmonny-e6c5e-default-rtdb.asia-southeast1.firebasedatabase.app/";
var FOLDER_ID = "1sVWnX8VXh6h2L0JvnUweh5hfyZnxejRV"; 

// 1. ฟังก์ชัน Sync ข้อมูลจาก Sheet ไปยัง Firebase (คงเดิม 100%)
function syncToFirebase() {
  var ss = SpreadsheetApp.getActiveSpreadsheet();
  var sheets = ss.getSheets();
  var allData = {};

  sheets.forEach(function(sheet) {
    var name = sheet.getName();
    var data = sheet.getDataRange().getValues();
    if (data.length > 1) {
      var headers = data[0]; 
      var rows = [];
      for (var i = 1; i < data.length; i++) {
        var obj = {};
        data[i].forEach(function(val, index) {
          var headerName = headers[index] ? headers[index].toString().trim() : "Column_" + (index + 1);
          var key = headerName;
          
          if (index === 1) key = "ระดับชั้น";
          if (index === 17) key = "สถานะการตรวจสอบหอพัก";
          if (index === 18) key = "URL_สลิปค่าหอพัก";
          if (index === 19) key = "เวลาที่แนบหอพัก";
          if (index === 20) key = "สถานะการตรวจสอบค่าอาหาร";
          if (index === 21) key = "URL_สลิปค่าอาหาร";
          if (index === 22) key = "เวลาที่แนบค่าอาหาร";

          var finalValue = (val instanceof Date) ? Utilities.formatDate(val, "GMT+7", "dd/MM/yyyy HH:mm") : val;
          obj[key] = finalValue;
        });
        rows.push(obj);
      }
      allData[name] = rows;
    }
  });
  UrlFetchApp.fetch(FIREBASE_URL + ".json", {'method':'put','payload':JSON.stringify(allData)});
}

// 2. ฟังก์ชัน doPost สำหรับรับไฟล์และบันทึกลิงก์ลงคอลัมน์ S และ V
function doPost(e) {
  try {
    var data = JSON.parse(e.postData.contents);
    var ss = SpreadsheetApp.getActiveSpreadsheet();
    var sheet = ss.getSheetByName(data.sheetName);
    
    if (!sheet) return ContentService.createTextOutput("Error: ไม่พบชีต").setMimeType(ContentService.MimeType.TEXT);

    var values = sheet.getDataRange().getValues();
    var folder = DriveApp.getFolderById(FOLDER_ID);
    
    // บันทึกไฟล์รูปภาพไปยัง Google Drive
    var blob = Utilities.newBlob(Utilities.base64Decode(data.fileData.split(",")[1]), data.fileType, data.fileName);
    var file = folder.createFile(blob);
    
    // *** จุดสำคัญ: สร้างลิงก์ URL ของไฟล์ที่เพิ่งอัปโหลด ***
    var fileUrl = file.getUrl(); 
    var timestamp = new Date();

    // ค้นหาแถวนักเรียนในคอลัมน์ A
    for (var i = 1; i < values.length; i++) {
      if (values[i][0].toString().trim() === data.studentName.toString().trim()) {
        var row = i + 1;
        
        if (data.paymentType === "dorm") {
          // R: สถานะ (18) | S: URL สลิป (19) | T: เวลา (20)
          sheet.getRange(row, 18).setValue("รอตรวจสอบการโอน"); 
          sheet.getRange(row, 19).setValue(fileUrl); // บันทึกลิงก์สลิปลงคอลัมน์ S
          sheet.getRange(row, 20).setValue(timestamp); 
        } else {
          // U: สถานะ (21) | V: URL สลิป (22) | W: เวลา (23)
          sheet.getRange(row, 21).setValue("รอตรวจสอบการโอน"); 
          sheet.getRange(row, 22).setValue(fileUrl); // บันทึกลิงก์สลิปลงคอลัมน์ V
          sheet.getRange(row, 23).setValue(timestamp); 
        }
        
        // หลังจากบันทึกลิงก์ลง Sheet แล้ว ให้ Sync ไป Firebase ทันที
        syncToFirebase();
        return ContentService.createTextOutput("Success").setMimeType(ContentService.MimeType.TEXT);
      }
    }
    return ContentService.createTextOutput("Error: ไม่พบรายชื่อนักเรียน").setMimeType(ContentService.MimeType.TEXT);
  } catch (err) {
    return ContentService.createTextOutput("Error: " + err.toString()).setMimeType(ContentService.MimeType.TEXT);
  }
}
