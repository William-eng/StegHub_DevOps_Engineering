Traceroute and Ping are network tools or utilities that use the ICMP protocol to perform testing to diagnose issues on a network. 
Internet Control Message Protocol (ICMP) is an error reporting and diagnostic utility.  ICMPs are used by routers, intermediary devices, or hosts to communicate updates or error 
information to other routers, intermediary devices, or hosts.

## Ping
Ping is a network utility used to test the reachability of a host on an IP network and measure the round-trip time for messages sent from the originating host to a destination computer. 
It uses Internet Control Message Protocol (ICMP) Echo Request messages to elicit an ICMP Echo Reply from the target host.

### How Ping Works:
- When you run ping, the utility sends a series of ICMP echo request packets to a target IP address or hostname.
- The target responds with ICMP echo reply packets.
- Ping calculates the round-trip time (RTT) for each packet and reports the results.

 ### Common Ping Syntax:

      ping google.com

 ### Understanding Ping Output:

    Pinging google.com [142.250.182.142] with 32 bytes of data:
    Reply from 142.250.182.142: bytes=32 time=14ms TTL=55
    Reply from 142.250.182.142: bytes=32 time=13ms TTL=55
    Reply from 142.250.182.142: bytes=32 time=12ms TTL=55
    
    Ping statistics for 142.250.182.142:
        Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
    Approximate round trip times in milli-seconds:
        Minimum = 12ms, Maximum = 14ms, Average = 13ms

        Bytes: Size of each packet sent (typically 32 bytes by default).

        
- Time: The round-trip time (RTT) it took for the packet to travel to the destination and back, measured in milliseconds.
- TTL (Time to Live): Indicates the remaining lifespan of the packet in the network. Each hop (intermediate device) decreases the TTL by 1.
- Packet Loss: Reports how many packets were sent and received, along with any packet loss (0% loss is ideal).
  
## Traceroute
Tracerouteis a network diagnostic tool used to track the pathway that a packet takes from the source to the destination. It records the route (the specific gateway computers at each hop) 
through the Internet between your computer and a specified destination computer. It also calculates and displays the time taken for each hop

### How Traceroute Works:
- Traceroute sends packets with incrementally increasing TTL values.
- Each time a packet reaches a hop, it decreases the TTL by 1. When TTL reaches 0, the hop returns an ICMP "time exceeded" message, revealing its IP address.
- The process continues until the destination is reached, showing the path and timing of each hop.

### Common Traceroute Syntax:
### On Linux/macOS:

    traceroute google.com
### On Windows:

    tracert google.com
### Understanding Traceroute Output:

    traceroute to google.com (142.250.182.142), 30 hops max
    1  192.168.1.1 (192.168.1.1)  1.532 ms  1.212 ms  1.121 ms
    2  10.0.0.1 (10.0.0.1)  3.223 ms  2.982 ms  2.937 ms
    3  203.0.113.5 (203.0.113.5)  4.120 ms  4.015 ms  4.001 ms
    4  142.250.182.142 (142.250.182.142)  14.354 ms  14.242 ms  14.128 ms

- Hops: Each row corresponds to a hop (router) along the route to the destination.
- IP Address: The IP address (or hostname) of each hop.
- Round-Trip Time: Three time values (in milliseconds) are shown for each hop, representing the time for three packets to travel to that hop and back. Consistency across these values is ideal.
