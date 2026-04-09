# 數位信封喜帖 — 前端質感技巧筆記

## 概述

在網頁上重現「實體信封 + 蠟封」的質感，核心挑戰是讓數位畫面看起來不像廉價的 CSS 色塊。本文件記錄開發過程中的關鍵技巧與踩坑經驗。

---

## 1. 紙張紋理：Canvas 逐像素生成

### 為什麼不用 CSS？

| 方案 | 結果 |
|------|------|
| 純色 `background-color` | 完全平面，塑膠感 |
| CSS `linear-gradient` 疊加 | 有漸層但沒有纖維感 |
| SVG `feTurbulence` filter | 偏灰、偏假，像 Photoshop 濾鏡 |
| 透明紋理 PNG 疊加 | 紋路太淡，在深色底上幾乎看不見 |
| **Canvas 逐像素生成** | **最接近真實紙張** |

### 三層疊加法

```javascript
function makePaperTexture(canvas, w, h, baseR, baseG, baseB) {
  // Layer 1: 細微噪點 — 每個像素隨機偏移 ±11
  // 模擬紙張表面的微觀不規則
  for (let i = 0; i < data.length; i += 4) {
    const n = (Math.random() - 0.5) * 22;
    data[i] += n; data[i+1] += n; data[i+2] += n;
  }

  // Layer 2: 纖維線條 — 水平方向偏移的短線段
  // 模擬紙漿中的植物纖維
  for (let f = 0; f < 1200; f++) {
    // 每條纖維: 8~43px 長，水平走向，隨機亮度
  }

  // Layer 3: 柔和斑塊 — 大面積明暗變化
  // 模擬紙張的厚薄不均
  for (let b = 0; b < 50; b++) {
    // 每個斑塊: 半徑 12~52px，用 falloff² 柔化邊緣
  }
}
```

### 關鍵參數

- Canvas 尺寸用 **2x CSS 尺寸**（如 340×240 → 680×480），確保 Retina 清晰
- 噪點強度 `22` 是甜蜜點，太低看不見，太高變電視雪花
- 纖維數量 `1200` 適合 680px 寬度，密度約 1.7 條/px
- 斑塊的 `falloff²` (quadratic) 比線性更自然

---

## 2. 蠟封/印泥質感：Canvas + CSS 多層

### 與紙張的差異

紙張是「平面 + 纖維」，印泥是「立體 + 顆粒」。需要額外處理：

1. **徑向漸層底色** — 中心亮邊緣暗，模擬凸起表面的受光
2. **帶暖色的噪點** — R 通道額外偏移，蠟天然偏暖
3. **顆粒感** — 比紙張噪點更大（±25），每個顆粒影響周圍 3×3 像素
4. **邊緣壓痕** — radial gradient 在 82%~100% 區間做亮暗交替

### CSS 輔助層（疊在 Canvas 上方）

```css
/* 頂光反射 — 模擬光源從左上打下 */
.seal-disc::after {
  background:
    radial-gradient(ellipse at 35% 28%, rgba(255,255,255,0.35) 0%, transparent 40%),
    radial-gradient(ellipse at 65% 70%, rgba(0,0,0,0.06) 0%, transparent 35%);
}

/* 邊緣立體感 — inset shadow 模擬厚度 */
.seal-shadow {
  box-shadow:
    inset 0 2px 3px rgba(255,255,255,0.4),   /* 頂部亮邊 */
    inset 0 -2px 4px rgba(0,0,0,0.08),        /* 底部暗邊 */
    inset 2px 0 3px rgba(255,255,255,0.15),    /* 左亮 */
    inset -2px 0 3px rgba(0,0,0,0.05),         /* 右暗 */
    0 3px 8px rgba(0,0,0,0.14);                /* 投射陰影 */
}

/* 凹刻文字 — text-shadow 做上暗下亮 */
.seal-text {
  color: rgba(160,150,140,0.35);
  text-shadow:
    0 -0.5px 0 rgba(0,0,0,0.06),   /* 上方暗邊 = 凹下去 */
    0 1px 0 rgba(255,255,255,0.35); /* 下方亮邊 = 光照 */
}
```

