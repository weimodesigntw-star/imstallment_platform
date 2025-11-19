# 藍新金流 - 線上交易幕前支付技術串接手冊

## 概述

藍新金流幕前支付（MPG - Multi Payment Gateway）技術串接文件。

## API 端點

### 交易送出網址
- **測試環境**：`https://ccore.newebpay.com/MPG/mpg_gateway`
- **正式環境**：`https://core.newebpay.com/MPG/mpg_gateway`

### 回傳網址
- **Notify URL**：藍新金流伺服器主動通知的網址
- **Return URL**：付款完成後導回的網址

## 交易流程

1. 商店端產生交易參數
2. 將交易參數加密後 POST 到藍新金流
3. 使用者於藍新金流頁面完成付款
4. 藍新金流透過 Notify URL 回傳結果
5. 藍新金流透過 Return URL 導回商店頁面

## 交易參數

### 必填欄位

| 欄位名稱 | 說明 | 型別 | 範例 |
|---------|------|------|------|
| MerchantID | 商店代號 | String | MS1234567 |
| RespondType | 回傳格式 | String | JSON |
| TimeStamp | 時間戳記 | String | 1403243287 |
| Version | 串接版本 | String | 2.3 |
| MerchantOrderNo | 商店訂單編號 | String | 20140901001 |
| Amt | 訂單金額 | Integer | 200 |
| ItemDesc | 商品資訊 | String | 測試商品 |
| TradeInfo | 交易資料（加密後） | String | （加密字串） |
| TradeSha | 交易資料 SHA256 | String | （SHA256 雜湊值） |

### 選填欄位

| 欄位名稱 | 說明 | 型別 |
|---------|------|------|
| LangType | 語系 | String |
| ReturnURL | 支付完成返回商店網址 | String |
| NotifyURL | 支付通知網址 | String |
| CustomerURL | 商店取號網址 | String |
| ClientBackURL | 返回商店網址 | String |
| Email | 付款人電子信箱 | String |
| EmailModify | 電子信箱是否開放修改 | Integer |
| LoginType | 是否需要登入藍新會員 | Integer |
| OrderComment | 商店備註 | String |
| CREDIT | 信用卡一次付清啟用 | Integer |
| InstFlag | 信用卡分期付款啟用 | Integer |
| CreditRed | 信用卡紅利啟用 | Integer |
| UNIONPAY | 銀聯卡啟用 | Integer |
| WEBATM | WebATM 啟用 | Integer |
| VACC | ATM 轉帳啟用 | Integer |
| CVS | 超商代碼繳費啟用 | Integer |
| BARCODE | 超商條碼繳費啟用 | Integer |
| P2G | ezPay 電子錢包啟用 | Integer |

## 加密規則

### TradeInfo 加密

1. 將交易參數轉換為 JSON 字串
2. 使用 AES-256-CBC 加密
3. 使用商店的 HashKey 和 HashIV 作為金鑰

### TradeSha 計算

1. 取得加密後的 TradeInfo
2. 組合字串：`HashKey + TradeInfo + HashIV`
3. 使用 SHA256 雜湊計算

## 回傳資料

### Notify URL 回傳格式

```json
{
  "Status": "SUCCESS",
  "Message": "交易成功",
  "Result": {
    "MerchantID": "MS1234567",
    "Amt": 200,
    "TradeNo": "14061313541640927",
    "MerchantOrderNo": "20140901001",
    "PaymentType": "CREDIT",
    "RespondType": "JSON",
    "PayTime": "2014-06-13 13:54:14",
    "IP": "127.0.0.1",
    "EscrowBank": "HNCB",
    "AuthBank": "ESUN",
    "RespondCode": "00",
    "Auth": "123456",
    "Card6No": "123456",
    "Card4No": "7890",
    "Exp": "2506",
    "TokenUseStatus": "",
    "InstFirst": 0,
    "InstEach": 0,
    "Inst": 0,
    "ECI": "",
    "PaymentMethod": "CREDIT",
    "ChannelID": "CREDIT",
    "BankCode": "ESUN",
    "BankName": "玉山銀行",
    "PayBankCode": "",
    "PayBankName": "",
    "PayerAccount5Code": "",
    "CodeNo": "",
    "Barcode_1": "",
    "Barcode_2": "",
    "Barcode_3": "",
    "PayStore": "",
    "ExpireDate": "",
    "ExpireTime": "",
    "TradeSha": "（SHA256 雜湊值）"
  }
}
```

### Return URL 回傳格式

與 Notify URL 相同，但透過 POST 方式回傳。

## 解密規則

### TradeInfo 解密

1. 接收加密後的 TradeInfo 字串
2. 使用 AES-256-CBC 解密
3. 使用商店的 HashKey 和 HashIV 作為金鑰
4. 解析 JSON 字串取得交易資料

### 驗證 TradeSha

1. 組合字串：`HashKey + TradeInfo + HashIV`
2. 計算 SHA256 雜湊值
3. 與回傳的 TradeSha 比對

## 工具函式需求

### encryptTradeInfo
- 功能：加密交易資訊
- 輸入：交易參數物件
- 輸出：加密後的 TradeInfo 字串

### decryptTradeInfo
- 功能：解密交易資訊
- 輸入：加密後的 TradeInfo 字串
- 輸出：交易參數物件

### generateTradeSha
- 功能：產生 TradeSha 雜湊值
- 輸入：TradeInfo 字串
- 輸出：SHA256 雜湊值

## 環境變數設定

需要在環境變數中設定：

- `NEWEBPAY_MERCHANT_ID`：商店代號
- `NEWEBPAY_HASH_KEY`：HashKey
- `NEWEBPAY_HASH_IV`：HashIV
- `NEWEBPAY_RETURN_URL`：Return URL
- `NEWEBPAY_NOTIFY_URL`：Notify URL
- `NEWEBPAY_ENV`：環境（test/production）

## 注意事項

1. 所有金額以「元」為單位，不包含小數點
2. 時間戳記為 Unix timestamp（秒）
3. 商店訂單編號需唯一，不可重複
4. 加密/解密需使用正確的 HashKey 和 HashIV
5. 回傳資料需驗證 TradeSha 確保資料完整性
6. Notify URL 需回傳字串 "SUCCESS" 或 "1" 確認收到通知

## 參考資料

- 藍新金流官方文件
- MPG API 規格書

