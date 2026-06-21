# QOM

## basic
- ObjectClass
    - The base for all classes

```C
static const TypeInfo object_info = {
        .name = TYPE_OBJECT,
        .instance_size = sizeof(Object),
        .class_init = object_class_init,
        .abstract = true,
};

- 对象构造
    - 类型的构造
    - 类型的初始化
    - 类对象的构造
        - ```-device```
    - realized
```

## class
- declartion
```C
static const TypeInfo edu_info ={
    .name          = TYPE_PCI_EDU_DEVICE,
    .parent        = TYPE_PCI_DEVICE,
    .instance_size = sizeof(EduState),
    .instance_init = edu_instance_init,
    .class_init    = edu_class_init,
    .interfaces = interfaces,
};
```

- cast

- init
    - qemu初始化的时候就会初始化类，类的实例化是通过参数传进来的
```C
type_initialize(TypeImpl *ti) // qom/object.c:337
```

## instance
- 通过命令行传入```-device```
- ```XXXState```

- init
```C
device_init_func
    - qdev_device_add
    - object_new
    - object_new_with_type
    - object_initialize_with_type
    - object_init_with_type
        - if (ti->instance_init) {
            ti->instance_init(obj);
        }
```

## 属性


## type_init

```C
// allocate a ModuleEntry and add to MODULE_INIT_QOM list
type_init(pci_edu_register_types)
    - module_init(pci_edu_register_types, MODULE_INIT_QOM)
        - register_module_init
            - e->init = fn;
            - e->type = type;
            - l = find_type(type);
            - QTAILQ_INSERT_TAIL(l, e, node);
                - init_type_list[MODULE_INIT_QOM]
```

```C
// util/module.c:97
module_call_init()
{
    l = find_type(type);

    // call init function of every entry in init_type_list[MODULE_INIT_QOM]
    QTAILQ_FOREACH(e, l, node) {
        e->init();
    }
}
```
