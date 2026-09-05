# IP Address Building Blocks

<details>
<summary><b>IP Address Building Blocks</b></summary>

### The Big Picture
The goal of **Layer 3 (Network Layer)** is to deliver packets between different networks.

To accomplish this, every host needs:

- An **IP Address**
- A **Subnet Mask (or CIDR Prefix)**
- A **Default Gateway** (for communication outside its subnet)

Using these three pieces of information, a host can answer four questions:

1. Who am I?
2. Which network am I on?
3. Is the destination on my local network?
4. If not, which router should I send the packet to?

---

### 1. IP Address
An **IP Address** is a **logical Layer 3 address** assigned to a network interface.

Unlike a MAC Address:

| MAC Address | IP Address |
|-------------|------------|
| Layer 2 | Layer 3 |
| Physical identity | Logical identity/location |
| Usually permanent | Can change |
| Assigned by manufacturer | Assigned manually or automatically |

Example:

```
192.168.1.20
```

Think of an IP address as:

> The logical location of a device on a network.

---

### 2. Dynamic vs Static IP

**Dynamic IP (DHCP)**

The address is assigned automatically by a **DHCP server**.

Example:

```
Computer
    │
    ▼
"Can I have an IP address?"

DHCP Server
    │
    ▼
"Use 192.168.1.20"
```

**Advantages**

- Automatic configuration
- Easy management
- Used by laptops, phones, tablets, etc.

---

**Static IP**

The administrator manually configures:

- IP Address
- Subnet Mask
- Default Gateway
- DNS Server

Used for:

- Backend servers
- Databases
- Routers
- Printers

because these devices should always have the same address.

---

### 3. IPv4 Address
IPv4 addresses consist of:

- 32 bits
- 4 octets (bytes)

Example:

```
192.168.1.20
```

Each octet contains 8 bits.

```
8 bits × 4 = 32 bits
```

---

### 4. Network Portion and Host Portion
Every IP address has two parts:

- Network Portion
- Host Portion

Example:

```
192.168.1.20/24
```

Network Portion:

```
192.168.1
```

Host Portion:

```
20
```

Meaning:

> Device number 20 inside network 192.168.1.

---

**Analogy**

```
Street Name
    ↓
 Network

House Number
    ↓
  Host
```

---

### 5. Subnet
A **Subnet** is simply a logical network.

Example:

```
192.168.1.0/24
```

Devices inside this subnet may be:

```
192.168.1.1
192.168.1.20
192.168.1.30
192.168.1.100
```

All of these devices belong to the same subnet because they share the same network portion.

---

### 6. CIDR Notation
CIDR notation tells us how many bits belong to the network.

Example:

```
/24
```

means:

- First 24 bits → Network
- Remaining 8 bits → Host

Example:

```
192.168.1.20/24
```

Network:

```
192.168.1
```

Host:

```
20
```

---

### 7. Subnet Mask
The subnet mask defines where the network portion ends.

Example:

```
255.255.255.0
```

Equivalent to:

```
/24
```

Binary representation:

```
11111111.11111111.11111111.00000000
```

Where:

- 1 → Network bit
- 0 → Host bit

---

### 8. How a Host Uses the Subnet Mask
The host performs a **bitwise AND** between:

- IP Address
- Subnet Mask

Example:

```
192.168.1.20
AND
255.255.255.0
```

Result:

```
192.168.1.0
```

The same calculation is performed on the destination IP.

If both network addresses match:

> Same subnet

Otherwise:

> Different subnet

---

### 9. Same Subnet Communication
Example:

```
Laptop
192.168.1.20

↓

Printer
192.168.1.40
```

Both belong to:

```
192.168.1.0/24
```

The laptop communicates directly with the printer.

No router is involved.

---

### 10. Different Subnet Communication
Example:

```
Laptop
192.168.1.20

↓

Database
192.168.2.10
```

Different subnet.

The laptop sends the packet to the:

```
Default Gateway
```

The router forwards it to the destination subnet.

---

### 11. Default Gateway
The **Default Gateway** is the **router** that a host uses whenever the destination is outside its local subnet.

Think of it as:

> The exit door from your network.

Example:

