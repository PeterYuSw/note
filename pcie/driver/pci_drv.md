# PCI driver
#### basic
1.bus
- electrical interface
- 编程接口

2.PCI设计目标
- 提高cpu和外设之间数据传输速度
    - clock rate提高
    - 支持64bit数据总线
- 平台无关
- 便于添加/删除设备
    - PCI设备在boot时自动被配置 --> **driver不需要probe就可以访问PCI device**

3.PCI bridge
- 特殊的PCI设备
- 连接两个PCI bus
- 系统里插入多个PCI bus需要bridge

4.PCI layout
- 树状结构
- bus 0在根节点
- 每个bus都和uplayer bus相连

5.地址空间
- memory location
- IO port
    - **前两个地址空间被相同bus的PCI设备共享**
    - 通过```readb/inb```指令访问

- configuration space
    - 需要访问config reg来配置

#### 引导阶段
1.