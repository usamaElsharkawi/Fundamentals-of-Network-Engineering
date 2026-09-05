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

## IPv4 Packet Size Limits

### Maximum Packet Size: 65,535 Bytes

The **Total Length** field in the IPv4 header is **16 bits**.

```
2^16 = 65,536 (values from 0 to 65,535)
Max value = 65,535 bytes
```

This is the **entire packet** (header + data):

```
┌──────────────────────────────────────────────────────┐
│               IPv4 Packet (max 65,535 bytes)          │
├─────────────────────────┬────────────────────────────┤
│   IP Header (20-60 B)  │      Data (up to 65,515 B)  │
└─────────────────────────┴────────────────────────────┘
```

### Maximum Data Payload

```
65,535 bytes (total)
- 20 bytes (minimum header)
─────────────────────────
65,515 bytes of actual data
```

### Why the 65,535 Limit?

The 16-bit field **physically cannot store a number larger than 65,535**.

```
16 bits = can store 0 to 65,535
If you try to make a bigger packet, the field overflows
```

---

## MTU (Maximum Transmission Unit)

### The Real-World Limit

The IPv4 spec allows 65,535 bytes, but **MTU** is the actual limit:

**MTU = the largest packet a network link can transmit without splitting it.**

```
Standard Ethernet MTU: 1,500 bytes
```

So even though IP allows packets up to 65,535 bytes, the link layer says:

> "I can only carry 1,500 bytes at a time."

### MTU Values by Network Type

| Network Type | MTU |
|--------------|-----|
| Standard Ethernet | 1,500 bytes |
| PPPoE (DSL) | 1,492 bytes |
| Loopback (localhost) | 65,535 bytes |
| Jumbo frames (custom) | 9,000 bytes |
| AWS VPC | 9,000 bytes (with jumbo frames enabled) |

### Fragmentation

When an IP packet exceeds the MTU:

```
You want to send: 4,000 byte IP packet
Link MTU: 1,500 bytes

→ Packet gets split into 3+ fragments
→ Each fragment ≤ 1,500 bytes
→ Sent separately
→ Reassembled at destination
```

Fragmentation is generally avoided because:
- Adds processing overhead
- If one fragment is lost, whole packet is lost
- Older, less efficient

### Custom MTU (Large Companies)

Big companies (Amazon, etc.) with custom hardware may use **jumbo frames**:

```
MTU: 9,000 bytes (instead of 1,500)
Benefits: fewer packets, less header overhead, higher throughput
```

**But** — jumbo frames only work within their private network. As soon as traffic hits the public internet, it hits the 1,500 byte MTU barrier.

```
Your Server (MTU 9,000)
        ↓
Internet Router (MTU 1,500)  ← fragmentation happens here
        ↓
Destination Server
```

### Full-Stack Perspective

When setting up Docker, Kubernetes, or cloud services, MTU settings matter:

```
Docker default bridge: 1,500 bytes
AWS VPC: 9,000 bytes (with jumbo frames enabled)
Kubernetes pod network: Some CNI plugins use 9,000 bytes
```

If a Docker container with MTU 9,000 talks to a host with MTU 1,500, packets can be dropped or fragmented.

### Summary

| Concept | Value |
|---------|-------|
| IP max packet | 65,535 bytes |
| Standard Ethernet MTU | 1,500 bytes |
| Jumbo frames | 9,000 bytes |
| Real-world max | Limited by MTU, not IP spec |

**You'll never see a 65,535 byte IP packet in the real world** — the MTU limits it first. The only exception is custom internal networks at large companies with specialized hardware.

---

## IPv4 Header Structure

The IPv4 header is a fixed structure, minimum **20 bytes** (up to 60 bytes with options).

