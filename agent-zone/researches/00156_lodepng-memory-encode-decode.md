# LodePNG 記憶體內 PNG 編解碼：不寫入檔案系統的操作方式

LodePNG 是一個純 C/C++ 的 PNG 編解碼函式庫，無需依賴 zlib 或 libpng，所有函式皆可直接操作記憶體緩衝區，無需寫入磁碟。[^lodepng]

## 核心原理

LodePNG 的所有編解碼函式皆以**記憶體緩衝區指標** (`unsigned char*`) 和**長度** (`size_t`) 作為輸入輸出介面。輸出緩衝區由 LodePNG 內部動態分配，使用完畢需呼叫 `free()`（C API）或由 `std::vector` 自動管理（C++ API）。[^lodepng]

## C API 用法

### 解碼（PNG 二進位資料 → 原始像素）

```c
// 從記憶體中的 PNG 資料解碼為 RGBA 像素
unsigned char* png_data;  // 已有的 PNG 二進位資料
size_t png_size;          // 已有的 PNG 資料長度

unsigned char* image;
unsigned width, height;

unsigned error = lodepng_decode32(&image, &width, &height, png_data, png_size);
// error == 0 表示成功
// image 為 width * height * 4 的 RGBA 像素陣列
free(image);
```

- `lodepng_decode32()`：輸出 RGBA 32-bit（每像素 4 bytes）
- `lodepng_decode24()`：輸出 RGB 24-bit（每像素 3 bytes）
- `lodepng_decode_memory()`：可自訂輸出色彩類型與位深度 [^decode_doc]

### 編碼（原始像素 → PNG 二進位資料）

```c
// 將原始像素編碼為記憶體中的 PNG 二進位資料
unsigned char* image;   // 已有的像素資料
unsigned w, h;          // 圖片寬高

unsigned char* png;
size_t png_size;

unsigned error = lodepng_encode32(&png, &png_size, image, w, h);
// error == 0 表示成功
// png 為完整的 PNG 檔案二進位資料（共 png_size bytes），可直接傳輸、嵌入或使用
free(png);
```

- `lodepng_encode32()`：輸入 RGBA 32-bit
- `lodepng_encode24()`：輸入 RGB 24-bit
- `lodepng_encode_memory()`：可自訂色彩類型與位深度 [^encode_doc]

## C++ API 用法

C++ API 使用 `std::vector<unsigned char>` 自動管理記憶體，無需手動 `free`。[^examples]

### 解碼

```cpp
std::vector<unsigned char> png_data;  // 已有的 PNG 二進位資料
std::vector<unsigned char> image;
unsigned width, height;

unsigned error = lodepng::decode(image, width, height, png_data);
// image 即為 RGBA 像素資料
```

### 編碼

```cpp
std::vector<unsigned char> image;  // 已有的像素資料（RGBA，w*h*4 bytes）
unsigned w = 256, h = 256;

std::vector<unsigned char> png;
unsigned error = lodepng::encode(png, image, w, h);
// png 即為編碼後的完整 PNG 二進位資料
```

`encode()` 與 `decode()` 函式也支援接收 `(const unsigned char*, size_t)` 原始指標，與外部資料來源相容。[^examples]

## 輸出緩衝區的後續用途

編碼後的 `png` 向量或 `png`/`png_size` 指標值為**完整的 PNG 檔案二進位資料**，可直接：

- 透過網路 socket 傳送
- 嵌入 HTML/Base64（`data:image/png;base64,...`）
- 傳遞給 GPU/紋理載入函式
- 傳入其他需要 PNG 二進位串流的函式庫

無需回寫至磁碟，除非使用者明確需要存檔。

## 錯誤處理

所有編解碼函式回傳 `unsigned` 錯誤代碼，`0` 代表成功。可透過 `lodepng_error_text(error)` 取得人類可讀的錯誤描述。

[^lodepng]: LodePNG homepage. (n.d.). LodePNG. Retrieved 2026-09-25, from https://lodev.org/lodepng/
[^decode_doc]: LodePNG header documentation (`lodepng.h`). Retrieved 2026-09-25, from https://raw.githubusercontent.com/lvandeve/lodepng/master/lodepng.h
[^encode_doc]: LodePNG example_encode.cpp. Retrieved 2026-09-25, from https://github.com/lvandeve/lodepng/blob/master/examples/example_encode.cpp
[^examples]: LodePNG examples directory. Retrieved 2026-09-25, from https://github.com/lvandeve/lodepng/tree/master/examples