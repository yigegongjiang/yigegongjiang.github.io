---
title: Socket 与 ICMP
date: 2023-05-13 22:07:57
categories:
- 技术
tags:
- 网络
---

关于网络相关的历史文章中，很多次提到 Socket 套接字的重要性。最近看到 ICMP 的一些知识，发现和 Socket 有非常强的依赖。于是对 Socket 和周边，再做一次逻辑的梳理。
本文会对 **ICMP 查询/差错报文** 如何 **NAT网关** 和**写入 Socket 接收缓冲区**进行具体阐释。

### Socket、文件描述符、API、套接字

Socket 在系统内部创建后是唯一的。和 源 ip(本机 ip)、目标 ip(服务端 ip)、源端口、目标端口 有一定的绑定关系，根据具体协议而定。
通过这种绑定关系，可以将对应的数据包写入到对应的 Socket 接收缓冲区中供上层读取。
如果绑定关系出现 1-N，即一个数据包找到了多个 Socket，那么数据包就会往多个 Socket 中都写入一份，在后面 ICMP 中做分析。

Socket api，各个语言都有上层实现，底层都是对 Socket C Api 的封装。就是大家知道的 `gethostbyname`、`bind/connect/write/sendto` 这些。

Socket 文件描述符，不是真的文件。在 Unix 下习惯对资源进行抽象，即一切接文件，包括内存数据。Socket 文件描述符是内核内存区域的指针。

套接字，就是指 Socket。一般也就是指 api 或者 文件描述符 其中的一种，看具体场景。

Socket api 可以直接操作传输层(TCP/UDP)或者网络层(ICMP)，从 Socket 往下(传输层/网络层...)开始，都属于操作系统管辖。
所以 HTTP 经历很多版本的优化，依旧差强人意，就是因为核心层 TCP 不好改动，需要系统升级。最后通过 UDP 做上层改造来实现 QUIC。

<!-- more -->

### TCP Socket

Socket 是 TCP 的中流砥柱。TCP 需要那么多的控制信息，都是需要记录的。这些记录，传输层是无法完成的。也都会存储于 Socket 文件中。

TCP 的 Socket，可以理解是一个隧道。即 A Socket 写入，B Socket 读出，像隧道的两个出入口，比喻很形象。
服务端 B bind 只需要一个 Socket，但是接收到每一个客户端请求后，都会单独为该客户端生成一个独立的 B Socket。
之后，A Socket 和 B Socket 在 connect(三次握手)阶段为 Socket 文件描述符确定 src\_ip & src\_port & des\_ip & des\_port 四个绑定元素。
这四个绑定元素，可以使得隧道的入口数据包，一定能够在隧道的出口出现，即双向通行。

### UDP Socket

因为 TCP 非常重要，又是 HTTP 的御用，所以很多人对 TCP 更了解，反而对直来直往的 UDP 陌生了。

TCP 是数据流，读取和接收都是不固定的字节，所以需要进行黏包处理。
UDP 是数据报，即发送 n 个字节，接收 n 个字节，每个消息都有边界。
对于 UDP 开发来说，要主动做四个最基础的核心事情：**超时检测、回执、包序号、分段**。
包序号和分段，也没有什么复杂的逻辑，用 UDP 上传一张图片来理解：
客户端要发送的图片为 8000 字节，分成 8 个包，每个包 1000 字节。增加序号 0 - 7。客户端把这套数据结构缓存在内存里面，随时应对失败等场景的重发。
服务端收到乱序后的包，立刻发送回执，并根据序号进行重组。
还需要一些控制用的数据包，比如客户端发送完成的标识、图片分组标识等。

UDP 的服务端，不会像 TCP 一样，为每个客户端都单独维护一个 Socket。UDP 只有一个 Socket，这个 Socket 只会绑定 Server\_ip & Server\_port。
UDP 的客户端，不用进行 connect，UDP 的 Socket 只会绑定 Client\_ip & Client\_port。
这样 Socket 也就变成了**单向隧道**，即等待对方发给自己数据(对方拥有自己的ip和port)。发给对方数据，则通过 sendto 指定 ip&port 的方式空投给对方数据包。
> 备注1:实际上没有单向隧道一说，UDP 也是通过 Socket 隧道的发送缓存区发送数据的。只是通过 sendto，不像 TCP 的 write 那样默认会填充 ip&port。
> 备注2:实际上也有单向隧道一说，因为 Socket 只有自己的 ip&port，所以反向的 ICMP 差错报文数据包是没有办法接收的。后面再说。