```
Laptop

IP Address: 192.168.1.20

Default Gateway:
192.168.1.1
```

When sending data to:

```
8.8.8.8
```

The path becomes:

```
Laptop
    │
    ▼
Default Gateway
    │
    ▼
ISP Router
    │
    ▼
Internet
    │
    ▼
Destination Server
```

Your computer only knows the first hop (its default gateway).

The routers handle the rest.

---

### 12. Backend and Database in Different Subnets
Example:

```
Backend
192.168.1.20

↓

Router

↓

Database
192.168.2.10
```

Every request passes through the router.

If the router becomes congested:

- Packets wait in a queue.
- This introduces **queueing delay**.
- Database requests become slower.

---

**Important Note**

Being on different subnets **does not automatically make communication slow**.

The latency depends on:

- Router load
- Network design
- Hardware
- Distance

Modern cloud providers (AWS, Azure, GCP) have extremely fast networking, so routing between subnets inside the same data center usually adds only a very small amount of latency.

---

### Why Put Databases in Different Subnets?
Mostly for **security**.

Example:

```
Internet
    │
Load Balancer
    │
Backend Subnet
    │
Firewall
    │
Private Database Subnet
```

This ensures:

- Users can access the backend.
- Users cannot access the database directly.
- Only the backend can communicate with the database.

---

### Decision Process of Every Host
Whenever a host wants to send a packet:

```
Destination IP
      │
      ▼
Apply Subnet Mask
      │
      ▼
Is the destination in my subnet?
```

If **Yes**:

```
Send directly
```

If **No**:

```
Send to Default Gateway
```

This simple decision is the foundation of IP networking.

---

### Complete Mental Picture
```
                     Internet
                         │
                  Multiple Routers
                         │
                 Default Gateway
                   192.168.1.1
                         │
        ┌────────────────┴────────────────┐
        │                                 │
   Laptop                           Printer
192.168.1.20                     192.168.1.30
        │
        │
        ▼
 Database
192.168.2.10
(Different Subnet)
```

- Same subnet → communicate directly.
- Different subnet → communicate through the default gateway.
- The subnet mask determines which path is used.

---

### Key Takeaways

- An **IP Address** is a logical Layer 3 address.
- IP addresses can be assigned **dynamically (DHCP)** or **statically**.
- Every IP address consists of a **Network Portion** and a **Host Portion**.
- A **Subnet** is a logical network whose devices share the same network prefix.
- A **Subnet Mask** (or CIDR prefix) defines where the network portion ends.
- A host uses the subnet mask to determine whether the destination is local.
- Devices in the **same subnet** communicate directly.
- Devices in **different subnets** communicate through the **Default Gateway**.
- The **Default Gateway** is simply the first router a host sends packets to.
- Separating backend servers and databases into different subnets improves security, though every request must pass through a router.

---

### Vocabulary

| Term | Definition |
|------|------------|
| IP Address | A logical Layer 3 address identifying a network interface. |
| Logical Address | A software-assigned address that can change. |
| DHCP | Dynamic Host Configuration Protocol; automatically assigns IP addresses. |
| Static IP | A manually configured IP address. |
| Subnet | A logical subdivision of an IP network. |
| Network Portion | The part of the IP address identifying the network. |
| Host Portion | The part identifying a specific device within the network. |
| CIDR | Slash notation (e.g., /24) indicating the network prefix length. |
| Subnet Mask | A value that separates the network and host portions of an IP address. |
| Bitwise AND | The binary operation used to calculate the network address. |
| Default Gateway | The router used to reach networks outside the local subnet. |
| Router | A Layer 3 device that forwards packets between networks. |
| Queueing Delay | The time a packet waits in a router before being forwarded. |
| Latency | The total time taken for data to travel from source to destination. |
| Hop | One step in a packet's journey between routers. |

---

### What's Next?
The next networking topics naturally build on these concepts:

1. **ARP (Address Resolution Protocol)** — How a host finds the MAC address corresponding to an IP address.
2. **Ethernet Frames** — How IP packets are encapsulated for transmission over a local network.
3. **IPv4 Packet Structure** — How an IP packet is organized and forwarded.
4. **Routing** — How routers determine the best path to a destination.

![IP Address Building Blocks](The%20IP%20building%20blocks.png)