| Offset | Bits | Field |
|--------|------|-------|
| 0 | 0-3 | Version |
| 0 | 4-7 | IHL (Header Length) |
| 0 | 8-13 | DSCP |
| 0 | 14-15 | ECN |
| 0 | 16-31 | Total Length |
| 4 | 32-47 | Identification |
| 4 | 48-50 | Flags |
| 4 | 51-63 | Fragment Offset |
| 8 | 64-71 | Time to Live (TTL) |
| 8 | 72-79 | Protocol |
| 8 | 80-95 | Header Checksum |
| 12 | 96-127 | Source IP Address |
| 16 | 128-159 | Destination IP Address |
| 20 | 160-191 | Options (if IHL > 5) |
| ... | ... | Data |

Each row represents **4 bytes (32 bits)**. Data begins at byte offset 56 (448 bits) after the minimum 20-byte header.

---

## IPv4 Header Fields — Complete Reference

### Complete Header Diagram

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
├─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┤
│Version│  IHL  │    DSCP   │ECN│           Total Length            │
├────────┴───────┴───────────────────┴───────────────────────────────────┤
│        Identification         │Flags│         Fragment Offset           │
├─────────────────────────────┴─────┴─────────────────────────────────────┤
│  Time to Live │   Protocol      │         Header Checksum              │
├────────────────┴─────────────────┴─────────────────────────────────────┤
│                         Source IP Address                               │
├─────────────────────────────────────────────────────────────────────────┤
│                       Destination IP Address                            │
├─────────────────────────────────────────────────────────────────────────┤
│                    Options (if IHL > 5)                                 │
└─────────────────────────────────────────────────────────────────────────┘
```

---

### Version

**Size:** 4 bits | **Position:** Byte 0, Bits 0-3

Indicates which IP version is being used.

| Value | Version |
|-------|---------|
| 4 | IPv4 |
| 6 | IPv6 |

When a device receives an IP packet, it reads the **first 4 bits** first to determine how to parse the rest of the header.

```
0100 = IPv4
0110 = IPv6
```

---

### IHL (Internet Header Length)

**Size:** 4 bits | **Position:** Byte 0, Bits 4-7

Tells the receiver how long the IP header is, so they know where the data starts.

| IHL Value | Header Length | Options |
|-----------|---------------|---------|
| 5 | 20 bytes | None |
| 6 | 24 bytes | 4 bytes |
| ... | ... | ... |
| 15 | 60 bytes | 40 bytes |

Stored in **32-bit words** (4-byte chunks), so:

```
IHL = 5  →  5 × 4 bytes = 20 bytes (minimum header, no options)
IHL = 15 → 15 × 4 bytes = 60 bytes (maximum header, with options)
```

Why in 4-byte units? 4 bits can store 0-15, but 15 bytes isn't enough for the max 60-byte header. By counting in 4-byte units, it can express up to 60 bytes.

---

### DSCP (Differentiated Services Code Point)

**Size:** 6 bits | **Position:** Bytes 0-1, Bits 8-13

**QoS (Quality of Service)** — tells routers how to prioritize this packet.

| DSCP Value | Name | Use Case |
|------------|------|----------|
| 0 | Best Effort | Regular browsing, downloads |
| 46 | Expedited Forwarding (EF) | VoIP, video calls — low latency |
| 34 | AF41 | High priority video |
| 26 | AF31 | Medium priority |

When a router is congested, higher DSCP packets get processed first.

**In practice:** DSCP is often ignored on the public internet. It works best in controlled environments (enterprise networks, data centers, AWS VPC with proper config).

---

### ECN (Explicit Congestion Notification)

**Size:** 2 bits | **Position:** Bytes 0-1, Bits 14-15

Allows routers to signal **congestion** without dropping packets.

```
Traditional: Router full → drop packet → sender slows down (via timeout)
ECN:         Router full → mark packet "congested" → sender slows down proactively
```

Values:

```
00 = Not ECN capable
01 = ECN capable (ECT)
10 = ECN capable (ECT)
11 = Congestion Experienced (CE)
```

Both sender and receiver must support ECN. It's negotiated at the TCP level. The benefit: no packet drops needed — the network tells endpoints about congestion **before** it becomes a problem.

---

### Total Length

**Size:** 16 bits | **Position:** Bytes 0-1, Bits 16-31

Tells the receiver the **total size of the entire IP packet** (header + data) in bytes.

```
2^16 - 1 = 65,535 bytes (maximum packet size)
```

The 16-bit field physically cannot store a number larger than 65,535.

| Scenario | Total Length |
|----------|--------------|
| DNS response | ~50-100 bytes |
| HTTP page request | ~500-1,500 bytes |
| File download | Up to MTU-limited fragments |

---

### Identification

**Size:** 16 bits | **Position:** Bytes 4-5, Bits 32-47

Uniquely identifies each IP packet. Used when a packet is **fragmented** — all fragments of the same packet share the same Identification.

```
Original packet: ID = 12345
    ↓ split into fragments
