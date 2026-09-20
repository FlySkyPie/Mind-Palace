# C++ 中將 Raw RGB 陣列編碼為 PNG 的函式庫調查

## 概述

在 C/C++ 開發中，將原始 RGB 像素陣列（raw RGB array）編碼為 PNG 圖片是一個常見需求。本報告調查最適合此用途的輕量級 C/C++ 函式庫，重點關注支援直接將記憶體中的 `unsigned char*` RGB/RGBA 資料寫入 PNG 檔案的功能。

---

## 推薦函式庫

### 1. stb_image_write（最推薦：輕量、單檔、零依賴）

**說明**

stb_image_write 是 Sean Barrett 開發的單檔公共領域 C/C++ 函式庫，隸屬於知名的 stb 函式庫集合。它只需一個檔案 `stb_image_write.h`，在使用前定義 `STB_IMAGE_WRITE_IMPLEMENTATION` 巨集即可獲得實作。它支援輸出 PNG、BMP、TGA、JPEG、HDR 五種格式。[^stb]

**核心 API**

```c
// 寫入 PNG 檔案
int stbi_write_png(char const *filename, int w, int h, int comp, const void *data, int stride_in_bytes);

// 寫入記憶體 callback 版本
int stbi_write_png_to_func(stbi_write_func *func, void *context, int w, int h, int comp, const void *data, int stride_in_bytes);
```

- `comp` 參數：1=灰階, 2=灰階+Alpha, 3=RGB, 4=RGBA
- `data` 指向左上角第一個像素的記憶體位址
- 像素排列為 row-major 交錯格式 `R, G, B, R, G, B, ...`

**使用範例**

```c
#define STB_IMAGE_WRITE_IMPLEMENTATION
#include "stb_image_write.h"

// 建立 256x256 的 RGB 漸層圖
unsigned char pixels[256 * 256 * 3];
for (int y = 0; y < 256; y++)
    for (int x = 0; x < 256; x++) {
        pixels[(y * 256 + x) * 3 + 0] = x;         // R
        pixels[(y * 256 + x) * 3 + 1] = y;         // G
        pixels[(y * 256 + x) * 3 + 2] = 128;       // B
    }

stbi_write_png("gradient.png", 256, 256, 3, pixels, 256 * 3);
```

**授權條款**：Public Domain（無限制）或 MIT License 二擇一。[^stb_license]

---

### 2. LodePNG

**說明**

LodePNG 是 Lode Vandevenne 開發的單檔、零依賴 C/C++ PNG 編解碼函式庫。整個函式庫只有 `lodepng.h` 和 `lodepng.cpp`（或 `.c`）兩個檔案，無需連結 zlib、libpng 或其他外部函式庫。它內建了 deflate 壓縮、CRC 計算、zlib 格式處理等所有 PNG 內部操作。[^lodepng]

**核心 API**

```c
// 簡易 RGB 編碼（每像素 3 bytes）
unsigned lodepng_encode24_file(const char* filename,
                                const unsigned char* image,
                                unsigned w, unsigned h);

// 簡易 RGBA 編碼（每像素 4 bytes）
unsigned lodepng_encode32_file(const char* filename,
                                const unsigned char* image,
                                unsigned w, unsigned h);

// 進階版本：編碼至記憶體
unsigned lodepng_encode_memory(unsigned char** out, size_t* outsize,
                                const unsigned char* image,
                                unsigned w, unsigned h,
                                LodePNGColorType colortype,
                                unsigned bitdepth);

// C++ 簡易版本
void lodepng::encode(std::vector<unsigned char>& out,
                     const unsigned char* image,
                     unsigned w, unsigned h,
                     LodePNGColorType colortype,
                     unsigned bitdepth);
```

**使用範例**

```c
#include "lodepng.h"

unsigned char rgb_data[width * height * 3];
// ... 填入像素資料 ...
unsigned error = lodepng_encode24_file("output.png", rgb_data, width, height);
if (error) printf("error %u: %s\n", error, lodepng_error_text(error));
```

**授權條款**：zlib-style（極為寬鬆，可用於商業及專有軟體）。[^lodepng_license]

---

### 3. PNGwriter

**說明**

PNGwriter 是基於 libpng 的 C++ 封裝函式庫，支援 Linux、Unix、macOS 和 Windows。它提供更高層級的 API，如畫點、畫線、填色矩形、文字渲染等，適合需要影像生成而不僅僅是編碼的場景。需額外連結 libpng 與 FreeType2（若需要文字支援）。[^pngwriter]

**使用範例**

