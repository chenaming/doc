
# 8.1 System Model


- A system consists of a finite number of resources to be distributed among a number of competing threads.
- The resources may be partitioned into several types(or classes),each consisting of some number of identical instances.
	- CPU cycles,files,and I/O devices(such as network interfaces and DVD drives) are examples of resource types
- If a system

System Model

- System consists of resources
- Resource types R1, R2, . . ., Rm
	- CPU cycles, memory space, I/O devices
- Each resource type Ri has Wi instances.
- Each process utilizes a resource as follows:
	- request
	- use
	- release


# 8.3 Deadlock Characterization

Deadlock Characterization

Deadlock can arise if four conditions hold simultaneously.
- Mutual exclusion:  only one process at a time can use a resource
- Hold and wait:  a process holding at least one resource is waiting to acquire additional resources held by other processes
- No preemption:  a resource can be released only voluntarily by the process holding it, after that process has completed its task
- Circular wait:  there exists a set {P0, P1, …, Pn} of waiting processes such that P0 is waiting for a resource that is held by P1, P1 is waiting for a resource that is held by P2, …, Pn–1 is waiting for a resource that is held by Pn, and Pn is waiting for a resource that is held by P0.
