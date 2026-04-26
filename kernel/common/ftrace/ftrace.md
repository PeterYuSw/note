# ftrace

## 1. usage
```bash
cd /sys/kernel/tracing

# 1. 设置追踪器：使用function追踪器，它记录函数调用
echo function > current_tracer
# 使用 function_graph 追踪器查看单次调用耗时
echo function_graph > current_tracer

# 2. 设置过滤函数：只追踪目标函数
echo kmem_cache_alloc > set_ftrace_filter

# 3. 开启栈跟踪
echo func_stack_trace > trace_options

# 4. 开启追踪
echo 1 > tracing_on

# 5. 结束并查看
echo 0 > tracing_on
cat trace
```

- trace options

|选项             |作用                         |使用场景   |
|----------------|-----------------------------|---------|
|func_stack_trace|每次函数被调用时打印内核栈       |追踪特定函数调用源|
|stacktrace      |每次跟踪点(Event)触发时打印内核栈|追踪特定内核事件|
|sym-offset      |显示函数名及偏移量              |需要知道具体代码行位置时|
|sym-addr        |显示函数的实际内存地址           |底层调试或反汇编分析|
|verbose         |打印更详细的原始数据             |调试ftrace自身问题|

## 
