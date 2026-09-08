# User Datagram Protocol (UDP)

<details>
<summary><b>UDP — Section 4</b></summary>

### Lectures

- [x] 15. What Is UDP? (13min)
- [ ] 16. User Datagram Structure (6min)
- [ ] 17. UDP Pros & Cons (12min)
- [ ] 18. UDP Server with Javascript using NodeJS (9min)
- [ ] 19. UDP Server with C (8min)
- [ ] 20. Capturing UDP traffic with TCPDUMP (10min)

---

## What Is UDP?

**UDP = User Datagram Protocol**

Layer 4 (Transport Layer) protocol. It is **encapsulated inside an IP packet** — the IP header's Protocol field is set to `17` to indicate UDP.

```
┌────────────────────────────────────────┐
│ UDP Datagram ( encapsulated in IP )    │
│   Protocol = 17 (UDP)                 │
└────────────────────────────────────────┘
         ↓ embedded in
┌────────────────────────────────────────┐
│ IP Header (Protocol = 17)              │
│   Source IP                            │
│   Destination IP                       │
└────────────────────────────────────────┘
```

---

## Layer 4 — Transport Layer

The **Transport Layer** sits between the Network Layer (Layer 3) and the Application Layer (Layer 7).

### Main Role

**It provides communication between applications running on different hosts.**

```
┌─────────────────────────────────────┐
│  Application Layer (Layer 7)        │  HTTP, DNS, SSH, etc.
├─────────────────────────────────────┤
│  Transport Layer (Layer 4)          │  ← THIS IS WHERE WE ARE
│  TCP, UDP                           │  Provides app-to-app communication
├─────────────────────────────────────┤
│  Network Layer (Layer 3)            │  IP — gets packets to the right host
├─────────────────────────────────────┤
│  Link Layer (Layer 2)               │  Ethernet — gets frames on the local network
└─────────────────────────────────────┘
```

### Layer 3 vs Layer 4

| Layer | What it does | Analogy |
|-------|-------------|---------|
| **Layer 3 (IP)** | Gets packets to the **right device** | Delivers a letter to the right house |
| **Layer 4 (TCP/UDP)** | Gets data to the **right application** on that device | Delivers the letter to the right person inside the house |

### The Key Concept: Ports

Layer 4 uses **port numbers** to identify which application should receive the data.

```
IP Address: 192.168.1.20  → "This is my computer"
Port 80:    → "Send HTTP traffic here"
Port 443:   → "Send HTTPS traffic here"
Port 53:    → "Send DNS traffic here"
```

### Layer 4 Responsibilities

| Responsibility | TCP | UDP |
|----------------|-----|-----|
| Connection-oriented | Yes | No |
| Reliable delivery | Yes | No |
| Ordering | Yes | No |
| Flow control | Yes | No |
| Error checking | Yes | Yes (checksum) |
| Speed | Slower | Faster |

---

## How UDP Addresses Processes Using Ports

### The Problem

A single host runs **many applications** at the same time:

```
Host: 192.168.1.20
  ├── Web browser (port 52341)
  ├── SSH client (port 22)
  ├── DNS resolver (port 53)
  ├── Video call (port 3478)
  └── Game (port 27015)
```

When data arrives at the host, **how does the OS know which application gets it?**

### The Solution: Port Numbers

**Port numbers identify which application/process receives the data.**

```
Packet arrives at 192.168.1.20
    ↓
OS reads Destination Port
    ↓
Port 53 → DNS resolver
Port 80  → Web server
Port 22  → SSH
```

### How UDP Uses Ports

```
┌─────────────────────────────────────────────────────┐
│              UDP Segment                              │
├─────────────────────────────────────────────────────┤
│ Source Port (16 bits)  │ Destination Port (16 bits) │
├─────────────────────────────────────────────────────┤
│ Length (16 bits)       │ Checksum (16 bits)         │
├─────────────────────────────────────────────────────┤
│ Data (application payload)                           │
└─────────────────────────────────────────────────────┘
```

**Source Port:** Which application sent this
**Destination Port:** Which application should receive this

### Real-World Example

```
DNS Query:
  Source Port: 52341 (your app, randomly chosen)
  Destination Port: 53 (DNS server)

DNS Response:
  Source Port: 53 (DNS server)
  Destination Port: 52341 (your app)
```

The OS uses the destination port to deliver the response to the correct application.

### Well-Known Ports

| Port | Protocol | Purpose |
|------|----------|---------|
| 53 | DNS | Domain name resolution |
| 67/68 | DHCP | IP address assignment |
| 69 | TFTP | Simple file transfer |
| 123 | NTP | Time synchronization |
| 5353 | mDNS | Multicast DNS (local network) |

