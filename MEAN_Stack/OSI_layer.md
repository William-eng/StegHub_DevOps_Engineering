## What is OSI Model?
OSI stands for **Open Systems Interconnection** , where open stands to say non-proprietary. It is a 7-layer architecture with each layer having specific functionality to perform. 
All these 7 layers work collaboratively to transmit the data from one person to another across the globe. The OSI reference model was developed by ISO – ‘International Organization for Standardization ‘, 
in the year 1984. The OSI model provides a theoretical foundation for understanding network communication.
However, it is usually not directly implemented in its entirety in real-world networking hardware or software . 
Instead, specific protocols and technologies are often designed based on the principles outlined in the OSI model to facilitate efficient data transmission and networking operations.
Data Flow In OSI Model
When we transfer information from one device to another, it travels through 7 layers of OSI model. First data travels down through 7 layers from the sender’s end and then climbs back 7 layers on the receiver’s end.

## Data flows through the OSI model in a step-by-step process:

- Application Layer: Applications create the data. The application layer is the closest to the user and is responsible for providing network services to end-users. It interacts with software applications to implement communication.
- Presentation Layer: Data is formatted and encrypted. The presentation layer ensures that data is in a usable format and is delivered in a way that the application layer can understand.
- Session Layer: Connections are established and managed. The presentation layer ensures that data is in a usable format and is delivered in a way that the application layer can understand.
- Transport Layer: Data is broken into segments for reliable delivery.
- Network Layer : Segments are packaged into packets and routed.
- Data Link Layer: Packets are framed and sent to the next device.
- Physical Layer: Frames are converted into bits and transmitted physically.
- Each layer adds specific information to ensure the data reaches its destination correctly, and these steps are reversed upon arrival.
  
## How the OSI Model Works in Practice:
When data is transmitted from one device to another, it flows from Layer 7 (Application) down to Layer 1 (Physical) on the sender's side.
Each layer adds a header (sometimes a trailer) to the data. This process is called encapsulation.
The data is then transmitted over the physical medium (Layer 1).
On the receiving device, the data flows from Layer 1 (Physical) back up to Layer 7 (Application). At each layer, the relevant headers are stripped away, leaving the original data.
  

  Ref: https://www.geeksforgeeks.org/open-systems-interconnection-model-osi/
