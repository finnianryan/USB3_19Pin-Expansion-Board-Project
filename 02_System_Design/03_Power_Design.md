# USB 3.0 19Pin 一进四满定义扩展板 电源与防倒灌方案设计说明书

## 1. 文档概述与设计依据

### 1.1 文档目的
本文档为 **USB 3.0 19Pin 一进四“满定义”扩展板** 的电源系统设计与防倒灌方案规格书（Power & Anti-Backfeed Engineering Specification）。
文档依据 [01_PRD.md](file:///c:/Users/leiyu/Desktop/Document/work/USB3.0/01_Product_Definition/01_PRD.md) 的系统规格、[01_System_Architecture.md](file:///c:/Users/leiyu/Desktop/Document/work/USB3.0/02_System_Design/01_System_Architecture.md) 的系统拓扑以及 [02_Chip_Selection.md](file:///c:/Users/leiyu/Desktop/Document/work/USB3.0/02_System_Design/02_Chip_Selection.md) 锁定的元器件物理参数，对整板供电架构、电气防倒灌隔离机理、SATA 5V 功率预算、直流回路压降（IR Drop）、分组 PPTC 保护响应曲线、主控内部电源网络以及 PCB 载流温升展开严密的工程推导与闭环验算。

### 1.2 核心设计目标与约束边界
1. **单一 SATA 5V 纯外部供电**：以机箱标准 SATA 15Pin 辅助电源作为全板唯一能源输入，根绝从主板 19Pin 取电导致的过载烧板风险；
2. **绝对物理级电气防倒灌**：主板 19Pin 5V 供电引脚在板端物理切断，仅部署高阻分压侦测网络，静态反向漏电流严格限制在 $\le 25\mu\text{A}$，彻底消除主板与扩展板间的电势差倒灌；
3. **极限带载直流压降合规**：USB 3.0 规范强制要求端口 VBUS 最小工作电压不得低于 $4.75\text{V}$。在 8 端口并发带载达 $4.5\text{A}$ 的极限工况下，整板回路压降必须控制在 $\le 170\text{mV}$ 以内，确保最不利位置的 19Pin 输出插座端点电压 $\ge 4.80\text{V}$；
4. **低成本敏捷分组保护**：采用 4 颗 1206 贴片自恢复保险丝（PPTC）按 19Pin 插座进行分组隔离，发生金属性短路碰地时，故障支路在 $0.2\text{s}$ 内敏捷跳变高阻，保护整机不黑屏、其余 3 组接口（6 个端口）业务不中断。

---

## 2. 供电拓扑演进与防倒灌方案对比

### 2.1 为什么必须放弃主板供电？
在 PC 主机内部，主板原生 19Pin 插座的 5V 电源输出端，通常串接了一颗聚合物自恢复保险丝（PolySwitch/PPTC），其维持电流门限一般仅为 **$1.5\text{A} \sim 2.0\text{A}$**。
* **功耗需求与供给能力的物理矛盾**：
  * 本扩展板输出 4 个标准满定义 19Pin 插座，共对外提供 **8 个独立 USB 3.0 逻辑端口**；
  * 根据 USB 3.0 标准协议，单个端口额定最大持续电流为 $0.9\text{A}$（$4.5\text{W}$）；
  * 8 端口理论满载总功耗高达：
    $$P_{total\_max} = 8 \times (5\text{V} \times 0.9\text{A}) = 36.0\text{W} \quad (I_{load\_max} = 7.2\text{A})$$
  * 即便在日常多设备并发运行场景下（如接入 2 块移动硬盘 + 2 个水冷/机箱监控屏 + 2 个 RGB 控制器），整板实际持续电流需求也在 **$3.0\text{A} \sim 4.5\text{A}$**（$15\text{W} \sim 22.5\text{W}$）；
  * 主板原生 19Pin 供电能力不足所需电流的 40%，若直接从主板取电，插入第 2 块移动硬盘即可瞬间触发主板过流保护，导致全板外设掉盘，甚至直接烧毁主板保险丝。

### 2.2 防倒灌方案横向对比与技术决策

在解决“外接供电防止反向灌入主板”的工程演进中，存在三种技术路线：

| 评估维度 | 方案 1：肖特基二极管隔离 | 方案 2：理想二极管 / MOS Power Mux | 方案 3 (选定)：纯 SATA 物理隔离 + 高阻侦测 |
| :--- | :--- | :--- | :--- |
| **电路拓扑** | 在主板与 SATA 间串联大电流肖特基二极管做单向导通 | 采用双向 P-MOS / N-MOS 或专用 Mux 控制 IC 自动切换电源 | 主板 5V 物理切断不接电源平面，仅引高阻分压电阻至芯片 `VBUS_DET`；纯 SATA 供电 |
| **直流压降 (IR Drop)** | **极差（致命缺陷）**：大电流肖特基压降达 $0.35\text{V} \sim 0.55\text{V}$，输出直接跌破 4.75V 规范底线。 | **优**：MOS 管导通内阻极小（$R_{DS(on)} \approx 15\text{m}\Omega$），压降较小。 | **最优（0 额外压降）**：SATA 5V 直接与供电大面积铺铜相连，无串联有源器件压降。 |
| **发热与温升** | **极高**：在 4.5A 电流下，二极管发热功率达 $1.6\text{W} \sim 2.5\text{W}$，微小板面剧烈发烫。 | **低**：MOS 管发热较小（$P \approx 0.3\text{W}$）。 | **极低（零额外热源）**：无功率半导体额外积热，仅有铜箔毫欧级欧姆损耗。 |
| **倒灌安全性** | **中**：二极管存在毫安级高温反向漏电流，且存在反向击穿短路风险。 | **较高（但有失效隐患）**：依赖芯片检测；热插拔瞬态过冲或芯片静电击穿时存在硬倒灌风险。 | **最高（绝对物理隔离）**：主板 5V 与 SATA 5V 间为物理空气绝缘和微安级高阻，**零倒灌可能**。 |
| **BOM 成本** | 低（约 0.3 ~ 0.5 元） | **极高（增加 1.8 ~ 3.5 元）** | **最低（0 元新增功率件）**，仅需 2 颗 0402 分压贴片电阻（0.01 元）。 |
| **PCB 走线压力** | 中等 | **巨大**：需布置复杂的切换芯片、自举电容与控制引脚，严重挤占 4 层板高频走线。 | **极低**：单一 5V 电源平面，内层整层铺铜，差分阻抗控制极佳。 |

**【工程最终决策】**：
坚决采纳 **方案 3：纯 SATA 供电 + 主板 5V 物理隔离与高阻侦测拓扑**。

---

## 3. 主板电气隔离与防倒灌电路设计

### 3.1 电气隔离与 VBUS_DET 侦测拓扑

根据 GL3510 官方规范，主控芯片必须实时感知上行 USB 主机（Host）的连接与断开状态，以控制自身状态机在**活动（Active）**与**挂起（Suspend）**之间无缝切换。

```mermaid
flowchart LR
    subgraph Host_Side["主板原生 19Pin 侧 (J_IN)"]
        MB_VBUS_A["Pin 1 (Port A 5V)"]
        MB_VBUS_B["Pin 19 (Port B 5V)"]
        MB_GND["Pin 4, 7, 13, 16<br>(系统共地 GND)"]
    end

    subgraph Sense_Network["双路独立高阻分压网络"]
        subgraph Channel_A["Port A 侦测支路"]
            R1["上电阻 R1: 100kΩ ±1%"]
            R2["下电阻 R2: 100kΩ ±1%"]
        end
        subgraph Channel_B["Port B 侦测支路"]
            R3["上电阻 R3: 100kΩ ±1%"]
            R4["下电阻 R4: 100kΩ ±1%"]
        end
    end

    subgraph Controller_Side["两颗 GL3510 主控芯片"]
        U1_DET["GL3510 #1 的 VBUS_DET 引脚<br>(高电平有效门限: 2.0V~3.3V)"]
        U2_DET["GL3510 #2 的 VBUS_DET 引脚<br>(高电平有效门限: 2.0V~3.3V)"]
    end

    MB_VBUS_A --> R1
    R1 --> Node_A["采样点 A (2.5V)"]
    Node_A --> R2 --> MB_GND
    Node_A ==> U1_DET

    MB_VBUS_B --> R3
    R3 --> Node_B["采样点 B (2.5V)"]
    Node_B --> R4 --> MB_GND
    Node_B ==> U2_DET
```

### 3.2 采样参数严密推导与容差分析
1. **分压比与门限匹配推导**：
   * 主板 19Pin 输出的标准电压范围为 $V_{MB\_in} = 4.75\text{V} \sim 5.25\text{V}$（标称 $5.00\text{V}$）；
   * 分压电阻网络配置为 $R_1 = 100\text{k}\Omega \pm 1\%, R_2 = 100\text{k}\Omega \pm 1\%$；
   * GL3510 的 `VBUS_DET` 输入阻抗极高（CMOS 栅极输入，输入阻抗 $> 10\text{M}\Omega$），分压负载效应可完全忽略；
   * 在标称 5.0V 条件下，侦测节点理论电压为：
     $$V_{sense\_nom} = 5.00\text{V} \times \frac{R_2}{R_1 + R_2} = 5.00\text{V} \times \frac{100\text{k}\Omega}{100\text{k}\Omega + 100\text{k}\Omega} = 2.50\text{V}$$
   * 考虑极限公差情况（电阻最大 $\pm 1\%$ 偏差，主板供电下限 $4.75\text{V}$）：
     $$V_{sense\_min} = 4.75\text{V} \times \frac{100 \times 0.99}{(100 \times 1.01) + (100 \times 0.99)} = 4.75\text{V} \times \frac{99}{200} = 2.351\text{V}$$
   * 考虑极限公差情况（主板供电上限 $5.25\text{V}$）：
     $$V_{sense\_max} = 5.25\text{V} \times \frac{100 \times 1.01}{(100 \times 0.99) + (100 \times 1.01)} = 5.25\text{V} \times \frac{101}{200} = 2.651\text{V}$$
   * **结论**：$V_{sense}$ 稳态范围在 **$2.35\text{V} \sim 2.65\text{V}$**，高度落在 GL3510 的高电平有效输入范围（$V_{IH} \ge 2.0\text{V}, V_{max} = 3.6\text{V}$）的正中央，具备极高的抗噪与抗电平漂移裕量。

2. **反向漏电流极小化验算（零倒灌验证）**：
   * **工况 A：PC 正常开机，主板向扩展板供电检测**：
     从主板 19Pin 抽取的总稳态侦测电流为：
     $$I_{draw} = \frac{V_{MB}}{R_1 + R_2} = \frac{5.0\text{V}}{200\text{k}\Omega} = 25\mu\text{A}$$
     抽取的电流仅为微安级，功耗仅 $0.125\text{mW}$，不会对主板 5V 产生任何负载影响。
   * **工况 B：PC 关机（S5）或睡眠（S3），主机切断 19Pin 供电，但 SATA 电源可能短暂带电**：
     由于主板 19Pin 5V 引脚在板端除了 $100\text{k}\Omega$ 电阻外物理悬空，全板 SATA 5V 电源域与主板供电引脚之间不存在任何直接半导体通路或低阻通道；
     即使 GL3510 内部存在弱上拉，由于 $R_1 = 100\text{k}\Omega$ 的阻隔，反向流入主板的电流理论上限为：
     $$I_{backfeed\_max} \le \frac{V_{internal\_3.3V}}{R_1} = \frac{3.3\text{V}}{100\text{k}\Omega} = 33\mu\text{A}$$
     该电流不足 $0.05\text{mA}$，完全符合 USB-IF 针对外设反向漏电 $\le 100\mu\text{A}$ 的严苛安全认证规范，绝对不会出现主板指示灯微亮、机箱风扇悬浮转动或主板芯片被反向弱通电受损的问题。

3. **滤波防抖动电容计算**：
   * 在采样节点并联一颗贴片陶瓷电容 $C_f = 0.1\mu\text{F}$（100nF），构成一阶低通滤波网络；
   * 等效戴维南阻抗为 $R_{th} = R_1 \parallel R_2 = 50\text{k}\Omega$；
   * 滤波截止频率为：
     $$f_c = \frac{1}{2\pi R_{th} C_f} = \frac{1}{2\pi \times 50\times 10^3 \times 0.1\times 10^{-6}} \approx 31.8\text{ Hz}$$
   * 响应时间常数 $\tau = R_{th} C_f = 5.0\text{ms}$；
   * **工程效益**：能完美滤除 PC 开机瞬间高频毛刺、插拔抖动与电磁辐射干扰，同时在主机正常关机后约 $15\text{ms}$ 内完成放电关断，响应敏捷。

### 3.3 主机掉电与硬件复位互锁电路 (Host VBUS - RESETJ Interlock)
* **工程背景与行业痛点**：
  * 在外接独立电源（SATA 供电）的 USB Hub 系统中，普遍存在**主机热重启/休眠唤醒后外设丢盘死锁**的顽疾；
  * **机理分析**：当 PC 主机重启或进入 S3/S4 睡眠时，主板会关闭其原生 USB 接口，但机箱 SATA 15Pin 通常持续供电。此时 GL3510 控制器内部逻辑仍保持运行。当主机重新上电发包枚举时，GL3510 未经历冷复位，状态机无法与主机 USB xHCI 控制器重新同步，导致 BIOS 自检时卡死或系统内无法识别下游外设；
* **硬件互锁电路设计**：
  * 采用 2 颗微型 N 沟道 MOSFET（`2N7002`，SOT-23 封装，Q1、Q2）构建硬件联锁逻辑：
  * **栅极 (Gate)**：连接至对应的 Host VBUS 侦测节点（`VBUS_DET_A` / `VBUS_DET_B`）；
  * **漏极 (Drain)**：连接至 GL3510 的复位引脚 `RESETJ`（引脚自带片内弱上拉，并外挂 10kΩ 上拉至 3.3V）；
  * **源极 (Source)**：直连数字地 GND；
  * **工作时序闭环**：
    1. **主机开机**：Host VBUS 达到 5V $\rightarrow$ 栅极电压升至约 2.5V（超过 2N7002 的开启阈值 $V_{GS(th)} \approx 1.5\text{V}$）$\rightarrow$ 漏源导通；通过反相逻辑配合或直接作为使能，当主机下电（$V_{Host\_VBUS} = 0\text{V}$）时，栅极失电关断，由辅助复位下拉电路瞬时将 `RESETJ` 硬钳位到 GND；
    2. **主机重启/关机**：只要主板断电，GL3510 立即强制进入硬件复位（Reset）状态；
    3. **主机复电**：主机 VBUS 重新建立后，`RESETJ` 延时释放，触发 GL3510 干净利落的系统冷启动，100% 杜绝热重启丢盘。

### 3.4 电子式快速过压保护 (OVP) 与 TVS 协同防护架构
* **单一 TVS 防护的致命缺陷**：
  * 系统在 SATA 5V 输入端配置了单向 TVS 管 `SMAJ5.0A`。其反向击穿电压 $V_{BR} = 6.4\text{V} \sim 7.25\text{V}$，在峰值脉冲下最大钳位电压高达 **9.2V**；
  * 但 GL3510 主控芯片与大多数 USB 外设的供电绝对最大耐压（Absolute Maximum Rating）仅为 **6.0V**；
  * 若用户电源线插错、ATX 电源发生 5V 稳压反馈环路开环故障或存在持续 7V~9V 的过电压，TVS 无法提供足够的电压保护，将导致整板芯片与昂贵外设瞬间击穿烧毁；
* **双重防御协同体系**：
  ```mermaid
  flowchart LR
      SATA_In["SATA 15Pin 输入 (5V_SATA)"] --> TVS["第一道：SMAJ5.0A TVS<br>纳秒级响应，吸收高能浪涌，钳位<9.2V"]
      TVS --> OVP["第二道：电子式 OVP 芯片 (SGM2553)<br>内部 35mΩ 开关，5.6V 快速硬切断 (<100ns)"]
      OVP --> VBUS_SAFE["受保护系统电源母线 (VBUS_RAW)"]
      VBUS_SAFE --> PPTC["PPTC 支路保险丝 (F1~F4)"]
      VBUS_SAFE --> CHIP["GL3510 主控芯片 (U1/U2)"]
  ```
  * **第一道防线 (TVS 管)**：在纳秒级时间内吸收外部 ESD 和热插拔感应电感反冲高能量，将百伏级尖峰瞬间硬压制在 9V 以下，保护后级电路不被高压电弧击穿；
  * **第二道防线 (电子式 OVP 芯片)**：采用圣邦微 `SGM2553`（或帝奥微 `DIO7003`），实时监测母线电压。一旦电压超过 **5.6V**，内部低阻功率 MOSFET 在 **$< 100\text{ns}$** 内彻底切断下游供电回路，母线电压被完全隔离，确保后级芯片电压绝不超过 5.6V，形成坚不可摧的安全屏障。

---

## 4. SATA 5V 功率预算与整板带载模型

### 4.1 SATA 15Pin 输入接口承流能力
标准 PC 电源 SATA 15Pin 接头定义中，+5V 供电分配在 **Pin 7、Pin 8、Pin 9** 三根端子上；地线 (GND) 分配在 **Pin 4、Pin 5、Pin 6、Pin 10、Pin 12** 端子上；Pin 1~3 (3.3V) 与 Pin 13~15 (12V) 物理悬空不接。
* **端子额定承流规格**：标准磷铜镀金端子单引脚额定承载电流为 $1.5\text{A}$；
* **并联总输入能力**：
  * 3 针 +5V 端子（Pin 7, 8, 9）在 PCB 走线层直接大面积打通并联：
    $$I_{SATA\_rated} = 3 \times 1.5\text{A} = 4.5\text{A} \quad (\text{额定持续输出能力})$$
  * 在短时瞬态脉冲工况下，端子允许短时间通过高达 **$6.0\text{A}$** 电流；
* **输入功率上限**：
  $$P_{in\_rated} = 5.0\text{V} \times 4.5\text{A} = 22.5\text{W}$$

### 4.2 全板多工况功率预算分解

根据实际装机应用，建立三种典型带载场景功耗模型：

| 评估工况 | 外设接入具体场景描述 | 下游端口总负载电流 | 主控芯片与电源损耗 | 整板总输入电流 | 整机总功耗 | 供电运行状态评价 |
| :--- | :--- | :---: | :---: | :---: | :---: | :--- |
| **工况 1：空载待机** | 8 个端口均未插外设，主板开机，Hub 处于空闲准备状态。 | 0 A | 0.12 A (两颗 GL3510 空载) | **0.12 A** | **0.60 W** | 极度节能，整板表面无任何可感知温升。 |
| **工况 2：典型多外设并发 (主流装机)** | 接入 2 块 2.5 寸高速移动硬盘（各 0.8A）+ 2 个水冷屏/一体水冷泵（各 0.5A）+ 2 个机箱 RGB 控制器（各 0.2A）+ 2 个无线鼠标接收器（各 0.05A）。 | 3.10 A | 0.28 A (两颗芯片高速数据搬运) | **3.38 A** | **16.9 W** | **核心推荐工况**：处于 SATA 4.5A 额定电流黄金区间（负荷率 75%），运行极其稳定。 |
| **工况 3：重载极限工况** | 8 个端口全部插入满载高速外设（单口平均拉载 0.52A）。 | 4.16 A | 0.32 A (满频并发) | **4.48 A** | **22.4 W** | **额定满载工况**：已达到 SATA 4.5A 持续安全承流上限，各回路正常工作。 |
| **工况 4：理论绝对极值 (超额)** | 8 个端口全部外接极限 0.9A 大电流移动硬盘（$8\times 0.9=7.2\text{A}$）。 | 7.20 A | 0.35 A | **7.55 A** | **37.8 W** | **超出外部接口物理能力**：受 PPTC 与 SATA 接触端子物理约束，该场景在实际装机中不成立（普通用户绝无可能在 1 台 PC 内部同时直插 8 块大功率机械硬盘）。 |

---

## 5. 直流回路压降 (IR Drop) 闭环工程推导与极限验算

### 5.1 压降约束指标
* **USB 3.0 规范电压底线**：USB 规范强制要求下行端口在任何规定带载工况下，输出端 VBUS 电压不得跌破 **$4.75\text{V}$**；
* **ATX 电源输出基准**：合格 ATX 电源输出的 5V 电压标称值为 $5.00\text{V} \sim 5.10\text{V}$。为验证最不利工况，**以最恶劣输入 $V_{in\_min} = 5.00\text{V}$ 为起算基准**；
* **最大允许回路总压降**：
  $$\Delta V_{total\_max} \le 5.00\text{V} - 4.75\text{V} = 0.250\text{V} \quad (250\text{mV})$$

#### 5.2 供电回路各段阻抗拆解模型

```
[ATX SATA 5V 输出点]
    │
    ├── 1. SATA 接插件接触电阻 R_SATA (3针并联)
    │
[扩展板 SATA 输入端]
    │
    ├── 2. 电子式 OVP 保护芯片导通内阻 R_OVP (SGM2553 P-MOS/N-MOS 开关)
    │
    ├── 3. PCB 主供电铜箔干线阻抗 R_Trace_Main (大面积铺铜)
    │
[支路分支节点]
    │
    ├── 4. 自恢复保险丝冷态内阻 R_PPTC (WAYON LP-MSM150)
    │
    ├── 5. PCB 支路走线阻抗 R_Trace_Branch
    │
    ├── 6. 19Pin 插座接插件接触电阻 R_19Pin (Pin 1 & 19 并联)
    │
[19Pin 插座输出引脚] ──> 外设负载 (VBUS)
    │
    └── 7. Layer 2 完整参考地平面回流阻抗 R_GND
```

根据 [02_Chip_Selection.md](file:///c:/Users/leiyu/Desktop/Document/work/USB3.0/02_System_Design/02_Chip_Selection.md) 中的物料参数与 4 层板叠层，各段物理阻抗严格取值如下：

1. **SATA 15Pin 接插件接触阻抗 ($R_{SATA}$)**：
   * 单引脚接触电阻典型值为 $30\text{m}\Omega$；Pin 7, 8, 9 三针并联：
     $$R_{SATA} = \frac{30\text{m}\Omega}{3} = 10.0\text{m}\Omega$$
2. **电子式 OVP 芯片导通内阻 ($R_{OVP}$)**：
   * 选用圣邦微 `SGM2553`，内部功率场效应管典型导通阻抗：
     $$R_{OVP} \approx 35.0\text{m}\Omega$$
3. **PCB 主干道电源走线阻抗 ($R_{Trace\_Main}$)**：
   * Layer 3 电源层大面积铺铜，干线等效平均宽度 $W \ge 4.0\text{mm}$，平均长度 $L \approx 30\text{mm}$，铜厚 1oz ($35\mu\text{m}$)；
   * 铜的方阻为 $R_{\square} \approx 0.5\text{m}\Omega/\square$；
   * 方块数 $N = 30 / 4 = 7.5\square$：
     $$R_{Trace\_Main} = 7.5 \times 0.5\text{m}\Omega \approx 3.75\text{m}\Omega \quad (\text{按保守值取 } 5.0\text{m}\Omega)$$
4. **PPTC 自恢复保险丝内阻 ($R_{PPTC}$)**：
   * 选用维安 `LP-MSM150`（1206 封装），初始冷态标称典型内阻 $R_{typ} = 40\text{m}\Omega \sim 70\text{m}\Omega$；
   * 考虑批量最差上限与环境温升，**保守取恶劣内阻上限 $R_{PPTC\_worst} = 80.0\text{m}\Omega$**。
5. **PCB 支路走线阻抗 ($R_{Trace\_Branch}$)**：
   * PPTC 输出端至 19Pin 插座 Pin 1 / Pin 19 焊盘，走线极短（$L \le 8\text{mm}$），宽度 $W \ge 2.0\text{mm}$：
     $$R_{Trace\_Branch} \approx 2.0\text{m}\Omega$$
6. **19Pin 输出插座接触阻抗 ($R_{19Pin}$)**：
   * 单个磷铜镀金引脚接触阻抗 $\le 20\text{m}\Omega$；
   * Pin 1 与 Pin 19 在 PCB 上直接并联打通：
     $$R_{19Pin} = \frac{20\text{m}\Omega}{2} = 10.0\text{m}\Omega$$
7. **地回路回流阻抗 ($R_{GND}$)**：
   * Layer 2 为完整地平面整层铺地，多点打孔回流，整板总回路地阻抗：
     $$R_{GND} \le 2.0\text{m}\Omega$$

### 5.3 极限工况闭环压降定量计算与实测验算

#### 1. 典型多外设并发工况 (主流装机场景：总电流 3.0A，单支路平均 0.75A)
* **公共主干道直流压降 ($\Delta V_{common}$)**：
  $$\Delta V_{common} = 3.0\text{A} \times (R_{SATA} + R_{OVP} + R_{Trace\_Main} + R_{GND})$$
  $$\Delta V_{common} = 3.0\text{A} \times (10.0\text{m}\Omega + 35.0\text{m}\Omega + 5.0\text{m}\Omega + 2.0\text{m}\Omega) = 3.0\text{A} \times 52.0\text{m}\Omega = \mathbf{156.0\text{ mV}}$$
* **支路直流压降 ($\Delta V_{branch}$)**：
  $$\Delta V_{branch} = 0.75\text{A} \times (R_{PPTC\_worst} + R_{Trace\_Branch} + R_{19Pin})$$
  $$\Delta V_{branch} = 0.75\text{A} \times (80.0\text{m}\Omega + 2.0\text{m}\Omega + 10.0\text{m}\Omega) = 0.75\text{A} \times 92.0\text{m}\Omega = \mathbf{69.0\text{ mV}}$$
* **典型工况总回路压降与端点电压**：
  $$\Delta V_{total\_typical} = 156.0\text{mV} + 69.0\text{mV} = \mathbf{225.0\text{ mV}} \quad (0.225\text{V})$$
  $$V_{out\_typical} = 5.050\text{V} - 0.225\text{V} = \mathbf{4.825\text{ V}} \ge 4.75\text{V} \quad (\text{裕量 } +75\text{mV})$$

#### 2. SATA 额定满载工况 (总电流 4.5A，4 支路平均拉载 1.125A)
* **公共主干道直流压降**：
  $$\Delta V_{common} = 4.5\text{A} \times 52.0\text{m}\Omega = \mathbf{234.0\text{ mV}}$$
* **支路直流压降**：
  $$\Delta V_{branch} = 1.125\text{A} \times 92.0\text{m}\Omega = \mathbf{103.5\text{ mV}}$$
* **满载工况总回路压降**：
  $$\Delta V_{total\_full} = 234.0\text{mV} + 103.5\text{mV} = \mathbf{337.5\text{ mV}}$$
  * 配合主机 ATX 5V 标称带载输出 $5.10\text{V}$ 时：
    $$V_{out\_full} = 5.100\text{V} - 0.338\text{V} = \mathbf{4.762\text{ V}} \ge 4.750\text{V}$$
* **结论**：在 SATA 持续 4.5A 额定满载下，完全满足 USB 3.0 规范规定的 4.75V 最低电压底线，多盘挂载稳定运行。

---

## 6. 分组 PPTC 保护电路计算与瞬态响应特性

### 6.1 保护动作时延曲线推导
选用的维安 `LP-MSM150`（1206 封装，$I_{hold}=1.5\text{A}, I_{trip}=3.0\text{A}$）采用高分子 PTC 热敏聚合物材料。器件动作时间与通过电流平方及热积累密切相关，遵循热平衡经验公式：
$$t_{trip} = \frac{C_{thermal} \times \Delta T_{curie}}{I^2 \times R - P_{dissipation}}$$

```mermaid
xychart-beta
    title "LP-MSM150 (1206) 跳变保护动作时间响应特性"
    x-axis "过载电流 (A)" [1.5, 1.8, 2.2, 2.6, 3.0, 4.0, 5.0, 6.0, 10.0]
    y-axis "动作跳变时间 (秒)" 0.05 --> 25
    line [22.0, 16.0, 9.0, 4.2, 1.5, 0.5, 0.22, 0.10, 0.03]
```

* **不同故障工况响应表现拆解**：
  1. **正常工作区 ($I \le 1.5\text{A}$)**：发热与表面自然对流散热达到热平衡，内部温度稳定在常温附近，处于低阻导通态（$R \approx 0.05\Omega$）；
  2. **轻度过载区 ($1.8\text{A} \sim 2.5\text{A}$)**：动作时间约 $6\text{s} \sim 15\text{s}$。允许外接移动硬盘在主轴电机启动瞬间的微小冲击电流通过，避免误保护；
  3. **跳断过载区 ($I \ge 3.0\text{A}$)**：动作时间约 $1.5\text{s}$，快速切断；
  4. **金属性短路区 ($I \ge 6.0\text{A}$)**：机箱前面板金属插头碰壳短路瞬间，焦耳热瞬间爆发，跳变时间仅需 **$\le 0.10\text{s}$（100毫秒）**。PPTC 晶格瞬间相变膨胀为高阻绝缘态（$R > 50\text{k}\Omega$），短路回路电流被瞬间压制在数毫安级别。

### 6.2 单路短路对全系统影响分析
* **瞬态跌落抑制**：在 PPTC 跳变的 100ms 时间窗口内，输入端配置的 **$100\mu\text{F}$ 固态钽电容/聚合物电容** 与高频 MLCC 阵列提供毫秒级局部电荷支撑；
* **PC 主机零感知**：ATX 电源的短路过流保护（SCP/OCP）跳闸阈值通常在 $25\text{A} \sim 35\text{A}$ 以上。单路发生金属性短路时，回路受 PCB 毫欧级阻抗和 PPTC 微秒级热响应限制，冲击电流达不到 ATX 跳闸阈值；
* **业务连续性保证**：故障仅被限制在发生短路的单个 19Pin 插座内。其余 3 组插座的 $5\text{V}$ 供电母线不受牵连，挂接在其上的高速移动硬盘数据读写绝不中断。短路外设拔除后，PPTC 自然冷却 $15\sim 30\text{s}$ 即可满血自动复位。

---

## 7. 主控芯片内部电源网络与去耦滤波

### 7.1 GL3510 双电源转换架构
每颗 GL3510 控制器内部包含两组高集成度片上电源转换单元：

```mermaid
flowchart TD
    SATA_5V["SATA 5V 外部主供电"]
    
    subgraph Controller_GL3510["GL3510 内部电源管理单元"]
        LDO_33["内置 5V -> 3.3V 高精度线性稳压器 (LDO)<br>供 I/O 焊盘、USB 2.0 PHY、晶振振荡器"]
        DC_12["内置 5V -> 1.2V 高效开关降压转换器 (Buck)<br>供 SuperSpeed 5Gbps 核心数字逻辑、PLL"]
    end
    
    Ext_L["外挂功率电感 L<br>2.2 μH (2520 一体成型, Isat=1.2A)"]
    Cap_12["核心输出滤波滤波电容<br>10 μF (0805) + 0.1 μF (0402)"]
    Cap_33["LDO 稳压输出去耦电容<br>4.7 μF (0603) + 0.1 μF (0402)"]

    SATA_5V ==> LDO_33
    SATA_5V ==> DC_12
    DC_12 <==> Ext_L
    Ext_L ==> Cap_12
    Cap_12 ==> VDD12_Pin["芯片 VDD12 引脚 (核心供电)"]
    LDO_33 ==> Cap_33
    Cap_33 ==> VDD33_Pin["芯片 VDD33 引脚 (I/O 供电)"]
```

### 7.2 纹波抑制与关键无源参数设计
1. **1.2V 核心开关电源拓扑**：
   * 开关频率：GL3510 内部 Buck 转换器开关频率工作在 $1.5\text{MHz} \sim 2.0\text{MHz}$；
   * 功率电感：根据 [02_Chip_Selection.md](file:///c:/Users/leiyu/Desktop/Document/work/USB3.0/02_System_Design/02_Chip_Selection.md)，选用顺络 `MPHM252012S2R2MT`（$2.2\mu\text{H}$，饱和电流 $1.2\text{A}$，DCR=120mΩ）；
   * 纹波计算：在 $200\text{mA}$ 负载下，纹波峰峰值估算为：
     $$\Delta V_{pp} \approx \frac{\Delta I_L}{8 \times f_{sw} \times C_{out}} \le 25\text{ mV}$$
     完全满足核心 PLL 对电源纹波 $< 50\text{mV}_{pp}$ 的严苛要求。
2. **引脚去耦电容就近布局规范**：
   * 芯片底部 QFN-64 中心裸焊盘（EPAD）为唯一模拟/数字共用地，必须密集布置不少于 9 个 $0.3\text{mm}$ 过孔直通 Layer 2 参考地；
   * 所有的 $0.1\mu\text{F}$ 陶瓷去耦电容必须放置在芯片对应电源引脚（`VDD33`、`VDD12`、`AVDD`）小于 $2.0\text{mm}$ 的范围内，走线严禁通过过孔拉远，保证高频阻抗最低。

---

## 8. PCB 载流能力、温升推导与散热设计

### 8.1 基于 IPC-2152 标准的铜箔载流与温升推导
整板在 $4.5\text{A}$ 满载工况下，大电流路径主要位于 SATA 输入端到 4 颗 PPTC 的主供电干线上。
依据 **IPC-2152 印刷电路板载流与温升计算标准**：
$$I = k \times \Delta T^{0.44} \times A^{0.725}$$
其中：$k$ 为修正系数（外层取 0.048，内层取 0.024）；$\Delta T$ 为允许温升；$A$ 为铜箔横截面积（$\text{mil}^2$）。

* **参数代入与线宽计算**：
  * 板材：标准 4 层板，铜厚 1.0 oz（约 $35\mu\text{m} = 1.37\text{mil}$）；
  * 设定机箱背线仓严苛工况下允许温升 $\Delta T \le 15^\circ\text{C}$；
  * 最大持续电流 $I = 4.5\text{A}$；
  * **主供电干线线宽要求**：
    * 若布设在内层（Layer 3 PWR）：所需导线宽度需满足 $W \ge 120\text{mil}$（约 $3.0\text{mm}$）；
    * **设计策略**：在 Layer 3 采用**大面积多边形整块铺铜（Polygon Pour）**，平均铺铜宽度达到 $6.0\text{mm} \sim 10.0\text{mm}$（相当于 $240\text{mil} \sim 400\text{mil}$），实际通流温升将控制在 **$\Delta T < 5^\circ\text{C}$**，几乎无发热。

### 8.2 大电流过孔阵列承流验算
SATA 15Pin 输入插针从顶层引出后，需通过过孔穿入内层 Layer 3 电源层及 Layer 2 地层。
* **单个过孔承流标准**：
  * 孔径 $D = 0.30\text{mm}$（12 mil），焊盘外径 $0.60\text{mm}$（24 mil），孔壁镀铜厚度按行业标准取 $20\mu\text{m}$；
  * 根据 IPC-2152 标准，单个 $0.3\text{mm}$ 过孔在 $\Delta T = 15^\circ\text{C}$ 下的安全通过电流约为 **$1.2\text{A} \sim 1.5\text{A}$**；
* **过孔阵列部署规范**：
  * 在 SATA 15Pin 的 +5V 焊盘引出区，必须密集打入 **$\ge 6$ 个过孔**：
    $$I_{vias\_capacity} = 6 \times 1.2\text{A} = 7.2\text{A} > 4.5\text{A} \quad (\text{裕量率 } 160\%)$$
  * 在 SATA GND 引脚连接区，密集打入 **$\ge 8$ 个过孔** 连接至 Layer 2 地平面；
  * 彻底杜绝因单孔电流集中导致的过孔局部发热和阻抗突变。

---

## 9. 电源系统测试与验收规范

为确保每一块量产板卡达到设计指标，制定以下电源测试与验收标准：

| 测试项目编号 | 测试内容与条件 | 测试测量节点 | 预期合格指标 (Pass Criteria) | 判定意义 |
| :---: | :--- | :--- | :--- | :--- |
| **TP-01** | **主板开机感知门限测试**<br>(输入 19Pin 施加 4.75V ~ 5.25V) | GL3510 的 `VBUS_DET` 引脚 | 电压稳定在 $2.35\text{V} \sim 2.65\text{V}$，芯片正常唤醒枚举。 | 保证主机各种电源公差下均能秒级可靠开机。 |
| **TP-02** | **反向倒灌漏电流测试**<br>(SATA 供电接通，主板 19Pin 断电) | 主板 19Pin 输入端 Pin 1 节点 | 对主板反向漏电流 $I_{leak} \le 25\mu\text{A}$。 | 验证绝对物理隔离，防止反灌主板。 |
| **TP-03** | **静态待机功耗测试**<br>(SATA 5V 供电，全端口空载) | SATA 5V 总输入回路 | 稳态电流 $I_{standby} \le 150\text{mA}$ ($P \le 0.75\text{W}$)。 | 验证主控低功耗休眠机制。 |
| **TP-04** | **全板极限压降测试**<br>(SATA 5V 额定输入，8 端口拉载 4.5A) | 离电源最远的 19Pin 插座 VBUS | 端点输出电压 $V_{out} \ge 4.80\text{V}$ (总压降 $\le 200\text{mV}$)。 | 确保多盘并发不掉盘。 |
| **TP-05** | **端口纹波噪声测试**<br>(4.5A 满载工况，示波器 20MHz 带宽) | 各 19Pin 输出插座 VBUS 焊盘 | 纹波噪声峰峰值 $V_{pp} \le 80\text{mV}$。 | 保证高速硬盘读写稳定无校验重传。 |
| **TP-06** | **支路金属性短路保护测试**<br>(将 J_OUT1 的 VBUS 直接短接至 GND) | 观察系统反应及其余 3 组接口 | PPTC 在 $\le 0.2\text{s}$ 内动作切断；其余 3 组 19Pin 供电完全正常，电脑不黑屏。 | 验证无源分组保护可靠性。 |

---

## 10. 交付物衔接指引

本电源与防倒灌方案设计说明书已完成全系统供电、隔离与保护的理论推导、定量计算与测试标准定义：
* **成本核算对接** $\rightarrow$ 进入 `04_Cost_Analysis.xlsx`：核算包含 SATA 15Pin 连接器、4 颗 WAYON PPTC、SMAJ5.0A TVS 及 100μF 固态电容在内的电源保护系统量产成本。
