# Network Performance — Section 7

<details>
<summary><b>Section 7: Network Performance</b></summary>

### Lectures

- [x] 37. What is this section?
- [x] 38. MSS vs MTU vs PMTUD
- [x] 39. Nagle's Algorithm's Effect on Performance
- [x] 40. Delayed Acknowledgment Effect on Performance
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

### Lecture Notes — Discussion

---

### Unit 1: The Problem — Why Does Nagle's Algorithm Exist?

Back to our **header overhead** lessons from IP and TCP sections:

```
TCP segment with 1 byte of data:
  20 bytes TCP header + 20 bytes IP header + 1 byte data = 41 bytes sent
  Efficiency: 1/41 = 2.4%
```

**The problem:** When an application sends many small writes, each one becomes its own segment. Each segment carries 40+ bytes of headers for just a few bytes of actual data. Terrible waste.

**Real example — Chat app typing character by character:**

```
send("H")  → 41 bytes sent (2.4% efficient)
send("e")  → 41 bytes sent (2.4% efficient)
send("l")  → 41 bytes sent (2.4% efficient)
send("l")  → 41 bytes sent (2.4% efficient)
send("o")  → 41 bytes sent (2.4% efficient)
─────────────────────────────────────────────
Total: 205 bytes sent to deliver "Hello" (5 bytes of data)
```

**That's the problem Nagle's Algorithm solves.**

---

### Unit 2: The Solution — How Nagle's Algorithm Works

Nagle's idea:

> **Don't send small pieces. Buffer them and send them together.**

**The Rule:**

```
If there is unacknowledged data still in flight → buffer new data
If no unacknowledged data → send immediately
```

```
send("H") → No data in flight → SEND NOW (1 packet)
                ↓
          Waiting for ACK...

send("e") → Data in flight (H unacknowledged) → BUFFER
send("l") → Data in flight → BUFFER
send("l") → Data in flight → BUFFER
send("o") → Data in flight → BUFFER

ACK for "H" arrives → Now send "ello" as ONE packet
```

**Result:**

```
Before Nagle: 5 packets = 205 bytes (for 5 bytes of data)
After Nagle:  2 packets = 82 bytes (for 5 bytes of data)
```

Even better if all 5 characters arrive before the ACK:

```
1 packet: "Hello" = 41 bytes (97.6% efficient)
```

---

### Unit 3: The Core Mechanism

```
┌──────────────────────────────────────────────────────┐
│              Nagle's Algorithm                        │
│                                                       │
│  When app calls send(data):                           │
│                                                       │
│   ┌─────────────────────────────┐                     │
│   │ Is there unacknowledged     │                     │
│   │ data in flight?             │                     │
│   └──────────┬──────────────────┘                     │
│              │                                       │
│         YES  │         NO                             │
│              │         │                              │
│              ▼         ▼                              │
│     Buffer the    Send immediately                   │
│     new data      (no unacked data)                  │
│              │                                       │
│   When ACK arrives for previous data:                │
│   → Flush buffered data as one segment                │
│   → Repeat                                            │
└──────────────────────────────────────────────────────┘
```

**Key insight:** Nagle's algorithm is essentially saying — *"Wait a tiny bit. If more data is coming, batch it. If not, send what you have."*

---

### Unit 4: The Trade-off

| Nagle **Enabled** (default) | Nagle **Disabled** |
|---|---|
| Less header overhead | More header overhead |
| Adds latency (waits for ACK) | Lower latency |
| Good for bulk transfer | Good for real-time apps |
| Efficient bandwidth | Immediate delivery |

**Latency cost:**

```
send("H") → sent immediately
send("e") → WAITS for ACK of "H" (up to 1 RTT)
send("l") → WAITS
send("l") → WAITS
send("o") → WAITS

Total delay: ~1 RTT (not 5 RTTs — all buffered data sent together)
```

---

### Unit 5: Maximum Latency — Correction

**Wrong formula:** `number of segments × RTT`

**Correct answer:** `~1 RTT`

Why: When the ACK finally arrives, ALL buffered data is flushed together as one burst. You don't send them one by one with one RTT each.

```
t=0:   send("H") → SENT IMMEDIATELY
t=0:   send("e") → BUFFERED
t=0:   send("l") → BUFFERED
t=0:   send("l") → BUFFERED
t=0:   send("o") → BUFFERED
                ↓
t=RTT: ACK for "H" arrives
                ↓
t=RTT: "ello" SENT AS ONE SEGMENT ← ALL at once!

Total time from first send to last data: ~1 RTT
```

---

