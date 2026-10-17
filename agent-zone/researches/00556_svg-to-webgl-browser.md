# 瀏覽器端 SVG 轉 WebGL 渲染方案調查

## 摘要

SVG 是向量格式（以貝茲曲線、線段、圓弧描述圖形），而 WebGL 本質上只能繪製三角形。因此「把 SVG 用 WebGL 渲染」這個問題，核心在於如何跨越兩者之間的落差。現行解決方案主要分為三條技術路線：

1. **光柵化路線（Rasterize）**：先將 SVG 點陣化為點陣圖，再當作 WebGL 材質（texture）貼上。做法簡單但失去解析度無關性。
2. **向量細分路線（Tessellation）**：把 SVG 路徑離散化、三角化為網格，直接上傳到 GPU 頂點緩衝區。保留向量特性、可在頂點著色器中動畫化，但難以完整支援 SVG 規格。
3. **GPU 向量渲染引擎路線**：以 Vello、ThorVG 等專為 GPU 設計的向量引擎（透過 WASM）直接渲染 SVG，兼顧解析度無關性與規格覆蓋率，但整合複雜度較高。

結論是：**確實存在多種可行方案**，且選擇取決於「是否需要解析度無關」「是否需要動畫」「需要多完整的 SVG 規格支援」三個維度。

## 一、瀏覽器內建光柵化：SVG → Image → WebGL 材質

最基本且最少依賴的做法，是借用瀏覽器自身的 SVG 引擎先把 SVG 畫成點陣圖，再上傳至 WebGL 當材質。

其流程為：把 SVG 字串包成 data URL 或 Blob URL，載入 `<img>` 元素（或用 `Image()`），再用 `drawImage()` 繪製到 `<canvas>` 或 `OffscreenCanvas`，最後呼叫 `gl.texImage2D()` 上傳為材質，或直接作為 Three.js 的 `CanvasTexture`[^svgtexture]。

此方法的優點是零依賴、使用瀏覽器完整的原生 SVG 引擎，支援 CSS、漸層、濾鏡、遮罩、`<foreignObject>` 等完整規格[^native]。缺點是：輸出為固定解析度的點陣圖，放大時會模糊或像素化；不能逐頂點操作向量路徑；外部資源（圖片、字型）可能觸發 canvas taint（畫布汙染）而使 `getImageData()` 失敗[^native]。此外各瀏覽器對 SVG 的渲染結果（尤其是濾鏡、文字、複雜裁切）並不一致[^native]。

## 二、Canvas 光柵化函式庫：canvg 與 resvg

若需要跨平台一致的點陣化結果，可改用純 JS 或 WASM 的 SVG 光柵化器，輸出點陣圖後再餵給 WebGL。

### canvg

canvg 是一套以純 JavaScript/TypeScript 撰寫的 SVG 解析與渲染器，將 SVG 直接繪製到 Canvas 2D 上[^canvg]。GitHub 星數約 3.8k，授權 MIT，最後一次提交約在 2025 年 10 月，維護速度較慢[^canvgstars]。它支援動畫與事件處理，且可在 Node.js 搭配 node-canvas 使用[^canvgraster]。缺點是不直接輸出像素緩衝區（須經由 canvas 再 `getImageData` 或 `createImageBitmap`），且純 JS 解析在邊界案例上不如 Rust 實作來得精確[^canvgraster]。

### resvg / resvg-js / resvg-wasm

resvg 是以 Rust 撰寫、專注於正確性的 SVG 渲染庫，目標是完整支援靜態 SVG 規格，內含約 1600 個回歸測試，被視為非瀏覽器渲染器中的規格覆蓋率標竿[^resvg]。其架構為兩段式：`usvg` 負責解析並簡化 SVG 為與渲染器無關的樹狀結構，`resvg` 再以 `tiny-skia`（純 Rust 的 CPU 光柵化器）渲染[^resvg]。GitHub 星數約 4.1k，授權 Apache 2.0/MIT，持續活躍維護[^resvgstars]。

