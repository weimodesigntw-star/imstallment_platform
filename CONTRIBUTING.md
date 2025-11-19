# 開發規範

## 開發流程

### Step 0：設定上下文

選取：
- `cursor-plan.md`
- `ARCHITECTURE.md`
- 相關檔案（比如 `app/Http/Controllers`、`app/Services` 資料夾）

用 Cursor 的「Add to Context」。

### Step 1：只做 DB schema

提示：
「依照 cursor-plan.md，請幫我建立三個 Laravel Migration：create_plans_table、create_orders_table、create_payments_table，只限新增這三個 migration，不要動現有檔案。」

做完後你：
- 看 diff（超重要）
- 不滿意就直接請它「只調整欄位型別，不要動其他段落」

### Step 2：只做 Service

「在 app/Services/NewebPayService.php 新增 encryptTradeInfo、decryptTradeInfo、generateTradeSha 三個方法，依照藍新 MPG 規則（Version 2.3），並加上 PHP 型別提示。」

一樣：只新增一個檔案，不要跟其它東西扯在一起。

### Step 3：只做 POST /api/checkout

「請依照 cursor-plan.md 的 API 規格，在 app/Http/Controllers/CheckoutController.php 實作 POST /api/checkout。

僅允許修改這個檔案，並呼叫剛剛的 NewebPayService，勿改動其它檔案。」

### Step 4：Notify & Return URL 各自一個回合

- 一次只做一個 endpoint
- 做完再看 diff、跑一下程式/測試

## 開發規則

1. **新功能一律先在 `cursor-plan.md` / `TODO.md` 寫清楚再實作**

2. **API 不直接存取 DB，一律經過 service**
   - Controller/Route 只負責：接參數、呼叫 service、回應結果
   - 不寫 if-else 業務邏輯
   - Service 集中商業邏輯（分期計算、訂單狀態判斷）
   - Model/Repository 只負責 DB 存取

3. **嚴格看 diff，不一次接受全部改動**
   - 在 Cursor 每次完成修改：打開 Source Control / Git 看一下 diff
   - 如果發現「它順便幫你改了一堆無關檔案」：
     - 直接退回那個檔案的變更
     - 或跟 Cursor 說：「剛剛這段修改有動到 XXX.php 但我不想更動那個，請把那個檔案還原，只保留對 YYY.php 的修改。」
   - 也可以手動 reset 那個檔案，然後請 Cursor「再做一次，但限制檔案」

4. **每次改動請跑 `composer lint`**
   - 對 Cursor 說：「請幫我修正剛改動的檔案，使 composer lint 可以通過，不要改其它沒有 error 的檔案。」

5. **不接受大改名 / 大翻修的 PR，除非有明確說明理由**

6. **寫測試**
   - 對 Cursor 說：「幫我為 OrderService::createOrder 寫一個 PHPUnit 單元測試，檔案放在 tests/Unit/Services/OrderServiceTest.php。」

## 守門程式

在 `composer.json` 裡放好：

```json
{
  "scripts": {
    "lint": "./vendor/bin/phpcs --standard=PSR12 app/",
    "test": "phpunit",
    "test:coverage": "phpunit --coverage-html coverage"
  }
}
```

讓「程式能不能被正常使用」這件事，有客觀的指標（lint、test），而不是只靠肉眼看。

