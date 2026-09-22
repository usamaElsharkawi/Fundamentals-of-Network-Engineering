# Network Routing

<details>
<summary><b>Section 8: Network Routing</b></summary>

### Lectures

- [x] 48. Fundamentals of Network Routing

---

### Study Notes

#### Lecture 48 — Fundamentals of Network Routing

---

### Unit 1: Data Links and MAC Addresses

**The most basic concept — two machines talking directly.**

```
Machine A (MAC: AA) ↔ Machine B (MAC: BB)
```

- Every machine has a **unique link address** called a **MAC address**
- A **frame** is the unit of communication at this level
- Frame structure: `[Source MAC] [Destination MAC] [Data]`
- When A sends a frame to B, **every machine on the network receives it**
- Each machine checks: "Is the destination MAC mine?" → Yes → process it, No → drop it
- This is **Layer 2 (Data Link)**
- **Layer 1 (Physical)**: the actual medium — electric pulses, radio, light, fiber

---

### Unit 2: Hub vs Switch

**How frames actually get delivered in a real network.**

**Hub** (dumb device):
- Receives a frame → sends it to **ALL** ports
- Wasteful, inefficient

**Switch** (intelligent hub):
- Learns which machine is on which port
- Builds a table: `Port 1 = A, Port 2 = B, Port 3 = C`
- When A sends to B → switch looks up B → forwards **only** to Port 2
- No unnecessary broadcasting

```
Hub:    A sends to B → everyone gets it
Switch: A sends to B → only B gets it
```

---

### Unit 3: The MAC Address Problem

**MAC addresses are random — they don't scale.**

```
A wants to talk to Z (thousands of machines away)
Switch doesn't know where Z is
→ Broadcasts to everyone
→ Everyone discards except Z
→ Works, but doesn't scale
```

**The core problem:**
- MAC addresses have **no structure** — they're random hex values
- No way to determine where a machine is without scanning
- On a large network, switches constantly flood unknown destinations → **broadcast storm**

**The fix:** IP addresses with **structure**.

---

### Unit 4: IP Addresses — Network + Host

> **"IP address is very similar to an index in databases because it has this concept of a network and a host."**

```
192.168.1.20/24
         ├── Network: 192.168.1 (the elimination key)
         └── Host: 20 (specific machine)
```

**Why it scales:**
- You can **skip entire networks** that don't match yours
- If destination network ≠ your network → don't waste time looking
- Only machines in YOUR network matter for direct communication

---

### Unit 5: Subnet Mask

**The subnet mask tells you where the network ends and the host begins.**

```
IP:       192.168.1.20
Mask:     255.255.255.0

Bitwise AND:
192.168.1.20  AND 255.255.255.0 = 192.168.1.0  → My network
10.0.0.2      AND 255.255.255.0 = 10.0.0.0     → Different network!
```

- Same result = same network = can communicate directly
- Different result = different network = need a gateway

---

### Unit 6: ARP (Address Resolution Protocol)

> **"ARP translates IP → MAC, like DNS translates hostname → IP."**

**When A wants to talk to B on the same network:**

```
1. A knows B's IP (192.168.1.5) but NOT B's MAC
2. A broadcasts: "Who has 192.168.1.5?"
3. B replies: "That's me! My MAC is BB"
4. A stores this in ARP cache and sends directly
```

**Critical rule:** ARP only works on the **same subnet**. Different networks → no ARP.

---

### Unit 7: Default Gateway

**When the destination is outside your network.**

```
A (192.168.1.4) wants to talk to Z (10.0.0.2)
Different networks → can't ARP → can't talk directly

Solution: Send to a ROUTER (gateway)

A's gateway IP: 192.168.1.1 (must be in A's network!)
Gateway has two network cards:
  - One on A's network: 192.168.1.1
  - One on Z's network: 10.0.0.1
```

**The gateway acts as a middleman** between networks.

---

### Unit 8: Encapsulation (Matryoshka Doll)

> **"Each layer wraps the previous one."**

```
┌──────────────────────────────────────────┐
│ Frame (Layer 2)                           │
│  [Src MAC] [Dst MAC] [Payload]          │
│  ┌────────────────────────────────────┐  │
│  │ IP Packet (Layer 3)                │  │
│  │  [Src IP] [Dst IP] [Payload]     │  │
│  │  ┌────────────────────────────┐    │  │
│  │  │ TCP Segment (Layer 4)      │    │  │
│  │  │  [Src Port] [Dst Port]     │    │  │
│  │  │  ┌──────────────────────┐  │    │  │
│  │  │  │ HTTP (Layer 7)       │  │    │  │
│  │  │  └──────────────────────┘  │    │  │
│  │  └────────────────────────────┘    │  │
│  └────────────────────────────────────┘  │
└──────────────────────────────────────────┘
```