在瀏覽器端可透過 `@resvg/resvg-wasm` npm 套件使用（底層由 resvg-js 專案編譯為 WASM），該專案星數約 2.0k，授權 MPL-2.0[^resvgwasm]。它的關鍵優勢是**直接回傳 RGBA 像素緩衝區**（`Uint8Array`），非常適合直接餵給 `gl.texImage2D()`，且輸出具跨平台像素一致性、無 canvas taint 問題[^resvgwasm]。限制在於僅支援靜態 SVG（不含動畫與腳本）、需自行提供字型緩衝區、WASM 初始化為非同步且增加約 200 至 500 KB 的體積[^resvgwasm]。

## 三、向量三角化路線：路徑細分為 WebGL 網格

這是能真正保留「向量」特性的路線，把 SVG 路徑轉為三角形網格，上傳到 GPU。一般流程為：解析 SVG 路徑 → 將曲線離散化為折線 → 對填色做三角化、對描邊產生三角帶 → 上傳頂點緩衝區 → 以著色器繪製[^webglpath]。

### Three.js SVGLoader

Three.js（星數約 116.5k）內建範例用的 `SVGLoader`，以瀏覽器 `DOMParser` 解析 SVG，走訪 DOM 轉為 `ShapePath` 物件，支援 `path`、`rect`、`circle`、`ellipse`、`polygon`、`polyline`、`line`、`use`、`g`、`style` 等元素[^threejsloader]。填色部分再交給 `ShapeGeometry`，內部使用 Earcut 演算法三角化為 `BufferGeometry`；描邊則由 `pointsToStroke()` 產生擠出的三角網格[^threejsloader]。

限制包括：**不支援 `<text>` 文字元素**（會被靜默忽略）；**不支援 SVG 濾鏡**（遇 `filter` 屬性會警告）；**不支援漸層描邊**，但**支援漸層填色**（以 `CanvasTexture` 烘焙，256×1 線性或 256×256 放射）[^threejsloader]。它同時支援行內樣式與內嵌 `<style>` 區塊，但不載入外部樣式表[^threejsloader]。這是最成熟、最廣泛被採用的 SVG 轉 WebGL 管線之一。

### Earcut 與其他三角化工具

Earcut（星數約 2.6k）是最快且最小的 JS 多邊形三角化庫（gzip 後約 4 KB），實作改良式耳切法並以 z-order 曲線最佳化，被 Three.js 與 Mapbox GL JS 內部採用[^earcut]。對於含孔洞、自相交的複雜多邊形，可改用更穩健的 `libtess.js`（GLU tessellator 移植，星數約 313）或其 C 版 `libtess2`（星數約 587）[^jsgl]。

### svg-mesh-3d 管線

Matt DesLauriers 的 `svg-mesh-3d` 示範了純 npm 模組組成的三角化管線：`parse-svg-path` → `normalize-svg-path` → `adaptive-bezier-curve` → `simplify-path` → `clean-pslg` → `cdt2d`（約束型 Delaunay 三角化）[^csssvg]。其限制為三角化耗時（不適合逐幀計算）、難以正確處理任意含孔路徑、放大時可見折線「階梯」失真，且增加打包體積[^csssvg]。

### 其他小型專案

`svg-webgl-loader`（星數僅約 4）是直接以三角化將 SVG 轉為 WebGL 的 TypeScript 小專案，解析路徑、離散化為點、三角化後以著色器繪製[^svgwebglloader]。由於星數極低且自 2021 年後未更新，實務上不建議倚賴。

## 四、框架層級：PixiJS 與 Two.js

### PixiJS

PixiJS（星數約 48.4k）是最普及的 2D WebGL/WebGPU 渲染器。v8 版支援載入 SVG 資源，且有兩條截然不同的路徑[^pixisvg]：

