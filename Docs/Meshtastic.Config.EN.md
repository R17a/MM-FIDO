# Meshtastic-FIDO: config.json reference

`config.json` is plain JSON with no comments. The program rewrites the whole file on every
settings change (F8 theme toggle, F5 rescan, F3 tagline edit, etc.), so any comments written
straight into it would be wiped — hence this separate reference file.

Most keys are set by the **first-run setup wizard**. The wizard appears only on the very
first launch (when `config.json` has no language yet) and never opens again on its own — it
is a one-time screen with fields for name, language, device, history depth
(`replication_period`) and regions (`echo_regions`). To change any key set there later, edit
`config.json` by hand **while the program is closed** (edits made while it runs are
overwritten on the next settings save).

## Personal data / interface

- `language` — interface language code (`ru`/`en`/`zh`), set by the wizard.
- `tree_mode` — F4/Ctrl+T, show the reply tree by default instead of a flat list.
- `theme` — `textual-light`/`textual-dark`, toggled with F8.
- `first_name` / `last_name` — author name, goes into the `From` of every sent message.
- `taglines` — list of taglines, edited in F3; a single entry is always used, several — a
  random pick on each new message/reply; an empty list — no tagline added.

## Connecting to the board

- `mock_mode` — `true` = MOCK mode (runs without a physical board, for a first look);
  `false` / no key = connect to a real board. Set in the wizard; to switch later — edit
  here by hand, or re-run the wizard (delete `config.json`).
- `connection_type` — `serial`/`ble`/`tcp`, which transport to reach the board over
  (USB / Bluetooth / local network). It decides which of the two keys below is needed.
- `ble_address` — used **only with `connection_type: "ble"`**. MAC address of a specific
  BLE board (empty — connect to the first one found).
- `tcp_host` — used **only with `connection_type: "tcp"`**, i.e. when connecting to the
  board over Wi-Fi / the local network. IP or hostname of the board (port 4403 unless another
  is given as `host:port`). With a USB/BLE connection this key is not needed and does not
  appear in `config.json`. It has nothing to do with internet sync (DHT/Relay/MQTT) — that
  runs over the computer's own network.

## Diagnostics / radio

- `log_level` — `DEBUG`/`INFO`/`WARNING` etc. At `DEBUG` the log file name in `Logs/` becomes
  descriptive — it carries the pacing mode and an internet-sync marker; at other levels the
  name is fixed (`mm-fido.log`). **This is the only key that affects the log file name** — if
  a log is named oddly, look here first.
  Suffix decoding: `internet_` at the start — `internet_sync_enabled` was on;
  `sequential_A` / `sequential_B` — `radio_sequential_window_enabled` `false`/`true`
  respectively; `batch` — `radio_fragment_pacing == "batch"` (see below).
- `radio_watchdog_enabled` — diagnostics and auto-recovery of a hung board interface; off by
  default, turned on by hand for field tests. If the reconnect process itself hangs for a
  long time (about 6 minutes) with no response — the program exits with code `90`, expecting
  an external launcher to restart it (on Linux `run.sh` restarts on that code automatically).

## Radio: fragment pacing and batch sync (field-test playground)

- `radio_fragment_pacing` — `"sequential"` (default) or `"batch"`: wait for an ACK on each
  fragment one by one, or send/confirm the whole round at once.
- `radio_sequential_window_enabled` — only with `radio_fragment_pacing="sequential"`:
  `false` — strictly one fragment in flight (`sequential_A`), `true` — several at once in a
  window (`sequential_B`).
- **What to set:**
  - `sequential_B` (`radio_fragment_pacing="sequential"` + `radio_sequential_window_enabled=true`)
    — the recommended default for LoRa: almost as fast as `batch`, but tolerates packet loss
    better.
  - `batch` — when the boards are known to be close and the signal is strong: fewest
    round-trips, ~3–4 min in the field vs 6–8. On a poor channel the burst mode loses more.
  - `sequential_A` — for a very marginal link (distance, obstacles): one fragment at a time,
    maximum robustness, but slow.
  - The two sides may run different modes — it does not hinder the exchange (see
    "Mode compatibility" below).
- `radio_batch_sync_enabled` — one sync request for ALL not-yet-polled areas of a round
  instead of one per area (batch reconnaissance); `false` by default.
