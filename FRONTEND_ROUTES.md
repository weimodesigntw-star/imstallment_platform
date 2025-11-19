# 前端路由建議

## 付款結果頁面路由

### 建議路徑

根據 Laravel 的 RESTful 路由慣例，建議使用以下路徑：

#### 方案一：簡潔路徑（推薦）
```
/payment/result/{order_id}
```
- **優點**：簡潔明瞭，符合 RESTful 風格
- **用途**：顯示訂單付款結果
- **範例**：`/payment/result/123`

#### 方案二：使用 merchant_order_no
```
/payment/result/{merchant_order_no}
```
- **優點**：使用商店訂單編號，對前端更友善
- **用途**：顯示訂單付款結果
- **範例**：`/payment/result/20240101001`

#### 方案三：使用訂單狀態參數
```
/payment/result/{order_id}?status={status}
```
- **優點**：可以根據狀態顯示不同頁面
- **用途**：顯示訂單付款結果，可區分成功/失敗
- **範例**：`/payment/result/123?status=success`

### 推薦方案

**建議使用方案一：`/payment/result/{order_id}`**

**理由：**
1. 符合 Laravel RESTful 路由慣例
2. 路徑簡潔，易於維護
3. 訂單 ID 是唯一識別碼，安全性較高
4. 可以透過訂單 ID 查詢完整訂單資訊

### Return Controller 實作建議

在 `ReturnController` 中：

```php
public function handle(Request $request)
{
    // ... 處理藍新回傳資料 ...
    
    // 重導向到結果頁面
    return redirect()->route('payment.result', [
        'order' => $order->id
    ]);
}
```

### 路由定義建議

在 `routes/web.php` 中：

```php
// 付款結果頁面（前端路由）
Route::get('/payment/result/{order}', [PaymentResultController::class, 'show'])
    ->name('payment.result');
```

### 錯誤處理

如果訂單不存在或處理失敗，建議重導向到：

```
/payment/error?message={error_message}
```

或使用 Laravel 的錯誤頁面：

```
/errors/payment-failed
```

### 前端整合建議

1. **結果頁面應顯示：**
   - 訂單編號
   - 付款狀態（成功/失敗）
   - 付款金額
   - 付款時間
   - 分期資訊（如果有）

2. **SEO 友善：**
   - 使用有意義的 URL
   - 避免使用過長的查詢參數

3. **安全性：**
   - 驗證訂單是否屬於當前使用者（如果有登入系統）
   - 不要在前端顯示敏感資訊（如完整卡號）

