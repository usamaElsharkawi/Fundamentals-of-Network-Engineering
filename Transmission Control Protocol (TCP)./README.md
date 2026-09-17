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

### What's Next?
Lecture 22: **TCP Segment** — the detailed structure of a TCP segment.

---

## TCP Segment

<!-- Lecture 22 notes will be added here -->

---

## Flow Control

<!-- Lecture 23 notes will be added here -->

---

## Congestion Control

<!-- Lecture 24 notes will be added here -->

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

<!-- Lecture 28 notes will be added here -->

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