总之，因为 Socket 只有自己的 ip&port，没有对方的 ip&port，实际上和 TCP 会有不少变化。
比如：
1. 对于 NAT 不严格类型，多个服务端都可以向 Client 发送数据(sendto(Client\_ip,Client\_port))。这一点相比 TCP(双方 Socket 是 1-1 的)，会比较方便实现 NAT 穿越。
2. UDP 可以通过同一个 Socket，使用 sendto 向不同的 Server 发送数据。
3. ICMP 差错报文可能会通知 Client 目的地址和端口不可达。Socket 中没有 Server\_ip & Server\_port，内核协议栈无法找到 Socket，也就无法通知到 Client。(Client 会干等，一直等)

当然，**UDP Socket 也可以执行 connect**。这样就可以绑定服务端的 ip&port 了。这样，上面的几个问题也就解决了。

### Socket bind/connect 的作用

**bind，是自定义自己的 ip 和 port 到 Socket 中**。
**connect，是自定义对方的 ip 和 port 到 Socket 中**。

所以，客户端也可以 bind 的，这样就可以指定自己的端口了，就不需要操作系统随机指定了。如果两台公网的 UDP，就可以这样操作，摆脱 Client-Server 模型，两台设备都可以随时向对方发送消息，即两台设备同时充当客户端和服务端。
所以，UDP 也可以 connect，这样就可以将四元组信息绑定到 Socket 中，接收到 ICMP 差错报文，就可以立刻接收到错误信息了。(下面会说明为什么)

### ！！ICMP 是如何穿过 NAT 的

先简要描述下 NAT 场景。
一个数据包，在正常的路由器、交换机传输中，是不会改变网络层 src\_ip 和 des\_ip 的。所以我们收到的数据包，网络层的 des\_ip 都是服务端 ip，不会变成网关或者中间路由器的 ip。
上面这种场景是**转发网关**。
但是对于内网设备，在 NAT 路由的时候，会把 内网 IP 映射为 NAT 设备的公网 IP，以解决 ipv4 短缺问题。这个时候数据包网络层的 src\_ip 会从内网 ip 192.xx 变为公网 ip xx.xx。
上面这种场景是**NAT 网关**。
NAT 网关会有很多有类型，按照严格程度，有锥形、完全锥形等，主要通过数据包的 src\_ip&src\_port 建立一个映射表。这对 NAT 穿越增加不小的难度。

ICMP 是网络层协议，没有端口的概念。NAT 是需要根据端口做内网和公网映射表，来**实现内网访问互联网**和**外网数据包进入内网**。
显然，这有些矛盾。预期来说，从内网出去的 ICMP 查询报文，公网返回后就 NAT 不进来了。从公网返回的 ICMP 差错报文 也同样 NAT 不进来。

ICMP 包括查询和差错两种。这两种方式穿过 NAT 的方式是不一样的。
查询报文，有 type/code/identifier 字段，NAT 设备会将这些字段组合出一个虚拟的 Client port，写入映射表中。公网返回的时候，type 会有规律变化(8->0)，code/identifier 不会变，可以逆向从 映射表 中找到对应的内网 ip&port。
差错报文，**数据包体会把源出错数据包的内容也携带上**，里面会有 ip&port。NAT 设备将这个 ip&port 拿出来，就可以从 映射表 中找到对应的内网 ip&port。

### ！！ICMP 是如何写入进程(Socket)的

ICMP 数据包，也是通过 Socket 进行发送的。Socket 创建的时候，可以选择 SOCK_STREAM(TCP)、SOCK_DGRAM(UDP)，还有一个选项就是 **SOCK_RAW**(源数据包)。
通过 SOCK_RAW，再指定一个协议类型 **IPPROTO_ICMP**，然后按照 ICMP 查询/差错 报文的数据包格式填充数据，就是 ICMP 数据包了。