</details>

---

## Additional Q&A — Building Blocks Review

### Broadcast Address

**What is it?**

The **broadcast address** is the "all hosts" address in a subnet. When a device sends a packet to the broadcast address, **every device** in that subnet receives it.

Using `192.168.1.0/24`:

```
192.168.1.255 → broadcast address
```

Any packet sent to `192.168.1.255` reaches **all devices** in the `192.168.1.0/24` network.

**Why subtract 2 from 256?**

| Address | Purpose | Usable for hosts? |
|---------|---------|-------------------|
| `192.168.1.0` | Network address (the network itself) | No |
| `192.168.1.255` | Broadcast address (all hosts) | No |
| `.1` through `.254` | Individual host addresses | Yes |

**Practical examples:**

- **ARP:** When a host wants to find someone's MAC address, it broadcasts "who has this IP?" to `192.168.1.255`. Every device sees it; only the one with that IP replies.
- **DHCP:** When your device first joins a network (no IP yet), it sends a DHCP discovery broadcast to find a DHCP server.

**Summary:** Broadcast = "everyone listen up." The `.0` is the network name, the `.255` is the "all devices" address. Neither can be assigned to a host.

---

### Can the CIDR Prefix Length Change?

Yes. The CIDR prefix length is **configurable**, not fixed.

The same IP can belong to different sized subnets:

Take `192.168.1.20`:

| CIDR | Subnet Mask | Usable Hosts | Network |
|------|-------------|--------------|---------|
| `/24` | `255.255.255.0` | 254 | `192.168.1.0` |
| `/25` | `255.255.255.128` | 126 | `192.168.1.0` |
| `/26` | `255.255.255.192` | 62 | `192.168.1.0` |
| `/27` | `255.255.255.224` | 30 | `192.168.1.0` |

Same IP, different subnet sizes depending on how you subnet.

**Critical rule:** For two devices to communicate **directly** (same subnet), they must have the **same subnet mask**. If Device A is `/24` and Device B is `/26`, they may disagree on whether they're on the same network — and direct communication breaks.

**Who decides the prefix length?**

- Network admin (for static IP)
- DHCP server (for dynamic IP)

It's a configuration on each host, not an intrinsic property of the IP address itself.

---

### Most Common CIDR Prefixes

**`/24`** — by far the most common.

| Prefix | Hosts | Why popular |
|--------|-------|-------------|
| `/24` | 254 | Perfect balance — big enough for most use cases, small enough to manage |
| `/23` | 510 | Slightly bigger |
| `/25` | 126 | Smaller teams, specific purposes |
| `/26` | 62 | Even smaller |
| `/27` | 30 | Point-to-point links |

---

### CIDR Prefixes You'll Encounter as a Full-Stack Engineer

**`/24` and `/16`** are what you'll see most.

| Where | Example | Use Case |
|-------|---------|----------|
| Home/Lab/Local Dev | `192.168.1.0/24` | Your local network |
| Cloud VPCs (AWS, GCP, Azure) | `10.0.1.0/24` | Default subnet |
| Cloud VPCs (whole VPC) | `10.0.0.0/16` | Entire VPC range |
| Docker | `172.17.0.0/16` | Docker default bridge |
| Kubernetes | `10.244.0.0/16` | Pod network |

When you SSH into a server and run `ifconfig` or `ip addr`, you'll see the prefix. When you configure a firewall rule or security group, you'll use it.

---

### Subnet Definition

**Subnet = Sub-network = a smaller network derived from a larger network.**

```
Large Network (e.g., 10.0.0.0/8)
     │
     ├── Subnet A: 10.0.0.0/24  (devices 10.0.0.1 – 10.0.0.254)
     ├── Subnet B: 10.0.1.0/24  (devices 10.0.1.1 – 10.0.1.254)
     └── Subnet C: 10.0.2.0/24  (devices 10.0.2.1 – 10.0.2.254)
```

The `/24` is a subnet of the larger `/8` network. You "subnet" when you divide a large network into smaller, manageable pieces.

**Why subnet?**

