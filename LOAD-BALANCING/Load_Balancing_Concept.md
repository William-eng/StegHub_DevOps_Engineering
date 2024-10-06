# Load Balancing 
Load balancing is the process of distributing incoming network traffic across multiple servers to ensure no single server becomes overwhelmed and to improve the availability and performance of
applications or services. There are two key types of load balancers: Layer 4 (L4) Network Load Balancers and Layer 7 (L7) Application Load Balancers. These operate at different layers of the
OSI model and have distinct functionalities.
Load balancing and scaling are key strategies used to handle increased traffic and ensure the availability and reliability of web applications or services. 
In addition to the types of load balancers (L4 and L7), there are two important scaling strategies: vertical scaling and horizontal scaling. Both methods are essential for managing the growing 
demands on servers.

## Vertical Scaling (Scaling Up)
Vertical scaling involves increasing the capacity of a single server by adding more resources (CPU, RAM, or storage). This improves the server’s ability to handle more load without changing the 
overall system architecture. When a server is reaching its capacity limits, you can "scale up" by upgrading its hardware components (e.g., adding more CPUs, increasing memory, or using faster storage).

## Horizontal Scaling (Scaling Out)
Horizontal scaling involves adding more servers to the system to distribute the load. This type of scaling allows applications to handle higher traffic by spreading requests across multiple 
machines (nodes) rather than relying on a single server. As traffic increases, additional servers are added to the system, often behind a load balancer, which distributes incoming traffic among 
the servers.

## Load Balancing Concepts:
- Session Persistence (Sticky Sessions): Some load balancers provide session persistence, ensuring that a user’s requests are always sent to the same server during their session, which is necessary for certain applications.

- Health Checks: Load balancers periodically check the health of servers. If a server fails the health check, it is removed from the pool until it recovers.

- SSL Termination: Some load balancers handle SSL encryption and decryption, reducing the workload on backend servers.

- Traffic Distribution Algorithms:

     - Round Robin: Distributes requests sequentially to each server in a loop.
     - Least Connections: Directs traffic to the server with the fewest active connections.
     - IP Hash: Assigns requests to servers based on the client's IP address.
     - Failover: If one server goes down, traffic is automatically rerouted to other healthy servers.
 
## Difference Between L4 Network Load Balancer and L7 Application Load Balancer:


|Aspect	|L4 Network Load Balancer	|L7 Application Load Balancer|
|-------|-------------------------|----------------------------|
|OSI Layer	|Layer 4 (Transport Layer: TCP/UDP).	|Layer 7 (Application Layer: HTTP/S, WebSocket).|
|Traffic Handling	|Routes based on IP addresses and ports.	|Routes based on HTTP/S content (URL, headers, cookies).|
|Protocol Awareness	|Protocol-agnostic; handles all TCP/UDP traffic.	|HTTP/S protocol-aware; can inspect and route based on content.|
|Performance	|Highly efficient, lower latency.|	Slightly higher latency due to deeper packet inspection.|
|SSL Termination|	Generally passes encrypted traffic.	|Can handle SSL termination (decrypts HTTPS traffic).|
|Scaling	|Can support both vertical and horizontal scaling but better suited for horizontal scaling due to low overhead.|	Better suited for horizontal scaling where routing decisions depend on specific HTTP/S content.|
|Session Persistence	|Available but limited, requires additional setup.	|Natively supports sticky sessions based on cookies.|
|Use Case|	Non-HTTP traffic (e.g., streaming, gaming, VoIP).|	Web applications, APIs, and microservices where content-based routing is essential.|



## Real-World Application of Scaling and Load Balancing:
- Vertical Scaling in Cloud: On cloud platforms like AWS, EC2 instances can be resized to larger instance types to provide more resources. However, this approach is limited by the maximum instance size available.

- Horizontal Scaling with Auto Scaling: Platforms like AWS or GCP use Auto Scaling with load balancers to automatically add or remove instances based on traffic demand. For example, during high traffic, more instances are added behind an Application Load Balancer (L7), and during off-peak hours, instances are automatically removed.

- Microservices: Applications using microservices architecture typically benefit from horizontal scaling. With a Layer 7 Load Balancer, traffic can be routed to different services based on URLs or headers (e.g., /api/products goes to one service, while /api/users goes to another).









