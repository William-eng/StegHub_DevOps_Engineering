# Network-Attached Storage (NAS)
NAS is a file-level storage architecture that allows multiple clients to access shared storage over a network.  It functions like a dedicated file server, providing centralized storage 
accessible via standard network protocols.

## Key Characteristics of NAS:
- File-Level Storage: NAS stores and manages files, making it easy to access data from multiple devices. It operates at the file level, meaning users interact with the data as individual files.
  
- Uses TCP/IP Networks: NAS connects to a LAN (Local Area Network) via Ethernet, which allows it to be accessed by multiple clients over standard network protocols.
  
- Accessible via File Sharing Protocols: Common protocols for file access on NAS include:
  - NFS (Network File System): Primarily used in Unix/Linux environments.
  - SMB/CIFS (Server Message Block/Common Internet File System): Typically used in Windows environments.

## Use Cases of NAS
- Home media servers
- File sharing for small to medium businesses
- Backups and archives

# Storage Area Network (SAN)
  SAN is a high-performance, block-level storage architecture designed to provide storage to servers over a dedicated, high-speed network.
  It differs from NAS in that SANs typically focus on block-level storage for applications such as databases and virtual machines.
## Key Characteristics of SAN:
- Block-Level Storage: SAN provides raw block storage to servers, allowing for flexibility in how the storage is formatted (e.g., for databases or virtual machines).
- Uses Fibre Channel or iSCSI: SANs often use high-speed Fibre Channel networks, though iSCSI (Internet Small Computer Systems Interface) can be used over Ethernet as a lower-cost alternative.
- Requires Dedicated Network: Unlike NAS, SAN typically requires a dedicated network for storage traffic, separate from the normal LAN.

## Use Cases of SAN:
- Large databases
- Virtualized environments
- High-performance applications in enterprise data centers

## Comparison Between NAS and SAN

| Features| NAS | SAN|
|---------|------|---|
|Storage Typical ProtocolsType|File-level storage|Block-level storage|
|Typical Protocols|NFS, SMB/CIFS|iSCSI, Fibre Channel|
|Use Case|File sharing, backups, archives|High-performance applications|
|Network|Uses existing Ethernet/LAN|Dedicated storage network|
|Performance|Moderate|High|
|Cost|Lower|Higher|

# Related Protocols
## 1. NFS (Network File System)
NFS is a distributed file system protocol developed by Sun Microsystems, enabling users to access files over a network as if they were on a local disk.
Common Use: Primarily used in Unix/Linux environments, NFS allows for sharing directories and files between machines on a network.
Advantages:
Easy integration in Unix/Linux environments.
Lightweight and efficient for sharing files over a LAN.

## 2. SMB/CIFS (Server Message Block/Common Internet File System)
SMB is a network protocol used for sharing files, printers, and serial ports between computers. CIFS is a dialect of SMB and was primarily used in older versions of Windows.
Common Use: Primarily used in Windows environments to provide shared access to files and resources.
Advantages:
Widely supported on Windows.
Integrated with Active Directory for permissions and authentication.

## 3. (S)FTP (Secure File Transfer Protocol)
FTP is a standard protocol for transferring files between a client and server over a network. SFTP (Secure FTP) adds encryption to secure the file transfer process.
Common Use: Used to upload/download files between systems, often for website management, backups, and secure data transfer.
Advantages:
SFTP provides encryption, improving security over traditional FTP.
Widely supported by file transfer tools.

## 4. iSCSI (Internet Small Computer Systems Interface)
iSCSI is a protocol that allows SCSI commands to be sent over IP networks. It is used to connect data storage facilities like SANs over Ethernet.
Common Use: Often used in SAN environments to provide block-level storage over an existing Ethernet infrastructure.
Advantages:
Lower cost compared to Fibre Channel SANs.
Allows for block-level storage over IP networks, making SANs more accessible.


