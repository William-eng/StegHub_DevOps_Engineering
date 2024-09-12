TCP and UDP are used to connect two devices over the Internet or other networks. However, to give data packages an entrance to the PC or server at the other end of the connection, the “doors” have to be open. These openings into the system are called ports. 
The transmission control protocol (TCP) is defined as a connection-oriented communication protocol that allows computing devices and applications to send data via a network and verify its delivery, forming one of the crucial pillars of the global internet. User datagram protocol (UDP) is a message-oriented communication protocol that allows computing devices and applications to send data via a network without verifying its delivery, which is best suited to real-time communication and broadcast systems. 

|Port number | Service name    | Transport protocol | Description                                                                              |
|------------|-----------------|--------------------|------------------------------------------------------------------------------------------|
|7           |  Echo           |TCP, UDP            |Echo service                                                                              |
|                                                                                                                                              |
|19          |CHARGEN          |TCP, UDP             |Character Generator Protocol, has severe vulnerabilities and thus is rarely used nowadays |
20           | FTP-data        |TCP, SCTP               |File Transfer Protocol data transfer
21 | FTP |TCP, UDP, SCTP |File Transfer Protocol command control
22 | SSH/SCP/SFTP | TCP, UDP, SCTP | Secure Shell, secure logins, file transfers (scp, sftp), and port forwarding
23 | Telnet | TCP |Telnet protocol, for unencrypted text communications
25 | SMTP | TCP | Simple Mail Transfer Protocol, used for email routing between mail servers
42 | WINS Replication       | TCP, UDP | Microsoft Windows Internet Name Service, vulnerable to attacks on a local network
43 | WHOIS | TCP, UDP |Whois service, provides domain-level information
49 |TACACS | UDP; can also use TCP but not necessarily on port 49 | Terminal Access Controller Access-Control System, provides remote authentication and related services for network access
53 | DNS | TCP, UDP  | Domain Name System name resolver
67 | DHCP/BOOTP | UDP | Dynamic Host Configuration Protocol and its predecessor Bootstrap Protocol Server; server port
68 | DHCP/BOOTP | UDP | Dynamic Host Configuration Protocol and its predecessor Bootstrap Protocol Server; client port
69 | TFTP | UDP | Trivial File Transfer Protocol
70 | Gopher | TCP |Gopher is a communication protocol for distributing, searching, and retrieving documents in Internet Protocol (IP) networks
79 | Finger| TCP | Name/Finger protocol and Finger user information protocol, for retrieving and manipulating user information
80 | HTTP | TCP, UDP, SCTP | Hypertext Transfer Protocol (HTTP) uses TCP in versions 1.x and 2. HTTP/3 uses QUIC, a transport protocol on top of UDP
88 | Kerberos | TCP, UDP | Network authentication system 
102 | Microsoft Exchange ISO-TSAP | TCP | Microsoft Exchange ISO Transport Service Access Point (TSAP) Class 0 protocol
110 | POP3| TCP |Post Office Protocol, version 3 (POP3)
113 |Ident| TCP| Identification Protocol, for identifying the user of a particular TCP connection
119| NNTP (Usenet) | TCP |Network News Transfer Protocol
123 |NTP |UDP |Network Time Protocol 
| 135| Microsoft RPC EPMAP | TCP, UDP| Microsoft Remote Procedure Call (RPC) Endpoint Mapper (EPMAP) service, for remote system access and management
137 | NetBIOS-ns| TCP, UDP| NetBIOS Name Service, used for name registration and resolution
138| NetBIOS-dgm| TCP, UDP | NetBIOS Datagram Service, used for providing access to shared resources
139 | NetBIOS-ssn | TCP, UDP | NetBIOS Session Service
143 | IMAP | TCP, UDP | Internet Message Access Protocol (IMAP), management of electronic mail messages on a server
161 | SNMP-agents (unencrypted) | UDP |Simple network management protocol; agents communicate on this port
162|SNMP-trap (unencrypted) |UDP |Simple network management protocol; listens for asynchronous traps
177 |XDMCP |UDP |X Display Manager Control Protocol
179 |BGP|TCP|Border Gateway Protocol
194 |IRC|UDP |Internet Relay Chat
201 |AppleTalk | TCP, UDP | AppleTalk Routing Maintenance. Trojan horses and computer viruses have used UDP port 201.
264| BGMP| TCP, UDP |Border Gateway Multicast Protocol
318| TSP TCP, UDP |Time Stamp Protocol
381| HP Openview |TCP, UDP| HP performance data collector
383 |HP Openview |TCP, UDP |HP data alarm manager
389| LDAP TCP, UDP| Lightweight directory access protocol
411 |(Multiple uses) |TCP, UDP |Direct Connect Hub,| Remote MT Protocol
412| (Multiple uses) | TCP, UDP |Direct Connect Client-to-Client,| Trap Convention Port
427| vSLP |TCP |Service Location Protocol
443 |HTTPS (HTTP over SSL) |TCP, UDP, |SCTP Hypertext Transfer Protocol Secure (HTTPS) uses TCP in versions 1.x and 2. HTTP/3 uses QUIC, a transport protocol on top of UDP.
445 |Microsoft DS SMB |TCP, UDP Microsoft Directory Services: TCP for Active Directory, Windows shares; UDP for Server Message Block (SMB) file-sharing


