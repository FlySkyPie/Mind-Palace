# 以 Web 技術實作可縮放、可拖曳的「畫布 UX」：Three.js 正交相機與 CSS2D/CSS3D Renderer 方案評估

## 摘要

本報告針對「畫布式使用者體驗（canvas UX）」需求進行研究：使用者需要能以滑鼠滾輪／觸控縮放（zoom in/out）並以拖曳移動視埠（viewport），如同 Web GIS 地圖操作，但操作對象是示意圖（diagram）或圖論圖（graph），而非地理地圖。報告先釐清既有純 CSS transform 縮放／拖曳方案在「縮放比例（ratio）」上的缺陷，以及純 HTML/CSS 方案缺乏細節層級（LOD, level of detail）而導致的效能問題；再分別評估兩種候選架構：(A) 以 Three.js 的 `OrthographicCamera` 呈現 2D 視圖並搭配內建控制器、(B) 以 `CSS2DRenderer`／`CSS3DRenderer` 渲染主體內容。最後提出建議架構與取捨。

---

## 1. 需求與問題定義

### 1.1 需求

需求可拆解為三項能力：

1. **縮放（zoom）**：以游標為錨點（zoom-to-cursor / anchor）放大縮小，捲動一格所對應的縮放倍率必須符合使用者直覺，可跨瀏覽器與裝置一致。
2. **平移（pan）**：拖曳以移動視埠，且平移速度須與當下縮放層級一致（放大時拖曳距離變小、縮小時變大），不可出現「越放大拖越快」或「越縮小拖越慢」的錯覺。
3. **細節層級（LOD）**：當内容（節點、邊、標籤）數量龐大時，低縮放層級顯示簡化表示，高縮放層級才顯示完整細節，以維持幀率。

### 1.2 Web GIS 的類比

Web GIS 之所以能在大範圍地圖上流暢縮放，關鍵是「**瓦片金字塔（tile pyramid）＋向量瓦片（vector tiles）＋連續縮放**」三者的組合：

- **瓦片金字塔**：縮放層級 0 時整個世界在一片瓦片內，每一層把每片瓦片再切成 4 片（四叉樹）。縮放層級 z 的地圖像素寬度為 `256 × 2^z`（或 512 像素瓦片則為 `512 × 2^z`）[^tilepyramid]。
- **向量瓦片**：與傳統點陣瓦片不同，向量瓦片儲存的是原始幾何與屬性，由瀏覽器端以 GPU 重新繪製，因此「縮放不失真」且可動態改樣式；同時幾何資料比影像更好壓縮[^vector].
- **連續（小數）縮放**：MapLibre GL JS 以 WebGL 每幀重算投影矩陣，原生支援小數縮放（如 z = 4.73）；Leaflet 則有 `zoomSnap`（預設 1）控制是否吸附到整數層級，設為 0 即為完全連續縮放[^maplibre][^leaflet]。
- **過度縮放（overzoom）**：當目前縮放超過最高可用瓦片層級時，沿用較低層級瓦片再放大顯示。MapLibre 以 `OverscaledTileID` 同時記錄「canonical zoom（瓦片資料實際層級）」與「overscaled zoom（顯示層級）」[^overscale]；Leaflet 則以 `maxNativeZoom` 選項讓瓦片被自動放大[^leaflet]。

**錨點數學**（zoom-to-cursor）在 GIS 中的標準做法為：取得游標螢幕像素 → 反投影（unproject）取得世界座標 → 計算新縮放 → 將同一世界座標投影回螢幕 → 計算位移量並反向平移，使游標下的點保持不動。MapLibre 將其封裝為 `calculateAnchoredCameraOptions()`[^anchor]。

### 1.3 既有純 CSS transform 方案的缺陷

市面上最常見的縮放／拖曳方案（例如 `anvaka/panzoom`[^anvaka]、`timmywil/panzoom`[^timmywil]、`BetterTyped/react-zoom-pan-pinch`[^rzpp]、`d3/d3-zoom`[^d3zoom]）本質上都是對一個容器套用 CSS `transform: translate(...) scale(k)`。其固有的限制如下：

**(a) 縮放比例（ratio）問題**

- **滾輪步進在不同作業系統不一致**：`timmywil/panzoom` 的 issue #560 指出，同一 `step` 屬性在 Windows 的 Chrome／Firefox 產生接近 120 的 `deltaY`，而 Edge、Opera 與 macOS 瀏覽器則接近 3，導致縮放步幅在不同瀏覽器差異極大，至今未解[^pz560]。
- **錨點漂移**：`timmywil/panzoom` issue #538 回報反覆以固定焦點呼叫 `zoomToPoint()` 後視圖會隨時間漂移；即使在 Firefox 加入延遲仍會發生[^pz538]。issue #331 指出在 `animate: true` 時快速捲動會因過場尚未結束就計算下一次縮放而「暴衝」[^pz331]。issue #611 則指出當容器以 flex 置中（如彈窗圖片）時焦點縮放計算錯誤，至今仍為 open[^pz611]。
- **巢狀縮放的倍率失真**：issue #647 指出若 panzoom 被放在一個本身帶有 `transform: scale()` 的父層內，捏合中點、捏合距離、contain 邊界與平移速度全部失準，維護者以「多數使用者不需要」為由拒絕修復[^pz647]。
- **`react-zoom-pan-pinch`** 亦有多個同類問題：issue #19 縮放時內容出現 x 偏移[^rzpp19]、#49 當內容大於 wrapper 時縮放中心錯誤[^rzpp49]、#264 `zoomToElement` 位置錯誤[^rzpp264]、#286 `resetTransform` 未回到初始置中位置（v4 才修）、#392 `centerOnInit` 對超大圖失效（v4 才修）[^rzpp286][^rzpp392]。
- **`d3-zoom`** 系列的經典陷阱：`zoomIdentity.translate().scale()` 與 `scale().translate()` 因矩陣不可交換而結果不同[^d3zoom95]；`scaleBy` 搭配 SVG `viewBox` 時中心錯誤[^d3zoom85]；高 DPI（retina）canvas 上縮放中心偏移[^d3zoom133]；視窗 resiz後內部 transform 狀態未更新而偏移[^d3zoom136]；以及缺失可自訂參考點的 `zoom.center()`（issue #211 仍 open）[^d3zoom211]。

