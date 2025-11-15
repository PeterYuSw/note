# PCI driver

## basic
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

## 引导阶段
1.

## driver

### data structure
```C
struct pci_dev {
	/* I/O and memory regions + expansion ROMs */
	struct resource resource[DEVICE_COUNT_RESOURCE];
};
```

### init process
```C
probe(struct pci_dev *pdev, const struct pci_device_id *id)
	- alloc device // 分配自定义device结构
	- pci_set_drvdata // 将自定义device存入pdev
	- pci_enable_device
	- pci_request_regions // reserve PCI I/O and memory resources
		- pci_resource_start // 获取bar空间起始PA
		- pci_resource_len // 获取bar空间大小
		- ioremap // 映射到kernel VA space
	- pci_set_master
	- pci_set_dma_mask
```

- 为PCI设备分配资源
	- pci_request_regions
	- **负责声明并保留PCI设备的所有BAR资源，防止其他驱动或模块意外占用相同的资源**
	- I/O 端口资源: 在```/proc/ioports```中注册
	- 内存映射资源: 在```/proc/iomem```中注册
	- 防冲突: 如果资源已被占用，函数返回-EBUSY

```C
pci_request_regions(struct pci_dev *pdev, const char *res_name)
	- pci_request_selected_regions((1 << PCI_STD_NUM_BARS) - 1)
		// If @exclusive is set, then the region is marked so that userspace is explicitly not allowed to map the resource via /dev/mem or sysfs MMIO access.
		- __pci_request_region(struct pci_dev *pdev, int bar,
				const char *res_name, int exclusive)
			- request_region // IORESOURCE_IO
			- __request_mem_region // IORESOURCE_MEM
```
