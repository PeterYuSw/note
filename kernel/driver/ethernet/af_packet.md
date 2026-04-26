# AF_PACKET

## 1. basic
- packet protocol
	- The Packet protocol is used by applications which communicate directly with network devices without an intermediate network protocol implemented in the kernel, e.g. tcpdump.

## 2. af_packet
- register ```PF_PACKET``` family protocol handler
```C
struct list_head ptype_all __read_mostly;	/* Taps */

// global array of protocol handler
static const struct net_proto_family __rcu *net_families[NPROTO] __read_mostly;

static const struct proto_ops packet_ops = {
	.family =	PF_PACKET,
	.bind =		packet_bind,
	.connect =	sock_no_connect,
	.accept =	sock_no_accept,
	.poll =		packet_poll,
	.ioctl =	packet_ioctl,
	.listen =	sock_no_listen,
	.setsockopt =	packet_setsockopt,
	.getsockopt =	packet_getsockopt,
	.sendmsg =	packet_sendmsg,
	.recvmsg =	packet_recvmsg,
	.mmap =		packet_mmap,
	// ...
};

static const struct net_proto_family packet_family_ops = {
	.family =	PF_PACKET,
	.create =	packet_create,
	.owner	=	THIS_MODULE,
};

module_init(packet_init);
	/**
	*	sock_register - add a socket protocol handler
	*	@ops: description of protocol
	*
	*	This function is called by a protocol handler that wants to
	*	advertise its address family, and have it linked into the
	*	socket interface. The value ops->family corresponds to the
	*	socket system call protocol family.
	*/
	- sock_register(&packet_family_ops);
		- rcu_assign_pointer(net_families[ops->family], ops);
```

## 3. socket process
- socket()
```C
// userspace
socket(AF_PACKET)
	|
	v
// socket.c:1567
SYSCALL_DEFINE3(socket, int, family, int, type, int, protocol)
	- __sys_socket(family, type, protocol);
		- sock_create(family, type, protocol, &sock);
			- __sock_create
				- pf = rcu_dereference(net_families[family]);
					- sock = sock_alloc();
					- pf->create(net, sock, protocol, kern);
					- packet_create(struct net *net, struct socket *sock, int protocol int kern)
						- sock->ops = &packet_ops;
						- po->prot_hook.func = packet_rcv;
						- __register_prot_hook(sk); // add to ptype_all
							- dev_add_pack(&po->prot_hook);
								- struct list_head *head = ptype_head(pt);
								- list_add_rcu(&pt->list, head);
```

- bind
```C
// userspace
bind(raw_socket, (struct sockaddr *)&addr, sizeof(addr))
	|
	v
packet_bind(struct socket *sock, struct sockaddr *uaddr, int addr_len)
	- packet_do_bind
		- register_prot_hook(sk);
			- __register_prot_hook
				- dev_add_pack(&po->prot_hook);
					// add to dev->ptype_all
					- struct list_head *head = ptype_head(pt);
					- list_add_rcu(&pt->list, head);
```

## 4. tcpdump
```C
// 1. create AF_PACKET socket
socket_fd = socket(AF_PACKET, SOCK_RAW, htons(ETH_P_ALL));

// 2. bind to specific device
bind(raw_socket, (struct sockaddr *)&addr, sizeof(addr))

__netif_receive_skb_core {
    /*...*/
    list_for_each_entry_rcu(ptype, &ptype_all, list) {
		if (pt_prev)
			ret = deliver_skb(skb, pt_prev, orig_dev);
		pt_prev = ptype;
	}

	list_for_each_entry_rcu(ptype, &skb->dev->ptype_all, list) {
		if (pt_prev)
			ret = deliver_skb(skb, pt_prev, orig_dev);
		pt_prev = ptype;
	}
    /*...*/
}
```