**(b) 縮放必然造成的視覺劣化（ratio 問題的根源）**

CSS `transform: scale()` 是「**先依原始尺寸點陣化（rasterize），再於合成階段拉伸點陣**」的後置運算。像素管線為 JavaScript → Style → Layout → Paint → Composite，而 `transform` 只影響 Composite，因此：

- 文字在原始像素網格上做過的 hinting 與反鋸齒，在放大後不再正確；放大時出現插值模糊，縮小時筆畫細節被平均掉[^mdntransform][^spin].
- 邊框／筆畫會出現 **次像素（sub-pixel）** 取樣問題：1px 邊框在不同位置可能有時呈 1px、有時呈 2px，且邊緣半透明而造成對比下降[^spin]。
- 瀏覽器**不會**在 CSS transform 之下重新向量化：向量內容（SVG）只在原始尺寸點陣化一次，之後整張材質被 GPU 拉伸[^chromium]。`vector-effect="non-scaling-stroke"` 雖可讓筆畫寬度不隨 transform 改變，但規範僅保證在 SVG 自身 `transform` 屬性下可靠；若縮放來自套用在 `<svg>` 根元素上的 CSS `transform`，因整張 SVG 已被點陣化為材質再被合成器縮放，行為可能失效[^nss]. 且該屬性不影響文字本身的大小（僅影響文字的**描邊**）[^nss].
- `devicePixelRatio` 交互作用：高 DPI 螢幕上，若背板材質解析度與最終顯示解析度不匹配，會白白浪費高 DPI 的清晰度優勢[^dpr].
- **大型 DOM 樹的效能**：對一個龐大子樹套用 transform 會使其被提升為單一合成層，其背板材質以 `devicePixelRatio` 倍率配置（記憶體 ×4 或 ×9）；一旦內容變動，整個圖層需重新點陣化，可能造成掉幀；多個子元素各自形成圖層時更會「圖層爆炸（layer explosion）」，而圖層管理部分操作具有與圖層樹大小成正比的 CPU 成本[^webdev][^chromium]。

**(c) 缺乏 LOD**

純 HTML/CSS 方案沒有「依縮放層級切換幾何細節」的原生概念，只能靠人工判斷可見性（culling）勉強維持效能，沒有幾何簡化機制。相較之下：

- **Excalidraw**（約 134k stars[^excalidraw]）以雙層 Canvas 2D（static canvas 繪形 + interactive canvas 繪選取框）繪製，僅以 `isElementInViewport()` 做視埠剔除，**沒有**超過剔除以外的 LOD[^excalidraw].
- **tldraw**（約 51k stars[^tldraw]）以 React 產生 HTML/SVG 作為主體形狀（不是 canvas），輔以 Canvas 2D 繪製選取指示器；它實作了業界較完整的 LOD：空間索引＋`display: none` 剔除、`getEfficientZoomLevel()` 在超過 500 個形狀時對鏡頭移動中的縮放做「去抖（debounced zoom）」、圖片依 `steppedScreenScale`（貼近 2 的冪）請求低解析版本、低縮放下以純色取代陰影／虛線／網點填充、以及 `textShadowLod`（預設 0.35）等[^tldrawperf][^tldrawcull].

---

## 2. 方案 A：Three.js `OrthographicCamera` ＋ 內建控制器

### 2.1 正交相機的焦距與縮放機制

`OrthographicCamera(left, right, top, bottom, near, far)` 以 6 個平面定義一個長方體視錐；與透視相機不同，物件在成像中的大小不隨距離改變，故極適合 2D 場景[^orthodoc]. 其縮放實作於 `updateProjectionMatrix()`：

```
const dx = (this.right - this.left) / (2 * this.zoom);
const dy = (this.top - this.bottom) / (2 * this.zoom);
```

`zoom` 愈大，可見世界區域愈小，物件看起來愈大。**縮放是純粹的投影矩陣縮放，相機世界座標本身不變**[^orthosrc]. 這與 CSS `transform: scale()` 有本質差異：WebGL 是**重新光柵化向量幾何**，而非拉伸既有點陣，因此不繼承 1.3(b) 的模糊問題。

2D 設定範例（世界座標 (0,0) 置中，`zoom=1` 時視錐恰等於像素尺寸）：

```javascript
const camera = new THREE.OrthographicCamera(-w/2, w/2, h/2, -h/2, 0, 1000);
camera.position.set(0, 0, 100);
```

注意：正交相機的 `near` 可為 0（透視相機不可）[^orthosrc].

### 2.2 內建控制器比較

Three.js 於 `examples/jsm/controls/` 提供多種控制器[^orbit][^map][^trackball]：

| 特性 | `OrbitControls` | `MapControls` | `TrackballControls` |
|---|---|---|---|
| 基礎 | `Controls` | 繼承 `OrbitControls` | `Controls` |
| 維持 `up` 方向 | 是 | 是 | 否（越過極點會翻轉） |
| `screenSpacePanning` 預設 | `true` | `false` | 不適用 |
| 正交縮放限制 | `minZoom`/`maxZoom` | 同左（繼承） | `minZoom`/`maxZoom` |
| `zoomToCursor` | 支援 | 支援 | 不支援 |
| 平移速度隨縮放縮放 | 是 | 是（同 OrbitControls） | 是 |
| 滑鼠左鍵預設 | 旋轉 | **平移** | 旋轉 |
| 適合 2D 圖表 | 中等（需設定） | **最佳** | 差（無 up 約束） |

**`MapControls` 是 2D 圖表最合適的選擇**，因為其互動模型即為「左鍵拖曳平移、滾輪縮放」，與 Google Maps 一致；`screenSpacePanning = false` 會以世界平面射線求交來鎖定平移於世界 XZ 平面[^map]. 其正交縮放為**乘法／指數式**：

```
_getZoomScale(delta) { return Math.pow(0.95, this.zoomSpeed * Math.abs(delta*0.01)); }
```

意即每一格滾動將 `camera.zoom` 乘上固定倍率，高縮放時絕對世界變化較小（細膩），低縮放時較大（粗放），避免了線性（加法）縮放在放大時過快、縮小時過慢的違和感[^orbit].

平移與縮放皆正確地除以 `zoom`。此為 2016 年修復 issue #8340 的成果：修正前正交相機的平移距離會受縮放層級影響（放大後拖太快、縮小後拖太慢），修正後改為 `deltaX * (right-left) / zoom / clientWidth`[^issue8340].

