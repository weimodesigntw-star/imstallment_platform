# 架構規範

## 分層架構

本專案採用嚴格的分層架構，各層職責明確：

### Controller 層
- **職責**：只負責接參數、呼叫 service、回應結果
- **禁止**：不寫 if-else 業務邏輯
- **位置**：`app/Http/Controllers/`

### Service 層
- **職責**：集中商業邏輯（分期計算、訂單狀態判斷、藍新金流整合）
- **位置**：`app/Services/`

### Model 層
- **職責**：只負責 DB 存取
- **位置**：使用 Eloquent ORM，定義在 `app/Models/`
- **Migration**：資料庫結構定義在 `database/migrations/`

## 資料庫模型

### Plans（方案）
- 定義分期方案資訊

### Orders（訂單）
- 儲存訂單相關資訊

### Payments（付款）
- 記錄付款交易資訊

## 第三方整合

### 藍新金流（NewebPay MPG）
- 加密/解密工具：`app/Services/NewebPayService.php`
- 提供 `encryptTradeInfo`、`decryptTradeInfo`、`generateTradeSha` 三個方法
- Version：2.3

## API 端點

### POST /api/checkout
- 處理結帳流程
- Controller：`CheckoutController`
- 呼叫藍新金流相關工具

### POST /api/payment/newebpay/notify
- 處理藍新金流伺服器主動通知
- Controller：`NotifyController`
- 回應格式：`"1|OK"`

### POST /payment/newebpay/return
- 處理藍新金流付款完成後導回
- Controller：`ReturnController`
- 接收 POST 資料（與 Notify URL 相同格式）
- 重導向到訂單結果頁面

