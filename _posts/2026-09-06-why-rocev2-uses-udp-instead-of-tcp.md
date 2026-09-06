---
title: "Why RoCEv2 uses UDP instead of TCP"
date: 2026-09-06 09:00:00 -0400
category: Ethernet
tags:
  - RDMA
  - RoCE
  - UDP
  - TCP
reading_time: 3
---

Remote Direct Memory Access (RDMA) over Converged Ethernet, or RoCE, is often deployed on lossless Ethernet fabrics to work efficiently. Then why does RoCE run on UDP instead of TCP?

TCP provides reliable transport through mechanisms such as acknowledgments and retransmissions. On the surface, TCP looks like a good fit for carrying RDMA traffic over an inherently lossy medium like Ethernet. RoCEv2, however, already has a transport layer that can provide reliability above UDP.

To understand why TCP isn't necessary, we need to go a bit deeper into how RoCE transport works. This article focuses on RoCEv2 using Reliable Connected (RC) transport. RoCEv1 runs directly over Ethernet; [RoCEv2 adds UDP/IP encapsulation](https://networking-docs.nvidia.com/mlnxofedswum/24010331/rdma-over-converged-ethernet-roce).

A simplified RoCEv2 stack looks like this, from the RDMA operations down to Ethernet:

| Layer | Role |
|---|---|
| RDMA operations | Remote memory operations |
| InfiniBand transport (RC) | Sequence numbers, acknowledgments, retries |
| UDP | Lightweight encapsulation |
| IP | Routing |
| Ethernet | Link-layer delivery |

The distinction is between reliable delivery and a lossless network:

* Reliable delivery: The endpoints detect missing packets and retransmit them, subject to retry limits.
* Lossless network: The network tries to prevent congestion-related packet drops in the first place.

RoCE uses InfiniBand (IB) transport mechanisms, which provide reliability in RC mode. These mechanisms run in hardware on an RDMA network interface card (NIC). Reliability does not mean delivery under every failure: [a connection can fail when its retry limits are exceeded](https://docs.nvidia.com/networking/display/nvidiawinof2documentationv25750000/rdma%2Bcapabilities).

This explains why TCP is unnecessary for reliability in this case. TCP would introduce another transport layer when IB transport already handles acknowledgment and recovery. UDP provides a lightweight, stateless way to encapsulate those transport packets over IP.

An interesting side note: RDMA over TCP actually exists, in the iWARP protocol suite. [RFC 5044](https://www.rfc-editor.org/rfc/rfc5044.html) specifies the framing layer used to carry its data-placement protocol over TCP.

If RoCE has IB transport underneath to ensure reliable delivery, why is there a big fuss about implementing "lossless" Ethernet for AI and HPC (high-performance computing)?

Packet recovery can be expensive and inefficient, resulting in suboptimal RoCE performance. It takes time to detect lost packets and then retransmit them, usually with a [Go-Back-N mechanism or selective repeat on newer RNICs](https://networking-docs.nvidia.com/winof2driverum/251050020/ethernet-network). With Go-Back-N, a single packet loss can result in several later packets being retransmitted, amplifying the cost. A lossless Ethernet fabric commonly uses Priority-based Flow Control (PFC) and Explicit Congestion Notification (ECN). PFC tells the sender to pause selected traffic. With ECN-based congestion control, switches mark packets traveling toward the receiver, and the receiver sends congestion notification packets (CNPs) back to the sender so it can reduce its rate. These mechanisms help prevent congestion-related drops and reduce the need for recovery. [NVIDIA's RoCE configuration documentation](https://docs.nvidia.com/networking-ethernet-software/cumulus-linux/Layer-1-and-Switch-Ports/Quality-of-Service/RDMA-over-Converged-Ethernet-RoCE/) describes both lossless and lossy modes.

RoCE can also operate on lossy Ethernet. How well it performs depends on the NIC's recovery and congestion-control capabilities and the network conditions. NVIDIA's [Zero Touch RoCE implementation](https://developer.nvidia.com/blog/scaling-zero-touch-roce-technology-with-round-trip-time-congestion-control/) is one example.

Reliable RDMA needs reliable transport semantics, but it does not inherently require a network that never drops packets. Preventing congestion-related drops reduces the recovery work that the endpoints must do.
