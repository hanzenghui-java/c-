graph TD
    %% 样式定义
    classDef bus fill:#E1F5FE,stroke:#0288D1,stroke-width:2px;
    classDef prot fill:#FFF3E0,stroke:#F57C00,stroke-width:2px;
    classDef power fill:#E8F5E9,stroke:#388E3C,stroke-width:2px;
    classDef ldo fill:#F3E5F5,stroke:#7B1FA2,stroke-width:2px;
    classDef rail fill:#ECEFF1,stroke:#455A64,stroke-width:2px;
    classDef load fill:#FAFAFA,stroke:#616161,stroke-width:1px,stroke-dasharray: 5 5;
    classDef note fill:#FFFDE7,stroke:#FBC02D,stroke-width:1px;

    %% 节点定义
    BUS[24V 母线输入]:::bus
    L3[L3 共模扼流圈]:::prot
    D41[D41 TVS<br/>SMBJ28A]:::prot
    F2[F2 自恢复/保险丝<br/>2.2A]:::prot

    subgraph Power_Stage ["辅助供电转换系统 (3 轨)"]
        direction TB
        subgraph Branch_A ["支路 A: 标准降压 (Buck)"]
            U29["U29: TPS5430DDAR<br/>f_sw = 500 kHz<br/>L2 = 68 µH (原22µH优化)"]:::power
        end

        subgraph Branch_B ["支路 B: 反相降压-升压 (Inverting Buck-Boost)"]
            U30["U30: TPS5430DDAR<br/>f_sw = 500 kHz<br/>L4 = 22 µH"]:::power
        end

        REG_LDO["U3 线性稳压器<br/>MC7805 (标准三端稳压)"]:::ldo
    end

    %% 电源轨输出
    R_15V["+15V 电源轨<br/>(35 ~ 60 mA)"]:::rail
    R_5V["+5V 电源轨<br/>(控制与逻辑供电)"]:::rail
    R_N5V["-5V 电源轨 (V_EE)<br/>(1 ~ 2 mA)"]:::rail

    %% 负载侧
    LOAD_ISO["14 × 光耦 VDD 侧<br/>• I_DD = 1.8~2.4 mA/只<br/>• 合计 25~34 mA<br/>• ⚠️ 每只并联 0.1~1 µF 退耦"]:::load
    LOAD_CTRL["控制与信号电路<br/>• MCU 核心及外设 / 采样电路<br/>• 差分总线驱动器<br/>• 状态指示灯 (2.6 mA)"]:::load
    LOAD_GATE["栅极驱动负压偏置<br/>• IGBT 关断安全电平 (V_EE)<br/>• 抑制 dv/dt 寄生导通"]:::load

    %% 独立输入信号负载说明
    LOAD_LED["14 × 光耦 LED 输入侧 (发射极)<br/>• 6 mA/只 × 14 = 84 mA<br/>• 🔑 由【差分信号驱动器】驱动<br/>• ❌ 不直接取自主电源轨"]:::note

    %% 拓扑连接关系
    BUS --> L3
    L3 --> D41
    D41 --> F2
    F2 ==>|VIN| U29
    F2 ==>|VIN| U30

    U29 -->|输出| R_15V
    U30 -->|输出| R_N5V

    R_15V -->|主输出负载| LOAD_ISO
    R_15V -->|降压输入| REG_LDO
    REG_LDO -->|稳压输出| R_5V

    R_5V --> LOAD_CTRL
    R_N5V --> LOAD_GATE

    LOAD_CTRL -.->|逻辑控制/信号驱动| LOAD_LED
    LOAD_LED -.->|光电隔离传输| LOAD_ISO
