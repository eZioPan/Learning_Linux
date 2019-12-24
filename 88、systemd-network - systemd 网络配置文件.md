# systemd.network

## 名称

systemd.network — 网络配置

## 概要

*network*`.network`

## 描述

网络设置通过 *systemd-networkd(8)* 实现

主网络文件必须具有 `.network` 的文件后缀名；其它后缀名将被忽略。一旦链接出现，则对应网络即被应用。

`.network` 文件从系统网络文件夹中读取，包含 `/usr/lib/systemd/network`， `/usr/local/lib/systemd/network`，易失性运行时网络文件夹 `/run/systemd/network`，以及本地管理员网络文件夹 `/etc/systemd/network`。所有配置文件均以字符方式排序并处理，无关它们存在与哪个目录中。但是，具有完全相同的文件名的文件将相互替换。在 `/etc/` 中的文件具有最高优先级，`/run` 中的其次，`/usr` 下的优先级最低。这样系统提供的文件就可以按需被本地文件替换。在特殊形况下，一个空文件（文件大小为 0）或指向 `/dev/null` 的软链接将完全禁用该配置文件（也就是被 “masked”）。

在网络文件 `foo.network` 目录下，可以存在名为 `foo.network.d/` 的 “插入” 目录。该目录下所有以 `.conf` 结尾的文件都会在主文件解析之后被解析。这有助于修改或添加配置，而不需要修改主配置文件。每个插入文件必须具有合适的段头部（section header）。

除了 `/etc/systemd/network`，插入文件夹 `.d` 也可以被放置在 `/usr/lib/systemd/network` 以及 `/run/systemd/network` 目录下。`/etc` 下的插入文件优先于 `/run`，`/run` 下的文件优先于 `/usr/lib`。插入文件优先于高于对应的主网络文件。

注意，一个界面若未配置 IPv6 地址，也不启用 DHCPv6 和 IPv6LL，则应该被认为不支持 IPv6。将会自动向对应界面的 `/proc/sys/net/ipv6/conf/<ifname>/disable_ipv6` 文件写入 `1` 来禁用 IPv6。

## `[Match]` 段选项

包含 `[Match]` 段的网络文件，将决定给定的网络文件是否会套用在给定的设备；`[Network]` 段则决定了设备该如何被配置。首个（以字母序）匹配上的设备名的网络文件将会被启用，所有其它文件将被忽略，即便它们也匹配上了。

一个网络文件与一个设备匹配的条件是，所有在 `[Match]` 段出现的内容都被满足。当一个网络文件不含有任何有效的 `[Match]` 段时，该文件将匹配所有的界面，此时 **systemd-networkd** 将发出警告。提示：若要避免警告，并清楚指明全部的界面都应该被匹配，则添加下面的行：

```systemd.network
Name=*
```

该段可接受下方的键：

- `MACAddress=`

    用白空格隔开的硬件地址列表。使用逗号、连字符、点号分割的十六进制表示的全地址。  
    见下方的例子。  
    这个选项可以多次出现，如此这样列表即被合并。  
    若一个空字符串赋予了该选项，则在该定义之前的硬件地址将被全部重置。

    *例子：*

    ```systemd.network
    MACAddress=01:23:45:67:89:ab 00-11-22-33-44-55 AABB.CCDD.EEFF
    ```

- `Path=`

    一个由白空格隔开的列表，该列表由 shell-style 通配样式匹配的永久路径组成，与 udev 属性 `ID_PATH` 导出的名称一致。  
    若该列表前缀为 `!`，则测试反转；也即是说，当 `ID_PATH` 不与任何该列表中的元素匹配时返回真。

- `Driver=`

    一个由白空格隔开的列表，该列表由 shell-style 通配样式匹配的当前与设别绑定的驱动组成，与 udev 属性 `ID_NET_DRIVER` 的父级设备一致，若其未被设置，则由 `ethtool -i` 返回值确定。  
    若该列表前缀为 `!`，则测试反转。

- `Type=`

    一个由白空格隔开的列表，该列表由 shell-style 通配样式匹配的设备类型，与 udev 属性 `DEVTYPE` 导出的名称一致。  
    若该列表前缀为 `!`，则测试反转。

- `Name=`

    一个由白空格隔开的列表，该列表由 shell-style 通配样式匹配的设备名称，与 udev 属性 `INTERFACE` 导出的名称一致。  
    若该列表前缀为 `!`，则测试反转。

