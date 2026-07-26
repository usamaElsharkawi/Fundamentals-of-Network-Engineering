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
