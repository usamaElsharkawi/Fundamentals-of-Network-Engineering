# Overview of Popular Networking Protocols

<details>
<summary><b>Networking Protocols — Section 6</b></summary>

### Lectures

- [ ] 33. Networking Protocols Introduction (3min)
- [ ] 34. DNS (39min)
- [ ] 35. TLS (27min)
- [ ] 36. HTTPS, TLS, Keys and Certificates (1hr 2min)

---

## Networking Protocols Introduction

<!-- Lecture 33 notes will be added here -->

---

## DNS

### Definition
**DNS = Domain Name System** — Maps human-readable domain names to machine-readable IP addresses. The **phonebook of the internet**.

```
google.com → 142.250.80.46
```

Without DNS, you'd need to memorize every IP address.

---

### DNS Hierarchy

DNS is organized as a **tree**:

```
                    .                    (Root)
                   /|\
                  / | \
                 com org net gov        (TLD - Top Level Domain)
                /   |   |   \
              google microsoft facebook aws  (Domain)
              / |     |        \
             www mail api  docs       (Subdomain)
```

| Level | Example | Controlled By |
|-------|---------|---------------|
| **Root** | `.` | IANA/ICANN (13 root server clusters worldwide) |
| **TLD** | `.com`, `.org`, `.net` | Registrars (Verisign for .com) |
| **Domain** | `google.com` | Domain owner |
| **Subdomain** | `www.google.com`, `api.google.com` | Domain owner |

---

### DNS Record Types

Entries stored in DNS, each serving a different purpose:

| Record | Full Name | Purpose | Example |
|--------|-----------|---------|---------|
| **A** | Address | Domain → IPv4 | `google.com → 142.250.80.46` |
| **AAAA** | Address | Domain → IPv6 | `google.com → 2607:f8b0:...` |
| **CNAME** | Canonical Name | Alias → another domain | `www → google.com` |
| **MX** | Mail Exchange | Domain → mail server | `google.com → aspmx.l.google.com` |
| **NS** | Name Server | Domain → DNS servers | `google.com → ns1.google.com` |
| **TXT** | Text | Arbitrary text data | `google.com → "v=spf1 ..."` |
| **SOA** | Start of Authority | Zone metadata | Primary NS, serial, refresh timers |
| **PTR** | Pointer | IP → domain (reverse DNS) | `8.8.8.8 → dns.google` |
| **SRV** | Service | Service location | `_sip._tcp.example.com → server` |

**Most common for full-stack engineers:**
- **A** — Where is your web server?
- **CNAME** — Alias (CDNs use this heavily)
- **MX** — Where do emails go?
- **TXT** — Verification, SPF, DKIM for email security

---

### DNS Resolution — The Full Journey

When you type `google.com` in your browser:

```
Browser: "Where is google.com?"
  ↓
OS checks local cache → not found
  ↓
Resolver (ISP's DNS or 8.8.8.8):
  ↓
  Step 1: Ask ROOT server
    "Who handles .com?"
    → "Here are the .com TLD servers"

  Step 2: Ask .com TLD server
    "Who handles google.com?"
    → "Here are google.com's authoritative NS"

  Step 3: Ask google.com's authoritative NS
    "What's the IP for google.com?"
    → "142.250.80.46"

  ↓
Resolver returns 142.250.80.46 to browser
  ↓
Browser connects to 142.250.80.46
```

**Two query types:**

| Type | Description |
|------|-------------|
| **Recursive** | Client asks resolver to do ALL the work (root → TLD → authoritative). Resolver returns final answer. |
| **Iterative** | Resolver asks each server step by step. Each server says "ask THIS server next." |

In practice: Browser does a **recursive** query to the resolver. Resolver does **iterative** queries to find the answer.

---

### Caching — Why DNS is Fast

Every DNS response has a **TTL (Time To Live)** — how long to cache:

```
google.com → 142.250.80.46 (TTL: 300 seconds)

For 300 seconds, every query answers from cache.
After 300 seconds, query again for fresh answer.
```

**Caching layers:**

```
Browser cache (~1 min)
  ↓ miss
OS cache (~few min)
  ↓ miss
Router/ISP cache (~hours)
  ↓ miss
Recursive resolver (8.8.8.8) — does full resolution
  ↓ caches result
Returns answer to client
```

**TTL trade-offs:**
- **Low TTL** (60s): Changes propagate fast, but more DNS queries
- **High TTL** (86400s = 24h): Fewer queries, but changes take longer to propagate
- **During migration**: Lower TTL days BEFORE the move, then change, then raise TTL back up

---

### DNS + Your Full-Stack Apps

| Scenario | DNS Role |
|----------|----------|
| **Deploying to new server** | Update A record or CNAME to point to new IP |
| **Load balancing** | Multiple A records → DNS returns different IPs (round-robin) |
| **CDN (Cloudflare, AWS CloudFront)** | CNAME → CDN → returns nearest edge server |
| **Email delivery** | MX records tell sending servers where to deliver |
| **Service discovery** | SRV records for service locations |
| **Blue-green deployment** | Switch CNAME from blue to green instantly |
| **Kubernetes** | CoreDNS resolves pod/service names internally |

---

### DNS Security Concerns

**1. DNS Spoofing / Cache Poisoning**
```
Attacker: "Resolver, google.com → 1.2.3.4 (malicious IP)"
Resolver caches fake answer
Users → sent to malicious server
```
**Solution: DNSSEC** — Digital signatures on DNS records
```
DNS server signs records with private key
Resolver verifies with public key
If signature invalid → discard the answer
```

**2. DNS Hijacking**
```
Router compromised → DNS settings changed
All DNS queries go to attacker's resolver
→ Users sent to phishing sites
```

**3. DNS Amplification DDoS**
```
Attacker spoofs victim's IP
Sends small DNS query to open resolver
Resolver sends large response to victim
Small query → large response = amplification (up to 70x)
```

---

### DNS Resolution — Visual Summary

```
Client: "google.com?"
  ↓ (recursive query)
Resolver:
  ↓ (iterative)
Root Server: ".com servers are: ..."
  ↓
TLD Server (.com): "google.com NS are: ..."
  ↓
Authoritative NS: "google.com → 142.250.80.46 (TTL 300)"
  ↓
Resolver caches → returns to client
  ↓
Browser: "Got it! Connecting to 142.250.80.46"
```

---

### Key Takeaways

```
DNS = Domain Name System
  Purpose:  Map names → IP addresses (phonebook of the internet)
  Hierarchy: Root → TLD → Domain → Subdomain
  Records:  A, AAAA, CNAME, MX, NS, TXT, SOA, PTR, SRV
  Resolution: Recursive (client) + Iterative (resolver)
  Caching:  TTL-based at multiple layers
  Security: DNSSEC (signing), vulnerable to spoofing/hijacking
  Full-Stack: Deployments, CDNs, load balancing, email, service discovery
```

---

### What's Next?
Lecture 35: **TLS**

---

## TLS

<!-- Lecture 35 notes will be added here -->

---

## HTTPS, TLS, Keys and Certificates

<!-- Lecture 36 notes will be added here -->

---

### What's Next?

</details>