- **預設材質模式**：抓取 SVG 文字 → 以 `<img>` 載入 → 用 Canvas 2D `drawImage()` 繪製到離屏 canvas → 包成 `Texture`。此即前述光柵化路線，解析度固定、放大會模糊[^pixisvg]。
- **GraphicsContext 向量模式**（`parseAsGraphicsContext: true`）：以 PixiJS 自製的 `SVGParser` 解析為 `GraphicsContext` 指令，保持向量資料並由 PixiJS 引擎三角化為 GPU 幾何[^pixisvg]。

`SVGParser` 支援 `path`、`circle`、`rect`、`ellipse`、`line`、`polygon`、`polyline`、`g`、漸層等，但**不支援** `<text>`、`<image>`、`<clipPath>`、`<mask>`、`<filter>`、`<use>`、`<symbol>`、`<pattern>`、`<marker>` 及 CSS 樣式表[^pixisvg]。換言之，向量模式對複雜 SVG 支援受限，實務上複雜插圖仍常退回材質模式。

### Two.js

Two.js（星數約 8.7k）是渲染器無關的 2D 繪圖 API，同一套 API 可輸出 SVG、Canvas2D 與 WebGL；其 WebGL 渲染器會將向量圖形三角化為 GPU 幾何[^twojs]。它能透過 `two.interpret()` 匯入 SVG 路徑，但本質是繪圖庫而非完整 SVG 檔案渲染器，僅支援基本圖形與路徑[^twojs]。

## 五、GPU 向量渲染引擎：Vello 與 ThorVG

這是目前最先進的方向：專為 GPU 設計的向量渲染引擎，以 WASM 編譯後在瀏覽器直接渲染 SVG。

### Vello（Linebender）

Vello（星數約 4.4k）是 Rust 撰寫的高效能 2D 向量渲染器，內含三種實作：`vello_cpu`（CPU/SIMD 多執行緒，最成熟）、`vello_gpu`（GPU 加速，採 Sparse Strips 架構，支援 wgpu 與 **WebGL2** 後端）、以及研究階段的 compute-shader 版本[^vello]。其 WebGL2 後端透過 `webgl` feature flag 啟用，並提供獨立的 `WebGlRenderer` 結構，可編譯至 `wasm32-unknown-unknown`，官方已有「Vello GPU Direct WebGL2 Demo」實際於瀏覽器中運行[^vello]。

然而 Vello **並非 SVG 渲染器**：它渲染 SVG 所描述的向量圖元（路徑、填色、描邊、漸層、影像、文字、裁切），但不提供公開的 SVG 解析 API（僅有供內部開發用的 `pico_svg` feature）[^vello]。因此若要渲染 SVG，須自行解析後呼叫其 scene API。此外 Vello GPU 對遮罩層、複雜濾鏡圖、部分混合模式尚未支援（會 panic），且需自行編譯 WASM（npm 上無預編譯發行版）[^vello]。

### ThorVG

ThorVG（星數約 1.9k，授權 MIT）是目前**唯一專門為 SVG + GPU 打造的完整向量引擎**。它以 C++ 撰寫，提供四種渲染後端：CPU/SIMD、OpenGL/ES、**WebGL** 與 **WebGPU**[^thorvg]。官方指出 WebGPU 後端相較 OpenGL 後端平均約有 1.8 倍吞吐量提升，在描邊、漸層與影像渲染上增益最大[^thorvgweb]。ThorVG 已被 Godot、Tizen OS、LVGL、LottieFiles、Camtasia 等採用，支援 SVG Tiny 1.2 與 Lottie[^thorvg]。

在瀏覽器端，ThorVG 以 Emscripten 編譯為 WASM，可將 SVG 渲染到 `<canvas>`，提供兩種使用方式[^thorvgweb]：