Fragment 1: ID = 12345, Offset = 0
Fragment 2: ID = 12345, Offset = 1480
Fragment 3: ID = 12345, Offset = 2960
```

The receiver uses the Identification to group fragments and reassemble them into the original packet.

---

### Flags

**Size:** 3 bits | **Position:** Bytes 4-5, Bits 48-50

Three flags control fragmentation behavior:

| Bit | Flag | Purpose |
|-----|------|---------|
| 0 | Reserved | Must be 0 (unused) |
| 1 | DF (Don't Fragment) | 1 = Do NOT fragment this packet |
| 2 | MF (More Fragments) | 1 = More fragments coming |

```
DF = 0 → May be fragmented if needed
DF = 1 → Do NOT fragment — drop if exceeds MTU

MF = 0 → This is the last fragment (or not fragmented)
MF = 1 → More fragments are coming
```

---

### Fragment Offset

**Size:** 13 bits | **Position:** Bytes 4-5, Bits 51-63

Tells the receiver **where this fragment sits** in the original packet.

Measured in **8-byte chunks** (64-bit units), not bytes:

```
Offset = 185 → 185 × 8 = 1,480 bytes into the original data
```

Why 8-byte units? 13 bits can store 0-8191, but in bytes that would only cover 8KB. With 8-byte units, it covers 8191 × 8 = 65,528 bytes — enough for the max IP packet.

Example:

```
Original data: 4,000 bytes
MTU: 1,500 bytes (1,480 bytes data per fragment after header)

Fragment 1: Offset = 0,     MF = 1, Data = bytes 0-1479
Fragment 2: Offset = 185,   MF = 1, Data = bytes 1480-2959
Fragment 3: Offset = 370,   MF = 0, Data = bytes 2960-3999
```

---

### TTL (Time To Live)

**Size:** 8 bits | **Position:** Byte 8, Bits 64-71

**Purpose:** Prevents packets from circulating forever if they can't find their destination.

How it works:

```
Packet leaves with TTL = 64
    ↓
Router 1: TTL = 63 (decrement by 1)
    ↓
Router 2: TTL = 62
    ↓
Router 3: TTL = 61
    ...
    ↓
TTL reaches 0: Router drops the packet, sends ICMP "Time Exceeded" back to source
```

Typical initial values:

| OS | Typical TTL |
|----|-------------|
| Linux | 64 |
| Windows | 128 |
| Network gear | 255 |

**Practical use — traceroute:** TTL is decremented by 1 at each hop. When TTL hits 0, the router sends back an ICMP "Time Exceeded" message. By incrementing TTL from 1, traceroute maps each router along the path.

---

### Protocol

**Size:** 8 bits | **Position:** Byte 8, Bits 72-79

Tells the receiver what **transport layer protocol** is inside this IP packet.

| Protocol Number | Name | Purpose |
|----------------|------|---------|
| 1 | ICMP | Ping, error reporting |
| 6 | TCP | Reliable, connection-oriented |
| 17 | UDP | Fast, connectionless |
| 47 | GRE | VPN tunnels |
| 50 | ESP | IPSec encryption |
| 51 | AH | IPSec authentication |
| 89 | OSPF | Routing protocol |

```
IP layer: "Here's a packet"
    ↓
Reads Protocol = 6
    ↓