对于 ICMP 差错报文，都是外部(网关、路由器、交换机等)发给 Client，这个时候操作系统会根据 ICMP 携带的 目标 ip&port，找到可能的 Socket 并写入接收缓冲区。
注意，这里找到的 Socket 可能不是一个。因为 ICMP 不是传输层协议，不和 TCP/UDP 有强依赖。(马上回说到)
举例来说，UDP 如果没有 connect，那么 Socket 在 sendto 后，会把 src\_id & src\_port 绑定在 Socket 中。这个时候，所有能发到 src\_id & src\_port 的 UDP 数据包，都会被写入该 Socket 的接收缓冲区中。
但是对于 ICMP，此时是无法写入的。因为 Client 可能会 sendto 到多个服务端，其中一个服务端不可达，返回了 ICMP，如果被 Socket 写入，那么代表所有服务端都不可达了，这显然不合理。
所以对于 TCP/UDP 来说，ICMP 要写入到 Socket 缓冲区中，一定需要四元组完备。这个时候 UDP 可以调用 connect，将 Socket 绑定 des\_id & des\_port，这个时候 UDP 就不能在给其他服务端发送消息了，收到的 ICMP 一定是当前服务端出了问题。

对于 ICMP 查询报文，操作系统干预的就少了。
ICMP 查询报文是通过开启进程使用 Socket api 发送 type=8 的数据包，然后接受 type=0 的数据包。
**这个 Socket 里面，只绑定了 src\_ip**。因为 ICMP 没有端口号概念(非传输层协议)，所以在 sendto 后，Socket 只绑定了 src\_ip。
那么，**这个 Socket 就只能强制接收所有发往这个 src\_ip 的 ICMP 报文，包括查询和差错报文**。
或者，**这个 Socket 就一个 ICMP 报文都不接收**，这显然不可能。
所以，**开发 ICMP 查询报文的进程，会收到所有发往本机的 ICMP 报文，需要通过 type/code/identifier 进行过滤，才能拿到最终的查询回显报文**。
这也就是上面刚说到的，**ICMP 差错报文，不仅仅会给到 UDP 的 Socket，还会给到发出 ICMP 查询报文的 Socket**。

下面会给出 ICMP 查询报文的 chatGPT 版本实现，在 recvfrom 里面，如果本机同时 traceroute，那么是可以收到 traceroute UDP ICMP 差错报文的。
注意代码内部 `recv_ping` 里面的过滤，判断了 `icmp->icmp_type` & `icmp->icmp_id` & `icmp->icmp_seq`。
icmp_type 是 0，即 ICMP 查询回显类型(此时 icmp_code 按照协议一定也为0)。
icmp_id 是进程号，即前面说的 identifier，同一个机子可能同时多进程发送多个 ICMP 查询。
icmp_seq 是单次请求号，一个进程可能发送多个 ICMP 查询。
如果没有这三个过滤，实际上是会接收到所有本机 ICMP 数据包的。