`zoomToCursor` 的正交實作為：以 `unproject()` 取得縮放前的世界座標與縮放後的世界座標，再以位移差修正相機位置，使游標下的點保持不動[^orbit]：

```
const mouseBefore = new Vector3(this._mouse.x, this._mouse.y, 0);
mouseBefore.unproject(this.object);
this.object.zoom = ...;
this.object.updateProjectionMatrix();
const mouseAfter = new Vector3(this._mouse.x, this._mouse.y, 0);
mouseAfter.unproject(this.object);
this.object.position.sub(mouseAfter).add(mouseBefore);
```

### 2.3 LOD 與效能

- **`LOD` 類別**：`addLevel(object, distance, hysteresis)` 依距離切換層級，`update(camera)` 內部計算 `distance = camera 與物件距離 / camera.zoom`，**已除以 `camera.zoom`**，故正交相機縮放亦能正確觸發 LOD 切換[^lodsrc]. `hysteresis` 為避免邊界抖動的遲滯帶[^lodsrc].
- **`InstancedMesh`**：以單一 draw call 繪製大量同幾何物件，但 Three.js 對其以整體包圍球做視錐剔除，**不會**對個別實例剔除；需手動調整 `mesh.count` 或改用第三方 `InstancedMesh2`[^instanced].
- **視錐剔除**：`Object3D.frustumCulled` 預設為 `true`，渲染器以包圍球與視錐求交決定是否送 draw call[^o3d].
- **官方效能建議**：合併幾何（`mergeGeometries()`）以減少 draw call、動畫用 morph targets、靜態物件設 `matrixAutoUpdate = false`、盡量避免透明物件、重用 `Vector3`/`Matrix4`、正確 `dispose()` GPU 資源[^optlots][^discover].

### 2.4 線條與文字在「任意縮放」下的清晰度

純 CSS 方案難以維持筆畫寬度；Three.js 有多種對策：

- **`Line2` / `LineSegments2` + `LineMaterial`**：以 instanced 三角帶（fat line）繪製寬線，頂點著色器以 `offset /= resolution.y` 將像素寬度換算回 NDC，因此**預設（`worldUnits: false`）線寬維持固定的螢幕像素寬，不隨相機縮放改變**；設 `worldUnits: true` 才會隨縮放改變。相對地 `THREE.Line` + `LineBasicMaterial` 的 `linewidth` 在 WebGL/WebGPU **被忽略**，恆為 1 像素[^linematerial][^linebasic]. `LineSegments2.onBeforeRender` 每幀自動更新 `resolution` uniform[^lineseg2].
- **文字清晰度**：SDF（signed distance field）計算法在片段著色器以 `smoothstep(0.5 - fwidth(sd), 0.5 + fwidth(sd), sd)` 於當下解析度重建邊緣，故縮放不模糊。`troika-three-text`（所屬 monorepo 約 2.0k stars[^troika]）可直接解析 ttf/otf/woff 並在 Web Worker 產生 SDF atlas，支援 kerning、連字、RTL、自動 Unicode 回退；限制為 SDF atlas 尺寸上限（`sdfGlyphSize` 需為 2 的冪）、非同步產生需監聽 `synccomplete`、不支援 woff2、worker 以字串 eval 建立故可能被 CSP 阻擋（可設 `useWorker: false`）[^troika]. MSDF（多通道 SDF，`msdfgen` 約 5k stars）尖角更銳利，但需離線預先產生 atlas[^msdf]. `TextGeometry` 為真實擠出幾何、可任意縮放，但 tessellation 固定、長字串昂貴、須先轉換字型 JSON[^textgeo]. Canvas 材質文字則因固定點陣解析度，**在放大時必然模糊**，即使以 `devicePixelRatio` 放大亦僅延後問題[^canvastext].

---

## 3. 方案 B：`CSS2DRenderer` / `CSS3DRenderer` 渲染主體

### 3.1 CSS2DRenderer

`CSS2DRenderer` 是 `CSS3DRenderer` 的簡化版，**僅支援平移與 2D 旋轉**；它把 `CSS2DObject` 轉為 DOM 元素，將物件 3D 世界座標經 `viewProjectionMatrix` 投影為螢幕座標，並設定：

```
element.style.transform = `translate(${-cx}%, ${-cy}%) translate(${tx}px, ${ty}px) rotate(${angle}rad)`;
```

其中**沒有** `scale()`[^css2dsrc]. 因此：

- **CSS2D 標籤不隨相機縮放改變像素大小**——這是設計使然，讓標籤在任意距離都保持可讀；社群曾提出需求（issue #9246）欲讓其縮放，被 mrdoob 以情境過於小眾關閉，最常見的 workaround 是自行在 transform 後追加 `scale(camera.zoom, camera.zoom)`[^css2dscale]. 若希望標籤隨縮放縮放，須在動畫迴圈動態調整 `font-size`（`baseFontSize * camera.zoom`）[^css2dscale].
- 官方文件亦註明其**僅支援 100% 瀏覽器與顯示縮放**（否則座標對應會失準）[^css2dlim].
- 效能上為純 DOM 定位、無 canvas/WebGL，但每個 `CSS2DObject` 都是一個真實 DOM 節點，且每次 `render()` 皆重算 transform、並可能對整個場景做 z-index 排序，故適合**數十至數百**個標籤，不適合數千個[^css2dlim].
- **適用情境**：3D 物件上的標籤、提示、註解、量測標籤等需要恆定可讀大小的疊加 UI。

### 3.2 CSS3DRenderer

`CSS3DRenderer` 以 CSS `transform` 的 `matrix3d()` 對 DOM 套用完整階層式 3D 變換，並以 CSS `perspective()` 模擬透視；其 DOM 結構為 `domElement → viewElement → cameraElement → CSS3DObject`，且有樣式快取（僅在字串變動時寫入 DOM）[^css3dsrc]. 重要事實：