Each layer adds its own header and wraps everything inside.

---

### Unit 9: Same Network vs Different Network

| | Same Network | Different Network |
|---|---|---|
| **Can ARP?** | ✅ Yes | ❌ No |
| **Communication** | Direct (via MAC) | Through gateway |
| **How?** | Broadcast ARP → get MAC | ARP gateway → gateway forwards |

---

### Unit 10: ARP Poisoning (Security)

> **"The router sends the MAC address pretending to be the gateway."**

```
Normal:    Client → "Who has gateway?" → Router: "That's me, MAC=X"
Poisoned:  Client → "Who has gateway?" → Attacker: "That's me, MAC=Y"

Client sends all traffic to Attacker instead of Router.
```

This is a **man-in-the-middle** attack using fake ARP replies.

---

### Unit 11: How Gateway Forwarding Works (Detailed)

**Step-by-step: A (192.168.1.4) → Z (10.0.0.2)**

```
Step 1: A knows gateway IP (192.168.1.1)
Step 2: A ARPs for gateway MAC → gets X
Step 3: A creates frame:
        [Dst MAC: X] [IP Packet: Src=192.168.1.4, Dst=10.0.0.2]
Step 4: Switch forwards to router's port (smart switch knows port)
Step 5: Router receives frame → "Dst MAC X = that's me!"
Step 6: Router opens frame → finds IP packet → "Dst IP = 10.0.0.2"
Step 7: Router checks: "10.0.0.2 is NOT my IP"
Step 8: Router checks IP forwarding → enabled!
Step 9: Router checks interfaces → "10.0.0.2 is on my other interface"
Step 10: Router forwards IP packet on that interface
```

---

### Unit 12: MAC Changes, IP Stays

> **"This is the whole power here."**

```
Hop 1 (A → Router):
  Frame: [Src MAC: A] [Dst MAC: X]
  IP:    [Src IP: A] [Dst IP: Z]

Hop 2 (Router → Z):
  Frame: [Src MAC: X] [Dst MAC: D]   ← MACs CHANGED
  IP:    [Src IP: A] [Dst IP: Z]     ← IPs UNCHANGED!
```

**MAC addresses are local** — they only matter on the current link.
**IP addresses are end-to-end** — they stay the same across the entire journey.

---

### Unit 13: IP Forwarding — The Router Bit

> **"There is literally a bit called IP forwarding. If you set it to true, your machine becomes a router."**

```bash
# Linux: Enable IP forwarding
sysctl net.ipv4.ip_forward = 1
```

**Without IP forwarding:**
```
Machine receives packet → IP not for me → DROP
```

**With IP forwarding:**
```
Machine receives packet → IP not for me → Check other interfaces → Forward
```

**Key insight:** A regular machine with two NICs + IP forwarding enabled = a router.

---

### Unit 14: Re-encapsulation at the Router

> **"We cannot just forward an IP packet naked. That doesn't make any sense."**

The router receives:
```
Frame: [Src MAC: A] [Dst MAC: X]  ← X is router's MAC
IP Packet: [Src IP: A] [Dst IP: Z]
```

The router **cannot** forward the same frame. It must create a **new frame**:

```
New Frame: [Src MAC: X] [Dst MAC: D]  ← router's other interface MAC
IP Packet: [Src IP: A] [Dst IP: Z]    ← SAME IP packet inside
```

**How the router gets D's MAC:**
- Router does ARP for 10.0.0.2 on its own network
- "Who has 10.0.0.2?" → D replies: "That's me, MAC=D"
- Router caches this and builds the new frame

---

### Unit 15: Source IP Preservation vs NAT

**In our example:** Source IP stays the same (192.168.1.4).

**On the internet:** Source IP MUST change.

**Why?** Private IP addresses can't exist on the internet:

```
Private IP ranges:
  192.168.0.0/16
  10.0.0.0/8
  172.16.0.0/12
```

> Google has no idea what 10.0.0.2 is. It's a private address.

**NAT (Network Address Translation)** changes the source IP when sending to the internet. The router replaces your private IP with its public IP.

---

### Unit 16: Multiple Gateways — The Problem

**Home setup:** One gateway (your router). Simple.

**Internet:** Thousands of gateways (ISPs, routers, autonomous systems).

**The problem:**

```
A (10.0.0.2) wants to reach C (172.16.6.2)
Default gateway sends it → Intermediate router doesn't know 172.16.6.2 → DROPS
```

One gateway can't know every network. We need **routing tables**.

---

### Unit 17: Routing Tables

> **"Everything is just a bunch of rules that you play with."**

**Routing table fields:**

| Field | Meaning |
|---|---|
| **Destination Network** | Which network you want to reach |
| **Subnet Mask** | Size of the network |
| **Gateway/Next Hop** | Which machine to send it to |
| **Interface** | Which NIC to use (Ethernet, Wi-Fi, Docker) |
| **Metric** | Priority (lower = better/preferred) |

