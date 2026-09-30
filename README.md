# USB 3.0 19Pin 1-to-4 Expansion Board

## 1. 项目目的 | Project Purpose

本项目旨在开发一款 PC 机箱内部专用的 **USB 3.0 19Pin 一进四“满定义”扩展板**：

- **解决接口匮乏**：主流主板通常仅有 1 个内置 19Pin 接口，难以满足多前置面板、一体水冷监控屏、RGB 控制器等内置设备的需求；
- **全速满定义扩展**：利用主板原生 19Pin 内部自带的两套 USB 3.0 通道，采用双 GL3510 控制器并行设计，将 1 个物理 19Pin 扩展为 4 个“满定义”19Pin 接口（对外提供 8 个完整独立的 USB 3.0 逻辑端口），避免传统 Hub 级联导致的带宽挤占与降速；
- **安全稳定供电**：采用板载 SATA 15Pin 辅助供电，主板供电与板载电源物理隔离防倒灌，确保多外设并发运行稳定。

---

## 2. 文档结构 | Document Structure

项目所有资料均按英文规范命名，并按研发流程顺序进行数字编号归档：

```text
USB3_19Pin_Expansion_Board/
│
├── 01_Product_Definition/          # 产品定义与需求
│   ├── 01_PRD.md                   # 产品需求文档
│   ├── 02_Competitive_Analysis.md  # 竞品分析
│   └── 03_Product_Roadmap.md       # 产品规划
│
├── 02_System_Design/               # 方案设计与架构
│   ├── 01_System_Architecture.md   # 系统架构设计
│   ├── 02_Chip_Selection.md        # 芯片与关键器件选型
│   ├── 03_Power_Design.md          # 电源与防倒灌方案
│   └── 04_Cost_Analysis.xlsx       # 成本核算分析表
│
├── 03_Hardware_Design/             # 硬件设计源文件
│   ├── 01_Schematic/               # 原理图工程及导出文件
│   ├── 02_PCB/                     # PCB Layout 工程源文件
│   ├── 03_BOM/                     # 物料清单 (BOM)
│   └── 04_Component_Library/       # 原理图符号与 3D 封装库
│
├── 04_Manufacturing_Files/         # 生产加工工程资料
│   ├── 01_Gerber/                  # PCB 制板光绘文件
│   ├── 02_Stencil/                 # SMT 激光钢网文件
│   └── 03_Pick_and_Place/          # 贴片机元件坐标文件
│
├── 05_Testing_and_Validation/      # 测试与验证
│   ├── 01_Test_Report.xlsx         # 功能与性能测试报告
│   └── 02_Bug_Tracker.xlsx         # 硬件缺陷与整改记录
│
└── 06_Mass_Production/             # 量产与支持资料
    ├── 01_User_Manual.md           # 用户使用与安装手册
    ├── 02_Maintenance_Manual.md    # 产线检修与维修手册
    └── 03_Revision_History.md      # 硬件版本变更记录
```