# Katabasis 專案

## Techstack

- https://github.com/cuberite/cuberite
  - 5.4k ⭐
  - C++
- https://github.com/adepierre/Botcraft
  - 639 ⭐
  - C++
- BehaviorTree
  - https://github.com/BehaviorTree/BehaviorTree.CPP
    - 4.2k ⭐
    - C++
  - https://github.com/BehaviorTree/Groot
    - 890 ⭐
    - C++
- https://github.com/openembedded/bitbake
  - 531 ⭐

## 技術決策

### OpenBMC

最近從 Web Frontend 轉職 OpenBMC，所以需要累積 C++ 的經驗，因此專案將使用以 C++ 為基礎的生態系，bitbake 的技術選型也是出於相同的考慮。

### BehaviorTree.CPP

Botcraft 雖然也有自己的行為樹實作，但是缺乏完整生態的配套，BehaviorTree.CPP 除了支援將行為樹的序列化與反序列化以外，也有圖形化編輯器、實時遙測、日誌播放...等功能。

### bitbake

Botcraft 的更新週期為一個月，屬於個人專案，作為函式庫使用的設計略有不足（例如將 Minecraft Assets 下載教本與編譯榜定），預計實做額外的 Meson 架空原本的 CMake 設定，除此之外若要建立其他組件可能面臨 C++ 仰賴鏈的問題。

bitbake 提供額外的抽象除了可以用來處理仰賴鏈的問題，亦可配置解偶的 patch，讓專案保持對上游更新能力。

## 命名

- Katabasis 是「冥界之旅」的意思。
- 特修斯去過冥界。
- 在「來自深淵」中，有著名為「上升負荷」與「絕界行」的名稱與概念。
- 專案本質上是一個熟悉 Typescript 的開發者下降至 C++ 領域的旅程。
