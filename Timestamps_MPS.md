# Timestamps in Message Passing Systems that preserve Partial ordering

## Abstract:

* Total ordering is inappropriate for applications requiring access to a global state.

* This paper presents methods to timestamp in a synchronous and asynchronous message passing systems that allow access to partial ordering inherent in a parallel system.

## Introduction:

* **Problem**: Determining order in which events occurred is a challenge. Attaching a timestamp to each event assumes that each process can access an accurate clock, but parallel scalable systems (by their very nature) make it difficult to ensure consistency among processes.

* One solution is to have a central timestamping server, which assigns timestamps to each event in every process. While we standardize the time issuing activity, the downside is that we need extensive communication links.

* Another solution is to have clocks in each process, while constantly synchronizing the clocks, thereby ensuring the timestamps represent a possible ordering of events.

* Paper presents an implementation of the partially ordered relation - happened before. The algorithm is first defined for an asynchronous case and then extended to a synchronous message passing system.

## Total Ordering: (Defined by Lamport)

* Each process maintains an integer value, which it periodically increments (eg. once after every atomic event). This value is assigned to each event as the timestamp. 

* This gives off an impression that the system is recording each event, either centrally or by each process.

* The local clocks eventually will go out of sync. In such events, we need to find a mechanism to bring them back to synchronization.

* We piggyback the current local time along with each message(outgoing signal), and the receiving process has to update it's current time to be greater than this value.

* This ensures consistency in timing, since departure of a signal always precedes the arrival.

* In case two events belonging to different processes are assigned the same timestamp such that (a!=>b) and (b!=>a), then we arbitrarily order the events.

## Problem Statement:

* This algorithm struggles to deal with events that could occur concurrently. 

* We aim to implement a happened-before relation, where if a->b then a causes b. If a!->b and b!->a,, then either a->b or b->a and they could be concurrent processes.

## Partial Ordering:

* Key aspect of the algorithm is that communication events form boundaries that limit the possible interleavings of concurrent events. (check notes for illustration)

* It is necessary for each process to know when it last communicated with other processs.

## Asynchronous communication:

* Each event e has an array of timestamps associated with it, with the i<sup>th</sup> index in the array representing the timestamp of the i<sup>th</sup> process during the execution of the event.

* Following are the rules for maintaining the timestamp array:
    1. Initially all values are zero. <br/>
    2. The local clock value is incremented at least once before each atomic event. <br/>
    3. The current value of the entire timestamp is piggybacked on every outgoing message. <br/>
    4. Upon receiving the signal, the process sets the value of each index of the timestamp array to be the maximum of the two corresponding elements in the local array and the piggybacked array. The value corresponding the the sender however, is set to one greater than the value received, to allow for transit time. This is done only if the local value is not greater than that received. <br/>
    5. Values in the timestamp array are never decremented. <br/>

* Thus each process receives updates about the clocks in other processes, including ones which aren't it's neighbours.

* We construct temporal relationships e<sub>p</sub> --> f<sub>q</sub> if T<sub>e<sub>p</sub></sub>[p] < T<sub>f<sub>q</sub></sub>[q] among the processes.


## Synchronous communication: 

* If the Lamport method is directly applied to a system using synchronous communication, it will fail, since Lamport's algorithm expects communication to take a finite amount of time.  (check the diagram)

* We can resolve this by enforcing bidirectional communication, with both the processes exchanging their timestamps, and then each setting their clocks to the highest value. If all processes adhere to this, we can avoid deadlocks occuring.

* Timestamp arrays are maintained according to the following rules:
    1. Initially all clock values are zero. <br/>
    2. The local clock value is incremented once before each atomic event. <br/>
    3. During the communication event, both the processes involved exchange timestamps and set their local copy of the timestamp to the maximum of the old value and the corresponding array value received. <br/>
    4. Values in the timestamp array are never decremented. <br/>

* Temporal relationships are constructed as follows: e<sub>p</sub> --> f<sub>q</sub> if T<sub>e<sub>p</sub></sub>[p] <= T<sub>f<sub>q</sub></sub>[p] AND T<sub>e<sub>p</sub></sub>[q] < T<sub>f<sub>q</sub></sub>[q]

* The first part of this asserts that process q has received a clock value from process p at least as recent as the execution of event e<sub>p</sub>. Hence, f<sub>q</sub> must have been executed after e<sub>p</sub>.

* The second part asserts that p does not have necessary information about process q.

## Applications:

* Implementation of the relationship depicted here allows timestamps to be attached to state snapshots saved independently by several processes, making it possible to check which states form a valid consistent slice of the global program state.

* Used in error recovery / rollback.

## Conclusion:

* Used for timestamping events in a distributed system that define an order between two events when their temporal relationship is unambiguously defined by inter-process communications.

* Don't change the communication graph, neither introduce any extra communication events.