- **對正交相機的支援是後來才加入的**。2017 年 issue #11534 中 mrdoob 曾言「我不認為 CSS3D 支援正交投影」；2018 年（R92）由 yomotsu 補上支援後關閉[^css3dortho]. 官方現有範例 `css3d_orthographic.html` 即以 `OrthographicCamera` 同時搭配 WebGL 與 CSS3D 渲染同一場景[^css3dexample].
- **兩種相機的 CSS 變換策略不同**：透視相機用 `translateZ()` ＋ `perspective()`；正交相機改用 `scale()` ＋ `translate()`，不使用 `perspective()`[^css3dsrc].
- **正交路徑反而較銳利**：issue #17641 中使用者發現透視相機路徑因 `perspective()` 觸發 GPU 光柵化而使 CSS 內容模糊；yomotsu 解釋「一旦套用 CSS-Transform3D，元素就會依初始尺寸光柵化，因 GPU 加速而不再是向量圖」[^css3dpixel]. 正交相機因改用 `scale()` 可較清晰。
- **限制**：官方文件明列「無法使用 Three.js 材質系統」「無法使用幾何」，舊版文件另列「僅支援 100% 瀏覽器與顯示縮放」[^css3dlim]. 另有 `setViewOffset` 曾不受支援、後於 2023 年補上 `viewElement` 處理的問題[^css3dviewoffset]. DOM 開銷與 CSS2D 類似，適合數十至數百個「3D 卡片」型內容。

### 3.3 以 CSS Renderer 作為「主體」的問題

若以 CSS2D／CSS3D 渲染圖表主體，會遇到：

1. **DOM 節點數 = 內容數**，數千節點即造成 layout/reflow/paint 瓶頸[^css2dlim]。
2. **`CSS3D` 沒有材質與幾何系統**，複雜圖形須以 HTML/CSS 拼湊[^css3dlim].
3. **僅支援 100% 瀏覽器縮放**，使用者以瀏覽器縮放即錯位[^css2dlim].
4. **`CSS2D` 根本不縮放**，若硬要縮放需自行 hack；`CSS3D` 雖縮放，但 `matrix3d` 之光柵化仍可能使內容模糊[^css3dpixel].
5. 仍不具備自動 LOD 與空間剔除，僅能做 DOM 層的人工剔除。

換言之，CSS Renderer 適合當「**標籤／疊加層**」，不適合作為「**大量幾何主體**」。

---

## 4. 方案比較

| 面向 | 純 CSS transform 方案 | Three.js `OrthographicCamera` | `CSS2D`／`CSS3D` 為主體 |
|---|---|---|---|
| 縮放比例正確性 | 差（步進跨瀏覽器不一致、錨點漂移）[^pz560][^pz538] | 佳（乘法式縮放、`zoomToCursor` 以 unproject 修正）[^orbit] | CSS2D 不縮放[^css2dlim]；CSS3D 可但受限 |
| 縮放後清晰度 | 點陣拉伸必然模糊[^mdntransform][^chromium] | 重新光柵化向量，清晰；SDF 文字不模糊[^troika] | CSS3D 正交路徑較清晰但仍光柵化[^css3dpixel] |
| 筆畫寬度控制 | 次像素失真，需 `non-scaling-stroke` 且不可靠[^nss] | `LineMaterial` 螢幕像素寬恆定[^linematerial] | 依 CSS 樣式，縮放下不穩 |
| LOD | 無原生機制[^excalidraw] | `LOD` 類別、`InstancedMesh`、視錐剔除[^lodsrc][^instanced] | 僅人工 DOM 剔除[^css2dlim] |
| 大量內容效能 | 大型 DOM 樹／圖層爆炸[^webdev][^chromium] | GPU draw call 為瓶頸，可合併幾何與 instance 化[^optlots] | 每物件一 DOM 節點，數千即瓶頸[^css2dlim] |
| 可維護性 | 高（少量程式碼） | 中（需理解相機、材質、shader） | 低（hack 比例高） |
| 適合當標籤／UI | 一般 | 佳（WebGL 主體 + CSS2D 疊加） | **最佳** |
| 適合當幾何主體 | 小量尚可 | **最佳** | 不佳 |

參考生態系規模（stars，擷取時點）：Three.js 約 116.5k[^threejsrepo]、React Three Fiber 約 33.0k[^r3f]、drei 約 10.0k[^drei]、sigma.js 約 12.2k[^sigma]、Cytoscape.js 約 11.2k[^cytoscape]、Konva 約 14.9k[^konva]、pixi.js 約 48.4k[^pixi]、React Flow 約 38.7k[^xyflow]、MapLibre GL JS 約 11.9k、Leaflet 約 45.8k、OpenLayers 約 12.6k[^gisrepos].

---

## 5. 建議架構

綜合以上，建議採用**混合式（hybrid）架構**，而非單一方案：

1. **幾何主體交由 WebGL＋`OrthographicCamera` 渲染**：節點、邊以 `InstancedMesh`／合併幾何與 `Line2` 系列繪製，獲得正確的縮放比例與清晰度。
2. **控制器採 `MapControls`**：左鍵拖曳平移、滾輪縮放、啟用 `zoomToCursor`；設定 `minZoom`/`maxZoom` 並於頂點場景設定 `screenSpacePanning = false`[^map]. 若日後需要自由旋轉再切回 `OrbitControls`.
3. **LOD 三層策略**（對應 Web GIS 的瓦片金字塔概念）：
   - **幾何 LOD**：以 `LOD` 類別（`addLevel` + `hysteresis`）依 `distance / camera.zoom` 切換簡化幾何[^lodsrc]，或用螢幕空間誤差（screen-space error，SSE）門檻。SSE 的權威定義來自 Cesium 3D Tiles 規範：`SSE = (geometricError × viewportHeight) / (2 × distance × tan(fovY/2))`，當 SSE 超過門檻即細化到子節點[^sse].
   - **內容 LOD**：低縮放下隱藏或簡化次要元素（標籤合併、隱藏邊權重、以純色取代陰影與虛線），可參考 tldraw 的實務門檻（`debouncedZoomThreshold` 500、`textShadowLod` 0.35 等）[^tldrawperf].
   - **剔除（culling）**：保留 Three.js 預設視錐剔除[^o3d]；對 `InstancedMesh` 手動依可見實例調整 `mesh.count`[^instanced].
4. **標籤／UI 交由 `CSS2DRenderer`**：將需要恆定可讀大小的標籤以 `CSS2DObject` 疊加於 WebGL canvas 之上（兩者可共用同一場景與相機，各自忽略對方型別的物件）[^css2dlim]. 若需求為「世界座標等寬的標籤」，則改以 SDF 文字（`troika-three-text`）繪於 WebGL 內[^troika]，避免 DOM 開銷。
5. **`CSS3DRenderer` 僅在需要 3D 卡片式內容時才使用**，且優先搭配正交相機路徑以避免透視路徑的模糊問題[^css3dpixel].