- `Property=`

    一个由白空格隔开的列表，该列表由 udev 属性名，一个等号，以及属性值组成。若指定了多个属性，则测试结果为多个属性的 `且` 结果。  
    若该列表前缀为 `!`，则测试反转。  
    若某个值具有空格，则该键值对必须用引号包裹起来。  
    若一个值具有引号，则用 `\` 来转义。

    *Example：* 如果一个 `.network` 具有以下行：

    ```systemd.network
    Property=ID_MODEL_ID=9999 "ID_VENDOR_FROM_DATABASE=vendor name" "KEY=with \"quotation\""
    ```

    那么，该 `.network` 文件则仅匹配符合上述三个条件的界面

- `WLANInterfaceType=`

    一个由白空格隔开的列表，该列表由 无线网络类型 组成。支持的值为 `ad-hoc` `station` `ap` `ap-vlan` `wds` `monitor` `mesh-point` `p2p-client` `p2p-go` `p2p-device` `ocb` `nan`。  
    若该列表前缀为 `!`，则测试反转。

- `SSID=`

    一个由白空格隔开的列表，该列表由 shell-style 通配样式匹配的 当前连接的无线 LAN 的 SSID。  
    若该列表前缀为 `!`，则测试反转。

- `BSSID=`

    用白空格隔开的当前连接的无线 LAN 硬件地址列表。使用逗号、连字符、点号分割的十六进制表示的全地址。  
    参见 `MACAddress=` 的例子。  
    这个选项可以多次出现，如此这样列表即被合并。  
    若一个空字符串赋予了该选项，则在该定义之前的 `BSSID` 将被全部重置。

- `Host=`

    通过主机名（hostname）或主机的机器 ID（machine ID）来匹配。参见 *systemd.unit(5)* 中的 `ConditionHost=`。  
    若该列表前缀为 `!`，则测试反转。  
    若一个空字符串赋予了该选项，则在该定义之前的值将被全部重置。

- `Virtualization=`

    测试该系统是否运行在虚拟环境下，且额外测试其是否则特定的实现。参见 *systemd.unit(5)* 中的 `ConditionVirtualization=`。  
    若该列表前缀为 `!`，则测试反转。  
    若一个空字符串赋予了该选项，则在该定义之前的值将被全部重置。

- `KernelCommandLine=`

    测试特定的内核命令行是否设置。参见 *systemd.unit(5)* 中的 `ConditionKernelCommandLine=`。  
    若该列表前缀为 `!`，则测试反转。  
    若一个空字符串赋予了该选项，则在该定义之前的值将被全部重置。

- `KernelVersion=`

    测试内核版本（由 `uname -r` 返回）是否符合特定的表达式。参见 *systemd.unit(5)* 中的 `ConditionKernelVersion=`。  
    若该列表前缀为 `!`，则测试反转。  
    若一个空字符串赋予了该选项，则在该定义之前的值将被全部重置。

- `Architecture=`

    测试系统是否在特定的架构上运行。参见 *systemd.unit(5)* 中的 `ConditionArchitecture=`。  
    若该列表前缀为 `!`，则测试反转。  
    若一个空字符串赋予了该选项，则在该定义之前的值将被全部重置。

## `[Link]` 段选项

`[Link]` 段接受下列键：

- `MACAddress=`

    设置设备的硬件地址

- `MTUBytes=`

    以字节为单位，设置设备的最大传输单元。  
    可以使用 `K` `M` `G` 后缀，且以 `1024` 为底数。

    注意，若该界面启用了 `IPv6`，且 `MTU` 小于 1280（IPv6 的 MTU 最小值），则其会自动增加到 1280。

- `ARP=`

    接受布尔值。若设置为真，则启用该界面的 `ARP`（底层地址解析协议）。若未设置，则使用系统的默认设置。

    举例来说，在单个底层物理界面上创建多个 `MACVLAN` 或 `VLAN` 虚拟界面时，禁用 ARP 是十分有用的。其后它们将仅作为一个 link/"bridge" 设备将流量汇总至相同的物理链接上，并不会参与网络的其它部分。

- `Multicast=`

    接受布尔值。若设置为真，则设置设备的多播 flag。

- `AllMulticast=`

    接受布尔值。若设置为真，则驱动会呈递网络中所有的多播网络包。当启用多播路由时出现。

- `Unmanaged=`

    接受布尔值。当为 `yes` 时，将不会尝试开启或配置匹配的链路，等价于没有对应的网络文件。默认为 `no`。

    这个选项常用于阻止其后匹配的网络文件影响特定的、通过其它程序控制的界面。

- `RequiredForOnline=`

    接受布尔值，或运行状态。请参考 *networkctl(1)* 来了解可能的运行状态。  
    当设置为 `yes` 时，在运行 **systemd-networkd-wait-online** 时，该界面被用于判定系统是否在线。  
    当设置为 `no` 时，在检测系统是否在线时忽略该网络。  
    当设置为某运行状态时，`yes` 为隐含标志，且网络界面必须处于指定的运行状态时，才会被认为是在线状态。  
    默认为 `yes`。

    在通常情况下，网络将被正常地启用，但当出现未被 DHCP 赋予地址，或网线未插入，则链接会保持离线状态，并在存在 `RequiredForOnline=no` 的情况下自动被 **systemd-networkd-wait-online** 跳过。

## `[Network]` 段选项

`[Network]` 段接受下列键：

- `Description=`

    设备的描述。仅做展示用。

- `DHCP=`

    启用 **DHCPv4** 且/或 **DHCPv6** 支持。  
    接受 `yes` `no` `ipv4` `ipv6`。  
    默认为 `no`。

    注意，`DHCPv6` 将自动被路由通告（Router Advertisement）触发，而不管该参数。  
    当明确了 `DHCPv6` 时，DHCPv6 客户端 将无视链路是否有路由，或路由传递了何种参数。  
    参见 `IPv6AcceptRA=`。

    注意，默认情况下，通过 DHCP 获得的域名将不被用于名称解析。  
    参见 `UseDomains=`。

    参见 `[DHCPv4]` 或 `[DHCPv6]` 段来了解更多 DHCP 客户端 的支持。

- `DHCPServer=`

    接受一个布尔值。若设置为 `yes`，将启用 DHCPv4 服务器。  
    默认为 `no`，
    更多 DHCP 服务器设置可以参考 `[DHCPServer]` 段的设置。

- `LinkLocalAddressing=`

    启用 本地链路地址自动配置。  
    接受 `yes` `no` `ipv4` `fallback` 或 `ipv4-fallback`。  
    若设置为 `fallback` 或 `ipv4-fallback`，则仅在 DHCPv4 失败时配置本地链路地址。  
    注意，fallback 机制仅当 DHCPv4 客户端启用时起效，也就是说，他要求 `DHCP=yes` 或 `DHCP=ipv4`。  
    若设置了 `Bridge=`，则默认为 `no`，若未设置 `Bridge=`，默认为 `ipv6`。

- `IPv4LLRoute=`

    接受一个布尔值。若设置为真，则架设一个路由，该路由用于 非 IPv4LL 主机与 仅 IPv4LL 主机之间的通信。  
    默认为假。

- `DefaultRouteOnDevice=`

    接受一个布尔值。若设置为真，则架设一个绑定至该界面的路由。  
    默认为假。  
    当在点对点界面上创建路由时十分有效。  
    等价于下述命令：

    ```bash
    ip route add default dev veth99
    ```

- `IPv6Token=`

    一个前 64 bit 未设置的 IPv6 地址。  
    当设置时，为该链路指示 SLAAC IPv6 地址的 64 bit 界面部分。  
    注意，该 token 仅用于 SLAAC，并不用于 DHCPv6 地址，即便在这种情况下路由宣告需要 DHCP。  
    默认情况下，该 token 会自动生成。

- `LLMNR=`

    接受一个布尔值或者 `resolve`。为真时，在链路上启用 本地链路多播名称解析（Link-Local Multicast Name Resolution）。当设置为 `resolve` 时，仅启用解析，但不启用主机注册和声明。  
    默认为真。  
    该设置被 *systemd-resolved.service(8)* 读取。

- `MulticastDNS=`

    接受布尔值或者 `resolve`。为真时，在链路上启用多播域名解析（Multicast DNS）支持。当设置为 `resolve` 时，仅启用解析，但不启用主机或服务的注册和声明。
    默认为假。
    该设置被 *systemd-resolved.service(8)* 读取。

- `DNSOverTLS=`

    接受布尔值或者 `opportunistic`。当为真时，在链路上启用 安全传输层承载的域名解析（DNS-over-TLS）支持。当设置为 `opportunistic` 时，增加与 non-DNS-over-TLS 服务器的兼容度。这是通过自动关闭 DNS-over-TLS 来实现的。  
    这个选项对应着 *resolved.conf(5)* 中的全局设置 `DNSoverTLS=` 的每界面设置。  
    默认为假。  
    该设置被 *systemd-resolved.service(8)* 读取。

- `DNSSEC=`

    接受一个布尔值或者 `allow-downgrade`。当为真时，在链路上启用 域名解析安全扩展域名解析证实（DNSSEC DNS validation）支持。当设置为 `allow-downgrade` 时增加与 non-DNSSEC 网络的兼容性。这是通过自动关闭 DNSSEC 来实现的。  
    这个选项对应着 `resolved.conf(5)` 中的全局设置 `DNSSEC=` 的每界面设置。  
    默认为假。  
    该设置被 *systemd-resolved.service(8)* 读取。

- `DNSSECNegativeTrustAnchors=`

    白空格间隔的列表，由 `DNSSEC negative trust anchor domains` 组成。若该相定义且 `DNSSEC` 启用，通过该界面的 DNS 服务器的查找操作将仅限于 negative trust anchors 列表的内容，且不要求对特定域名及其子域名的认证。使用这个选项来关闭对特定私有域名的 DNSSEC 认证。由于这些域名并不能通过互联网 DNS 层级来认证。  
    默认为空列表。  
    该设置被 *systemd-resolved.service(8)* 读取。

- `LLDP=`

    控制对 以太网 链路层发现协议（LLDP） 网络包的接收的支持。LLDP 为一个链路层协议，通常被专业路由以及网桥实现，它们会声明一个系统是通过哪个物理端口连接的，以及一些其它的相关数据。  
    接受布尔值或者特定的 `routers-only`。当为真时，传入的 LLDP 网络包将被接受，并会维护一个关于所有 LLDP 邻居的数据库。若设置为 `routers-only`，则仅会收集来自各种类型的路由的 LLDP 数据，其它设备类型（比如工作站、电话等等）的 LLDP 网络包将被忽略。若为假，LLDP 收发将被关闭。  
    默认为 `routers-only`。  
    使用 *networkctl(1)* 来罗列已收集的邻居设备数据。LLDP 仅在以太网链路上可用。参见 `EmitLLDP=` 从本地系统来发送 LLDP 网络包。

- `EmitLLDP=`

    控制对于 以太网 链路层发现协议（LLDP）网络包的发送的支持。  
    接受布尔值或者特定的 `neareset-bridge` `non-tpmr-bridge` `customer-bridge`。  
    默认为假，也就是关闭 LLDP 网络包的发送。若不为假，以固定的间隔向链路发送一个包含本地系统信息的短 LLDP 网络包。  
    LLDP 网络包会包含本地主机名、本地机器 ID（与 *machine-id(5)* 中的一样）、本地界面名，以及系统的（在 *machine-info(5)* 中设置的）灵活主机名（pretty hostname）。LLDP 发送仅在以太网链路上启用。  
    注意，这个设置会在网络上发送可以确认主机的数据，所以不应该在不可信网络中启用，在不可信网络中这些认证数据不应该可见。  
    使用这个选项来允许其它系统确认它们是通过哪个界面与该系统相连的。三个特殊的值用来控制 LLDP 网络包的传播。`nearest-bridge` 设置仅允许该网络包传播至最近的相连的网桥上，`non-tpmr-bridge` 允许通过 Two-Port MAC Relays 传播，但不允许其它类型的网桥，`customer-bridge` 允许一直传播，直到到达一个特定的网桥。具体概念参见 *IEEE 802.1AB-2016*。  
    注意将该值设置为真等价于 `nearest-bridge`，也就是推荐的也是最严格的传播级别。查看 `LLDP=` 选项来启用 LLDP 接收。

- `BindCarrier=`

    一个链路名或一个链路名的列表。当设置时，控制当前链路的行为。当所有在该列表中的链路都处于离线（operational down）状态时，当前链路即被关闭。当至少一个链路有承载者时，当前界面即被启用。

- `Address=`

    一个静态的包含它们的前缀长度的 IPv4 或者 IPv6 地址，用 `/` 隔开地址和前缀长度。多次指定这个值将配置多个地址。地址的格式必须符合 *inet_pton(3)* 中的描述。该选项是仅包含一个 `Address` 键的 `[Address]` 段的简略写法（参见下文）。该值可以被多次指定。

    若指定的地址为 `0.0.0.0`（IPv4）或者 `::`（IPv6），则会自动从系统中未使用的地址池中占用一个地址段（address range）。注意，在 IPv4 中前缀长度必须大于等于 8，在 IPv6 中必须大于等于 64。获取地址段时，将于所有的网络界面，以及任何一致的网络配置文件进行比较，以防止地址段冲突。默认的系统级别的地址池网络包括 IPv4 的 `192.168.0.0/16` `172.16.0.0/12` `10.0.0.0/8`，以及 IPv6 的 `fd00::/8`。  
    这项功能在控制大量的动态创建的网络界面，且这些界面具有相同的网络配置，且需要自动地址段分配。

- `Gateway=`

    网关地址，其格式必须符合 *inet_pton(3)* 中的描述。该选项时仅包含一个 `Gateway` 键的 `[Route]` 段的简略写法。该选项可以被多次指定。

- `DNS=`

    一个 DNS 服务器地址，其格式必须符合 *inet_pton(3)* 中的描述。该选项可以被多次指定。该设置被 *systemd-resolved.service(8)* 读取。

- `Domains=`

    一个域名列表，这些域名应该使用该链路的 DNS 服务器解析。列表中的每个条目因该时一个域名，可选前缀 `~`。具有 `~` 前缀的域名被称为 限定路由域名”（routing-only domains）。不具有前缀的域名被称为 “搜索域名”（search domains），这些域名首先作为搜索后缀，用来将单标签（single-label）主机名扩展为完全限定域名`fully qualified domain names, FQDNs）。  
    若一个单标签主机名在该界面被解析，每一个定义的搜索域都会轮流作为主机名的后缀，将该主机名扩展为完全限定域名，直到某个完全限定域名被成功解析。

    `search` 和 `routing-only` 域名均用于 DNS 查询的路由：查找以这些域名结尾的主机名（因此也包含单标签名，若列表中包含 “搜索域名”），将会被路由至该界面配置的 DNS 服务器。域名路由逻辑 在多宿主主机 和 每界面 DNS 服务器服务特定私有 DNS 域时十分有用。

    `routing-only` 域名 `~.`（波浪号指出了一个路由域的定义，点号表明了所有有效 DNS 名的后缀，也就是 DNS 根域名）具有特殊的效果。它导致所有不被其它配置过的域名路由条目匹配的 DNS 流量，均路由至该界面配置的 DNS 服务器。这个设置在 若具有偏好的 DNS 服务器的链路连接上时，就直接使用它们。

    该设置被 *systemd-resolved.service(8)* 读取。“搜索域名”对应 *resolve.conf(5)* 中的域名和搜索条目。域名路由在传统 glibc API 中无对应 API，也就是说传统 glibc API 不具有将域名服务器限制在特定链接上的概念。