- **`@thorvg/webcanvas`** npm 套件：可指定 `renderer: 'gl'`（WebGL）或 `'wg'`（WebGPU），載入 SVG 後加入 canvas 渲染。
- **`@thorvg/lottie-player`** 網頁元件：以 `fileType="svg"` 與 `renderConfig` 指定後端，並提供 `svg-static-render-gl.html`、`svg-static-render-wg.html`、`svg-static-render-sw.html` 三種後端的官方範例。

官方亦有多個瀏覽器展示（如 Thor Janitor、Thor Pirates、Thor Marble Race、ThorVG Playground 等），顯示其 WebGL/WebGPU 後端已達生產可用狀態[^thorvgweb]。

## 六、其他技術：Stencil Buffer 與 Loop-Blinn

### Stencil Buffer 技術

使用模板緩衝區（stencil buffer）繪製含孔洞的凹多邊形：對每個邊緣交叉增減模板值，最終只保留模板值非零的像素，可省去 CPU 端三角化[^csssvg]。NanoVG、PixiJS 的複雜圖形繪製即採用此法[^csssvg]。缺點是多數瀏覽器下缺乏 MSAA 反鋸齒（須搭配 FXAA 後處理），且逐像素操作對填充率負擔較大[^csssvg]。

### Loop-Blinn 曲線渲染

Loop 與 Blinn（2005）提出直接在 GPU 上渲染三次貝茲曲線、無須三角化的解析度無關技術：將曲線繪為三角形並把控制點編碼於頂點屬性，由片段著色器逐像素評估曲線方程[^csssvg]。此法可提供最佳曲線品質與 GPU 原生反鋸齒，但**已被專利保護**（美國專利 US20070097123），不適合開源專案，且對重疊、自相交曲線有大量邊界案例，因而未見於主流開源 WebGL 庫[^csssvg]。

## 七、方案比較與選型建議

| 路線 | 代表工具 | 解析度無關 | GPU 可動畫 | 反鋸齒 | SVG 規格覆蓋 | 難度 |
|---|---|---|---|---|---|---|
| 瀏覽器光柵化為材質 | 原生 `Image` + `texImage2D`、Three.js `TextureLoader` | 否 | 否 | 瀏覽器原生 | 完整（瀏覽器引擎） | 極易 |
| Canvas 光柵化為材質 | canvg、@resvg/resvg-wasm | 否 | 否 | Canvas 原生 | canvg 高、resvg 最高 | 易 |
| 路徑三角化為網格 | Three.js SVGLoader、svg-mesh-3d、earcut | 是 | 是 | 需 MSAA/FXAA | 中（不含文字/濾鏡） | 難 |
| 模板緩衝區 | NanoVG、PixiJS Graphics | 是 | 有限 | 瀏覽器缺 MSAA | 僅限圖形 | 中 |
| GPU 向量引擎 | Vello（WebGL2）、ThorVG（WebGL/WebGPU） | 是 | 是 | 內建 | ThorVG 完整、Vello 成長中 | 難（需 WASM） |
| Loop-Blinn 曲線 | （受專利保護，無主流實作） | 是 | 是 | 內建 | 極難實作 | 極難 |

**選型建議**：

- **要最完整 SVG 規格 + 最省事** → 瀏覽器原生光柵化為材質（幾行程式即完成），或 `@resvg/resvg-wasm`（跨平台像素一致、直接輸出 RGBA）。
- **要真正的向量 SVG 於 WebGL 渲染，且接受規格限制** → **ThorVG**（WebGL 與 WebGPU 雙後端、生產可用、有現成 npm 套件與範例）是當前最完整的答案。
- **要成熟、廣泛採用的三角化管線** → Three.js `SVGLoader` + `ShapeGeometry`，或直接用 `earcut` 自建管線。
- **要最先進的 GPU 向量渲染但願意自行整合 WASM** → Vello（WebGL2/wgpu 後端）；但須注意它不直接解析 SVG。
- **要 GPU 動畫化向量形狀** → 三角化路線是唯一選擇（靜態建構幾何一次，之後以頂點著色器動畫）。