| Reason | Explanation |
|--------|-------------|
| Security | Devices in subnet A cannot reach subnet B without a router/firewall |
| Broadcast containment | Broadcast stays within the subnet, doesn't flood the whole network |
| Organization | Frontend, backend, databases each get their own subnet |
| Performance | Fewer devices per subnet = less broadcast traffic |

In practice, "subnet" and "network" are used interchangeably. `192.168.1.0/24` is both a network and a subnet (of `192.168.0.0/16`).

---

## Multi-Tier Subnet Architecture

### The Classic 3-Tier Architecture

```
┌─────────────────────────────────────────────────────────┐
│                        INTERNET                          │
└─────────────────────┬───────────────────────────────────┘
                      │ port 80/443
                      ▼
              ┌───────────────┐
              │  Load Balancer │  (public IP, distributes traffic)
              └───────┬───────┘
                      │ internal
                      ▼
            ┌─────────────────────┐
            │   Frontend Subnet    │  e.g., 10.0.1.0/24
            │  10.0.1.10 (nginx)  │
            │  10.0.1.11 (nginx)  │
            └──────────┬──────────┘
                       │ port 8080
                       ▼
            ┌─────────────────────┐
            │   Backend Subnet    │  e.g., 10.0.2.0/24
            │  10.0.2.10 (api)    │
            │  10.0.2.11 (api)    │
            └──────────┬──────────┘
                       │ port 5432
                       ▼
            ┌─────────────────────┐
            │  Database Subnet     │  e.g., 10.0.3.0/24
            │  10.0.3.10 (postgres)│
            └─────────────────────┘
```

**Traffic flow is ONE direction: Internet → Frontend → Backend → Database**

---

### The Firewall Rules

On the router/firewall between each subnet:

**Frontend → Backend:**
```
ALLOW: 10.0.1.0/24 → 10.0.2.0/24 on TCP 8080
DENY:  everything else
```

**Backend → Database:**
```
ALLOW: 10.0.2.0/24 → 10.0.3.0/24 on TCP 5432
DENY:  everything else
```

---

### Security Principle: Least Privilege

Every tier only has the minimum permissions it needs to function.