- **Mode compatibility.** `radio_fragment_pacing` and `radio_sequential_window_enabled` are
  purely sender-side settings (the order and pace of laying frame fragments onto the air).
  Pacing is not part of the frame format, is not negotiated with the peer, and neither
  fragmentation nor the receiver's behavior depend on it; ACKs are transport-level
  (board / MQTT), not ours. The keys are read from the node's OWN `config.json` on every
  send. → Nodes on different modes (`batch` / `sequential_A` / `sequential_B`) exchange mail
  freely, including differently per direction (`A→B` batch, `B→A` sequential). A stuck
  fragment is recovered in any mode by the common round-based retry.

- `radio_mtu_bytes` — the frame size of the internal packing protocol, `200` by default;
  the same for LoRa and for the MQTT bridge (one fragmentation for both transports). A lower
  value = more fragments per frame, but each fragment smaller — a lever for the MQTT-only
  "city" scenario, where the chance of losing at least one fragment of a burst grows with
  their count.

## Depth of replicated history

- `replication_period` — how much echo-area history to pull onto this node:
  `"all"` (default — no limit), `"year"`, `"month"`, `"day"`.
  **This is a SLIDING window:** the "now − period" boundary is recomputed on every exchange
  and moves forward with the clock. Pick "month" — the node always receives the last 30 days;
  new mail arrives as usual (it is always inside the window at the moment it is posted), only
  history older than the period is cut off. This is a DIFFERENT axis than the incremental
  catch-up of "everything after the last known one".
  **Applies ONLY when the node has no internet access.** A node with internet gets no window
  — bandwidth is not the problem, and if it is a `CLIENT_BASE`, its clients would otherwise
  miss history the node trimmed from itself. Internet discovery (DHT/Relay) never applies the
  window either. **Personal mail (NETMAIL) is exempt** — it does not "go stale". Messages
  already stored locally are not deleted — widen the period and the next exchange pulls the
  rest.
  **How it is set:** ONLY in the first-run wizard — a dropdown "Echo-area history depth"
  (options "All" → `all` default, "Past year" → `year`, "Past month" → `month`, "Past day"
  → `day`). The wizard does not reopen — to change the period after the first launch you can
  only edit `config.json` by hand while the program is closed.

## Echo-area catalog geo-filter

- `echo_regions` — a list of `"COUNTRY"` or `"COUNTRY.REGION"`, e.g. `["RU.MSK", "RU.SPB"]`.
  The responding hub returns only **global echoes + those matching the filter**. Saves the
  LoRa channel: no dragging hundreds of other cities'/countries' echo names.
  - Global echoes (`SYS.*`, `EN.TALK`, `RU.TALK` — language, not geo) — always in the reply.
  - A country-level echo — if its `COUNTRY` is in any element of `echo_regions`.
  - A region-level echo — if its `COUNTRY.REGION` is listed exactly in `echo_regions`.
  - Geo echoes are named `<COUNTRY>.<REGION>.<TOPIC>` (`RU.MSK.TALK`); membership is
    determined from the name.
  - **Applies ONLY when the node has no internet** (same criterion as `replication_period`).
    A node/hub with internet carries and serves the WHOLE catalog.
  - `[]` (wizard done, no regions given) → global echoes only. No key at all
    (old config) → the filter is not applied, full list as before.
  - Mail polling automatically adds to `echo_regions` from the config the regions of all
    locally subscribed regional echoes — what you subscribe to is what gets synced.
  - The Subscribe dialog (`S`) does not touch this key: it picks Country → Region from the
    hub's "index" and subscribes the ticked echoes (two short requests instead of the full
    list — a LoRa saving).
  **How it is set:** in the first-run wizard — a text field, regions separated by spaces
  (`RU.MSK RU.SPB`; comma/`;` are also accepted). The wizard is shown once on the first
  launch; to change it later — by hand in `config.json` while the program is closed.

## Echo areas hidden from the list

