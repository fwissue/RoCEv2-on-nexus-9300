# 🚀 RoCEv2 Configuration & Tuning Guide — Cisco Nexus 9300

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
class-map match-any ROCE-TRAFFIC
  match cos 3
```

> Use `cos 3` or an appropriate DSCP/ACL match to identify RDMA traffic.

---

## 📦 QoS Policy (Marking)

```bash
policy-map type qos ROCE-QOS
  class ROCE-TRAFFIC
    set qos-group 3
```

---

## 🚦 Queuing Policy (PFC + ECN)

```bash
policy-map type queuing ROCE-QUEUE
  class type qos group 3
    bandwidth percent 50
    queue-limit 10000000 bytes
    random-detect ecn
    random-detect min-threshold 3000000 bytes
    random-detect max-threshold 6000000 bytes
```

> Tune `queue-limit`, `min-threshold`, and `max-threshold` based on your workload.

---

## 🌐 Interface-Level Configuration

```bash
interface Ethernet1/1
  priority-flow-control receive on
  priority-flow-control send on
  service-policy type qos input ROCE-QOS
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

- Use **only one CoS** (e.g., `cos 3`) for RoCEv2 traffic to avoid PFC cross-interference.
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

