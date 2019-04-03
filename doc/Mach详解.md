# Mach详解

## 内核

在操作系统中，都有一个关键的核心组件，叫做**内核（kernel）**。  
内核能够充分利用底层 CPU 提供的所有特性和能力，为客户提供各种各样的服务。

内核一般分为以下三种：

- 单内核
- 微内核
- 混合内核

iOS以及Mac OS X 系统的内核就是混合内核，我们叫它 **XNU**，XNU的核心是 Mach。

在 Mach之上建立了一个**BSD层** ，他们都在同一地址空间中，和单内核一样具有较高的运行效率。

### Mach的职责

- 进程和线程管理：我们平时所用到的 POSIX 线程和 NSThread 都是和 Mach 层线程一一对应的，POSIX 线程是BSD 层对 线程的更高层次抽象
- 虚拟内存的分配和管理
- CPU 等物理设备的分配和调度
- 异常：Mach 在已有的消息传递机制上实现了一种异常处理机制

## Mach概述
### 1.Mach的设计原则
在Mach中所有东西（Task、线程、虚拟内存等））都是对象（Resource）。  
对象与对象之间通信**只能通过端口收发消息**。
### 2.Mach消息
Mach消息分为：
- 简单消息
- 复杂消息
#### 2.1简单消息
简单消息包含最基本的两部分：消息头、消息体。可以选择性的添加消息尾。  
- 消息头：mach_msg_header_t
- 消息体：mach_msg_body_t
- 消息尾：mach_msg_trailer_t  

其结构体定义如下：
```
typedef struct 
{
mach_msg_bits_t   msgh_bits;//标志位
mach_msg_size_t   msgh_size;//大小
mach_port_t       msgh_remote_port;//目标端口（发送：接受方，接收：发送方）
mach_port_t       msgh_local_port; //源端口（发送：发送方，接收：接收方）
mach_port_name_t  msgh_voucher_port;
mach_msg_id_t     msgh_id;
} mach_msg_header_t; //消息头

typedef struct
{
mach_msg_size_t msgh_descriptor_count;
} mach_msg_body_t;//消息体

typedef struct
{
mach_msg_header_t       header;
mach_msg_body_t         body;
} mach_msg_base_t; //基本消息

typedef unsigned int mach_msg_trailer_type_t;//消息尾的类型

typedef struct 
{
mach_msg_trailer_type_t   msgh_trailer_type;
mach_msg_trailer_size_t   msgh_trailer_size;
} mach_msg_trailer_t; //消息尾
```
#### 2.2复杂消息
将消息头的标志位**mach_msg_bits_t**设置为MACH_MSGH_BITS_COMPLEX，就表示复杂消息。

#### 2.3消息收发
消息的收发在用户态都是通过如下方法进行的：
```
extern mach_msg_return_t    mach_msg(
mach_msg_header_t *msg,
mach_msg_option_t option,
mach_msg_size_t send_size,
mach_msg_size_t rcv_size,
mach_port_name_t rcv_name,
mach_msg_timeout_t timeout,
mach_port_name_t notify);  
```

