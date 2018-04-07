---
title: Linux防火墙-iptables
date: 2017-04-02 09:42:23
---
# Linux防火墙--iptables学习

转自[简书](http://www.jianshu.com/p/5f38e7253fc8?utm_campaign=hugo&utm_medium=reader_share&utm_content=note)

iptables是Linux系统提供的一个强大的防火墙工具，可以实现包过滤、包重定向、NAT转换等功能。iptables是免费的，iptables是一个工具，实际的功能是通过netfilter模块来实现的，在内核2.4版本后默认集成到了Linux内核中。

# 一、 iptables的构成

## 1. 规则（rules）

规则是iptables对数据包进行操作的基本单元。即“当数据包符合规则定义的条件时，就按照规则中定义的动作去处理”。

规则中定义的条件一般包括源地址/端口、目的地址/端口、传输协议（TCP/UDP/ICMP）等。而规则定义的动作一般有放行（ACCEPT）、拒绝（REJECT）和丢弃（DROP）等。

配置iptables实际上就是增删修改这些规则。

## 2. 链（chains）

链是数据包传播的路径，每条链中都有若干个规则。当一个数据包到达一条链时，iptables会按照规则的顺序，从该链的第一条规则开始往下检查，如果有条件匹配的规则，则按照规则定义的动作执行；否则继续检查下一条规则。如果该数据包和链中所有的规则都不匹配，则iptables会根据该链预先定义的默认策略来处理数据包。

## 3. 表（tables）

iptables内置了4个表，即filter表、nat表、mangle表、raw表，分别用于实现包过滤、网络地址转换、包修改和数据跟踪处理等功能。每个表中含有若干条链，具体的规则就是根据实现目的的不同，添加到不同表的不同链中。

如下图所示，各个表相关的链：

![img](http://upload-images.jianshu.io/upload_images/3329890-fcea46b652d9b2fa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

raw表有2条链：PREROUTING、OUTPUT

mangle表有5条链：PREROUTING、POSTROUTING、INPUT、OUTPUT、FORWARD

nat表有3条链：PREROUTING、POSTROUTING、OUTPUT

filter表有3条链：INPUT、FORWARD、OUTPUT

4个表的优先级为：raw > mangle > nat > filter

# 二、iptables的传输过程

![img](http://upload-images.jianshu.io/upload_images/3329890-3851549020dc2b60.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

当数据包到达网卡时，首先会进入PREROUTING链（注意，raw表处理完后就不会再进入nat表了），完成PREROUTING链中规则的匹配和执行后（比如DNAT），iptables根据数据包的目的IP是否为本机地址，判断是否需要将该数据包转发出去。

1.  如果数据包的目的IP为本机地址，它就会进入INPUT链。可以在filter表的INPUT链中添加包过滤规则，限制哪些数据包可以访问本机；经过INPUT链中的规则处理后，剩下的数据包在本机上任何进程都可以收到，并根据需要对它们进行处理；当进程处理完后，需要发送出去的数据包会经过OUTPUT链的处理，然后到达POSTROUTING链，经过处理（比如SNAT）后输出。

\2. 如果数据包的目的IP不是本机地址（比如做了DNAT、或者只是作为默认网关时走过来的包），并且系统内核开启了转发功能（ip_forward参数为1），则数据包会进入FORWARD链；此时可以在filter表的FORWARD链中设置相应的规则进行处理；经过FORWARD链的数据包再走到POSTROUTING链中处理（比如执行SNAT），最后输出。

综上，数据包在iptables中的传输链路有两种情况：

第一种：PREROUTING -> FORWARD -> POSTROUTING

第二种：PREROUTING -> INPUT -> LOCALHOST -> OUTPUT -> PUSTROUTING

所以，要想对数据包进行控制，主要可以在上面几条链路中添加规则。

（可以发现LVS的NAT模式下，会经过PREROUTING链(nat表) -> FORWARD链 -> POSTROUTING链(nat表)，请求的转发在PREROUTING链中进行DNAT，响应在POSTROUTING链中进行SNAT。无论时请求还是响应都不会走到INPUT链和OUTPUT链中）

# 三、 iptables的使用

iptables对链的操作方法有：-I（插入），-A（追加），-R（替换），-D（删除），-L（显示）

注意，-I是将规则插入到第一行，-A是将规则追加到链的最后一行。

iptables -F：清除所有的规则

iptables -F -t nat：清除nat表的规则

iptables -P INPUT DROP：配置INPUT链的默认规则为DROP

示例：

iptables -t filter -A INPUT -s 192.168.1.10 -j ACCEPT

这是一条简单的规则，表示在filter表的INPUT链中追加一条规则，规则的匹配条件是数据包的源IP为192.168.1.10，执行动作为允许（ACCEPT），即允许源IP为192.168.1.10地址的消息访问本机。

iptables中，默认表名就是filter，所以上面的规则也可以写成：

iptables -A INPUT -s 192.168.1.10 -j ACCEPT

iptables规则的匹配条件中常用的参数有：

-s匹配源地址

-d匹配目的地址

--sport匹配源端口

--dport匹配目的端口

-p匹配协议

-i匹配输入的网卡

-o匹配输出的网卡

！取反

-j规则的执行动作

示例：配置NAT转发实现内网节点访问公网

环境假设：

路由节点：

Lan口：192.168.1.10/24 eth0

Wan口：50.75.153.98/24 eth1

内网节点：192.168.1.92

\1. 在内网节点上配置默认网关为路由节点的Lan口，确保内网节点的包都能走到路由节点。

route add default gw 192.168.1.10

\2. 在路由节点上打开linux的路由转发功能。

sysctl -w net.ipv4.ip_forward=1

\3. 将FORWARD链的默认策略设置为DROP

iptables -P FORWARD DROP

这样确保对内网的控制，只把允许访问公网的IP添加到规则中。

\4. 允许任意地址之间的确认包和关联包通过。

iptables -A FORWARD -m state --state ESTABLISHED,RELATED -j ACCEPT

这条规则很关键，否则即使后面添加了允许IP访问的规则也没用。

\5. 允许指定IP地址访问公网

iptables -A FORWARD -s 192.168.1.92/24 -j ACCEPT

\6. 配置一条SNAT规则，将内网节点的源IP转换为公网IP后，再将消息发出。

iptables -t nat -A POSTROUTING -s 192.168.1.92 -j SNAT --to 50.75.153.98

示例：配置NAT转发实现公网用户访问内网节点

环境假设：

路由节点：

Lan口：192.168.1.10/24 eth0

Wan口：50.75.153.98/24 eth1

内网节点：192.168.1.92

1.  在路由节点上打开linux的路由转发功能。

sysctl -w net.ipv4.ip_forward=1

\2. 将FORWARD链的默认策略设置为DROP

iptables -P FORWARD DROP

这样确保对内网的控制，只把允许访问的IP添加到规则中。

\3. 允许任意地址之间的确认包和关联包通过。

iptables -A FORWARD -m state --state ESTABLISHED,RELATED -j ACCEPT

这条规则很关键，否则即使后面添加了允许IP访问的规则也没用。

\4. 允许访问指定的IP地址

iptables -A FORWARD -d 192.168.1.92 -j ACCEPT

\5. 配置一条DNAT规则，将访问路由节点的公网地址转换为访问内网节点的私网地址。

iptables -t nat -A POSTROUTING -d 50.75.153.98 -j DNAT --to 192.168.1.92

**PS：用iptables的raw表解决ip_conntrack:table full, dropping packet的问题**

raw表中包含PREROUTING链和OUTPUT链，优先级最高，可以对数据包在进入NAT表的PREROUTING链之前对消息进行处理。当用户启用了RAW表，消息在经过RAW表的PREROUTING链处理后，将跳过NAT表和ip_conntrack处理，不再做地址转换和数据包的链接跟踪处理。

所以raw表可以用在那些不需要做nat和链接跟踪的情况，提升系统性能。因为启用链接跟踪时，系统会建立一个链接跟踪表，每个消息进来时，都会去查询链接跟踪表，当系统业务量过大时，可能会引发系统CPU消耗过高。

可以通过在RAW表的PREROUTING链中配置NOTRACK标记，使数据包不再进入nat表，达到不做链接跟踪的目的，如：

iptables -t raw -A PREROUTING -p tcp -d 192.168.1.10 --dport 80 -j NOTRACK

iptables -A FORWARD -m state --state UNTRACKED -j ACCEPT

表示对访问192.168.1.10:80服务的消息在raw表的PREROUTING链中添加NORTRACK标记，以后访问这个服务的消息都不会再进行链接跟踪。

另外，再附上鸟哥的iptables流程图，可以看到各个链也是根据不同的表区分优先级的。

![img](http://upload-images.jianshu.io/upload_images/3329890-df0602ca5ba7ff35.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)