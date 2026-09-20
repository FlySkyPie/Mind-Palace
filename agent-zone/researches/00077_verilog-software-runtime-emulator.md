# Verilog 作為 Linux 軟體程式的執行方式探討

## 動機

Verilog 是一種硬體描述語言（HDL），本質上是用來描述邏輯閘（AND、OR、NOT、flip-flop 等）的連線與行為。傳統上 Verilog 的執行方式有兩種：

1. **模擬（Simulation）** — 用 EDA 工具（如 ModelSim、VCS、Icarus）對設計施加測試訊號，觀察波形以除錯。
2. **合成（Synthesis）** — 將 Verilog 轉換為 FPGA 位元流（bitstream）或 ASIC 閘級網表，在真實晶片上執行。

但若想**像一般程式語言（C、Python）一樣直接將 Verilog 「跑起來」，和 Linux 上的其他軟體進行 IPC、網路通訊或檔案互動**，就需要一種「軟體執行期」（software runtime）的橋樑。本報告探討目前四種可行的技術方案。

## 方案一：Verilator — 編譯 Verilog 為原生 C++ 可執行檔

[Verilator](https://www.veripool.org/verilator/) 是目前最成熟、最快的開源方案。它將可綜合的 Verilog/SystemVerilog 設計轉換為**最佳化 C++ 程式碼**，再由 g++/clang 編譯為 Linux ELF 二進位檔[^verilator-doc]。

### 建立獨立執行檔

使用 `--binary` 旗標，一條指令即可完成所有工作：

```bash
verilator --binary -j 0 -Wall our.v
./obj_dir/Vour
```

此指令會自動：讀取 Verilog、產出 C++ 模型、建立 Makefile、編譯連結，最終產生一個可直接執行的 ELF binary[^verilator-guide]。

### 撰寫自訂 C++ Wrapper

若需控制執行流程並與外部軟體互動，使用 `--cc --exe --build` 加上自訂的 `sim_main.cpp`：

```cpp
#include "Vour.h"
#include "verilated.h"

int main(int argc, char** argv) {
    VerilatedContext* contextp = new VerilatedContext;
    contextp->commandArgs(argc, argv);
    Vour* top = new Vour{contextp};

    // 驅動輸入訊號
    top->clk = 1;
    top->data_in = 42;
    top->eval();

    // 讀取輸出
    printf("data_out = %d\n", top->data_out);

    while (!contextp->gotFinish()) {
        top->eval();
    }
    delete top;
    delete contextp;
}
```

### 與其他軟體互動的機制

Verilator 提供多種橋接方式[^verilator-connecting]：

| 機制 | 說明 |
|---|---|
| **DPI-C** (Direct Programming Interface) | Verilog 與 C++ 雙向函式呼叫，標準 IEEE 介面。從 Verilog 中 `import` C 函式，或從 C++ 呼叫 Verilog 的 `export` task。 |
| **VPI** (Verilog Procedural Interface) | 依名稱檢查與修改信號、註冊值變化回呼。 |
| **SystemC 封裝** (`--sc`) | 產出 `SC_MODULE`，可接入 SystemC 網路、與其他 SystemC IP 模組協同模擬。 |
| **Cocotb** | 用 Python 撰寫 testbench 透過 VPI/DPI 驅動 Verilog 模型。 |

因為 C++ wrapper 本身就是一般的 Linux 行程，從中可以使用**任何 POSIX IPC 機制**：socket、pipe、shared memory (`shm_open`/`mmap`)、message queue、DBus 等。

### 速度與限制

- 號稱「最快的開源 Verilog 模擬器」；單執行緒比 Icarus Verilog 快約 100 倍[^verilator-doc]。
- 支援多執行緒模擬 (`--threads N`)。
- **限制**：僅處理可綜合的 Verilog（加上部分 SystemVerilog）。它仍然是模擬器（simulator），會維護模擬時間與 delta cycle 語意。

## 方案二：Icarus Verilog + VVP — 編譯為位元組碼後解譯

[Icarus Verilog](https://steveicarus.github.io/iverilog/) 將 Verilog 編譯為一種專屬位元組碼格式（`.vvp` 檔案），然後由 **VVP（Verilog Virtual Processor）** 執行時期引擎解譯執行[^iverilog-vvp]。

```bash
iverilog -o my_design.vvp my_design.v
vvp my_design.vvp
```

VVP 擁有自己的指令集（`%set`、`%assign`、`%jmp`、`%fork`、`%join` 等），並維護事件排程器（event scheduler）、邏輯閘網路（由「函式器」functor 構成）與執行緒。它更像一個**專門為 Verilog 事件驅動語意設計的虛擬機器**[^iverilog-arch]。

VVP 支援動態載入的 **VPI 模組**（`.vpi` 檔），允許 C 程式碼在執行期與模擬互動。

## 方案三：Cascade — Verilog JIT 編譯器（已歸檔）

[Cascade](https://github.com/vmware-archive/cascade) 是 VMware Research 推出的**世界首個 Verilog JIT 編譯器**，發表於 ASPLOS 2019[^cascade-paper]。它代表了一種從根本上不同的方法：

### 兩階段執行模型

1. **軟體優先執行**：程式碼立即在內建的軟體模擬器中開始執行，提供 REPL 環境——輸入 Verilog 馬上看到結果。
2. **背景 JIT 編譯**：同時在背景使用傳統 FPGA 工具鏈（Verilator + Quartus/Yosys+NextPNR）將設計編譯到 FPGA 硬體。完成後，程式透明地從軟體**遷移到真實 FPGA** 上執行，速度大幅提昇。

此模型讓使用者獲得**即時回饋**（如同寫 JavaScript/Python），同時最終效能僅比完全編譯的設計慢約 3 倍以內，而最終確定設計則**無效能損失**[^cascade-readme]。

### 軟體後端

支援 `--march sw` 參數，表示純軟體執行。此時 Cascade 扮演一個虛擬 FPGA 模擬器——但與一般模擬器不同的是，它同時在背景編譯硬體。

### 特殊能力

Cascade 支援**不可綜合的系統任務**（`$display`、`$fopen`、`$fread`、`$fwrite`、`$save`/`$restart`），即使程式碼最終跑在 FPGA 硬體上時也支援——因為它可以將需要這些操作的部分動態遷移回軟體執行。

### 現狀

VMware 已於 **2021 年 7 月**將此專案歸檔，不再維護。原始碼仍可從 GitHub 取得與建置，但非活躍專案[^cascade-archive]。

## 方案四：Verilog 的標準 File I/O 系統任務

僅使用 Verilog 語言本身，可以透過標準系統任務與檔案系統互動[^verilog-fileio]：

- `$fopen` / `$fclose`
- `$fread` / `$fwrite` / `$fdisplay`
- `$fscanf`
- `$readmemh` / `$readmemb`（將檔案載入 memory array）
- `$swrite` / `$sformat`（字串格式化）

在此基礎上可實作基礎的行程間通訊——例如透過命名管道（FIFO）讓另一個行程寫入檔案，Verilog 從中讀取。不過此方式效率與彈性遠低於 DPI/VPI。

## 模擬與「程式執行」的根本差異

執行 Verilator 編譯後的 binary 時，**並非在「執行 Verilog 程式」**（如 C/Python 的指令序列），而是在執行一個**硬體行為的模擬模型**[^sim-vs-prog]。

| 特性 | 模擬（Verilator/Icarus） | 一般程式（C/Python） |
|---|---|---|
| **執行模型** | 事件驅動，delta cycle | 指令序列執行 |
| **並行性** | 所有 `always` 區塊「邏輯上同時」 | 單執行緒循序；多執行緒需顯式管理 |
| **時間概念** | `#10` 延遲、`@(posedge clk)`、模擬時間 | 掛鐘時間（僅 sleep） |
| **信號值** | 四值邏輯：0、1、X、Z | 布林值、整數、浮點數 |
| **指定方式** | 阻塞 (`=`) vs 非阻塞 (`<=`) 有根本差異 | 所有指定立即生效 |
| **平行排程** | 排程器管理 delta cycle 與事件順序 | 作業系統排程執行緒 |
| **OS 介面** | 僅透過 DPI/VPI 或 `$fopen` | 完整系統呼叫、IPC、網路 |

## 結論與建議

若想在 Linux 上將 Verilog 當作「可與其他軟體互動的執行期程式」，**Verilator 是最務實的選擇**：

1. 它產出的是真正的原生 Linux ELF binary。
2. C++ wrapper 可自由使用 socket、pipe、shared memory 與其他行程通訊。
3. DPI-C 提供了 Verilog 與 C 世界的雙向函式橋樑。
4. 效能極高，適合需要持續執行的應用（如軟體定義無線電、協定處理、數位信號處理）。

**需要注意**：即便編譯為原生碼，Verilator 仍然是「模擬器」——它模擬的是硬體的並行/事件驅動行為，而非轉換為指令序列。這不是缺點，而是 Verilog 的本質。

Icarus Verilog + VVP 則是需要快速原型或偏好解譯式工作流程時的替代方案。Cascade 雖概念前瞻但已歸檔，不建議作為新專案的基礎。

---

[^verilator-doc]: Veripool. (n.d.). Verilator — Overview. Retrieved 2026-09-20, from https://www.veripool.org/verilator/
[^verilator-guide]: Veripool. (n.d.). Verilator Guide — Verilating. Retrieved 2026-09-20, from https://verilator.org/guide/latest/verilating.html
[^verilator-connecting]: Veripool. (n.d.). Verilator Guide — Connecting to C++. Retrieved 2026-09-20, from https://verilator.org/guide/latest/connecting.html
[^iverilog-vvp]: Icarus Verilog. (n.d.). Simulation with VVP. Retrieved 2026-09-20, from https://steveicarus.github.io/iverilog/usage/simulation.html
[^iverilog-arch]: Icarus Verilog. (n.d.). VVP Architecture. Retrieved 2026-09-20, from https://steveicarus.github.io/iverilog/developer/guide/vvp/vvp.html
[^cascade-paper]: VMware Research. (2019). Just-in-Time Compilation for Verilog (ASPLOS 2019). Retrieved 2026-09-20, from https://dl.acm.org/doi/10.1145/3297858.3304010
[^cascade-readme]: VMware Research. (n.d.). Cascade README. Retrieved 2026-09-20, from https://github.com/vmware-archive/cascade
[^cascade-archive]: VMware Research. (2021). Cascade repository archived notice. Retrieved 2026-09-20, from https://github.com/vmware-archive/cascade
[^sim-vs-prog]: Various authors. (n.d.). Simulation vs Synthesis in HDL. Retrieved 2026-09-20, from https://github.com/h3nrry/notes/blob/master/hdl/verilog/detail/simulation_synthesis.md
[^verilog-fileio]: IEEE. (2005). IEEE Std 1364-2005 — Verilog HDL — System tasks and functions. Retrieved 2026-09-20, from https://ieeexplore.ieee.org/document/1620780