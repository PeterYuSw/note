# SPI

## basic
- 对比uart
	- uart只能依靠双方协商波特率来传输，容易误码
	- spi加了一个时钟信号，在时钟上升沿采样即可，不需要协商波特率了
	- uart速度慢，115200才11.25KB/s, spi传输速度快

- signal
	- MISO
	- MOSI
	- SCLK
	- CS

- mode
	- 0
		- clk低电平空闲，在clk上升沿取样
	- 1
	- 2
	- 3

- linux
	- CPU <--platform bus--> spi master <--spi bus--> spi slave

## kernel spi framework
- data structure
```C
// spi master device
struct spi_controller;

// spi slave device
struct spi_device;

// slave device driver
struct spi_driver;

// single operation
struct spi_transfer;
struct spi_message;
```

## userspace操作spi设备
- spidev
	- 无法收中断
	- 无法使用内核机制