### Unit 6: The Exact Rule (Formal Definition)

One precise rule, two branches:

```
If there is unacknowledged data in flight:
   → Buffer the new data (don't send)

If there is NO unacknowledged data in flight:
   → Send immediately (even if data is tiny)
```

**Important nuance:** Nagle doesn't control WHEN the ACK comes. It only controls whether to send or buffer. ACK timing depends on:
- Network speed (RTT)
- Delayed ACKs (receiver waits before acknowledging — Lecture 40)
- Receiver's buffer processing speed

---

### Unit 7: Interaction with Delayed ACKs (Preview)

Nagle and Delayed ACKs can create a problematic interaction:

```
Nagle says:       "Wait for ACK before sending more"
Delayed ACK says: "Wait before sending ACK"

Both waiting → potential deadlock → latency spikes
```

**Example:**

```
Client: send("H") → sent immediately
Client: send("e") → buffered (waiting for ACK)
                ↓
Server: received "H" → Delayed ACK timer starts (waits 40ms or for 2nd segment)
                ↓
Client: still waiting for ACK (Nagle won't send "e")
Server: waiting 40ms before sending ACK (Delayed ACK)
                ↓
t=40ms: Server sends ACK
                ↓
Client: ACK received → sends "ello"
```

**Result:** Extra 40ms delay from the interaction. We'll cover this in depth in Lecture 40.

---

### Unit 8: How to Disable Nagle — `TCP_NODELAY`

When real-time performance is needed, turn Nagle off:

```c
// C / Linux
int flag = 1;
setsockopt(socket, IPPROTO_TCP, TCP_NODELAY, &flag, sizeof(flag));
```

```javascript
// Node.js
socket.setNoDelay(true);
```

```python
# Python
socket.setsockopt(socket.IPPROTO_TCP, socket.TCP_NODELAY, 1)
```

**What this does:** Tells TCP — "Never buffer my data. Send every write immediately."

**Trade-off:** Lowest latency but maximum header overhead.

---

### Unit 9: Real-World Decision Framework

```
                    ┌─────────────────────────┐
                    │  Is your app real-time?   │
                    └──────────┬──────────────┘
                               │
                    YES ───────┴─────── NO
                    │                     │
                    ▼                     ▼
            TCP_NODELAY ON        Nagle ON (default)
            Low latency            Efficient bandwidth
            High overhead          Low overhead
            Example:               Example:
            • Chat typing          • File upload
              indicators             • Email sync
            • Game state           • API bulk requests
              updates                • Log streaming
            • VoIP/Video           • Database replication
```

---

### Key Takeaways — Lecture 39

1. **Nagle's Algorithm** buffers small writes and sends them together to reduce header overhead
2. **The Rule:** If unacknowledged data in flight → buffer; if none → send immediately
3. **Max latency added:** ~1 RTT (not N × RTT — all buffered data sent at once)
4. **Trade-off:** Efficiency vs latency — Nagle for bulk, TCP_NODELAY for real-time
5. **Nagle + Delayed ACKs** can interact causing extra delay (covered in Lecture 40)
6. **TCP_NODELAY** disables Nagle — used by real-time apps (chat, gaming, VoIP)
7. **Header overhead** is the root cause — 40+ bytes of headers for 1 byte of data = 2.4% efficiency

---

#### Lecture 40 — Delayed Acknowledgment Effect on Performance

### Lecture Notes — Discussion

---

### Unit 1: What is a Delayed ACK?

**Delayed ACK = The receiver holds off on sending the ACK for a short time.**

Why would it do that? **Efficiency.** Same logic as Nagle but on the receiver side.

```
Without Delayed ACK:
  receive 1 byte -> send ACK (40 bytes of overhead for 1 byte!)
  receive 1 byte -> send ACK (40 bytes of overhead for 1 byte!)
  receive 1 byte -> send ACK (40 bytes of overhead for 1 byte!)

With Delayed ACK:
  receive 1 byte -> wait...
  receive 1 byte -> wait...
  receive 1 byte -> NOW send ONE ACK for all three
```

**Same idea as Nagle, but for ACKs instead of data.**

---

### Unit 2: The Two Rules of Delayed ACK

TCP implementations typically use **two triggers** to send a delayed ACK:

```
Rule 1: Wait up to 40ms (Linux: 40ms, some: 50-200ms)
        -> If no second packet arrives within the timer, send ACK

Rule 2: Receive 2 segments (packets)
        -> Immediately send ACK for both
```

**The key numbers:**

