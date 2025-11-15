# char dev

## driver框架
1.步骤
- 分配设备号
- 注册字符设备
- 注册file operation

### 分配设备号
1.设备号定义
```C
#include <linux/types.h>
dev_t dev_id;  // 设备号定义

#include <linux/kdev_t.h>
MAJOR(dev_t dev);  // 获取主设备号
MINOR();  // 获取次设备号
MKDEV(int major, int minor);  // 创建设备号
```

2.分配/释放设备号
```C
#include <linux/fs.h>
/**
 * @param firstminor: 要使用的被请求的第一个次设备号，通常为0
 */
int alloc_chrdev_region(dev_t *dev, unsigned int firstminor, unsigned int count, char *name);

// 释放设备号
void unregister_chrdev_region(dev_t first, unsigned int count);
```
- 分配后可以在/proc/devices里读到

### 注册字符设备
```C
#include <linux/cdev.h>
struct cdev chrdev;

// 初始化cdev结构体
void cdev_init(struct cdev *cdev, struct file_operations *fops);
cdev->owner = THIS_MODULE;

// 注册chr dev
int cdev_add(struct cdev *cdev, dev_t num, unsigned int count);

// 删除chr dev
void cdev_del(struct cdev *cdev);
```

### 注册file operation
