# 器材檢索表 — 專案交接文件

## 一、專案概覽

這是一個**團隊器材庫存查詢工具**，部署在 GitHub Pages 上。

- **GitHub Repo**: https://github.com/kessyCheung/inventory-lookup
- **線上網址**: https://kessycheung.github.io/inventory-lookup/
- **資料來源**: Google Sheets → GitHub Actions 每 30 分鐘同步 → `data.csv`
- **技術棧**: 純前端（HTML + CSS + JS），無框架，使用 PapaParse CDN 解析 CSV

---

## 二、檔案結構

```
inventory-lookup/
├── index.html          # HTML 結構（純結構，無內嵌 CSS/JS）
├── style.css           # 所有樣式
├── script.js           # 所有邏輯（模組化 IIFE）
├── data.csv            # 器材資料（GitHub Actions 自動同步，不要手動改）
├── README.md           # 專案說明
├── HANDOFF.md          # 本交接文件
├── images/             # 圖片資料夾（目前有 SVG 但尚未成功上傳到 GitHub）
│   ├── placeholder.svg
│   └── categories/     # 6 個類別 SVG 圖示
│       ├── camera.svg, adapter.svg, lighting.svg
│       ├── audio.svg, accessory.svg, mount.svg
│       └── default.svg
└── .github/workflows/
    └── sync-sheet.yml  # Google Sheets 同步 workflow
```

---

## 三、目前已完成的功能

### 3.1 程式碼架構
- 從原本一個 544 行的 `index.html` 拆成三個檔案
- `script.js` 使用模組化 IIFE，內含：
  - `CONFIG` — 設定（CSV 路徑、類別顏色、欄位配置）
  - `SVG_ASSETS` — 6 個類別 SVG 圖示 + placeholder，直接嵌入 JS（避免需要上傳圖片檔）
  - `PRODUCT_IMAGES` — 產品圖片 URL 對照表（keyword → 外部圖片 URL）
  - `state` — 應用狀態
  - `DataService` — CSV 載入與正規化
  - `FilterEngine` — 搜尋、篩選、排序、分組
  - `ImageResolver` — 圖片路徑解析 + fallback 鏈
  - `Renderer` — 卡片、tabs、統計渲染
  - `Modal` — 設備詳情彈窗

### 3.2 三種瀏覽模式
- **分類瀏覽**: 按類別（相機、燈光、收音等）分組，有 tab 切換
- **位置瀏覽**: 按收納位置（木櫃子右中、洞洞板、大推車等）分組
- **全部列表**: 平鋪顯示所有設備

### 3.3 搜尋與篩選
- 關鍵字搜尋（跨所有欄位）帶 200ms debounce
- 位置下拉篩選
- 排序（編號 / 項目 / 位置）

### 3.4 卡片設計
- 左邊有類別彩色邊條
- 左邊有縮圖區域（產品圖或類別 SVG fallback）
- 顯示：項目名稱、編號、類別 badge、位置
- hover 有微升起 + 陰影效果
- 點擊可開啟詳情 modal

### 3.5 詳情彈窗 (Modal)
- 顯示設備大圖（產品圖 → 類別 SVG → placeholder 的 fallback 鏈）
- 顯示所有欄位資訊
- ESC / 點背景可關閉
- 有淡入 + 滑上動畫

### 3.6 未來欄位自動擴充
- `DataService.normalizeRow()` 會自動抓取 CSV 所有欄位
- `FIELD_CONFIG` 定義每個欄位的顯示規則（卡片上顯示 or 只在 modal 顯示）
- 已預設支援：狀態、借用人、借出日期、預計歸還日、備註、最後更新時間
- Google Sheets 加新欄位 → CSV 自動有 → 前端自動顯示，不需改 code

---

## 四、目前的問題 / 未完成項目

### 4.1 ⚠️ script.js 尚未成功上傳最新版到 GitHub
最後一次修改了 thumbnail 的渲染方式（修掉 onerror SVG 引號衝突），但用戶還沒來得及上傳。
**本地最新版在**: `~/Downloads/inventory-lookup/script.js`
**需要做**: 把本地的 `script.js` 和 `style.css` 上傳到 GitHub 覆蓋舊版本。

### 4.2 ⚠️ images/ 資料夾沒上傳到 GitHub
GitHub 網頁版不支援拖曳上傳資料夾。
**解決方案**: SVG 圖示已嵌入 `script.js` 的 `SVG_ASSETS` 物件中，不依賴外部 SVG 檔案。
但如果未來要放實際設備照片（.jpg），需要用 git push 或一個一個上傳到 GitHub。

### 4.3 ⚠️ Git push 認證問題
用戶的 Mac 沒有設定 GitHub CLI 或 SSH key，也沒有 Homebrew。
HTTPS push 需要 Personal Access Token，用戶嘗試過但失敗。
目前只能透過 **GitHub 網頁版** 的 「Add file → Upload files」 來更新檔案。

### 4.4 產品圖片覆蓋率不足
目前 `PRODUCT_IMAGES` 只有 6 個產品有真實圖片 URL：
- DJI Osmo Pocket 3 ✅
- DJI Action 4 ✅
- Rode Wireless GO / GO II ✅
- Rode Wireless PRO ✅
- Rode VideoMicro ✅

其他約 100 個品項都使用類別 SVG 圖示作為 fallback。
需要補充更多產品圖片 URL，或讓用戶自己拍照上傳。

### 4.5 手機版尚未實測
CSS 已經寫了手機 breakpoint（≤640px），但還沒用真實手機或模擬器測試。

---

## 五、曾經踩過的坑

