# Python 單元測試框架與函式庫調查報告

## 概述

本報告調查 Python 生態系中主要的單元測試框架與相關函式庫，涵蓋測試執行器、模擬（mocking）、屬性型測試（property-based testing）、覆蓋率測量、測試資料生成、以及跨環境測試自動化等面向。報告基於 2023 年至 2026 年間的網路資料與官方文件撰寫。

## 主要測試框架

### pytest

pytest 是目前 Python 生態系中最廣泛使用的測試框架，JetBrains 開發者調查中持續排名第一[^jetbrains-survey]。其核心特色為簡潔的 `assert` 語法（不需包裝在 `assertEqual` 等方法中）、強大的 fixture 系統（支援函式/模組/工作階段範圍的依賴注入）、以及 `@pytest.mark.parametrize` 裝飾器實現的參數化測試。截至 2026 年，pytest 已發展至 v8.x，擁有超過 800 個外掛，包括 `pytest-cov`（覆蓋率）、`pytest-xdist`（平行執行）、`pytest-mock`（模擬工具整合）、`pytest-asyncio`（非同步測試）、`pytest-django`（Django 整合）等[^momentic-2026]。pytest 可以原生執行 `unittest.TestCase` 風格的測試，無需修改，且自動發現命名為 `test_*.py` 的檔案和 `test_*` 開頭的函式。

**優勢**：社群最大、語法簡潔、外掛生態豐富、IDE 支援完善、可原生執行 unittest 測試。
**劣勢**：為第三方套件（非標準庫）、初學者對 fixture 系統有學習曲線。

### unittest（PyUnit）

unittest 是 Python 標準庫內建的測試框架，源自 JUnit 的 xUnit 風格[^python-unittest]。使用類別式組織（`class MyTest(unittest.TestCase)`）、內建斷言方法（`assertEqual`、`assertRaises` 等）、以及 `setUp`/`tearDown` 各層級的生命週期方法。unittest 也是 `unittest.mock` 模組的宿主。

**優勢**：零依賴、穩定、文件完善、對 xUnit 背景開發者友善。
**劣勢**：語法冗長、缺乏參數化測試原生支援、無 fixture 注入、camelCase 命名風格與 Python 慣例不一致。

### nose2

nose 的繼承者，基於 unittest2 擴展，提供更好的外掛支援和測試自動發現[^nose2-docs]。然而官方文件明確建議新專案考慮 pytest，認為其維護團隊更大、社群更活躍[^pytest-recommendation]。nose2 目前維護頻率低，`nose2-cov` 覆蓋率外掛已被棄用。

**注意**：原始的 **nose** 框架已被官方宣告棄用（deprecated），自 2015 年最後一版 1.3.7 後停止維護[^nose-deprecated]。**不應在新專案中使用**。

### testify

由 Yelp 開發的測試框架，提供類別層級 fixture、延遲評估的 deferred fixtures、以及裝飾器式 setup/teardown[^testify-pypi]。最後 PyPI 版本為 0.1.5（2015 年），已無維護。

## 屬性型測試

### hypothesis

hypothesis 是一個基於屬性的測試（property-based testing）函式庫，使用者定義不變量（invariant），hypothesis 會自動生成大量的隨機邊界輸入來找出失敗案例，並自動縮小（shrinking）到最簡單的觸發範例[^hypothesis-docs]。整合 pytest 和 unittest，支援 `@given` 裝飾器搭配各種策略（strategies），包括整數、字串、列表、numpy/pandas 的資料型別。

**優勢**：能找到傳統範例式測試難以發現的邊界錯誤、自動縮小失敗案例。
**劣勢**：執行速度較慢、需要不同的測試思維模式。

## 測試環境自動化

### tox

tox 是命令列層級的 CI 前端工具，透過 `tox.ini` 或 `pyproject.toml` 宣告式設定，自動建立隔離的虛擬環境來測試多個 Python 版本和依賴組合[^tox-docs]。確保套件在發布前能在不同環境中正確安裝。

**優勢**：多環境測試的事實標準、減少 CI 設定重複。

### nox

nox 與 tox 相似，但使用標準 Python 檔案（`noxfile.py`）進行設定，提供指令式（imperative）的彈性控制[^nox-docs]。支援迴圈、條件判斷、參數化工作階段。

**優勢**：比 tox 更靈活、適合複雜動態的工作流程。
**劣勢**：生態系較小、採用度不如 tox。

## 覆蓋率工具

### coverage.py

coverage.py 是 Python 程式碼覆蓋率測量的標準工具，支援行覆蓋率和分支覆蓋率，輸出格式包括文字、HTML、XML、LCOV、JSON 等[^coverage-docs]。可透過 `coverage run -m pytest` 或 `pytest-cov` 外掛與 pytest 整合。資料儲存在 SQLite 資料庫中，支援進階分析。

**優勢**：精確快速、與所有主流測試框架整合、格式豐富。

## 模擬與測試替身

