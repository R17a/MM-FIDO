# Meshtastic-FIDO — Interface Walkthrough

A decentralized offline echo-conference network (Fidonet/GoldED-style) running over Meshtastic (LoRa).
Below is a sequence of screenshots from a real run-through: from first launch to reading a message thread in an echo conference.

---

### 1. First-run setup wizard — user data and device selection

![Setup wizard: first name, last name, device selection, language selection](Sccreenshots/s1.RU.png)

The first screen on program startup: entering first/last name and choosing a device (defaults to `Mock mode`, no board required, for testing).

### 2. Choosing a connection method

![Setup wizard: expanded connection method list — Mock mode, real device over USB, Bluetooth (BLE), Network (TCP/WiFi)](Sccreenshots/s2.1.RU.png)

The expanded list of available connection methods: `Mock mode` (no board, for testing), `Real device` (auto-detect over USB), `Bluetooth (BLE)`, and `Network (TCP/WiFi)`.

### 3. Connecting to the board over Bluetooth (BLE)

![Setup wizard: MAC address field, "Scan" button, found device in a dropdown list, ESP32 warning](Sccreenshots/s2.2.RU.png)

The same wizard with `Bluetooth (BLE)` selected: scanning the air, the found board `R17a_891c` by MAC address, and a warning that on ESP32-based boards the Meshtastic firmware disables Bluetooth while the network (WiFi/TCP) is enabled — this is expected board behavior, not a program bug.

### 4. Network policy — accepting the rules before joining

![Modal window "Network Policy" with 5 rules and Accept/Decline buttons](Sccreenshots/s3.RU.png)

Before joining the network, the user sees and confirms the network policy: respecting the radio airtime, the immutability of the Root Node hierarchy, moderator sovereignty within their own echo conference, a ban on commerce, and a mandatory shared MQTT bridge (`mqtt.meshtastic.org`, common PSK) between cities for nodes with infrastructure roles.

### 5. Main screen — area list

![Area list: System/News, Personal mail, Local folders, RU/EN/ZH echo conferences](Sccreenshots/s4.1.RU.png)
![The same screen after switching to dark theme (F8)](Sccreenshots/s4.2.RU.png)

The application's main screen, GoldED-style: the system area `SYS.ANNOUNCE`, personal mail (`NETMAIL`), local folders (`ARCHIVE`/`IN`/`OUT`), and the starter set of echo conferences (`EN.TALK`, `RU.TALK`, `ZH.TALK`). The second screenshot is the same screen after toggling the light theme to dark (the `F8` key).

### 6. Nearby nodes list (F6)

![Table of nearby nodes: roles, hop-count path, internet indicator](Sccreenshots/s5.RU.png)

The "Nearby Nodes" screen (`F6` key): the `available / known` counter in the header, the own node, participant nodes (`CLIENT`/`CLIENT_MUTE`/`CLIENT_BASE`) with a "Path" column that merges hop count with the connection method (e.g. "7 hops away, via MQTT" or "Unknown"). The own node shows an internet-access indicator.

### 7. Connection log (F9)

![Log: MQTT bridge startup, topic subscription, packet send/receive](Sccreenshots/s6.RU.png)

The connection diagnostic log (`F9` key): starting the MQTT bridge to the `mqtt.meshtastic.org` broker, subscribing to `msh/RU/#`, and sending/receiving packets over `msh/RU/2/e/LongFast/!<id>` topics (including the encrypted `PKI` channel) — all localized in the interface language.

### 8. Reading a message — quoted reply

![Message reading screen: AREA/FROM/TO/DATE/SUBJ header "Re: Проверка!" and text quoting the original message](Sccreenshots/s8.RU.png)

Reading a reply in the `RU.TALK` echo conference (message 2 of 3 in the area) — area, sender, recipient, date, and subject in the header, with the text below auto-quoting the original message.

### 9. Composing a new message in an echo conference

![New message editor: recipient All, subject "Проверка связи", typing the message text](Sccreenshots/s9.RU.png)

The built-in editor for a new message in the `RU.TALK` echo conference — recipient, subject, and message text.

### 10. Message thread tree

![List of messages in the echo conference with thread tree enabled: a topic, a nested reply to it, and a separate second topic](Sccreenshots/s10.RU.png)

The echo conference's message list with "Thread Tree" mode enabled — the reply is visually nested under the original message, while a second, unrelated topic is shown as its own separate branch alongside it.

### 11. Built-in help — hotkeys (part 1)

![Help: area list, message reading — hotkeys](Sccreenshots/s11.RU.png)

The built-in hotkey help (`?`): navigating the area list (including `F6` — nearby nodes list and `F9` — connection log) and the message reading mode.

### 12. Built-in help — hotkeys (part 2)

![Help: replying and composing messages, built-in text editor — hotkeys](Sccreenshots/s12.RU.png)

The rest of the help: replying to and composing messages, plus the built-in text editor's commands (formatting to FIDO limits, clipboard, etc.).
