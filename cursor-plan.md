# 開發計劃

## 專案概述

分期付款平台，整合藍新金流（NewebPay MPG）處理付款流程。

## 技術棧

- **後端框架**：Laravel (PHP)
- **資料庫**：MySQL
- **ORM**：Eloquent ORM
- **前端**：React（前端分離，後續實作）

## 資料庫模型

### Plans（方案）
定義分期付款方案資訊。

**欄位（Laravel Migration）：**
- `id` (bigIncrements) - 主鍵
- `name` (string) - 方案名稱，例如「3期0利率」
- `installments` (integer) - 分期期數（3, 6, 12, 24等）
- `interest_rate` (decimal(5,2), default(0)) - 利率（0表示0利率）
- `enabled` (boolean, default(true)) - 是否啟用
- `timestamps` - created_at, updated_at

### Orders（訂單）
儲存訂單相關資訊。

**欄位（Laravel Migration）：**
- `id` (bigIncrements) - 主鍵
- `merchant_order_no` (string, unique) - 商店訂單編號（唯一，對應藍新的 MerchantOrderNo）
- `plan_id` (unsignedBigInteger) - 關聯到 Plans
- `amount` (integer) - 訂單金額（單位：元，不含小數點）
- `item_desc` (string) - 商品資訊
- `status` (string, default('pending')) - 訂單狀態：pending（待付款）、paid（已付款）、failed（付款失敗）、cancelled（已取消）
- `customer_email` (string, nullable) - 付款人電子信箱
- `timestamps` - created_at, updated_at
- `foreign('plan_id')->references('id')->on('plans')` - 外鍵關聯

### Payments（付款）
記錄付款交易資訊，對應藍新金流的交易結果。

**欄位（Laravel Migration）：**
- `id` (bigIncrements) - 主鍵
- `order_id` (unsignedBigInteger) - 關聯到 Orders
- `trade_no` (string, nullable) - 藍新金流的交易編號（TradeNo）
- `payment_type` (string, nullable) - 付款方式（CREDIT, WEBATM, VACC, CVS等）
- `amount` (integer) - 付款金額
- `status` (string) - 付款狀態：success（成功）、failed（失敗）
- `respond_code` (string, nullable) - 回應代碼（RespondCode）
- `pay_time` (datetime, nullable) - 付款時間（PayTime）
- `trade_info` (text, nullable) - 加密的交易資訊（用於驗證）
- `trade_sha` (string, nullable) - SHA256 雜湊值（TradeSha）
- `raw_data` (text, nullable) - 原始回傳資料（JSON字串）
- `card6_no` (string, nullable) - 卡號前6碼（Card6No）
- `card4_no` (string, nullable) - 卡號後4碼（Card4No）
- `inst` (integer, nullable) - 分期期數（Inst）
- `inst_first` (integer, nullable) - 首期金額（InstFirst）
- `inst_each` (integer, nullable) - 每期金額（InstEach）
- `auth_bank` (string, nullable) - 授權銀行（AuthBank）
- `bank_code` (string, nullable) - 銀行代碼（BankCode）
- `bank_name` (string, nullable) - 銀行名稱（BankName）
- `timestamps` - created_at, updated_at
- `foreign('order_id')->references('id')->on('orders')` - 外鍵關聯

## API 規格

### POST /api/checkout
處理結帳流程，建立訂單並產生藍新金流的交易參數。

**請求參數：**
```json
{
  "plan_id": 1,              // 分期方案 ID
  "amount": 10000,           // 訂單金額（元）
  "item_desc": "測試商品",    // 商品資訊
  "customer_email": "test@example.com"  // 付款人電子信箱（選填）
}
```

**回應格式：**
```json
{
  "success": true,
  "data": {
    "order_id": 1,
    "merchant_order_no": "20240101001",
    "form_data": {
      "MerchantID": "MS1234567",
      "RespondType": "JSON",
      "TimeStamp": "1704067200",
      "Version": "2.3",
      "MerchantOrderNo": "20240101001",
      "Amt": 10000,
      "ItemDesc": "測試商品",
      "TradeInfo": "（加密後的交易資訊）",
      "TradeSha": "（SHA256 雜湊值）",
      "ReturnURL": "https://example.com/payment/newebpay/return",
      "NotifyURL": "https://example.com/api/payment/newebpay/notify",
      "Email": "test@example.com",
      "InstFlag": "3"  // 分期期數：0=一次付清, 3=3期, 6=6期, 12=12期, 18=18期, 24=24期（字串格式）
    },
    "gateway_url": "https://ccore.newebpay.com/MPG/mpg_gateway"
  }
}
```

**流程：**
1. 驗證請求參數
2. 查詢分期方案（plan_id）
3. 建立訂單記錄（Orders）
4. 產生藍新金流交易參數
   - Version 設為 "2.3"
   - InstFlag 根據 plan 的 installments 設定（0=一次付清, 3=3期, 6=6期, 12=12期, 18=18期, 24=24期）
