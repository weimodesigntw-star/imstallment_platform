# 確認事項總結

## 1. Return URL 處理方式 ✅

**已確認：使用 POST 方式**

根據 `NEWEBPAY_MPG.md` 第 126-128 行：
> "Return URL 回傳格式：與 Notify URL 相同，但透過 POST 方式回傳。"

**已更新：**
- `cursor-plan.md`：Return URL 改為 POST
- `ARCHITECTURE.md`：Return URL 改為 POST

## 2. InstFlag 值對應 ✅

**確認值對應：**
- `0` = 一次付清
- `3` = 3期
- `6` = 6期
- `12` = 12期
- `18` = 18期
- `24` = 24期

**格式：** 字串格式（如 `"3"`）

**實作建議：**
在 `NewebPayService` 中建立對應方法：

```php
/**
 * 將分期期數轉換為 InstFlag
 */
private function getInstFlag(int $installments): string
{
    $instFlagMap = [
        0 => '0',   // 一次付清
        3 => '3',   // 3期
        6 => '6',   // 6期
        12 => '12', // 12期
        18 => '18', // 18期
        24 => '24', // 24期
    ];
    
    return $instFlagMap[$installments] ?? '0';
}
```

**注意事項：**
- 如果分期期數不在支援範圍內，預設為 `"0"`（一次付清）
- InstFlag 必須是字串格式，不是整數

## 3. 環境變數命名建議 ✅

**建議命名：**
- `NEWEBPAY_MERCHANT_ID`：商店代號
- `NEWEBPAY_HASH_KEY`：HashKey
- `NEWEBPAY_HASH_IV`：HashIV
- `NEWEBPAY_ENV`：環境（test/production）
- `NEWEBPAY_RETURN_URL`：Return URL
- `NEWEBPAY_NOTIFY_URL`：Notify URL

**詳細建議請參考：** `ENV_VARIABLES.md`

**優點：**
- 符合 Laravel 環境變數命名慣例
- 前綴 `NEWEBPAY_` 清楚標示用途
- 簡潔明確，易於維護

## 4. 前端結果頁面路徑建議 ✅

**推薦路徑：**
```
/payment/result/{order_id}
```

**範例：**
- `/payment/result/123`
- `/payment/result/456`

**詳細建議請參考：** `FRONTEND_ROUTES.md`

**優點：**
- 符合 Laravel RESTful 路由慣例
- 路徑簡潔，易於維護
- 訂單 ID 是唯一識別碼，安全性較高

**實作建議：**
在 `ReturnController` 中：
```php
return redirect()->route('payment.result', [
    'order' => $order->id
]);
```

## 總結

所有確認事項已完成：

1. ✅ Return URL 使用 POST 方式
2. ✅ InstFlag 值對應確認（0, 3, 6, 12, 18, 24）
3. ✅ 環境變數命名建議已提供（見 `ENV_VARIABLES.md`）
4. ✅ 前端結果頁面路徑建議已提供（見 `FRONTEND_ROUTES.md`）

所有文件已更新，可以開始開發。

