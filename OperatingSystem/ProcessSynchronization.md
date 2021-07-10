# Process Synchronization

## Background
* `Concurrent` access to shared data may result in `data inconsistency`
* Maintaining data consistency requires mechanism to `ensure the orderly execution` of cooperating processes

### Example: Concurrent Operations on counter 
* The statement "counter++" may be implemented in machine language as:
	```
	move ax, counter
	add  ax, 1
	move counter, ax
	```
* The statement "counter--" may be implemented  as:
	```
	move bx, counter
	add  bx, 1
	move counter, bx
	```

#### Instruction Interleaving
* Assume the counter is initially 5. One interleaving of statement os:
	```
	producer: move ax, counter    -> ax = 5
	producer: add ax, 1           -> ax = 6
	context switch
	consumer: move bx, counter    -> bx = 5
	consumer: sub bx, 1           -> bx = 4
	context switch
	producer: move counter, ax    -> counter = 6
	context switch
	producer: move counter, bx    -> counter = 4 (wrong)
	```
* The value of the counter may be either 4, 5, or 6, where the correct result should be 5.
* Happens with Preemptive scheduling.

### Race Condition
* `Race condition`: 
	* The situation where several process `access and manipulate shared data concurrently`
	* The final value of the shared data `depends upon which process finished last`
* To prevent race condition, concurrent processes must be `synchronized`
	* On a single-processor machine, we could `disable interrupt` or use `non-preemptive CPU scheduling`
* Commonly described as `critical section problem`

## `Critical Section`

### The Critical-Section Problem
* Purpose: `a protocol` for processes to cooperate
* Problem description:
    * `N processes` are competing to use some `shared data`.
    * Each process has a `code segment`, called `critical section`, in which the shared data is accessed.
    * Ensure that when one process is executing in its critical section, `no other process is allowed to execute in its critical section` -> mutually exclusive.

### Critical Section Requirements
1. `Mutual Exclusion`: if process P is executing in its CS. no other processes can be executed in their CS.
2. `Progress`: `if no process is executing in its CS` and there exist some processes that wish to enter their CS, these `processes cannot be postponed indefinitely`
3. `Bounded Waiting`: `A bound must exist` on the number of times that `other processes are allowed to enter their CS` after a process has made a request to enter its CS

* `How to design the entry and exit section to satisfy the above requirement?`

### Critical Section Solutions & Synchronization Tools

#### Software Solution
* Peterson's Solution for Two Processes
* Bakery Algorithm(n processes)
    * Before enter its CS, each `process receives a #`
    * Holder of the `smallest # enters CS`
    * The numbering scheme always generates `# in non-decreasing order`; i.e., 1,2,3,3,4,5,5,5
    * If processes Pi amd Pj receive the `same #, if i < j, then Pi is served first`
    * Notation: (a, b) < (c, d) if a < c or if a == c && b < d
* Pthread Lock/Mutex Routines
* Condition Variables(CV)
	* CV represent some `condition` that a thread can:
		* Wait on, until the condition occurs; or
		* Notify other waiting threads that the condition has occurred.
	* Three operations on condition variables:
		* `wait()` --- `Block` until another thread calls signal() or broadcast() on the CV.
		* `signal()` --- Wake up `one thread` waiting on the CV
		* `broadcast()` -- Wake up `all threads` waiting on the CV
	* In Pthread, CV `type` is a `pthread_cond_t`
    * ThreadPool Implementation

#### Synchronization Hardware Support
* The CS problem occurs because the modification of a shared variable may be `interrupted`
* `If disable interrupts when in CS...`
	* not feasible in multiprocessor machine
	* clock interrupts cannot fire in any machine
* HW support solution: `atomic instructions`
	* atomic: `as one uninterruptible unit`
	
## `Semaphores`
* A `tool` to generalize the synchronization problem(`easier to solve, but no guarantee for correctness`)
* More specifically...
	* `a record` of `how many units` of a particular resource are available
		* if #record = 1 -> `binary semaphore, mutex lock`
		* if #record > 1 -> `counting semaphore`
	* accessed only through 2 `atomic` ops: `wait` & `signal`

* `Spinlock` implementation:
	* Semaphore is an `integer variable`
		```
		wait(S){
			while(S <= 0);
			S--;
		}
		```
		```
		signal(S){
			S++;
		}
		```
### POSIX Semaphore
* Semaphore is part of `POSIX standard` BUT it is `not belonged to Pthread`
	* `It can be used with or without thread`
* POSIX Semaphore routines:
	```
	* sem_init(sem_t*sem, int pshared, unsigned int value)
	* sem_wait(sem_t*sem)
	* sem_post(sem_t*sem)
	* sem_getvalue(sem_t*sem, int*valptr)
	* sem_destory(sem_t*sem)
	```
* n-Process Critical Section Problem
	* shared data: 
		`semaphore mutex` //initially mutex = `1`
	* Process Pi:
		```
		do {
			wait(mutex); //pthread_mutex_lock(&mutex)
				critical section
			signal(mutex); //pthread_mutex_unlock(&mutex)
				remainder section
		} while(1);
		```
	* `Progress? Yes`
	* `Bounded waiting? Depends on the implementation of wait()`

### Non-busy waiting Implementation
* Semaphore is `data struct with a queue`
	* may use any queuing strategy(FIFO, FILO, etc)
		```
		typedef struct{
			int value;//`init to 0`
			struct process * L;
			//`"PCB" queue`
		} semaphore;
		```
* wait() and signal()
	* use system calls: `block() and wakeup()`
	* must be executed `atomically`
		```
		void wait(semaphore S){
			S.value--;//`subtract first`
			if(S.value < 0){
				add this process to S.L;
				`sleep`();
			}
		}
		```
		```
		void signal(semaphore S){
			S.value++;
			if(S.value <= 0){
				remove a process P from S.L;
				`wakeup`(P);
			}
		}
		```

### Cooperation Synchronization
* P1 executes S1; P2 executes S2
	* S2 be executed only after S1 has completed
* Implementation
	* shared var:
		* semaphore `sync`; //initially sync =0

	P1:				    P2:
	--			        --
	S1;			        wait(`sync`)
	signal(`sync`);     S2;


# References
* [Operating System Course by Jerry Chou](https://www.youtube.com/watch?v=CNO_I8jhX3I&list=PLS0SUwlYe8czigQPzgJTH2rJtwm0LXvDX&index=52)