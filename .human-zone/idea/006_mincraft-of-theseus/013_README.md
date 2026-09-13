# 特修斯的 Minecraft

一個標準的 Minecraft 配置需要有兩個東西：

- Minecraft Client
- Minecraft Server

兩者都是 Mojang （現在是 Microsoft） 持有智慧財產權的產品，並且透過 Minecraft 的協定連線。

架設伺服器方面，為了用更少的資源提供更多的服務，並且並沒有渲染相關的需求，因此有強烈的動機被使用 Java 以外的語言重新實做，諸如：[Cuberite (C++)](https://github.com/cuberite/cuberite)、[Valence (Rust)](https://github.com/valence-rs/valence)、[bareiron (C)](https://github.com/p2r3/bareiron)...。

另外也有像 [Mineflayer (Javscript)](https://github.com/prismarinejs/mineflayer) 或 [Botcraft (C++)](https://github.com/adepierre/Botcraft) 這樣為了製作機器人而存在的函式庫可作為 Minecraft Client 端運作，[stevenarella (Rust)](https://github.com/iceiix/stevenarella) 或 [Leafish (Rust)](https://github.com/Lea-fish/Leafish) 這類專案則是試圖打造具有渲染能力的 Client 端。

因此倘若我們將任意一個開源的 Client 端和 Server 端做組合，Mojang 創造的程式碼實作就會完全消失，只留下 Minecraft 協定在兩個程式之間通訊。

這就是特修斯的 Minecraft。
