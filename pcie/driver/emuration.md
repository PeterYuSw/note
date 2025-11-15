# emuration

## device emu
- 初始化pci_dev->resource[]
```C
pci_scan_device()
	- pci_setup_device()
		- pci_read_bases() // 读取bar寄存器获取bar地址
			- __pci_read_base()
```
