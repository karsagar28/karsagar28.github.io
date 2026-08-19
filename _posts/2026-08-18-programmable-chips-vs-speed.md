---
title: "Programmable Chips vs Speed"
date: 2026-08-18 09:00:00 -0400
category: Network Architecture
tags:
  - P4
  - ASIC
  - Cisco Silicon One
  - AI Networking
  - Programmability
reading_time: 5
---

An early misconception I had about ASICs was that they are fast because they are one-trick ponies. A computing chip that can perform only one specific operation is fast and cheap to make. Traditional data-center switches were filled with these ASICs. In my head, a packet entered one rigid pipeline on one side, ran along the only lane available, and came out the other side at line rate.

Cisco Silicon One is Cisco's new approach to designing networking ASICs. While reading through some training materials on S1 architecture, these lines caught my eye.

<blockquote class="source-quote">
  <p>Most networks have fixed rules for how they process traffic. That approach works fine until the requirements change. In artificial intelligence and machine learning (AI/ML) data centers, cloud networks, or 5G environments, traffic patterns shift fast, and rigid hardware can slow you down.</p>
  <p>Cisco Silicon One fixes this situation by supporting Programming Protocol-independent Packet Processors (P4). This language lets you customize how to process data packets, all in software, without replacing the processor.</p>
  <p>For example, engineers can program Silicon One (like the G200 or P100) to prioritize traffic for AI workloads like LLM training or to manage different types of services more efficiently.</p>
  <footer>Cisco U training material on Cisco Silicon One</footer>
</blockquote>

Okay, sweet. But that raised two questions:

1. For a use case that is famously sensitive to speed, why are we making the chips less 'rigid'? Are we teaching our fast pony new tricks?
2. How far can this programmability be stretched? Is Cisco selling us blank Play-Doh hardware that I, the network engineer, can mold into whatever I want?

Turns out my preconception of what an ASIC is was wrong. An ASIC isn’t necessarily hardwired to perform exactly one algorithm. Instead, it's designed for a specific application domain. One such domain is extremely fast packet forwarding.

Cisco Silicon One is essentially a specialized packet-processing machine:

- The physical machinery is fixed: Ethernet interfaces, packet buffers, lookup engines, arithmetic units, queues, schedulers, memory widths, and processing capacity.
- The forwarding recipe executed by that machinery is programmable.
- P4 describes that recipe: which headers to recognize, which fields to use as lookup keys, what tables to consult, and what actions to perform.

For example, conceptually, a P4 program might say:

```text
parse Ethernet → IPv6 → SRv6
look up destination and policy
decrement hop limit
attach telemetry metadata
select output port and queue
rewrite headers
```

Cisco’s compiler converts that program into bytecode or microcode that runs on Silicon One’s packet-processing engines. Cisco describes this as combining P4 programmability with a “run-to-completion” processing model. See Cisco's [Silicon One architecture paper](https://www.cisco.com/c/en/us/solutions/collateral/silicon-one/silicon-one-wp.html).

> So the correct intuition is: “The chip is programmable within a fixed hardware envelope.”

## So, it's programmable AND fast?

My mistake was treating *specialized* and *rigid* as the same thing.

The chip is fast because it has purpose-built packet machinery working in parallel. Lookups happen in dedicated hardware. Buffers and forwarding engines are designed around the single purpose of moving packets as fast as possible. It is not asking a general-purpose CPU to interpret some arbitrary Python program for every frame.

P4 does not remove those constraints. A program still has to fit into the processing budget offered by that ASIC. If it asks for something the hardware cannot do at line rate, the compiler cannot wish the problem away.

The flexibility exists before the packets arrive. Once the program is compiled and installed, the same specialized hardware executes it at speed.

## But who gets to program it?

The word programmable can mean different things depending on what part of the lifecycle we are talking about:

<figure class="post-figure post-figure--compact">
  <img src="{{ '/assets/images/posts/p4-programmability/layers-of-programmability.svg' | relative_url }}"
       alt="Four stages of control in a programmable switch ASIC: runtime and day-2, boot and day-1, compile time, and ASIC design time. Compile time is where most P4 programmability lives.">
</figure>

1. Physical ASIC RTL/design time - Chip design by the HW vendor. Things like buffer architecture, SerDes count, VOQ, TCAM/SRAM sizing, etc. This can't be changed once set. There is no customization offered here.
2. Compile time - Things that go in the NOS image, controlled by the software vendor. The parser, header definitions, and match-action pipeline are expressed in a P4-ish language, compiled to pipeline microcode, and shipped as part of the software release. This is the layer where SRv6 uSID or a new encap shows up as a software upgrade instead of a silicon respin. It's "software," but it's Cisco's software, not yours.
3. Boot / day-1 - Network architect controlled. Hardware forwarding profiles: TCAM carving, table-scale profiles that trade LPM against ACL space, port breakout, sometimes standalone-vs-fabric mode. You're picking from a menu of vendor-precompiled configurations, not recompiling the pipeline. The network architect controls this, but in most cases a reload is required. It is not typically a day-2 operations activity.
4. Runtime / day-2 - Network engineer controlled. This consists of populating table entries. Normal networking. The pipeline behavior is already fixed.

So the P4 programmability is mostly at layer 2, and I am not the one doing the customizing. Cisco doesn't expose the P4 toolchain on IOS-XR or NX-OS. Hyperscalers with direct Silicon One SDK access may get more rope.

> Intel's Tofino is the counterexample. It genuinely handed P4 to end users, and it turns out customers didn't really want it anyway. Tofino 2 sat at 12.8T while Tomahawk 4 shipped 25.6T and Tomahawk 5 hit 51.2T. Customers chose bandwidth-per-watt over the ability to write their own ASIC pipeline.

## Where AI fits

AI networks need very fast hardware, but the way they use that hardware is still changing. Congestion control, multipath forwarding, packet trimming, telemetry, and load balancing are all moving faster than a silicon replacement cycle.

This is where P4 helps. Cisco says it has added things like Multipath Reliable Connection support and packet trimming to existing Silicon One hardware through P4 software updates.

That gives AI networks both things I originally thought were incompatible: specialized hardware for speed, plus enough flexibility to support new forwarding behavior before the next ASIC architecture change.

## So, what did I get wrong?

The speed of an ASIC does not come only from freezing one algorithm into a rigid chip. It comes from specialized hardware, parallel processing, and a limited set of operations engineered to run at line rate.

P4 makes the forwarding pipeline flexible, but it does not make different ASICs interchangeable. It can change packet parsing and match-action behavior within the chip’s limits. It cannot add buffer capacity, bandwidth, SerDes, or new physical machinery.

A P4-programmable microwave cannot be converted to a refrigerator.

## References

- [Cisco 8000 Powered by Cisco Silicon One: Foundation for Success](https://www.cisco.com/c/en/us/solutions/collateral/silicon-one/silicon-one-foundation-success-wp.html)
- [P4~16~ Language Specification](https://p4.org/wp-content/uploads/sites/53/p4-spec/docs/P4-16-v1.2.4.html)
- [Cisco: Scaling the Future, Why Ethernet Is the Backbone of AI Supercomputing](https://blogs.cisco.com/datacenter/scaling-the-future-why-ethernet-is-the-backbone-of-ai-supercomputing)