---

## 3. 信封開啟動畫：四階段 Keyframes

### 為什麼不用 3D rotateX？

| 方案 | 問題 |
|------|------|
| `rotateX(-180deg)` + `backface-visibility` | `clip-path` 和 `backface-visibility` 衝突，翻面渲染破圖 |
| 雙面 div (front + back) | 3D transform 在不同瀏覽器表現不一致 |
| **scaleY(0→1) + rotateX** | **可靠，視覺效果好** |

### 動畫時間軸

```
0.0s ─── 蠟封 scale(0) 消失
0.0s ─── 翻蓋 scaleY(0→1)，從信封頂部「長出」三角形
0.6s ─── 翻蓋 rotateX(0→170°)，向後翻開
1.5s ─── 信封整體 translateY(30px) + scale(0.95) + opacity→0
2.0s ─── 卡片 opacity→1 + scale(0.96→1)
```

```css
@keyframes flapOpen {
  0%   { transform: scaleY(0); }
  35%  { transform: scaleY(1) rotateX(0deg); }
  100% { transform: scaleY(1) rotateX(170deg); }
}
```

### 關鍵：封閉狀態不露出三角形

翻蓋用 `transform: scaleY(0)` 完全隱藏，信封就是一個乾淨的長方形。不需要 `display: none`（會斷動畫）或 `opacity: 0`（會佔空間影響點擊）。

---

## 4. 佈局結構

### 正確的 z-index 堆疊

```
z-index: 10  卡片 (opened)
z-index: 5   蠟封
z-index: 3   翻蓋 (opened, 動畫中)
z-index: 2   信封本體
z-index: 1   翻蓋 (closed, scaleY=0)
z-index: 0   卡片 (closed, opacity=0)
```

### 卡片必須在信封外面

```html
<div class="scene">
  <div class="card">...</div>      <!-- 獨立元素，不是 envelope 的子元素 -->
  <div class="envelope">
    <div class="env-body">...</div>
    <div class="flap">...</div>
    <div class="seal">...</div>
  </div>
</div>
```

如果卡片放在 envelope 裡面，`overflow: hidden` 或 `filter: drop-shadow()` 會建立新的 stacking context，卡片滑出時會被裁切。

---

## 5. 踩坑紀錄

### Canvas 尺寸為 0

`canvas.offsetWidth` 在 DOM 還沒渲染完時可能是 0。**永遠傳入明確的像素數值**，不要依賴 `offsetWidth`。

### clip-path + backface-visibility 不相容

`clip-path: polygon(...)` 會建立新的 compositing layer，導致 `backface-visibility: hidden` 失效。解決方案：不用背面，翻過 90° 後正面自然消失。

### 透明紋理 PNG 太淡

Transparent Textures 的圖片設計是給白色背景用的（透明 + 淡灰線條），在深色信封底上幾乎看不見。要嘛用 Canvas 生成不透明紋理，要嘛用掃描的真實紙張照片。

### CSS 3D transform 跑版

多層 `position: absolute` + `clip-path` + `perspective` + `transform-style: preserve-3d` 組合在不同瀏覽器會有不同表現。**盡量減少 3D transform 的使用**，用 2D 動畫模擬即可。

---

## 6. 素材來源

