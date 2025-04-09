# RoCEv2-on-nexus-9300
📘 RoCEv2 Configuration & Tuning Guide — Cisco Nexus 9300
🔎 Overview
This document outlines the configuration steps and tuning best practices to enable RoCEv2 (RDMA over Converged Ethernet v2) on Cisco Nexus 9300 switches using PFC and ECN.

🧩 Global Settings
bash
Copy
Edit
! Enable Priority Flow Control (PFC)
priority-flow-control mode on
priority-flow-control priority 3 enable

! Enable ECN globally
ecn enable
🎯 QoS Classification
bash
Copy
Edit
class-map match-any ROCE-TRAFFIC
  match cos 3
📝 Use cos 3 or DSCP/ACL match to isolate RDMA traffic.

📦 QoS Policy (Marking)
bash
Copy
Edit
policy-map type qos ROCE-QOS
  class ROCE-TRAFFIC
    set qos-group 3
🚦 Queuing Policy (PFC + ECN)
bash
Copy
Edit
policy-map type queuing ROCE-QUEUE
  class type qos group 3
    bandwidth percent 50
    queue-limit 10000000 bytes
    random-detect ecn
    random-detect min-threshold 3000000 bytes
    random-detect max-threshold 6000000 bytes
🧪 Tune thresholds based on link speed and RDMA workload profile.

🌐 Interface Configuration
bash
Copy
Edit
interface Ethernet1/1
  priority-flow-control receive on
  priority-flow-control send on
  service-policy type qos input ROCE-QOS
  service-policy type queuing output ROCE-QUEUE
🎛 RoCEv2 Tuning Profiles
Profile	Use Case	Queue Limit	ECN Min Threshold	ECN Max Threshold
Conservative	Low-latency RDMA (HFT, AI inference)	6 MB	1 MB	2 MB
Balanced	General-purpose RDMA (NVMeoF, storage)	10 MB	3 MB	6 MB
Aggressive	High-throughput RDMA (ML, training jobs)	16 MB	6 MB	12 MB