### unittest.mock

Python 3.3+ 標準庫中的模擬物件函式庫，提供 `Mock`、`MagicMock`、`AsyncMock` 類別、`patch` 裝飾器/上下文管理器、以及 `autospec` 自動規格比對功能[^unittest-mock-docs]。可透過 `pytest-mock` 外掛以 pytest fixture 的方式使用。

### pytest-mock

提供 `mocker` fixture，將 `unittest.mock` 包裝為更方便的 pytest 風格使用方式[^pytest-mock]。

## 測試資料生成

### Factory Boy

宣告式測試資料生成函式庫，支援 Django 和 SQLAlchemy ORM，可與 Faker 整合產生擬真資料[^factory-boy-docs]。使用定義 Factory 類別的方式來建立測試物件。

### Faker

假資料生成函式庫，可產生姓名、地址、電子郵件、電話號碼等各類擬真資料[^faker-docs]，用於測試和資料庫填充。

## 文件中的可執行測試

### doctest

Python 標準庫中的模組，掃描 docstring 中看起來像 Python 交談會話的文字區塊並執行比對其輸出[^doctest-docs]。pytest 可透過 `--doctest-modules` 參數整合。

**優勢**：確保文件與程式碼同步、零安裝成本。
**劣勢**：僅比對輸出文字（脆弱）、無測試隔離或 fixture。

## 結論與建議

對於 2026 年的新 Python 專案，建議的測試技術棧為：

1. **pytest** — 作為主要測試執行器和框架
2. **pytest-cov** / **coverage.py** — 覆蓋率報告
3. **unittest.mock** 或 **pytest-mock** — 外部依賴模擬
4. **hypothesis** — 屬性型測試以發現邊界案例
5. **tox** 或 **nox** — 多 Python 版本 CI 測試
6. **factory_boy** + **Faker** — 測試資料生成

pytest 已成為 Python 測試的事實標準，其基本語法與 unittest 相當接近，但透過 fixture 和 parametrize 等功能大幅減少了測試程式碼的重複。unittest 作為標準庫選項，仍然適合對外部依賴有嚴格限制的專案或 xUnit 熟悉的團隊。

---

## 參考文獻

[^jetbrains-survey]: JetBrains. (2024). pytest vs unittest: Which One Should You Choose? Retrieved 2026-09-19, from https://blog.jetbrains.com/pycharm/2024/03/pytest-vs-unittest/

[^momentic-2026]: Momentic. (2026). Python Test Automation Frameworks: The Ultimate Guide for 2026. Retrieved 2026-09-19, from https://momentic.ai/blog/python-test-automation-frameworks

[^python-unittest]: Python Software Foundation. (n.d.). unittest — Unit testing framework. Retrieved 2026-09-19, from https://docs.python.org/3/library/unittest.html

[^nose2-docs]: nose2 Contributors. (n.d.). nose2 documentation. Retrieved 2026-09-19, from https://docs.nose2.io/en/latest/

[^pytest-recommendation]: nose2 Contributors. (n.d.). nose2 documentation — Getting Started. Retrieved 2026-09-19, from https://docs.nose2.io/en/latest/

[^nose-deprecated]: nose Contributors. (n.d.). nose documentation. Retrieved 2026-09-19, from https://nose.readthedocs.io/en/latest/

[^testify-pypi]: Python Package Index. (n.d.). testify 0.1.5. Retrieved 2026-09-19, from https://pypi.org/project/testify/

[^hypothesis-docs]: Hypothesis Contributors. (n.d.). Hypothesis documentation. Retrieved 2026-09-19, from https://hypothesis.readthedocs.io/

[^tox-docs]: tox Contributors. (n.d.). tox documentation. Retrieved 2026-09-19, from https://tox.wiki/

[^nox-docs]: nox Contributors. (n.d.). nox documentation. Retrieved 2026-09-19, from https://nox.thea.codes/en/stable/index.html

[^coverage-docs]: Batchelder, N. (n.d.). Coverage.py documentation. Retrieved 2026-09-19, from https://coverage.readthedocs.io/

[^unittest-mock-docs]: Python Software Foundation. (n.d.). unittest.mock — mock object library. Retrieved 2026-09-19, from https://docs.python.org/3/library/unittest.mock.html

[^factory-boy-docs]: Factory Boy Contributors. (n.d.). Factory Boy documentation. Retrieved 2026-09-19, from https://factoryboy.readthedocs.io/en/stable/

[^faker-docs]: Faker Contributors. (n.d.). Faker documentation. Retrieved 2026-09-19, from https://faker.readthedocs.io/

[^pytest-mock]: pytest-mock Contributors. (n.d.). pytest-mock documentation. Retrieved 2026-09-19, from https://pypi.org/project/pytest-mock/

[^doctest-docs]: Python Software Foundation. (n.d.). doctest — Test interactive Python examples. Retrieved 2026-09-19, from https://docs.python.org/3/library/doctest.html