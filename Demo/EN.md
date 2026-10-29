# Meshtastic-FIDO — Interface Walkthrough

A decentralized offline echo-conference network (Fidonet/GoldED-style) running over Meshtastic (LoRa).
Below is a sequence of screenshots from a real run-through: from first launch to reading a message thread in an echo conference.

---

### 1. First-run setup wizard — user data and device selection

![Setup wizard: first name, last name, device selection, language selection, echo history depth](Sccreenshots/s1.EN.png)

The first screen on program startup: entering first/last name, choosing a device (defaults to `Mock mode`, no board required, for testing) and the interface language. Below is the "Echo history depth": how much old mail to pull onto a node that has no internet access (`All` / `Month` / `Week` / `Day`). The limit is one-time — once an echo has been fetched successfully, all new mail arrives regardless of age; personal mail (`NETMAIL`) is never limited. The value can be changed later in `config.json` (`replication_period`) or per echo in the "Subscribe" dialog.

### 2. Choosing a connection method

![Setup wizard: expanded connection method list — Mock mode, real device over USB, Bluetooth (BLE), Network (TCP/WiFi)](Sccreenshots/s2.1.EN.png)

The expanded list of available connection methods: `Mock mode` (no board, for testing), `Real device` (auto-detect over USB), `Bluetooth (BLE)`, and `Network (TCP/WiFi)`.

### 3. Connecting to the board over Bluetooth (BLE)

![Setup wizard: MAC address field, "Scan" button, found device in a dropdown list, ESP32 warning](Sccreenshots/s2.2.EN.png)

The same wizard with `Bluetooth (BLE)` selected: scanning the air, the found board `OR_0_891c` by MAC address, and a warning that on ESP32-based boards the Meshtastic firmware disables Bluetooth while the network (WiFi/TCP) is enabled — this is expected board behavior, not a program bug.

### 4. Network policy — accepting the rules before joining

![Modal window "Network Policy" with 5 rules and Accept/Decline buttons](Sccreenshots/s3.EN.png)

Before joining the network, the user sees and confirms the network policy: respecting the radio airtime, the immutability of the Root Node hierarchy, moderator sovereignty within their own echo conference, a ban on commerce (advertising is allowed only through the Root Node; how to apply — the `I` key), and the bridge between cities, regions and countries: when the internet is available, replication runs through the built-in DHT and Relay and does not depend on MQTT server settings. The project License is accepted together with the policy (its text is at the end of the window).

### 5. Main screen — area list

![Area list: System/News, Personal mail, Local folders, RU/EN/ZH echo conferences](Sccreenshots/s4.1.EN.png)
![The same screen after switching to dark theme (F8)](Sccreenshots/s4.2.EN.png)

The application's main screen, GoldED-style: the system area `SYS.ANNOUNCE`, personal mail (`NETMAIL`), local folders (`ARCHIVE`/`IN`/`OUT`), and the starter set of echo conferences (`EN.TALK`, `RU.TALK`, `ZH.TALK`). Under the header is a network summary ("FIDO network: 2 · RU 2 · Meshtastic 2 / MeshCore 0 · updated 7 min ago"); the bottom line shows live status: `C:1 H:1` (nearby clients/hubs), `RADIO`, `Internet: ✓`, `HUB: ✓`, `Channel: fast`, with key hints below. The second screenshot is the same screen after toggling the light theme to dark (the `F8` key).

### 6. Quick echo search (A)

![Dialog "Echo search": mask field and area list](Sccreenshots/s4.3.EN.png)

The `A` key (or `Alt+A`) opens a quick area search by name or mask: the list narrows as you type, `OK` jumps to the selected area.

### 7. Taglines (F3)

![Dialog "Taglines": list, tagline text field, Add/Delete/Close buttons](Sccreenshots/s4.4.EN.png)

The `F3` key — taglines appended to the end of new messages and replies. A single tagline in the list is always used; with several, one is picked at random on every send. An empty list — no tagline is added.

### 8. Subscribing to an echo conference (S)

![Dialog "Subscribe to echo conference": country, region, history depth, Subscribe (%JOIN) button](Sccreenshots/s4.5.EN.png)

The `S` key — subscribe to an echo conference from the shared catalog by country and region, with a history depth for that echo. The "Subscribe (%JOIN)" button sends the request to the hub (unsubscribe is the `D` key, `%LEAVE`). In the screenshot the catalog has no regional echoes yet — only the base `RU/EN/ZH.TALK` are available.

### 9. Nearby nodes list (F6)

![Table of nearby nodes: roles, path, internet indicator, battery](Sccreenshots/s5.EN.png)

The "Nearby Nodes" screen (`F6` key): the `available / known` counter in the header, the own node, and the "FIDO Hubs/Clients" and "Participants" groups (`CLIENT`/`CLIENT_MUTE`/`CLIENT_BASE`). The "Path" column merges hop count with the connection method ("1 hop away, via MQTT", "Relay, confirmed by protocol at 13:22", "Unknown"); then an internet-access indicator and battery level.

### 10. Connection log (F9)

![Log: MQTT bridge, frame send/receive, area sync](Sccreenshots/s6.EN.png)

The diagnostic log (`F9` key): MQTT bridge activity (receiving from the broker, forwarding to the board, `PUBACK`), sending and assembling protocol frame fragments (`SYNC_REQ`, `SYNC_OFFER`), delivery confirmed over MQTT without a LoRa ACK, and per-echo exchange totals — all localized in the interface language.

### 11. Message thread tree

![List of messages in the echo conference with thread tree enabled: topics and nested replies](Sccreenshots/s10.EN.png)

The echo conference's message list with "Thread Tree" mode enabled (`F4`/`Ctrl+T`) — replies are visually nested under the original messages, while unrelated topics are shown as separate branches. Unread messages are marked `*`.

### 12. Reading a message — quoted reply

![Message reading screen: AREA/FROM/TO/DATE/SUBJ header "Re: Test message" and text quoting the original message](Sccreenshots/s8.EN.png)

Reading a reply in the `EN.TALK` echo conference (message 2 of 3 in the area) — area, sender, recipient, date, and subject in the header, with the text below auto-quoting the original message. The bottom line lists reply (`R`), personal reply (`P`), quote (`Q`), forward (`F`) and thread navigation (`N`/`Shift+P`).

### 13. Composing a new message in an echo conference

![New message editor: recipient All, subject "Test message", typing the message text](Sccreenshots/s9.EN.png)

The built-in editor for a new message in the `EN.TALK` echo conference — recipient, subject, and message text. Send is `F2`/`Ctrl+S`, cancel is `F10`/`Esc`.

### 14. Built-in help — hotkeys (part 1)

![Help: area list, message list — hotkeys](Sccreenshots/s11.EN.png)

The built-in hotkey help (`?`/`F1`): navigating the area list (subscribe `S`/`D`, search `A`, request a new echo `E`, advertising `I`, taglines `F3`, neighbors `F6`, log `F9`, theme `F8`) and the area's message list.

### 15. Built-in help — hotkeys (part 2)

![Help: reading, editor, echo moderation](Sccreenshots/s12.EN.png)

The rest of the help: message reading mode, composing messages and the built-in text editor's commands (selection, clipboard, formatting to FIDO limits), plus echo moderation — retracting a message (`Ctrl+R`/`Ctrl+X`) and penalizing an author (`Ctrl+B`: warning or ban within that echo).
