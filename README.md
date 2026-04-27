# 學生自我監控系統

純前端、單檔 `index.html` 的課堂自我監控工具。  
提供打卡、服藥、教務處檢核、CSV 匯出與 Google Sheet 同步。

## 目前功能

- 課表範圍：`週一～週五`（固定節次：早自習到第七節，含下課、午餐/午休、掃地）
- 顯示模式：
  - `當天`：自動判斷今天星期幾，只顯示當日
  - `當週`：顯示整週
- 若今天是週末且切到 `當天`，畫面顯示 `無課堂`
- 表頭顯示 `星期 + 日期`（例如 `週一 4/28`）

### 記錄模式

- **打卡**：`簽到`、`簽退`
- **服藥**：在 `早自習`、`午餐/午休` 顯示 `服藥` 按鈕
- **教務處檢核**（指定節次）：兩項 checkbox（完成教學規劃表活動 / 未離開教務處）+ `✔`
- **學習節次**：目前顯示為空白格（不再有檢核項目）

### 顏色狀態

- `normal`：綠色（正常/完成）
- `pullout`：黃色（教務處檢核節次）
- `warning`：紅色（檢核未完成）

## 儲存與匯出

- `localStorage` 自動保存：
  - 打卡與服藥時間
  - 教務處檢核勾選
  - 顯示模式（當天/當週）
- `匯出資料`（CSV）：
  - 當週模式：匯出整週
  - 當天模式：只匯出當天
  - 當天為週末（無課堂）時，匯出按鈕會 disabled
  - 只匯出有資料的列（避免空白列）
  - 欄位固定為：`星期`、`節次`、`類型`、`簽到`、`簽退`、`服藥`、`完成規劃表活動`、`未離開學務處`

## Google Sheet 同步

右上角有 `同步 Google Sheet` 按鈕，會將目前模式可見資料送到 Apps Script Web App。

### 同步狀態燈號

同步按鈕旁有燈號與文字狀態：

- 灰燈：`未同步`
- 黃燈：`等待同步` / `同步中`
- 綠燈：`同步成功`
- 紅燈：`同步失敗`

### 自動同步機制

- 使用者有操作會啟動同步倒數（預設約 3 秒）
- 若倒數期間又有新操作，會重設同一個計時器（debounce）
- 同一時間只會有一個同步請求（避免競爭）
- 若同步進行中又有新操作，會在本次完成後再排下一次同步
- 觸發操作包含：
  - 簽到 / 簽退 / 服藥
  - checkbox 勾選變更
  - 按 `✔` 儲存教務處檢核

### 前端設定（`index.html`）

- `GOOGLE_SHEET_WEB_APP_URL`：Apps Script Web App URL
- `GOOGLE_SHEET_API_KEY`：同步用 API key（預設 `YOUR_SIMPLE_KEY`）
- `GOOGLE_SHEET_API_KEY_STORAGE_KEY`：使用者輸入 key 的 localStorage key
- `AUTO_SYNC_DELAY_MS`：自動同步延遲毫秒數

畫面上 `當週` 右邊有 `設定 API Key` 按鈕，可讓使用者輸入並儲存 API key（重新整理後保留）。

送出的 payload 格式：

- `apiKey`
- `rows`（陣列）
  - `recordId`
  - `savedAt`
  - `day`
  - `date`
  - `period`
  - `type`
  - `checkIn`
  - `checkOut`
  - `medicine`
  - `check1`
  - `check2`
  - `check3`
  - `activity`
  - `viewMode`

## 建議 Apps Script `doPost`

請在 Google Apps Script 端：

- 使用 `getActiveSpreadsheet()` 或 `openById()` 指定目標試算表
- 用 `getSheetByName("Data")` 指定寫入分頁
- 驗證 `apiKey === "YOUR_SIMPLE_KEY"`
- 接收 `rows` 並批次寫入

## 使用方式

1. 用瀏覽器開啟 `index.html`
2. 左上切換 `當天` 或 `當週`
3. 若需要，按 `設定 API Key` 輸入 Google Apps Script 驗證 key
4. 依節次填寫紀錄（會自動排程同步）
5. 右上可選：
   - `匯出資料`（CSV）
   - `同步 Google Sheet`

## 可調整設定

可在 `index.html` 調整：

- `days`：顯示哪些星期
- `schedule`：節次與類型
- `selfPeriods`：哪些 `週X_節次` 顯示教務處檢核
- `GOOGLE_SHEET_WEB_APP_URL`：雲端同步端點
- `GOOGLE_SHEET_API_KEY`：同步金鑰
- `AUTO_SYNC_DELAY_MS`：自動同步延遲

## 目前限制

- 目前無帳號系統與權限管理
- 資料以單機瀏覽器為主，跨裝置需仰賴 Google Sheet 同步
- 未提供歷史查詢頁與圖表儀表板
