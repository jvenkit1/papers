# Time, Clocks and Ordering of Events in a Distributed System

## Abstract:

* One event occuring before another is shown to define a partial ordering of events.
* A Distributed Algorithm for synchronizing a system of logical clocks is given.
* A method for solving the synchronization problems in clocks is also provided.
* This algorithm is then specialized for physical clocks and a bound is derived on how far out of sync can the clocks become.

## Introduction:

* **Distributed System**: Collection of distinct processes that are spatially separated and communicate with each other using message passing.

* Any system is distributed if the message transmission delay is not negligible compared to the time between events in a single process.

## Partial Ordering:

* A Happened-Before relation can be used to order events in a process. That is, event A happened before event B, if their physical timestamps are in sequence.

* This observation however is made based on the actual physical values of time and hence requires physical clocks to exist, to define physical timestamps.

* Even if there are real clocks, the ordering is not guaranteed to be completely accuracy, since clocks may not be precise.

* Assumption: Events of a process form a sequence such that 'a' occurs before 'b' in this sequence, if event 'a' occurs before event 'b'.

* **Happened-Before** : Relation denoted by '->' which satisfies the following conditions: <br/>
    1. If 'a' and 'b' are events in the same process, and 'a' comes before 'b', then ```a->b```.
    2. If 'a' denotes sending of a message and 'b' denotes receipt of the same message, then ```a->b```.
    3. If ```a->b``` and ```b->c```, then ```a->c```.

* Two events 'a' and 'b' are said to be concurrent if ```a !->  b``` and ```b !-> a```.

* ```a->b``` also implies that it is possible for event 'a' to causally affects event 'b'.

* Two events are concurrent if neither can causally affect the other.

## Logical Clocks:

* Consider a clock function that assigns a numerical value {C(a)} to each event 'a' in process P<sub>j</sub>.

* This clock function doesn't have to be related to the physical time at all. It could be something as trivial as a counter mechanism and is just a means of ordering.

* **Clock Condition**: For any events a, b: if a -> b, then C(a) < C(b). <br/>

* **Note**: Converse isn't true, as it implies two concurrent events occur at the same time. {Check diagram from source}

* There are two conditions to be met for the clock condition to hold: <br/>
    1. If a, b belong to Process P<sub>i</sub> and 'a' comes before 'b', then C<sub>i</sub>(a) < C<sub>i</sub>(b).
    2. If a denotes sending a message from process P<sub>i</sub> and 'b' denotes receipt of the message at process P<sub>j</sub>, then C<sub>i</sub>(a) < C<sub>j</sub>(b).

* Implementation Rule: <br/>
    1. Each process P<sub>i</sub> increments C<sub>i</sub> between any two successive events
    2. a) If event 'a' denotes sending a message 'm' by process P<sub>i</sub>, then the message m contains a timestamp T<sub>m</sub> = C<sub>i</sub>(a). <br/>
    b) Upon receiving 'm', process P<sub>j</sub> sets value of C<sub>j</sub> greater than or equal to its present value AND greater than T<sub>m</sub>. <br/>

## Ordering the Events Totally:

* Define a relation '=>' such that ```a=>b``` ('a' precedes 'b') if and only if:
    1. C<sub>i</sub>(a) < C<sub>j</sub>(b) OR
    2. C<sub>i</sub>(a) = C<sub>j</sub>(b) and P<sub>i</sub> < P<sub>j</sub> <br/>

* This relation defines total ordering, and if ```a->b```, then ```a=>b``` [From Clock condition]. P<sub>i</sub> is an arbitratry condition to decide ordering.

* **NOTE**: The ordering '=>' is not unique and depends on the clocks used and the arbitrary function P<sub>i</sub> used.

* Consider a mutual exclusion problem with 3 conditions: <br/>
    1. If a process gets a resource, it must eventually release it before the resource can be given to other processes. <br/>
    2. Different requests for the resource must be granted in the order they are made. <br/>
    3. If every process requesting a resource eventually releases the resource, then all requests are eventually granted. <br/>

* **Assumption**: <br/>
    1. For any two processes P<sub>i</sub> and P<sub>j</sub>, messages sent from P<sub>i</sub> to P<sub>j</sub> are received in the same order they are sent. <br/>
    2. Every message is eventually received. <br/>
    3. Every process can send messages to every other process. <br/>

* We use the **Implementation Rules** to define a total ordering '=>' of all events.