### UDP vs TCP Port Usage

| Feature | UDP | TCP |
|---------|-----|-----|
| Port concept | Same | Same |
| Connection | No handshake | 3-way handshake |
| Reliability | No guarantee | Guaranteed delivery |
| Speed | Faster | Slower |

### The Full Picture

```
Application wants to send DNS query
    ↓
UDP adds: Source Port (random), Destination Port (53)
    ↓
IP adds: Source IP, Destination IP
    ↓
Ethernet adds: Source MAC, Destination MAC
    ↓
Transmitted over network
```

At the receiver:

```
Ethernet removes MAC header
    ↓
IP removes IP header
    ↓
UDP reads Destination Port = 53
    ↓
OS delivers data to DNS resolver process
```

---

## UDP: Stateless & No Prior Communication Required

### What "No Prior Communication" Means

```
TCP:  You must establish a connection first
      Client → SYN → Server
      Server → SYN-ACK → Client
      Client → ACK → Server
      Connection established → data flows

UDP:  No connection needed
      Client → Data → Server
      Server → Data → Client
      No handshake, no setup
```

You can just **send data** without asking permission or establishing anything first.

### What "Stateless" Means

The server **doesn't remember** anything about you between messages.

```
TCP (stateful):
  Server remembers: "This is client X, I sent them packet #5"
  Server tracks: connection state, sequence numbers, window size

UDP (stateless):
  Server receives packet → processes it → forgets it
  Server has NO memory of previous packets
```

Each packet is **independent**. The server treats every UDP datagram as a brand new request.

### The Double-Edged Sword

**Edge 1 — The Good Side:**

| Advantage | Explanation |
|-----------|-------------|
| Fast | No connection setup overhead |
| Scalable | Server handles millions of clients without tracking state |
| Simple | No connection management logic |
| Low memory | Server doesn't store connection state |

**Edge 2 — The Bad Side:**

| Disadvantage | Explanation |
|--------------|-------------|
| No guarantee | Packet might be lost, duplicated, or arrive out of order |
| No ordering | Packets arrive in whatever order the network gives them |
| No recovery | If a packet is lost, there's no automatic retransmission |
| No flow control | Server can't slow down a fast sender |

### Real-World Analogy

```
TCP = Phone call
  You dial → connection established → talk → hang up
  Both parties know the call is active
  If you drop, you reconnect

UDP = Walkie-talkie / Radio broadcast
  You just press button and speak
  No connection needed
  Message might not reach the other person
  They might hear messages out of order
  They don't remember what you said last time
```

### Why It Matters for Full-Stack Engineers

| Use Case | Why UDP works |
|----------|---------------|
| DNS | Single query, single response — no need for connection |
| Video streaming | Drop a frame? Better to skip it than wait for retransmission |
| VoIP/Video calls | Latency matters more than perfect delivery |
| Gaming | Position updates — old data is useless, send new data fast |
| IoT sensors | Small frequent readings — connection overhead is wasteful |

---

## Speed vs Reliability — The Core Trade-off

### The Spectrum

```
←————— Reliability —————→
  TCP                          UDP
  Reliable                     Fast
  Ordered                      Unordered
  Connection-based             Connectionless
  Slow                         Fast
```

### Why TCP Is Reliable (But Slower)

Every reliability feature adds **time and overhead**:

```
1. Connection setup (3-way handshake) → 1 RTT delay
2. Acknowledgments for every packet → waiting time
3. Retransmission on loss → waiting time
4. Ordering → buffering and reordering
5. Flow control → sender slows down
```

```
TCP:  Data → [handshake] → [ACK wait] → [retransmit if lost] → [reorder] → [flow control] → delivered
```

Each feature adds latency. That's why TCP is **slower** but **reliable**.

### Why UDP Is Fast (But Unreliable)

```
UDP:  Data → [send immediately] → delivered
```

No handshake, no ACKs, no retransmission, no ordering, no flow control.

```
UDP:  Data → send → done
```

That's why UDP is **faster** but **unreliable**.

### Side-by-Side Comparison

| Feature | TCP | UDP | Impact |
|---------|-----|-----|--------|
| Connection setup | 3-way handshake | None | TCP adds 1 RTT delay |
| Acknowledgments | Yes | No | TCP waits for ACKs |
| Retransmission | Yes | No | TCP resends lost packets |
| Ordering | Yes | No | TCP buffers and reorders |
| Flow control | Yes | No | TCP slows down sender |
| Speed | Slower | Faster | UDP has minimal overhead |
| Reliability | Guaranteed | Not guaranteed | TCP delivers, UDP doesn't |

