# Transmission Control Protocol (TCP)

<details>
<summary><b>TCP — Section 5</b></summary>

### Lectures

- [ ] 21. What is TCP?
- [ ] 22. TCP Segment
- [ ] 23. Flow Control
- [ ] 24. Congestion Control
- [ ] 25. Slow Start vs Congestion Avoidance
- [ ] 26. NAT
- [ ] 27. TCP Connection States
- [ ] 28. TCP Pros and Cons
- [ ] 29. Sockets, Connections and Kernel Queues
- [ ] 30. TCP Server with Javascript using NodeJS
- [ ] 31. TCP Server with C
- [ ] 32. Capturing TCP Segments with TCPDUMP

---

## What is TCP?

### Definition
**TCP = Transmission Control Protocol**

Layer 4 (Transport Layer) protocol. Encapsulated inside IP packets — the IP header's Protocol field is set to `6` to indicate TCP.

```
┌────────────────────────────────────────┐
│ TCP Segment ( encapsulated in IP )     │
│   Protocol = 6 (TCP)                   │
└────────────────────────────────────────┘
         ↓ embedded in
┌────────────────────────────────────────┐
│ IP Header (Protocol = 6)               │
│   Source IP                              │
│   Destination IP                         │
└────────────────────────────────────────┘
```

---

### 1. Reliability & Control

TCP is a **reliable** protocol — it **controls** the transmission. Unlike UDP (which is unreliable/unbothered), TCP tracks every byte, confirms delivery, retransmits losses, and reorders packets.

```
IP:   "Get this packet to 192.168.1.30 somehow"
      → Doesn't care if it arrives, in order, or at all

TCP:  "Get this byte stream to port 80 on 192.168.1.30 reliably"
      → Tracks every byte, confirms receipt, retransmits losses, reorders
```

**How TCP achieves reliability — each mechanism controls a specific failure mode:**

| Failure Mode | TCP's Control Mechanism |
|---|---|
| Data might be lost | **Retransmission** — sender resends unacknowledged data |
| Data might arrive out of order | **Sequence numbers** — receiver reorders based on seq# |
| Data might be corrupted | **Checksum** — receiver validates integrity |
| Receiver might be overwhelmed | **Flow control** — sender slows down |
| Network might be congested | **Congestion control** — sender reduces rate |
| Connection might drop | **Keep-alive / timeout** — detect and recover |

**The Cost of Reliability:** Every control feature adds **time** (waiting for ACKs, retransmitting), **memory** (buffering data, storing state), and **CPU** (calculating checksums, managing sequences). This is why TCP is slower than UDP.

**Practical Impact for Full-Stack Engineers:** When you use HTTP (web browsing, REST APIs), you're using TCP underneath. You never have to resend lost packets, reorder data, or check integrity — TCP handles all of that. Your application just reads a clean, ordered byte stream.

---

### 2. Layer 4 — Transport Layer

TCP sits between your application and IP. Your app says "send this data," TCP says "I'll make sure it gets there properly," then hands it to IP for delivery.

```
┌──────────────────────────────────────────┐
│ Layer 7: Application                       │
│   HTTP, DNS, SSH, FTP, gRPC              │
│   YOUR CODE LIVES HERE                   │
├──────────────────────────────────────────┤
│ Layer 4: Transport  ← TCP / UDP         │
│   TCP (reliable) / UDP (fast)            │
│   Adds port numbers + reliability/speed  │
├──────────────────────────────────────────┤
│ Layer 3: Network     ← IP               │
│   Gets packets to the right HOST         │
├──────────────────────────────────────────┤
│ Layer 2: Data Link     ← Ethernet       │
│   Gets frames on the local WIRE          │
└──────────────────────────────────────────┘
```

**The key boundary:** Layer 3 (IP) gets data to the right **machine**. Layer 4 (TCP/UDP) gets data to the right **application** on that machine.

```
IP:  "Deliver to 10.0.2.10"     → That machine
TCP: "Deliver to port 5432 on 10.0.2.10"  → That specific app (PostgreSQL)
```

**Where TCP sits in your stack:**

```
Your App Code (Node.js, Python, Java...)
    ↓ calls
TCP Library (built into OS)
    ↓ hands to
IP Layer (OS kernel)
    ↓ hands to
Network Hardware
```

Understanding TCP helps you debug: connection timeouts, low throughput, slow requests (handshake overhead), running out of connections.

---

### 3. Process Addressing via Ports

TCP uses **port numbers** to address specific applications on a host (same as UDP). TCP uses a **4-tuple** to identify connections: `(src IP, src port, dst IP, dst port)`

```
Your laptop (192.168.1.20) opens browser:
  → Browser uses ephemeral port 52341 (random)
  → Sends request to 10.0.2.10:8080 (backend API)

The backend server (10.0.2.10) receives:
  → Source: 192.168.1.20:52341
  → Destination: 10.0.2.10:8080

The database server (10.0.2.20) receives from backend:
  → Source: 10.0.2.10:49152 (ephemeral), Destination: 10.0.2.20:5432
```

This means your laptop can have multiple simultaneous connections:
- Connection 1: (192.168.1.20:52341, 10.0.2.10:8080) → API call
- Connection 2: (192.168.1.20:52342, 10.0.2.10:8080) → another API call
- Connection 3: (192.168.1.20:52343, 10.0.2.20:5432) → database query

**Port Ranges:**

| Range | Name | Example |
|-------|------|---------|
| 0-1023 | Well-known | 80 (HTTP), 443 (HTTPS), 22 (SSH), 53 (DNS) |
| 1024-49151 | Registered | 3000 (Node.js dev), 5432 (PostgreSQL) |
| 49152-65535 | Ephemeral | 52341 (random client port) |

---

### 4. Stateful Connection

TCP is **stateful** — both client and server store connection state.

