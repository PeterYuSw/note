# SPI

## basic
- signal
	- MISO
	- MOSI
	- SCLK
	- CS

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
