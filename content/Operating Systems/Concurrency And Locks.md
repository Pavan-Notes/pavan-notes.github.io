---
title: Concurrency and Locks
draft: false
tags:
  - Operating Systems
  - Concurrency
---
Multiple threads within a single process share:
- Process ID (PID)
- Address space
- Open file descriptors
- Current working directory
- User and group id

Each thread has its own
- Thread ID (TID)
- Set of registers, including PC and SP
- Stack of local variables and return addresses

## Timeline view

Due to having separate stacks but same address space, threads from a single process can overwrite the memory written by the other thread.
This behavior is called `Non-Determinism`

- Different results even with same inputs
- Race conditions

This can lead to `Heisen-bugs` which are non-deterministic. The bugs which are deterministic are called `Bhor-bugs`.

It is a good idea to assume CPU scheduler is malicious.

This happens when:

- There are multiple cores
- Interrupts

### Therac-25
Non determinsitic execution in threads led to radiation overdose.

### How do we address this issue?

Execute instructions as an uninterruptable group. - `Atomic`

`Mutual Exclusion` for critical sections - If thread A is in critical section C. Thread B is not.

## Synchronization

Build higher-level synchronization primitives in OS
Operations that ensure correct ordering of instructions across threads.

### Locks

#### Goals

**Correctness:**

- Mutual Exclusion
    Only one Thread in critical section at a time
- Progress (deadlock-free)
    If several simultaneous requests, must allow one to proceed
- Bounded (starvation-free)
    Must eventually allow each waiting thread to enter

**Fairness:**
- Each thread waits for same amount of time

**Performance:**
- CPU is not used unnecessarily

**Allocate and intialize**

- `Pthread_mutex_t mylock = PTHREAD_MUTEX_INITIALIZER;`

**Acquire**

- Acquire exclusion access to lock
- Wait if lock is not available (some other process in critical seciton)
- Spin or block (relinquish CPU) while waiting
- `PThread_mutex_lock(&mylock);`

**Release**

- Release exclusive acces to lock - Let another process enter critical seciton
- `Pthread_mutex_unlock(&mylock);`

**Question:** How do you decide who(which thread) gets the lock next?

#### Implementing Locks

##### W/ Interrupts

Turn off interrupt for critical section

- Prevent dispatcher from running another thread
- Code between interrupts executes atomically

```c
void acquire(lockT *l) {
    disableInterrupts();
}
```

```c
void release(lockT *l) {
    enableInterrupts();
}
```

**Disadvantages?**
- Only works on uniprocessors
- Process can keep control of CPU for arbitrary length
- Cannot perform other necessary work

This is what Linux and Windows do on single processors

##### W/ Load + Store

Code uses a single `shared` lock variable

```c
// shared variable
boolean lock = false;
void acquire(Boolean *lock) {
    while (*lock); /* wait */ 
    *lock = true;
}
```

```c
void release(Boolean *lock) {
    *lock = false;
}
```

**Does not work:**
- Both threads could look at the lock variable at the same time and acquire it. - `Race condition`
- Testing lock and setting lock are not atomic together.

##### Why need H/W support
The above software instrutions show why H/W support is needed.

To implement we need atomic operations

`Atomic operation`: No other instructions can be interleaved.

Examples:

- Code between interrupts on uniprocessors
    Disable timer interrupts, don't do any I/O
- Load and stores of single words
    - Load r1, B
    - Store r1, A
- Special h/w instructions
    - Test&Set
    - Compare&Swap

## XCHG: Atomic Exchange or Test-And-Set