| Parameter | Typical Value |
|---|---|
| Delay timer | 40ms (Linux) |
| Segment threshold | 2 segments |
| ACK always sent immediately | When window is 0 or PSH flag is set |

---

### Unit 3: Why Does Delayed ACK Exist?

**Same reason as Nagle — reduce overhead.**

```
Without Delayed ACK:
  1 byte data + 40 bytes headers = 41 bytes -> ACK sent immediately
  1 byte data + 40 bytes headers = 41 bytes -> ACK sent immediately
  Total: 82 bytes for 2 bytes of data (2.4% efficiency)

With Delayed ACK:
  2 bytes data + 40 bytes headers = 42 bytes -> ONE ACK sent after delay
  Total: 42 bytes for 2 bytes of data (95% efficiency)
```

**Delayed ACK cuts ACK traffic roughly in half.**

---

### Unit 4: The Interaction with Nagle — THE Critical Part

```
Nagle says:       "Wait for ACK before sending more"
Delayed ACK says: "Wait before sending ACK"

Both waiting -> potential deadlock -> latency spikes
```

**Step-by-step trace:**

```
t=0:   Client sends "H" (1 byte)
       Nagle: No unacked data -> sent immediately (OK)
       Server receives "H"
       Server: Delayed ACK timer starts (40ms)
                |
t=0:   Client sends "e" (1 byte)
       Nagle: "H" is unacknowledged -> BUFFER it (!!)
                |
t=40ms: Server timer fires
       Server sends ACK for "H"
                |
t=40ms: Client receives ACK for "H"
       Nagle: No unacked data -> flush buffer -> send "e" (OK)
                |
t=40ms: Server receives "e"
       Server: Delayed ACK timer starts (40ms)
                |
t=40ms: Client sends "l" (1 byte)
       Nagle: "e" is unacknowledged -> BUFFER (!!)
                |
t=80ms: Server timer fires -> sends ACK
                |
t=80ms: Client flushes buffer -> sends "llo"
```

**Pattern:** Every single byte has to wait for:
1. The Delayed ACK timer (40ms)
2. Then Nagle to flush

**Total extra delay per byte: ~40ms**

For typing "Hello":

```
"H" -> immediate
"e" -> +40ms wait
"l" -> +40ms wait
"l" -> +40ms wait
"o" -> +40ms wait

Total added latency: 160ms for 5 characters
```

---

### Unit 5: Processing vs ACK — The Key Clarification

**The server does NOT wait for the Delayed ACK timer before processing the request.**

These are **two completely separate things:**

```
Application Layer: "I got data, let me process it" -> STARTS PROCESSING (OK)
TCP Layer:         "I got data, I will ACK later"  -> Starts 40ms timer (OK)
```

**The Delayed ACK timer lives in the TCP stack.** It has **nothing to do** with application processing.

**The corrected flow:**

```
t=0:   Client sends GET /api (small packet)
t=0:   Server NIC receives packet
t=0:   TCP stack: stores in receive buffer
t=0:   TCP stack: Delayed ACK timer starts (40ms)
t=0:   Application: "Got GET /api" -> STARTS PROCESSING (OK)
t=0:   Application: "I have the response" -> sends HTTP 200 + body
t=0:   TCP stack: Nagle check -> no unacked data -> SENDS response (OK)
t=0:   Client receives response
t=0:   Client sends ACK for response
t=0:   Server receives ACK -> Nagle happy (OK)
                |
t=40ms: Server Delayed ACK timer for original GET fires
        -> Sends ACK for GET (finally, but who cares?)
```

**The server processed the request and sent the response IMMEDIATELY at t=0.** The Delayed ACK fired 40ms later, but by then the response was already sent and received.

---

### Unit 6: When Delayed ACK ACTUALLY Hurts

**The 40ms problem only happens in a specific pattern:**

```
Pattern: Small request -> Small response -> Small request -> Small response...

Step 1: Client sends small request (1 segment)
Step 2: Server receives it -> Delayed ACK timer starts
Step 3: Server sends small response (1 segment)
Step 4: Client receives response -> Delayed ACK timer starts
Step 5: Client sends next request -> Nagle says "wait for ACK of response"
Step 6: Client is BLOCKED until Delayed ACK timer fires (40ms)
Step 7: ACK sent -> Nagle flushes -> next request sent
```

**The deadlock happens when:**
1. Each side sends small packets
2. Each side has Delayed ACK enabled
3. The sender has unacknowledged data when it wants to send the next thing

---

### When It Does NOT Hurt

**Large data flows fine:**

