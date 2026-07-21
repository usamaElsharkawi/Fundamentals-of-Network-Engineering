
# Fundamentals of Networking

## Client-Server Architecture

<details>
<summary><b>1. The Client-Server Architecture</b></summary>

The **client-server architecture** is a paradigm where an **always-on host (the server)** services requests from many other **initiating hosts (clients)**. 
*   **The Server:** Typically possesses "beefy" hardware to handle expensive workloads and maintains a fixed, well-known IP address.
*   **The Client:** Generally uses commodity hardware and performs lightweight tasks, contacting the server as needed.
*   **Centralization:** A defining characteristic is that **clients do not communicate directly** with one another; all coordination happens through the central server.

</details>

<details>
<summary><b>2. The Three-Tier Architecture</b></summary>

Considered a specialized case of the client-server model, the **three-tier architecture** physically and logically separates an application into three levels to improve scalability and security.
*   **Presentation Tier (Frontend):** The user interface that collects data and displays results.
*   **Application Tier (Logic/Middleware):** The "heart of the application" where business rules are implemented. 
*   **Data Tier (Backend):** Where information is stored and managed by a DBMS.
*   **Interaction Rule:** A key principle is that the **presentation tier cannot talk directly to the data tier**; all requests must pass through the Application Tier's APIs.

</details>

<details>
<summary><b>3. Remote Procedure Call (RPC) and gRPC</b></summary>

To manage the complexity of these architectures, developers use **RPC** to allow one machine to invoke code on another as if it were a local function call.
*   **gRPC:** A modern, high-performance RPC framework developed by Google.
*   **Efficiency:** It operates at **Layer 7 (Application Layer)** and uses **Protocol Buffers** for efficient binary encoding, which is significantly faster than JSON.
*   **Transport:** It is built on top of **HTTP/2**, allowing multiple concurrent calls over a single long-lived TCP connection.

</details>

<details>
<summary><b>4. Scaling the Architecture</b></summary>

"Scaling better" in this model involves centralizing work on powerful infrastructure to handle massive traffic.
*   **Data Centers:** Companies use hundreds of thousands of hosts in a data center to act as a single **powerful "virtual" server**.
*   **Load Balancers:** These devices distribute traffic across internal hosts. **Layer 7 Load Balancers** are "protocol-aware," looking at application data (like HTTP headers) to route requests to specific microservices.
*   **CDNs:** Content Distribution Networks scale capacity globally by caching content in distributed "edge" locations closer to users.

</details>

<details>
<summary><b>5. Microservices and Disaggregation</b></summary>

Microservices borrowed the request-response and RPC models from client-server architecture but expanded them into **"disaggregated" functional units**. 
*   This allows for **decoupled innovation**, where individual services (Network Functions in 5G, for example) can be updated or scaled independently without changing the whole system.

</details>

<details>
<summary><b>6. Edge Computing: The Modern Trend</b></summary>

**Edge computing** involves placing workloads and data processing as close to the "edge" of the network—where data is created—as possible.
*   **The Spectrum:** It ranges from **Edge Devices** (like cars with 50 CPUs) to **Edge Servers** (on-premise racks) and the **Network Edge** (5G base stations).
*   **Benefits:** It reduces latency for real-time apps, saves bandwidth costs, and improves security by keeping sensitive data local.
*   **Trade-offs:** Edge environments often have **minimal runtimes** (e.g., a 1MB code limit) and can actually be **slower** if they must fetch data from a distant central database. 

</details>
<details>
<summary><b>7. Architectural Evolution: From Client-Server to Edge Computing</b></summary>

The following summary synthesizes the architectural concepts discussed, linking them together as an evolution of networking engineering designed to manage complexity, cost, and performance.

### 1. The Foundational Client-Server Paradigm
The **client-server architecture** is the root of modern network applications. It was born from the need to separate complex, "expensive" workloads from the user's interface. In this model, a **centralized, always-on host (the server)** provides resources or services to multiple **initiating hosts (clients)**. A defining rule is that clients typically use commodity hardware and **do not communicate directly with one another**, instead relying on the server for all coordination.

### 2. Specialized Case: Three-Tier Architecture
As web and database applications grew, the standard two-tier client-server model evolved into the **three-tier architecture**. This model introduces a middle layer to handle the "heart" of the application:
*   **Presentation Tier:** The front-end user interface (GUI).
*   **Application Tier:** The middleware where **business logic and rules** are implemented.
*   **Data Tier:** The back-end where a Database Management System (DBMS) stores and manages physical data.
**Linking concept:** A critical rule in this architecture is that the presentation tier **cannot communicate directly with the data tier**; it must interact through the application tier's APIs. This allows each tier to be **scaled and updated individually**.

### 3. Backend Communication: RPC and gRPC
To enable these different tiers or services to work together, developers use **Remote Procedure Calls (RPC)**. RPC allows one machine to invoke code on another as if it were a local function call.
*   **gRPC:** A high-performance implementation of RPC created by Google. It operates at **Layer 7 (Application Layer)** and is built on **HTTP/2**, allowing for multiple concurrent "streams" of data over a single connection.
*   **Efficiency:** Unlike older models using JSON, gRPC uses **Protocol Buffers** for binary encoding, which is approximately **five times faster**.

### 4. Scaling through Centralization
"Scaling better" in a client-server context typically involves centralizing resources to handle massive traffic.
*   **Data Centers:** Companies build massive data centers housing hundreds of thousands of servers that act as a **powerful "virtual" server**.
*   **Load Balancers:** To manage traffic within these centers, **Load Balancers** distribute incoming requests across various internal hosts. **Layer 7 Load Balancers** are protocol-aware, looking at application-level data (like HTTP headers) to route traffic to specific services.

### 5. The Evolution to Microservices
Microservices represent the **"disaggregation"** of the traditional monolithic server into small, independent functional units. 
*   **Borrowing from Client-Server:** Microservices inherit the **request-response and RPC models**. 
*   **Benefit:** This architecture allows different teams to innovate on specific services independently without changing the entire infrastructure.

### 6. The Modern Shift: Edge Computing
While centralization in data centers provides power, it introduces **latency** because the speed of light is not instant. **Edge computing** addresses this by placing workloads as close to the "edge"—where data is created—as possible.
*   **The Spectrum:** Workloads are moved to **Edge Devices** (like cars with 50 CPUs) or **Edge Servers** (on-premise racks in factories or warehouses).
*   **Trade-off:** While the edge reduces latency for real-time tasks, it is often limited to **lightweight execution** (e.g., 1MB code limits) and can be slower if the device still needs to fetch data from a distant central database. 

![Client-Server Architecture](Client-server-architecture.png)

</details>


