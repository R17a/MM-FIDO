
# Meshtastic-FIDO: Technology

Meshtastic-FIDO is an offline echo-conference and personal mail (netmail)
network in the spirit of classic FidoNet / the GoldED editor, built on top of
the **Meshtastic (LoRa)** physical radio protocol. The network requires
neither internet nor a provider: every node stores messages locally, and they
propagate over the airwaves between nodes.

> The low-level binary frame packing and fragmentation protocol
> (`PackerEngine`) is our own development and a proprietary part of the
> technology. This document describes it only at the level of purpose and
> role within the system — without frame format details, opcodes, or the
> chunking algorithm.

---

## 1. Client Types

Several logically distinct node types take part in the network. Physically
it's the same TUI application — the difference between types is defined by
the **device role** (see section 2) and whether the node runs a background
service that shares history with neighbors.

```
●CLIENT ◄──LoRa──► ●CLIENT ◄──LoRa──► ■CLIENT_BASE ◄──LoRa──► ●CLIENT (TRACKER/SENSOR/…)
                                            │
                                          LoRa
                                            │
                                            ▼
                                  ▲ROUTER / ROUTER_LATE ◄──LoRa──► ( Internet uplink )
                                                                     bridge to other
                                                                     network segments

  Any two nodes within radio range connect over LoRa directly — the network
  is flat, with no hierarchy and no single mandatory intermediary. CLIENT_BASE
  and ROUTER are called out separately not because traffic must pass through
  them, but because they additionally carry a service load: CLIENT_BASE holds
  a history cache for catch-up, ROUTER performs priority relaying that extends
  network reach (including to nodes outside direct range — via a chain of
  intermediate relays).
```

### Regular client (network operator)
The standard participant: reads and writes echomail and netmail through the
TUI, subscribes to echo areas, and replies to messages. Runs on any "client"
device role (`CLIENT`, `CLIENT_MUTE`, `CLIENT_HIDDEN`, `TRACKER`, `SENSOR`,
`TAK`, `TAK_TRACKER`, `LOST_AND_FOUND` — see section 2). Keeps its own mail
and subscription list in its own local database. Connects over LoRa directly
to any other node within radio range — to another regular client exactly the
same way as to a `CLIENT_BASE` or `ROUTER`.

### Local cache node / "point" (`CLIENT_BASE`)
The same TUI application, but on a device with the **`CLIENT_BASE`** role
that stays physically powered on for extended periods. Such nodes run a
background service that shares echo-area history and serve catch-up of missed
messages to neighboring clients that return to the air — without needing
internet access. This is the closest analogue of a "point" in FidoNet
terminology.

### Infrastructure relay node (`ROUTER` / `ROUTER_LATE`)
Extends the network's radio range by relaying packets further. May
additionally have internet access and act as a bridge between disjoint
network segments or into a common backbone.

---

## 2. Client Roles (Meshtastic `Config.DeviceConfig.Role`)

The role is read from the firmware of the connected Meshtastic device and
determines the node's behavior in the network:

| Role             | Purpose                                                              | Category |
|------------------|------------------------------------------------------------------------|----------|
| `CLIENT`         | Regular client device (default)                                       | client |
| `CLIENT_MUTE`    | Does not relay other devices' packets                                 | client |
| `CLIENT_HIDDEN`  | Stealth / power-saving mode, minimal airtime activity                 | client |
| `TRACKER`        | Prioritizes GPS position packets                                      | client |
| `SENSOR`         | Prioritizes telemetry packets                                         | client |
| `TAK`            | Integration with the ATAK system                                      | client |
| `TAK_TRACKER`    | ATAK + automatic PLI position beacons                                 | client |
| `LOST_AND_FOUND` | Regularly broadcasts location — for recovering a lost device          | client |
| `ROUTER`         | Infrastructure relay, visible in the node list                        | infrastructure |
| `ROUTER_LATE`    | Low-priority relay — activates last, extends coverage for local clusters | infrastructure |
| `CLIENT_BASE`    | Personal base station — cache node for catch-up sync                  | infrastructure |

The network also recognizes nodes with the `ROUTER_CLIENT` and `REPEATER`
roles — earlier variants of the infrastructure role found on devices running
older versions of Meshtastic firmware — and treats them on par with
`ROUTER` / `ROUTER_LATE` / `CLIENT_BASE`.

### How the role affects network behavior
- Every node — client or infrastructure — independently stores its own mail
  and subscription list in its own local database. Subscribing to an echo
  area is the client's own decision, not a permission that has to be asked
  of anyone.
- Infrastructure nodes (`ROUTER`, `ROUTER_LATE`, `CLIENT_BASE`) additionally
  hold echo-area history and serve catch-up of missed messages to neighbors
  returning to the air. This service runs independently on each
  infrastructure node: a client isn't tied to one specific node — if one is
  unavailable, history can be caught up from any other one in range.
- The `[H]`/`[C]` indicator in the interface reflects whether the current
  node is acting as infrastructure or as a regular client; the visible
  neighbor counters show how many nodes of each type are nearby.
- The client obtains the list of available echo areas from any available
  infrastructure node — not necessarily always the same one.

---

## 3. Data Delivery (general outline)

Messages (echomail, netmail, control commands) pass through a single packing
and fragmentation pipeline (`PackerEngine`) before going out over the air,
which brings them within the LoRa radio channel's MTU limits and reassembles
them on receipt — the same path for every message, with no special cases.

The network is decentralized at the delivery level: live messages propagate
by broadcast relaying over the air, with no dependency on one specific node,
and catch-up of missed history is served by any available infrastructure
node, not a single fixed one.
