# Network Performance — Section 7

<details>
<summary><b>Section 7: Network Performance</b></summary>

### Lectures

- [x] 37. What is this section?
- [x] 38. MSS vs MTU vs PMTUD
- [ ] 39. Nagle's Algorithm's Effect on Performance
- [ ] 40. Delayed Acknowledgment Effect on Performance
- [ ] 41. Cost of Connection Establishment
- [ ] 42. TCP Fast Open
- [ ] 43. Listening Server
- [ ] 44. TCP Head of line blocking
- [ ] 45. The importance of Proxy and Reverse Proxies
- [ ] 46. Load Balancing at Layer 4 vs Layer 7
- [ ] 47. Network Access Control to Database Servers

---

### Study Notes

#### Lecture 38 — MSS vs MTU vs PMTUD

### Lecture Notes — Discussion

---

### MTU (Maximum Transmission Unit) — In Isolation

**MTU = The largest single piece of data (in bytes) that a specific network link can carry in one frame.**

- It's a **physical limit of a wire** — each network interface has a maximum frame size it can handle
- Not a software setting — it's a hardware/interface property (cable, switch port, NIC driver)
- **Belongs to Layer 2** — it's the link's own rule
- Everything above it (IP, TCP, app) must respect it, but MTU itself doesn't care about them

```
Ethernet NIC → MTU = 1500 bytes
```

No single frame on that link can exceed 1500 bytes. Not the IP layer, not TCP, not your app — nobody gets to send more than 1500 bytes in one piece on that wire.

---

### MSS (Maximum Segment Size) — In Isolation

**MSS = The largest amount of data (payload) that TCP is willing to carry in one segment.**

- It's **TCP's own personal limit** on how big each chunk of application data can be
- **Not a physical wire limit** — it's a **protocol agreement** between the two endpoints, negotiated during the 3-way handshake
- **Belongs to Layer 4 (TCP)**
- **Excludes ALL headers** — data payload only

```
TCP says: "I will never send more than 1460 bytes of your data in one segment."
```

**Where does the number come from?** MSS is **derived** from MTU. TCP looks at the path and calculates:

```
MSS = MTU - IP Header - TCP Header
    = 1500 - 20 - 20 = 1460 bytes (IPv4)
    = 1500 - 20 - 40 = 1440 bytes (IPv6)
```

**Key distinction:** MSS is just a number negotiated at connection setup. It happens to be calculated based on MTU, but conceptually they're separate things — MTU is a hardware limit, MSS is a protocol agreement.

---

### MTU vs MSS — Comparison

| | MTU | MSS |
|---|---|---|
| **Layer** | Layer 2 (Data Link) | Layer 4 (Transport) |
| **Who sets it?** | Network interface (hardware) | TCP endpoints (negotiated in handshake) |
| **What it limits?** | Entire frame (all headers + data) | Data payload only |
| **Includes headers?** | ✅ Yes | ❌ No |
| **Can it change?** | Per interface, configurable | Per connection, negotiated |
| **Memory trick** | The size of the **whole envelope** | The size of the **letter inside** |

---

### PMTUD (Path MTU Discovery) — In Isolation

**PMTUD answers one question:** "What is the **smallest MTU** on the path between me and the destination?"

Because the path isn't just one link — it's many routers chained together, each with potentially different MTUs.

```
Your Server ── MTU 9000 ──► Router A ── MTU 1500 ──► Router B ── MTU 1500 ──► Destination
              (jumbo)           (internet)           (internet)
```

The bottleneck is Router A at 1500. PMTUD finds that number.

**Mechanism — 3 steps:**

```
Step 1: Sender sends packet with DF bit = 1 (Don't Fragment)
        Packet size exceeds path MTU

Step 2: Router can't fragment it (DF=1 says "don't break it")
        Router DROPS the packet
        Router sends ICMP "Fragmentation Needed" + its MTU value

Step 3: Sender reads ICMP message
        Learns: "Smallest MTU on path = X"
        Adjusts MSS accordingly
        From now on: sends packets that fit
```