**決策準則**：

- 內容以**簡單幾何（節點球、邊線）+ 大量**為主 → Three.js WebGL 主體 ＋ CSS2D 標籤。
- 內容以**豐富 HTML（表格、影片、可選文字）** 為主且數量在數百以內 → `CSS3DRenderer`（正交相機）。
- 內容僅**數十個**且需與 DOM 生態密切整合、不介意模糊 → 純 CSS transform 方案（成本最低，但不可期待 LOD 與清晰度）。

---

## 6. 結論

1. 既有純 CSS transform 縮放／拖曳方案之所以「縮放比例處理不好」，根因在於 `transform: scale()` 是後置的點陣拉伸（必然模糊、次像素失真），加上滾輪 `deltaY` 跨瀏覽器不一致與錨點計算在 flex 置中、巢狀縮放、retina 等情境下的實作缺陷；這不是個別函式庫的疏漏，而是架構性的限制[^spin][^chromium][^pz560][^pz611][^pz647].
2. 純 HTML/CSS 沒有 LOD 概念，難以支撐大型圖表；即使是 Excalidraw 也只做到視埠剔除，tldraw 才建立了較完整的 LOD 機制[^excalidraw][^tldrawperf].
3. **Three.js `OrthographicCamera` ＋ `MapControls`** 是需求的正解方向：正交投影適合 2D、縮放為投影矩陣縮放而非點陣拉伸、內建控制器已修正平移速度隨縮放的問題並提供 `zoomToCursor`[^orthosrc][^map][^issue8340][^orbit]. 搭配 `LOD`、`InstancedMesh`、`Line2`、SDF 文字可同時滿足清晰度、比例正確性與效能[^lodsrc][^instanced][^linematerial][^troika].
4. **`CSS2DRenderer` 應定位為「標籤／疊加層」而非主體**：它不隨縮放改變像素大小（設計使然）且受限於 100% 瀏覽器縮放與 DOM 節點數[^css2dlim]. **`CSS3DRenderer` 可與正交相機搭配**（R92 後支援），且在正交路徑下比透視路徑更清晰，但缺乏材質／幾何系統、效能受 DOM 節點數限制，適合作為 3D 卡片式內容而非大量幾何主體[^css3dortho][^css3dpixel][^css3dlim].
5. 最佳實務為**混合架構**：WebGL 繪幾何主體與 LOD，CSS2D 疊標籤，必要時才以 CSS3D 承載豐富 DOM 內容；並以 Web GIS 的「錨點縮放 + 分層 LOD + 剔除」三原則指導實作[^anchor][^sse][^map].

---

## 註腳與參考文獻

每條註腳以改良式 APA 格式（ISO 8601 日期）直接附上來源連結，並補充該來源所支持的事實。