- `hidden_areas` — a list of EchoIDs that the node caches and replicates as usual, but that
  the operator does NOT want to see in the list on the main screen. Relevant for an
  infrastructure node (`CLIENT_BASE`/`ROUTER`): full catalog replication and the geo-catalog
  pull in not only its own subscriptions but also other nodes' echoes — the list gets
  cluttered.
  - Element format: an exact EchoID (`"DE.BER.MEET"`) or a trailing pattern
    (`"DE.*"` — all echoes of country DE, `"TEST.*"` — the whole TEST topic). Case-insensitive.
  - **A purely visual filter.** Hidden echoes are still received, replicated in both
    directions and served to other nodes in the catalog — the node stays a full mirror. The
    filter acts only in the interface: the echo list on the main screen, the per-echo walk of
    `R` ("read all new") and `Space` ("next unread"), the echo picker for forwarding.
  - **Works only on an infrastructure node** — role `CLIENT_BASE` / `ROUTER` /
    `ROUTER_LATE` / `ROUTER_CLIENT`. On a plain `CLIENT` the key is ignored, echoes are
    always shown in the list (there it only holds the node's own subscriptions anyway). If
    the role changes to a client one, the hidden echoes reappear.
  - `SYS.ANNOUNCE` and `NETMAIL` cannot be hidden (service). A lone `"*"` is ignored — a
    typo must not hide the whole list.
  - No key / empty list → nothing hidden (previous behavior).
  - To stop storing an echo entirely — that is an unsubscribe (`D`), not `hidden_areas` (but
    on a `CLIENT_BASE` with the full catalog the echo comes back on the next discover — which
    is exactly why this filter exists). Edited by hand in `config.json`, no wizard step (on
    the first launch there is nothing to hide yet).

## DHT/Relay (internet)

DHT/Relay is a path for finding a peer over the internet, independent of LoRa and MQTT. It
consists of two mechanisms: DHT rendezvous with a direct-connection attempt, and separately
Relay (an intermediary server) with its own node list. The keys below let you control them
separately — all `true` by default (DHT first, Relay as a last attempt if DHT made no
progress).

- `internet_sync_enabled` — the master switch for DHT/Relay AS A WHOLE (both outgoing
  attempts and accepting others' connections through the relay). `false` — the node works
  over LoRa/MQTT only.
- `internet_sync_interval_connected_sec` — how often (in seconds) to try DHT/Relay when
  local LoRa/MQTT neighbors are ALREADY present (default 900 = 15 minutes). When the node is
  fully isolated (no neighbors at all) — it tries on every regular mail-poll tick and this
  key has no effect.
- `internet_direct_dht_enabled` — `true` by default. `false` — skips DHT and direct-
  connection attempts entirely, goes straight to Relay. For testing Relay ALONE without DHT.
- `internet_direct_relay_enabled` — `true` by default. `false` — Relay is not tried at all,
  even if DHT found nothing useful (the cycle just ends without progress). For testing DHT
  ALONE.
- `lan_discovery_enabled` — `true` by default. A third path, independent of DHT/Relay — a
  constant background UDP broadcast within one local network, no internet and no DHT swarm.
  `false` — for testing DHT/Relay ALONE without LAN. Works only within one broadcast domain
  (will not cross VLAN routing). Subordinate to the overall `internet_sync_enabled`.
- `upnp_port_mapping_enabled` — `true` by default. Auto-forwarding of the node's TCP port
  (9999) via UPnP — removes the need to configure the router by hand for the direct DHT
  connection, the same protocol torrent clients use to open a port automatically. Only for
  infrastructure roles (forwarding a port makes sense only if something listens on it).
  Best-effort: if UPnP on the router is off/unsupported — silently does nothing. `false` —
  do not try at all (for example, if the router requires manual approval of UPnP requests).
- `relay_target_node_id` — an optional specific node id for the relay, ADDED to the
  auto-discovered list (does not replace it) — to override/test a specific pair of nodes.

## MQTT bridge

- `mqtt_bridge_enabled` — `true` by default. A kill-switch for the whole bridge, regardless
  of the board's own MQTT-module settings — for DHT/Relay field tests where you need to
  isolate "no LoRa and no MQTT, only DHT/Relay" without going into the Meshtastic app on
  every device.
- `mqtt_channel_filter_enabled` — `false` by default; `true` forwards to the board and logs
  only the traffic of the channels actually configured on this board (not the whole regional
  broker).
- `mqtt_keepalive_seconds` — `60` by default (a known-good value). The key exists so you can
  test intermediate values (45/30) without editing code. Lower values on Windows once
  coincided with a receive collapse (incoming messages to zero, does not recover) — if MQTT
  receiving stopped and does not come back, set it back to `60`.
- `mqtt_outgoing_publish_pace_seconds` — `0.2` by default. A pause between publishing
  consecutive fragments of ONE frame of the "Meshtastic-FIDO" channel. Without it the
  fragments went into MQTT in a gapless burst — on an unstable path (a phone hotspot) every
  fragment but the first was consistently lost. This is a timing delay, not an ACK timeout.
