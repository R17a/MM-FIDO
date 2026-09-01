# Meshtastic-FIDO：config.json 参考

`config.json` 是不带注释的纯 JSON。程序在每次设置变更时（F8 切换主题、F5 重新扫描、F3
编辑签名等）都会整体重写该文件，因此直接写进去的注释会被抹掉——所以另立此参考文件。

大多数键由**首次运行向导**设置。向导只在第一次启动时出现（当 `config.json` 里还没有
语言时），此后不会自行再打开——它是一个一次性界面，包含姓名、语言、设备、历史深度
（`replication_period`）和地区（`echo_regions`）字段。要在之后修改其中任何一个键，请在
**程序关闭时**手动编辑 `config.json`（程序运行时所做的修改会在下一次保存设置时被覆盖）。

## 个人数据 / 界面

- `language` —— 界面语言代码（`ru`/`en`/`zh`），由向导设置。
- `tree_mode` —— F4/Ctrl+T，默认显示回复树而不是平铺列表。
- `theme` —— `textual-light`/`textual-dark`，用 F8 切换。
- `first_name` / `last_name` —— 作者姓名，写入每封已发送信件的 `From`。
- `taglines` —— 签名（tagline）列表，在 F3 中编辑；只有一条时始终使用，多条时每封新信件/
  回复随机选一条；空列表则不加签名。

## 连接到设备板

- `mock_mode` —— `true` = MOCK 模式（无物理设备板运行，用于初步了解）；`false` / 无此键 =
  连接真实设备板。在向导中设置；之后切换——在此手动编辑，或重新运行向导（删除
  `config.json`）。
- `connection_type` —— `serial`/`ble`/`tcp`，用哪种传输方式连接设备板（USB / Bluetooth /
  局域网）。它决定下面两个键中需要哪一个。
- `ble_address` —— **仅在 `connection_type: "ble"` 时使用**。特定 BLE 设备板的 MAC 地址
  （留空——连接找到的第一块）。
- `tcp_host` —— **仅在 `connection_type: "tcp"` 时使用**，即通过 Wi-Fi / 局域网连接设备板
  时。设备板的 IP 或 hostname（端口 4403，除非以 `主机:端口` 形式另行指定）。USB/BLE
  连接时不需要此键，也不会出现在 `config.json` 中。与互联网同步（DHT/Relay/MQTT）无关——
  后者通过计算机自身的网络运行。

## 诊断 / 无线电

- `log_level` —— `DEBUG`/`INFO`/`WARNING` 等。在 `DEBUG` 下，`Logs/` 中的日志文件名会变得
  有含义——携带节奏模式和互联网同步标记；其他级别下文件名固定（`mm-fido.log`）。
  **这是唯一影响日志文件名的键**——如果日志名字看不懂，先看这里。
  后缀解读：开头的 `internet_` —— 曾开启 `internet_sync_enabled`；`sequential_A` /
  `sequential_B` —— 分别对应 `radio_sequential_window_enabled` 为 `false`/`true`；`batch`
  —— `radio_fragment_pacing == "batch"`（见下文）。
- `radio_watchdog_enabled` —— 对卡死的设备板接口进行诊断和自动恢复；默认关闭，做现场测试
  时手动开启。如果重连过程本身长时间（约 6 分钟）无响应而卡住——程序以代码 `90` 退出，
  期待外部启动器重启它（Linux 上 `run.sh` 会按此代码自动重启）。

## 无线电：分片节奏与批量同步（现场测试试验场）

- `radio_fragment_pacing` —— `"sequential"`（默认）或 `"batch"`：逐个等待每个分片的确认，
  还是一次性发送/确认整轮。
- `radio_sequential_window_enabled` —— 仅在 `radio_fragment_pacing="sequential"` 时：
  `false` —— 严格只有一个分片在途（`sequential_A`），`true` —— 窗口内多个同时在途
  （`sequential_B`）。
- **该设什么：**
  - `sequential_B`（`radio_fragment_pacing="sequential"` + `radio_sequential_window_enabled=true`）
    —— LoRa 推荐的默认模式：几乎和 `batch` 一样快，但更能容忍丢包。
  - `batch` —— 当设备板确知靠近且信号强时：往返次数最少，现场约 3–4 分钟对 6–8 分钟。
    信道差时突发模式丢得更多。
  - `sequential_A` —— 用于非常边缘的链路（距离、遮挡）：一次一个分片，稳健性最高，但慢。
  - 双方可以运行不同模式——这不妨碍交换（见下文「模式兼容性」）。
- `radio_batch_sync_enabled` —— 对一轮中所有尚未轮询的信区发一个同步请求，而不是每个信区
  一个（批量勘测）；默认 `false`。