- `DNSDefaultRoute=`

    接受一个布尔值。若为真，则该链路配置的 DNS 服务器将用于解析不匹配任何链路的 `Domains=` 设置的域名。若为假，则该链路配置的 DNS 服务器将永不用于这类域名，并仅用于解析至少匹配上该链路所配置的一个域名的名称。若不指定则默认为自动模式：若搜索条目不匹配任何已配置的域名，且没有配置 `routing-only` 域名，则将网络包路由至该链接。

- `NTP=`

    一个网络授时协议（NTP）服务器地址。该选项可以多次指定。这个设置被 *systemd-timesyncd.service(8)* 读取。

- `IPForward=`

    配置系统的 IP 网络包转发（forwarding）。若启用，则会根据路由表转发由任何网络界面输入的网络包，并转发至任何其它界面上。  
    接受一个布尔值，或者 `ipv4` 或 `ipv6`，表示仅针对特定的地址组开启转发。它控制了该网络界面的 `net.ipv4.ip_forward` 以及 `net.ipv6.conf.all.forwarding` sysctl 选项（参见 *ip-sysctl.txt* 了解 sysctl 的选项）。  
    默认为 `no`。  

    注意：这个选项公职了一个全局的内核选项，因此它是一个单向设置：若一个网络启用了这个设置，则全局设置即被启用；但是，即便所有开启该选项的网络之后被再次关闭，它也不会被关闭。

    要让 IP 网络包转发仅发生在特定的网络界面，请使用防火墙。

- `IPMasquerade=`

    在网络界面上配置 IP 伪装（IP masquerading）。若启用，则从网络界面传入的网络包看起来就像是从本地主机传来的一样。  
    接受一个布尔值。隐含了 `IPForward=ipv4`。  
    默认为 `no`

- `IPv6PrivacyExtensions=`

    配置随时间变化的无状态临时地址（stateless temporary addresses）（参见 *RFC 4941, Privacy Extensions for Stateless Address Autoconfiguration in IPv6*）。  
    接受一个布尔值，或特定值 `perfer-public` `kernel`。当为真时，启用隐私扩展，且相对于公共地址，倾向于临时地址。设置为 `prefer-public` 时，启用隐私扩展，单相对于临时地址，倾向于公共地址。当为假时，隐私扩展保持关闭。当为 `kernel` 时，将不改变内核的默认设置。  
    默认为 `no`

- `IPv6AcceptRA=`

    接受一个布尔值。控制一个界面的 IPv6 路由通告接收（Router Advertisement）支持。若为真，则接受 RA；若为假，则 RA 被忽略，与本地的转发状态相互独立。当 RA 被接受，且 RA 中设置了相关的 flag，或者链路上没有发现路由，则它们可以触发 DHCPv6 客户端的启动。

    更多 IPv6 RA 支持可在 `[IPv6AcceptRA]` 段，参见下文。

    同时参见内核文档 *ip-sysctl.txt* 的 `accept_ra`，注意 systemd 设置为 1（也即为真），等价于内核设置 2。

    注意，内核实现的 IPv6 RA 协议总是被关闭的，无论这个设置是什么。若这个设置被启用了，则一个用户空间实现的 IPv6 RA 协议将被启用，而内核实现的协议依旧保持关闭。由于 **systemd-networkd** 需要知道通告的所有细节，但若使用内核的实现，则它们是不可以被获取的。

- `IPv6DuplicateAddressDetection=`

    配置 IPv6 重复地址检测探针（Duplicate Address Detection, DAD）的发送数量。当未设置时，将使用内核默认值。

- `IPv6HopLimit=`

    配置 IPv6 跳数限制（Hop Limit）。对于每个转网络包的路由，跳数限制就减 1。当跳数限制到达 0 时，网络包即被丢弃。当未设置时，使用内核默认值。

- `IPv4ProxyARP=`

    接受一个布尔值。配置 IPv4 的 ARP 代理（proxy ARP）。APR 代理是一项技术，一个主机，通常未路由器，回复了一个应该由另一个机器回复的 APR 请求。使用了对它的身份进行“造假”，路由器接受了将网络包转发至“真实的”最终地址的任务（参见 *RFC 1027*）。当未设置时，使用内核的默认值。

- `IPv6ProxyNDP=`

    接受一个布尔值。配置 IPv6 的 NDP 代理（proxy NDP）。NDP（Neighbor Discovery Protocol 邻居发现协议）代理，是一项技术，在 IPv6 中，允许将地址路由至另一个不同的地址，当同行希望它们出现在一个特定的物理链接上。在这种情况下，一个路由将用它自己的 MAC 地址作为最终地址，回答应该由另一台机器回复的邻居通告（Neighbour Advertisement）。与 IPv4 的 ARP 代理，它不是全局启用的，仅会对 IPv6 邻居代理表中的地址发送邻居通告，可以通过 `ip -6 neighbour show proxy` 展示出来。**systemd-networkd** 将通过该选项控制每个已配置界面的 `proxy_ndp` 开关。当未设置时，使用内核默认值。

- `IPv6ProxyNDPAddress=`

    一个 IPv6 地址，该地址的的邻居通告报文将被代理。该选项可多次声明。**systemd-networkd** 将会将 `IPv6ProxyNDPAddress=` 的条目添加至 IPv6 邻居代理表中。  
    该选项将隐含 `IPv6ProxyNDP=yes`，但若 `IPv6ProxyNDP` 被设置未假，则该设置无效果。  
    当未设置时，使用内核默认值。

- `IPv6PrefixDelegation=`

    开启或关闭一个链路上的路由宣告的发送。接受的值为：  
    `static`：发送在 `[IPv6PrefixDelegation]` 段 或者 `[IPv6Prefix]` 段 定义的前缀值  
    `dhcpv6`：需要使用为另一个链路配置的 DHCPv6 客户端的前缀，或者任何在 `[IPv6PrefixDelegation]` 段配置的、除了所有静态前缀配置的部分  
    `yes`：同时使用静态配置以及 DHCPv6  
    `no`：关闭 IPv6 前缀委派（prefix delegation）

    参见 `[IPv6PrefixDelegation]` 段 以及 `[IPv6Prefix]` 段 获得更多配置选项

- `IPv6MTUBytes=`

    配置 IPv6 最大传输单元（MTU）。应为大于等于 1280 bytes 的整数。  
    当未设置时，使用内核默认值。

- `Bridge=`

    该链路加入的 网桥 的名称，参见 *systemd.netdev(5)*

- `Bond=`

    该链路加入的 绑定 的名称，参见 *systemd.netdev(5)*

- `VRF=`

    该链路加入的 虚拟路由和转发 的名称，参见 *systemd.netdev(5)*

- `VLAN=`

    该链路上创建的 虚拟局域网 的名称，参见 *systemd.netdev(5)*。  
    该值可多次指定。

- `IPVLAN=`

    该链路上创建的 IPVLAN 的名称，参见 *systemd.netdev(5)*。  
    该值可多次指定。

- `MACVLAN=`

    该链路上创建的 MACVLAN 的名称，参见 *systemd.netdev(5)*。  
    该值可多次指定。

- `VXLAN=`

    该链路上创建的 虚拟局域网扩展 的名称，参见 *systemd.netdev(5)*。  
    该值可多次指定。

- `Tunnel=`

    该链路上创建的 隧道 的名称，参见 *systemd.netdev(5)*。  
    该值可多次指定。

- `MACsec=`

    该链路上创建的 MAC 安全 设备 的名称，参见 *systemd.netdev(5)*。  
    该值可多次指定。

- `ActiveSlave=`

    接受一个布尔值。指定一个新的活动从设备。  
    活动从设备仅可在下述模式下指定：`active-backup` `balace-alb` `balace-tlb`。  
    默认为假。

- `PrimarySlave=`

    接受一个布尔值。指定那个从设备为主设备。  
    当指定设备在线时，即成为活动从设备。仅当主设备离线时，才切换至其余的设备。该选项在偏好指定的设备时很有用，比如某个设备比另一个设备具有更高吞吐量时。主设备选项仅可在下述模式下指定：`active-backup` `balance-alb` `balance-tlb`。  
    默认为假。

- `ConfigureWithoutCarrier=`

    接受一个布尔值。允许 **networkd** 在一个链路没有承载者的情况下，也能配置它。  
    默认为假。

- `IgnoreCarrierLoss=`

    接受一个布尔值。允许 **networkd** 在一个界面丢失承载者的时候依旧保持其静态与动态配置。  
    默认为假。

- `Xfrm=`

    该链路上创建的 `xfrm` 的名称，参见 *systemd.netdev(5)*。  
    该值可多次指定。

