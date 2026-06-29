# CCNA-Day-15-Multi-Router-OSPF-Area-0

## Overview

In this lab, I expanded my OSPF knowledge by configuring **Single Area OSPF (Area 0)** across **three routers**. This lab demonstrates how routers automatically exchange routing information and learn remote networks without using static routes.

The objective was to build a small enterprise topology, establish OSPF neighbor relationships, verify dynamically learned routes, and troubleshoot common OSPF issues.

---

# Lab Objectives

After completing this lab, I was able to:

- Configure OSPF on three routers
- Build OSPF neighbor relationships
- Advertise directly connected networks
- Verify OSPF routes
- Verify neighbor adjacencies
- Understand multi-hop routing
- Troubleshoot OSPF neighbor issues

---

# Network Topology

```text
                         LAN 1
                    192.168.1.0/24

       PC1
        |
      Fa0
        |
     SW1 Fa0/1
        |
    SW1 Fa0/24
        |
     R1 G0/0
        |
     R1 G0/1
        |
==============================
      10.10.10.0/30
==============================
        |
     R2 G0/0
        |
     R2 G0/1
        |
==============================
      20.20.20.0/30
==============================
        |
     R3 G0/0
        |
     R3 G0/1
        |
    SW2 Fa0/24
        |
     SW2 Fa0/1
        |
       PC2

                    192.168.3.0/24
                         LAN 2
```

---

# Physical Connections

| Device | Interface | Connected To |
|---------|-----------|--------------|
| PC1 | Fa0 | SW1 Fa0/1 |
| SW1 | Fa0/24 | R1 G0/0 |
| R1 | G0/1 | R2 G0/0 |
| R2 | G0/1 | R3 G0/0 |
| R3 | G0/1 | SW2 Fa0/24 |
| SW2 | Fa0/1 | PC2 Fa0 |

---

# IP Addressing

## LAN 1

| Device | Interface | IP Address | Subnet Mask |
|---------|-----------|------------|-------------|
| PC1 | Fa0 | 192.168.1.10 | 255.255.255.0 |
| R1 | G0/0 | 192.168.1.1 | 255.255.255.0 |

**Default Gateway**

```
192.168.1.1
```

---

## R1 ↔ R2 Link

| Device | Interface | IP Address | Subnet Mask |
|---------|-----------|------------|-------------|
| R1 | G0/1 | 10.10.10.1 | 255.255.255.252 |
| R2 | G0/0 | 10.10.10.2 | 255.255.255.252 |

---

## R2 ↔ R3 Link

| Device | Interface | IP Address | Subnet Mask |
|---------|-----------|------------|-------------|
| R2 | G0/1 | 20.20.20.1 | 255.255.255.252 |
| R3 | G0/0 | 20.20.20.2 | 255.255.255.252 |

---

## LAN 2

| Device | Interface | IP Address | Subnet Mask |
|---------|-----------|------------|-------------|
| R3 | G0/1 | 192.168.3.1 | 255.255.255.0 |
| PC2 | Fa0 | 192.168.3.10 | 255.255.255.0 |

**Default Gateway**

```
192.168.3.1
```

---

# Step 1 – Configure IP Addresses

## R1

```bash
enable
configure terminal

interface g0/0
 ip address 192.168.1.1 255.255.255.0
 no shutdown

interface g0/1
 ip address 10.10.10.1 255.255.255.252
 no shutdown
```

---

## R2

```bash
enable
configure terminal

interface g0/0
 ip address 10.10.10.2 255.255.255.252
 no shutdown

interface g0/1
 ip address 20.20.20.1 255.255.255.252
 no shutdown
```

---

## R3

```bash
enable
configure terminal

interface g0/0
 ip address 20.20.20.2 255.255.255.252
 no shutdown

interface g0/1
 ip address 192.168.3.1 255.255.255.0
 no shutdown
```

---

# Step 2 – Verify Direct Connectivity

Before configuring OSPF, verify that all directly connected routers can communicate.

