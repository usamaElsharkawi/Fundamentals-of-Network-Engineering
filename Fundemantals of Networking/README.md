# Fundamentals of Networking

<details>
<summary><b>Client-Server Architecture</b></summary>

### 1. The Client-Server Architecture
The **client-server architecture** is a paradigm where an **always-on host (the server)** services requests from many other **initiating hosts (clients)**. 
*   **The Server:** Typically possesses "beefy" hardware to handle expensive workloads and maintains a fixed, well-known IP address.
*   **The Client:** Generally uses commodity hardware and performs lightweight tasks, contacting the server as needed.
*   **Centralization:** A defining characteristic is that **clients do not communicate directly** with one another; all coordination happens through the central server.

### 2. The Three-Tier Architecture
Considered a specialized case of the client-server model, the **three-tier architecture** physically and logically separates an application into three levels to improve scalability and security.
*   **Presentation Tier (Frontend):** The user interface that collects data and displays results.
*   **Application Tier (Logic/Middleware):** The "heart of the application" where business rules are implemented. 
*   **Data Tier (Backend):** Where information is stored and managed by a DBMS.
*   **Interaction Rule:** A key principle is that the **presentation tier cannot talk directly to the data tier**; all requests must pass through the Application Tier's APIs.

### 3. Remote Procedure Call (RPC) and gRPC
To manage the complexity of these architectures, developers use **RPC** to allow one machine to invoke code on another as if it were a local function call.
*   **gRPC:** A modern, high-performance RPC framework developed by Google.
*   **Efficiency:** It operates at **Layer 7 (Application Layer)** and uses **Protocol Buffers** for efficient binary encoding, which is significantly faster than JSON.
*   **Transport:** It is built on top of **HTTP/2**, allowing multiple concurrent calls over a single long-lived TCP connection.

### 4. Scaling the Architecture
"Scaling better" in this model involves centralizing work on powerful infrastructure to handle massive traffic.
*   **Data Centers:** Companies use hundreds of thousands of hosts in a data center to act as a single **powerful "virtual" server**.
*   **Load Balancers:** These devices distribute traffic across internal hosts. **Layer 7 Load Balancers** are "protocol-aware," looking at application data (like HTTP headers) to route requests to specific microservices.
*   **CDNs:** Content Distribution Networks scale capacity globally by caching content in distributed "edge" locations closer to users.

### 5. Microservices and Disaggregation
Microservices borrowed the request-response and RPC models from client-server architecture but expanded them into **"disaggregated" functional units**. 
*   This allows for **decoupled innovation**, where individual services (Network Functions in 5G, for example) can be updated or scaled independently without changing the whole system.

### 6. Edge Computing: The Modern Trend
**Edge computing** involves placing workloads and data processing as close to the "edge" of the network—where data is created—as possible.
*   **The Spectrum:** It ranges from **Edge Devices** (like cars with 50 CPUs) to **Edge Servers** (on-premise racks) and the **Network Edge** (5G base stations).
*   **Benefits:** It reduces latency for real-time apps, saves bandwidth costs, and improves security by keeping sensitive data local.
*   **Trade-offs:** Edge environments often have **minimal runtimes** (e.g., a 1MB code limit) and can actually be **slower** if they must fetch data from a distant central database. 

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

<details>
<summary><b>The OSI Model</b></summary>

The OSI (Open Systems Interconnection) model is a standardized reference framework that describes how information from a software application in one computer moves through a network medium to a software application in another computer. Defined by the International Organization for Standardization (ISO), it organizes networking into seven distinct layers, each responsible for a specific facet of communication.

### Why We Need a Communication Model
A standardized model is essential for three primary reasons:
*   **Agnostic Applications:** It allows developers to write applications that do not need to know the specifics of the underlying network medium (e.g., your app works the same on Wi-Fi, Ethernet, or LTE).
*   **Network Equipment Management:** It makes upgrading and managing hardware easier because devices can interoperate as long as they follow the standard.
*   **Decoupled Innovation:** Changes or innovations can be made in one layer independently without affecting the rest of the stack.

