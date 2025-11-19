# 環境變數設定建議

## 藍新金流相關環境變數

### 建議命名（Laravel 標準格式）

在 `.env` 檔案中設定：

```env
# 藍新金流設定
NEWEBPAY_MERCHANT_ID=MS1234567
NEWEBPAY_HASH_KEY=your_hash_key_here
NEWEBPAY_HASH_IV=your_hash_iv_here
NEWEBPAY_ENV=test
# 或 NEWEBPAY_ENV=production

# 藍新金流網址（可選，建議使用預設值）
NEWEBPAY_GATEWAY_URL_TEST=https://ccore.newebpay.com/MPG/mpg_gateway
NEWEBPAY_GATEWAY_URL_PRODUCTION=https://core.newebpay.com/MPG/mpg_gateway

# 回傳網址（建議使用相對路徑，Laravel 會自動轉換為完整網址）
NEWEBPAY_RETURN_URL=/payment/newebpay/return
NEWEBPAY_NOTIFY_URL=/api/payment/newebpay/notify
```

### 在 config/newebpay.php 中使用

建議建立 `config/newebpay.php` 設定檔：

```php
<?php

return [
    'merchant_id' => env('NEWEBPAY_MERCHANT_ID'),
    'hash_key' => env('NEWEBPAY_HASH_KEY'),
    'hash_iv' => env('NEWEBPAY_HASH_IV'),
    'env' => env('NEWEBPAY_ENV', 'test'),
    
    'gateway_url' => [
        'test' => env('NEWEBPAY_GATEWAY_URL_TEST', 'https://ccore.newebpay.com/MPG/mpg_gateway'),
        'production' => env('NEWEBPAY_GATEWAY_URL_PRODUCTION', 'https://core.newebpay.com/MPG/mpg_gateway'),
    ],
    
    'return_url' => env('NEWEBPAY_RETURN_URL', '/payment/newebpay/return'),
    'notify_url' => env('NEWEBPAY_NOTIFY_URL', '/api/payment/newebpay/notify'),
    
    'version' => '2.3',
];
```

### 命名說明

1. **前綴 `NEWEBPAY_`**：清楚標示這是藍新金流相關設定
2. **使用大寫與底線**：符合 Laravel 環境變數命名慣例
3. **簡潔明確**：變數名稱直接表達用途
4. **分離環境**：使用 `NEWEBPAY_ENV` 區分測試與正式環境

### 安全性建議

- **不要將 `.env` 檔案提交到 Git**
- **在正式環境使用不同的 HashKey 和 HashIV**
- **定期輪換金鑰**
- **使用 Laravel 的加密功能保護敏感資訊**

