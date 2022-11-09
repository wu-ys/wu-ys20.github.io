# Operating Systems and Distributed Systems

- Real-world coding
- performance, scalability, fault tolerance, cooperate

- How to evaluate a system?

- How to solve the above challenges? **Engineering** Principles ot system design.
  - Learn how existing systems work?

Challenges for modern system design:

- Running on a Range of Devices

- No more Moore’s Law
- Software Complexity
- Less qualified programmers

- Internet Scale: $>1$ billion hosts





## 1 OS Overview

Users are the biggest enemy to any system.

What's an OS, really? “*The one program running at all times on the computer*” is the kernel 

- Everything else is either a system program (ships with the OS) or an application program

Abstraction, protection

## 2 Synchronization(1)

### Concepts

#### Thread

single unique execution context

- A thread is executing on a processor (core) when it's resident; is *suspended* (not executing) when its state is not loaded (resident) into the processor (stored in memory).

#### Address Space

The set of accessible addresses (Valid domain as interface)

- Protection mechanism: each process's memory is protected.
- Address space translation (Virtual address)

#### Process

Execution environment with Restricted Rights

- With protected address space and 1 or more threads
- OS protected from user's processes.

#### Dual Mode

hardware supports at least two modes: kernel and user

- a bit of state
- certain operations / actions only permitted in system / kernel modes;
- User to Kernel transition sets system mode AND saves user PC;
- Kernel to User transition clears system mode AND restores appropriate user PC;