### The Seven Layers of the OSI Model
The layers are numbered from bottom (Layer 1) to top (Layer 7):
*   **Layer 7: Application:** This is the layer users interact with directly through software. It handles protocols like HTTP for the web, FTP for file transfers, and gRPC.
*   **Layer 6: Presentation:** This layer is responsible for encoding, serialization, and format conversions. For example, it converts complex application data like JSON into flat byte strings that can be transmitted.
*   **Layer 5: Session:** It manages the establishment, coordination, and termination of connections (sessions) between applications. It also handles protocols like TLS for securing the session.
*   **Layer 4: Transport:** This layer provides end-to-end communication services. It is responsible for error control, flow control, and congestion control. Key protocols here are TCP (reliable, ordered) and UDP (unreliable, fast).
*   **Layer 3: Network:** This layer handles host-to-host routing across different networks. It uses IP (Internet Protocol) to identify devices via IP addresses and determines the best path for data packets (datagrams).
*   **Layer 2: Data Link:** It provides hop-to-hop connectivity between two adjacent nodes on the same medium. It uses MAC addresses and organizes data into frames. Ethernet and Wi-Fi are common technologies at this layer.
*   **Layer 1: Physical:** This is the actual hardware level where digital bits are converted into electrical signals, radio waves, or pulses of light.

### The Data Journey: Encapsulation
When data is sent, it travels from the top layer down to the bottom. At each layer, the protocol adds its own control information—a header—to the data received from the layer above. This process is called encapsulation.
*   The Application Layer creates a message.
*   The Transport Layer adds a header to create a segment.
*   The Network Layer adds a header to create a datagram (or packet).
*   The Data Link Layer adds a header and often a trailer to create a frame.
*   The Physical Layer transmits these as raw bits.

When the data reaches the receiver, the process is reversed (decapsulation), and each layer strips off its corresponding header as the data moves back up the stack.

### OSI vs. TCP/IP Model
In modern networking practice, the OSI model is often considered to have too many layers, which can make it hard to distinguish specific functions (like where Layer 5 ends and Layer 6 begins). The TCP/IP model is a simpler, four-layer alternative that is more commonly used in the actual Internet architecture. It typically merges OSI Layers 5, 6, and 7 into a single Application Layer.

### Frontend and Backend Engineers in the OSI Model
In terms of the OSI model, frontend and backend engineers operate at different levels of the protocol stack, with backend engineers typically covering a broader range of the upper layers.

**Frontend Engineers: Shining at Layer 7 (Application)**
Frontend engineers primarily live and "shine" at the Application Layer (Layer 7). This is the top of the stack where the software interacts directly with the user.
*   **User Interface Logic:** Their expertise is in the Presentation Tier, creating the "face" of the application using HTML, CSS, and JavaScript to collect user information and display results.
*   **Browser-Side Execution:** They manage how the application behaves within the browser environment, specifically the Document Object Model (DOM) and CSS Object Model (CSSOM) pipelines.
*   **Application-Level Performance:** They focus on "front-end performance analysis," which involves optimizing the resource waterfall to ensure assets are discoverable and that rendering isn't blocked by heavy scripts.

**Backend Engineers: Shining Across Layers 4 through 7**
Backend engineers cover the "heart" (Application Tier) and "memory" (Data Tier) of the system, requiring them to manage functions across multiple OSI layers:
*   **Layer 7 (Application):** They develop the server-side logic that processes requests. This includes implementing protocols like HTTP or high-performance frameworks like gRPC for service-to-service communication.
*   **Layer 6 (Presentation):** They excel in this layer by managing serialization and encoding. For example, they handle the conversion of complex data into flat byte strings (JSON or Protocol Buffers) so it can be transmitted across the network.
*   **Layer 5 (Session):** They are responsible for connection management, which includes the establishment and termination of sessions, as well as managing TLS (Transport Layer Security) for secure communication.
*   **Layer 4 (Transport):** Backend engineers shine here by optimizing the "plumbing" of the application. They must understand and configure TCP and UDP to handle flow control, congestion control, and retransmissions to ensure reliability and speed. They also work with Layer 4 Load Balancers, which make routing decisions based on IP addresses and port numbers.

**Collaborative Performance**
While their primary layers differ, both roles must collaborate on web performance. A frontend engineer’s work on the rendering pipeline at Layer 7 can be negated by backend issues at lower layers, such as high network latency or slow Time to First Byte (TTFB) caused by unoptimized database queries or slow TCP handshakes.

![OSI Model](OSI%20model.png)

</details>
