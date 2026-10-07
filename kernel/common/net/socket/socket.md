# socket

## 数据结构
1. ```struct socket```
```C
/**
 *  struct socket - general BSD socket
 *  @state: socket state (%SS_CONNECTED, etc)
 *  @type: socket type (%SOCK_STREAM, etc)
 *  @flags: socket flags (%SOCK_NOSPACE, etc)
 *  @ops: protocol specific socket operations
 *  @file: File back pointer for gc
 *  @sk: internal networking protocol agnostic socket representation
 *  @wq: wait queue for several uses
 */
struct socket {
	socket_state		state;

	short			type;

	unsigned long		flags;

	struct file		*file;
	struct sock		*sk;
	const struct proto_ops	*ops;

	struct socket_wq	wq;
};
```

2. ```struct sock```
```C

```

3. 协议族 
```C
struct net_proto_family {
	int		family;
	int		(*create)(struct net *net, struct socket *sock,
				  int protocol, int kern);
	struct module	*owner;
};

// af_inet.c:1138
// AF_INET协议族所有的协议的socket接口
static struct inet_protosw inetsw_array[] = {};
```

## 系统调用
```C
SYSCALL_DEFINE3(socket, int, family, int, type, int, protocol)
{
	return __sys_socket(family, type, protocol);
}

__sys_socket
- sock_create(family, type, protocol, &sock);
	- __sock_create // 最后一个参数用来区分socket属于userspace还是kernel space
		- sock = sock_alloc();
			- inode = new_inode_pseudo(sock_mnt->mnt_sb);
				- 调用sock_alloc_inode分配inode
					- // slab分配的就是struct socket_alloc
					- ei = kmem_cache_alloc(sock_inode_cachep, GFP_KERNEL);
			- sock = SOCKET_I(inode);
		- pf = rcu_dereference(net_families[family]); // 根据传入的family找到协议族
		- // 每个协议会调用int sock_register(const struct net_proto_family *ops)注册协议处理函数
		- err = pf->create(net, sock, protocol, kern); // 调用对应协议处理函数
			- inet_create(struct net *net, struct socket *sock, int protocol,
		       int kern) // af_inet.c
- return sock_map_fd(sock, flags & (O_CLOEXEC | O_NONBLOCK)); // 返回fd
```
 
