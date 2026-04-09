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

### 2. 花瓣扣信封 (`petal-envelope.html`)

圓弧翻蓋 + 愛心扣的信封樣式，四瓣展開動畫。（開發中）

## 技術筆記

見 [TECHNIQUES.md](./TECHNIQUES.md) — 包含 3D 翻蓋物理邏輯、`preserve-3d` 踩坑、Canvas 紋理生成細節。

## 技術棧

- 純 HTML / CSS / JavaScript
- Canvas 2D API（紋理生成）
- CSS 3D Transforms + `transform-style: preserve-3d`
- Google Fonts（Cormorant Garamond / Noto Serif TC）