- **模式兼容性。** `radio_fragment_pacing` 和 `radio_sequential_window_enabled` 是纯发送端
  设置（把帧分片放上空口的顺序和节奏）。节奏不进入帧格式，不与对端协商，分片和接收端行为
  都不依赖它；确认是传输层的（设备板 / MQTT），不是我们的。这些键在每次发送时从本节点
  自己的 `config.json` 读取。→ 不同模式的节点（`batch` / `sequential_A` / `sequential_B`）
  自由交换信件，包括每个方向不同（`A→B` batch，`B→A` sequential）。卡住的分片在任何模式
  下都由通用的轮次重试补上。

- `radio_mtu_bytes` —— 内部打包协议的帧大小，默认 `200`；LoRa 和 MQTT 桥共用（两种传输
  一套分片）。值越小 = 每帧分片越多，但每个分片更小——用于纯 MQTT 的「城市」场景的杠杆，
  那里丢失一批分片中至少一个的概率随分片数增加。

## 已复制历史的深度

- `replication_period` —— 往这个节点拉多少信区历史：
  `"all"`（默认——不限制）、`"year"`、`"month"`、`"day"`。
  **这是一个滑动窗口：**「现在 − 周期」的边界在每次交换时重新计算，随时钟向前移动。选择
  「过去一个月」——节点始终收到最近 30 天；新信件照常到达（发布时它总在窗口内），只有比
  周期更旧的历史被截断。这与「最后已知之后的一切」这种增量补收是**不同的轴**。
  **仅当节点没有互联网接入时生效。** 有互联网的节点不设窗口——带宽不是问题，而且如果它是
  `CLIENT_BASE`，否则它的客户端会缺失该节点给自己截掉的历史。通过互联网发现（DHT/Relay）
  也从不应用该窗口。**私信（NETMAIL）不受限制**——它不会「过期」。已在本地保存的消息不会
  被删除——扩大周期，下一次交换会补上其余部分。
  **如何设置：** 仅在首次运行向导中——下拉框「信区历史深度」（选项「全部」→ `all` 默认、
  「过去一年」→ `year`、「过去一个月」→ `month`、「过去一天」→ `day`）。向导不会重新打开
  ——首次启动后要改周期，只能在程序关闭时手动编辑 `config.json`。

## 信区目录地理过滤

- `echo_regions` —— `"国家"` 或 `"国家.地区"` 的列表，例如 `["RU.MSK", "RU.SPB"]`。
  应答的 HUB 只返回**全局信区 + 匹配过滤的信区**。节省 LoRa 信道：不用拖来数百个其他
  城市/国家的信区名称。
  - 全局信区（`SYS.*`、`EN.TALK`、`RU.TALK` —— 语言，非地理）——始终在应答中。
  - 国家级信区——如果其 `国家` 出现在 `echo_regions` 的任一元素中。
  - 地区级信区——如果其 `国家.地区` 精确列在 `echo_regions` 中。
  - 地理信区命名为 `<国家>.<地区>.<主题>`（`RU.MSK.TALK`）；归属由名称确定。
  - **仅当节点没有互联网时生效**（与 `replication_period` 相同的判据）。有互联网的
    节点/HUB 携带并提供整个目录。
  - `[]`（走完向导，未给地区）→ 仅全局信区。完全没有此键（旧配置）→ 不应用过滤，
    像以前一样返回完整列表。
  - 邮件轮询会自动把所有本地已订阅的地区信区的地区加到配置里的 `echo_regions` 上——
    订阅了什么就同步什么。
  - 订阅对话框（`S`）不碰这个键：它从 HUB 的「索引」中选择国家 → 地区，并订阅勾选的
    信区（两次短请求代替完整列表——节省 LoRa）。
  **如何设置：** 在首次运行向导中——一个文本字段，地区以空格分隔（`RU.MSK RU.SPB`；
  逗号/`;` 也接受）。向导在首次启动时显示一次；之后要改——在程序关闭时手动编辑
  `config.json`。

## 从列表中隐藏的信区

