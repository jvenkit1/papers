# Large Scale Cluster Management using Borg

## Introduction:

### WHAT:
Borg is a cluster manager that can run multiple applications across several clusters, where each cluster may have over thousand machines.


### WHY BORG?:

Borg provides the following features:
* High Availability
* Fault Tolerance and minimal fault recovery time
* Smart Scheduling that reduce probability of correlated failure.

Borg also offers various HLL features:
* Declarative job specification language
* Name service integration
* Real-time job monitoring


Benefits provided by Borg:
1. Hides details of resource management and failure handling. <br>
-- This means the user can create & deploy applications without knowing how the infrastructure is managing resources for these applications.
2. Operates with high availability and reliability.
3. Runs workloads across several (eg. 10000) machines effectively.


### Terminologies:

**Borg Job**: Users work is called as Job. Each job may contain one or more tasks. <br>
**Borg Cell**: Set of nodes/machines that are considered as one unit. Each job is run in one cell.<br>
**Borg Cluster**: Machines in a cell belong to a single cluster. Lives inside a single datacenter<br>
**Borg Alloc**: Shorthand for Borg Allocation, an alloc is a reserved set of resources on a machine in which one or more tasks can be run. The resources will remain assigned, regardless of whether they are used or not.<br>
**Priority**: Attributed to every job, this is a small positive integer. Higher priority jobs preempt the lower priority ones for resources.<br>
**Quota**: A vector of resource quantities (RAM, CPU, disk etc), this is used to decide which jobs to admit for scheduling.  Analogous to resource limits in Kubernetes.<br>
**Borg Name Service(BNS)**: Borg's internal resource name resolution service.<br>
**Borgmaster**: Centrallized controller for a Borg Cell.<br>
**Borglet**: Local Agent present in every machine in a cell.<br>
**Checkpoint**: Borgmaster's state at a point of time is called as Checkpoint. This is a periodic snapshot and is used to build a persistent log. <br>
**Fauxmaster**: A high fidelity Borgmaster simulator, it is used to read checkpoint files. It contains <br>
**Task Startup latency**: Time from job submission to job starting. Median typically lies at 25s.
**Recovery Storm**: <br>
**Equivalence Class**: Group of tasks that have similar resource requirements. <br>
**Cell Compaction**: This is an advanced evaluation metric that determines, for a given workload, how small a cell can this be fit into. <br>
**Resource Reclamation**:  We estimate how many resources a particular task will require and reclaim the rest for work that can tolerate lower quality resources(bath jobs) <br>


### Understanding Borg Workload:

Borg workload is classified into two types:

1. Services that never go down:
    * Long running services that never go down.
    * Handle short lived requests.
    * Latency Sensitive
    * Duration ranges from few microseconds to milliseconds

2. Batch jobs:
    * Take few seconds - few days to complete.
    * Less sensitive to short-term performance fluctuations.

### Understanding Borg Jobs:

* A job holds the following attribute(properties):
    1. Name
    2. Owner
    3. Number of Tasks
    4. Priority

* Borg programs are statically linked to reduce dependencies on the runtime environment.

* Borg jobs have the following lifecycle:
    1. Pending
    2. Running
    3. Dead

* A user changes the properties of a running Borg job, by pushing a new job configuration and then instructing Borg to update tasks according to the new configuration.

**Note**: Machines in a cell are heterogenous in nature. They have different specifications for components such as RAM, CPU, disk, network etc.


### Priority, Preemption and Quotas:

* Higher priority tasks will preempt the lower priority ones for resources. 
* Due to this behavior, a very possible scenario is preemption cascading, where a higher priority task-'A' preempts a slightly lower priority task-'B' resulting in this 'B' getting rescheduled elsewhere. If there was another task-'C' slightly lower in priority to 'B', this would result in 'C' getting preempted and rescheduled. This cycle could hence continue endlessly.

* This situation is avoided by mandating that tasks in the production band cannot preempt each other.

* Quota checking is part of the admission control. Usage of quota reduces the need for DRF.

### Naming and Monitoring:

* Borg creates a BNS name for each of the task. This name is composed of the cell name, job name and the task number.
* Borg writes the hostname and the port of the task into a consistent highly available file in Chubby - Google's lock based data store.


## Architecture:

