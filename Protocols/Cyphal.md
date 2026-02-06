# Overview
-  [Cyphal](https://opencyphal.org/) is an open technology for real-time intravehicular distributed computing and communication based on modern networking standards (Ethernet, CAN FD, etc.)
- Cyphal is the underlying protocol, while OpenCyphal is the open-source project that provides the actual implementation.
- Broke off from UAVCAN project - this is v1 of it, while v0 is now called DroneCAN, which is more widely adopted. See [comparison](https://forum.opencyphal.org/t/cyphal-vs-dronecan/1814)
- CAN is the data-link layer of intravehicular comms, while Cyphal is the protocol that takes care of the actual messaging
- Cyphal is transport agnostic, allowing heterogeneous transport redundancy, where multiple physical transports with different failure modes (e.g. both CAN and a wireless link)



### Application Layer
- The applications that developers develop on top of Cyphal, e.g. flight controller for UAV
- All application-layer Cyphal nodes must implement the heartbeat message

### Presentation Layer
- The DSDL (Data Structure Description Language) defines the data types used e.g. `uavcan.node.Heartbeat`
- Some of them are define *registers*, which are essentially configurable parameters for nodes
- Subjects for publishing/subscribing are given a numerical code agreed upon by standard

- Register access is service request and response
- Typical messaging is publish subscribe