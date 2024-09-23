# load balancing
Load balancing is a crucial technique in networking and distributed systems, designed to distribute incoming network traffic across multiple servers, services, or resources.
The primary goal is to improve application availability,  fault tolerance, and scalability by ensuring no single server is overwhelmed by traffic, leading to better performance and reliability.
The simple answer to “What is load balancing?” or even “What is server load balancing?” is this: load balancing is about troubleshooting the distribution of inbound network and application 
traffic across multiple servers. With hundreds of user (or client) requests coming in every minute, it’s hard for any one server to keep up and continually display high-quality photos, videos, 
text, and application data at the speed at which many users have become accustomed. Lags are considered “unacceptable” while complete downtime is intolerable.
Load balancing techniques essentially serve as the director on a big-time movie set. They direct application and network traffic to specific servers within the “server farm” or “server pool.” 
This helps prevent any one server from carrying too heavy a load, thereby optimizing application and network availability and responsiveness.

## What Load Balancer Types Exist?
There are a number of specific types of load balancing you might need to consider for your network, including SQL Server load balancing for your relational database, global server load balancing 
for troubleshooting across multiple geographic locations, and DNS server load balancing to ensure domain name functionality. You can also think about types of load balancers in terms of the
various cloud-based balancers available (including the well-known AWS Elastic Load Balancer): 

### - Network Load Balancing: Network load balancing, as its name suggests, leverages network layer information to decide where to send network traffic.
This is accomplished through layer 4 load balancing, which is designed to handle all forms of TCP/UDP traffic. Network load balancing is considered the fastest of all the load balancing solutions,
but it tends to fall short when it comes to balancing the distribution of traffic across servers.

### - HTTP(S) Load Balancing: HTTP(S) load balancing is one of the oldest forms of load balancing.
This form of load balancing relies on layer 7, which means it operates in the application layer. HTTP load balancing is often dubbed the most flexible type of load balancing because 
it allows you to form distribution decisions based on any information that comes with an HTTP address.

### - Internal Load Balancing: Internal load balancing is nearly identical to network load balancing but can be leveraged to balance internal infrastructure.
When talking about types of load balancers, it’s also important to note there are 
### hardware load balancers, software load balancers, and virtual load balancers.

### Hardware Load Balancer: 
A hardware load balancer, as the name implies, relies on physical, on-premises hardware to distribute application and network traffic. 
These devices can handle a large volume of traffic but often carry a hefty price tag and are fairly limited in terms of flexibility.
### Software Load Balancer: 
A software load balancer comes in two forms—commercial or open-source—and must be installed prior to use. Like cloud-based balancers,
these tend to be more affordable than hardware solutions.

## Virtual Load Balancer: 
A virtual load balancer differs from software load balancers because it deploys the software of a hardware load balancing device on a virtual machine.

## Load Balancing Algorithms/Techniques:

### Round Robin:
 Distributes traffic sequentially, sending each new request to the next server in the list.
Use Case: Simple and effective for equally performing servers.
Drawback: Doesn’t account for differences in server performance or current load.

### Least Connections:
Directs traffic to the server with the fewest active connections.
Use Case: Works well when traffic patterns vary and some requests take longer than others.
Drawback: Doesn’t always account for the computational intensity of individual requests.

### Least Response Time:
Routes traffic to the server that has the fewest active connections and the lowest response time.
Use Case: Useful when both the number of active connections and the server's speed matter for performance.

### Weighted Round Robin:
Similar to round-robin but assigns a weight to each server based on capacity, directing more traffic to more powerful servers.
Use Case: Ideal for environments where servers have varying resources (CPU, memory).

### Weighted Least Connections:
A variation of the Least Connections algorithm, where each server is assigned a weight and the load balancer directs traffic to the server with the fewest connections relative to its weight.
Use Case: Useful for environments with servers of unequal power.

### IP Hash:
Determines which server to forward a request to based on the hash of the client’s IP address.
Use Case: Helps in session persistence, ensuring that a user’s requests go to the same server.
Drawback: Doesn’t dynamically account for server health or load.

### Geographic Load Balancing (Geo-Load Balancing):
Directs traffic to the server closest to the user’s geographic location, improving latency and speed.
Use Case: Ideal for globally distributed services where users are spread across multiple regions.

### Random:
Distributes traffic randomly among the available servers.
Use Case: Can be useful when the system is homogenous and server performance is identical.

### Source IP Affinity (Sticky Sessions):
Ensures that traffic from a particular client (based on their IP address) is always directed to the same server.
Use Case: Critical for session-based applications where keeping a user on the same server ensures consistency.
Drawback: Doesn’t handle dynamic server changes or load effectively.

### Priority Load Balancing:
Servers are prioritized, and requests are sent to the highest-priority server first. Only if the top-priority server becomes overloaded will requests be sent to lower-priority servers.
Use Case: Useful when specific servers are designed to handle the bulk of the traffic, with others as backups.

Ref: https://www.dnsstuff.com/what-is-server-load-balancing
