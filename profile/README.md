# 🌊 ReifyFlow

<div align="center">
  <img src="https://avatars.githubusercontent.com/u/248118397?s=400&u=0d029fc7994a08e90c07ae40a4fdb80fc5963073&v=4" width="120" alt="ReifyFlow Logo" />
  
  <h3>From Eidos to Matter.</h3>
  <p><b>从理念到物质。下一代 AI 原生嵌入式开发基础设施。</b></p>

  [![License](https://img.shields.io/badge/License-AGPL_v3-blue.svg)](LICENSE)
  [![Status](https://img.shields.io/badge/Status-MVP_Dev-orange.svg)]()
  [![Platform](https://img.shields.io/badge/Platform-STM32-blue.svg)]()
</div>

---

## 📖 项目简介 (Introduction)

**ReifyFlow** 是一个开源的工程化生态系统，致力于打破 **抽象软件逻辑 (Idea)** 与 **物理硬件实现 (Reality)** 之间的鸿沟。

传统的嵌入式开发中，代码编辑器、硬件配置工具（CubeMX）、数据手册（PDF）和调试工具是割裂的孤岛。ReifyFlow 通过 **AI Agent**、**标准化协议** 和 **双回环验证**，构建了一套 **AI Native** 的自动化流水线：

1.  **意图驱动**：用自然语言描述需求，AI 自动拆解任务。
2.  **软硬解耦**：业务逻辑与底层驱动物理隔离，支持在 PC 端进行纯软件仿真。
3.  **交互验证**：在生成代码前，通过可视化拓扑图确认硬件连接，拒绝黑盒操作。
4.  **自愈闭环**：通过硬件回环日志，自动诊断故障并修正底层配置。

---

## 🏗️  系统集成架构图 


```mermaid
graph TD
    %% ================== 样式定义 ==================
    classDef ui fill:#e1f5fe,stroke:#0277bd,stroke-width:2px,color:#000;
    classDef brain fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px,color:#000;
    classDef data fill:#fff9c4,stroke:#fbc02d,stroke-width:2px,stroke-dasharray: 5 5,color:#000;
    classDef tool fill:#fff3e0,stroke:#e65100,stroke-width:2px,color:#000;
    classDef ext fill:#cfd8dc,stroke:#37474f,stroke-width:2px,color:#000;

    %% ================== 1. 前端交互层 ==================
    UI["🟢 reify-studio<br>(VS Code Extension)"]:::ui

    %% ================== 2. 核心编排层 (Backend) ==================
    %% 修复：标题使用双引号包裹
    subgraph Core_Layer ["reify-core (Orchestrator)"]
        direction TB
        Orchestrator[总线调度器]:::brain
        Agents[AI Agents]:::brain
        SIL_Runner[SIL 软件仿真器]:::brain
        Log_Engine[日志分析引擎]:::brain
    end

    %% 连接 UI 与 Core
    UI <==>|JSON Protocol / WebSocket| Orchestrator

    %% ================== 3. 数据与协议层 ==================
    subgraph Data_Layer ["Knowledge Base"]
        direction TB
        Protocol["🟡 reify-protocol<br>(JSON Schemas)"]:::data
        Chips["🟣 reify-chips<br>(SVD / Meta / Templates)"]:::data
    end

    %% Core 读取数据
    Orchestrator -.-> Protocol
    Orchestrator -.-> Chips

    %% ================== 4. 驱动适配层 ==================
    subgraph Driver_Layer ["reify-driver (Adaptor)"]
        direction TB
        IOC_Handler[IOC Text Manipulator]:::tool
        Build_Runner[Build Runner]:::tool
    end

    %% Core 指挥 Driver
    Orchestrator ==>|执行指令| IOC_Handler
    Orchestrator ==>|执行指令| Build_Runner

    %% ================== 5. 外部工具链 (Vendor Tools) ==================
    subgraph Vendor_Tools ["STM32 Official Ecosystem"]
        direction TB
        CubeMX["STM32CubeMX CLI<br>(Code Generator)"]:::ext
        CubeCLT["STM32CubeCLT<br>(GCC / GDB / Programmer)"]:::ext
    end

    %% Driver 调用外部工具
    IOC_Handler -->|1. 修改| IOC_File(.ioc 文件):::ext
    IOC_File -->|2. 输入| CubeMX
    
    Build_Runner -->|3. 调用| CubeCLT

    %% ================== 6. 物理硬件回环 ==================
    Hardware["🖥️ Physical Hardware<br>(STM32 PCB)"]:::ext
    
    CubeCLT ==>|SWD Download| Hardware
    Hardware -.->|UART Telemetry| Log_Engine

    %% 这里的闭环：日志回到 Core 进行分析
    Log_Engine -.->|诊断结果| Orchestrator
```

---

## 🔄 全链路执行流程 (Execution Workflow)

这是一个 **人机协作 (Human-in-the-Loop)** 的过程。AI 负责繁琐的生成工作，人类负责关键节点的决策与确认。

```mermaid
sequenceDiagram
    autonumber
    
    %% ================= 角色定义 =================
    actor User as 👤 User
    participant Core as 🔵 reify-core<br>(AI Brain)
    participant Proto as 🟡 reify-protocol<br>(State/JSON)
    participant Chips as 🟣 reify-chips<br>(Knowledge)
    participant Driver as 🟤 reify-driver<br>(Execution)
    participant ExtTools as ⚙️ ST-Tools<br>(CubeMX/CLT)
    participant Board as 🖥️ Hardware

    %% ================= Phase 1: 意图与功能定义 =================
    Note over User, Board: ── Phase 1: Definition (定义意图) ──
    
    User->>Core: 1. 自然语言需求<br>"做个温控风扇，低成本"
    Core->>Core: AI 产品经理分析
    Core->>Proto: 💾 0_functional_spec.json<br>(Func: Temp_Control, Cost: Low)

    %% ================= Phase 2: 虚拟架构与约束注入 =================
    Note over User, Board: ── Phase 2: Virtualization & Constraints (虚拟化与约束) ──
    
    Core->>Proto: 读取功能规格
    Core->>Core: AI 架构师设计
    Core->>Proto: 💾 1_virtual_system.json<br>(Virtual: ADC, PWM, PID_Algo)
    
    %% ★★★ 关键点：硬件选型与约束提取 ★★★
    Core->>Chips: 🔍 选型 (Match "Low Cost")
    Chips-->>Core: 选中 STM32F030 (48MHz, No FPU)
    Core->>Proto: 💾 2_hw_constraints.json<br>(Limit: No Float, Low RAM)
    
    %% 约束反向注入软件设计
    Core->>Core: AI 软件工程师读取约束<br>decision: PID 使用定点数运算
    Core->>Proto: 💾 3_software_arch.json<br>(Logic: Fixed-Point PID)
    
    %% SIL 验证 (软件回环)
    Core->>Core: 生成 mock_driver.c & app.c
    Core->>Core: 🧪 运行 SIL 仿真 (PC端)
    Core->>Proto: ✅ 软件逻辑验证通过

    %% ================= Phase 3: 物理映射与交互确认 =================
    Note over User, Board: ── Phase 3: Mapping & Verification (映射与交互) ──
    
    Core->>Chips: 读取 SVD/Datasheet
    Core->>Core: 尝试引脚分配 (Auto-Routing)
    Core->>Proto: 💾 4_physical_map.json<br>(Map: ADC->PA0, PWM->PA6)
    
    %% ★★★ 人机交互点 ★★★
    Core-->>User: 🎨 发送数据至 reify-studio<br>(渲染拓扑连线图)
    User->>Core: 🖱️ 确认/拖拽修改引脚
    Core->>Proto: 🔒 锁定物理映射

    %% ================= Phase 4: 落地实现 (CubeMX + CubeCLT) =================
    Note over User, Board: ── Phase 4: Realization (落地执行) ──
    
    %% A. 底层生成
    Core->>Driver: 调用生成指令
    Driver->>Proto: 读取 4_physical_map.json
    
    Driver->>Driver: 🐍 Python 修改 .ioc 文本<br>(Set PA0=ADC_IN0)
    Driver->>ExtTools: ⚙️ 调用 CubeMX CLI (-q script)<br>生成 HAL 库代码
    
    %% B. 适配层生成
    Core->>Core: 生成 adapter.c (胶水代码)<br>连接 HAL_ADC_GetValue 与 app_get_temp
    
    %% C. 构建与烧录
    Driver->>ExtTools: 🔨 调用 CubeCLT (GCC) 编译
    Driver->>ExtTools: ⚡ 调用 CubeCLT (Programmer) 烧录
    ExtTools->>Board: 写入固件

    %% ================= Phase 5: 物理回环诊断 =================
    Note over User, Board: ── Phase 5: Loopback Diagnosis (自愈) ──
    
    Board->>Driver: 📡 串口回传 Telemetry JSON
    Driver->>Core: 转发日志
    Core->>Core: 🩺 AI 诊断逻辑
    
    alt 发现异常
        Core->>Proto: 修改 4_physical_map 或 .ioc
        Core->>Driver: 🔄 触发重构流程
    else 运行正常
        Core-->>User: ✅ 任务完成
    end
```

---

## 🧩 仓库矩阵 (Repository Matrix)

ReifyFlow 采用微服务化的 **Multi-Repo** 架构，各模块职责分明：

| 仓库名称 | 核心职责 | 技术栈 |
| :--- | :--- | :--- |
| **[`reify-protocol`](https://github.com/ReifyFlow/reify-protocol)** | 定义所有组件交互的数据标准（软件定义、硬件映射、日志格式）。**所有开发由此开始。** | JSON Schema |
| **[`reify-core`](https://github.com/ReifyFlow/reify-core)** | 核心编排引擎。集成 LLM、SVD 解析器、PDF 检索引擎、日志分析器。 | Python (FastAPI) |
| **[`reify-driver`](https://github.com/ReifyFlow/reify-driver)** | 通用硬件适配器。屏蔽厂商差异，操作 CubeMX/CMake，调用编译器与下载器。 | Python CLI |
| **[`reify-studio`](https://github.com/ReifyFlow/reify-studio)** | 可视化交互工作台 (VS Code 插件)。提供硬件拓扑图绘制、手册联动阅读。 | TS / React |
| **[`reify-chips`](https://github.com/ReifyFlow/reify-chips)** | 芯片知识库。存放 SVD 寄存器定义、Datasheet 索引映射、代码模板。 | Data |

---

## 📂 标准化工程结构 (Project Structure)

ReifyFlow 生成的工程遵循严格的 **分层解耦** 原则：

```text
Project_Root/
├── .oef/                       # [OEF配置区] - 存放核心元数据
│   ├── task_spec.json          # 1. 任务规格书 (AI生成)
│   ├── arch_graph.json         # 2. 软件架构图数据
│   └── hw_map.json             # 3. 硬件映射数据
│
├── core/                       # [软件架构层 - 纯逻辑] - 可以在电脑上跑
│   ├── inc/
│   │   ├── interface_temp.h    # 抽象接口：只定义 Get_Temp()，不含 HAL 库
│   │   └── interface_motor.h
│   ├── src/
│   │   ├── app_main.c          # 调度器/主循环
│   │   └── business_logic.c    # 核心算法 (PID, 状态机)
│   └── tests/                  # SIL 测试代码 (PC端运行)
│
├── bsp/                        # [硬件实现层 - 适配器] - 连接软硬的胶水
│   ├── stm32f103/              # 具体芯片实现
│   │   ├── adapter_temp.c      # 实现 interface_temp.h，调用 HAL_ADC
│   │   └── adapter_motor.c     # 实现 interface_motor.h，调用 HAL_TIM
│
├── drivers/                    # [底层驱动层 - 自动生成] - CubeMX 的地盘
│   ├── STM32CubeMX/            # CubeMX 工程目录
│   │   ├── Core/Src/main.c     # 硬件初始化
│   │   └── Drivers/            # HAL 库文件
│   └── linker/                 # 链接脚本
│
├── tools/                      # [工具链]
│   ├── build.py                # 自动构建脚本
│   └── monitor.py              # 日志监控脚本
│
└── Makefile                    # 顶层构建规则
```

## 🔄 The Workflow (标准工作流)

我们定义了 **T-V-E-L** 标准流程，确保每一步都可控：

1. **Translate (翻译)** : AI 将自然语言需求转化为 **Task Spec** (JSON)。
2. **Verify (验证)** : `reify-studio` 渲染出硬件连线图，用户进行**可视化确认**。
3. **Execute (执行)** : `reify-driver` 修改底层配置，生成代码并烧录。
4. **Loopback (回环)** : 硬件日志回传，AI 进行故障诊断。

---

## 🗺️ MVP Roadmap (当前计划)

目前项目处于 **MVP 开发阶段**，主要聚焦于 STM32F103 的点灯与串口通信闭环。

- [ ] **Step 1**: 完成 `reify-protocol` 协议定义。
- [ ] **Step 2**: 实现 `reify-driver` 对 STM32CubeMX `.ioc` 文件的读写。
- [ ] **Step 3**: 跑通 "自然语言 -> 代码生成 -> 硬件运行" 的单向链路。
- [ ] **Step 4**: 开发 VS Code 插件可视化界面。

---

## 🤝 Join Us

ReifyFlow 是一个开放的实验。如果你对 **AI Agent**、**嵌入式开发自动化** 或 **编译器设计** 感兴趣，欢迎 Star 或提交 PR。

*License: AGPL-3.0 (Core/Driver) & MIT (Protocol/UI)*
