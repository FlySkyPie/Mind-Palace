# CityGML / CityJSON 是否有類似 OpenStreetMap 的本地社群？

## 結論

**CityGML 和 CityJSON 不存在類似 OpenStreetMap 的草根／志願者本地社群（如 osm.tw）。** 現存的交流管道均由學術機構或標準組織主導，而非公民自發組成的志願者團體。

---

## 現存交流管道（均為學術／機構主導）

| 管道 | 主導單位 | 性質 | 狀態 |
|------|----------|------|------|
| CityJSON GitHub Discussions[^cjgh] | TU Delft 3D Geoinformation 研究群 | 工具討論與問答 | 活躍，約 30+ 討論串 |
| SIG 3D[^sig3d] | 德國產官學聯盟 | 專業工作組 | **已停止運作** |
| OGC CityGML SWG[^ogc] | OGC 標準委員會 | 標準制定 | 僅限會員組織參與 |
| CityGML Wiki / Forum[^cgwiki] | KIT (Karlsruhe Institute of Technology) | 論壇與維基 | **實質關閉**，註冊需手動審核 |
| 3DCityDB[^3dcitydb] | TUM Geoinformatics 講座 | 開源資料庫 | 由學術單位維護 |
| GIS Stack Exchange (#citygml)[^gisse] | 一般 GIS 問答平台 | 51 則問題 | 非專屬社群 |
| Reddit r/gis[^reddit] | 一般 GIS 子版 | 零星討論 | 非專屬社群 |

---

## 為何不存在類似 OSM 的社群生態？

根本原因在於定位差異：

- **OpenStreetMap** 是 **群眾製圖計畫**，任何人都可以貢獻一條路、一棟建築的外框；其治理模式底層由 OSM Foundation 和各國本地分會（如 osm.tw）構成。
- **CityGML / CityJSON** 是 **專業 3D 城市模型的資料標準**，由學術機構（TU Delft、TUM）與標準組織（OGC）制定維護。產出 CityGML/CityJSON 資料需要 **專業級 3D 建模工具與知識**，門檻遠高於 OSM 的路線繪製，難以形成草根志願者參與模式。

現有的交流圈本質上是**開發者／工具社群**（GitHub Discussions）和**專業網絡**（SIG 3D），而非公民志願者團體。

---

## 中文圈狀況

搜尋 CityGML/CityJSON 在中文圈的社群（CityGML 社區、CityJSON 愛好者、QQ 群）未找到任何活躍的社群組織。相關內容僅限於：

- 少數 CSDN 教學文[^csdn]
- 知乎零星文章[^zhihu]
- CityGML 標準中文翻譯專案[^cncn] (GitHub)
- 鳳凰地理社群[^phoenix] 上的一篇 CityGML/CityJSON 入門文章

以上均不是專屬且活躍的社群。

---

[^cjgh]: CityJSON. (n.d.). CityJSON specs GitHub Discussions. Retrieved 2026-09-27, from https://github.com/cityjson/specs/discussions
[^sig3d]: SIG 3D. (n.d.). About SIG 3D. Retrieved 2026-09-27, from https://www.sig3d.org/en/home/about-sig3d
[^ogc]: Open Geospatial Consortium. (n.d.). CityGML 3.0 Standards Working Group. Retrieved 2026-09-27, from https://github.com/opengeospatial/CityGML-3.0CM
[^cgwiki]: CityGML Wiki. (n.d.). CityGML Forum. Retrieved 2026-09-27, from https://www.citygmlwiki.org/index.php/CityGML_Forum
[^3dcitydb]: 3DCityDB. (n.d.). 3DCityDB — The Open Source CityGML Database. Retrieved 2026-09-27, from https://www.3dcitydb.net/3dcitydb/index.php
[^gisse]: Stack Exchange. (n.d.). Questions tagged [citygml]. GIS Stack Exchange. Retrieved 2026-09-27, from https://gis.stackexchange.com/questions/tagged/citygml
[^reddit]: Reddit. (2021). CityJSON vs CityGML. r/gis. Retrieved 2026-09-27, from https://www.reddit.com/r/gis/comments/obovaf/cityjson_vs_citygml/
[^csdn]: CSDN. (2023). CityGML 介紹. Retrieved 2026-09-27, from https://blog.csdn.net/shebao3333/article/details/132285649
[^zhihu]: 知乎. (2023). CityGML 是什麼？Retrieved 2026-09-27, from https://zhuanlan.zhihu.com/p/650012472
[^cncn]: DayuYu-3D. (n.d.). CityGML-CN — CityGML 中文翻譯. GitHub. Retrieved 2026-09-27, from https://github.com/DayuYu-3D/CityGML-CN
[^phoenix]: 鳳凰地理社區. (n.d.). 使用 QGIS 瀏覽 CityGML/CityJSON 數據. Retrieved 2026-09-27, from https://www.phoenix-gis.cn/d/196-shi-yong-qgisliu-lan-citygmlcityjsonshu-ju