| 素材 | 來源 | 用途 |
|------|------|------|
| `natural-paper.png` | [Transparent Textures](https://www.transparenttextures.com/) | 背景紋理 |
| `cream-paper.png` | 同上 | 卡片紋理 |
| `textured-paper.png` | 同上 | 蠟封紋理覆蓋 |
| Canvas 生成 | 自製 | 信封 / 翻蓋 / 蠟封主紋理 |

---

## 7. 手機版：全螢幕信封特寫

### 設計思路

手機螢幕 = 信封正面的特寫。不是看到完整的信封外型，而是只看到**中間那一段** — 紙張交疊的摺痕 + 壓印。

```
┌─────────────────────┐
│   ╲               ╱ │  ← 翻蓋摺痕（從畫面外延伸進來）
│     ╲    ___    ╱   │
│       ╲ /   \ ╱     │  ← 壓印在交會點
│        | ● |        │
│       ╱ \___/ ╲     │
│     ╱           ╲   │  ← 底折摺痕
│   ╱               ╲ │
│                     │
│  這份邀請 專屬於你   │
└─────────────────────┘
```

### 摺痕角度

**錯誤**：從螢幕四角到中心（45° 對角線）— 這是看到完整信封的角度。

**正確**：線從螢幕外延伸進來，角度約 15-20°。因為手機只看到信封的中間段，摺線在特寫視角下非常平緩。

```
SVG viewBox="0 0 100 100"
上方左線：(-30, 20) → (50, 42)   // 從左邊畫面外進入
上方右線：(130, 20) → (50, 42)   // 從右邊畫面外進入
下方左線：(-30, 80) → (50, 42)   // 角度更平
下方右線：(130, 80) → (50, 42)
```

### 摺痕三層結構

每條摺痕由三層組成：

1. **寬柔陰影** (`feGaussianBlur: 0.3`)：模擬紙張高低差造成的漫射陰影
2. **銳利摺痕線** (`feGaussianBlur: 0.12`)：實際的折疊線
3. **高光線** (`rgba(255,255,255,0.06-0.08)`)：摺痕旁凸起紙面的受光面

四條線深淺不同（翻蓋壓在最上面 = 最深，底折在最下層 = 最淺）。

### 紙張層次感

`::after` 疊加兩層漸層：
- 垂直：上半微亮（翻蓋蓋住 = 多一層紙 = 略亮）、下半微暗
- 水平：左右邊緣微暗（紙張邊緣自然陰影）

### RWD 切換

```css
@media (max-width: 480px) {
  .envelope {
    position: fixed; inset: 0;     /* 撐滿螢幕 */
    width: 100vw; height: 100dvh;
  }
  .seal {
    width: 88px; height: 88px;     /* 壓印放大 */
  }
}
```

Canvas 紋理尺寸也跟著螢幕大小動態調整：
```javascript
const sw = Math.ceil(window.innerWidth * dpr);
const sh = Math.ceil(window.innerHeight * dpr);
makePaperTexture(envCanvas, sw, sh, 201, 189, 179);
```

---

## 8. 壓印立體感：凸起外圈 + 凹陷中心

### 截面示意

```
      ╱‾‾‾‾‾‾‾╲         ← 外圈凸起
     ╱    ____   ╲
    ╱   ╱      ╲   ╲     ← 中心凹陷
───╱───╱────────╲───╲─── ← 紙面
```

### CSS 實現

```css
/* 外圈凸起 — 多層 inset shadow */
.seal-shadow {
  box-shadow:
    inset 0 3px 4px rgba(255,255,255,0.45),   /* 頂部亮 = 凸起受光 */
    inset 0 -3px 5px rgba(0,0,0,0.12),         /* 底部暗 = 凸起背光 */
    0 2px 6px rgba(0,0,0,0.12);                 /* 投射陰影 = 凸出紙面 */
}

/* 中心凹陷 — 獨立 div */
.seal-center-dip {
  inset: 15%;
  box-shadow: inset 0 2px 4px rgba(0,0,0,0.08);  /* 上方暗 = 凹下去 */
  background: radial-gradient(rgba(0,0,0,0.03), transparent);
}
```

Canvas 紋理也配合：邊緣用 `radialGradient` 做壓痕（亮→暗交替模擬凸起邊緣）。

---

## 9. 可改進方向

- **陀螺儀互動**：手機傾斜時蠟封高光跟著移動（`DeviceOrientationEvent`）
- **GSAP 動畫**：更精確的時間軸控制，支援暫停/倒轉
- **真實紙張掃描**：用 300dpi 掃描實體紙張作為紋理，效果最逼真
- **WebGL 材質**：Three.js 的 MeshPhysicalMaterial 可做真實蠟面反射
