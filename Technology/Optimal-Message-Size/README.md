# How Small Is “Small”? Understanding Message Size in Event-Driven Systems

You’ve probably heard the advice:

> **“Events should be small.”**

It’s common. It’s repeated often.
But on its own, it’s not very useful.

**What does “small” actually mean?**
And more importantly — **why does message size matter in the first place?**

Let’s unpack this properly.

---

## There Is No Universal “Small vs Large” Formula

It’s important to be clear from the start:

> **There is no single formula that can definitively classify a payload as “small” or “large.”**

Why? Because optimal message size is not a fixed number — it’s a **trade-off between efficiency and reliability**, very similar to how optimal MTU size is determined in networking.

---

## The Fundamental Trade-Off

Every messaging system lives between two competing forces:

### Larger Messages

**Pros**

* Lower protocol overhead per byte
* Fewer messages to route, store, and acknowledge
* More efficient use of bandwidth (in theory)

**Cons**

* Much higher cost when loss or retransmission occurs
* Greater impact on buffers, queues, and brokers
* Slower recovery during failures

### Smaller Messages

**Pros**

* Lower retransmission cost
* Faster recovery from errors
* Reduced blast radius during failures

**Cons**

* Higher relative overhead
* Higher message rate
* More routing and acknowledgment operations

---

## The Theoretical vs. Real World

In a **perfect network** (infinite bandwidth, zero loss, no congestion), the optimal choice would always be:

> **Send the largest possible message.**

But real systems are not perfect.
Effective bandwidth is continuously reduced by:

* Network reliability fluctuations
* Congestion and background traffic
* Contention in shared infrastructure
* Broker load and replication overhead

These conditions are dynamic and unpredictable. That’s why optimizing for one “ideal” payload size is often misleading.

---

## What Actually Influences Effective Message Size?

At a high level, the total effective size (headers + properties + payload) is shaped by several opposing forces:

### Factors That Push Payload Size Larger

* Available network bandwidth
* Network latency (RTT)

Higher bandwidth and latency favor batching and larger payloads to amortize overhead.

### Factors That Push Payload Size Smaller

* Average network utilization (background traffic)
* Packet loss or interference probability
* Cost of retransmission (time, buffering, replay)
* Fragmentation and reassembly overhead

These factors increase the **risk cost** of large messages.

---

## Why a Single “Optimal” Number Is Misleading

Most of these inputs:

* Change continuously
* Vary by location and time
* Cannot be reliably averaged without oversimplifying

A single number risks masking important failure modes. Instead of chasing one ideal size, the better practice is:

> **Design systems to handle different message characteristics appropriately.**

Which leads to the practical question…

---

# Should Large Messages Be Sent Over the Event Mesh?

### From an event-driven architecture perspective

Events represent **facts or state changes**, not bulk data movement. Large payloads are generally discouraged.

### From an operational perspective

Real systems are messy. Some use cases legitimately involve larger payloads.

The issue isn’t just size — it’s **mixing traffic types**.

---

## The Real Problem: Mixing Small and Large Traffic

When small, high-volume events and large, low-volume messages share the same event mesh, you get **head-of-line blocking**.

Think of a food stall:

> One customer placing a huge order can delay everyone else behind them — even if the total number of customers is small.

Large messages take longer to:

* Transmit
* Store
* Replicate
* Process

This delays unrelated event flows.

The effect is amplified in **site-to-site event meshes**, where messages are replicated across VPN bridges or DMR links.

---

## A More Robust Approach

Instead of forcing all traffic through one pattern:

### 1. Design the event mesh around business and non-functional requirements

Not individual use cases in isolation.

### 2. Choose messaging patterns based on data characteristics

Not just convenience.

### 3. Separate where necessary

For example:

| Use Case                     | Recommended Path                                                                                  |
| ---------------------------- | ------------------------------------------------------------------------------------------------- |
| Small, high-frequency events | Event Mesh                                                                                        |
| Large data transfers         | Dedicated channels (separate VPNs, DMR/static bridges, or external storage with event references) |

This:

* Reduces contention
* Protects event flows
* Preserves predictable mesh behavior

---

## Even Then, Networks Are Shared

Even with logical separation, traffic may still share the same physical network. That introduces:

* Bandwidth planning considerations
* QoS and prioritization strategies
* Capacity and growth modeling

These are solution design topics, not just messaging configuration issues.

---

## The Key Takeaway

> **“Events should be small” is not a rule — it’s a risk management strategy.**

Message size is a balance between:

* Efficiency
* Reliability
* Operational stability
* Failure blast radius

And that balance depends heavily on:

* Business requirements
* Traffic patterns
* Topology
* Failure tolerance

Which is why these decisions are best made as part of **interactive architecture and design discussions**, not just static guidelines.