**DF (Don't Fragment) bit — why it matters:**

| DF bit | Router behavior | PMTUD works? |
|--------|----------------|--------------|
| **DF = 0** (default for non-TCP) | Router silently fragments the packet | ❌ No — packet gets fragmented, no ICMP sent, sender never learns |
| **DF = 1** (TCP default) | Router drops + sends ICMP "Fragmentation Needed" | ✅ Yes — sender discovers the real MTU |

**TCP sets DF = 1 by default specifically so PMTUD can work.**

**The ONE thing that breaks PMTUD: ICMP is blocked.**

```
Sender: sends packet DF=1, too big for path
  ↓
Router: drops it, sends ICMP "Fragmentation Needed"
  ↓
Firewall: 🚫 BLOCKS ICMP
  ↓
Sender: never learns the MTU limit
  ↓
Keeps sending oversized packets
  ↓
They ALL get dropped silently
  ↓
💀 Connection hangs / timeouts
```

This is the #1 real-world PMTUD failure. Not a bug in TCP. Not a bug in the network. A firewall blocking a single ICMP message type.

---

### Real-World Example: Sending a High-Quality Image

**Scenario:** Upload a **5 MB high-quality JPEG** (5,000,000 bytes) from browser to server.

**Normal Path (all MTU ≥ 1500):**

```
5,000,000 bytes ÷ 1460 (MSS) = ~3,425 segments

Each segment: 1,460 bytes data + 20 bytes TCP header = 1,480 bytes (TCP Segment)
Each packet: 1,480 bytes + 20 bytes IP header = 1,500 bytes (fits MTU)

5MB image → 3,425 segments → 3,425 packets of 1500B → ✅ Done
```

**Path with smaller MTU (DF=1, ICMP works — e.g., VPN tunnel at MTU 1400):**

```
First packet (1500B) arrives at VPN Tunnel
  ↓
VPN Tunnel: "I can't carry 1500 bytes. DF=1 says don't fragment me."
  ↓
Drops packet, sends ICMP: "Fragmentation Needed. My MTU = 1400"
  ↓
OS learns: Path MTU = 1400 → New MSS = 1400 - 20 - 20 = 1360
  ↓
From now on: segments carry 1360 bytes of data
  ↓
New packet size = 1360 + 20 + 20 = 1400 ✅ Fits!

5MB image → 3,425 segments (1500B) → dropped at VPN
→ ICMP says "MTU=1400" → shrink to 1360 MSS
→ 3,676 segments of 1400B → ✅ Done (slower but works)
```

**Path with smaller MTU (ICMP blocked — the disaster):**

```
5MB image → 3,425 segments (1500B) → dropped at VPN
→ ICMP blocked → sender never learns
→ ALL packets dropped → 💀 FAILURE
```

**What you experience as a user:**

| Scenario | What you see |
|---|---|
| Normal | Image uploads smoothly |
| PMTUD working | Image uploads fine, maybe slightly slower over VPN |
| PMTUD broken | Image upload hangs, times out, or fails — especially over VPN/tunnel |

---

### IPv6 MSS — Why 1440, Not 1460?

```
IPv4 Header:  20 bytes minimum (variable, 20-60)
IPv6 Header:  40 bytes fixed (no options, always 40)

IPv4 MSS = 1500 - 20 - 20 = 1460
IPv6 MSS = 1500 - 20 - 40 = 1440
```

IPv6 moved options out of the base header into Extension Headers. This makes routing faster (routers don't need to parse options) but costs 20 extra bytes of overhead.

---

### Debugging MTU Issues (Practical Commands)

```bash
# Find the MTU to a destination (Linux)
tracepath 8.8.8.8

# Send a specific size packet with DF=1 (tests if path can handle it)
ping -M do -s 1472 8.8.8.8
# 1472 + 28 (IP+ICMP headers) = 1500 → if this works, MTU ≥ 1500

# If above fails, try smaller:
ping -M do -s 1400 8.8.8.8

# Windows equivalent:
ping -f -l 1472 8.8.8.8
```

---

### Practical Impact for Full-Stack Engineers

| Scenario | What happens | What to do |
|---|---|---|
| **Docker + Kubernetes** | Overlay networks may have lower MTU (e.g., 1450) | Set `MTU` in CNI config; otherwise packets fragment or drop |
| **VPN (WireGuard, OpenVPN)** | Adds encapsulation headers, reduces effective MTU | Lower MTU on VPN interface (e.g., `MTU=1280`) |
| **Large file uploads** | Exceeds MTU → fragmentation or drops | Server should handle PMTUD; client should handle errors gracefully |
| **Cloud (AWS VPC)** | Default MTU = 9000 (jumbo frames) within VPC | Works internally, but traffic to internet = 1500 |
| **Database queries** | Large result sets may exceed path MTU | Connection poolers help; PMTUD must work |
| **Firewall blocking ICMP** | PMTUD breaks → connections hang for large payloads | Don't block all ICMP; allow Type 3 Code 4 (Fragmentation Needed) |

---

### Key Takeaways — Lecture 38

1. **MTU** = physical limit of a network link (Layer 2), set by hardware
2. **MSS** = TCP's data payload limit (Layer 4), negotiated in handshake
3. **MSS = MTU - IP Header - TCP Header** (1460 for IPv4, 1440 for IPv6)
4. **PMTUD** = discovering the smallest MTU along the path using DF=1 + ICMP
5. **DF bit** = tells routers "don't fragment" (TCP sets this to enable PMTUD)
6. **ICMP blocked** = PMTUD breaks = silent connection failures for large data
7. **The frame dictates everything above it** — the smallest link in the path determines your effective MSS

---

#### Lecture 39 — Nagle's Algorithm's Effect on Performance

<!-- Discussion notes will be added here -->

#### Lecture 40 — Delayed Acknowledgment Effect on Performance

<!-- Discussion notes will be added here -->

#### Lecture 41 — Cost of Connection Establishment

<!-- Discussion notes will be added here -->

#### Lecture 42 — TCP Fast Open

<!-- Discussion notes will be added here -->

#### Lecture 43 — Listening Server

<!-- Discussion notes will be added here -->

#### Lecture 44 — TCP Head of Line Blocking

<!-- Discussion notes will be added here -->

#### Lecture 45 — The Importance of Proxy and Reverse Proxies

<!-- Discussion notes will be added here -->

#### Lecture 46 — Load Balancing at Layer 4 vs Layer 7

<!-- Discussion notes will be added here -->

#### Lecture 47 — Network Access Control to Database Servers

<!-- Discussion notes will be added here -->

</details>