Hands payload to TCP handler
```

---

### Header Checksum

**Size:** 16 bits | **Position:** Bytes 8-9, Bits 80-95

**Error-checking** — verifies that the IP header arrived intact.

How it works:

```
Sender:  Calculates sum of all 16-bit words in header
         Takes ones' complement
         Stores result in Header Checksum field

Receiver: Calculates same sum
          Compares to Header Checksum field
          If match → header is good
          If mismatch → header corrupted → packet dropped
```

Important: Every router that decrements TTL must **recalculate the checksum** because the header changed.

What it protects:

| Protected | NOT Protected |
|-----------|---------------|
| IP header only | Data payload (TCP/UDP have their own checksums) |
| Bit flips | Reordered packets |
| Corrupted header | Deliberate tampering |

---

### Source IP Address

**Size:** 32 bits (4 bytes) | **Position:** Bytes 12-15

The IP address of the **sender** of this packet. Used by the recipient to send replies back.

```
Source IP: 192.168.1.20
```

When google.com responds to your request, it uses your Source IP as its Destination IP.

---

### Destination IP Address

**Size:** 32 bits (4 bytes) | **Position:** Bytes 16-19

The IP address of the **intended recipient** of this packet. Used by every router along the path to forward the packet.

```
Destination IP: 142.250.x.x
```

Routing decisions are based on Destination IP only. Routers don't use Source IP for forwarding.

---

### Options

**Size:** 0-40 bytes | **Position:** Bytes 20-59 (if IHL > 5)

Optional field rarely used in practice. When IHL > 5, options are present.

Examples:

| Option | Purpose |
|--------|---------|
| Record Route | Track the path a packet takes |
| Timestamp | Record time at each router |
| Loose Source Routing | Specify routers packet must pass through |
| Strict Source Routing | Specify exact path packet must take |

In practice: Most packets have IHL = 5 (no options). Options are used for diagnostics and specialized networking.

---

## Layering Summary

```
┌─────────────────────────────────────────────────────────┐
│                    IP Packet                             │
├─────────────────────────────────────────────────────────┤
│ Source IP │ Dest IP │ TTL │ Protocol │ Checksum │ ... │  ← IP Header (20-60 bytes)
├─────────────────────────────────────────────────────────┤
│                  Payload (TCP/UDP/ICMP)                  │  ← Transport Layer
├─────────────────────────────────────────────────────────┤
│                      Application Data                    │  ← Application Layer
└─────────────────────────────────────────────────────────┘
```

The Protocol field bridges Layer 3 (IP) and Layer 4 (TCP/UDP).

---

## Key Takeaways

1. **Version** tells the receiver whether this is IPv4 or IPv6
2. **IHL** tells where the header ends and data begins
3. **DSCP/ECN** handle quality of service and congestion signaling
4. **Total Length** specifies the full packet size (max 65,535 bytes)
5. **Identification + Flags + Fragment Offset** handle packet fragmentation
6. **TTL** prevents routing loops by dropping packets that traverse too many hops
7. **Protocol** tells the receiver which transport protocol to hand the payload to
8. **Header Checksum** validates the IP header wasn't corrupted
9. **Source/Destination IP** identify who sent and who should receive the packet

---

## ICMP (Internet Control Message Protocol)

### What is ICMP?

**ICMP = Internet Control Message Protocol**

It lives in **Layer 3** (Network Layer), alongside IP. It is **encapsulated inside an IP packet** — the IP header's Protocol field is set to `1` to indicate ICMP.

```
┌────────────────────────────────────────┐
│ ICMP Message ( encapsulated in IP )    │
│   Protocol = 1 (ICMP)                 │
└────────────────────────────────────────┘
         ↓ embedded in
