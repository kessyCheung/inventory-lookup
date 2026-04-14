# 請貼到新的 Claude Code 開場的完整指令

---

請先 clone 我的 repo 並閱讀交接文件，然後幫我繼續改進這個專案。

## 第一步：Clone 與熟悉

```
git clone https://github.com/kessyCheung/inventory-lookup.git
cd inventory-lookup
```

請閱讀以下檔案了解完整背景：
- `HANDOFF.md` — 專案交接文件（架構、踩坑、現狀、未來需求）
- `index.html` — HTML 結構
- `style.css` — 樣式
- `script.js` — 所有邏輯
- `data.csv` — 器材資料（約 106 筆）

## 第二步：本地跑起來

```
python3 -m http.server 8000
```
打開 http://localhost:8000 確認網站能正常顯示。

## 專案背景摘要

這是一個**器材檢索表網頁**，部署在 GitHub Pages：
- 線上網址：https://kessycheung.github.io/inventory-lookup/
- 資料來源：Google Sheets → GitHub Actions 每 30 分鐘同步 → `data.csv` → 前端讀取
- 技術：純前端 HTML + CSS + JS，用 PapaParse CDN 解析 CSV，無框架
- 目前有三種瀏覽模式：分類瀏覽 / 位置瀏覽 / 全部列表
- 有搜尋、篩選、排序功能
- 有設備詳情 modal（點卡片彈出）
- SVG 類別圖示已嵌入 script.js，不依賴外部圖片檔

## 目前的問題需要你修

### 問題 1：本地最新版的 script.js 和 style.css 可能還沒上傳到 GitHub
請比對 GitHub 上的版本和本地的版本。如果不同，幫我上傳（透過 GitHub API 或 git push）。

### 問題 2：卡片縮圖
每張卡片左邊應該有一個縮圖區域。目前：
- 有少數產品有外部圖片 URL（在 script.js 的 `PRODUCT_IMAGES` 陣列）
- 其他用類別 SVG 圖示 fallback（嵌在 `SVG_ASSETS` 物件中）
- 之前有個 bug：SVG 放在 img onerror 屬性會因為引號衝突壞掉
- 已改用 data URI + addEventListener 處理，但需要確認是否真的修好了

### 問題 3：產品圖片不足
目前只有 6 個產品有真實圖片。需要幫更多設備找到參考圖片。
設備清單在 data.csv 裡，主要設備包括：
- Sony a6400 微單相機
- INSTA 360 ONE 運動相機
- DJI Action4 運動相機
- DJI Osmo Pocket 3
- DJI Mavic Air 2 空拍機
- DJI Mini 3 Pro 空拍機
- Rode Wireless GO II 無線麥克風
- Rode Wireless PRO
- SHURE 麥克風
- Godox SL60W 補光燈
- 各種環形燈、三腳架、記憶卡等

### 問題 4：手機版需要實測
CSS 有寫 ≤640px 的 breakpoint，但還沒用真實手機測試。
未來會生成 QR code 讓同事用手機掃描查看，所以手機體驗很重要。

## 關鍵規則（請嚴格遵守）

1. **不要改 `data.csv`** — 它是 GitHub Actions 自動同步的
2. **不要改 `.github/workflows/sync-sheet.yml`** — 它目前正常運作
3. **不要改回直接從前端抓 Google Sheets** — 用 `./data.csv` 就好
4. **不要把欄位寫死** — 保持 `normalizeRow()` 動態抓取所有 CSV header 的設計
5. **data.csv 第一列必須是表頭**：`類別,編號,項目,位置`
6. **上傳到 GitHub 的方式**：如果 git push 認證失敗，就用 GitHub 網頁版「Add file → Upload files」

## 我希望你接下來幫我做的事

（以下根據你的需求修改）

1. 確認網站能正常運作，修掉任何現有 bug
2. 幫更多設備找到產品參考圖片，加入 `PRODUCT_IMAGES` 對照表
3. 確認手機版排版正確，用模擬器測試
4. （在這裡加入你的新需求...）

## script.js 架構速查

```javascript
// 嵌入的 SVG 圖示（不需要外部檔案）
SVG_ASSETS = { placeholder, camera, adapter, lighting, audio, accessory, mount }

// 產品圖片對照表 [品名關鍵字, 圖片URL]
PRODUCT_IMAGES = [ ['Osmo Pocket 3', 'https://...'], ... ]

// 設定
CONFIG = { csvUrl, categoryColors, fieldConfig, statusColors }

// 應用狀態
state = { rows, sortKey, currentTab, currentMode, searchQuery, locationFilter }

// 主要模組
DataService    — load(), normalizeRow(), getCategories(), getLocations()
FilterEngine   — getBaseFiltered(), getCurrent(), countByField(), groupByField()
ImageResolver  — getThumbUrl(), getModalImageUrl(), handleImageError()
Renderer       — renderCards(), renderTabs(), renderContent(), bindCardClicks()
Modal          — open(), close(), render()
```

## CSS 類別顏色對照

```
轉接頭: #6366f1 (indigo)    相機: #0891b2 (teal)
燈光:   #f59e0b (amber)     收音: #10b981 (green)
配件:   #8b5cf6 (purple)    支架: #64748b (gray)
```
