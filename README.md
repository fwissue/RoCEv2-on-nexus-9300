# 🚀 RoCEv2 Configuration & Tuning Guide — Cisco Nexus 9300
https://www.cisco.com/c/en/us/td/docs/dcn/whitepapers/roce-storage-implementation-over-nxos-vxlan-fabrics.html

## 📖 Overview

This guide provides configuration and tuning instructions for enabling **RoCEv2 (RDMA over Converged Ethernet v2)** on **Cisco Nexus 9300** switches, including **Priority Flow Control (PFC)** and **Explicit Congestion Notification (ECN)**.

---

## 🔧 Global Settings

```bash
! Enable Priority Flow Control (PFC)
priority-flow-control mode on
priority-flow-control priority 3 enable

! Enable ECN globally
ecn enable
```

---

## 🎯 QoS Classification

```bash
class-map type qos match-any CNP
  match dscp 48

class-map type qos match-any ROCEv2
  match dscp 26
```

> DSCP 48 is used for Congestion Notification Packets (CNP).
> DSCP 26 is used for RDMA payload traffic.

---

## 📦 QoS Policy (Marking)

```bash
policy-map type qos QOS_MARKING
  class ROCEv2
    set qos-group 3
  class CNP
    set qos-group 7
  class class-default
    set qos-group 0
```

---

## 🔁 CoS to DSCP Mapping

```bash
mls qos map cos-dscp 3 26
```

> Maps CoS 3 to DSCP 26. Adjust the value to 48 as needed based on your upstream or application marking policy.

---

## 🚦 Queuing Policy (PFC + ECN)

```bash
policy-map type queuing ROCE-QUEUE
   class type queuing c-out-8q-q3
 # ROCEv2
    bandwidth remaining percent 50
    random-detect minimum-threshold 100 kbytes maximum-threshold 700 kbytes drop-probability 40 weight 0 ecn
class type queuing c-out-8q-q7
#q7 for CNP packet
    priority level 1 

policy-map type network-qos qos_network
  class type network-qos c-8q-nq3
 #enable pause frame
    pause pfc-cos 3
    mtu 4200

system qos
  service-policy type network-qos qos_network
  service-policy type queuing output ROCE-QUEUE


```

> Tune `queue-limit`, `min-threshold`, and `max-threshold` based on your workload.

---

## 🌐 Interface-Level Configuration

```bash
interface Ethernet1/1
  priority-flow-control receive on
  priority-flow-control send on
  service-policy type qos input QOS_MARKING
  service-policy type queuing output ROCE-QUEUE
```

---

## 🎛 Tuning Profiles

| **Profile**     | **Use Case**                              | **Queue Limit** | **ECN Min Threshold** | **ECN Max Threshold** |
|-----------------|--------------------------------------------|------------------|------------------------|------------------------|
| **Conservative** | Low-latency RDMA (HFT, AI inference)     | `6 MB`           | `1 MB`                 | `2 MB`                 |
| **Balanced**     | General-purpose RDMA (NVMeoF, storage)   | `10 MB`          | `3 MB`                 | `6 MB`                 |
| **Aggressive**   | High-throughput RDMA (ML, training jobs) | `16 MB`          | `6 MB`                 | `12 MB`                |

---

## 📊 Monitoring & Validation

### 🔍 Useful NX-OS Commands

```bash
show interface Ethernet1/1 counters
show queuing interface Ethernet1/1
show queuing buffer statistics
show policy-map interface Ethernet1/1
```

### 📈 Key Metrics

- **Pause Frames In/Out**: Indicate PFC behavior.
- **Buffer Drops**: Sign of inadequate queue-limit or congestion.
- **ECN Marking Rate**: Tune thresholds to prevent excessive marking or congestion.

---

## 🧐 Best Practices

- Use **only one CoS** (e.g., `cos 3`) and/or **consistent DSCP values** for RoCEv2 traffic to avoid PFC cross-interference.
- Apply **ECN** to reduce dependency on PFC and mitigate head-of-line blocking.
- Use **telemetry (NX-API or gRPC)** to monitor buffers, drops, and ECN stats.
- Start with the **Balanced Profile**, then tune based on performance metrics.

---

## 📂 Related Topics

- DCBX negotiation support
- Buffer monitoring via `show queuing buffer statistics`
- Nexus 9300 ASIC-specific tuning references
- VXLAN integration with RoCEv2

---

## 📌 Version

Tested with NX-OS release: `9.x(3)` and later  
Platform: Cisco Nexus 9300 series (e.g., C9336C-FX2, C93180YC-EX)

---