整體而言，「SVG 轉 WebGL 渲染」並無單一霸主方案：光柵化路線簡單但非真向量；三角化路線是真向量但規格支援受限；GPU 向量引擎（尤其 ThorVG）最接近理想，但需要 WASM 整合的投入。實務上多依「解析度無關」「規格覆蓋」「動畫需求」三者權衡取捨。

## 參考資料

[^svgtexture]: Salivity. (n.d.). Using SVG assets as textures in WebGL. Retrieved 2026-10-17, from https://salivity.github.io/svg/article/using-svg-assets-as-textures-in-web-gl/

[^native]: MDN Web Docs. (n.d.). CanvasRenderingContext2D: drawImage() method. Retrieved 2026-10-17, from https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D/drawImage

[^canvg]: canvg. (n.d.). canvg/canvg: JavaScript SVG parser and renderer on Canvas. GitHub. Retrieved 2026-10-17, from https://github.com/canvg/canvg

[^canvgstars]: canvg. (2025, October 18). Commit history. GitHub. Retrieved 2026-10-17, from https://github.com/canvg/canvg/commits/master

[^canvgraster]: thx. (n.d.). resvg-js benchmark: canvg comparison. GitHub. Retrieved 2026-10-17, from https://github.com/thx/resvg-js

[^resvg]: Linebender. (n.d.). linebender/resvg: An SVG rendering library. GitHub. Retrieved 2026-10-17, from https://github.com/linebender/resvg

[^resvgstars]: Linebender. (2026, October 16). Commit history. GitHub. Retrieved 2026-10-17, from https://github.com/linebender/resvg/commits/main

[^resvgwasm]: thx. (n.d.). @resvg/resvg-wasm. npm. Retrieved 2026-10-17, from https://unpkg.com/@resvg/resvg-wasm/package.json

[^webglpath]: DesLauriers, M. (n.d.). Rendering SVG paths in WebGL. CSS-Tricks. Retrieved 2026-10-17, from https://css-tricks.com/rendering-svg-paths-in-webgl/

[^threejsloader]: three.js. (n.d.). SVGLoader. three.js documentation. Retrieved 2026-10-17, from https://threejs.org/docs/#examples/en/loaders/SVGLoader

[^earcut]: Mapbox. (n.d.). mapbox/earcut: The fastest and smallest JavaScript polygon triangulation library. GitHub. Retrieved 2026-10-17, from https://github.com/mapbox/earcut

[^jsgl]: Mapbox. (n.d.). mapbox/libtess2: A refactor of libtess2. GitHub. Retrieved 2026-10-17, from https://github.com/memononen/libtess2

[^csssvg]: DesLauriers, M. (n.d.). svg-mesh-3d. GitHub. Retrieved 2026-10-17, from https://github.com/mattdesl/svg-mesh-3d/

[^svgwebglloader]: MoeYc. (n.d.). svg-webgl-loader. GitHub. Retrieved 2026-10-17, from https://github.com/MoeYc/svg-webgl-loader

[^pixisvg]: PixiJS. (n.d.). Assets — supported file types. PixiJS documentation. Retrieved 2026-10-17, from https://pixijs.com/8.x/guides/components/assets

[^twojs]: br1, J. (n.d.). jonobr1/two.js: A renderer agnostic two-dimensional drawing api for the web. GitHub. Retrieved 2026-10-17, from https://github.com/jonobr1/two.js

[^vello]: Linebender. (n.d.). linebender/vello: A GPU compute-centric 2D renderer. GitHub. Retrieved 2026-10-17, from https://github.com/linebender/vello

[^thorvg]: ThorVG. (n.d.). thorvg/thorvg: A production-ready C++ vector graphics engine. GitHub. Retrieved 2026-10-17, from https://github.com/thorvg/thorvg

[^thorvgweb]: ThorVG. (n.d.). ThorVG — The vector graphics engine. Retrieved 2026-10-17, from https://www.thorvg.org/
