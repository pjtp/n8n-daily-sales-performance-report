# n8n - Daily Sales Performance Report

![CDN](https://cdn.jsdelivr.net/gh/pjtp/n8n-daily-sales-performance-report@main/assets/flow.png)

## ✨ Getting Started

### 1. Mock up Google Sheets Data

![CDN](https://cdn.jsdelivr.net/gh/pjtp/n8n-daily-sales-performance-report@main/assets/mockup-data.png)

### 2. n8n Interface - Create Workflow Step by Step

#### Step 1: add Schedule Trigger

1. click "+ Add first step"
2. select "Schedule Trigger"
3. settings
   - Trigger Interval - Days
   - Trigger at Hour - 8am

#### Step 2: add Google Sheets Node

1. click + after Schedule Node select Google Sheet
2. Connect Account and authorize Google
3. Operation - Get Row(s)
4. Settings:
   - Document: Mockup - Sale Data (if authorize google complete can get all sheets)
   - Sheet: Sheet1

#### Step 3: Function Node

1. add "Function" node
2. add code below this:

```
// ดึงข้อมูลจากวันก่อนหน้า
const yesterday = new Date();
yesterday.setDate(yesterday.getDate() - 1);
const targetDate = yesterday.toISOString().split('T')[0];

const items = $input.all();
const salesData = items.filter(item => item.json.Date === targetDate);

if (salesData.length === 0) {
  return [{ json: { error: 'No data for yesterday' } }];
}

// คำนวณ metrics
const totalSales = salesData.reduce((sum, item) =>
  sum + (parseInt(item.json.Quantity) * parseInt(item.json.Unit_Price)), 0
);

const totalOrders = salesData.length;
const avgOrderValue = Math.round(totalSales / totalOrders);

// หา top product
const productSales = {};
salesData.forEach(item => {
  const total = parseInt(item.json.Quantity) * parseInt(item.json.Unit_Price);
  productSales[item.json.Product] = (productSales[item.json.Product] || 0) + total;
});
const topProduct = Object.keys(productSales).reduce((a, b) =>
  productSales[a] > productSales[b] ? a : b
);

// หา top sales rep
const repSales = {};
salesData.forEach(item => {
  const total = parseInt(item.json.Quantity) * parseInt(item.json.Unit_Price);
  repSales[item.json.Sales_Rep] = (repSales[item.json.Sales_Rep] || 0) + total;
});
const topSalesRep = Object.keys(repSales).reduce((a, b) =>
  repSales[a] > repSales[b] ? a : b
);

// ยอดขายตามภูมิภาค
const regionSales = {};
salesData.forEach(item => {
  const total = parseInt(item.json.Quantity) * parseInt(item.json.Unit_Price);
  regionSales[item.json.Region] = (regionSales[item.json.Region] || 0) + total;
});

return [{
  json: {
    date: targetDate,
    totalSales,
    totalOrders,
    avgOrderValue,
    topProduct,
    topSalesRep,
    regionSales
  }
}];
```

#### Step 4: HTML node

1. add "HTML" node
2. add HTML teplate

```
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <style>
        body { font-family: 'Arial', sans-serif; margin: 20px; }
        .header { background: #4f46e5; color: white; padding: 20px; border-radius: 8px; text-align: center; }
        .metrics { display: grid; grid-template-columns: repeat(3, 1fr); gap: 15px; margin: 20px 0; }
        .metric-card { background: #f8fafc; padding: 15px; border-radius: 8px; border-left: 4px solid #4f46e5; text-align: center; }
        .metric-value { font-size: 24px; font-weight: bold; color: #4f46e5; }
        .metric-label { font-size: 12px; color: #64748b; margin-top: 5px; }
    </style>
</head>
<body>
    <div class="header">
        <h1>📈 Daily Sales Report</h1>
        <p>วันที่: {{$json.date}}</p>
    </div>

    <div class="metrics">
        <div class="metric-card">
            <div class="metric-value">฿{{$json.totalSales.toLocaleString()}}</div>
            <div class="metric-label">ยอดขายรวม</div>
        </div>
        <div class="metric-card">
            <div class="metric-value">{{$json.totalOrders}}</div>
            <div class="metric-label">จำนวนออเดอร์</div>
        </div>
        <div class="metric-card">
            <div class="metric-value">฿{{$json.avgOrderValue.toLocaleString()}}</div>
            <div class="metric-label">ค่าเฉลี่ยต่อออเดอร์</div>
        </div>
    </div>

    <h3>🏆 Top Performers</h3>
    <p><strong>สินค้าขายดี:</strong> {{$json.topProduct}}</p>
    <p><strong>เซลส์ยอดเยี่ยม:</strong> {{$json.topSalesRep}}</p>

    <h3>🗺️ ยอดขายตามภูมิภาค</h3>
    {{#each $json.regionSales}}
    <p><strong>{{@key}}:</strong> ฿{{this.toLocaleString()}} บาท</p>
    {{/each}}
</body>
</html>
```

#### Step 5: Send Email Node

1. add "send email" node

![CDN](https://cdn.jsdelivr.net/gh/pjtp/n8n-daily-sales-performance-report@main/assets/stteing-mail.png)

![CDN](https://cdn.jsdelivr.net/gh/pjtp/n8n-daily-sales-performance-report@main/assets/mail.png)
