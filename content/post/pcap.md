---
title: "libpcap代码剖析"
date: 2017-06-24T19:56:58+08:00
draft: false
---
<div id="table-of-contents">
<h2>Table of Contents</h2>
<div id="text-table-of-contents">
<ul>
<li><a href="#sec-1">1. 简介</a></li>
<li><a href="#sec-2">2. 硬件设备的发现</a></li>
<li><a href="#sec-3">3. 打开接口</a>
<ul>
<li><a href="#sec-3-1">3.1. <code>pcap_create</code></a></li>
<li><a href="#sec-3-2">3.2. <code>pcap_activate</code></a>
<ul>
<li><a href="#sec-3-2-1">3.2.1. 获取link层协议</a></li>
</ul>
</li>
<li><a href="#sec-3-3">3.3. 注意点:</a></li>
</ul>
</li>
</ul>
</div>
</div>


# 简介<a id="sec-1" name="sec-1"></a>

`libpcap` 是抓包不可或缺的一个跨平台库，本篇文档试图剖析 `libpcap`  Linux平台上核心API的实现细节。

使用 `libpcap` 的主要步骤如下：

1.  找到(或直接指定)接口(find)
2.  打开此接口(setup)
3.  启动，开始(start)

关注的主要问题有：

-   硬件设备的发现
-   网络数据的获取
-   包格式
-   filter的设置机制

# 硬件设备的发现<a id="sec-2" name="sec-2"></a>

主要通过这个接口实现：

    pcap_if_t *pcap_findalldevs()

内部的实现细节是通过调用 `getifaddrs` C API来获得本地网卡地址信息，由于一块网卡可能含多个地址，所以在处理时有去考虑这样的情况。可以查看 `pcap_findalldevs` 接口调用的static函数 `add_or_find_if` 了解具体实现细节。

接口的返回值指针类型是这样的一个结构体：

    struct pcap_if {
      struct pcap_if *next; # 通过next来将系统所有网络设备串在单链表上。
      char *name;
      char *description;
      struct pcap_addr *addresses;
      bpf_u_int32 flags;
    }

主要是在地址之上再包装些描述信息。 `addresses` 的类型定义如下：

    struct pcap_addr {
      struct pcap_addr *next;
      struct sockaddr *addr;
      struct sockaddr *netmask;
      struct sockaddr *broadaddr;
      struct sockaddr *dstaddr;
    }

`tcpdump` 的实现中当不指定网卡时调用libpcap的 `pcap_lookdev` 接口，此接口就是获得 `pcap_findalldevs` 返回的单链表第一个设备的name属性。

# 打开接口<a id="sec-3" name="sec-3"></a>

此接口名称: `pcap_open_live` 此函数的调用栈如下:

    .
    └── pcap_open_live
      ├── pcap_create
      │   └── pcapp_create_common
      ├── pcap_set_snaplen
      ├── pcap_set_promisc
      └── pcap_set_timeout
      ├── pcap_activate

## `pcap_create`<a id="sec-3-1" name="sec-3-1"></a>

此函数主要是初始化 `pcap_t` 核心结构体。完成的主要任务有：

1.  分配 `pcap_t` 结构体内存
2.  对结构体中的 **平台无关** 属性采用默认值设置
3.  对结构体中的 **平台相关** 属性（主要是一些以 `_op` 字样结尾）指定默认一个无任何操作的函数。这一过程可以在其调用的static函数 `initialize_op` 可以体现。
4.  指定平台相关的初始化函数入口，比如Linux平台上指定active操作的函数为 `pcap_activate_linux`

## `pcap_activate`<a id="sec-3-2" name="sec-3-2"></a>

此函数主要是绑定raw socket来实现抓取数据。

实现的步骤如下：
1.  socket()
2.  setsocketopt()
3.  mmap()

### 获取link层协议<a id="sec-3-2-1" name="sec-3-2-1"></a>

在 `pcap/bpf.h` 中定义了一些link层类型，先通过函数 `iface_get_arp_type` 获得arp层的类型，然后通过函数 `map_arphrd_to_dlt` 来将此类型转化为自己定义的类型。

这个过程代码看的懂，但是与kernel深度耦合，不理解kernel里面 `TPACKET_V2` 等实现完全白搭，所以至此，代码跟踪开展不下去了，给跪了，了解kernel 网络部分！！

## 注意点:<a id="sec-3-3" name="sec-3-3"></a>

libpcap性能还是一般般,对于像镜像流量分析这样的场景显得无能为力,以前遇到的这个场景的解决办法是采用ntop中的pfring,这样稍微好点. pfring里有零拷贝的pcap插件,采用那个会好很多.