5. 加密交易資訊（使用 encryptTradeInfo）
6. 產生 TradeSha（使用 generateTradeSha）
7. 回傳表單資料和藍新金流網址

### POST /api/payment/newebpay/notify
處理藍新金流伺服器主動通知的端點（Notify URL）。

**端點路徑：** `/api/payment/newebpay/notify`

**請求格式：**
藍新金流會 POST 以下資料：
```json
{
  "Status": "SUCCESS",
  "Message": "交易成功",
  "Result": {
    "MerchantID": "MS1234567",
    "Amt": 10000,
    "TradeNo": "14061313541640927",
    "MerchantOrderNo": "20240101001",
    "PaymentType": "CREDIT",
    "RespondCode": "00",
    "PayTime": "2014-06-13 13:54:14",
    "Card6No": "123456",
    "Card4No": "7890",
    "Inst": 3,
    "InstFirst": 3334,
    "InstEach": 3333,
    "AuthBank": "ESUN",
    "BankCode": "ESUN",
    "BankName": "玉山銀行",
    // ... 其他欄位見 NEWEBPAY_MPG.md
    "TradeSha": "（SHA256 雜湊值）"
  }
}
```

**處理流程：**
1. 接收 POST 資料
2. 解密 TradeInfo（使用 decryptTradeInfo）
3. 驗證 TradeSha（使用 generateTradeSha 比對）
4. 根據 MerchantOrderNo 查詢訂單
5. 更新訂單狀態和付款記錄
   - 如果 RespondCode 為 "00"，訂單狀態設為 "paid"，付款狀態設為 "success"
   - 儲存 Card6No, Card4No, Inst, InstFirst, InstEach, AuthBank, BankCode, BankName 等欄位
6. 回傳 "1|OK" 確認收到通知

**回應格式：**
- 成功：字串 "1|OK"
- 失敗：HTTP 錯誤狀態碼或錯誤訊息

### POST /payment/newebpay/return
處理藍新金流付款完成後導回的端點（Return URL）。

**端點路徑：** `/payment/newebpay/return`

**請求格式：**
藍新金流會透過 POST 方式回傳，格式與 Notify URL 相同：
```json
{
  "Status": "SUCCESS",
  "Message": "交易成功",
  "Result": {
    "MerchantID": "MS1234567",
    "Amt": 10000,
    "TradeNo": "14061313541640927",
    "MerchantOrderNo": "20240101001",
    "PaymentType": "CREDIT",
    "RespondCode": "00",
    "PayTime": "2014-06-13 13:54:14",
    // ... 其他欄位見 NEWEBPAY_MPG.md
    "TradeSha": "（SHA256 雜湊值）"
  }
}
```

**處理流程：**
1. 接收 POST 資料（與 Notify URL 相同格式）
2. 解密 TradeInfo（使用 decryptTradeInfo）
3. 驗證 TradeSha（使用 generateTradeSha 比對）
4. 根據 MerchantOrderNo 查詢訂單
5. 更新訂單狀態和付款記錄（與 Notify URL 相同邏輯）
6. 導向前端結果頁面（例如：`/payment/result/{order_id}`）

**回應格式：**
- 成功：重導向到訂單結果頁面
- 失敗：重導向到錯誤頁面

## 工具函式

### app/Services/NewebPayService.php
Laravel Service 類別，提供藍新金流相關功能：

- `encryptTradeInfo(array $tradeData): string` - 加密交易資訊
  - 將交易參數轉換為 JSON 字串
  - 使用 AES-256-CBC 加密
  - 使用 HashKey 和 HashIV
  
- `decryptTradeInfo(string $encryptedData): array` - 解密交易資訊
  - 使用 AES-256-CBC 解密
  - 解析 JSON 字串取得交易資料
  
- `generateTradeSha(string $tradeInfo): string` - 產生 TradeSha 雜湊值
  - 組合字串：`HashKey + TradeInfo + HashIV`
  - 使用 SHA256 雜湊計算

依照藍新 MPG 規則實作（Version 2.3）。

## 開發順序

1. 建立 Laravel Migration（plans、orders、payments 三個資料表）
2. 建立 Eloquent Model（Plan, Order, Payment）
3. 實作藍新金流 Service（NewebPayService）
4. 實作 POST /api/checkout（CheckoutController）
5. 實作 POST /api/payment/newebpay/notify（NotifyController）
6. 實作 GET /payment/newebpay/return（ReturnController）

## 注意事項

- 嚴格遵守分層架構（見 ARCHITECTURE.md）
- 每次只修改一個檔案或一個功能
- 完成後檢查 diff，確保沒有意外修改其他檔案