┌────────────────────────────────────────┐
│ IP Header (Protocol = 1)               │
│   Source IP                            │
│   Destination IP                       │
└────────────────────────────────────────┘
```

---

### ICMP vs Ports

| Concept | Layer | Identifies |
|---------|-------|------------|
| IP Address | Layer 3 | Which **device/host** |
| Port Number | Layer 4 | Which **application** on that device |

**ICMP has no ports** — it's Layer 3. It communicates with hosts, not applications.

```
ping 192.168.1.1      ← targeting a device (IP address)
vs
curl 192.168.1.1:8080 ← targeting port 8080 on that device
```

---

### Purpose

Designed for **operational diagnostic and informational status messages**:

| Message | When it happens |
|---------|-----------------|
| Host unreachable | Destination network/device is down |
| Port unreachable | No application listening on that port |
| Fragmentation needed | Packet too large, needs to be split |
| TTL expired | Packet crossed too many routers (prevents infinite loops) |

---

### Operation

**ICMP uses IP directly** — it doesn't use ports or have listeners like TCP/UDP.

```
ICMP Message → wrapped in IP packet → Protocol field = 1 (ICMP)
```

The OS handles ICMP messages directly. No user application needs to be running.

```
You: ping 8.8.8.8
Target's OS: processes ICMP, sends reply
No application needed — kernel handles it
```

---

### Key Analogy

```
TCP/UDP = You knock on a door (port), someone answers (application)
ICMP    = You yell "Hey!" across the room, anyone can hear it (host-level)
```

---

### Tools That Use ICMP

- **ping** — sends Echo Request, waits for Echo Reply
- **traceroute** — uses TTL expiration to map each hop along the path

---

## ICMP Packet Header Structure

The ICMP header is **much simpler than IP** — only 8 bytes minimum.

```
Byte 0: Type (8 bits)
Byte 1: Code (8 bits)
Bytes 2-3: Checksum (16 bits)
Bytes 4-7: Rest of Header (32 bits)
```

| Offset | Size | Field | Purpose |
|--------|------|-------|---------|
| 0 | 1 byte | Type | What kind of ICMP message |
| 1 | 1 byte | Code | More specific info about the type |
| 2-3 | 2 bytes | Checksum | Error checking (same concept as IP header checksum) |
| 4-7 | 4 bytes | Rest of Header | Content varies by Type/Code |

---

### Type and Code Together

```
Type = What happened
Code = Details about what happened
```

| Type | Name | Common Codes |
|------|------|--------------|
| 0 | Echo Reply | 0 (Echo reply) |
| 3 | Destination Unreachable | 0 (Network unreachable), 1 (Host unreachable), 3 (Port unreachable) |
| 8 | Echo Request | 0 (Echo request — ping) |
| 11 | Time Exceeded | 0 (TTL expired), 1 (Fragment reassembly time exceeded) |

**Example combinations:**

```
Type = 3, Code = 1 → "Host Unreachable"
Type = 8, Code = 0 → "Echo Request" (ping)
Type = 0, Code = 0 → "Echo Reply" (pong)
```

---

### The Rest of Header Field

Content varies by Type:

**For Echo Request/Reply (ping):**

```
Bytes 4-5: Identifier (helps match request to reply)
Bytes 6-7: Sequence Number (tracks multiple pings)
```

```
ping 8.8.8.8
Echo Request: Type=8, Code=0, ID=1234, Seq=1
Echo Reply:   Type=0, Code=0, ID=1234, Seq=1
```

**For Destination Unreachable:**

```
Bytes 4-7: Unused (set to 0)
```

**For Time Exceeded:**

```
Bytes 4-7: Unused
```

---

## ICMP Blocking & Security

### Why Firewalls Block ICMP

| Attack | How it works |
|--------|--------------|
| **Smurf attack** | Attacker sends ping to broadcast address with spoofed source IP → victim gets flooded with replies |
| **Covert channel** | Data can be hidden inside ICMP packets (since it's not commonly monitored) |

Some firewalls block ICMP entirely as a security measure.

### The Problem

Blocking ICMP breaks diagnostic tools:

```
ping 8.8.8.8 → blocked by firewall → "Request timed out"
```

But remember: **ping is just an IP packet wrapping ICMP**. There's no special "ping port" — it's just an ICMP Echo Request (Type=8, Code=0).

### The Trade-off

Disabling ICMP sounds secure — but it breaks legitimate functionality:

```
Breaking Path MTU Discovery (PMTUD):
  Router needs to tell sender "your packet is too big, needs fragmentation"
  This is done via ICMP "Fragmentation Needed" message
  If ICMP is blocked → sender never learns the MTU limit → connection stalls or times out
