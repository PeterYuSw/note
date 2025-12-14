# GPIO driver

## basic

## sysfs
- /sys/class/gpio/gpio768
	- 显示注册的gpio controller
	- base: 控制器管理的起始编号
	- label: 控制器标签
	- ngpio: 管理的gpio引脚总数
- 通过/sys/class/gpio操作GPIO的sysfs接口，在较新的Linux内核中已被去掉

- 操作控制器下的某个gpio引脚
```bash
echo N > /sys/class/gpio/export
# 生成gpioN目录
cd gpioN
echo out > /sys/class/gpio/gpioN/direction # 设置为输出
# 写任何非0值都会输出高电平
echo 1 > /sys/class/gpio/gpioN/value # 输出高电平
echo 0 > /sys/class/gpio/gpioN/value # 输出低电平

echo in > /sys/class/gpio/gpioN/direction # 设置为输入
cat /sys/class/gpio/gpioN/value # 读取当前电平

# 取消导出
echo N > /sys/class/gpio/unexport
```

## libgpiod
- example
```C
#include <gpiod.h>
#include <stdio.h>

int main() {
	// 打开 GPIO 芯片
	struct gpiod_chip *chip = gpiod_chip_open("/dev/gpiochip0");

	// 获取 GPIO 线
	struct gpiod_line *line = gpiod_chip_get_line(chip, 18);

	// 设置为输出
	gpiod_line_request_output(line, "example", 0);

	// 控制输出
	gpiod_line_set_value(line, 1); // 设置高电平
	gpiod_line_set_value(line, 0); // 设置低电平

	// 清理
	gpiod_line_release(line);
	gpiod_chip_close(chip);
	return 0;
}
```

- 命令行工具
```bash
# 安装 libgpiod 工具
sudo apt install gpiod

# 查看 GPIO 信息
gpiodetect # 检测所有 GPIO 控制器
gpioinfo # 查看 GPIO 状态

# 控制 GPIO
gpioset gpiochip0 18=1 # 设置高电平
gpioget gpiochip0 18 # 读取电平值
```
