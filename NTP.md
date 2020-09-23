# Internet Time Synchronization: Network Time Protocol

## Abstract:

* Designed to distribute time information in a large, diverse internet system operating at speeds ranging from fast to slow.

* Uses a symmetric architecture, in which a distributed subnet of time servers operating in a hierarchical configuration synchronize clocks within the subnet, to national time standards.

* Can also distribute time information within a network using routing protocols and time daemons.

## Introduction:

* Manual methods of time keeping are prone to errors, some to the scale of ~0.5-1 minutes.

* Distributed Internet applications require much greater accuracy and reliability.

* Service reliability and operating speeds vary considerably across the various backbone networks and gateways. This places severe demand on NTP to deliver accurate results inspite of component failures, service disruption and network latency.

## Section 1:
### Definitions:

* **Stability**: How well a clock can maintain a constant frequency.
* **Accuracy**: How well it's time compares to national time standards.
* **Precision**: How precisely time can be resolved in a time keeping system.
* **Offset**: Time difference between two clocks.
* **Skew**: Frequency difference between them.
* **Reliabiltiy**: Fraction of time it can be kept operating and connected in the system.
* **Synchronize Frequency**: Adjust clocks in the same subnet to run at the same frequency.
* **Synchronize Time**: Set them to agree at a particular epoch.
* **Synchronize Clocks**: Synchronize clocks in both frequency and time.

### Performance Requirements:

* Traffic Loads, route selection and facility outage can cause delays and reliability issues in transmission paths.

* Requirements:
    1. Primary reference source must be synchronized to national standards.
    2. Timer servers must provide accurate and precise time, even during large delay variations on the transmission paths.
    3. Synchronization subnet must be reliable and survivable, even under unstable network conditions.
    4. Synchronization protocol must operate continuously and provide update information at rates capable of compensating delays.
    5. Must operate in any system, while making minimal demands to the client OS.
    6. Must include protection against accidental or willful intrusion.

* NTP used address filtering for access control and encryted checksums for authentication.

### NTP Protocol Approach:

* Distributed system of NTP Time servers operate as a hierarchically organized subnet of loosely coupled time servers which exchange periodic update messages containing precision timestamps to update local frequency.

* Summary of the design:
    1. System consists of a hierarchical network of time servers configured on the basis of estimated accuracy, precision and reliability.
    2. Protocol operates in connectionless mode to minimize latencies and simplify implementations.
    3. Uses symmetric design that tolerates packet loss, duplication and mis-ordering.

## Time Standards and Distribution:

* NTP system consists of primary and secondary time servers. 
* Primary server is directly sychronized to a primary reference source (usually an atomic clock or a timecode receiver).
* Secondary servers derive time synchronization from a primary server over network paths, shared with other services.
* Synchronization paths assume a hierarchical structure, so that the most accurate clocks are usually at the upper level, with the less accurate ones at the lower levels.
* **Stratum**: Defines accuracy of each time server.
* Path for a synchronization subnet is calculated by building a minimum spanning tree, using a modified Bellman-Ford Algorithm.

## Network Time Protocol:

* Built on top of IP and UDP. These protocols provide data integrity(through checksums).
* One or more primary servers synchronize to external reference sources such as timecode receivers.

### Determining Time and Frequency:

* Each NTP Message contains the last 3 timestamps, T<sub>i-1</sub>, T<sub>i-2</sub>, T<sub>i-3</sub>, as well as T<sub>i</sub>. Through this, both the server and the peer can individually calculate delay and offset using a single message stream.

### Modes of Operation:

* Operate in one of 3 service classes:
    1. Multicast: 
        * Used on high speed LAN's where highest accuracy aren't required.
        * Server sends periodic NTP broadcast. Peers operating in client mode determine time accounting for a slight delay.
        * Server announces it's willingness to provide synchronization to other peers, but accept NTP messages from none of them.
    2. Procedure call:
        * Used on file servers and workstations requiring highest accuracies.
        * Client sends a message to server which returns the message with it's timestamp. Client accounts for delay and synchronizes its clock.
    3. Symmetric:
        * Distributed participation of a number of time servers arranged in reconfigurable, hierarchically distributed configuration.
        * Two modes: Active and Passive.
        * 

* The 3 classes are differentiated on the following factors:
    1. Number of peers involved.
    2. Synchronization is to be given or received.
    3. State information is retained.

### Procedures:

* Events of interest in NTP occur on:
    1. Expiration of peer timer.
    2. Arrival of an NTP message.
* Transmit procedure is called when a peer timer decrements to zero. Peer timer is then reset and an NTP message is sent including all information.
* Receive procedure is called when the NTP message arrives. It is then matched with the association indicated by its addresses and ports.
* Update procedure is called when a new set of estimates become available.

### Robustness Issues:

* **Timewarp**: Bogus timestamp caused by issues/discrepancies in any dependent part of the synchronization system.
* To improve reliability, a simple reachability protocol is used, wherein a node is marked unreachable if it does not respond to messages/ no messages are received for 8 consecutive poll intervals.
* In active mode, the peer is marked unreachable, but polls continue. In passive mode, the association is dissolved and the resources are reclaimed for subsequent use.
* **Sanity Checks Provided**:
    1. If transmit timestamp of a message is identical to one previously received, the message is a duplicate or contains bogus data.
    2. If a message contains an originate timestamp that differs from the previously transmitted message's transmit timestamp, the message is either out of order or bogus.
    3. Messages are encrypted using DES.

## Filtering, Selection and Combining Operations:

* Key components of NTP are the algorithms used to improve the accuracy of estimated delays and offsets between various servers.

* Algorithms are complex because of they depend on statistical properties of the transmission path, as well as the accuracy and the precision involved.

* NTP Data filtering is designed to provide high accuracy along with low computational burden.

## Local Clock design:

* Precision timekeeping requires a stable local oscillator reference to deliver accurate time when the synchronization path to a primary server has failed.

