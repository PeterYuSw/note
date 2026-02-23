# PA2

## 疑问
- build/*.txt里的汇编是怎么生成的
    - objdump elf

## AM
- 编译得到的可执行文件的行为
    - 第一条指令从abstract-machine/am/src/$ISA/nemu/start.S开始, 设置好栈顶之后就跳转到abstract-machine/am/src/platform/nemu/trm.c的_trm_init()函数处执行
    - 在_trm_init()中调用main()函数执行程序的主体功能, main()函数还带一个参数
    - 从main()函数返回后, 调用halt() -> nemu_trap结束运行
        - ```# define nemu_trap(code) asm volatile("mv a0, %0; ebreak" : :"r"(code))```