![image-20220922110154562](https://wu-ys.github.io/courses/os-ds/midterm-sheatsheet.assets/image-20220922110154562.png)

Types of mode transfer: syscall, interrupt, trap or exception

### Process from user level

Creating processes: `fork()`, copy current process and return pid >= 0

Wait for a process to finish: `wait()`

Change the *program* being run by the current process: `exec()`

Set handlers for signals `sigaction()`

### Cooperating threads

Advantage: share resourses, speed up, modularity

Thread pools, representing the maximum level of multiprogramming

Concurrency Problems: Shared data, sharing granularity

- **Atomic Operation**: an operation that always runs to completion or not at all.
  - Indivisible
  - Fundamental building block
- **Synchronization**: using atomic operations to ensure cooperation between threads
- **Critical Section**: piece of code that only one thread can execute at once
- **Mutual Exclusion**: ensuring that only one thread executes critical section
- **Lock**: prevents someone from doing something

#### Ideas of Synchronization

- lock everything
- busy waiting
- Atomic operations: section of code between `Lock.Acquire()` and `Lock.Release()` is a Critical Section

### Semaphores

integer, a kind of generalized locks

- `P()`: *proberen* (to test). An atomic operation that waits for semaphore to become positive, then decrements it by 1.
  - Think of this as the wait() operation

- `V()`: *verhogen* (to increment). An atomic operation that increments the semaphore by 1, waking up a waiting P, if any
  - Think of this as the signal() operation

- Producer-consumer with a bounded buffer (FIFO)

## 3 Synchronization(2)

### Monitors & Condition Variables

Monitor: a lock and zero or more conditional variables for managing concurrent access to shared data.

Lock: the lock provides mutual exclusion to shared data

- Always acquire before accessing shared data structure, always release after finishing with shared data
- Initially free

Condition Variable: a queue of threads waiting for something *inside* a critical section

- Key idea: make it possible to go to sleep inside critical section by atomically releasing lock at time we go to sleep

- Operations:

  - `Wait(&lock)`: Atomically release lock and go to sleep. Re-acquire lock later before returning. 

  - `Signal()`: Wake up one waiter, if any

  - `Broadcast()`: Wake up all waiters

**Hoare-style Monitors**: Signaler gives up lock and CPU to waiter; waiter runs immediately; waiter gives up lock, processor back to signaler when it exits critical section or if it waits again.

![image-20221109160855868](https://wu-ys.github.io/courses/os-ds/midterm-sheatsheet.assets/image-20221109160855868.png)

**Mesa-style Monitors**: Signaler keeps lock and processor; waiter placed on a local ready queue for the monitor; practically, need to check condition again after wait.

![image-20221109160847276](https://wu-ys.github.io/courses/os-ds/midterm-sheatsheet.assets/image-20221109160847276.png)

#### Application: Readers & Writers Problem

A shared database, two classes of users:

- readers, who never modify the DB
- writers, who read and modify DB

Exclusive lock & shared lock

#### Performance Model

Four times of context switching for acquiring and releasing lock.



## 4.1 MultiThread: Deadlocks

*Resources:* passive entities needed by threads to do their work

- Preemptable & non-preemptable
- Exclusive access & Sharable

Problems: Starvation & Deadlock

### Four Requirements for Deadlock

- Mutual exclusion
  - Only one thread at a time can use a resource

- Hold and wait
  - Thread holding at least one resource is waiting to acquire additional resources held by other threads

- No preemption
  - Resources are released only voluntarily by the thread holding the resource, after thread is finished with it

- Circular wait

### Solving Deadlock

- All system to enter deadlock and then recover
  - Require detection algorithm
  - forcibly preempting resources and/or terminating tasks
- Deadlock prevention
  - Infinite Resources (Not realistic?)
  - No sharing of resources
  - Don't allow waiting
  - **Banker's Algorithm**

- Ignore the problem and pretend deadlocks never occurs

- Resource Allocation Graph
  - A set of threads, resources types and instances
  - `Request`, `Use`, `Release`
  - Request edge, assignement edge -> directed edges

- Detection Algorithm

![image-20221105150951637](https://wu-ys.github.io/courses/os-ds/midterm-sheatsheet.assets/image-20221105150951637.png)

### Prevention

- Infinite Resources
- No sharing of resources
- Don't allow waiting
- Pre-request everything that will be needed for the thread to execute.
  - Force all threads to request resources in a perticular order.

- Banker's algorithm: allocate dynamically, keep the system safe.
  - Allow the particular thread to proceed if `Avail + Alloc >= Max`.


![image-20221105151158075](https://wu-ys.github.io/courses/os-ds/midterm-sheatsheet.assets/image-20221105151158075.png)

## 4.2 I/O

Goal: Provide uniform interfaces, despite wide range of different devices

### Categories of IO Device Interfaces

- Datatype: Block Devices, Character/Byte Devices, Network Devices
- Timing: Blocking Interface, Non-blocking Interface, Asychronous Interface.
- Kernel-level drivers: for critical devices
- User-level drivers: for non-threatening devices

Example: Memory-mapped Display Controller

DMA (Data Memory Access)

Notifying the OS: Interrupt & Polling

### Performance

Response Time = Queue + I/O device service time

- Metrics: Response Time, Throughput

- Contributing factors to latency:

  - Software paths (can be loosely modeled by a queue)

  - Hardware controller

  - I/O device service time

- Solutions:
  - Make everything faster?
  - Decouple systems
  - Buffering

## 4.3 Intro to Queuing Theory

Arrival distribution: Requests arrive following exp distribution $f(x) = \lambda e^{-\lambda x}$

Service time: Mean $m_1$, Variance $\sigma^2$, coefficeint of variance $C=\sigma^2/m_1^2$.

Little's Law: the average number of tasks in the system (N) is equal to the throughput (B) times the response time (L).

Results:

- Memoryless service distribution (C = 1):
  - Called M/M/1 queue: $T_q = T_{ser} \times u/(1 – u)$

- General service distribution (no restrictions), 1 server:
  - Called M/G/1 queue: $T_q = T_{ser} \times \frac12(1+C) \times  (u/(1 – u))$.

- Disk: $C\approx 1.5$.

## 5 Networking

Hosts, Connection: multiplexing.

Packet Switching: each packet travels independently to the dst host.

### Main Functionalities

- Delivery
- Reliability
- Flow Control
- Congestion Control

### Delivery: Multiplexing

Network (Interface) card/Controller: Hardware that physically connects a computer to the network. Each NC is associated with two addresses:

- **MAC/Physical address**: 48-bit unique identifier assigned by card vendor

- **IP/Network Address**: 32-bit (or 128-bit for IPv6) address assigned by network administrator or dynamically when computer connects to network

**Connection**: communication channel between two processes. Each endpoint is identified by a **port number**. Globally, an endpoint is identified by (IP address, port number)

- **Port number**: 16-bit identifier assigned by app or OS

### Layering

Introduce intermediate layers that provide **set of abstractions**.

- Partition the system: Each layer solely relies on services from layer below, exports services to layer above
- Interface between layers defines interaction: Hiding implementation details; Layers can change without disturbing other layers

OSI Model: 7 layers (application, presentation, session, transport, network, datalink, physical)

Internet Protocol (IP): Only **five** layers; The functionalities of the missing layers (i.e., Presentation and Session) are provided by the Application layer.



#### 1 Physical Layer

**Service**: move information between two systems connected by a physical link

**Interface**: specifies how to send and receive bits 

**Protocol**: coding scheme used to represent a bit, voltage levels, duration of a bit

#### 2 Datalink Layer

**Service**: Enable end hosts to exchange frames (atomic messages) on the same physical line or wireless link; Possible other services:

- Arbitrate access to common physical media

- *May* provide reliable transmission, flow control

**Interface**: send *frames* to other end *hosts*; receive *frames* addressed to end host

- Each frame has a **header** containing a source and destination MAC address.
  - MAC Address: uniquely identifies a network interface; 48 bit assigned by the manufacturer.

**Protocols**: addressing, Media Access Control (MAC) (e.g., CSMA/CD - *Carrier Sense Multiple Access / Collision Detection*) 

- MAC Protocols (for broadcast channel): Hub / Switch LAN
  - Channel partitioning protocols: 1/N bandwidth to each host (inefficient)
  - Taking turns protocols: pass a token between active hosts (vulnerable)
  - Random Access

#### 3 (Inter) Network Layer

**Service**: Deliver packets to specified **network (IP) addresses** across multiple datalink layer networks; Possible other services:

- Packet *scheduling/priority*

- Buffer management
- IP Address: unique addr. assigned to each device by administrator or dynamically when device connected.

**Interface**: send *packets* to specified network address destination; receive *packets* destinated for end host

**Protocols**: define network addresses (globally unique); construct forwarding tables; packet forwarding

- Wide Area Network (WAN): connects multiple datalink layer networks using routers; different LANs can use different technologies
- Routers: Forward each packet on an incoming link to an outgoing link based on dst IP address; packets are buffered before forwarded. Forwarding table.
- IP Address vs MAC Address: location dependent, scalable.

#### 4 Transport Layer

**Service**: Provide end-to-end communication between processes (In contrast, IP is end-to-end communication between hosts); Demultiplexing of communication between hosts (support multiple processes on the same hosts communicating simultaneously); Possible other services:

- Reliability in the presence of errors

- Timing properties

- Rate adaption (flow-control, congestion control)

**Interface**: send message to specific process at given destination; local process receives messages sent to it

**Protocol**: port numbers, perhaps implement reliability, flow control, packetization of large messages, framing

- Port number: 16-bit number identifying endpoint of a transport connection (identifies a process, can be assigned by OS or requested by app)
- Transport Portocols: Datagram services(UDP), Reliable in-order delivery (TCP)

#### 7 Application Layer

![image-20221107203332061](https://wu-ys.github.io/courses/os-ds/midterm-sheatsheet.assets/image-20221107203332061.png)

![image-20221107203423599](https://wu-ys.github.io/courses/os-ds/midterm-sheatsheet.assets/image-20221107203423599.png)

#### 0 Hourglass Model

![image-20221107211405107](https://wu-ys.github.io/courses/os-ds/midterm-sheatsheet.assets/image-20221107211405107.png)

Single Internet-layer module (**IP**)**:**

- Allows arbitrary networks to interoperate: Any network technology that supports IP can exchange packets

- Allows applications to function on all networks: Applications that can run on IP can use any network

- Supports simultaneous innovations above and below IP; But changing IP itself, i.e., **IPv6** is very complicated and slow

### Performance Model

Single Packet Performance: Packet Delay

- On each link: Propagation, Transmission

- On each router: Processing, Queuing

Sustained Throughput: determined by bottlenect link

### TCP: *Transmission Control Protocol*

#### UDP vs TCP

* Reliable – guarantee delivery

* Byte stream – in-order delivery

* Connection-oriented – single socket per connection

- Setup connection followed by data transfer

![img](https://wu-ys.github.io/courses/os-ds/midterm-sheatsheet.assets/tcp.png)

#### Solutions

- Ordered Message: Assign packet sequence numbers, through socket interface to sort upon arrival.
- Reliable Delivery: checksum and receive send acknowledge packet if received ungarbled; Resend if no acknowledgement, then deal with duplicates.
- How to ACK? Stop & wait protocol: If sender doesn’t receive ACK after a reasonable timeout, it retransmits frame.
- Window-based ACK: Send up to N packets without ack (Allows pipelining of packets); Ack says “received all packets up to sequence number X”.
- Congestion Avoiding: adjust window size.

#### Performance Implication



## 6.1 RPC

### RPC: *Remote Procedure Call*

- A type of client/server communication, attempts to make remote procedure calls look like local ones.

- Goals: A nice abstraction than programming on the network, to make distribution transparent.

- Challenges: possible failure, different running environment -> local rep. of data.

### Basic Protocol

- Provides access to the exported methods of an object across a network or other I/O connection. 
  - A server registers an object, making it visible as a service with the name of the type of the object.
  - After registration, exported methods of the object will be accessible remotely.
  - A server may register multiple objects (services) of different types but it is an error to register multiple objects of the same type.

Stubs: Compiler generates from API stubs for a procedure on the client and server

- Client stub

  - **Marshals** arguments into machine-independent format

  - Sends request to server

  - Waits for response

  - **Unmarshals** result and returns to caller

- Server stub

  - **Unmarshals** arguments and builds stack frame

  - Calls procedure
  - Server stub **marshals** results and sends reply

### Stubs and Marshaling

RPC stubs do the work of marshaling and unmarshaling -> understand the datatype?

- Write a description of function signature using interface definition language
- Passing value parameters:

![image-20221107232917004](D:\wys\github-website\wu-ys20\courses\os-ds\midterm-sheatsheet.assets\image-20221107232917004.png)

### RPC failure

Request lost, reply lost, server crash, client crash?

- Partial Failure: one cannot tell the difference of a machine failure and network failure. Make transparent to client?
- Strawman solution: Make remote behavior identical to local behavior.
- Real solution: break transparency
  - Exactly once: impossible in practice
  - At least once: for idempotent operations
    - client keep trying until get a response
  - At most once: 
    - server send previous reply and do not process replicate requests
    - keep sliding window of valid (procecssed) RPC IDs.
  - Zero or once: transactional semantics
- Synchronous & Asynchronous RPC

![image-20221109103209704](D:\wys\github-website\wu-ys20\courses\os-ds\midterm-sheatsheet.assets\image-20221109103209704.png)

![image-20221109103201015](D:\wys\github-website\wu-ys20\courses\os-ds\midterm-sheatsheet.assets\image-20221109103201015.png)

## 6.2 Hard Drives

Disk, SSD

## 7 File System

File system: Layers of OS that transforms block interface of disks (or other) into files, directories, etc.

- Components: Disk Management, naming, protection, reliability/durability.
- Goals: maximize sequential performance, efficient random access to file, functionality

File: a collection of blocks (4kB in UNIX)

### File System Interfaces

Durable data store => it’s all on disk

(Hard) Disks Performance: Maximize sequential access, minimize seeks

Open before Read/Write: Can perform protection checks and look up where the actual file resource are, in advance

Size is only determined as they are used: Can write (or read zeros) to expand the file; Start small and grow, need to make room

Organized into directories

Need to allocate / free blocks 

#### Disk Management

File (Groups of sequentially arranged blocks), Directory (mapping of names to files)

Access disk as linear array of sectors (LBA, Logical Block Addressing)

- Every sector has integer address, from zero up to max number of sectors
- Controller translates from address to physical position

Functionality: (maintained somewhere in disk)

- Track free disk blocks
- Track block - file relation
- Track files in a directory

### Components

Name resolution: translate pathname into file number (as an index to locate blocks); create file descriptor; return a handle to user process (mapped to descriptor and blocks)

![image-20221109123522484](D:\wys\github-website\wu-ys20\courses\os-ds\midterm-sheatsheet.assets\image-20221109123522484.png)

#### Directory

Basically a hierarchical structure. Each directory entry is a collection of files or directories (a link to another entries)

Each has a name and attributes.

Links (hard links) make it a DAG, not just a tree.

- Soft links (aliases) are another name for an entry

#### File

Named permanent storage, contains data (blocks on disk somewhere), metadata (attributes, including owner, size, last opened, access rights, ...)

### In-memory File System Structure

- Open system calls:

  - Resolves file name, finds file control block (inode)

  - Makes entries in per-process and system-wide tables

  - Returns index (called “file handle”) in open-file table

- Read/write system calls:

  - Use file handle to locate inode

  - Perform appropriate reads or writes 

#### FAT (File Allocation Table)

#### UNIX BSD 4.1

#### UNIX BSD 4.2

### Directories and links

- `link File1 File2`: link existing file `file1` to a directory. Forms a DAG

- Hard link: Sets another directory entry to contain the file number for the file; Creates another name (path) for the file
- Soft link (or Symbolic Link or Shortcut): Directory entry contains the path and name of the file; Map one name to another name



 

### Storage Reliability

**Availability**: the probability that the system can accept and process requests

- Often measured in “nines” of probability. So, a 99.9% probability is considered “3-nines of availability”

**Durability**: the ability of a system to recover data despite faults

- This idea is fault tolerance applied to data

- Doesn’t necessarily imply availability

**Reliability**: the ability of a system or component to perform its required functions under stated conditions for a specified period of time

- Usually stronger than simply availability: means that the system is not only “up”, but also working correctly

- Includes availability, security, fault tolerance/durability

- Must make sure data survives system crashes, disk crashes, other problems