* Borg has 3 components:
    1. Set of Machines
    2. A centrallized controller called Borgmaster
    3. Agent service called Borglets, that run on each machine in the cell.

### Borgmaster:

* Logically a single process but is replicated 5 times.
* Each replica maintains an in-memory copy of most of the state of the cell(Borg Cell), in a highly available Paxos-based store. (Leader-follower model)
* Borgmaster takes regular snapshots of it's internal state which is stored in a Paxos based store. This snapshot is called as a Checkpoint.
* Checkpoints have the following uses:
    1. Revert Borgmaster's state to a previous point in time.
    2. Build and maintain a persistent log
    3. Offline simulations (What kind ?)

### Job Scheduling:

* When a new job request is submitted, Borgmaster records the job details into the Paxos based store. It then adds the job's tasks to a pending queue.
* A scheduler asynchronously reads tasks from this pending queue and assigns them to nodes/machines.
* Consider the fact that each job is assigned to a Borg Cell, and each Borg Cell has it's own Borgmaster and Borglets.
* Scheduling algorithm has 2 parts:
    1. Feasibility Checking: Finds machines where the given task can run.
    2. Scoring: Picks one of the feasible machines.
* Score tries to minimize the number and the priority of preempted tasks.

* Several Scoring strategies exist, with the three most popular being:
    1. Worst Fit: Advantage is that load is spread across all nodes, leaving room for spikes. However this is at the risk of fragmentation.
    2. Best Fit: Tries to fill machines as tightly as possible. Results in several machines having no user tasks. Disadvantage is that, this tight packing penalizes mis-estimations in resource requirements and hurts batch jobs the most.
    3. Hybrid fit: Currently used by Borg, this tries to reduce the amount of stranded resources. Provides up to 3-5% better packing efficiency that best-fit.

* Package installation takes up 80% of the total 'Task Startup latency'. Borgmaster hence, in an effort to reduce this component of the Task Startup Latency, tries to schedule tasks to nodes that already have majority of the required packages installed.

###  Borglet:

* Performs the following important functions:
    1. Starts and stops the tasks.
    2. Restarts tasks whenever they fail.
    3. Manages local resources by manipulating local OS kernel.
    4. Rolls over the debug log and reports the state of the machine to the Borgmaster.

* If a Borglet does not respond to poll messages, the machine is marked as down and any tasks running on it are rescheduled to other nodes.
* In case the communication is restored, Borgmaster asks the Borglet to kill the rescheduled tasks, to avoid duplicates.
* A Borglet continues to work as normal, even if communication with Borgmaster is lost. This way there is no disruption to the service.

### Scalability:

* Early versions of the Borgmaster had a simple, synchronous loop that accepted requests, scheduled tasks and communicated with the Borglets.
* To handle larger cells, the scheduler process was split from Borgmaster, so that it could operate in parallel with the other Borgmaster functions that are replicated for failure tolerance.
* A scheduler replica repeatedly retrieves state changes from the elected master, updates its local copy, does a scheduling pass to assign tasks, informs the master about these assignments.
* Master will reject these assignments only if they are performed in an out of order (timestamp) fashion.

* Features that make Borg scalable:
    1. Score caching: Borg caches scores of a machine until the properties of the machine change.
    2. Equivalence Classes: Tasks in borg job usually have identical requirements and constraints. Hence, rather than determining feasibility for every task, it is rather determined for each equivalence class. 
    3. Relaxed Randomization: It is wasteful to calculate feasibilty and scores for each machine in the cell. Hence machines are randomly selected and a score is generated for them. The machine to which a task is scheduled to, is determined as the best available machine from this set of randomly sampled machines.

## Utilization Policies on Borg:

### Evaluation methodology:

* Cell compaction was used as an evaluation metric. For a given workload, this strategy is used to find the smallest cell that it can completely fit into. This is found by repeatedly checking if the workload fits into a cell and then removing cell resources until the workload no longer fit.


### Resource Reclamation:

* This is a method of resource conservation and putting resources to best use.
* Resource requirement for a task is estimated and then the rest, unclaimed resources are reclaimed. They are put to use for other tasks that can tolerate lower quality resources (Batch jobs).
* A machine may run out of resources at runtime, if the resource predictions are incorrect. In such a case, non-prod tasks are killed/bottlenecked.