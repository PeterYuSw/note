# PCIe

## basic
- 串行总线
    - 端到端连接
    - 每个port只能与一个设备相连
    - 只能通过switch扩展链路

- 每个lane有TX/RX两组差分信号
    - D+/D-为一组差分信号
    - 通过D+和D-的差值表示一个bit

- PCIe分层
    - core
    - transaction layer
        - 组TLP包
        - 可以乱序传输
        - 基于credit流控
    - data link layer
        - 组LLDP
        - 通过sequence num + crc保证传输正确性
        - 通过ack/nack保证不丢包
    - physical layer
        - LTSSM状态机管理链路状态，进行链路training

- 信号
    - PERST#: 复位信号
    - REFCLK#: 时钟
    - PRSNT1#: 用于热插拔的信号
        - 金手指半高
        - PCIe卡插拔这个信号会变 -> 系统软件执行初始化/去初始化操作

- reset
    - 传统复位
        - cold/warm/hot reset
        - 即使cold reset也无法reset Vaux供电的逻辑，只能依靠host完全上下电
        - 需要重新training链路
    - FLR
        - reset功能相关的逻辑，依赖设备实现
        - 设备的BAR空间需要提供寄存器来触发FLR
        - VM退出时会触发FLR
        - 不会影响LTSSM状态机

## extended config space
- 扩展配置空间
    - 64byte基本配置空间0x00~0x3F
        - PCI/PCIe设备都支持
    - **0x40~0xFF**扩展配置空间
        - 存放**MSI/MSI-X中断机制**和电源管理相关capability
    - **0x100~0xFFF**扩展配置空间
        - 存放PCIe设备独有的capability

- capability寄存器
    - 单向链表结构
    - capability pointer指向capability链表头
    - 每个capability有唯一id

- PCI express capability
    - 存放和PCIe总线相关的信息
    - cap id为0x10
    - 组成
        - device capability
            - 能力 -> 支持的max_payload_size
        - **device control**
            - max_payload_size(bit[7:5]) -> 设置
            - MRRS(bit[14:12])
            - enable no snoop(bit[11])
            - enable relaxed ordering(bit[4])
        - device status
            - 指示error状态
        - **link capability**
            - 描述链路属性
            - supported link speed
            - maximum link width -> x16
            - ASPM
        - link status
            - 当前链路状态
            - current link status[3:0]
            - negotiated link width[9:4]
        - link control
            - link disable -> 写1会禁止PCIe链路
        - slot capability
        - slot status