| Tier | Can be reached by | Can reach |
|------|------------------|-----------|
| Frontend | Internet (80/443) | Backend (8080) |
| Backend | Frontend (8080) | Database (5432) |
| Database | Backend (5432) | Nothing (it's a sink) |

No tier can skip the one before it.

---

### Attack Scenarios

**Without subnet separation (flat network):**
```
Attacker compromises frontend → direct access to database → game over
```

**With subnet separation:**
```
Attacker compromises frontend → blocked at router → cannot reach database
```

Even with a SQL injection vulnerability in the frontend, the attacker cannot connect to the database directly. Their attack is limited to what the frontend itself can do — which the backend controls.

---

### AWS Equivalent

```
Internet
    │
    ▼
┌─────────────────┐
│  Internet GW     │  (exposes ALB on port 80/443)
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  ALB / API GW   │  (load balancer, terminates TLS)
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Public Subnet  │  (Frontend)
│  10.0.1.0/24    │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Private Subnet  │  (Backend)
│  10.0.2.0/24    │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Data Subnet    │  (RDS, ElastiCache)
│  10.0.3.0/24    │
└─────────────────┘
```

Security Groups in AWS act as virtual firewalls between these subnets.

---

### DMZ Variation

Some architectures add a **DMZ** (De-Militarized Zone) as a buffer zone:

```
Internet → WAF → Load Balancer → Frontend → Backend → Database
```

The DMZ is the first line of defense. Anything public-facing is filtered there before reaching internal services.

---

### Network Layer vs Application Layer Security

| Layer | What it does |
|-------|--------------|
| Network (Layer 3) | Controls traffic based on IP addresses and ports. Subnet separation is this layer. |
| Application (Layer 7) | WAF rules, rate limiting, authentication. The second line of defense. |

You need **both**. The subnet architecture is your first line of defense — the network layer says "this traffic shouldn't even exist here."

---

## Security vs Performance Trade-off

### The Core Tension

```
Same Subnet (192.168.1.0/24)
├── Backend:    192.168.1.20
└── Database:  192.168.1.30

✓ Fast — direct L2 communication, no router involved
✗ Less secure — if backend is compromised, database is directly reachable
```

```
Different Subnets
├── Backend:  10.0.2.0/24  (10.0.2.10)
└── Database: 10.0.3.0/24  (10.0.3.10)
         │
         ▼
       Router

✓ Secure — traffic filtered by firewall
✗ Slower — every packet goes through router
```

---

### When to Prioritize Security (Different Subnets)

**In production systems — almost always.**

| Scenario | Why |
|----------|-----|
| Internet-facing app | Database must be protected |
| Multi-tenant environment | One client getting hacked shouldn't affect others |
| Compliance requirements | PCI-DSS, HIPAA, SOC2 all require network segmentation |
| Cloud (AWS/GCP/Azure) | Default and recommended pattern |

**Modern cloud data centers are fast:**

```
Same AZ (Availability Zone):
  Subnet A → Subnet B = microseconds of added latency
  Negligible for 99% of applications
```

**Router congestion is rarely the bottleneck in modern systems.** The bigger latency usually comes from database query time and network distance (AZ to AZ), not intra-subnet routing.

---

### When Same Subnet Is Acceptable

**Low-latency, high-throughput internal systems:**

| Scenario | Example |
|----------|---------|
| High-frequency trading | Microseconds matter |
| Real-time streaming | Kafka, video processing |
| Cache invalidation | Redis cluster on same subnet |
| Shared memory systems | Inter-process communication |

Even then, you'd use VLANs or security groups to approximate security without routing overhead.

---

### The Real-World Answer for Full-Stack Engineers

```
Internet → Frontend → Backend (different subnet from DB) → Database
```

**You almost always want separate subnets.**

| Concern | Reality |
|---------|---------|
| "Router adds latency" | < 1ms in same AZ. Usually not your problem. |
| "Router can be congested" | Use better hardware or switch. Security > microseconds. |
| "Different subnet = slow" | Wrong for modern cloud. Right for old enterprise gear. |

**Security wins by default. Optimize only when data proves you need to.**

---

### When Router Congestion IS a Real Problem

| Scenario | Solution |
|----------|----------|
| Thousands of routes (enterprise ISP edge) | Use a switch, not a router |
| Complex ACLs/firewall rules on router | Offload to dedicated firewall hardware |
| Cloud in different AZs/regions | Accept the latency, or cache aggressively |
| IoT with thousands of devices per subnet | Proper subnet planning (don't put 10k hosts in /24) |

---

### Design Decision Framework

```
                    ┌─────────────────────────────┐
                    │  Do you need to expose DB   │
                    │  directly to the internet?  │
                    └──────────────┬──────────────┘
                                   │
                         No ───────┴─────── No
                         │                    │
                         ▼                    ▼
              ┌─────────────────┐   ┌─────────────────┐
              │ Separate subnets │   │ Same subnet OK   │
              │ + firewall       │   │ (internal only)  │
              └─────────────────┘   └─────────────────┘
```

**Default: separate subnets. Optimize later only if you have measured evidence that the router is the bottleneck.**

---

## IP Packet — Quick Overview

### Structure

An IP packet contains:

```
[ Destination IP ] [ Source IP ] [ data ]
```

### IPv4 Address Mechanics

- **Total size:** 32 bits (binary combinations of `0` and `1`)
- **Example split** (`/24` subnet):
  - Network bits: `24` → `2^24` network possibilities
  - Host bits: `8` → `2^8 = 256` total host addresses
  - Usable hosts: `256 - 2 = 254` (subtract network + broadcast)
  - Network address: The first address (e.g., `192.168.1.0`)
  - Broadcast address: The last address (e.g., `192.168.1.255`)

---

## IP Header Overhead

### The Problem

The IP header is **20 bytes minimum** (up to 60 bytes with options).

```
IP Header:  20 bytes (minimum)
Data:       1 byte (your actual payload)
─────────────────────────────
Total:     21 bytes sent
```

You wanted to send **1 byte**. You actually sent **21 bytes**.

**Overhead ratio: 20:1** — 95% of the packet is metadata, not your data.

### Efficiency by Payload Size

| Payload | Header | Total | Efficiency |
|---------|--------|-------|------------|
| 1 byte | 20 bytes | 21 bytes | 4.8% |
| 100 bytes | 20 bytes | 120 bytes | 83% |
| 1460 bytes | 20 bytes | 1480 bytes | 98.6% |

The larger your data, the more efficient the transfer.

### Why Does This Matter?

| Scenario | Impact |
|----------|--------|
| DNS queries | Response of 20-30 bytes but costs 40+ bytes of headers |
| TCP handshake | 3 packets = 60+ bytes of headers before any data |
| IoT telemetry | Thousands of tiny sensors = millions of packets with high overhead |
| Real-time notifications | Each "user is typing" event wastes header space |

### Why Is the Header 20 Bytes Minimum?

The IP header has fixed required fields:

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
├─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┤
│Version│  IHL  │    DSCP   │           Total Length            │
├────────┴───────┴───────────┴───────────────────────────────────┤
│        Identification        │Flags│     Fragment Offset       │
├──────────────────────────────┴─────┴──────────────────────────┤
│  Time to Live     │    Protocol     │      Header Checksum      │
├──────────────────┴─────────────────┴───────────────────────────┤
│                       Source Address                          │
├──────────────────────────────────────────────────────────────┤
│                    Destination Address                         │
├──────────────────────────────────────────────────────────────┤
│                    Options (if IHL > 5)                      │
└──────────────────────────────────────────────────────────────┘
```

| Field | Size |
|-------|------|
| Version, IHL, DSCP, Total Length | 4 bytes |
| Identification, Flags, Fragment Offset | 4 bytes |
| TTL, Protocol, Header Checksum | 4 bytes |
| Source IP | 4 bytes |
| Destination IP | 4 bytes |
| **Total** | **20 bytes** |

Every field serves a purpose — none can be removed without breaking something.

### How Applications Deal With Overhead

| Technique | How it helps |
|-----------|--------------|
| Batch operations | Send 1000 bytes once instead of 1 byte 1000 times |
| Protocol choice | UDP has 8-byte header vs TCP's 20+ bytes |
| HTTP/2, HTTP/3 | Multiplexes streams, reuses connections |
| gRPC | Batches requests over HTTP/2 |
| Compression | Fit more data despite fixed header overhead |

### Full-Stack Perspective

| Scenario | Problem | Solution |
|----------|---------|----------|
| Chat app (typing notifications) | Tiny messages, high header overhead | Batch or use UDP |
| IoT telemetry | Millions of 10-byte readings | Aggregate on device before sending |
| Microservices | Many small RPC calls | gRPC/HTTP2 reuses connections |
| CDN static files | Large files | Header is negligible |

---

## Nagle Algorithm

Named after **John Nagle** (not "nigel"). A TCP optimization that **buffers small writes** and sends them together once the previous data is acknowledged.

### The Problem

```
send("H") → packet sent immediately (20 bytes IP + 20 bytes TCP + 1 byte = 41 bytes)
send("i") → packet sent immediately (20 bytes IP + 20 bytes TCP + 1 byte = 41 bytes)
──────────────────────────────────────────────────────────────
Total: 82 bytes to send "Hi"
```

Sending 2 bytes of data with 40+ bytes of headers = terrible efficiency.

### The Solution

```
send("H") → buffered (not sent yet, waits for ACK)
send("i") → buffered, now we have "Hi" (2 bytes)
→ send ONE packet with "Hi" = 41 bytes total
```

### Trade-off

| Enabled (default) | Disabled |
|------------------|----------|
| Less overhead | More overhead |
| Adds latency (waits for ACK) | Lower latency |
| Good for bulk transfer | Good for real-time apps |

### Where You Encounter It

- **HTTP/2, HTTP/3** — multiplex multiple streams over one connection
- **WebSocket** — designed for real-time, often circumvents Nagle
- **gRPC** — uses HTTP/2, combines small messages efficiently

**Key insight:** Nagle is the network's way of compensating for the small-packet waste problem. Protocol designers need to think about header overhead when designing systems, especially high-throughput or low-latency ones.

---

## What's Next?

Lecture 9 continues with the **IP Packet header structure** in detail:

- Version
- Header Length
- Total Length
- TTL (Time To Live)
- Protocol
- Header Checksum
- Flags & Fragment Offset
- Source IP
- Destination IP