```c
问：用 C 语言实现一个 ping icmp 功能
chatGPT：
要使用C语言实现一个ping ICMP功能，你可以使用原始套接字（raw socket）来发送ICMP回显请求（ping）并接收ICMP回显应答。下面是一个简单的示例代码。
在这个示例中，我们使用原始套接字创建一个ICMP套接字，然后发送ICMP回显请求（ping）并接收ICMP回显应答。
代码中的send_ping函数负责构造并发送ICMP回显请求，recv_ping函数负责接收并验证ICMP回显应答。time_diff函数用于计算时间差。
请注意，使用原始套接字需要具有管理员权限或root权限才能运行。在运行代码之前，请确保你具备适当的权限。
这只是一个简单的示例，实际的ping程序可能需要更多的错误处理和参数处理。此外，由于ICMP协议的使用可能受到网络和防火墙配置的限制，因此在某些情况下可能无法正常工作。

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/socket.h>
#include <netinet/ip.h>
#include <netinet/ip_icmp.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <netdb.h>
#include <sys/time.h>

#define PACKET_SIZE 64
#define MAX_WAIT_TIME 5
#define MAX_NO_PACKETS 3

// IP首部结构
struct ipheader {
    unsigned char      iph_ihl:4, iph_ver:4;
    unsigned char      iph_tos;
    unsigned short int iph_len;
    unsigned short int iph_ident;
    unsigned short int iph_flags:3, iph_offset:13;
    unsigned char      iph_ttl;
    unsigned char      iph_protocol;
    unsigned short int iph_chksum;
    unsigned int       iph_sourceip;
    unsigned int       iph_destip;
};

// ICMP首部结构
struct icmpheader {
    unsigned char icmp_type;
    unsigned char icmp_code;
    unsigned short int icmp_chksum;
    unsigned short int icmp_id;
    unsigned short int icmp_seq;
};

// 计算校验和
unsigned short checksum(void *b, int len) {
    unsigned short *buf = b;
    unsigned int sum = 0;
    unsigned short result;

    for (sum = 0; len > 1; len -= 2)
        sum += *buf++;

    if (len == 1)
        sum += *(unsigned char *)buf;

    sum = (sum >> 16) + (sum & 0xFFFF);
    sum += (sum >> 16);
    result = ~sum;
    return result;
}

// 发送ICMP回显请求
void send_ping(int sockfd, struct sockaddr_in *addr, int seq) {
    struct icmpheader icmp;
    icmp.icmp_type = ICMP_ECHO;
    icmp.icmp_code = 0;
    icmp.icmp_id = getpid();
    icmp.icmp_seq = seq;
    icmp.icmp_chksum = 0;
    icmp.icmp_chksum = checksum(&icmp, sizeof(icmp));

    sendto(sockfd, &icmp, sizeof(icmp), 0, (struct sockaddr *)addr, sizeof(*addr));
}

// 接收ICMP回显应答
int recv_ping(int sockfd, struct sockaddr_in *addr, int seq, struct timeval *tv) {
    fd_set readfds;
    FD_ZERO(&readfds);
    FD_SET(sockfd, &readfds);

    struct timeval timeout;
    timeout.tv_sec = MAX_WAIT_TIME;
    timeout.tv_usec = 0;

    int ret = select(sockfd + 1, &readfds, NULL, NULL, &timeout);
    if (ret <= 0)
        return -1;

    struct sockaddr_in from;
    socklen_t fromlen = sizeof(from);
    char packet[PACKET_SIZE];
    ret = recvfrom(sockfd, packet, sizeof(packet), 0, (struct sockaddr *)&from, &fromlen);
    if (ret <= 0)
        return -1;

    struct ipheader *ip = (struct ipheader *)packet;
    struct icmpheader *icmp = (struct icmpheader *)(packet + ip->iph_ihl * 4);

    if (icmp->icmp_type == ICMP_ECHOREPLY
        && icmp->icmp_id == getpid()
        && icmp->icmp_seq == seq) {
        gettimeofday(tv, NULL);
        return 0;
    }

    return -1;
}

// 计算时间差（毫秒）
long long time_diff(struct timeval *start, struct timeval *end) {
    long long start_ms = start->tv_sec * 1000 + start->tv_usec / 1000;
    long long end_ms = end->tv_sec * 1000 + end->tv_usec / 1000;
    return end_ms - start_ms;
}

int main(int argc, char *argv[]) {
    if (argc != 2) {
        printf("Usage: %s <hostname/IP address>\n", argv[0]);
        return 1;
    }

    struct hostent *host = gethostbyname(argv[1]);
    if (host == NULL) {
        printf("Failed to resolve hostname\n");
        return 1;
    }

    int sockfd = socket(AF_INET, SOCK_RAW, IPPROTO_ICMP);
    if (sockfd < 0) {
        printf("Failed to create socket\n");
        return 1;
    }

    struct sockaddr_in addr;
    memset(&addr, 0, sizeof(addr));
    addr.sin_family = AF_INET;
    addr.sin_addr = *(struct in_addr *)host->h_addr;

    int seq;
    for (seq = 0; seq < MAX_NO_PACKETS; seq++) {
        struct timeval start, end;
        gettimeofday(&start, NULL);

        send_ping(sockfd, &addr, seq);
        if (recv_ping(sockfd, &addr, seq, &end) == 0) {
            printf("Ping response from %s: seq=%d time=%lldms\n", argv[1], seq, time_diff(&start, &end));
        } else {
            printf("Ping timeout for seq=%d\n", seq);
        }

        sleep(1);
    }

    close(sockfd);
    return 0;
}
```
___

小玛蒂尔达：生活是否永远如此艰辛，还是仅仅童年才如此？
里昂：总是如此。