- `KeepConfiguration=`

    接受一个布尔值或 `static` `dhcp-on-stop` `dhcp`。  
    当设置为 `static` 时，**systemd-networkd** 不会在启动流程中丢弃静态地址以及静态路由。  
    当设置为 `dhcp-on-stop` 时，**systemd-networkd** 不会在停止服务时丢弃地址和路由。  
    当设置为 `dhcp` 时，由 DHCP 服务器提供的地址和路由将永不丢弃，即便 DHCP 租期已经过期。这和 DHCP 规范相违背，但也许是最好的选择，比如，根文件系统依赖该连接。  
    设置 `dhcp` 隐含了 `dhcp-on-stop`，`yes` 隐含了 `dhcp` 和 `static`。  
    默认为 `no`。

## `[Address]` 段选项

一个 `[Address]` 段接受下方的选项。定义数个 `[Address]` 段 来配置多个地址。

- `Address=`

    和 `[Network]` 段一样，这个键是必须存在的。每个 `[Address]` 段可以包含一个 `Address=` 设置。

- `Peer=`

    在点对点连接中，对方的地址。接受与 `Address=` 键相同的格式。

- `Broadcast=`

    广播地址，必须按照 *inet_pton(3)* 描述的格式。该键仅运用于 IPv4 地址。若未指定，则从 `Address=` 中推断。

- `Label=`

    一个地址标签

- `PreferredLifetime=`

    允许覆盖地址的默认 “推荐生存期”（"preferred lifetime"）。仅接受三种设置：`forever` 或者 `infinity` 是默认值，它意味着地址永不过期，`0` 表示地址被认为是立刻 “过期的”，不会被使用，除非明确要求。设置 `PreferredLifetime=0` 常用于某地址仅被用于特定的应用，它们之后应该被显式地配置使用。

- `Scope=`

    地址地作用域，可以为 `global` `link` `host`，或一个介于 `0` 至 `255` 之间地无符号整型。  
    默认为 `global`。

- `HomeAddress=`

    接受一个布尔值。指明该地址为 *RFC 6275* 中的 “home address”。仅在 IPv6 中支持。默认为假。

- `DuplicateAddressDetection=`

    接受一个布尔值。在添加该地址是，不进行 *RFC 4862* 中的重复地址检测（Duplicate Address Detection）。仅在 IPv6 上支持。默认为假。

- `ManageTemporaryAddress=`

    接受一个布尔值。若为真，则内核将该处创建的临时地址作为 *Privacy Extensions RFC 3041* 的模板地址。要让它活跃，则 sysctl 的 `use_tempaddr` 必须设置为大于 0 的值。给出的地址需要具有长度为 64 的前缀。这个 flag 允许在手动配置的网络中使用隐私扩展，就如同启用了 无状态自动配置（stateless auto-configuration）。默认为假。

- `PrefixRoute=`

    接受一个布尔值。当添加或修改一个 IPv6 地址时，用户空间的应用需要一个抑制添加前缀路由（prefix route）的方法。举例来说，与 `IFA_F_MANAGERTEMPADDR` 相关的，用户空间创建的 autoconf 生成的地址，但根据 on-link，不因该添加前缀路由。默认为否。

- `AutoJoin=`

    接受一个布尔值。如果以太网交换机执行 **IGMP snooping**，则无法通过 `ip maddr` 命令在以太网层级加入多播组，因为交换机不会复制不具有多播地址的 IGMP 报文 的端口上的多播网络包。由命令 `ip link add vxlan` 创建的 Linux vxlan 界面，或者启用了对应组选项的 **networkd** 的 **netdev** 类型的 vxlan，可以满足加入需求。使用扩展了 `autojoin` 选项的 ip 地址命令，我们就可以为 openvswitch (OVS) 的 vxlan 以及其它需要接收 多播流量的 隧道机制 提供相似的功能。默认为 `no`。

## `[Neighbor]` 段选项

`[Neighbor]` 段接受下列键。一个邻居段向 IPv6 的邻居表（neighbor table）或 IPv4 的 ARP 表 中加入一行永久、静态的条目，该条目为该链路上与网络匹配的给定的硬件地址。指定多个 `[Neighbor]` 段来配置多个静态邻居。

- `Address=`

    邻居的 IP 地址。

- `LinkLayerAddress=`

    邻居的链路层地址（MAC 地址 或者 IP 地址）。

## `[IPv6AddressLabel]` 段选项

一个 `[IPv6AddressLabel]` 段接受下方键。定义数个 `[IPv6AddressLabel]` 段来配置数个地址标签。IPv6 地址标签用于地址选择（address selection）。参见 *RFC 3484*。 Precedence 在用户空间控制， 仅标签本身存储在内核中。

- `Label=`

前缀的标签是一个 `0` 至 `4294967294` 的无符号整型。 `0xffffffff` 为保留数值。该键必须存在。

- `Prefix=`

IPv6 前缀是一个具有前缀长度的地址，用 `/` 隔开。该键必须存在。

## `[RoutingPolicyRule]` 段选项

一个 `[RoutingPolicyRule]` 接受下列键。指定多个 `[RoutingPolicyRule]` 来配置多条规则。

- `TypeOfService=`

    指定 `0` 至 `255` 的一个数字，该数字用于匹配一个的服务类型。

- `From=`

    指定要匹配的源地址前缀。可能会跟随一个斜线以及前缀的长度。

- `To=`

    指定要匹配的目标地址的前缀。可能会跟随一个斜线以及前缀的长度。

- `FirewallMark=`

    指定要匹配的 **iptables firewall mark** 值 （该值介于 `1` 和 `4294967295`）。

- `Table=`

    指定若匹配上 **rule selector**，则需要用于查找的 **routing table identifier**。接受下列值： `default` `main` `local`， 或一个介于 `1` 至 `4294967295` 之间的数字。  
    默认为 `main`。

- `Priority=`

    指定该规则的优先级。 `Priority=` 为一个无符号整型。数字越大优先级越低，所有规则按照数字的升序方向处理。

- `IncomingInterface=`

    指定要匹配的来源设备。若界面为回环（loopback），则仅匹配来自本主机的网络包。

- `OutgoingInterface=`

    指定要匹配的外流（outgoing）设备。外流界面仅针对从绑定在某设备上的本地套字节发出的包裹。

- `SourcePort=`

    指定匹配 **forwarding information base (FIB)** 规则的源 IP 端口或端口段。一个端口段通过 小端口、横线、大端口 的格式来指定。  
    默认为未设置。

- `DestinationPort=`

    指定匹配 **forwarding information base (FIB)** 规则的目标 IP 端口或端口段。一个端口段通过 小端口、横线、大端口 的格式来指定。  
    默认为未配置。

- `IPProtocol=`

     指定匹配 **forwarding information base (FIB)** 规则的 IP 协议。接受 IP 协议名，比如 `tcp` `udp` `sctp`，或者 IP 协议号，比如 `6`（对应 `tcp`） `17`（对应 `udp`）。  
     默认为未配置。

- `InvertRule=`

    接受布尔值。是否反转该规则。  
    默认为假。

- `Family=`

    接受 `ipv4` `ipv6` `both`。  
    默认情况下， **address family** 由 `To=` 或 `From=` 的地址决定。若未指定 `To=` 和 `From=`，则默认为 `ipv4`。

## `[NextHop]` 段选项

`[NextHop]` 段接受下列键。指定多个 `[NextHop]` 段来配置多个下一跳。下一跳用来操控内核的下一跳表的条目。

- `Gateway=`

    与 `[Network]` 段中的一致。该键为必须键。

- `Id=`

    下一跳的 id（一个无符号整型）。若未指定或设置为 `0`，则由内核自动选择。

## `[Route]` 段选项

`[Route]` 段接受下列键。指定多个 `[Route]` 段来配置多个路由。

- `Gateway=`

    如 `[Network]` 段所描述的。

- `GatewayOnLink=`

    接受布尔值。若设置为真，则内核不必检测网关是否真的直连至该设备（比如，内核不必检测网关是否连入了本地网络），我们即可以向内核的路由表插入条目，同时抑制内核针对该条目的警报。  
    默认为假。

- `Destination=`

    目标路由的前缀。可能紧跟一个 `/` 以及前缀的长度。若不指定，则架设为全长（full-length）主机路由。

- `Source=`

    源路由的前缀。可能紧跟一个 `/` 以及前缀的长度。若不指定，则架设为全长（full-length）主机路由。

- `Metric=`

    路由的 **metric**（一个无符号整型）。

- `IPv6Preference=`

    指定 路由偏好的（如 *RFC4191* 中所描述的）**Router Discovery** 报文。可以为 `low` 该路由具有最低优先级，`medium` 该路由具有默认偏好，`high` 该路由具有最高偏好。

- `Scope=`

    路由的作用域，可以为 `global` `site` `link` `host` `nowhere`。  
    对于 IPv4 路由，若 `Type=` 为 `local` 或 `nat`，则默认为 `host`；  
    若 `Type=` 为 `broadcast` 或 `multicast` 或 `anycast`，则默认为 `link`；
    其他情况默认均为 `global`。

- `PreferredSource=`

    该路由偏好的源地址。该地址必须符合 *inet_pton(3)* 中的描述。

- `Table=`

    该路由的 **table identifier**，接受 `default` `main` `local` 或介于 `1` 至 `4294967295` 之间的数。  
    这张表可以用 `ip router show table <num>` 来获取。  
    若未设置，且 `Type=` 为 `local` `broadcast` `anycast` `anycast` `nat`，则值为 `local`。  
    其他情况的默认值为 `main`。