**Special entries:**
- `0.0.0.0/0` = default route (anything → go here)
- Direct link = no gateway needed (can ARP directly)

**Example:**
```
Destination     Gateway         Interface   Metric
0.0.0.0/0       192.168.1.1     eth0        100    ← default
172.16.6.0/24   192.168.1.4     eth0        10     ← direct, faster
10.0.0.0/8      192.168.1.1     eth0        100    ← via gateway
```

---

### Unit 18: Two Ways to Fix Cross-Network Routing

**Scenario:** A (10.0.0.2) wants to reach C (172.16.6.2)

#### Fix 1: Per-Machine Route

```bash
# On EVERY machine:
sudo ip route add 172.16.6.0/24 via 192.168.1.4
```

- ✅ Fast (direct path, skips intermediate router)
- ❌ Must configure EVERY machine

#### Fix 2: Gateway Route (Centralized)

```bash
# On DEFAULT GATEWAY only:
ip route add 172.16.6.0/24 via 192.168.1.4
```

- ✅ Configure once, all machines benefit
- ❌ Slower (all traffic goes through router first)

> **Trade-off:** Configuration simplicity vs speed.

---

### Unit 19: How the Default Gateway Solves Return Traffic

**When C replies to A:**

```
C (172.16.6.2) wants to reply to A (10.0.0.2)
C checks: "10.0.0.2 is not my network"
C sends to its default gateway (172.16.6.1)
Gateway checks routing table → finds route to 10.0.0.0/8 → forwards
```

The default gateway on C's side knows how to reach A's network.

---

### Unit 20: Routing Protocols — They All Update the Routing Table

> **"All these protocols do is just write routes to your routing table. That's it."**

| Protocol | What it does |
|---|---|
| **DHCP** | Assigns IP addresses, provides default gateway |
| **Kernel** | Manually added routes, direct connections |
| **OSPF** | Builds network graph, calculates shortest path, writes routes |
| **BGP** | Internet backbone, routes between autonomous systems |

**OSPF in simple terms:**
```
1. Router discovers all directly connected networks
2. Shares this with all other routers
3. Each router builds a complete graph of the network
4. Calculates shortest path
5. Writes best routes to routing table
```

**BGP:** Similar concept but for the internet — routes between ISPs and large networks (autonomous systems).

---

### Unit 21: Practical Demo — Docker Routing

**Setup:**
```
Machine A (192.168.4.x) — the user's Mac
  └── Wants to reach Docker container (172.17.0.2:5432)
          on Machine B (192.168.4.x — same network)
              └── Docker Host
                  └── Docker Container (172.17.0.2)
```

**Problem:** Machine A has no route to 172.17.0.0/24.

```bash
# Check current routes
ip route show

# Try to connect — fails
telnet 172.17.0.2 5432
# Tries default gateway → no path → fails
```

**Fix:** Add a route on Machine A.

```bash
sudo ip route add 172.17.0.0/16 via 192.168.4.1
```

**Also required:** IP forwarding enabled on Machine B.

```bash
# On Machine B (Docker host):
sysctl net.ipv4.ip_forward = 1
```

**Result:**
```bash
telnet 172.17.0.2 5432  # ✅ Works!
traceroute 172.17.0.2   # Shows path through 192.168.4.1
```

---

### Unit 22: Listing Routing Tables

```bash
# Linux
ip route show
netstat -rn          # Also works on Mac

# Windows
route print          # Cleanest output
```

---

### Unit 23: Complete Summary of Networking

```
MAC Addresses (Layer 2)
  ↓ Doesn't scale (random, no structure)
IP Addresses + Subnet Mask (Layer 3)
  ↓ Enables network identification
Default Gateway / Router (Layer 3 forwarding)
  ↓ Forwards between networks using IP forwarding
Routing Table (Rules for where to send packets)
  ↓ Updated by protocols
DHCP, OSPF, BGP
```

> **"Each device has a unique link address (MAC). Direct link doesn't scale. We needed IP. Then networking. Then routing. Everything is just rules in a routing table."**

---

### Key Quotes from the Instructor

> **"Mac addresses are random. There is no logic when it comes to them."**

> **"IP address is very similar to an index in databases because it has this concept of a network and a host."**

> **"You can trash all of this scrap and build your own. Go build your own thing."**

> **"If you don't trust any of this, you don't even need to use IP. Just use Mac addresses and build your own frame."**

> **"There is literally a bit called IP forwarding. If you set that to true, your machine becomes a router."**

> **"All what these protocols do is just they write routes to your routing table. That's all what they do."**

> **"Now anything is basically just a bunch of rules that you play with them."**

</details>
