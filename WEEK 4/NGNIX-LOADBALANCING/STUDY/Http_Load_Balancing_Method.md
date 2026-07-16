Load balancing across multiple application instances is a commonly used technique for optimizing resource utilization, 
maximizing throughput, reducing latency, and ensuring fault‑tolerant configurations.


## Proxying HTTP Traffic to a Group of Servers
To start using NGINX Plus or NGINX Open Source to load balance HTTP traffic to a group of servers,
first you need to define the group with the upstream directive. The directive is placed in the http context.
Servers in the group are configured using the server directive (not to be confused with the server block that defines a virtual server running on 
NGINX). For example, the following configuration defines a group named **myproject** and consists of three server configurations (which may resolve in 
more than three actual servers):

            
      http {
          upstream myproject {
              server myproject1.example.com weight=5;
              server myproject2.example.com;
              server 192.0.0.1 backup;
          }
      }

To pass requests to a server group, the name of the group is specified in the proxy_pass directive (or the fastcgi_pass, memcached_pass, scgi_pass,
or uwsgi_pass directives for those protocols.) In the next example, a virtual server running on NGINX passes all requests to the **myproject** upstream 
group defined in the previous example:


        server {
            location / {
                proxy_pass http://myproject;
            }
        }

The following example combines the two snippets above and shows how to proxy HTTP requests to the backend server group. The group consists 
of three servers, two of them running instances of the same application while the third is a backup server. Because no load‑balancing algorithm is
specified in the upstream block, NGINX uses the default algorithm, Round Robin:


            http {
                upstream myproject {
                    server myproject1.example.com;
                    server myproject2.example.com;
                    server 192.0.0.1 backup;
                }
            
                server {
                    location / {
                        proxy_pass http://myproject;
                    }
                }
            }
## Choosing a Load-Balancing Method
NGINX Open Source supports four load‑balancing methods, and NGINX Plus adds two more methods:

- Round Robin – Requests are distributed evenly across the servers, with server weights taken into consideration. This method is used by default
   (there is no directive for enabling it):

 
        upstream myproject {
           # no load balancing method is specified for Round Robin
           server backend1.example.com;
           server backend2.example.com;
        }
- Least Connections – A request is sent to the server with the least number of active connections, again with server weights taken into consideration:

 
        upstream myproject {
            least_conn;
            server myproject1.example.com;
            server myproject1.example.com;
        }
- IP Hash – The server to which a request is sent is determined from the client IP address. In this case, either the first three octets of the IPv4
   address or the whole IPv6 address are used to calculate the hash value. The method guarantees that requests from the same address get to the
  same server unless it is not available.

        upstream myproject {
            ip_hash;
            server myproject1.example.com;
            server myproject2.example.com;
        }
If one of the servers needs to be temporarily removed from the load‑balancing rotation, it can be marked with the down parameter in order to preserve the current hashing of client IP addresses. Requests that were to be processed by this server are automatically sent to the next server in the group:

 
          upstream myproject {
              server myproject1.example.com;
              server myproject2.example.com;
              server myproject3.example.com down;
          }
- Generic Hash – The server to which a request is sent is determined from a user‑defined key which can be a text string, variable, or a combination
  . For example, the key may be a paired source IP address and port, or a URI as in this example:

         
        upstream myproject {
            hash $request_uri consistent;
            server myproject1.example.com;
            server myproject2.example.com;
        }
  
The optional consistent parameter to the hash directive enables ketama consistent‑hash load balancing. Requests are evenly distributed across all 
upstream servers based on the user‑defined hashed key value. If an upstream server is added to or removed from an upstream group, only a few keys 
are remapped which minimizes cache misses in the case of load‑balancing cache servers or other applications that accumulate state.

- Least Time (NGINX Plus only) – For each request, NGINX Plus selects the server with the lowest average latency and the lowest number of active
  connections, where the lowest average latency is calculated based on which of the following parameters to the least_time directive is included:

header – Time to receive the first byte from the server
last_byte – Time to receive the full response from the server
last_byte inflight – Time to receive the full response from the server, taking into account incomplete requests
 
          upstream myproject {
              least_time header;
              server myproject1.example.com;
              server myproject2.example.com;
          }
Random – Each request will be passed to a randomly selected server. If the two parameter is specified, first, NGINX randomly selects two servers
taking into account server weights, and then chooses one of these servers using the specified method:

least_conn – The least number of active connections
least_time=header (NGINX Plus) – The least average time to receive the response header from the server ($upstream_header_time)
least_time=last_byte (NGINX Plus) – The least average time to receive the full response from the server ($upstream_response_time)
 
      upstream myproject {
          random two least_time=last_byte;
          server myproject1.example.com;
          server myproject2.example.com;
          server myproject3.example.com;
          server myproject4.example.com;
      }



























































