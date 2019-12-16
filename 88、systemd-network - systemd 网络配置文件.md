# systemd.network

## 名称

systemd.network — 网络配置

## 概要

network.network

## 描述

网络设置通过 `systemd-networkd(8)` 实现

主网络文件必须具有 `.network` 的文件后缀名；其它后缀名将被忽略。一旦链接出现，则对应网络即被应用。

`.network` 文件从系统网络文件夹中读取，包含 `/usr/lib/systemd/network`， `/usr/local/lib/systemd/network`，易失性运行时网络文件夹 `/run/systemd/network`，以及本地管理员网络文件夹 `/etc/systemd/network`。所有配置文件均以字符方式排序并处理，无关它们存在与哪个目录中。但是，具有完全相同的文件名的文件将相互替换。在 `/etc/` 中的文件具有最高优先级，`/run` 中的其次，`/usr` 下的优先级最低。这样系统提供的文件就可以按需被本地文件替换。在特殊形况下，一个空文件（文件大小为 0）或指向 `/dev/null` 的软链接将完全禁用该配置文件（也就是被 “`masked`”）。

在网络文件 `foo.network` 目录下，可以存在名为 `foo.network.d/` 的 “插入” 目录。该目录下所有以 `.conf` 结尾的文件都会在主文件解析之后被解析。这有助于修改或添加配置，而不需要修改主配置文件。每个插入文件必须具有合适的段头部（`section header`）。

除了 `/etc/systemd/network`，插入文件夹 `.d` 也可以被放置在 `/usr/lib/systemd/network` 以及 `/run/systemd/network` 目录下。`/etc` 下的插入文件优先于 `/run`，`/run` 下的文件优先于 `/usr/lib`。插入文件优先于高于对应的主网络文件。

注意，一个界面若未配置 IPv6 地址，也不启用 DHCPv6 和 IPv6LL，则应该被认为不支持 IPv6。将会自动向对应界面的 `/proc/sys/net/ipv6/conf/<ifname>/disable_ipv6` 文件写入 `1` 来禁用 IPv6。

## `[Match]` 段选项

包含 `[Match]` 段的网络文件，将决定给定的网络文件是否会套用在给定的设备；`[Network]` 段则决定了设备该如何被配置。首个（以字母序）匹配上的设备名的网络文件将会被启用，所有其它文件将被忽略，即便它们也匹配上了。

一个网络文件与一个设备匹配的条件是，所有在 `[Match]` 段出现的内容都被满足。当一个网络文件不含有任何有效的 `[Match]` 段时，该文件将匹配所有的界面，此时 `systemd-networkd` 将发出警告。提示：若要避免警告，并清楚指明全部的界面都应该被匹配，则添加下面的行：

```systemd
Name=*
```

该段可接受下方的键名：

- `MACAddress=`

    用白空格隔开的硬件地址列表。使用逗号、连字符、点号分割的十六进制表示的全地址。  
    见下方的例子。  
    这个选项可以多次出现，如此这样列表即被合并。  
    若一个空字符串赋予了该选项，则在该定义之前的硬件地址将被全部重置。

    *例子：*

    ```systemd
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

    ```systemd
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

    通过主机名（`hostname`）或主机的机器 ID（`machine ID`）来匹配。参见 `systemd.unit(5)` 中的 `ConditionHost=`。  
    若该列表前缀为 `!`，则测试反转。  
    若一个空字符串赋予了该选项，则在该定义之前的值将被全部重置。

- `Virtualization=`

    测试该系统是否运行在虚拟环境下，且额外测试其是否则特定的实现。参见 `systemd.unit(5)` 中的 `ConditionVirtualization=`。  
    若该列表前缀为 `!`，则测试反转。  
    若一个空字符串赋予了该选项，则在该定义之前的值将被全部重置。

- `KernelCommandLine=`

    测试特定的内核命令行是否设置。参见 `systemd.unit(5)` 中的 `ConditionKernelCommandLine=`。  
    若该列表前缀为 `!`，则测试反转。  
    若一个空字符串赋予了该选项，则在该定义之前的值将被全部重置。

- `KernelVersion=`

    测试内核版本（由 `uname -r` 返回）是否符合特定的表达式。参见 `systemd.unit(5)` 中的 `ConditionKernelVersion=`。  
    若该列表前缀为 `!`，则测试反转。  
    若一个空字符串赋予了该选项，则在该定义之前的值将被全部重置。

- `Architecture=`

    测试系统是否在特定的架构上运行。参见 `systemd.unit(5)` 中的 `ConditionArchitecture=`。  
    若该列表前缀为 `!`，则测试反转。  
    若一个空字符串赋予了该选项，则在该定义之前的值将被全部重置。

## `[Link]` 段选项

`[Link]` 段接受下列键名：

- `MACAddress=`

    设置设备的硬件地址

- `MTUBytes=`

    以字节为单位，设置设备的最大传输单元。  
    可以使用 `K` `M` `G` 后缀，且以 `1024` 为底数。

    注意，若该界面启用了 `IPv6`，且 `MTU` 小于 `1280`（`IPv6` 的 `MTU` 最小值），则其会自动增加到 `1280`。

- `ARP=`

    接受布尔值。若设置为真，则启用该界面的 `ARP`（底层地址解析协议）。若未设置，则使用系统的默认设置。

    举例来说，在单个底层物理界面上创建多个 `MACVLAN` 或 `VLAN` 虚拟界面时，禁用 `ARP` 是十分有用的。其后它们将仅作为一个 link/"bridge" 设备将流量汇总至相同的物理链接上，并不会参与网络的其它部分。

- `Multicast=`

    接受布尔值。若设置为真，则设置设备的多播 flag。

- `AllMulticast=`

    接受布尔值。若设置为真，则驱动会呈递网络中所有的多播包。当启用多播路由时出现。

- `Unmanaged=`

    接受布尔值。当为 `yes` 时，将不会尝试开启或配置匹配的链路，等价于没有对应的网络文件。默认为 `no`。

    这个选项常用于阻止其后匹配的网络文件影响特定的、通过其它程序控制的界面。

- `RequiredForOnline=`

    接受布尔值，或运行状态。请参考 `networkctl(1)` 来了解可能的运行状态。  
    当设置为 `yes` 时，在运行 `systemd-networkd-wait-online` 时，该界面被用于判定系统是否在线。  
    当设置为 `no` 时，在检测系统是否在线时忽略该网络。  
    当设置为某运行状态时，`yes` 为隐含标志，且网络界面必须处于指定的运行状态时，才会被认为是在线状态。  
    默认为 `yes`。

    在通常情况下，网络将被正常地启用，但当出现未被 DHCP 赋予地址，或网线未插入，则链接会保持离线状态，并在存在 `RequiredForOnline=no` 的情况下自动被 `systemd-networkd-wait-online` 跳过。
