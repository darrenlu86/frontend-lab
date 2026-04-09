# 數位信封喜帖 — 前端質感研究

在網頁上重現實體信封 + 蠟封的材質感，不依賴圖片素材，純前端實現。

## Demo

直接開啟 `index.html` 即可。

## 效果

- 信封：Canvas 生成的紙張纖維紋理（噪點 + 纖維線條 + 斑塊三層）
- 蠟封：Canvas 生成的印泥質感 + CSS 光影立體效果
- 動畫：點擊蠟封 → 翻蓋展開 → 信封退場 → 喜帖卡片進場

## 技術細節

見 [TECHNIQUES.md](./TECHNIQUES.md)

## 技術棧

- 純 HTML / CSS / JavaScript
- Canvas 2D API（紋理生成）
- CSS Keyframes（動畫）
- Google Fonts（Cormorant Garamond / Noto Serif TC）