- `Protocol=`

    该路由的 **protocol identifier**，接受 `0` 至 `255` 之间的数，或特定的值 `kernel` `boot` `static` `ra` `dhcp`。  
    默认值为 `static`。

- `Type=`

    指定该路由的类型。接受 `unicast` `local` `broadcast` `anycast` `multicast` `blackhole` `unreachable` `prohibit` `throw` `nat` `xresolve`。  
    若为 `unicast`，则定义一条普通路由，例如，一个指明了通向目标网络地址路径的路由。  
    若为 `blackhole`，导向该路由的网络包将被静默丢弃。  
    若为 `unreachable`，导向该路由的网络包将被丢弃，并生成 **ICMP** 报文 **Host Unreachable**。  
    若为 `prohibit`，导向该路由的网络包将被丢弃，并生成 **ICMP** 报文 **Communication Administratively Prohibited**。  
    若为 `throw`，当前路由表的路由查找将失败，路由选择流程将返回 路由策略数据库（**Routing Policy Database**, **RPDB**）。  
    默认为 `unicast`。

- `InitialCongestionWindow=`

    在启动 TCP 连接时，所要使用的 TCP 初始拥塞窗口（congestion window）。在一个 TCP 会话的初始阶段，当一个客户端请求一个资源时，服务器的初始拥塞窗口决定了在初始数据传送中需要发送多少字节。  
    接受的字节大小介于 `1` 与 `4294967295`(2^32-1)之间。支持常用后缀 `K` `M` `G`，且底数为 `1024`。  
    若未设置，将使用内核的默认值。

- `InitialAdvertisedReceiveWindow=`

    TCP 初始提议接收窗口是 在一个连接上 初始化的可以缓存的接收数据（以字节计）的数量。在等待来自接收端的 ACK 和窗口更新之前，发送方主机仅可以发送的的数据量。  
    接受的字节大小介于 `1` 与 `4294967295`(2^32-1)之间。支持常用后缀 `K` `M` `G`，且底数为 `1024`。  
    若未设置，将使用内核的默认值。

- `QuickAck=`

    接受一个布尔值。  
    当为真时，启用该路由的 TCP **quick ack mode**。  
    若未设置，将使用内核默认值。

- `FastOpenNoCookie=`

    接受一个布尔值。  
    当为真时，启用无需基于每路由 **cookie** 的 TCP **fastopen**。  
    若未设置，将使用内核默认值。

- `TTLPropagate=`

    接受一个布尔值。  
    当为真时，启用 **TTL propagation at Label Switched Path (LSP) egress**。  
    当未设置时，使用内核默认值。

- `MTUBytes=`

    以字节为单位为路由设置最大传输单元。支持常用后缀 `K` `M` `G`，且底数为 `1024`。

    注意，若 IPv6 在界面上启用，且选择的 MTU 值小于 `1280`（IPv6 最小 MTU），都会将该值设置为 `1280`。

- `IPServiceType=`

    接受字符串 `CS6` 或 `CS4`。  
    用于设置 IP 服务类型至 **CS6（网络控制）**，或 **CS4（实时）**。  
    默认为 `CS6`。

## `[DHCPv4]` 段选项

若在 `DHCP=` 选项中配置了，则在 `[DHCPv4]` 段配置 DHCPv4 客户端：

- `UseDNS=`

    若为真（默认值），则来自 DHCP 服务器的 DNS 服务器将优先于任何静态配置的 DNS 服务器。

    该选项对应 **resolv.conf(5)** 的 **nameserver** 选项。

- `RoutesToDNS=`

    若为真（默认值），则接受来自 DHCP 服务器的通向 DNS 服务器的路由。当 `UseDNS=` 被禁用，则该设置被忽略。默认为假。

- `UseNTP=`

    若为真（默认值），则相较于任何静态配置的 NTP 服务器，**systemd-timesyncd** 将优先使用来自 DHCP 服务器的 NTP 服务器。

- `UseSIP=`

    若为真（默认值），则来自 DHCP 服务器的 SIP 服务器将被保存至状态文件中，并可以通过函数 `sd_network_link_get_sip_server()` 读取。

- `UseMTU=`

    若为真，则当前链路界面的最大传输单元将使用来自 DHCP 服务器的值。若设置了 `MTUBytes=`，则忽略该设置。默认为假。

- `Anonymize=`

    接受布尔值。若为真，则向 DHCP 服务器发送的选项将遵循 *RFC 7844 (Anonymity Profiles for DHCP Clients)* 来最小化可以透露出身份的信息。默认为假。

    当 `MACAddressPolicy=` 设置为 `random` 时（参见 *systemd.link(5)*）,该值才应该被设置为真。

    注意，该选项会覆盖其他选项。具体来说，下列变量将被忽略：`SendHostname=` `ClientIdentifier=` `UseRoutes=` `SendHostname=` `UseMTU=` `VendorClassIdentidier=` `UseTimezone=`

    启用了该选项会让 DHCP 请求模仿从 **Microsoft Windows** 中发送的请求，来减少分辨不同机器的能力。这意味着 DHCP 请求大小将增加，且租用数据将比通常更加全面，虽然大多数请求的数据并不会被用上。

- `SendHostname=`

    若为真（默认值），机器的主机名将被发送至 DHCP 服务器。注意机器的主机名只能包含 7-bit ASCII 小写字符，且不具有空格和点，且必须被格式化为有效的 DNS 域名。其他情况下，即便该值设置为真，主机名也不会被发送。

- `UseHostname=`

    若为真（默认值），来自 DHCP 服务器的主机名将作为系统的瞬态主机名（transient hostname）。

- `Hostname=`

    将该值作为主机名发送至 DHCP 服务器，而非机器的主机名。注意机器的主机名只能包含 7-bit ASCII 小写字符，且不具有空格和点，且必须被格式化为有效的 DNS 域名。

- `UseDomains=`

    接受一个布尔值，或者特定值 `route`。当为真时，来自 DHCP 服务器的域名将作为该链路的 DNS 搜索域名，与 `Domains=` 设置的效果类似。若设置为 `router`，则来自 DHCP 服务器的域名将仅用于路由 DNS 请求，但不用于搜索，与参数前缀了 `~` 的 `Domain=` 设置的效果类似。默认为假。

    建议仅在可信网络上启用，因为该选项将影响所有主机名，特别是单标签主机名的解析。通常来说，仅将提供的域名作为路由域名而非搜索域名是相对安全的，这样就不会影响单标签名在本地的解析。

    当设置为真时，该设置与 *resolve.conf(5)* 中的域名选项对应。

- `UseRoutes=`

    若为真（默认值），将从 DHCP 服务器请求静态路由，且该路由具有 **1024** 的 metric，且根据路由的目的地和网关，具有 **global**、**link**、**host** 三个不同的作用域。  
    若目的地在本地链路上，比如 **127.x.x.x**，或与链路自身地址相同，则作用域被设置为 `host`。其他情况，若网关为 null（直连路由），将使用 `link` 作用域。其他任何情况，作用域默认为 `global`。

- `UseTimezone=`

    若为真，则来自 DHCP 服务器的时区将作为本地系统的时区。默认为 `no`。

- `ClientIdentifier=`

    要使用的 DHCPv4 客户端标识（DHCPv4 client identifier）。  
    接受 `mac` `duid` `duid-only`。  
    若设置为 `mac`，则使用链路的 MAC 地址。  
    若设置为 `duid`，则使用符合 *RFC4361* 的用户 ID（RFC4361-compliant Client ID），该 ID 由 `IAID` 和 `DUID`（见下方）组合而成。  
    若设置为 `duid-only`，则仅使用 **DUID**，这可能并不符合 RFC，但有些设置需要这样。  
    默认为 `duid`。

- `VendorClassIdentifier=`

    制造商类别标识符，用来区分制造商类别和配置。

- `UserClass=`

    一个 DHCPv4 客户端可以用 `UserClass` 选项来区分它所代表的用户或应用的类别或分类。该选项包含的信息为一个字符串，该字符串表示了该客户端属于哪个用户类别。每个类别都设置了一个 DHCP server 用来区分用户的区分信息字符串。接受一个白空格区分的字符串列表。

- `MaxAttempts=`

    指定要尝试多少次 DHCPv4 客户端配置。接受一个数字，或者 `infinity`。默认为 `infinity`。注意重试的间隔时间是指数增长的，这样即便该数字很大也不会产生网络过载。

- `DUIDType=`

    覆盖全局 **DUIDType** 设置在该网络上的设置。参见 *networkd.conf(5)* 了解可能值的描述。

- `DUIDRawData=`

    覆盖全局 **DUIDRawData** 设置在该网络上的设置。参见 *networkd.conf(5)* 了解可能值的描述。

- `IAID=`

    该界面的 DHCP 身份相关标识符（**DHCP Identity Association Identifier**, **IAID**），是一个 32-bit 的无符号整型。

- `RequestBroadcast=`

    在 IP 地址配置前，要求服务器使用广播报文。这对于不可以接受 RAW 网络包的设备，或者在配置了一个 IP 地址之前根本无法接收网络包的设备是必须的。  
    从另一方面说，该选项在广播网络包被过滤掉的网络中不允许被启用。

- `RouteMetric=`

    为来自 DHCP 服务器的路由设置路由 metric。

- `RouteTable=`

    DHCP 路由的表标识（介于 `1` 至 `4294967295` 之间的一个数，或用 `0` 表示未设置）。该表可以用 `ip route show table <num>` 来获取。

    当与 `VRF=` 连用时，除非该参数被指定，否则使用 VFR 的路由表。

