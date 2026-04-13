# 前端質感研究 — Frontend Lab

數位信封、紋理生成、材質模擬等前端實驗。

## 實驗項目

### 1. 數位信封喜帖 (`index.html`)

在網頁上重現實體信封 + 蠟封的材質感，不依賴圖片素材，純前端實現。

- **信封**：Canvas 生成的紙張纖維紋理（噪點 + 纖維線條 + 斑塊三層）
- **蠟封**：Canvas 生成的印泥質感 + CSS 光影立體效果
- **3D 翻蓋動畫**：CSS `perspective` + `rotateX` + `preserve-3d`，蠟封跟著蓋子物理旋轉
- **陰影系統**：蓋子旋轉時 3D 空間自然遮擋，V 尖端處陰影逐漸露出
- **手機版**：全螢幕信封特寫，V 摺痕裝飾

### 2. 雙半圓信封 (`petal-envelope.html`)

上下兩片半圓翻蓋 + 愛心扣的信封樣式，三階段動畫（愛心消失 → 上蓋翻開 → 下蓋翻開 → 退場）。

### 3. 皮革信封 (`leather-envelope.html`)

深棕皮革材質信封，適合復古/紳士風格婚禮。

- **皮革紋理**：Canvas 生成的毛孔 + 皺褶 + 磨損斑塊 + 光澤層四層
- **金箔蠟封**：暖金色漸層 + 金屬閃光點
- **縫線裝飾**：CSS dashed border 模擬手工縫線

### 4. 木質信封 (`wood-envelope.html`)

木紋材質信封，適合森林系/戶外婚禮風格。

- **木紋紋理**：Canvas 用多頻率 sin 波模擬年輪 + 木纖維 + 隨機木節
- **燒印蠟封**：深木色底 + 微年輪 + 燒焦邊緣
- **木框邊飾**：CSS border 模擬木框質感

### 5. 天鵝絨信封 (`velvet-envelope.html`)

紫紅天鵝絨材質信封，適合奢華/浪漫風格婚禮。

- **絨面紋理**：Canvas 生成的斜向絨毛紋 + 方向性光斑 + 對角光帶
- **珍珠蠟封**：珍珠白底 + 虹彩色偏（角度決定色相）+ conic-gradient 光澤
- **金邊 + 微光**：radial-gradient 模擬絲絨的角度反光

### 6. 服務提案文件

電子喜帖 & 感謝卡片的客製化服務建議書與報價單。

- `proposal.md` — 提案內容（Markdown 原稿）
- `generate_pdf.py` — PDF 生成腳本（fpdf2 + 微軟正黑體）
- `2026 電子喜帖與感謝卡片 提案與報價單.pdf` — 最終輸出

## 技術筆記

見 [TECHNIQUES.md](./TECHNIQUES.md) — 包含 3D 翻蓋物理邏輯、`preserve-3d` 踩坑、Canvas 紋理生成細節。

## 技術棧

- 純 HTML / CSS / JavaScript
- Canvas 2D API（紋理生成）
- CSS 3D Transforms + `transform-style: preserve-3d`
- Google Fonts（Cormorant Garamond / Noto Serif TC）