### Real-World Analogy

```
TCP = Certified mail
  You send → recipient signs → you get confirmation
  If lost → you resend
  Guaranteed delivery, but slower

UDP = Regular postcard
  You send → hope it arrives
  No confirmation, no resend
  Fast, but no guarantee
```

### Why Choose One Over the Other?

| Choose TCP when... | Choose UDP when... |
|---------------------|-------------------|
| Data must arrive | Speed matters more |
| No data loss allowed | Some loss is acceptable |
| Order matters | Old data is useless |
| File transfer, email | Video, voice, gaming |
| Web browsing, databases | DNS, streaming, IoT |

### The Trade-off in One Sentence

> **TCP guarantees delivery at the cost of speed. UDP guarantees speed at the cost of delivery.**

---

## Key Takeaways

1. **UDP is Layer 4** — provides app-to-app communication using ports
2. **Stateless** — server remembers nothing between packets
3. **No prior communication** — send data immediately, no handshake
4. **No ports at Layer 3** — ICMP has no ports, UDP/TCP do
5. **Speed vs reliability** — UDP is fast but unreliable, TCP is slow but reliable
6. **Use cases** — DNS, video streaming, VoIP, gaming, IoT

---

## UDP Use Cases

### 1. DNS

**Why UDP?**

- Single query → single response
- Very small payload (usually < 512 bytes)
- No need for connection overhead
- Fast resolution is everything

```
You: "What's google.com's IP?"
DNS: "142.250.x.x"
Done. Why establish a TCP connection for that?
```

**But:** If the response is too large for a single UDP packet (> 512 bytes), DNS falls back to TCP.

---

### 2. Video Streaming

**Why UDP?**