- `ListenPort=`

    允许 DHCP 客户端监听自定义的端口。

- `SendRelease=`

    若为真，则 DHCPv4 客户端在停止时发送 DHCP 释放网络包（DHCP release packet）。默认为真。

- `BlackList=`

    白空格隔开的 IPv4 地址列表。若来自 DHCP 服务器的地址在该列表中，则被拒绝。

- `RequestOptions=`

    白空格隔开的列表，由 `1` 至 `254` 之间的整型组成。

- `SendOption=`

    在 DHCPv4 请求中发送一个任意的选项。接受一个用冒号隔开的 DHCP 选项号、数据类型和数据（`option:type:value`）。选项号必须是 `1` 至 `254` 之间的整型。类型必须是 `uint8` `uint16` `uint32` `ipv4address` 或 `string`。在数据字符串中的特殊字符可以用 C-type 转义字符表示。该设置可多次指定。若指定了空字符，则前序指定的所有选项都将清空。默认为未设置。

## `[DHCPv6]` 段选项

若在 `DHCP=` 中启用或被 **IPv6 Router Advertisement** 调用，则 `[DHCPv6]` 段配置 DHCPv6 客户端：

- `UseDNS=`, `UseNTP=`

    同 `[DHCPv4]` 段中的描述。

- `RapidCommit=`

    接受一个布尔值。DHCPv6 客户端可以通过 快速双报文交换（rapid two-message exchange）（征求与答复（solicit and reply））从 DHCPv6 服务器获取配置参数。当同时在 DHCPv6 客户端和服务器开启 **rapid commit** 选项，将使用双报文交换，而非默认的四方法交换（four-method exchange）（征求、宣告、请求和答复（solicit, advertise, request, and reply））。双报文交换提供快速的客户端配置，对网络负载较重的情形比较有利。参见 *RFC 3315* 了解详情。默认为真。

- `ForceDHCPv6PDOtherInformation=`

    接受一个布尔值，当设置了路由通告报文中的 `Other information` bit 时，也强制 DHCPv6 运行在有状态模式。默认情况下，若路由通告报文仅设置了 `Other information` bit，则 DHCPv6 以无状态行为请求网络信息，且使用 双报文信息请求与信息回复报文交换（two-message Information Request and Information Reply message exchange）。在 *RFC 7084* 中，需要 **WPD-4**，对自定义边界路由（Customer Edge router）更新了行为，这样有状态的 DHCPv6 前缀委托（DHCPv6 Prefix Delegation），在路由通告仅设置了 `Other information` 时也会一并请求。该选项启用了 CE 行为，区分了其他情况下不可能自动区分 `Other information` 的意图。默认该选项为假，在设备应该作为 CE 路由，却没有前缀被委托的情况下开启。

- `PrefixDelegationHint=`

    接受一个带前缀长度的 IPv6 地址，就如 `[Network]` 段中的 `Address=` 一样。指定 DHCPv6 客户端请求路由器将前缀提示包含进 DHCPv6 征求。前缀范围 `1` 至 `128`。默认为未设置。

## `[IPv6AcceptRA]` 段选项

若在 `IPv6AcceptRA=` 中启用了，则在 `[IPv6AcceptRA]` 段配置 IPv6 路由宣告（RA）客户端：

- `UseDNS=`

    若为真（默认值），从路由宣告中获取的 DNS 服务器将优先于任何静态配置的 DNS 服务器。

    该选项对应 *resolve.conf(5)* 中 名称服务器选项。

- `UseDomains=`

    接受一个布尔值，或特别的值 `route`。当为真时，通过 IPv6 路由宣告（RA）获取的域名将作为该链路的 DNS 搜索域名，与 `Domains=` 设置的效果类似。若设置为 `router`，则通过 IPv6 路由宣告（RA）获取的域名将仅用于路由 DNS 请求，但不用于搜索，与参数前缀了 `~` 的 `Domain=` 设置的效果类似。默认为假。

    建议仅在可信网络上启用，因为该选项将影响所有主机名，特别是单标签主机名的解析。通常来说，仅将提供的域名作为路由域名而非搜索域名是相对安全的，这样就不会影响单标签名在本地的解析。

    当设置为真时，该设置与 *resolve.conf(5)* 中的域名选项对应。

- `RouteTable=`

    路由宣告发送路由的表标识（介于 `1` 至 `4294967295` 之间的一个数，或用 `0` 表示未设置）。该表可以用 `ip route show table <num>` 来获取。

- `UseAutonomousPrefix=`

    若为真（默认值），从路由宣告获取的自治前缀（autonomous prefix）将优先于任何静态配置的自治前缀。

- `UseOnLinkPrefix=`

    若为真（默认值），从路由宣告获取的在链路前缀（onlink prefix）将优先于任何静态配置的在链路前缀。

- `BlackList=`

    白空格分隔的 IPv6 前缀列表。若通过路由宣告获取的 IPv6 前缀落在该列表中，就被忽略。

## `[DHCPServer]` 段选项

若在上面的 `DHCPServer=` 启用了，则在 `[DHCPServer]` 段配置 DHCP 服务器：

- `PoolOffset=`, `PoolSize=`

    配置要分发的地址池。地址池为一段连续的 IP 地址序列，该 IP 地址序列应该在为服务器配置的子网网段中，且不包含子网地址和广播地址。  
    `PoolOffset=` 定义了地址池的起始位置，该起始位置以从子网开头的偏移量计算，或者设置为 `0` 来使用默认值。  
    `PollSize=` 定义了该池中 IP 地址的数量，或者设置为 `0` 来使用默认值。  
    默认情况下，地址池从子网号之后的第一个地址开始，直到用尽除了该子网广播地址之外的所有地址。若地址池包含了服务器地址（默认值），则该地址将会被保留，不会分发给客户端。

- `DefaultLeaseTimeSec=`, `MaxLeaseTimeSec=`

    控制要传递给客户端的 默认 和 最大 DHCP 租期。这些选项使用秒做单位，或者其它通用的时间单位，以后缀为准。默认租期用于客户端未指定特定的租期时使用。若客户端要求的租期长于最长租期，则仅提供最长租期的时间。默认的租期为 1h，默认的最大租期为 12h。若 DHCP 租用配置数据频繁的变动，需要客户端时常学习新设置时，可以调低租期。使用较长的租期来减少生成的 DHCP 网络流量。

- `EmitDNS=`, `DNS=`

    接受一个布尔值。配置分发给客户端的 DHCP 租用是否需要包含 DNS 信息。默认为 `yes`。可以通过 `DNS=` 选项配置将要发送给客户端的 DNS 服务器地址，该地址由一系列 IPv4 地址组成。若 `EmitDNS=` 启用，却没有配置 DNS 服务器，则服务器会从具有正确服务器配置的“上行”（"uplink"）界面中传播至客户端。“上行”界面由系统中具有最高优先级的默认路由决定。  
    注意，该信息在该租用分发时获取，并不会将其后了解到的 DNS 或 NTP 服务器涵盖在内。DNS 服务器传播不会将 **/etc/resolv.conf** 的内容网络包括在内。  
    同时也要注意，即便上行网络的配置改变，分发也不会改变。为了保证客户端收到最新的上行 DNS 服务器信息，最好通过上文描述 `MaxLaeseTimeSec=` 缩短 DHCP 租期/。

- `EmitNTP=`, `NTP=`

    与上文描述的 `EmitDNS=` 与 `DNS=` 设置类似，这些设置配置了 NTP 服务器信息是否应该随着 DHCP 租用一同发送。与 `EmitDNS=` 和 `DNS=` 具有相同的语法、传播语义和默认值。

- `EmitSIP=`, `SIP=`

    与上文描述的 `EmitDNS=` 与 `DNS=` 设置类似，这些设置配置了 SIP 服务器信息是否应该随着 DHCP 租用一同发送。与 `EmitDNS=` 和 `DNS=` 具有相同的语法、传播语义和默认值。

- `EmitRouter=`

    与上文描述的 `EmitDNS=` 设置类似，这些设置配置了 路由选项 是否应该随着 DHCP 租用一同发送。与 `EmitDNS=` 具有相同的语法、传播语义和默认值。

- `EmitTimezone=`, `Timezone=`

    接受一个布尔值。这些设置配置了 时区信息 是否应该随着 DHCP 租用一同发送。默认值为 `yes`。`Timezone=` 设置接受一个时区字符串（比如 `Europe/Berlin` 或 `UTC`）用来传递给客户端。若没有设置明确的时区，则传递本地主机的系统时区设置，该设置由 /etc/localtime 符号链接决定。

- `SendOption=`

    通过 DHCPv4 服务器发送一个裸选项。接受一个 DHCP 选项号、一个数据类型和一个数据（"option:type:value"）。选项号是一个介于 `1` 至 `254` 之间的数。数据类型接受 `uint8` `uint16` `uint32` `ipv4address` `string`。数据字符串中的特殊字符可以用 C-type 转义字符转义。该选项可以多次指定。若指定了空字符串，则清楚前序指定的选项。默认为未设置。

## `[IPv6PrefixDelegation]`

若在上面的 `IPv6PrefixDelegation=` 启用了，则 `[IPv6PrefixDelegation]` 包含了发送 IPv6 路由宣告的设置，以及是否作为路由出现。IPv6 网络前缀使用一个或多个 `[IPv6Prefix]` 段定义。