```
Client sends: GET /large-file.zip (1 large packet, 1460 bytes)
Server receives it -> Delayed ACK timer starts
Server immediately sends response: 200 OK + file data (large)

Because the response is large (multiple segments),
the server Delayed ACK for the GET request
fires while data is flowing -> no deadlock

TCP_NODELAY not needed — large data flows fine
```

**Why it works:** Large data = multiple segments = Delayed ACK timer fires (2 segments received) = no blocking.

---

### Unit 7: TCP_QUICKACK — Directly Disable Delayed ACK

**TCP_QUICKACK = Directly disable Delayed ACK on the receiver.**

```c
// C / Linux
int flag = 1;
setsockopt(socket, IPPROTO_TCP, TCP_QUICKACK, &flag, sizeof(flag));
```

**What it does:** Tells the kernel — "Don't delay ACKs. Send them immediately."

**TCP_NODELAY vs TCP_QUICKACK:**

| Option | Controls | Side | Persistence |
|---|---|---|---|
| **TCP_NODELAY** | Nagle's Algorithm (sender buffers data) | **Sender** | Persistent (OK) |
| **TCP_QUICKACK** | Delayed ACK (receiver delays ACKs) | **Receiver** | Resets on next packet (!!) |

**Important:** TCP_QUICKACK is **not permanent** on Linux. After the next data packet is received, Linux automatically re-enables Delayed ACK. So you need to set it each time.

This is why most implementations focus on TCP_NODELAY — it's simpler and more persistent. But TCP_QUICKACK gives you **direct control** over the receiver side.

---

### Unit 8: The Complete Picture

```
SENDER                                    RECEIVER

Nagle's Algorithm (TCP_NODELAY)          Delayed ACK
Controls: Whether to buffer or send      Controls: When to send ACK
Disable with: TCP_NODELAY                Disable with: TCP_QUICKACK
```

**Both sides can independently control their buffering behavior.**

---

### Unit 9: Full-Stack Decision Framework

```
What does your app send?

Small + Frequent (chat, typing, game input, live
notifications, stock ticks)?
   -> TCP_NODELAY = true (sender)
   -> TCP_QUICKACK = true (receiver)
   -> Accept the header overhead

Large + Infrequent (file upload, API response,
batch operations, database sync)?
   -> Leave defaults (both Nagle + Delayed ACK)
   -> Efficient bandwidth

UNCLEAR -> Start with defaults
           Profile -> then decide
```

---

### Unit 10: Practical Impact for Full-Stack Engineers

| Scenario | What happens | What to do |
|---|---|---|
| **WebSocket chat** | Typing delay from Delayed ACK + Nagle | TCP_NODELAY + TCP_QUICKACK |
| **REST API** | ~40ms added per request in chain | Usually negligible; profile if slow |
| **Database protocol** | Query/response adds small delay | Connection poolers may help |
| **Real-time gaming** | Input lag from ACK delays | TCP_NODELAY essential |
| **HTTP/2, HTTP/3** | Multiplexing reduces impact | Built-in optimization |
| **File download** | Large data flows fine | Default is fine (OK) |

---

### Unit 11: The Debugging Rule

> "Mysterious ~40ms delays on small messages? It is Nagle + Delayed ACK."

```
Symptom:  "My WebSocket chat feels laggy"
Diagnosis: Nagle buffering + Delayed ACK waiting
Fix:       socket.setNoDelay(true) + TCP_QUICKACK on server

Symptom:  "My API responds fast for large payloads but slow for small"
Diagnosis: Nagle + Delayed ACK interaction
Fix:       TCP_NODELAY + TCP_QUICKACK

Symptom:  "My file upload works fine"
Diagnosis: Everything is working as designed
Fix:       Nothing needed (OK)
```

---

### Key Takeaways — Lecture 40

1. **Delayed ACK** = receiver waits before sending ACK (40ms or 2 segments)
2. **Purpose** = reduce ACK overhead (same idea as Nagle, opposite side)
3. **Two triggers**: timer (40ms) OR 2 segments received
4. **Nagle + Delayed ACK interaction** = both sides waiting = ~40ms delay per interaction
5. **Server processes immediately** — Delayed ACK is in TCP stack, not application layer
6. **TCP_QUICKACK** = directly disable Delayed ACK (receiver side)
7. **TCP_NODELAY** = disable Nagle (sender side) — more persistent than TCP_QUICKACK
8. **For real-time apps**: Both options together = maximum low-latency performance
9. **For bulk transfer**: Leave defaults — they work together efficiently

---

#### Lecture 41 — Cost of Connection Establishment

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