- **Latency matters more than perfection**
- A late frame is useless — better to skip it and show the next one
- TCP retransmission would introduce buffering and stuttering
- Slight packet loss is acceptable (you won't notice a few dropped frames)

```
TCP approach: frame lost → wait for retransmit → video stutters → bad UX
UDP approach: frame lost → skip it → show next frame → smooth video
```

**But:** Pure UDP streaming has no recovery, so protocols like **QUIC** (HTTP/3) or **RTP** add their own lightweight reliability on top.

---

### 3. VoIP / Video Calls (WebRTC)

**Why UDP?**

- **Real-time is everything** — you can't wait for retransmission
- 200ms delay = noticeable lag. 500ms = unusable.
- If a packet is lost, the audio/video has already moved on
- Human perception masks small losses

```
TCP: "You said 'hello'... wait, let me resend that packet... here it is"
UDP: "You said 'hello'... I missed part of it... I'll fill in the gap from context"
```

**WebRTC** uses UDP (via RTP) for media, and adds just enough reliability for signaling — not for the actual audio/video.

---

### 4. VPN Tunnels

**Why UDP?**

- VPNs need to encapsulate ALL traffic (TCP, UDP, ICMP)
- Using TCP-over-TCP causes **TCP meltdown** — two congestion control layers fighting each other
- UDP avoids this: no congestion control on the tunnel, only on the inner connection

```
Without VPN:
  Your TCP → Internet → Server

With TCP-based VPN:
  Your TCP → Tunnel TCP → Internet → Server
  Two TCP layers = both try to control congestion = chaos

With UDP-based VPN:
  Your TCP → Tunnel UDP → Internet → Server
  Only one TCP layer (your app) controls congestion = clean
```

**Most modern VPNs (WireGuard, OpenVPN) use UDP** for exactly this reason.

---

### Summary Table

| Use Case | Why UDP? | What breaks with TCP? |
|----------|----------|----------------------|
| DNS | Small, fast, single request/response | Connection overhead for tiny queries |
| Video streaming | Latency > perfection | Retransmission causes stuttering |
| VoIP/WebRTC | Real-time, human perception masks loss | Buffering from retransmission |
| VPN | Avoid TCP-over-TCP meltdown | Two congestion control layers |

---

### The Common Thread

All these use cases share the same principle:

> **Speed and freshness matter more than guaranteed delivery.**

If the data is old by the time it arrives, it's worthless. UDP delivers it fast — if it arrives at all.

---

## Multiplexing and Demultiplexing

### The Problem

A single host runs **many applications** at the same time:

```
Host: 192.168.1.20
  ├── Web browser (port 52341) → sending HTTP request
  ├── SSH client (port 22) → sending commands
  ├── DNS resolver (port 53) → resolving google.com
  └── Spotify (port 43768) → streaming music
```

All these applications are sending data **at the same time** over the **same network interface**. How does the OS keep them separate?

---

### Multiplexing (Sender Side)

**Multiplexing = combining multiple data streams into one.**

The transport layer takes data from multiple applications and sends it over a single network connection.

```
Browser: "Send this HTTP request"
  ↓
Transport layer adds: Source Port (52341), Destination Port (80)
  ↓
SSH: "Send this command"
  ↓
Transport layer adds: Source Port (22), Destination Port (22)
  ↓
Both are sent over the same network interface, same IP address
```

**How it works:**

1. Each application gets a **unique port number**
2. Transport layer wraps each app's data with its port number
3. All streams are sent over the same IP address

---

### Demultiplexing (Receiver Side)

**Demultiplexing = separating combined streams back to their original applications.**

When data arrives at the destination:

```
Packet arrives at 192.168.1.20
  ↓
Transport layer reads: Destination Port = 53
  ↓
Delivers to: DNS resolver process
  ↓
Next packet arrives
  ↓
Transport layer reads: Destination Port = 52341
  ↓
Delivers to: Web browser process
```

**How it works:**

1. Transport layer reads the **destination port** from each packet
2. Looks up which application owns that port
3. Delivers the data to that application

---

### The Port Table

Your OS maintains a **port table**:

```
Port 53    → DNS resolver (listening)
Port 80    → Web server (listening)
Port 22    → SSH daemon (listening)
Port 52341 → Web browser (ephemeral, temporary)
```

When a packet arrives with `Destination Port = 53`, the OS knows to deliver it to the DNS resolver.

---

### Analogy

```
Multiplexing = Multiple letters going into the same mailbox
Demultiplexing = Postman sorting mail by apartment number and delivering to each recipient

Mailbox = Your computer's IP address
Apartment number = Port number
Letters = Data packets
```

---

### TCP vs UDP Multiplexing

Both TCP and UDP use the same port concept:

| Feature | TCP | UDP |
|---------|-----|-----|
| Multiplexing | Uses ports | Uses ports |
| Demultiplexing | Uses ports | Uses ports |
| Connection state | Tracks connections | No connections |
| Port reuse | 4-tuple (src IP, src port, dst IP, dst port) | 2-tuple (dst IP, dst port) |

**TCP uses a 4-tuple:**
```
(src IP, src port, dst IP, dst port) = unique connection
```

**UDP uses a 2-tuple:**
```
(dst IP, dst port) = destination
```

This is why UDP is simpler — it doesn't need to track connections, just ports.

---

## UDP Datagram Structure

### UDP Header

The UDP header is only **8 bytes** — much smaller than IP's 20-byte minimum.

```
┌─────────────────────────────────────────────────────────┐
│                    UDP Header (8 bytes)                  │
├────────────────────┬────────────────────────────────────┤
│  Source Port       │  Destination Port                  │
│  (16 bits)         │  (16 bits)                         │
├────────────────────┴────────────────────────────────────┤
│  Length             │  Checksum                          │
│  (16 bits)          │  (16 bits)                         │
└──────────────────────┴──────────────────────────────────┘
```

| Field | Size | Purpose |
|-------|------|---------|
| Source Port | 16 bits (2 bytes) | Which application sent this |
| Destination Port | 16 bits (2 bytes) | Which application should receive this |
| Length | 16 bits (2 bytes) | Total size of UDP datagram (header + data) |
| Checksum | 16 bits (2 bytes) | Error checking |

---

### How UDP Sits Inside IP

The entire UDP datagram (header + payload) becomes the **data portion** of the IP packet:

```
┌────────────────────────────────────────┐
│ IP Header (Protocol = 17 for UDP)      │
├────────────────────────────────────────┤
│ UDP Header (8 bytes)                   │
│   Source Port | Destination Port        │
│   Length      | Checksum                │
├────────────────────────────────────────┤
│ UDP Payload (your actual data)         │
└────────────────────────────────────────┘
```

---

### Port Numbers Are 16 Bits

Valid port range: **0 to 65,535**

| Port Range | Purpose |
|------------|---------|
| 0-1023 | Well-known ports (DNS=53, HTTP=80, HTTPS=443) |
| 1024-49151 | Registered ports |
| 49152-65535 | Ephemeral ports (temporary, client-side) |

---

### The Length Field

```
Length = UDP header (8 bytes) + Data payload
```

Example:
```
Data = 100 bytes
UDP header = 8 bytes
Length field = 108 bytes
```

This tells the receiver where the UDP datagram ends.

---

### The Checksum Field

Same concept as IP header checksum — validates the UDP header wasn't corrupted. Unlike IP, UDP checksum also covers part of the IP header (pseudo-header) for extra validation.

---

## What's Next?

Lecture 17: **UDP Pros & Cons** — when to use UDP and when to avoid it.

</details>
