# Ethernet driver

## NAPI
- basic
	- 中断下半部，由top half通过napi_schedule触发


- 基本数据结构

```c
struct napi_struct {
	struct list_head poll_list; // 注册时添加到softirq的调度队列里
	unsigned long state;
	int weight;
	int (*poll)(struct napi_struct *, int); // poll
	struct net_device *dev;
	struct sk_buff *skb;
	struct task_struct *thread;
};

// 软中断号
enum {
	HI_SOFTIRQ=0,
	TIMER_SOFTIRQ,
	NET_TX_SOFTIRQ,
	NET_RX_SOFTIRQ, // napi收包软中断
	BLOCK_SOFTIRQ,
	IRQ_POLL_SOFTIRQ,
	TASKLET_SOFTIRQ, // tasklet
	SCHED_SOFTIRQ,
	HRTIMER_SOFTIRQ,
	RCU_SOFTIRQ,    /* Preferable RCU should always be the last softirq */ 
	NR_SOFTIRQS
};

// 每个CPU一个
struct softnet_data {
	struct list_head poll_list; // 维护本cpu注册的napi队列
	struct sk_buff_head	process_queue;
};

```

- 系统napi初始化
```C
// softirq thread
static struct smp_hotplug_thread softirq_threads = {
	.store			= &ksoftirqd,
	.thread_should_run	= ksoftirqd_should_run,
	.thread_fn		= run_ksoftirqd,
	.thread_comm		= "ksoftirqd/%u",
};

// 3. 初始化全局变量softnet_data，每个CPU一个
DECLARE_PER_CPU_ALIGNED(struct softnet_data, softnet_data);

// 4. net_dev_init注册softirq handle
// net/core/dev.c
open_softirq(NET_TX_SOFTIRQ, net_tx_action);
open_softirq(NET_RX_SOFTIRQ, net_rx_action);
	- softirq_vec[nr].action = action;

// 5. 驱动添加napi poll函数到softnet_data
napi_schedule
	- list_add_tail(&napi->poll_list, &sd->poll_list);

// 6. 设置CPU N的softirq_pending bit
or_softirq_pending(1UL << nr);

// 7. ksoftirqd/N线程将调用run_ksoftirqd，检查是否软中断的pending bit

// 8. 调用pending bit对应的软中断处理函数(net_rx_action)
run_ksoftirqd
	- __do_softirq
		- h->action(h);
			- net_rx_action

// 9. 调度napi poll函数
net_rx_action
	- budget -= napi_poll(n, &repoll);
```

- 驱动注册

```c
// 创建NAPI结构体
// 初始化成员变量
netif_napi_add(netdev, napi, poll_func, 64);
- 
```

- Schedule

```c
// 调度napi
napi_schedule(napi);
- __napi_schedule();
	- ____napi_schedule(this_cpu_ptr(&softnet_data), n);
		// 将napi添加到本cpu的poll list
		- list_add_tail(&napi->poll_list, &sd->poll_list);
		// 设置NET_RX_SOFTIRQ对应软中断pending bit
		- __raise_softirq_irqoff(NET_RX_SOFTIRQ);
			- trace_softirq_raise
			// 设置irq_stat.__softirq_pending pending bit
			- or_softirq_pending(1UL << nr);

```

## sk_buff

- field class
    - layout field
    - general field
    - feature-specific
    - management functions

- 定义

```c
struct sk_buff {
	// layout field
	struct sk_buff *next;
	struct sk_buff *prev;

	// socket owning this sk_buff
	struct sock *sock;

	u32 len; // buffer + fragments
	u32 data_len; // fragments
	u32 mac_len; // size of mac header

	atomic_t users; // refcount

	// |headroom|data|tailroom|
	char *head; // buffer start
	char *end; // buffer end
	char *data; // data start
	char *tail; // data end


	// general field


	// feature-specific field


	// management functions field

};
```

- kernel协议栈传给driver
    - 

- Driver传给kernel协议栈

## 拆包
- process
```C
ip_fragment
- ip_do_fragment

```

## 组包

## ref driver - intel
- basic
	- Intel 1G、10G、100G网卡分别对应的的驱动是igb、ixgbe、ice