- `Managed=`, `OtherInformation=`

    接受一个布尔值。控制当 `Managed=` 设置为真时，是否使用 DHCPv6 服务器获取 IPv6 地址；或者当 `OtherInformation=` 设置为真时，通过 DHCPv6 为网络链接获取信息时，是否仅获取附加网络信息。两个设置默认均为假，意味着 DHCPv6 服务器不被使用。

- `RouterLifetimeSec=`

    接受一个时间值。以秒为单位配置 IPv6 路由的生命周期（lifetime）。若设置，则该主机也在路由宣告中声明他自己为该网络链接的一个 IPv6 网络。当未设置时，主机并不会作为一个路由。

- `RouterPreference=`

    若 `RouterLifetimeSec=` 不为零，则配置 IPv6 路由选项。有效值为 `high` `medium` `low`，也可以使用 `normal` 和 `default` 表示 `medium`。参见 *RFC 4191* 了解详情。默认值为 `medium`。

- `EmitDNS=`, `DNS=`

    当 `EmitDNS=` 为真时，通过路由宣告报文发送由 `DNS=` 指定一个列表的递归式的 DNS 服务器的 IPv6 地址。若 `DNS=` 为空，则 DNS 服务器从 `[Network]` 段读取。若 `[Network]` 段同样未包含任何 DNS 服务器，从上行链路中具有最高优先级的默认路由中获取 DNS 服务器。当 `EmitDNS=` 为假时，DNS 服务器信息不随路由宣告报文一同发送。`EmitDNS=` 默认为真。

- `EmitDomains=`, `Domains=`

    当 `EmitDomains=` 未真时，通过路由宣告报文发送的 DNS 搜索域名列表。若 `Domains=` 未空，则 DNS 搜索域名从 `[Network]` 段读取。若 `[Network]` 段同样未包含任何 DNS 搜索域名，从上行链路中具有最高优先级的默认路由中获取 DNS 搜索域名。当 `EmitDomain=` 为假时，DNS 搜索域名信息不随路由宣告报文一同发送。`EmitDNS=` 默认为真。

- `DNSLifetimeSec=`

    以秒为单位的 `DNS=` 列表中的 DNS 服务器，以及 `Domains=` 列表中的 搜索域名的生命周期（lifetime）。

## `[IPv6Prefix]` 段选项

一个或多个 `[IPv6Prefix]` 段包含了将通过路由宣告发送的 IPv6 前缀。参见 *RFC 4861* 了解更多细节。

- `AddressAutoconfiguration=`, `OnLink=`

    接受一个布尔值，指定 IPv6 地址是否能通过该前缀自动配置；或者该前缀是否能用于在链路判定（onlink determination）。两个设置默认均为真，来简化配置。

- `Prefix=`

    将要分发的 IPv6 前缀。与配置静态 IPv6 地址类似，该设置配置为一个 IPv6 前缀、一个前缀长度，以及期间的分隔符 `/`。由于每个前缀的生命周期（lifetime）、地址自动配置和在链路状态都不尽一样，需要使用多段 `[IPv6Prefix]` 段配置多个 IPv6 前缀。

- `PreferredLifetimeSec=`, `ValidLifetimeSec=`

    以秒计为单位的 偏好 以及 有效 的前缀生命周期。`PreferredOLifetimeSec=` 默认值为 `604800` 秒（一周），`ValidLifetimeSec=` 默认为 `2592000` 秒（30 天）。

## `[IPv6RoutePrefix]` 段选项

一个或多个 `[IPv6Prefix]` 段包含了将通过路由宣告发送的 IPv6 前缀路由。参见 *RFC 4191* 了解更多细节。

- `Route=`

    将要发送的 IPv6 路由。与配置静态 IPv6 路由类似，该设置配置为一个 IPv6 前缀路由、一个前缀路由长度，以及期间的分隔符 `/`。使用多个 `[IPv6PrefixRoutes]` 段配置多个 IPv6 前缀路由。

- `LifetimeSec=`

以秒计算的路由前缀的生命周期（lifetime）。`LifetimeSec=` 默认为 `604800` 秒（一周）。

## `[Bridge]` 段选项

`[Bridge]` 接受下列键：

- `UnicastFlood=`

    接受一个布尔值。控制 当一个 **转发数据库**（FDB）条目缺失，且通过该端口无法得知目标地址时，网桥是否应将流量泛洪（flood traffic）。当未设置时，使用内核默认值。

- `MulticastFlood=`

    接受一个布尔值。控制 当一个 **多播组数据库**（MDB）条目缺失，且通过该端口无法得知目标地址时，网桥是否应将流量泛洪（flood traffic）。当未设置时，使用内核默认值。

- `MulticastToUnicast=`

    接受一个布尔值。**多播至单播**（multicast to unicast）基于网桥的 **多播嗅探**（multicast snooping）功能。这意味着单播副本将仅发送至对其感兴趣的主机上。当未设置时，使用内核默认值。

- `NeighborSuppression=`

    接受一个布尔值。配置是否在该端口上抑制 ARP 和 ND 邻居（ND neighbour）。当未设置时，使用内核默认值。

- `Learning=`

    接受一个布尔值。配置是否在该端口上启用 MAC 地址学习。当未设置时，使用内核默认值。

- `HairPin=`

    接受一个布尔值。配置是否将网络包从接收它的端口上发送回去。当该 flag 为假时，网桥不会将转发流量传回接收的端口。当未设置时，使用内核的默认值。

- `UseBPDU=`

    接受一个布尔值。配置网桥端口是否要处理 **生成树网桥协议数据单元**（STP Bridge Protocol Data Units）。当未设置时，使用内核的默认值。

- `FastLeave=`

    接受一个布尔值。该 flag 允许网桥器从一个端口处接收到一个 **IGMP Leave** 报文后，立刻停止该端口的多播流量。仅在该网桥上启用了 **IGMP 嗅探** 时有效。当未设置时，使用内核的默认值。

- `AllowPortToBeRoot=`

    接受一个布尔值。配置一个端口是否允许成为根端口（root port）。仅当在网桥上启用了 **生成树协议**（STP） 时有效。当未设置时，使用内核的默认值。

- `ProxyARP=`

    接受一个布尔值。配置是否在该端口开启 ARP 代理。当未设置时，使用内核的默认值。

- `ProxyARPWiFi=`

    接受一个布尔值。配置 ARP 代理是否应该在该端口上启用，以满足 **IEEE 802.11** 和 **Hotspot 2.0** 规范的扩展要求。当未设置时，将使用内核的默认值。

- `MulticastRouter=`

    配置该端口为连接至多播路由的端口。具有多播路由的端口将接受所有的多播路由。使用 `no` 来关闭该端口的多播路由，`query` 让系统检测路由的存在，`permanent` 来在该端口永久开启多播流量转发，`temporary` 在该端口临时启用多播路由，而不依照输入的请求。当未设置时，使用内核的默认值。

- `Cost=`

    设置从该界面发送网络包的“花费”（"cost"）。网桥的每个端口可能具有不同的速度，花费被用于决定使用哪个链路。更快的端口应该具有更低的花费。该值为一个 `1` 至 `65535` 之间的数。

- `Priority=`

    设置在该界面上发送网络包的“优先级”（"priority"）网桥的每个端口可能具有不同的优先级，优先级被用于决定该使用哪个链路。更低的值具有更高的优先级。该值为介于 `0` 至 `63` 之间的整型。**Networkd** 不会设置任何默认值，也就是说将使用内核的默认值 `32`。

## `[BridgeFDB]` 段选项

`[BridgeFDB]` 段管理了一个端口的转发数据库表（forwarding database table），接受下列键。指定多个 `[BridgeFDB]` 段来配置多个静态 MAC 表条目。

- `MACAddress=`

    如 `[Network]` 段所描述的。该值必须存在。

- `Destination=`

    接受一个 目标 VXLAN 隧道端点（destinatino VXLAN tunnel endpoint）的 IP 地址。

- `VLANId=`

    新静态 MAC 表条目的 VLAN ID。若不指定，则不在新静态 MAC 表条目上追加 VLAN ID 信息。

- `VNI=`

    用来连接远程 VXLAN 隧道端点的 VXLAN 网络识别码（VXLAN Network Identifier）（或 VXLAN 段 ID（VXLAN Segment ID））。接受一个介于 `1` 至 `16777215` 之间的值。默认为未设置。

- `AssociatedWith=`

    指定该地址所关联的地方。接受 `use` `self` `master` `router`。  
    `use` 意味着该地址正被使用。用户空间可以使用该选项向内核指明该 fdb 条目正被使用。  
    `self` 意味着该地址与端口驱动的 fdb 相关联。通常为硬件。  
    `master` 意味着该地址与主设备 fdb 相关联。  
    `router` 意味着目标地址与一个路由相关联。注意，若指向的设别时一个 VXLAN 类型的设别，且启用了路由短路（route shortcircuit），则该项有效。  
    默认值为 `self`。

## `[CAN]` 段选项

`[CAN]` 段管理控制器局域网（Controller Area Network （CAN bus））并接受下方的键。

- `BitRate=`

    以 位每秒 为单位的 CAN 设备的字节率。常用的国际单位制前缀（`K` `M`）也可以使用，且其底数为 **1000**。

- `SamplePoint=`

    可选采样点，使用具有一个小数点的百分数（e.g. `70%`，`87.5%`）或千分数（e.g. `875‰`）

- `RestartSec=`

    自动重启的延迟时间。若设置为非零值，则在总线掉线后的指定时间之后，将自动触发 CAN 控制器的重启。次秒级的延时可以通过小数指定（e.g. `0.1s`）或者使用 `ms` `us` 后缀。使用 `infinity` 或者 `0` 将关闭自动重启。默认禁用自动重启。

