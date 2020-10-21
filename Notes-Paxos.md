# Title:
Notes on Paxos: A consensus protocol

## Why Consensus:

* Consensus is simply getting a bunch of systems to agree on a particular decision. 
* Consensus can be used for a bunch of things:
    1. Enforce Mutual Exclusion on a resource: Who gets access to a resource.
    2. Agree on an ordering of events: Useful in replicated systems. Provides fault tolerance.
    3. Agree on a leader who is in charge: Elections.

* In fault tolerant systems, consensus ensures the same action is performed across all replicas. 2 design actions for such systems exist:
    1. Single Writer: All writes are sent to a single process in charge of writing. This process needs to be elected by consensus. (LEADER ELECTION)
    2. Multiple Readers: All nodes are capable of performing a read operation. Consensus/Voting has to occur on which node's read operation must be performed at the moment.

* Problem Statement: How to get a bunch of systems to agree on a particular proposed value?

## Paxos:

* Provides consensus on an asynchronous network.
* One or more clients propose a value and a majority of nodes have to agree on the value.
* Paxos provides abortable consensus, i.e a node may abort proposing values if it sees a contention for the value proposed.
* Assumptions in Paxos:
    1. Concurrent proposals: One or more systems may propose a value concurrently.
    2. Validity: The chosen value that is agreed upon must be one of the proposed values.
    3. Majority Rule: For a value to be chosen, majority of the nodes must agree to it. This also implies that there must be atleast 2m+1 nodes, if we need to accommodate m failures.
    4. Asynchronous networks: The network is async, i.e will be lossy and have delays.
    5. Fail-stop faults: Systems may exhibit faults. They may need to restart, but will remember their previous state before failure. Thus, failures aren't Byzantine in nature.
    6. Unicast: Point to point communication.
    7. Announcement: Once a consensus is reached, everybody gets to know about it.

### Fault Tolerance with multiple acceptors:

* A simple architecture for consensus is having multiple proposers, but a single acceptor. In such a case, we will have a single point of failure(acceptor), and no way to recover.

* Design Choices for a multi acceptor system:
    1. If an acceptor simply accepts the first message it receives, without having an ability to change it's mind, we could achieve a scenario where there are no majority values. Thus, acceptors need the ability to change their minds on which accepted value to choose. More importantly, a value must be chosen only when a **majority** of acceptors accept it.
    2. Since, an acceptor can change it's mind about the value chosen, a scenario of 'Value A' being accepted first by majority followed by 'Value B' being accepted by the majority nodes also can exist. In such a case, one acceptor would have changed its mind on the accepted value. We hence enforce, Once a value is **chosen**, there is no going back (changing mind).
    3. To solve #2, we use a 2 phase approach - Check majority acceptors and then propose a value.
    4. Once we accept a proposal, we abort all competing proposals. Paxos imposes an ordering on requests. A newer proposal takes precedence over an older proposal.

## Paxos Internals:

### Components:
* Paxos has 3 components:
    1. Proposers: Receives values from clients and tries to convince acceptors to take in the value.
    2. Acceptors: Accept certain proposals and let the proposers know if something else was accepted. A response from an acceptor means the value was accepted.
    3. Learner: Announce the outcome.

### Operation:

* Client sends a request to a proposer, which runs a 2 phase protocol with the acceptors.
* Requires 2m+1 servers running to tolerate m failures.
* Acceptors need to remember their decisions. They maintain state in disk storages.
* Whenever a proposer needs to propose a value, it creates a proposal number for the proposal. A **proposal number** must have 2 properties:
    1. Must be unique. No two proposals must have the same value.
    2. Proposal number must be monotonically increasing. That is, newer proposal has a higher proposal number than an older one.


### Phases of Paxos:

* Phase 1: 
    * Funda: Proposer asks acceptors if they have already accepted a value. If no, propose a new value.
    * Proposer receives a consensus request(value) from a client. It creates a proposal number for this request and sends a `Prepare` message to a majority of acceptors.
    * Each acceptor that receives the Prepare request needs to perform the following check:
    ```
    Prepare(id):
        if id > prev_id:  # (previously received value)
            prev_id = id
            send(Promise(id))  # Respond with a promise message
        else:
            send(Fail(id))  # Respond with a fail message.

    ```
    * Now, when a proposer receives a `Promise` message, it knows that a majority of acceptors have agreed to participate in consensus for this particular proposal number. Conversely it means that they will not accept any proposal with number lesser than this id.

* Phase 2: 
    * Funda: If majority of acceptors agree to a value, we have achieved consensus.
    * If a proposer receives the `Promise` message from a majority of acceptors, it has to inform the acceptors to accept that proposal. Otherwise it has to propose the same value again, with a newer id. That is, send a newer `Prepare` message.
    * Proposer now sends the message `Propose(id, value)` to all acceptors, and it is up to each acceptor to decide if it wants to accept the message. Following is the check occuring in acceptor's end:
    ```
    Propose(id, val):
        if id == prev_id:  # Is this the highest ID I have seen?
            send(Accepted(id))
            send_to_learner(Accepted(id))
        else:
            send(Fail(id))  # Respond with fail message
    ```
    * Whenever a proposer receives a majority of accept messages, it knows that a consensus on values have been attained.

### Handling Failures:
* Acceptor needs to keep track of what all have been accepted so far. This is to ignore a case where it has first promised one value, following which a higher ordered proposal comes and it sends a promise to that value as well.

```
Prepare(id):
    if id > prev_id:
        prev_id = id
        send(Promise(id))
        proposal_accepted=true
    else:
        if proposal_accepted:
            send(Promise(id, accepted_id, accepted_val))
        else:
            send(Fail(id))
```

* Thus the proposer checks at the highest accepted value from the acceptors, and is obligated to pick the highest accepted proposal value.


## Failure Examples:

* Acceptor fails in phase 1:
    * No promise message is sent. Thus as long as majority of acceptors are still running, consensus can be reached.
* Acceptor fails in phase 2:
    * No accept message is sent. Valid until majority acceptors are running.
* Proposer fails in Accept phase:
    * Since acceptor responds with `Promise(id, accepted_id, accepted_val)`, another proposer can take over and continue with the process.