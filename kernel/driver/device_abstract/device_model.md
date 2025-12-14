# device model

## basic
1.统一设备模型的**需求**
- 在shutdown之前遍历系统硬件
- 通过sysfs提供接口给user space
- 热插拔设备的管理
- 提供device class
- 管理device生命周期

2.device model
- bus：总线上连接了什么
- device：设备怎么连接到系统
- class：设备提供了什么功能

3.kobject作用
- 引用计数：标记生命周期
- sysfs node：sysfs中显示的每个对象都对应一个kobject
- hotplug event handling

## kobject
1.初始化步骤
- ```memset(0)```
- ```kobject_init```
    - 将ref count置为1
- ```kobject_set_name```
    - 设置的名字就是sysfs中的名字

2.引用计数管理
```C
// 引用计数加1
kobject_get(struct kobject *kobj);、
// 引用计数减1
kobject_put(struct kobject *kobj);
```
- 通过ref count判断kobject是否能被释放
- 初始化时会将ref cnt置为1，释放时需要调用```kobject_put```
- **user space可以持有对kobj的引用计数**
    - **打开对应的sysfs文件**

## hotplug event
1. 产生：kobject创建/删除

## bus/device/driver
1. bus
```C
struct bus_type {
    const char      *name;
    const char      *dev_name;
    struct device       *dev_root;
    const struct attribute_group **bus_groups;
    const struct attribute_group **dev_groups;
    const struct attribute_group **drv_groups;
    int (*match)(struct device *dev, struct device_driver *drv);
    int (*uevent)(struct device *dev, struct kobj_uevent_env *env);
    int (*probe)(struct device *dev);
};

bus_register();
bus_unregister();

```

2. device
```C
struct device {
    struct device *parent;  // 通常指向总线
    struct kobject kobj;
    struct bus_type *bus;
    struct device_driver *driver;
    void *driver_data;
    void (*release)(struct device *dev);
};

struct device_attribute {
    struct attribute    attr;
    ssize_t (*show)(struct device *dev, struct device_attribute *attr,
            char *buf);
    ssize_t (*store)(struct device *dev, struct device_attribute *attr,
             const char *buf, size_t count);
};
```
- 注册device: ```deivce_register```
- 增加设备属性：```device_create_file```

3. driver
```C
struct device_driver {
    const char      *name;
    struct bus_type     *bus;
    struct module       *owner;
    const char      *mod_name;  /* used for built-in modules */
    bool suppress_bind_attrs;   /* disables bind/unbind via sysfs */
    enum probe_type probe_type;
    const struct of_device_id   *of_match_table;
    const struct acpi_device_id *acpi_match_table;
    int (*probe) (struct device *dev);
    void (*sync_state)(struct device *dev);
    int (*remove) (struct device *dev);
    };
```
W