### R1

```bash
ping 10.10.10.2
```

Expected Result

```
Success
```

---

### R2

```bash
ping 10.10.10.1
ping 20.20.20.2
```

Expected Result

```
Success
```

---

### R3

```bash
ping 20.20.20.1
```

Expected Result

```
Success
```

> **Note:** At this stage, **R1 cannot ping R3**, and **PC1 cannot ping PC2** because routing has not yet been configured.

---

# Step 3 – Configure OSPF

## R1

```bash
router ospf 1

network 192.168.1.0 0.0.0.255 area 0
network 10.10.10.0 0.0.0.3 area 0
```

---

## R2

```bash
router ospf 1

network 10.10.10.0 0.0.0.3 area 0
network 20.20.20.0 0.0.0.3 area 0
```

---

## R3

```bash
router ospf 1

network 20.20.20.0 0.0.0.3 area 0
network 192.168.3.0 0.0.0.255 area 0
```

---

# Step 4 – Verify OSPF Neighbor Relationships

```bash
show ip ospf neighbor
```

Expected Output

| Router | Expected Neighbor(s) |
|---------|----------------------|
| R1 | R2 (FULL) |
| R2 | R1 (FULL), R3 (FULL) |
| R3 | R2 (FULL) |

---

# Step 5 – Verify Routing Table

```bash
show ip route
```

On **R1**, routes learned through OSPF should appear as:

```
O 20.20.20.0/30
O 192.168.3.0/24
```

---

# Step 6 – End-to-End Connectivity

From **PC1**

```text
ping 192.168.3.10
```

Expected Result

```
Success
```

Run a traceroute:

```text
tracert 192.168.3.10
```

Expected Path

```text
PC1
 ↓
R1
 ↓
R2
 ↓
R3
 ↓
PC2
```

---

# Verification Commands

```bash
show ip ospf neighbor
show ip route
show ip protocols
show ip ospf interface brief
show running-config
```

---

# Troubleshooting Practice

### Scenario 1

Configure **R3** in **Area 1** instead of Area 0.

**Expected Result**

- R2 and R3 fail to become neighbors.

---

### Scenario 2

Configure an incorrect wildcard mask.

**Expected Result**

- The interface will not participate in OSPF.

---

### Scenario 3

Shutdown interface **G0/1** on R2.

**Expected Result**

- The OSPF neighbor relationship between R2 and R3 goes down.
- Routes to LAN 2 disappear.

---

### Scenario 4

Remove the LAN network statement from R3.

**Expected Result**

- R1 and R2 will not learn the **192.168.3.0/24** network.

---

# Mini Challenge

Without referring to the lab guide:

- Build the topology.
- Configure all IP addresses.
- Verify direct connectivity.
- Configure OSPF on all routers.
- Verify neighbor relationships.
- Verify the routing table.
- Ping from PC1 to PC2.
- Run `tracert` and identify each hop.
- Break one OSPF configuration intentionally and troubleshoot it.

---

# Key Concepts Learned

- Dynamic Routing
- Single Area OSPF
- OSPF Neighbor Formation
- OSPF Route Advertisement
- Wildcard Masks
- Multi-Hop Routing
- OSPF Verification Commands
- OSPF Troubleshooting

---

# Skills Gained

- Configuring OSPF on multiple routers
- Verifying OSPF neighbor adjacencies
- Reading dynamically learned routes
- Understanding packet flow across multiple routers
- Troubleshooting OSPF configuration issues

---

# Conclusion

This lab expanded my OSPF knowledge from a two-router topology to a three-router enterprise network. I learned how OSPF automatically exchanges routing information between routers, allowing remote LANs to communicate without static routes. I also practiced verifying neighbors, analyzing routing tables, and troubleshooting OSPF issues, strengthening my understanding of dynamic routing for real-world enterprise environments.

---

# Author

**Muhammad Kausar**

**CCNA Enterprise Networking Lab Series**
