---
title: "Is PFC Its Own Ethernet Frame—and Does It Need LLDP?"
description: "A practical mental model for separating PFC's link-local pause mechanism from LLDP/DCBX configuration and ECN feedback."
date: 2026-08-16 09:00:00 -0400
category: Ethernet
tags:
  - PFC
  - LLDP
  - DCBX
  - ECN
  - RoCE
reading_time: 5
---

While learning about lossless Ethernet, I kept running into three acronyms: PFC, LLDP, and DCBX. They were often discussed together, which left me with a basic question:

> Is Priority-based Flow Control carried inside normal traffic, or is it a frame of its own?

The short answer is that **PFC is its own Ethernet MAC Control frame**. It is not a tag or flag added to a regular data packet. That distinction also explains why PFC does not technically require LLDP: the two protocols have different jobs.

## What does a PFC frame look like?

A PFC frame contains the usual destination and source MAC addresses, followed by fields that identify it as a MAC Control frame and specify the PFC operation:

```text
Destination MAC     01:80:C2:00:00:01
Source MAC          Address of the sender
EtherType           0x8808 (MAC Control)
Opcode              0x0101 (PFC)
Priority vector     Priorities affected by this frame
Pause time 0        Pause value for priority 0
Pause time 1        Pause value for priority 1
...                 ...
Pause time 7        Pause value for priority 7
```

The priority-enable vector tells the receiver which of the eight priority pause values are valid. Each enabled priority has its own timer, so one traffic class can be paused while traffic in other priorities continues.

This makes PFC:

- hop-by-hop and link-local;
- exchanged between directly connected devices;
- handled by the receiving NIC or switch rather than routed through the network; and
- selective, unlike classic Ethernet PAUSE, which stops all traffic on the link.

The IEEE description is precise: PFC inhibits transmission of data frames on one or more priorities for a specified time. It uses destination address `01:80:C2:00:00:01`, the PFC opcode, a priority-enable vector, and a time vector. In other words, it is a command to the adjacent device: **"For these priorities, stop transmitting for this long."**

## So, does PFC need LLDP?

No. Two neighbors can use PFC with both ends configured manually. PFC frames can still pause and resume traffic even if LLDP is not running.

LLDP becomes useful because manual configuration is easy to get wrong. Data Center Bridging Exchange, or DCBX, uses LLDP type-length-value fields (TLVs) to advertise and align settings such as:

- which priorities have PFC enabled;
- how many traffic classes can support PFC;
- whether a device is willing to accept its peer's configuration;
- which priority an application such as RoCE should use; and
- how Enhanced Transmission Selection allocates bandwidth.

So LLDP/DCBX belongs to the **configuration and discovery** side of the system. PFC belongs to the **real-time flow-control** side.

## A simple example

Suppose RoCE traffic is assigned to priority 3:

1. **Configuration:** A switch and NIC use LLDP/DCBX to communicate that RoCE uses priority 3 and PFC is enabled for that priority.
2. **Normal operation:** The NIC sends RoCE traffic classified as priority 3.
3. **Congestion:** The switch's queue begins to fill, so it sends a PFC MAC Control frame asking the NIC to pause priority 3.
4. **Recovery:** When the pressure clears, the switch sends an updated PFC frame with a zero pause value for priority 3, allowing that traffic to resume.

The LLDP frames do not perform the pause. They help the two devices agree on the setup that makes the later PFC exchange meaningful.

## Where does ECN fit?

PFC is sometimes discussed alongside Explicit Congestion Notification, but they signal congestion differently.

| Mechanism | What it does | Scope |
|---|---|---|
| LLDP/DCBX | Advertises and aligns PFC and QoS configuration | Between neighbors, periodically |
| PFC | Tells the adjacent device to pause selected priorities | Hop-by-hop, during congestion |
| ECN | Marks an IP packet to report congestion without immediately dropping it | Carried toward an endpoint |
| CNP in RoCEv2 | Tells the sender to reduce its transmission rate after ECN is observed | Feedback toward the source |

This difference matters. PFC provides immediate relief on one Ethernet link by stopping selected traffic temporarily. ECN participates in a broader congestion-control loop that asks the source to slow down. They can complement each other, but they are not interchangeable.

## The mental model that finally clicked

The easiest way for me to remember the relationship is:

> **LLDP/DCBX agrees on the rules. PFC applies the brakes on one link. ECN tells the source there is congestion ahead.**

That also exposes the main operational risk. PFC does not require LLDP/DCBX, but without some way to verify configuration, the neighbors may disagree—for example, the switch may send PFC for priority 3 while the NIC expects RoCE on priority 4. The frames would still be valid, but the intended traffic would not be protected.

PFC is therefore independent of LLDP in operation, yet LLDP/DCBX is often what makes a real deployment safer and easier to manage.

## References

- [IEEE 802.1 Data Center Bridging Task Group](https://www.ieee802.org/1/pages/dcbridges.html)
- [IEEE PFC frame-format proposal](https://www.ieee802.org/1/files/public/docs2008/bb-pelissier-pfc-proposal-0408v3.pdf)
- [NVIDIA: Priority Flow Control and LLDP/DCBX configuration](https://docs.nvidia.com/networking/display/ofedv490170/flow+control)
- [RFC 3168: The Addition of Explicit Congestion Notification to IP](https://www.rfc-editor.org/rfc/rfc3168.html)