[^tilepyramid]: OpenStreetMap. (n.d.). Zoom levels. Retrieved 2026-10-17, from https://wiki.openstreetmap.org/wiki/Zoom_levels —— 瓦片金字塔：每一層將每片瓦片切成 4 片之父四叉樹結構；縮放層級 z 的地圖像素寬度為 `256 × 2^z`.
[^vector]: MapLibre. (n.d.). MapLibre GL JS. GitHub. Retrieved 2026-10-17, from https://github.com/maplibre/maplibre-gl-js —— 向量瓦片儲存原始幾何與屬性的二進位資料，由 GPU 重新繪製，故縮放不失真、可動態改樣式、壓縮率優於影像.
[^maplibre]: MapLibre. (n.d.). MapLibre GL JS documentation. Retrieved 2026-10-17, from https://maplibre.org/maplibre-gl-js/docs/ —— 以 WebGL 每幀重算投影矩陣，原生支援小數縮放.
[^leaflet]: Leaflet. (n.d.). Leaflet API reference. Retrieved 2026-10-17, from https://leafletjs.com/reference.html —— `zoomSnap`（預設 1）控制縮放吸附；`maxNativeZoom` 控制過度縮放；向量圖層預設以 SVG 或可選 Canvas 2D 繪製.
[^overscale]: MapLibre. (n.d.). OverscaledTileID. Retrieved 2026-10-17, from https://maplibre.org/maplibre-gl-js/docs/API/interfaces/OverscaledTileID/ —— 同時記錄 canonical zoom 與 overscaled zoom 的瓦片識別碼，支援過度縮放.
[^anchor]: MapLibre. (n.d.). Map. Retrieved 2026-10-17, from https://maplibre.org/maplibre-gl-js/docs/API/classes/Map/ —— `calculateAnchoredCameraOptions()` 將地理錨點置於指定螢幕點的相機計算，即 zoom-to-cursor 的核心數學.
[^anvaka]: anvaka. (n.d.). panzoom. GitHub. Retrieved 2026-10-17, from https://github.com/anvaka/panzoom —— 通用 DOM/SVG 平移縮放函式庫（iOS 縮放模糊回報另見 https://github.com/anvaka/panzoom/issues/128 ）。
[^timmywil]: timmywil. (n.d.). panzoom. GitHub. Retrieved 2026-10-17, from https://github.com/timmywil/panzoom —— 以 CSS transform 平移縮放元素的函式庫.
[^rzpp]: BetterTyped. (n.d.). react-zoom-pan-pinch. GitHub. Retrieved 2026-10-17, from https://github.com/BetterTyped/react-zoom-pan-pinch —— React 的縮放／平移／捏合函式庫.
[^d3zoom]: d3. (n.d.). d3-zoom. GitHub. Retrieved 2026-10-17, from https://github.com/d3/d3-zoom —— 提供 `{k, x, y}` 之 `ZoomTransform`，由使用者自行套用為 CSS transform、SVG 或 Canvas 變換.
[^pz560]: timmywil. (2021). Browser normalization for scroll zoom [Issue #560]. Retrieved 2026-10-17, from https://github.com/timmywil/panzoom/issues/560 —— 滾輪 `deltaY` 在不同瀏覽器／OS 未正規化，導致縮放步幅不一致（open, bug）.
[^pz538]: timmywil. (2021). Incorrect zoomToPoint focal behavior (in Firefox)? [Issue #538]. Retrieved 2026-10-17, from https://github.com/timmywil/panzoom/issues/538 —— `zoomToPoint` 焦點行為在 Firefox 下會漂移.
[^pz331]: timmywil. (2019). Wildly veering on mousewheel zoom with animate: true [Issue #331]. Retrieved 2026-10-17, from https://github.com/timmywil/panzoom/issues/331 —— `animate: true` 時滾輪縮放會暴衝偏移.
[^pz611]: timmywil. (2022). Focal zoom with flex-centered images [Issue #611]. Retrieved 2026-10-17, from https://github.com/timmywil/panzoom/issues/611 —— flex 置中容器的焦點縮放錯誤（open）.
[^pz647]: timmywil. (2023). Nested CSS scale causes unexpected pinch midpoint and translation behaviors [Issue #647]. Retrieved 2026-10-17, from https://github.com/timmywil/panzoom/issues/647 —— 巢狀 CSS scale 導致捏合中點、距離、邊界與平移速度全部失準（closed as not planned）.
[^rzpp19]: BetterTyped. (2019). I will get x offset when scaling to left [Issue #19]. Retrieved 2026-10-17, from https://github.com/BetterTyped/react-zoom-pan-pinch/issues/19 —— 縮放時內容出現 x 偏移（已修）.
[^rzpp49]: BetterTyped. (2019). Zooming issues when TransformComponent larger than TransformWrapper [Issue #49]. Retrieved 2026-10-17, from https://github.com/BetterTyped/react-zoom-pan-pinch/issues/49 —— 內容大於 wrapper 時縮放中心錯誤（已修）.
[^rzpp264]: BetterTyped. (2023). zoomToElement broken since 2.1.2 [Issue #264]. Retrieved 2026-10-17, from https://github.com/BetterTyped/react-zoom-pan-pinch/issues/264 —— `zoomToElement` 忽略位移而縮放位置錯誤.
[^rzpp286]: BetterTyped. (2026). resetTransform doesn't reset back to initial position [Issue #286]. Retrieved 2026-10-17, from https://github.com/BetterTyped/react-zoom-pan-pinch/issues/286 —— `resetTransform` 未回到初始置中位置（v4 修復）.
[^rzpp392]: BetterTyped. (2026). centerOnInit not working properly [Issue #392]. Retrieved 2026-10-17, from https://github.com/BetterTyped/react-zoom-pan-pinch/issues/392 —— `centerOnInit` 對超大圖失效（v4 修復）.
[^d3zoom95]: d3. (2017). zoomIdentity.translate().scale() vs scale().translate() [Issue #95]. Retrieved 2026-10-17, from https://github.com/d3/d3-zoom/issues/95 —— 矩陣不可交換，故呼叫順序會改變結果.
[^d3zoom85]: d3. (2017). Wrong center when zoom with scaleBy and using svg with viewBox attribute [Issue #85]. Retrieved 2026-10-17, from https://github.com/d3/d3-zoom/issues/85 —— `scaleBy` 搭配 SVG `viewBox` 時縮放中心錯誤.
[^d3zoom133]: d3. (2019). Zoom behaviour with retina canvas [Issue #133]. Retrieved 2026-10-17, from https://github.com/d3/d3-zoom/issues/133 —— retina canvas 上縮放中心偏移.
[^d3zoom136]: d3. (2019). Resizing the window produces an offset when zooming and panning [Issue #136]. Retrieved 2026-10-17, from https://github.com/d3/d3-zoom/issues/136 —— 視窗 resize 後 zoom/pan 相對游標偏移.
[^d3zoom211]: d3. (2020). Allow a reference point to be specified for interactive zooming (zoom.center) [Issue #211]. Retrieved 2026-10-17, from https://github.com/d3/d3-zoom/issues/211 —— 缺乏可自訂參考點的 `zoom.center()`（open, 有未合併的 PR）.
[^mdntransform]: Mozilla. (n.d.). transform. MDN Web Docs. Retrieved 2026-10-17, from https://developer.mozilla.org/en-US/docs/Web/CSS/transform —— transform 為非 `none` 時建立堆疊脈絡，屬合成階段運算.
[^spin]: Modern CSS. (n.d.). Scaling elements without transform hacks. Retrieved 2026-10-17, from https://modern-css.com/scaling-elements-without-transform-hacks/ —— `transform: scale()` 為視覺後置運算（先點陣化再拉伸），故文字 hinting 與反鋸齒不再匹配最終像素網格、邊緣出現次像素失真；`zoom` 則於 layout 階段套用.
[^chromium]: Chromium. (n.d.). GPU accelerated compositing in Chrome. Retrieved 2026-10-17, from https://www.chromium.org/developers/design-documents/gpu-accelerated-compositing-in-chrome/ —— 合成器以材質（點陣圖）為單位套用 transform，並描述圖層爆炸與 layer squashing 的成本.
[^nss]: Mozilla. (n.d.). vector-effect. MDN Web Docs. Retrieved 2026-10-17, from https://developer.mozilla.org/en-US/docs/Web/SVG/Reference/Attribute/vector-effect —— `non-scaling-stroke` 使筆畫視覺寬度不隨 transform 改變；規範僅保證在 SVG 自身 `transform` 下可靠，於根元素 CSS transform 下可能失效；不影響文字大小（僅描邊）.
[^dpr]: BugNet. (n.d.). How to fix web canvas blurry on retina without devicePixelRatio. Retrieved 2026-10-17, from https://bugnet.io/blog/how-to-fix-web-canvas-blurry-on-retina-without-devicepixelratio —— 背板解析度須與顯示解析度匹配方可維持清晰.
[^webdev]: web.dev. (n.d.). Stick to compositor-only properties and manage layer count. Retrieved 2026-10-17, from https://web.dev/articles/stick-to-compositor-only-properties-and-manage-layer-count —— 說明圖層記憶體與頻寬成本，以及大 DOM 樹提升為單一圖層的風險.
[^excalidraw]: excalidraw. (n.d.). Excalidraw. GitHub. Retrieved 2026-10-17, from https://github.com/excalidraw/excalidraw —— 以雙層 Canvas 2D 繪製，僅以 `isElementInViewport()` 剔除，無超過剔除以外的 LOD.
[^tldraw]: tldraw. (n.d.). tldraw. GitHub. Retrieved 2026-10-17, from https://github.com/tldraw/tldraw —— 以 React 產生 HTML/SVG 為主體形狀，指示器改用 Canvas 2D.
[^tldrawperf]: tldraw. (n.d.). Performance. tldraw docs. Retrieved 2026-10-17, from https://tldraw.dev/sdk-features/performance —— 空間索引剔除、`getEfficientZoomLevel()` 去抖縮放、圖片解析度 LOD、低縮放形狀簡化、`textShadowLod` 等.
[^tldrawcull]: tldraw. (n.d.). Culling. tldraw docs. Retrieved 2026-10-17, from https://tldraw.dev/sdk-features/culling —— 以空間索引追蹤可見形狀，剔除者保留於 DOM 但 `display: none`.
[^orthodoc]: three.js. (n.d.). OrthographicCamera. Retrieved 2026-10-17, from https://threejs.org/docs/#api/en/cameras/OrthographicCamera —— 以 6 平面定義視錐，物件大小不隨距離改變.
[^orthosrc]: three.js. (n.d.). OrthographicCamera.js (dev). GitHub. Retrieved 2026-10-17, from https://raw.githubusercontent.com/mrdoob/three.js/dev/src/cameras/OrthographicCamera.js —— `updateProjectionMatrix()` 中 `dx = (right-left)/(2*zoom)`、`dy = (top-bottom)/(2*zoom)`，縮放為投影矩陣縮放、相機世界座標不變；`near` 可為 0.
[^orbit]: three.js. (n.d.). OrbitControls. Retrieved 2026-10-17, from https://threejs.org/docs/#examples/en/controls/OrbitControls —— `zoomToCursor`、`minZoom`/`maxZoom`、乘法式 `_getZoomScale`、以 `unproject` 修正錨點、平移除以 zoom.
[^map]: three.js. (n.d.). MapControls. Retrieved 2026-10-17, from https://threejs.org/docs/#examples/en/controls/MapControls —— 繼承 `OrbitControls`，`screenSpacePanning=false`、左鍵平移、支援 `zoomToCursor`；官方建議俯視地圖情境使用.
[^trackball]: three.js. (n.d.). TrackballControls. Retrieved 2026-10-17, from https://threejs.org/docs/#examples/en/controls/TrackballControls —— 不維持 up 方向；正交平移以 `clientWidth` 計算 x/y 尺度.
[^issue8340]: mrdoob. (2016). OrthographicCamera pan broken when zoomed [Issue #8340]. Retrieved 2026-10-17, from https://github.com/mrdoob/three.js/issues/8340 —— 正交相機平移距離受縮放影響，2016 年修復為除以 `zoom`.
[^lodsrc]: three.js. (n.d.). LOD.js (master). GitHub. Retrieved 2026-10-17, from https://raw.githubusercontent.com/mrdoob/three.js/master/src/objects/LOD.js —— `addLevel(object, distance, hysteresis)`，`update(camera)` 計算 `distance = 相機與物件距離 / camera.zoom`，並以 hysteresis 避免邊界抖動.
[^instanced]: three.js. (n.d.). InstancedMesh. Retrieved 2026-10-17, from https://threejs.org/docs/#api/en/objects/InstancedMesh —— 以單一 draw call 繪製大量同幾何物件；Three.js 對其以整體包圍球做視錐剔除，不個別剔除實例，需手動調整 `count`.
[^o3d]: three.js. (n.d.). Object3D. Retrieved 2026-10-17, from https://threejs.org/docs/#api/en/core/Object3D —— `frustumCulled` 預設 `true`，由渲染器以包圍球與視錐求交決定繪製.
[^optlots]: three.js. (n.d.). Optimize lots of objects. Retrieved 2026-10-17, from https://threejs.org/manual/#en/optimize-lots-of-objects —— 以 `mergeGeometries()` 合併幾何以減少 draw call.
[^discover]: discoverthreejs. (n.d.). Tips and tricks. Retrieved 2026-10-17, from https://discoverthreejs.com/tips-and-tricks/ —— `matrixAutoUpdate=false`、避免透明物件、重用向量矩陣、正確 `dispose()`、以 `renderer.info` 監控 draw call.
[^linematerial]: three.js. (n.d.). LineMaterial. Retrieved 2026-10-17, from https://threejs.org/docs/#examples/en/lines/LineMaterial —— fat-line 頂點著色器以 `offset /= resolution.y` 將像素寬換算回 NDC，故 `worldUnits: false` 時線寬固定螢幕像素.
[^linebasic]: three.js. (n.d.). LineBasicMaterial. Retrieved 2026-10-17, from https://threejs.org/docs/#api/en/materials/LineBasicMaterial —— `linewidth` 僅 SVGRenderer 有效，WebGL/WebGPU 忽略並恆以 1 像素繪製.
[^lineseg2]: three.js. (n.d.). LineSegments2.js (dev). GitHub. Retrieved 2026-10-17, from https://raw.githubusercontent.com/mrdoob/three.js/dev/examples/jsm/lines/LineSegments2.js —— `onBeforeRender` 每幀以 `renderer.getViewport()` 更新 `resolution` uniform.
[^troika]: protectwise. (n.d.). troika-three-text. GitHub. Retrieved 2026-10-17, from https://github.com/protectwise/troika/tree/main/packages/troika-three-text —— 解析 ttf/otf/woff 並於 Web Worker 產生 SDF atlas，以標準導數於片段著色器重建邊緣，支援 kerning／連字／RTL；限制含 `sdfGlyphSize` 上限、非同步 `synccomplete`、不支援 woff2、CSP 下需 `useWorker: false`.
[^msdf]: Chlumsky, V. (n.d.). msdfgen. GitHub. Retrieved 2026-10-17, from https://github.com/Chlumsky/msdfgen —— MSDF（多通道 SDF）以 RGB 三通道距離場取中位數重建，尖角更銳利，需離線預先產生 atlas；取樣須於線性空間進行.
[^textgeo]: three.js. (n.d.). TextGeometry. Retrieved 2026-10-17, from https://threejs.org/docs/#examples/en/geometries/TextGeometry —— 以 typeface.json 產生擠出幾何；tessellation 固定、更新昂貴、須預先轉換字型.
[^canvastext]: threejsresources. (n.d.). Text. Retrieved 2026-10-17, from https://threejsresources.com/guides/text —— Canvas 材質文字因固定點陣解析度，放大必然模糊；即使以 `devicePixelRatio` 放大亦僅延後問題.
[^css2dsrc]: three.js. (n.d.). CSS2DRenderer.js (dev). GitHub. Retrieved 2026-10-17, from https://raw.githubusercontent.com/mrdoob/three.js/dev/examples/jsm/renderers/CSS2DRenderer.js —— transform 僅含 `translate(...)` 與 `rotate(...)`，無 `scale()`；每幀重算並可做 z-index 排序.
[^css2dlim]: three.js. (n.d.). CSS2DRenderer. Retrieved 2026-10-17, from https://threejs.org/docs/#examples/en/renderers/CSS2DRenderer —— 僅支援平移與 2D 旋轉、僅支援 100% 瀏覽器與顯示縮放；每個物件為一 DOM 節點，適合數十至數百個標籤.
[^css2dscale]: mrdoob. (2016). add scaling to CSS2DRenderer [Issue #9246]. Retrieved 2026-10-17, from https://github.com/mrdoob/three.js/issues/9246 —— 要求 `CSS2DRenderer` 支援縮放的討論；社群 workaround 為在 transform 追加 `scale(camera.zoom, camera.zoom)` 或動態調整 `font-size`.
[^css3dsrc]: three.js. (n.d.). CSS3DRenderer.js (master). GitHub. Retrieved 2026-10-17, from https://raw.githubusercontent.com/mrdoob/three.js/master/examples/jsm/renderers/CSS3DRenderer.js —— 以 `matrix3d()` 及 CSS `perspective()` 進行階層式 3D 變換；正交相機改用 `scale()`＋`translate()`；具樣式快取.
[^css3dortho]: mrdoob. (2017). CSS3DRenderer not compatible with OrthographicCamera [Issue #11534]. Retrieved 2026-10-17, from https://github.com/mrdoob/three.js/issues/11534 —— CSS3D 對正交相機的支援於 R92（2018）由 yomotsu 加入並關閉；此前 mrdoob 認為不支援正交投影.
[^css3dexample]: three.js. (n.d.). css3d_orthographic.html (dev). GitHub. Retrieved 2026-10-17, from https://github.com/mrdoob/three.js/blob/dev/examples/css3d_orthographic.html —— 以 `OrthographicCamera` 同時搭配 WebGL 與 CSS3D 渲染同一場景.
[^css3dpixel]: mrdoob. (2019). CSS3D pixelation with PerspectiveCamera [Issue #17641]. Retrieved 2026-10-17, from https://github.com/mrdoob/three.js/issues/17641 —— 透視路徑因 CSS `perspective()` 觸發 GPU 光柵化而模糊，正交路徑（`scale()`）較清晰；yomotsu 解釋 CSS-Transform3D 一旦套用即依初始尺寸光柵化，不再是向量圖.
[^css3dlim]: three.js. (n.d.). CSS3DRenderer. Retrieved 2026-10-17, from https://threejs.org/docs/#examples/en/renderers/CSS3DRenderer —— 無法使用 Three.js 材質系統與幾何；舊版文件另列僅支援 100% 瀏覽器與顯示縮放.
[^css3dviewoffset]: mrdoob. (2021). CSS3DRenderer does not respect setViewOffset [Issue #21723]. Retrieved 2026-10-17, from https://github.com/mrdoob/three.js/issues/21723 —— CSS3DRenderer 曾不支援 `setViewOffset`，2023 年以新增 `viewElement` 處理.
[^threejsrepo]: mrdoob. (n.d.). three.js. GitHub. Retrieved 2026-10-17, from https://github.com/mrdoob/three.js —— 約 116.5k stars；最新版本 r186（2026-09-24）.
[^r3f]: pmndrs. (n.d.). react-three-fiber. GitHub. Retrieved 2026-10-17, from https://github.com/pmndrs/react-three-fiber —— 約 33.0k stars.
[^drei]: pmndrs. (n.d.). drei. GitHub. Retrieved 2026-10-17, from https://github.com/pmndrs/drei —— 約 10.0k stars.
[^sigma]: jacomyal. (n.d.). sigma.js. GitHub. Retrieved 2026-10-17, from https://github.com/jacomyal/sigma.js —— 約 12.2k stars；以 WebGL 渲染圖形.
[^cytoscape]: cytoscape. (n.d.). cytoscape.js. GitHub. Retrieved 2026-10-17, from https://github.com/cytoscape/cytoscape.js —— 約 11.2k stars；以 Canvas2D 渲染，提供 `hideEdgesOnViewport`、`textureOnViewport` 等渲染提示.
[^konva]: konvajs. (n.d.). Konva. GitHub. Retrieved 2026-10-17, from https://github.com/konvajs/konva —— 約 14.9k stars；Canvas2D 框架，`stage.scale()`／`stage.position()` 為基本變換，無內建 LOD.
[^pixi]: pixijs. (n.d.). PixiJS. GitHub. Retrieved 2026-10-17, from https://github.com/pixijs/pixijs —— 約 48.4k stars；以 WebGL/WebGPU 渲染，本身無內建視埠與 LOD，需搭配 `pixi-viewport`.
[^xyflow]: xyflow. (n.d.). xyflow. GitHub. Retrieved 2026-10-17, from https://github.com/xyflow/xyflow —— 約 38.7k stars；邊以 SVG path、節點以 HTML DOM 呈現，內建視埠縮放平移，無 LOD.
[^gisrepos]: MapLibre, Leaflet, OpenLayers 專案規模。Retrieved 2026-10-17, from https://github.com/maplibre/maplibre-gl-js （約 11.9k stars）；https://github.com/Leaflet/Leaflet （約 45.8k stars）；https://github.com/openlayers/openlayers （約 12.6k stars）。
[^sse]: CesiumGS. (n.d.). 3D Tiles specification. GitHub. Retrieved 2026-10-17, from https://github.com/CesiumGS/3d-tiles/blob/main/specification/README.adoc —— `geometricError` 定義為簡化表示相對原始幾何的誤差（公尺）；螢幕空間誤差 `SSE = (geometricError × viewportHeight) / (2 × distance × tan(fovY/2))`，當 SSE 超過 `maximumScreenSpaceError` 即細化至子節點.