- `hidden_areas` —— 一组 EchoID，节点照常缓存并复制它们，但操作员**不想**在主屏幕的
  列表中看到。对基础设施节点（`CLIENT_BASE`/`ROUTER`）有意义：完整目录复制和地理目录会
  拉入不仅是自己的订阅，还有其他节点的信区——列表被塞满。
  - 元素格式：精确的 EchoID（`"DE.BER.MEET"`）或尾部通配（`"DE.*"` —— 国家 DE 的所有
    信区，`"TEST.*"` —— 整个 TEST 主题）。不区分大小写。
  - **纯视觉过滤。** 隐藏的信区仍被接收、双向复制并在目录中提供给其他节点——节点仍是
    完整镜像。过滤只作用于界面：主屏幕的信区列表、`R`（「读所有新的」）和 `Space`
    （「下一条未读」）的逐信区遍历、转发时的信区选择。
  - **只在基础设施节点上生效**——角色 `CLIENT_BASE` / `ROUTER` / `ROUTER_LATE` /
    `ROUTER_CLIENT`。在普通 `CLIENT` 上此键被忽略，信区在列表中始终显示（那里本来也
    只有节点自己的订阅）。如果角色变为客户端类，隐藏的信区会重新出现。
  - `SYS.ANNOUNCE` 和 `NETMAIL` 不能隐藏（服务性）。单独的 `"*"` 被忽略——打错字不应
    隐藏整个列表。
  - 无此键 / 空列表 → 什么都不隐藏（原有行为）。
  - 要彻底停止存储某个信区——那是退订（`D`），不是 `hidden_areas`（但在带完整目录的
    `CLIENT_BASE` 上，该信区会在下一次 discover 时回来——这正是需要此过滤的原因）。
    在 `config.json` 中手动编辑，没有向导步骤（首次启动时还没有可隐藏的东西）。

## DHT/Relay（互联网）

DHT/Relay 是通过互联网寻找对端的路径，独立于 LoRa 和 MQTT。它由两个机制组成：带直连尝试
的 DHT 会合，以及单独的 Relay（一个中介服务器）及其自有节点列表。下面的键允许分别控制
它们——默认全为 `true`（先 DHT，Relay 作为 DHT 无进展时的最后一次尝试）。

- `internet_sync_enabled` —— 整个 DHT/Relay 的总开关（既包括出站尝试，也包括通过中继
  接受他人连接）。`false` —— 节点只通过 LoRa/MQTT 工作。
- `internet_sync_interval_connected_sec` —— 当已有本地 LoRa/MQTT 邻居时，多久（秒）尝试
  一次 DHT/Relay（默认 900 = 15 分钟）。当节点完全孤立（根本没有邻居）时——它在每个
  常规邮件轮询节拍上尝试，此键不生效。
- `internet_direct_dht_enabled` —— 默认 `true`。`false` —— 完全跳过 DHT 和直连尝试，
  直接转到 Relay。用于单独测试 Relay 而不用 DHT。
- `internet_direct_relay_enabled` —— 默认 `true`。`false` —— 根本不尝试 Relay，即使 DHT
  没找到任何有用的（循环只是无进展地结束）。用于单独测试 DHT。
- `lan_discovery_enabled` —— 默认 `true`。独立于 DHT/Relay 的第三条发现路径——一个
  局域网内的常驻后台 UDP 广播，无互联网、无 DHT 蜂群。`false` —— 用于单独测试 DHT/Relay
  而不用 LAN。只在一个广播域内工作（跨 VLAN 路由不通）。从属于总的
  `internet_sync_enabled`。
- `upnp_port_mapping_enabled` —— 默认 `true`。通过 UPnP 自动转发节点的 TCP 端口（9999）
  ——省去为直连 DHT 手动配置路由器，与 BT 客户端自动开端口是同一协议。仅对基础设施角色
  （只有端口上有东西在监听时转发才有意义）。尽力而为：如果路由器上的 UPnP 关闭/不支持
  ——静默地什么都不做。`false` —— 完全不尝试（例如路由器要求手动批准 UPnP 请求时）。
- `relay_target_node_id` —— 可选的用于中继的特定节点 id，**添加**到自动发现的列表
  （不替换它）——用于覆盖/测试特定的一对节点。

## MQTT 桥

- `mqtt_bridge_enabled` —— 默认 `true`。整个桥的终止开关，与设备板自身的 MQTT 模块设置
  无关——用于 DHT/Relay 现场测试，需要隔离出「没有 LoRa 也没有 MQTT，只有 DHT/Relay」
  而不必到每台设备的 Meshtastic 应用里去。
- `mqtt_channel_filter_enabled` —— 默认 `false`；`true` 只把这块设备板上实际配置的信道的
  流量转发到设备板并记录日志（而不是整个区域 broker）。
- `mqtt_keepalive_seconds` —— 默认 `60`（已知可用值）。此键的存在是为了不改代码就能测试
  中间值（45/30）。在 Windows 上较小的值曾与接收崩溃同时出现（入站消息归零，不恢复）
  ——如果 MQTT 接收停止且不回来，把它设回 `60`。
- `mqtt_outgoing_publish_pace_seconds` —— 默认 `0.2`。发布「Meshtastic-FIDO」信道同一帧
  连续分片之间的停顿。没有它，分片会无间隔地成串进入 MQTT——在不稳定的路径上（手机
  热点）除第一个外的每个分片都稳定丢失。这是一个时序延迟，不是 ACK 超时。
