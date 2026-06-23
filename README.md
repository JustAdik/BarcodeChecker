## BarcodeChecker

BarcodeChecker is a barcode scanning and inventory tracking system built using HTML, JavaScript, Google Apps Script, and Google Sheets.
The application allows users to scan barcodes using a phone or computer camera and automatically update inventory records stored in Google Sheets.

https://barcode-checker-theta.vercel.app/

## Features
- Barcode scanning using device camera
- Inventory entry (Giriş)
- Inventory exit (Çıkış)
- Automatic quantity updates
- Scan history tracking
- Product information lookup
- Real-time synchronization with Google Sheets
- Mobile-friendly interface

## Technologies Used
- HTML5
- CSS3
- JavaScript
- Google Apps Script
- Google Sheets
- html5-qrcode

## How It Works
1.User scans a barcode.
2.The application sends the barcode data to a Google Apps Script Web App.
3.Google Apps Script processes the request.
4.Product information is retrieved from predefined product data.
5.Google Sheets is updated automatically.
6.Quantity and scan statistics are recalculated.

## Installation
1.Clone the repository.
2.Deploy the Google Apps Script as a Web App.
3.Replace the Web App URL inside script.js.
4.Open index.html in a browser.
5.Allow camera access.
6.Start scanning.

## Screenshots
- Main Page
<img width="1600" height="859" alt="image" src="https://github.com/user-attachments/assets/e0dfd5fb-d348-4459-bf5a-e9e5674f48af" />

- Google Sheets Table
<img width="1600" height="661" alt="image" src="https://github.com/user-attachments/assets/aff9c699-4f17-439f-8ed2-18300aa46409" />

## Google Sheets Script

function doPost(e) {
  var sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();

  var data = JSON.parse(e.postData.contents);

  var barcode = data.barcode;
  var mode = data.mode || "giris"; 

  var products = {
    "1029384756104": { name: "BVB", type: "Hammadde", format: "Rulo (Metre)", amount: 839 },
    "192837465111": { name: "SMS", type: "Hammadde", format: "Rulo (Metre)", amount: 1300 },
    "2190384756100": { name: "PC LENS", type: "Hammadde", format: "Rulo (Metre)", amount: 2500 },
    "6491046194626": { name: "SPUNBOND", type: "Hammadde", format: "Adet", amount: 24000 },
    "1749562846921": { name: "SBS BAG", type: "Hammadde", format: "Adet", amount: 6275 },
    "7391640154929": { name: "OFFSET KUTU", type: "Hammadde", format: "Adet", amount: 240 },
    "6492046104684": { name: "TOGA", type: "Bitmiş ürün", format: "Adet", amount: 320 },
    "4750275916402": { name: "HOOD", type: "Bitmiş ürün", format: "Adet", amount: 720 }
  };

  var product = products[barcode];

  var name = "Unknown";
  var type = "-";
  var format = "-";
  var amount = 1;

  if (product) {
    name = product.name;
    type = product.type;
    format = product.format;
    amount = product.amount || 1;
  }

  var range = sheet.getDataRange().getValues();
  var foundRow = -1;

  for (var i = 0; i < range.length; i++) {
    if (range[i][1] == barcode) {
      foundRow = i + 1;
      break;
    }
  }

  var date = Utilities.formatDate(
    new Date(),
    Session.getScriptTimeZone(),
    "dd.MM.yyyy"
  );

  if (foundRow == -1) {
    var initialQuantity = mode === "cikis" ? -amount : amount;

    sheet.appendRow([
      date,             // date
      barcode,          // barcode
      name,             // product Name
      type,             // type
      format,           // format
      amount,           // initial quantity
      initialQuantity,  // last quantity
      1                 // can Count
    ]);
  } else {
    var currentQuantity = Number(sheet.getRange(foundRow, 7).getValue()) || 0;
    var currentScans = Number(sheet.getRange(foundRow, 8).getValue()) || 0;

    if (mode === "giris") {
      sheet.getRange(foundRow, 7).setValue(currentQuantity + amount);
    } else {
      sheet.getRange(foundRow, 7).setValue(currentQuantity - amount);
    }

    if (mode === "giris") {
  sheet.getRange(foundRow, 8).setValue(currentScans + 1);
} else {
  sheet.getRange(foundRow, 8).setValue(currentScans - 1);
}
  }

  return ContentService
    .createTextOutput("OK")
    .setMimeType(ContentService.MimeType.TEXT);
}
