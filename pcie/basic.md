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
        - 
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