**State stored on both sides:**
- Sequence numbers (where we are in the data stream)
- Acknowledgment numbers (what we've received)
- Window size (how much data we can receive)
- Connection status (established, closing, etc.)
- Timers (how long to wait for ACKs)
- Congestion window (how fast we can send)

**Connection = Knowledge between Client and Server:**

```
UDP:  Client → Data → Server (no prior knowledge)
TCP:  Client → SYN → Server
      Server → SYN-ACK → Client
      Client → ACK → Server
      Connection established → data flows
```

**Why 3 steps and not 2?** Both sides need to: (1) say they want to connect, (2) confirm the other side wants to connect, (3) confirm the confirmation. With 2 steps, one side might not know the other received their agreement. The 3rd step guarantees **both sides know** the connection is live.

**Why Stateful = Expensive:**

Every TCP connection uses resources:
```
Per connection on the server:
  ├── Memory: ~4KB-64KB for buffers, state, queues
  ├── File descriptor: Each connection = one fd
  ├── CPU: Processing seq#, ACKs, timers
  └── Port: Uses an ephemeral port on the server side
```

**Scale math:**
```
10,000 concurrent connections × 16KB each = 160 MB just for connection state
100,000 connections = 1.6 GB
1,000,000 connections = 16 GB (before any data flows!)
```

This is why:
- Web servers have `max_connections` limits (Nginx default: 1024, tunable)
- Load balancers track connection state and can become bottlenecks
- **Connection pooling** is critical — reuse existing connections instead of creating new ones
- **HTTP/2 and HTTP/3** were designed to reduce connection overhead

**3-Way Handshake Latency:**
```
Before handshake: 0 ms (no connection)
After handshake: 1 RTT delay before first byte

If RTT = 50ms (typical):
  Connection takes 50ms before any data flows

If you need 100 connections (no pooling):
  100 × 50ms = 5 seconds of pure handshake overhead

With connection pooling (reuse 10 connections):
  10 × 50ms = 500ms, then reuse for all 100 requests
```

---

### 5. Header Size: 20 Bytes Minimum, Up to 60 Bytes

The TCP header has fixed fields (20 bytes) plus an optional field (0-40 bytes).

**Fixed fields = 20 bytes:**

| Field | Size | Purpose |
|-------|------|---------|
| Source Port | 2 bytes | Which app sent this |
| Destination Port | 2 bytes | Which app receives this |
| Sequence Number | 4 bytes | Where in the stream this data is |
| Acknowledgment Number | 4 bytes | What byte I expect next |
| Data Offset + Flags | 1 byte | Header length + control flags (SYN, ACK, FIN, etc.) |
| Window Size | 2 bytes | How much data I can receive |
| Checksum | 2 bytes | Error detection |
| Urgent Pointer | 2 bytes | Marks urgent data (rarely used) |

**Options field = 0-40 bytes:**

| Option | Size | Purpose |
|--------|------|---------|
| Window Scaling | ~3 bytes | Allows window > 65KB (high-speed networks) |
| Timestamps | ~10 bytes | Better RTT measurement, PAWS protection |
| SACK Permitted | ~2 bytes | Allows selective retransmission |
| MSS (Max Segment Size) | ~2 bytes | Largest payload per segment |

**Why Header Size Matters:** Every byte of header is overhead that doesn't carry your data.

```
TCP segment with 1 byte of data:
  20 bytes TCP header + 20 bytes IP header + 1 byte data = 41 bytes sent
  Efficiency: 1/41 = 2.4%

TCP segment with 1460 bytes of data (typical MTU):
  20 bytes TCP + 20 bytes IP + 1460 bytes data = 1500 bytes
  Efficiency: 1460/1500 = 97.3%
```

**This is why batching matters.** Sending many small messages is incredibly wasteful. Your HTTP server should batch responses, your database should use bulk inserts, your API should accept batch requests.

**Header comparison:**

```
UDP Header (8 bytes):
┌────────────┬────────────┬────────┬─────────┐
│Src Port    │Dst Port    │Length  │Checksum │
│16 bits     │16 bits     │16 bits │16 bits  │
└────────────┴────────────┴────────┴─────────┘

TCP Header (20-60 bytes):
┌──────────┬──────────┬────────────┬────────────┬────┬─────┬──────┬──────┐
│Src Port  │Dest Port │Seq Number  │ACK Number  │Flags│Window│Checksum│Urgent│
│16 bits   │16 bits   │32 bits     │32 bits     │4+4b│16bt│16 bits │16 bt │
└──────────┴──────────┴────────────┴────────────┴────┴─────┴────────┴──────┘
                    + Options (0-40 bytes)
```

TCP's header is **2.5x to 7.5x larger** than UDP's — the cost of reliability.

---

### Summary: The 6 Points Connected

```
TCP = Transmission Control Protocol
  │
  ├── "Control" = Reliability mechanisms
  │     ├── Retransmission (lost data → resend)
  │     ├── Sequencing (out of order → reorder)
  │     ├── Checksum (corrupted → drop)
  │     ├── Flow control (overwhelmed → slow down)
  │     └── Congestion control (network busy → reduce rate)
  │
  ├── Layer 4 = Between your app and IP
  │     ├── IP gets data to the right HOST
  │     └── TCP gets data to the right APPLICATION (via ports)
  │
  ├── Stateful = Both sides remember the connection
  │     ├── Requires 3-way handshake to establish
  │     ├── Costs memory per connection
  │     ├── Connection pooling is essential
  │     └── HTTP/2, HTTP/3 reduce connection overhead
  │
  └── Header = 20-60 bytes overhead
        ├── Fixed fields: ports, seq#, ACK, flags, window, checksum
        ├── Options: scaling, timestamps, SACK
        └── Batching reduces overhead waste
```

---

### TCP Use Cases

#### 1. Reliable Communication (e.g., Chat Application)
Every message must arrive. TCP guarantees no message loss.

```
You: "Hey, are you there?"
    ↓ TCP: seq=100, ACK mechanism
Server: "Yes, I'm here"
    ↓ TCP: seq=200, ACK mechanism
```

**Modern apps use a hybrid approach:**
- **Text messages:** TCP (WebSocket over TCP) — every message must arrive
- **Typing indicators:** Often UDP — disappearing is fine
- **Voice/video calls:** UDP — latency matters more than perfection
- **File transfers:** TCP — every byte must be exact

**Full-Stack insight:** Socket.IO uses WebSocket (TCP) with HTTP long-polling fallback.

---

#### 2. Remote Shell — SSH
Command-line access over networks. Every character must arrive correctly.

```
Your laptop ──── SSH (TCP, Port 22) ──── Server
```

If TCP didn't guarantee delivery:
```
You type: "rm -rf /important/data"
If "i" is lost: "rm -rf /mportant/data" ← disaster
```

| Protocol | Transport | Why |
|----------|-----------|-----|
| SSH | TCP | Reliable command execution |
| RDP (Windows) | TCP | Reliable screen + input |
| VNC | TCP | Reliable frame buffer sync |
| Telnet | TCP | Legacy, unencrypted |

---

#### 3. Database Connection — PostgreSQL, MySQL
Reliable queries and transaction streaming.

```
Your App ──── TCP ──── PostgreSQL (Port 5432)
```

Why TCP for databases:
- Queries must execute correctly — lost query = missing data
- Transactions must be atomic — partial = corruption
- Result sets must be complete — missing rows = wrong answers

```
BEGIN TRANSACTION;
  UPDATE accounts SET balance = balance - 100 WHERE id = 1;
  UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;
```

If the second UPDATE is lost → $100 disappears. TCP prevents this.

**Connection pooling** (PgBouncer, ProxySQL) manages TCP connections so your app doesn't create a new handshake for every query.

---

#### 4. Web Communication — HTTP over TCP
All web traffic (HTML, CSS, JS, API calls) rides on TCP.

```
Browser ──── HTTP ──── TCP ──── IP ──── Server
```

Why HTTP needs TCP:
- HTML must arrive completely — missing `<script>` = broken page
- CSS must be complete — missing rules = broken layout
- API responses must be reliable — missing JSON = broken app

**The handshake cost:**
```
First request to new server:
  1. TCP 3-way handshake (1 RTT)
  2. TLS handshake if HTTPS (1-2 RTT)
  3. HTTP request → response (1 RTT)
  Total: 3-4 RTT before first byte

  At 50ms RTT: 150-200ms before any content
```

This is why:
- **HTTP/1.1 Keep-Alive:** Reuse TCP connection for multiple requests
- **HTTP/2 Multiplexing:** Multiple requests over one TCP connection
- **HTTP/3 QUIC:** Built on UDP to avoid head-of-line blocking

---

#### 5. HTTP/3 — Built on QUIC which is Built on UDP

```
HTTP/1.1 and HTTP/2:
  HTTP → TCP → IP → Network

HTTP/3:
  HTTP → QUIC → UDP → IP → Network
```

**Why HTTP/3 left TCP — Head-of-Line Blocking:**

```
TCP:  Packet 1 lost → ALL subsequent packets wait
      Even though packets 2, 3, 4 arrived fine
      They're stuck waiting for packet 1

HTTP/2 over TCP:
  Request A (stream 1) → packet 1 LOST
  Request B (stream 2) → packet 1 arrived fine
  Request B is BLOCKED because TCP waits for stream 1
```

**QUIC solves this:**

```
QUIC over UDP:
  Request A (stream 1) → packet 1 LOST
  Request B (stream 2) → packet 1 arrived → processed immediately
  Only stream 1 waits, stream 2 flows freely
```

**Trade-off:**
```
TCP:  Reliable but head-of-line blocking
QUIC: Reliable, independent streams, no blocking
      Includes TLS encryption by default
      Faster handshakes (0-RTT in some cases)
```

---

#### 6. Bidirectional Communication — Full-Duplex
TCP supports full-duplex natively — both sides send and receive simultaneously.

```
Client ←→ Server
  ↑          ↑
  Sends      Sends
  Receives   Receives
```

**Examples of bidirectional TCP communication:**

| Application | How it uses bidirectional TCP |
|-------------|-------------------------------|
| WebSocket | Persistent connection, both sides push anytime |
| SSH | You type commands, server sends output — simultaneously |
| Database | App sends queries, DB sends results — ongoing |
| Chat | Either side can message anytime |

**Full-Stack insight:** WebSockets (Socket.IO, `ws` library) use TCP's full-duplex. The connection stays open, either side sends data at any time. Foundation of real-time notifications, live dashboards, collaborative editing, multiplayer games.

---

#### Pattern Across All Use Cases

Every TCP use case shares ONE trait: **the data MUST arrive reliably and in order.**

| Use Case | Why TCP? |
|----------|----------|
| Chat | Messages can't be lost |
| SSH | Commands can't be corrupted |
| Database | Transactions can't be partial |
| Web (HTTP/1.1/2) | Pages can't be incomplete |
| Bidirectional | Both sides need reliable, simultaneous flow |

**The exception:** HTTP/3 moved to QUIC/UDP for performance (head-of-line blocking). Still uses reliable delivery — just implements its own on top of UDP.

---

### TCP Connection

A TCP connection is a **logical, stateful relationship** between two applications on different hosts. It's not a physical wire — it's a **contract** both sides agree to follow.

**Key facts:**
- Between **processes**, not hosts (a host can have many connections)
- Identified by a **4-tuple**: `(src IP, src port, dst IP, dst port)`
- **Stateful** — both sides store information
- Has a **lifecycle**: born (handshake), lives (data transfer), dies (teardown)

**4-Tuple — How Connections Are Identified:**

```
Connection 1: (192.168.1.20:52341, 10.0.2.10:80)    → Browser to API
Connection 2: (192.168.1.20:52342, 10.0.2.10:80)    → Another tab to same API
Connection 3: (192.168.1.20:52343, 10.0.2.10:443)   → Browser to HTTPS
Connection 4: (192.168.1.20:52344, 10.0.2.20:5432)  → App to database
```

Without source port, the server couldn't tell which app each response belongs to.

---

#### Connection Establishment — The 3-Way Handshake

The connection is **born** here. Both sides must agree before any data flows.

```
Client                    Server
  │                          │
  │──── SYN (Seq=X) ──────→│
  │     Flags: [SYN=1, ACK=0]  "I want to connect, my seq# is X"
  │                          │
  │←─── SYN-ACK (Seq=Y, Ack=X+1) ─│
  │     Flags: [SYN=1, ACK=1]  "I accept, my seq# is Y"
  │                          │
  │──── ACK (Seq=X+1, Ack=Y+1) →│
  │     Flags: [ACK=1]         "Confirmed, let's start"
  │                          │
  │    ← Connection ESTABLISHED →
```

**Why 3 steps and not 2?** With only 2 steps, if the SYN-ACK is lost, the client never knows the server agreed. The server holds resources for a ghost connection. The 3rd step (ACK) guarantees **both sides know** the connection is live.

**Sequence numbers are random** for security — predictable seq numbers enable TCP sequence prediction attacks.

---

#### Connection State — What Both Sides Store

**Server's state for (Client:52341, Server:80):**

```
  ├── Client IP, Client Port, Server Port
  ├── Client Seq# (last received)
  ├── Server Seq# (current)
  ├── Window Size (how much client can receive)
  ├── Connection State: ESTABLISHED
  ├── Send Buffer / Receive Buffer
  ├── Retransmission Timer
  ├── Keep-Alive Timer
  └── Congestion Window
```

**Memory cost per connection:** ~4KB-64KB

```
1,000 connections    =  16 MB
10,000 connections   = 160 MB
100,000 connections  = 1.6 GB
1,000,000 connections = 16 GB (before any data flows!)
```

This is why servers have `max_connections` limits and connection pooling is critical.

---

#### Connection Teardown — The 4-Way Handshake

Both sides must agree to close (TCP is full-duplex — each direction closes independently):

```
Client → FIN → Server     "I'm done sending"
Client ← ACK ← Server     "Got it"
Client ← FIN ← Server     "I'm done too"
Client → ACK → Server     "Goodbye"
```

---

#### TIME_WAIT State

After the client sends the final ACK, it waits for **2×MSL** (Maximum Segment Lifetime, typically 30-120 seconds).

**Why wait?**
1. **Ensure the final ACK arrives** — if lost, server retransmits FIN, client must be there to respond
2. **Prevent old duplicate packets** from confusing new connections using the same 4-tuple

**Practical impact:** Opening/closing many connections rapidly can exhaust ports. Solution: connection pooling or `SO_REUSEADDR`.

---

#### Half-Open Connection (Zombie)

One side crashes, the other doesn't know — server keeps sending data to a dead client, wasting resources.

**Detection — Keep-Alive:** After silence period, server sends probe. No response → connection closed.

---

#### Connection Lifecycle

```
CLOSED → SYN_SENT → SYN_RCVD → ESTABLISHED → FIN_WAIT_1 → FIN_WAIT_2
                                                          → CLOSE_WAIT
ESTABLISHED → CLOSING → TIME_WAIT → CLOSED
ESTABLISHED → LAST_ACK → CLOSED
```

**Most common states:**

| State | When | What's happening |
|-------|------|-----------------|
| **ESTABLISHED** | Data flowing | Both sides actively communicating |
| **TIME_WAIT** | After client closes | Waiting 2×MSL before fully closing |
| **CLOSE_WAIT** | Server received FIN | Server waiting app to close too |
| **FIN_WAIT_1** | Client sent FIN | Client waiting ACK or server FIN |
| **CLOSED** | Final state | Connection fully gone |

---

#### Practical Impact for Full-Stack Engineers

| Concept | What you experience |
|---------|---------------------|
| 3-way handshake | First request slow (1 RTT overhead) |
| Connection limits | Server has `max_connections` (Nginx: 1024 default) |
| TIME_WAIT | Can exhaust ports with many short-lived connections |
| Keep-alive | HTTP/1.1 default, reuses connections |
| Half-open | Zombie connections waste server memory |
| Full-duplex | WebSocket works — both sides can send anytime |

---

### What's Next?
Lecture 22: **TCP Segment** — the detailed structure of a TCP segment.

---

## TCP Segment

### Definition
A **TCP Segment** is the unit of data that TCP sends over the network. It's the TCP layer's version of a "packet."

**Precise formula:**

```
TCP Segment = TCP Header (20-60 bytes) + Application Data (serialized payload)
```

The **Application Data** is whatever your application produced after serialization at Layer 6/7 — HTTP request, JSON body, raw bytes from `socket.write()`, database query, binary file chunk, etc. **TCP does not interpret this data in any way.** It only numbers the bytes, checks their integrity, and delivers them in order.

**Terminology hierarchy:**

```
Application:    Message / Data / Byte stream
    ↓ serialization (encoding, formatting)
Layer 6/7:      Serialized bytes (payload)
TCP Layer:      Segment = Header (20-60 bytes) + Serialized bytes
IP Layer:       Packet = IP Header (20-60 bytes) + Segment
Layer 2:        Frame = Ethernet Header (14B) + Packet + FCS (4B)
Layer 1:        Bits on the wire
```

Each layer wraps the layer above with its own header:

```
┌─────────────────────────────────────────────────────┐
│ Application Data (your serialized message)          │
├─────────────────────────────────────────────────────┤
│ TCP Header (20-60 bytes) + Data = TCP Segment     │  Layer 4
├─────────────────────────────────────────────────────┤
│ IP Header (20-60 bytes) + Segment = IP Packet     │  Layer 3
├─────────────────────────────────────────────────────┤
│ Ethernet Header + IP Packet + FCS = Frame         │  Layer 2
├─────────────────────────────────────────────────────┤
│ Bits on the wire                                    │  Layer 1
└─────────────────────────────────────────────────────┘
```

**Key concept:** TCP is a **byte stream** protocol — it does NOT preserve message boundaries.

```
App: send("Hello")
App: send("World")

TCP sees: a single byte stream "HelloWorld"
TCP might deliver:
  "HelloWor"  ← segment 1 (Seq=0, 9 bytes)
  "ld"        ← segment 2 (Seq=9, 2 bytes)

Or any other split. Your app must handle framing.
```

**Why this matters:** TCP has no idea that "Hello" and "World" were two separate `send()` calls. It only knows byte offsets (Seq=0, Seq=9). If your protocol relies on message boundaries (e.g., one `read()` = one message), **you must add framing yourself** — length prefix, delimiter, or fixed-size messages.

---

### What is the Payload? — Real-World Examples

The payload inside a TCP Segment depends entirely on what the application is doing:

| App Activity | Serialized Payload Inside Segment |
|---|---|
| HTTP request | `GET /api HTTP/1.1\r\nHost:...\r\n\r\n` |
| WebSocket message | `{"type":"chat","text":"hi"}` |
| Database query | `SELECT * FROM users WHERE id=1` |
| File transfer | Raw bytes of the file chunk |
| Custom binary protocol | Your own binary format |

**TCP doesn't care what the payload is.** It treats everything as a byte stream. A segment carrying an HTTP request and a segment carrying a JPEG image are processed identically by TCP — same Seq#, same ACK, same checksum, same window logic.

This is both TCP's power and its limitation: TCP is **content-agnostic**. It provides reliable, ordered delivery regardless of what's inside, but it can't optimize for specific content types.


---

### TCP Segment Structure

```
┌─────────────────────────────────────────────────────────┐
│                    TCP Segment                            │
│                                                               │
│  ┌─────────────────────────────────────┐                │
│  │         TCP Header (20-60 bytes)      │                │
│  │  SrcPort, DstPort, Seq#, ACK#,      │                │
│  │  Flags, Window, Checksum, Options    │                │
│  └─────────────────────────────────────┘                │
│  ┌─────────────────────────────────────┐                │
│  │         Application Data              │                │
│  │    Serialized by application          │                │
│  │    (HTTP, JSON, binary, query...)     │                │
│  │    TCP treats as raw byte stream      │                │
│  └─────────────────────────────────────┘                │
└─────────────────────────────────────────────────────────┘
```

**The data inside is NOT "raw" — it's been through the application's serialization process.** TCP receives it as a byte stream and doesn't interpret it.

---

### Data Offset — How Header Size is Communicated

Since the Options field is variable-length, the receiver needs to know exactly where the header ends and data begins. The **Data Offset** field (4 bits) answers this:

```
Data Offset = Header Size ÷ 4

Data Offset = 5  → Header = 5 × 4 = 20 bytes (no options — minimum)
Data Offset = 6  → Header = 6 × 4 = 24 bytes (e.g., MSS option)
Data Offset = 15 → Header = 15 × 4 = 60 bytes (maximum)
```

**Why in 4-byte units?** 4 bits stores values 0-15, and 15 × 4 = 60 bytes — covers the maximum possible header. This encoding is compact and covers all cases.

---

### Header Fields — Deep Dive

#### Source Port (2 bytes)
Port number of the **sending** application. Receiver uses it to know where to send replies.

```
Client app on port 52341 sends:
  Source Port = 52341
```

#### Destination Port (2 bytes)
Port number of the **receiving** application.

```
Sending to web server:
  Destination Port = 80 (or 443 for HTTPS)
```

Together with Source Port, these identify the connection (part of the 4-tuple).

#### Sequence Number (4 bytes)
The **most important field for reliability**. Identifies the position of the first byte of data in this segment within the overall byte stream.

```
Segment 1: Seq = 0    → Bytes 0-499 (500 bytes)
Segment 2: Seq = 500  → Bytes 500-999
Segment 3: Seq = 1000 → Bytes 1000-1499
```

**Used for:** Reordering out-of-order segments, detecting missing segments, knowing what to retransmit.

**ISN (Initial Sequence Number):** Randomly chosen at connection start (prevents prediction attacks).

#### Acknowledgment Number (4 bytes)
Tells the sender: "I have received all bytes up to this number, send me the next one."

```
Server received bytes 0-499:
  ACK = 500 → "I have everything up to 500, send byte 500 next"
```

**Cumulative ACK:** One ACK can acknowledge many segments (everything before the number).

#### Data Offset / Header Length (4 bits)
Tells receiver where the header ends and data begins. Since Options field is variable-length.

```
Data Offset = 5  → Header = 5 × 4 = 20 bytes (no options)
Data Offset = 15 → Header = 15 × 4 = 60 bytes (maximum)
```

**Why in 4-byte units?** 4 bits stores 0-15, and 15 × 4 = 60 bytes — covers the maximum header.

#### Flags (6 main control bits)

| Flag | Full Name | Purpose | When used |
|------|-----------|---------|-----------|
| **URG** | Urgent | "This segment has urgent data" | Urgent data must skip queue |
| **ACK** | Acknowledgment | "The ACK number is valid" | Almost always (after handshake) |
| **PSH** | Push | "Deliver to app immediately" | App needs data now |
| **RST** | Reset | "Abort the connection" | Error, invalid connection |
| **SYN** | Synchronize | "Let's establish a connection" | Connection initiation |
| **FIN** | Finish | "I'm done sending, close" | Connection teardown |

**Flag combinations:**

```
[SYN=1, ACK=0]  → Connection request
[SYN=1, ACK=1]  → Connection accepted
[SYN=0, ACK=1]  → Regular data acknowledgment
[FIN=1, ACK=1]  → Closing connection
[RST=1]           → Connection reset/abort
```

**PSH flag — Push:** Without PSH, TCP may buffer small writes and batch them. With PSH, data is pushed to the receiving application immediately.

**RST flag — Connection Killer:**

```
RST = 1 means: "Something is wrong, abort immediately"

Common causes:
  → Connecting to a port with no app listening
  → Corrupted segment that can't be recovered
  → Security firewall rejecting connection
  → App crashed on server side
```

#### Window Size (2 bytes)
**Flow control** — tells sender how many bytes the receiver is willing to accept.

```
Server says: Window = 65535
Meaning: "I can receive 65,535 bytes before I need you to pause"

When buffer fills: Window = 0 → "Stop sending"
```

**Window Scaling (option):** 2-byte window maxes at 65,535. With scaling, up to **1 GB**.

```
Window = 65535 × Scale Factor (e.g., 128) = ~16 MB
```

#### Checksum (2 bytes)
Error detection for TCP header + data + pseudo-header (IP source/dest + protocol + TCP length).

```
Sender: Calculates checksum → stores in segment
Receiver: Recalculates → compares
  Match → process it
  Mismatch → drop it (no ACK → sender retransmits)
```

#### Urgent Pointer (2 bytes)
Used with URG flag. Points to the byte **after** the last urgent byte. Rarely used in modern systems.

#### Options (0-40 bytes)
Variable-length extensions:

| Option | Size | Purpose |
|--------|------|---------|
| **MSS** | ~2 bytes | Largest payload per segment (usually 1460) |
| **Window Scaling** | ~3 bytes | Multiply window beyond 65535 |
| **Timestamps** | ~10 bytes | RTT measurement, PAWS protection |
| **SACK Permitted** | ~2 bytes | "I accept selective retransmission" |
| **NOP** | 1 byte | No-operation (padding) |
| **EOL** | 1 byte | End-of-options-list (padding) |

**MSS — Max Segment Size:**

```
MTU = 1500 (Ethernet)
IP Header = 20 bytes
TCP Header = 20 bytes
MSS = MTU - IP Header - TCP Header = 1460 bytes
```

---

### How Segments Are Formed

When app calls `send()`:

```
App: send("Hello World, this is a test message")  (50 bytes)
  ↓
TCP checks:
  ├── Is there enough data? (depends on MSS = 1460)
  ├── Is receiver's window big enough?
  ├── Should I batch more data or send now?
  └── Should I set PSH flag?

If data ≤ MSS:
  → Create segment: Header + data
  → Seq = current sequence number
  → Send it

If data > MSS:
  → Split into multiple segments:
    Segment 1: Seq = 0,     1460 bytes
    Segment 2: Seq = 1460,  remaining bytes
```

**Nagle's Algorithm:** Buffers small writes and sends them together to reduce overhead (covered in IP section).

---

### How Segments Are Received — Reassembly

Segments arrive potentially out of order:

```
Segment 3: Seq = 2920  → buffered (missing 1460-2919)
Segment 1: Seq = 0     → delivered! → ACK = 1460
Segment 2: Seq = 1460  → delivered! → ACK = 2920
Segment 3: Seq = 2920  → now delivered → ACK = 4380
```

**Out-of-order handling:**

```
Expected: Seq = 1460
Received: Seq = 2920 (out of order!)

Action:
  → Buffer segment 2920
  → Send ACK = 1460 (still waiting for 1460)
  → When Seq = 1460 arrives → deliver both buffered segments
```

**Duplicate ACKs → Fast Retransmit:** After 3 duplicate ACKs, sender retransmits without waiting for a timeout.

---

### Segment Sizing — MSS, MTU, Relationship

```
Physical link limit:  MTU (e.g., 1500 for Ethernet)
IP overhead:          IP Header (20 bytes minimum)
TCP overhead:         TCP Header (20 bytes minimum)
Available for data:   MSS = MTU - IP Header - TCP Header = 1460 bytes
```

**If data exceeds MSS:**

```
App sends 5000 bytes:
  Segment 1: 1460 bytes data → 1500 bytes total (with headers)
  Segment 2: 1460 bytes data → 1500 bytes total
  Segment 3: 1460 bytes data → 1500 bytes total
  Segment 4: 540 bytes data  → 580 bytes total
```

**If IP packet exceeds MTU:**

```
DF flag NOT set → IP fragments the packet → receiver reassembles
DF flag SET → IP drops packet → ICMP "Fragmentation Needed" → sender reduces size
```

This is why **Path MTU Discovery** exists.

---

### Segment vs Packet vs Frame

| Term | Layer | What it is |
|------|-------|------------|
| **Segment** | Layer 4 (TCP) | TCP Header + Data |
| **Packet** | Layer 3 (IP) | IP Header + Segment |
| **Datagram** | Layer 3 (IP/UDP) | Header + Data (connectionless) |
| **Frame** | Layer 2 (Ethernet) | Ethernet Header + Packet + FCS |

Same data, different names at different layers.

```
┌────────────────────────────────────────────────────────┐
│ Layer 4: TCP Segment                                    │
│   TCP Header + TCP Data                                 │
├────────────────────────────────────────────────────────┤
│ Layer 3: IP Packet                                      │
│   IP Header + TCP Segment                               │
├────────────────────────────────────────────────────────┤
│ Layer 2: Ethernet Frame                                 │
│   Ethernet Header + IP Packet + FCS                     │
├────────────────────────────────────────────────────────┤
│ Layer 1: Bits                                             │
│   010110101101010101...                                 │
└────────────────────────────────────────────────────────┘
```

---

### Practical Impact for Full-Stack Engineers

| Concept | What you experience |
|---------|---------------------|
| **Byte stream** | TCP doesn't preserve message boundaries — you need framing |
| **MSS / MTU** | Large messages split into segments automatically |
| **Out-of-order** | TCP handles reordering — you get data in order |
| **Segmentation** | `send(5000 bytes)` might result in 4 segments — invisible to you |
| **PSH flag** | Affects latency — small messages might be delayed without it |
| **Checksum** | Corrupted segments silently dropped and retransmitted |
| **Window size** | Controls throughput — small window = slow transfer |

**Message framing in Node.js:**

```javascript
// TCP doesn't know where your message ends — you MUST add framing

// Length-prefixed framing:
const msg = Buffer.from("Hello World");
const packet = Buffer.alloc(4 + msg.length);
packet.writeUInt32BE(msg.length, 0);  // First 4 bytes = length
msg.copy(packet, 4);                   // Then the data
socket.write(packet);

// On receiver:
socket.on('data', (chunk) => {
  // Read length from first 4 bytes
  // Wait for full message
  // Process it
});
```

---

### Summary

```
TCP Segment = TCP Header (20-60 bytes) + TCP Data (up to MSS bytes)

Header tells receiver:
  → Where is this in the stream? (Seq #)
  → What have you received? (ACK #)
  → How much can you take? (Window)
  → What do I need from you? (Flags)
  → Is this valid? (Checksum)
  → Are there extensions? (Options)

Data is your application's byte stream — raw, unframed, ordered.
```

---

### What's Next?
Lecture 23: **Flow Control**

---

## Flow Control

### Definition
Flow control prevents the **sender** from overwhelming the **receiver**. It is **receiver-driven**.

### Mechanism — Window Size
The receiver advertises available buffer space via the **Window Size** field in the TCP header. The sender can send at most that many bytes before pausing.

```
Receiver buffer:
  Total: 65,535 bytes
  Used:  30,000 bytes
  Available: 35,535 bytes

  → Receiver says: Window = 35,535
  → Sender can send up to 35,535 bytes
```

### Three States

| State | Window | Meaning |
|---|---|---|
| **Normal** | > 0 | "Send me this much data" |
| **Full** | 0 | "STOP. My buffer is full." |
| **Recovering** | Increasing | "I'm processing data, come back" |

### How It Works Dynamically

```
1. Receiver buffer has space → Window = N → Sender sends
2. Receiver buffer fills up  → Window = 0 → Sender STOPS
3. Sender sends probe         → Receiver checks buffer
4. Buffer has space           → Window = M → Sender resumes
5. Cycle continues...
```

### Flow Control vs Congestion Control

| | Flow Control | Congestion Control |
|---|---|---|
| **Protects** | Receiver | Network |
| **Driven by** | Receiver's buffer | Network conditions |
| **Mechanism** | Window Size field | cwnd (congestion window) |
| **Algorithm** | Simple advertise/don't | AIMD (Additive Increase, Multiplicative Decrease) |

### Practical Impact

| Scenario | What Flow Control Does |
|---|---|
| **Slow database** | TCP automatically reduces send rate from your app |
| **Buffer overflow prevented** | Without Window = 0, receiver buffer would overflow |
| **Backpressure** | Your `socket.write()` may block if receiver can't keep up |
| **WebSocket streaming** | Window controls how much data can be in-flight |

### Summary

> **Flow control prevents the sender from drowning the receiver — the receiver sets the pace using the Window Size field.**

---

## Congestion Control

### Definition
Congestion control prevents the **sender** from overwhelming the **network**. It is **network-driven**.

### The Core Signal: Packet Loss = Congestion

```
If a packet is lost → the network was congested → slow down
```

Two ways TCP detects loss:
1. **Timeout** — no ACK received within retransmission timeout (RTO)
2. **3 Duplicate ACKs** — receiver keeps saying "I'm still waiting for Seq X"

### Two Windows

```
Effective Window = min(cwnd, rwnd)

cwnd = Congestion Window (network-driven, TCP algorithm)
rwnd = Receiver Window (receiver-driven, buffer space)
```

| Window | Controlled By | Purpose |
|---|---|---|
| **cwnd** | Sender (TCP algorithm) | Don't overwhelm the NETWORK |
| **rwnd** | Receiver (buffer space) | Don't overwhelm the RECEIVER |

### The Algorithm — Four Phases

**Phase 1: Slow Start**
```
cwnd starts at 1 (or 10 on modern Linux)
Every ACK → cwnd += 1
Result: cwnd doubles every RTT (exponential growth)
Purpose: Find available bandwidth quickly and conservatively
```

**Phase 2: Congestion Avoidance**
```
When cwnd ≥ ssthresh:
  Every RTT → cwnd += 1 (linear growth)
Purpose: Probe gently — avoid overwhelming the network
```

**Phase 3: Multiplicative Decrease (Loss Detected)**
```
On loss:
  ssthresh = cwnd / 2
  cwnd = 1 (timeout) or cwnd = ssthresh + 3 (3 dup ACKs)
  → Return to Slow Start or Fast Recovery
```

**Phase 4: Fast Retransmit & Fast Recovery**
```
3 Duplicate ACKs:
  1. Fast Retransmit: Immediately resend missing segment (no timeout wait)
  2. Fast Recovery: ssthresh = cwnd/2, cwnd = ssthresh + 3, skip Slow Start

Timeout (more severe):
  → Full Slow Start from cwnd = 1
```

### AIMD — Additive Increase, Multiplicative Decrease

```
Additive Increase:  cwnd += 1 per RTT (grow slowly when things are good)
Multiplicative Decrease: cwnd = cwnd/2 (cut fast when problems occur)

This creates the sawtooth pattern that keeps the network stable.

Without AIMD: Everyone sends max → routers overflow → collapse → repeat
With AIMD: Gentle growth → some congestion → those cut back → stable equilibrium
```

### Visual — cwnd Growth Over Time

```
cwnd
  │
  │                              /\
  │                             /  \
  │                            /    \
  │                    /\    /      \
  │                   /  \  /        \
  │                  /    \/          \
  │         /\    /                  \
  │        /  \  /                    \
  │       /    \/                      \
  │      /                               \
  │     /                                 \
  │    /                                   \
  │   /                                     \
  │  /                                       \
  │ /                                         \
  │/                                           \
  └──────────────────────────────────────────────── time
       ↑     ↑         ↑              ↑
    Slow   Cong.    Loss          Loss
    Start  Avoid.  (timeout)     (3 dup ACKs)
```

### Practical Impact

| Scenario | What Congestion Control Does For You |
|---|---|
| **API spike (1000 req/s)** | TCP automatically backs off — no app-level rate limiting needed |
| **Cross-region DB call** | Slow Start means first few requests are slow — connection pooling avoids this |
| **Video streaming** | Adjusts throughput in real-time during congestion |
| **WebSocket under load** | TCP reduces send rate when network congested |
| **Cloud deployment** | AWS/Azure network paths have different profiles — TCP adapts automatically |

### Summary

```
Congestion Control = Network-friendly send rate management

  Slow Start:     Exponential growth (cwnd × 2 per RTT)
  Congestion Avoidance: Linear growth (cwnd + 1 per RTT)
  Loss detected:  ssthresh = cwnd/2, cwnd reset
  Fast Retransmit: 3 dup ACKs → resend immediately
  Fast Recovery:   After dup ACKs, skip Slow Start, go to Congestion Avoidance

  Effective Window = min(cwnd, rwnd)
  AIMD: Additive Increase, Multiplicative Decrease
```

---

## Slow Start vs Congestion Avoidance

<!-- Lecture 25 notes will be added here -->

---

## NAT

<!-- Lecture 26 notes will be added here -->

---

## TCP Connection States

<!-- Lecture 27 notes will be added here -->

---

## TCP Pros and Cons

### Pros (Advantages)

#### 1. Reliable Delivery
Every byte sent is tracked. If a segment is lost, TCP retransmits it automatically.

```
Sender: sends segment with Seq = 100
Receiver: doesn't get it
Sender: no ACK for Seq = 100 → retransmits
Receiver: gets it → sends ACK
```

**Why it matters:** You never have to build your own retry logic. TCP handles it at the transport layer.

**Full-Stack impact:** Every HTTP request, every database query, every API call — all rely on this.

---

#### 2. Ordered Delivery
TCP uses sequence numbers to reorder out-of-order segments. The application always receives data in the exact order it was sent.

```
Send:    "Hello" → "World" → "!"
Network: "World" arrives first (out of order)
TCP:     Reorders → "Hello" → "World" → "!"
App:     Receives in correct order
```

**Why it matters:** Without ordering, files would be corrupted, JSON responses would be broken, HTML pages would not render.

---

#### 3. Flow Control
TCP uses a **sliding window** mechanism. The receiver tells the sender how much data it can handle.

```
Receiver: Window = 65535 → "Send me up to 65535 bytes"
Receiver buffer fills → Window = 0 → "Stop, I'm full"
Receiver processes data → Window opens → "Send more"
```

**Why it matters:** A fast sender can't overwhelm a slow receiver.

**Full-Stack impact:** When your database is under load and slows down, TCP automatically reduces the send rate from your app.

---

#### 4. Congestion Control
TCP monitors the **network** and adjusts its sending rate to avoid contributing to congestion.

```
Network is fast → increase sending rate (Slow Start)
Network is busy → slow down (Congestion Avoidance)
Network is congested → drastically reduce rate (Multiplicative Decrease)
```

**Why it matters:** If everyone sent at maximum speed all the time, the internet would collapse.

**Full-Stack impact:** During traffic spikes, TCP automatically backs off. Your app doesn't overwhelm the network.

---

#### 5. Connection-Oriented (Stateful)
The 3-way handshake establishes a verified connection before data flows. Both sides know who they're talking to and the connection is alive.

**Why it matters:** You can manage connections — close them gracefully, detect when they're dead, pool and reuse them.

**Full-Stack impact:** Connection pooling, keep-alive, WebSocket persistence — all built on this.

---

#### 6. Error Detection (Checksum)
Every segment has a checksum. The receiver validates it. Corrupted segments are silently dropped, triggering retransmission.

```
Sender: Calculates checksum
Receiver: Recalculates → compares
  Match → accept
  Mismatch → drop → no ACK → sender retransmits
```

**Why it matters:** Network corruption happens (interference, router bugs, memory errors). TCP catches it automatically.

---

#### 7. Multiplexing and Demultiplexing
TCP uses port numbers to multiplex multiple connections over a single IP address and demultiplex incoming data to the correct application.

```
Host: 192.168.1.20
  Port 80 → Web server
  Port 22 → SSH
  Port 3000 → Node.js app
  Port 5432 → PostgreSQL
```

**Why it matters:** Multiple services run simultaneously on one machine without interfering.

---

#### 8. Wide Window Size (with Scaling)
With Window Scaling option, TCP can use windows up to **1 GB** (instead of 65,535 bytes).

```
Standard: Window = 65535 bytes = 64 KB
Scaled:   Window = 65535 × 128 = 8 MB (or more)
```

**Why it matters:** On high-speed, high-latency networks, a 64 KB window would bottleneck throughput.

**Full-Stack impact:** When your server communicates with a database across continents, window scaling keeps the pipe full.

---

### Cons (Disadvantages)

#### 1. Header Overhead
TCP header is **20-60 bytes** vs UDP's **8 bytes**.

```
TCP:  20-60 bytes header
UDP:  8 bytes header
TCP is 2.5x to 7.5x more header overhead per segment.
```

**When it hurts:** Small messages with large headers. Each DNS query (often < 100 bytes) wastes significant space on headers.

---

#### 2. Connection Setup Cost — 3-Way Handshake
Every new connection requires 1 RTT before any data is sent.

```
First connection:
  1 RTT (TCP handshake) + 1-2 RTT (TLS) + 1 RTT (HTTP) = 3-4 RTT

  At 100ms RTT: 300-400ms before first byte
```

**When it hurts:** Short-lived connections, APIs that open a new connection per request.

**Solution:** Connection pooling, keep-alive, HTTP/2 multiplexing.

---

#### 3. Head-of-Line Blocking
If one segment is lost, ALL subsequent segments wait — even if they arrived fine.

```
Segment 1: LOST
Segment 2: Arrived fine → BLOCKED (waiting for Seq 1)
Segment 3: Arrived fine → BLOCKED
Segment 4: Arrived fine → BLOCKED

All stuck until segment 1 is retransmitted.
```

**When it hurts:** HTTP/2 multiplexing over TCP. One slow stream blocks all other streams. This was a major reason HTTP/3 moved to QUIC/UDP.

**Full-Stack impact:** If your HTTP/2 connection has one slow request, all other requests on that connection stall.

---

#### 4. Stateful = Memory Cost
Every connection consumes memory on both sides.

```
Per connection: ~4KB-64KB
10,000 connections: ~160 MB - 1 GB
100,000 connections: ~1.6 GB - 10 GB
```

**When it hurts:** High-concurrency systems (chat apps with millions of users, real-time gaming).

**Solution:** Load balancers, connection limits, stateless architectures.

---

#### 5. Slower Than UDP
Every reliability feature adds latency:

```
TCP:  Data → [handshake] → [wait for ACK] → [retransmit if lost] → [reorder] → [flow control] → delivered
UDP:  Data → send → done
```

**When it hurts:** Real-time applications where latency matters more than perfection. Voice calls, video streaming, gaming, IoT sensors.

---

#### 6. No Broadcasting or Multicasting
TCP is **point-to-point** — one sender, one receiver. Cannot:
- Send to all devices on a network (broadcast)
- Send to a group of devices (multicast)

**When it hurts:** Service discovery, network monitoring, stock tickers, live event streaming.

**UDP alternative:** UDP supports broadcast and multicast natively.

---

#### 7. Complex to Implement at Scale
Managing thousands of TCP connections requires:
- Connection pooling
- Timeout management
- Keep-alive configuration
- Load balancing (sticky sessions)
- Graceful shutdown handling
- File descriptor limits

**When it hurts:** Large-scale distributed systems. Each adds operational complexity.

---

### TCP vs UDP — Side-by-Side

| Feature | TCP | UDP |
|---------|-----|-----|
| **Reliability** | Guaranteed | Not guaranteed |
| **Ordering** | Yes | No |
| **Connection** | 3-way handshake | None |
| **Flow control** | Yes | No |
| **Congestion control** | Yes | No |
| **Header size** | 20-60 bytes | 8 bytes |
| **Speed** | Slower | Faster |
| **Broadcast/Multicast** | No | Yes |
| **State** | Stateful | Stateless |
| **Overhead** | High | Low |
| **Use when** | Data must arrive | Speed matters |

---

### Decision Framework

```
                    ┌──────────────────────────┐
                    │  Does data MUST arrive?  │
                    └────────────┬─────────────┘
                                 │
                        Yes ─────┴───── No
                         │                 │
                         ▼                 ▼
              ┌─────────────────┐  ┌─────────────────┐
              │  Use TCP        │  │  Consider UDP   │
              │  (reliable)     │  │  (fast)         │
              └─────────────────┘  └─────────────────┘
```

### Use TCP When:

| Scenario | Why TCP | Example |
|----------|---------|---------|
| Data must arrive completely | Every byte matters | File transfer, email |
| Order matters | Sequence must be preserved | Database transactions |
| Commands must execute correctly | No corruption | SSH, Telnet |
| Web pages must load fully | Incomplete = broken | HTTP/1.1, HTTP/2 |
| API responses must be reliable | Missing data = broken app | REST APIs, GraphQL |
| Transactions must be atomic | Partial = corruption | Financial operations |

### Use UDP When:

| Scenario | Why UDP | Example |
|----------|---------|---------|
| Speed matters more | Latency > perfection | Video streaming, VoIP |
| Old data is useless | Fresh data is everything | Gaming position updates |
| Single query/response | No need for connection | DNS queries |
| Broadcasting needed | One-to-many | Service discovery |
| Avoid TCP overhead | TCP overhead wasteful | VPN tunnels |
| Application has own reliability | Custom retry logic | QUIC (HTTP/3) |

---

### Real-World Protocol Choices

| Protocol | Transport | Why |
|----------|-----------|-----|
| HTTP/1.1 | TCP | Reliable web pages |
| HTTP/2 | TCP | Multiplexing over reliable connection |
| HTTP/3 | UDP (QUIC) | Avoid head-of-line blocking |
| DNS | UDP (usually) | Small, fast, single query/response |
| SSH | TCP | Reliable command execution |
| FTP | TCP | File transfer must be complete |
| SMTP (email) | TCP | Email must arrive |
| VoIP (RTP) | UDP | Real-time audio |
| Video streaming | UDP | Drop frames, don't wait |
| Online gaming | UDP | Position updates — old data useless |
| DHCP | UDP | Discovery before having an IP |

---

### The Core Trade-off

> **TCP guarantees delivery at the cost of speed. UDP guarantees speed at the cost of delivery.**

---

### Full-Stack Engineer's Cheat Sheet

```
Building a REST API?          → TCP (HTTP over TCP)
Building a chat app?          → TCP (WebSocket over TCP) for messages
                                → UDP for typing indicators
Building a video call?        → UDP (WebRTC/RTP)
Building a game?              → UDP (position updates)
Building a file transfer?     → TCP (FTP, HTTP)
Building a DNS resolver?      → UDP (fallback to TCP for large responses)
Building a VPN?               → UDP (avoid TCP-over-TCP)
Building a database driver?   → TCP (PostgreSQL, MySQL)
```

---

### Summary at a Glance

**Pros:**
1. Reliable delivery — automatic retransmission
2. Ordered delivery — sequence numbers
3. Flow control — receiver-driven rate
4. Congestion control — network-friendly
5. Connection-oriented — manageable state
6. Error detection — checksum
7. Multiplexing — port-based delivery
8. Wide windows — high throughput on fast networks

**Cons:**
1. Header overhead — 20-60 bytes vs 8
2. Handshake cost — 1 RTT before data
3. Head-of-line blocking — one loss stalls all
4. Memory cost — state per connection
5. Slower — every feature adds latency
6. No broadcast/multicast — point-to-point only
7. Complex at scale — pooling, limits, timeouts

---

### What's Next?
Lecture 29: **Sockets, Connections and Kernel Queues**

---

## Sockets, Connections and Kernel Queues

<!-- Lecture 29 notes will be added here -->

---

## TCP Server with JavaScript using NodeJS

<!-- Lecture 30 notes will be added here -->

---

## TCP Server with C

<!-- Lecture 31 notes will be added here -->

---

## Capturing TCP Segments with TCPDUMP

<!-- Lecture 32 notes will be added here -->

---

## Key Takeaways

<!-- Will be populated after completing all lectures -->

---

### What's Next?

</details>