- `TripleSampling=`

    接受一个布尔值。当为 `yes` 时，将使用三个采样（而非一个）中取众数的方法来决定一个收到的 bit 的值。当未设置时，使用内核的默认值。

## `[TrafficControlQueueingDiscipline]` 段选项

`[TrafficControlQueueingDiscipline]` 段管理流量控制（Traffic control）。它可以被用来配置内核的网络包调度器（packet scheduler），并为 UDP 或 TCP 应用模拟网络包延迟或丢失，或者限制特定服务的带宽来模拟互联网连接。

- `Parent=`

    指定父级队列规则（Queueing Discipline（qdisc））。接受 `root` 或 `clsact`。默认为 `root`。

- `NetworkEmulatorDelaySec=`

    指定要添加至从该网络界面发送的网络包需要延迟的固定时长。默认为未设置。

- `NetworkEmulatorDelayJitterSec=`

    指定要添加至从该网络界面发送的网络包需要延迟的选择时长（chosen delay）。默认为未设置。

- `NetworkEmulatorPacketLimit=`

    指定同一时间下，qdisc 可以持有的最大包裹数。一个介于 `0` 至 `4294967294` 之间的无符号整型。默认值为 1000。

- `NetworkEmulatorLossRate=`

    指定从该网络界面发送的网络包的独立丢包可能性。接受一个百分数，后缀以 `%`。默认为未设置。

- `NetworkEmulatorDuplicateRate=`

    指定在队列前要重复（duplicated）的包裹的选中百分比（chosen percent）。接受一个百分数，后缀以 `%`。默认为未设置。

## `[BridgeVLAN]` 段选项

`[BridgeVLAN]` 段管理网桥端口的 VLAN ID 的配置，接受下列的键。指定多个 `[BridgeVLAN]` 段来配置多个 VLAN 条目。`VLANFiltering=` 选项必须启用，参见 *systemd.netdev(5)* 中的 `[Bridge]` 段。

- `VLAN=`

    端口上允许的 VLAN ID。可以为一个单独的 ID，或者一个 M-N 的范围。VLAN ID 的有效值介于 `1` 至 `4094`。

- `EgressUntagged=`

    该处定义的 VLAN ID 将用于在网络帧离开时（on egress）移除标签（untag）。配置 `EgressUntagged=` 隐含了使用上文中的 `VLAN=`，并为传入（ingress）的网络帧启用 VLAN ID。可以为一个单独的 ID，或者一个 M-N 的范围。

- `PVID=`

    在该处定义的端口 VLAN ID（Port VLAN ID）将分配给所有传入的未标记的网络帧。`PVID=` 仅可以被使用一次。配置 `PVID=` 隐含了使用上文中的 `VLAN=`，并为传入的网络帧启用 VLAN ID。

## 案例

- 案例 1. 静态网络配置

    ```systemd.network
    # /etc/systemd/network/50-static.network
    [Match]
    Name=enp2s0

    [Network]
    Address=192.168.0.15/24
    Gateway=192.168.0.1
    ```

    该配置使用静态地址启动了 enp2s0。指定的网关将作为一个默认路由。

- 案例 2. 以太网链路上的 DHCP

    ```systemd.network
    # /etc/systemd/network/80-dhcp.network
    [Match]
    Name=en*

    [Network]
    DHCP=yes
    ```

    这将在所名称有以 en 开头的界面（i.e. 以太网界面）上启用 **DHCPv4** 以及 **DHCPv6**。

- 案例 3. IPv6 前缀委派（Prefix Delegation）

    ```systemd.network
    # /etc/systemd/network/55-ipv6-pd-upstream.network
    [Match]
    Name=enp1s0

    [Network]
    DHCP=ipv6
    ```

    ```systemd.network
    # /etc/systemd/network/56-ipv6-pd-downstream.network
    [Match]
    Name=enp2s0

    [Network]
    IPv6PrefixDelegation=dhcpv6
    ```

    这将在 “enp1s0” 上启用 **IPv6 PD**，并该该界面作为上行（upstream）界面，并启用 **DHCPv6 客户端**  
    同时将 “enp2s0” 作为下行（downstream）界面，且将前缀委任给该界面。

- 案例 4. 具有两个从链路的网桥

    ```systemd.network
    # /etc/systemd/network/25-bridge-static.network
    [Match]
    Name=bridge0

    [Network]
    Address=192.168.0.15/24
    Gateway=192.168.0.1
    DNS=192.168.0.1
    ```

    ```systemd.network
    # /etc/systemd/network/25-bridge-slave-interface-1.network
    [Match]
    Name=enp2s0

    [Network]
    Bridge=bridge0
    ```

    ```systemd.network
    # /etc/systemd/network/25-bridge-slave-interface-2.network
    [Match]
    Name=wlp3s0

    [Network]
    Bridge=bridge0
    ```

    这创建了一个网桥，并将设备 enp2s0 和 wlp3s0 附加上去。该网桥将具有指定的静态地址和网络号，以及通过网关指定的默认路由。所指定的 DNS 服务器将添加至全局 DNS 解析器列表中。

- 案例 5.

    ```systemd.network
    # /etc/systemd/network/20-bridge-slave-interface-vlan.network
    [Match]
    Name=enp2s0

    [Network]
    Bridge=bridge0

    [BridgeVLAN]
    VLAN=1-32
    PVID=42
    EgressUntagged=42

    [BridgeVLAN]
    VLAN=100-200

    [BridgeVLAN]
    EgressUntagged=300-400
    ```

    该配置覆盖了前一个案例中的 enp2s0 界面的配置，并为该网桥端口启用了 VLAN。可使用的 VLAN ID 为 1-32，42，100-400。具有 VLAN ID 42，300-400 的网络包在离开该界面时将被移除标签。为标记的网络包到达该界面时将被赋予 VLAN ID 42。、

- 案例 6. 多个隧道

    ```systemd.network
    # /etc/systemd/network/25-tunnels.network
    [Match]
    Name=ens1

    [Network]
    Tunnel=ipip-tun
    Tunnel=sit-tun
    Tunnel=gre-tun
    Tunnel=vti-tun
    ```

    ```systemd.netdev
    # /etc/systemd/network/25-tunnel-ipip.netdev
    [NetDev]
    Name=ipip-tun
    Kind=ipip
    ```

    ```systemd.netdev
    # /etc/systemd/network/25-tunnel-sit.netdev
    [NetDev]
    Name=sit-tun
    Kind=sit
    ```

    ```systemd.netdev
    # /etc/systemd/network/25-tunnel-gre.netdev
    [NetDev]
    Name=gre-tun
    Kind=gre
    ```

    ```systemd.netdev
    # /etc/systemd/network/25-tunnel-vti.netdev
    [NetDev]
    Name=vti-tun
    Kind=vti
    ```

    这将启用界面 ens1 并用该界面创建 IPIP 隧道，SIT 隧道，GRE 隧道，VIT 隧道。

- 案例 7. 绑定（bond）设备

    ```systemd.network
    # /etc/systemd/network/30-bond1.network
    [Match]
    Name=bond1

    [Network]
    DHCP=ipv6
    ```

    ```systemd.nedev
    # /etc/systemd/network/30-bond1.netdev
    [NetDev]
    Name=bond1
    Kind=bond
    ```

    ```systemd.network
    # /etc/systemd/network/30-bond1-dev1.network
    [Match]
    MACAddress=52:54:00:e9:64:41

    [Network]
    Bond=bond1
    ```

    ```systemd.network
    # /etc/systemd/network/30-bond1-dev2.network
    [Match]
    MACAddress=52:54:00:e9:64:42

    [Network]
    Bond=bond1
    ```

    这将创建一个名称为 bond1 的绑定设备，并将具有 MAC 地址 52:54:00:e9:64:41 和 52:54:00:e9:64:42 的两个设备作为从设备。将使用 IPv6 DHCP 获取一个地址。

- 案例 8. 虚拟路由及转发（Virtual Routing and Forwarding （VRF））

    将 bond1 界面 加入 VRF 主界面 vrf1。把该界面上生成的路由重定向至创建 VRF 时所建立的路由表中。对于 4.8 之前的内核，除非添加了特定的 ip 规则，否则流量不会重定向至 VFR。

    ```systemd.network
    # /etc/systemd/network/25-vrf.network
    [Match]
    Name=bond1

    [Network]
    VRF=vrf1
    ```

- 案例 9. MacVTap

    启用了网络界面 macvtap-test 并将起附加至 enp0s25 上

    ```systemd.network
    # /usr/lib/systemd/network/25-macvtap.network
    [Match]
    Name=enp0s25

    [Network]
    MACVTAP=macvtap-test
    ```

- 案例 10. 一个具有底层物理设备的 Xfrm 界面

    ```systemd.netdev
    # /etc/systemd/network/27-xfrm.netdev
    [NetDev]
    Name=xfrm0

    [Xfrm]
    InterfaceId=7
    ```

    ```systemd.network
    # /etc/systemd/network/27-eth0.network
    [Match]
    Name=eth0

    [Network]
    Xfrm=xfrm0
    ```

    这将创建一个 xfrm0界面，并将其绑定至设备 eth0。这允许基于硬件的 ipsec 负载转移至 eth0 网卡上。若不需要负载转移，xfrm 界面可以被分配至 lo 设备。

## 参见

*systemd(1)*, *systemd-networkd.service(8)*, *systemd.link(5)*, *systemd.netdev(5)*, *systemd-resolved.service(8)*
