# Study Notes: RDMA, RoCEv2, PFC, and ECN

## 1. Why does RoCE use UDP if RDMA needs reliable delivery?

**RoCEv2 implements reliability above UDP.** UDP is a lightweight encapsulation layer; it does not have to provide reliability itself.

```text
RDMA operations
InfiniBand transport  ← reliability for Reliable Connected mode
UDP                   ← stateless encapsulation
IP                    ← routing
Ethernet              ← link-layer delivery
```

In **Reliable Connected (RC)** mode, the RDMA NIC handles packet sequence numbers, acknowledgments, and retransmissions in hardware.

TCP would add another reliable transport layer and a byte-stream abstraction requiring adaptation to RDMA operations. RoCEv2 instead carries the existing InfiniBand transport over UDP/IP.

**RDMA over TCP also exists: iWARP.**

*Version distinction: RoCEv1 runs directly over Ethernet; RoCEv2 uses UDP/IP.*

[Source: NVIDIA transport overview](https://developer.nvidia.com/blog/?p=68265)

## 2. Reliable delivery versus a lossless network

| Concept | Meaning |
|---|---|
| **Reliable delivery** | Endpoints detect missing packets and recover through retransmission. |
| **Lossless network** | Network mechanisms try to prevent packet drops in the first place. |

Reliable RDMA does **not inherently require a network that never drops packets**. RoCE benefits from very low loss because packet recovery can hurt performance. Suitable hardware and configuration can support operation on lossy networks.

“Guaranteed delivery” is conditional: persistent failures eventually produce an error. Also, reliability depends on the selected transport mode; not every RDMA mode is reliable.

[Source: NVIDIA RoCE documentation](https://docs.nvidia.com/networking-ethernet-software/cumulus-linux/Layer-1-and-Switch-Ports/Quality-of-Service/RDMA-over-Converged-Ethernet-RoCE/)

## 3. PFC versus ECN

**PFC pauses a neighboring transmitter. ECN provides feedback that makes the original sender reduce its rate.**

| Property | PFC | ECN |
|---|---|---|
| Full name | Priority-based Flow Control | Explicit Congestion Notification |
| Mechanism | Ethernet control frames | Congestion marking in the IP header |
| Scope | Hop-by-hop | Supports an end-to-end feedback loop |
| Action | Temporarily pauses an Ethernet priority, 0–7 | Signals congestion so the sender can slow down |
| In RoCEv2 | Helps prevent buffer-overflow drops | Receiver responds to congestion marks with CNPs |

In RoCEv2, the receiving NIC sends a **Congestion Notification Packet (CNP)** back to the sending NIC, which reduces its transmission rate. ECN marking alone is only the signal; the endpoint congestion-control mechanism reacts to it.

[Source: NVIDIA RoCE documentation](https://docs.nvidia.com/networking-ethernet-software/cumulus-linux/Layer-1-and-Switch-Ports/Quality-of-Service/RDMA-over-Converged-Ethernet-RoCE/)

## 4. Where do DSCP and PCP fit?

**PFC is priority-based, not inherently DSCP-based.** Switches can use DSCP or VLAN PCP to classify traffic into a PFC-enabled priority.

| Field | Location | Purpose |
|---|---|---|
| **DSCP — 6 bits** | IP header | Classifies traffic for QoS treatment |
| **ECN — 2 bits** | IP header | Indicates ECN capability and congestion |
| **PCP — 3 bits** | Ethernet VLAN tag | Identifies a Layer 2 priority |

DSCP and ECN are separate fields within the same IP header byte:

```text
|       DSCP: 6 bits       | ECN: 2 bits |
```

A DSCP value may also select a queue configured for ECN marking. It is not itself the congestion mark.

[Sources: Cisco DSCP overview](https://www.cisco.com/c/en/us/support/docs/quality-of-service-qos/qos-packet-marking/10103-dscpvalues.html), [Cisco PFC configuration](https://www.cisco.com/c/en/us/td/docs/dcn/nx-os/nexus9000/101x/configuration/qos/cisco-nexus-9000-nx-os-quality-of-service-configuration-guide-101x/m-configuring-priority-flow-control.pdf)

## 5. Example: RoCEv2 traffic under congestion

Suppose a network is configured with **DSCP 26 → priority 3**, with PFC and ECN enabled for the corresponding traffic class. This mapping is a configuration choice, not a universal rule.

1. **Classification:** The switch uses DSCP 26 to assign the packet to the configured priority and queue.
2. **ECN feedback:** As congestion builds, the switch marks eligible packets. The receiving NIC sends CNPs back to the original sender, which slows down.
3. **PFC protection:** If buffer pressure reaches the PFC threshold, the switch asks its immediate upstream neighbor to pause priority 3.
4. **Loss recovery:** If packets are still lost, the reliable RDMA transport detects the loss and retries.

**Memory aid:** DSCP classifies. PFC pauses. ECN signals congestion. RC transport recovers loss.