* **Algorithm**:
    1. To request a resource, process P<sub>i</sub> sends a message ```T<sub>m</sub>:P<sub>i</sub> requests resource``` to every other process, and puts that message on it's request queue. <br/>
    2. When process P<sub>j</sub> receives this message, it puts the message on it's request queue and sends an ack. <br/>
    3. To release a resource, P<sub>i</sub> removes any ```T<sub>m</sub>:P<sub>i</sub> request resource``` message from it's request queue and sends a message ```T<sub>m</sub>:P<sub>i</sub> release resource``` to all other processes. <br/>
    4. When process P<sub>j</sub> receives this message, it removes all the request message for P<sub>i</sub> from it's request queue and sends an ack. <br/>
    5. Process P<sub>i</sub> is given the resource if the following 2 conditions are met: <br/>
        a) There is a ```T<sub>m</sub>:P<sub>i</sub> request resource``` in it's request queue which is ordered before any other request in the queue. We use the relation '=>' to maintain this ordering. <br/>
        b) For such a message in the queue, there has been an ack from all other processes with timestamp T<sub>resp</sub> greater than the message timestamp T<sub>m</sub>.<br/>

* **Verifying the algorithm**: <br/>
    * Rule 5.b) coupled with the assumption that messages are received in order guarantees that P<sub>i</sub> has learned about all the messages which preceeded it's current request.
    * Rules 3, 4 ensure that condition 1 holds.
    * Rules 3, 4 also ensure that Rule 5.a) holds which ensures condition 3.
    * Since total ordering '=>' extends partial ordering '->', condition holds.

* This algorithm can be used to attain a any synchronization for a distributed multiprocess system.

* Perhaps the biggest **shortcoming** of this approach is, Failure of a single process results in the entire algorithm failing.<br/>


## Anomalous Behavior:

* Consider a situation in a distributed system, where a Process P<sub>i</sub> sends a message M<sub>i</sub> to process P<sub>k</sub> on a far off node. Another process P<sub>j</sub> submits a message M<sub>j</sub> to the same process P<sub>k</sub> after P<sub>i</sub> sends the message.

* Clearly M<sub>i</sub> should be ordered before M<sub>j</sub>. However, if the message M<sub>i</sub> faces some congestion in the network and is delayed resulting in M<sub>j</sub> reaching P<sub>k</sub> first, the ordering will be different from what is expected, violating the planned behavior.

* 2 approaches are suggested for resolving this: <br/>
    1. Timestamp T<sub>j</sub> associated with M<sub>j</sub> could be greater than T<sub>i</sub> causing the ordering to be exactly what is expected. <br/>
    2. Add the following clock condition to the system of clocks: <br/>
        **Stronger Clock condition**: For any events a, b in Set S, if ```a=>b``` then C(a) < C(b).

## Physical Clocks:

* Assume clocks run in continuous intervals, rather than discrete ticks. C<sub>i</sub>(t) represents the reading of a physical clock at time 't'. dC<sub>i</sub>(t)/dt represents the rate at which the clock runs at time 't'.

* All clocks must run at approximately the same correct rate. That is dC<sub>i</sub>/dt~=1.

* **Condition 1**: There exists a constant k<<1, such that, for all i : dC<sub>i</sub>(t)/dt - 1 < k. <br/>

* It is also important for all the clocks to have similar/less amount of delays. That is, C<sub>i</sub>(t) ~= C<sub>j</sub>(t).

* **Condition 2**: For all i, j: C<sub>i</sub>(t) - C<sub>j</sub>(t) < e. <br/>

* New implementation rules for clocks:
    1. For each i, if P<sub>i</sub> does not receive a message at time 't', then C<sub>i</sub> is differentiable at t and dC<sub>i</sub>(t)/dt > 0. <br/>
    2. a) If P<sub>i</sub> sends a message 'm' at time 't', then the message contains a timestamp T<sub>m</sub> = C<sub>i</sub>(t). <br/>
    b) Upon receiving a message 'm' at time t<sub>1</sub>, process P<sub>j</sub> sets C<sub>j</sub>(t<sub>1</sub>) equal to a maximum of (C<sub>j</sub>(t<sub>1</sub> - 0), T<sub>m</sub> + u<sub>m</sub>) <br/>

## Conclusion:

* Described concept of happened-before along with a partial ordering of the events in the system.

* Extended the partial ordering into a total ordering.

* Total ordering can still show anomalous behavior if there is a disagreement with the time chosen by the users. This can be solved by using physical clocks.

* Describe a theorem to synchronize clocks.