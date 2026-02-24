<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>公司庫存系統</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background-color: #f2f2f2;
      max-width: 700px;
      margin: auto;
      padding: 20px;
    }

    h2 { text-align: center; color: #004080; }
    h3 { margin: 10px 0; }
    input {
      width: 100%;
      padding: 10px;
      margin: 5px 0;
      border-radius: 5px;
      border: 1px solid #ccc;
    }
    button {
      width: 100%;
      padding: 15px;
      margin: 10px 0;
      font-size: 16px;
      border: none;
      border-radius: 5px;
      cursor: pointer;
      color: white;
      display: flex;
      align-items: center;
      justify-content: center;
    }
    .section {
      background: white;
      padding: 15px;
      border-radius: 10px;
      margin-bottom: 20px;
      box-shadow: 0 4px 6px rgba(0,0,0,0.1);
      position: relative;
    }
    .section-left-line {
      position: absolute;
      left: 0;
      top: 0;
      bottom: 0;
      width: 8px;
      border-radius: 5px 0 0 5px;
    }
    .result { margin-top: 5px; font-weight: bold; }
    textarea { width: 100%; height: 200px; margin-top: 10px; border-radius:5px; padding:10px; border:1px solid #ccc; }

    /* LINE 風格按鈕顏色 */
    button.in { background: #28a745; }        /* 綠色進貨 */
    button.out { background: #dc3545; }       /* 紅色出貨 */
    button.check { background: #007bff; }     /* 藍色查庫存 */
    button.inventory { background: #ffc107; color: black; } /* 黃色盤點 */
    button.report { background: #6f42c1; }    /* 紫色報表 */

    /* ICON style */
    button span.icon { margin-right: 8px; }
  </style>
</head>
<body>

<h2>📦 公司庫存管理系統</h2>

<div class="section">
  <label>操作人員</label>
  <input id="user" placeholder="請輸入姓名">
</div>

<div class="section">
  <div class="section-left-line" style="background-color:#28a745;"></div>
  <h3>進貨 / 出貨</h3>
  <input id="code" placeholder="品號">
  <input id="location" placeholder="庫位">
  <input id="qty" type="number" placeholder="數量">
  <button class="in" onclick="inStock()"><span class="icon">📦</span>進貨</button>
  <button class="out" onclick="outStock()"><span class="icon">📦</span>出貨</button>
</div>

<div class="section">
  <div class="section-left-line" style="background-color:#007bff;"></div>
  <h3>庫存查詢</h3>
  <input id="q_code" placeholder="品號查詢">
  <input id="q_location" placeholder="庫位查詢">
  <button class="check" onclick="checkStock()"><span class="icon">🔍</span>查庫存</button>
  <div id="stockResult" class="result"></div>
</div>

<div class="section">
  <div class="section-left-line" style="background-color:#ffc107;"></div>
  <h3>盤點作業</h3>
  <input id="inv_code" placeholder="品號">
  <input id="inv_location" placeholder="庫位">
  <input id="inv_qty" type="number" placeholder="實際庫存">
  <button class="inventory" onclick="doInventory()"><span class="icon">📝</span>執行盤點</button>
  <div id="invResult" class="result"></div>
</div>

<div class="section">
  <div class="section-left-line" style="background-color:#6f42c1;"></div>
  <h3>每日報表</h3>
  <input type="date" id="report_date">
  <button class="report" onclick="getReport()"><span class="icon">📊</span>產生報表</button>
  <textarea id="reportResult" readonly></textarea>
</div>

<script>
function inStock(){send("進貨",1);}
function outStock(){send("出貨",-1);}
function send(type, sign){
  const data={
    user: document.getElementById("user").value,
    code: document.getElementById("code").value,
    location: document.getElementById("location").value,
    qty: Number(document.getElementById("qty").value)*sign,
    type:type,
    name:""
  };
  google.script.run.withSuccessHandler(alert).addRecord(data);
}

function checkStock(){
  const code=document.getElementById("q_code").value;
  const location=document.getElementById("q_location").value;
  if(!code || !location){ alert("請輸入品號與庫位"); return; }
  google.script.run.withSuccessHandler(res=>{
    let msg="目前庫存: "+res.stock;
    if(res.stock < res.safe) msg += " ⚠ 低庫存!";
    document.getElementById("stockResult").innerHTML = msg;
  }).getStockWithSafe(code,location);
}

function doInventory(){
  const code=document.getElementById("inv_code").value;
  const location=document.getElementById("inv_location").value;
  const qty=Number(document.getElementById("inv_qty").value);
  const user=document.getElementById("user").value;
  if(!code || !location || !qty || !user){ alert("請填完整"); return; }
  google.script.run.withSuccessHandler(res=>{
    document.getElementById("invResult").innerHTML=res;
  }).doInventory(code,location,qty,user);
}

function getReport(){
  const date=document.getElementById("report_date").value;
  if(!date){ alert("請選日期"); return; }
  google.script.run.withSuccessHandler(res=>{
    let text="";
    for(const code in res){
      const r=res[code];
      text+=`品號:${code} 品名:${r.品名} 進貨:${r.進貨} 出貨:${r.出貨} 盤點調整:${r.盤點}\n`;
    }
    document.getElementById("reportResult").value=text;
  }).getDailyReport(date);
}
</script>

</body>
</html>