```cpp
#include <pngwriter.h>

pngwriter png(width, height, 0, "output.png");
png.plot(x, y, red, green, blue);       // 逐像素繪製
png.filledsquare(x1, y1, x2, y2, r, g, b); // 填滿矩形
png.close();  // 寫入磁碟
```

**授權條款**：GPL v3 / LGPL（需注意授權相容性）。[^pngwriter_license]

---

### 4. FreeImage

**說明**

FreeImage 是一個功能完整的 C 語言影像處理函式庫，支援 BMP、PNG、JPEG、TIFF 等 30 種以上格式。它提供 Raw bits 存取與格式轉換 API，可將記憶體中的 raw RGB 資料封裝為 FIBITMAP 物件後輸出為 PNG。體積較大，但功能全面。[^freeimage]

**使用範例**

```c
#include "FreeImage.h"

FreeImage_Initialise();
FIBITMAP* bitmap = FreeImage_AllocateT(FIT_BITMAP, width, height, 24);
// 填入像素資料...
FreeImage_Save(FIF_PNG, bitmap, "output.png");
FreeImage_Unload(bitmap);
FreeImage_DeInitialise();
```

**授權條款**：FreeImage Public License（類 GPL，但允許專有軟體使用，需附加宣告）。[^freeimage_license]

---

### 5. libpng（底層官方參考函式庫）

**說明**

libpng 是 PNG 格式的官方參考實作函式庫（C 語言），提供最完整的控制，但也需要較多的樣板程式碼。需同時連結 zlib。適合對 PNG 編碼過程有精細控制需求的場景，一般應用建議優先選用 stb_image_write 或 LodePNG。[^libpng]

---

## 比較表

| 函式庫 | 檔案形式 | 外部依賴 | 一行寫 PNG | 授權條款 | 推薦場景 |
|---|---|---|---|---|---|
| **stb_image_write** | 單檔 header-only | 無 | `stbi_write_png()` | Public Domain / MIT | ⭐ 輕量首選 |
| **LodePNG** | 單檔 + .cpp | 無 | `lodepng_encode24_file()` | zlib-style | ⭐ 輕量首選 |
| PNGwriter | 多檔 | libpng + FreeType2 | `pngwriter::close()` | GPL/LGPL | 需要繪圖 API |
| FreeImage | 多檔 | 無（自含 codec） | `FreeImage_Save()` | FreeImage License | 多功能需求 |
| libpng | 多檔 | zlib | 約 30 行樣板 | libpng License | 底層控制 |

---

## 總結

若目標是「將 raw RGB array 轉換為 PNG 圖片」，**最推薦 stb_image_write 或 LodePNG**：

- **stb_image_write**：一行程式碼、公有領域授權、無任何依賴，適合嵌入任何專案。
- **LodePNG**：同樣輕量零依賴，提供更完整的編解碼雙向支援與更佳的壓縮品質控制。

兩者皆只需將原始檔案複製到專案中即可使用，無需連結系統函式庫或處理複雜的建置流程。

---

[^stb]: Sean Barrett. (n.d.). stb_image_write.h — public domain image writing library for C/C++. Retrieved 2026-09-20, from https://github.com/nothings/stb/blob/master/stb_image_write.h
[^stb_license]: stb_image_write is available under either MIT License or Public Domain (unlicense.org). Retrieved 2026-09-20, from https://github.com/nothings/stb
[^lodepng]: Lode Vandevenne. (n.d.). LodePNG — PNG encoder/decoder in C and C++. Retrieved 2026-09-20, from https://github.com/lvandeve/lodepng
[^lodepng_license]: LodePNG uses a zlib-style permissive license, no attribution required. Retrieved 2026-09-20, from https://github.com/lvandeve/lodepng
[^pngwriter]: PNGwriter — C++ library for writing PNG images. Retrieved 2026-09-20, from https://pngwriter.sourceforge.net/
[^pngwriter_license]: PNGwriter is licensed under GPL v3 / LGPL. Retrieved 2026-09-20, from https://sourceforge.net/projects/pngwriter/
[^freeimage]: FreeImage — Open Source C library for reading and writing image files. Retrieved 2026-09-20, from https://freeimage.sourceforge.io/
[^freeimage_license]: FreeImage is distributed under the FreeImage Public License, a GPL-compatible license allowing proprietary use with notice. Retrieved 2026-09-20, from https://freeimage.sourceforge.io/license.html
[^libpng]: PNG Development Group. (n.d.). libpng — Official PNG reference library. Retrieved 2026-09-20, from http://www.libpng.org/pub/png/libpng.html