```

### Best Practice

| Approach | Problem |
|----------|---------|
| Block all ICMP | Breaks PMTUD, legitimate diagnostics |
| Allow all ICMP | Vulnerable to Smurf, covert channels |
| Allow specific ICMP types | Best practice |

**Selective ICMP allows:**
- Echo Reply (Type=0) — ping responses
- Time Exceeded (Type=11) — traceroute
- Destination Unreachable (Type=3) — PMTUD and error reporting

---

## How PING Works

### The Process

```
You: ping 8.8.8.8

Step 1: Your machine sends ICMP Echo Request (Type=8, Code=0)
Step 2: 8.8.8.8 receives it
Step 3: 8.8.8.8's OS processes it
Step 4: 8.8.8.8 sends ICMP Echo Reply (Type=0, Code=0)
Step 5: Your machine receives the reply
```

### Packet Flow

```
My Machine                          8.8.8.8
    │                                    │
    │──── ICMP Echo Request (Type=8) ────→│
    │                                    │
    │←─── ICMP Echo Reply (Type=0) ─────│
    │                                    │
```

### What ping output shows

```
64 bytes from 8.8.8.8: icmp_seq=1 ttl=117 time=10ms
```

| Field | Meaning |
|-------|---------|
| `64 bytes` | Size of the ICMP reply |
| `icmp_seq` | Sequence number — tracks multiple pings |
| `ttl=117` | Reply had TTL=117 when it left Google (started with 64 or 128, decremented by routers along the path) |
| `time=10ms` | Round-trip time (request sent → reply received) |

---

## How Traceroute Works

### The Problem

```
ping 8.8.8.8
Reply from 8.8.8.8: time=10ms
```

You only see the destination — not the path in between.

### The Solution: TTL Expiration

Traceroute exploits the TTL field. Each router decrements TTL by 1. When TTL hits 0, the router drops the packet and sends back an **ICMP Time Exceeded** message.

### Step-by-Step Process

```
My Machine                          Router A    Router B    Router C    Google
    │                                  │           │           │           │
    │──── TTL=1 ──────────────────────→│ (drops)   │           │           │
    │←─── ICMP Time Exceeded (from A) │           │           │           │
    │                                    │           │           │           │
    │──── TTL=2 ──────────────────────────→│ (drops)  │           │           │
    │←─── ICMP Time Exceeded (from B)   │           │           │           │
    │                                    │           │           │           │
    │──── TTL=3 ────────────────────────────────→│ (drops)   │           │
    │←─── ICMP Time Exceeded (from C)   │           │           │           │
    │                                    │           │           │           │
    │──── TTL=4 ────────────────────────────────→│ (reaches) │           │
    │←─── ICMP Echo Reply (from Google)  │           │           │           │
```

### What traceroute output looks like

```
traceroute to google.com, 30 hops max

1:  router.local (192.168.1.1)      1ms    1ms    1ms
2:  10.0.0.1 (isp.gateway)          5ms    4ms    5ms
3:  * * * (router doesn't reply ICMP)
4:  72.14.215.85 (google.edge)      10ms   9ms    11ms
5:  142.250.x.x (google.com)        10ms   10ms   10ms
```

| Symbol | Meaning |
|--------|---------|
| `* * *` | Router doesn't send ICMP Time Exceeded — often blocked by firewalls |

Each line shows the router's IP and 3 round-trip times (sends 3 probes per TTL).

---

## Key Difference

| Tool | Purpose | ICMP Types Used |
|------|---------|----------------|
| **ping** | Test connectivity + measure latency | Echo Request (Type=8), Echo Reply (Type=0) |
| **traceroute** | Map the path to a destination | Time Exceeded (Type=11) from each router |

---

## What's Next?

Lecture 10 continues with: **PING** (Echo Request/Reply), **TraceRoute** (TTL-based path mapping), and capturing ICMP packets.