### 5.1 Google Sheets CSV 直接抓取失敗
一開始用 `Papa.parse(GoogleSheetsURL)` 直接抓，踩了：
- 貼成編輯頁連結而非 CSV 連結
- pubhtml 是 HTML 不是 CSV
- 跨域不穩定
**解法**: 改用 GitHub Actions 同步到 repo 內的 `data.csv`，前端讀本地檔。

### 5.2 CSV 第一列不是表頭
Google Sheets 匯出的 CSV 第一列是 `檢索表`，真正表頭在第二列。
PapaParse 的 `header: true` 會把第一列當表頭，導致欄位對不上。
**解法**: 修正 Google Sheets，確保第一列直接是 `類別,編號,項目,位置`。

### 5.3 SVG 放在 onerror 屬性會壞掉
卡片的 `<img onerror="...">` 裡嵌入 SVG 字串，引號衝突導致 HTML 壞掉。
**解法**: 改用 `data:image/svg+xml` data URI 格式，並用 JS addEventListener 處理 error。

### 5.4 GitHub 網頁版無法上傳資料夾
「Add file → Upload files」只能上傳平面檔案，不支援資料夾結構。
**解法**: 把 SVG 圖示嵌入 JS 的 `SVG_ASSETS` 物件，完全不依賴外部檔案。

### 5.5 CSV 有多餘空欄位
Google Sheets 匯出有很多 trailing commas（空欄位）。
`normalizeRow()` 用 `.trim()` 過濾空 key，所以不影響。

---

## 六、資料結構

### 6.1 data.csv 目前欄位
```
類別,編號,項目,位置
```

### 6.2 資料分類（6 大類）
| 類別 | 數量 | 顏色代碼 | 範例 |
|------|------|----------|------|
| 相機 | ~43 | #0891b2 (teal) | Sony a6400, DJI Action4, DJI Mavic Air 2 |
| 收音 | ~17 | #10b981 (green) | Rode Wireless GO II, SHURE |
| 配件 | ~16 | #8b5cf6 (purple) | SD卡, 讀卡機, 簡報筆 |
| 支架 | ~12 | #64748b (gray) | 三腳架, 手機架, 燈架 |
| 轉接頭 | ~9 | #6366f1 (indigo) | HDMI to USBC, UGREEN 集線器 |
| 燈光 | ~7 | #f59e0b (amber) | 環形燈, Godox SL60W |

### 6.3 收納位置
木櫃子右中 (44)、洞洞板 (18)、木櫃子左上 (12)、木櫃子右上 (12)、
大推車（中）(5)、大推車（下）(4)、小推車 (4)、地上 (6)

### 6.4 未來計畫加入的欄位
```
狀態（可借 / 借出中 / 維修中 / 停用）
借用人
借出日期
預計歸還日
備註
最後更新時間
```
前端的 `FIELD_CONFIG` 已預設好這些欄位的顯示規則。

---

## 七、GitHub Actions Workflow

檔案: `.github/workflows/sync-sheet.yml`

- **觸發**: 手動 dispatch 或每 30 分鐘 (*/30 * * * *)
- **動作**: 從 Google Sheets 下載 CSV → 驗證檔案大小 → commit 到 repo
- **Google Sheets ID**: `1uQ2i6pbZY-aykeNduv7HtxS1-hl2zL-mKdEjB5gEq-s`
- **Sheet GID**: `722714951`

⚠️ 不要修改這個 workflow，它目前運作正常。

---

## 八、如何繼續開發

### 8.1 在另一個 Claude Code 中開始
1. Clone repo: `git clone https://github.com/kessyCheung/inventory-lookup.git`
2. 本地測試: `cd inventory-lookup && python3 -m http.server 8000`
3. 開啟 `http://localhost:8000` 查看
4. 修改 `index.html` / `style.css` / `script.js`
5. 上傳方式: GitHub 網頁版「Add file → Upload files」（因為 git push 認證有問題）

### 8.2 如何加更多產品圖片
在 `script.js` 的 `PRODUCT_IMAGES` 陣列中新增：
```javascript
const PRODUCT_IMAGES = [
  // [品名關鍵字, 圖片URL]
  ['Osmo Pocket 3', 'https://...'],
  ['你的新產品', 'https://圖片網址'],
];
```
品項名稱包含關鍵字就會匹配到對應圖片。

### 8.3 如何新增 CSV 欄位
1. 在 Google Sheets 新增欄位（如「狀態」）
2. 等 30 分鐘同步，或手動觸發 workflow
3. 前端會自動在 modal 中顯示新欄位
4. 如果要在卡片上也顯示，修改 `FIELD_CONFIG` 的 `cardDisplay` 設定

---

## 九、用戶的未來需求方向

1. **更多產品實際圖片** — 根據品名型號找參考圖，不只是類別概念圖
2. **QR Code** — 最終要生成 QR code 讓同事用手機掃描查看，所以手機版體驗很重要
3. **借還系統（階段 2）** — 可能需要 Google Apps Script 或其他可寫入的後端
4. **狀態篩選** — 未來加入「可借 / 借出中 / 維修中」的篩選功能

---

## 十、關鍵提醒

- ❌ 不要改回直接從前端抓 Google Sheets，用 `./data.csv` 就好
- ❌ 不要動 `.github/workflows/sync-sheet.yml`
- ❌ 不要把欄位寫死，保持動態擴充能力
- ✅ data.csv 第一列必須是表頭：`類別,編號,項目,位置`
- ✅ 所有 SVG 圖示已嵌入 script.js，不需要 images/ 資料夾也能跑
- ✅ 用戶目前只能透過 GitHub 網頁上傳檔案（沒有 git push 認證）
