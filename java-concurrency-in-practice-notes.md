# Java concurrency in practice

## 1. Introduction
- history: in the past only one program was run on the computer. When OS was introduced it enabled cooperation of multiple programs on single machine. OS and multiple programs provides fairness, full resources utilization and convenience. 
  - threads: share memory, file handlers but have own stack, program counters and local variables
- existence of threads solves the problem of idling during IO actions
- makes is possible to decouple complex programs into smaller tasks that can be run independently
- livenes hazards: e.g. infinite pool or dead locks = program doesn’t move forward
- performance hazards: e.g costs of context switching 
- threads over abstractions are used in many frameworks

## 2. Thread safety
- if multiple threads access the same mutable variable without synchronization program becomes broken, to avoid that we can:
    - don't share the state variable across threads 
    - make the state immutable
    - use synchronization whenever accessing the state
- OO principles like immutability and encapsulation helps to write thread safe code
- class is thread-safe if it behaves correctly when accessed from multiple threads, regardles of how they use that class (serially or interleaving)
- stateless objects are always thread-safe
- atomicity = closing all operations on an object in one action, other thread cannot see state of that object during modifications of that action
- *race condition* occurs when the correctness of a computation depends on the relative timing or interleaving of multiple threads by the runtime
- having compund actions that have more that one steps usage of atomic data structures in not sufficient as it still can cause race conditions, we need lock with broader scope
- *synchronized* block has two parts: reference to an object that will serve as the lock and a block of code that will be guarded by that block (static synchronized methods take thc Class object for the lock )
- reentracy: when the same thread calls more than one synchronized methos on a single object it is not blocked
- every shared, mutable variable should be guarded by exactly one locka (commong pattern is to synchornize path that accesses that variable)
- synchronized block should be as small as possible and contain no long running operations, avoid network or I/O calls!
- *writing correct concurrent programs is primarly about managing access to shared, mutable state*

## 3. Sharing objects
- we want not only to prevent one thread from modyfing the state of an object but also ensure that when a thread modifies the state of an object, other threads can *see* the changes that were made
- in the absence of synchronization the compiler, processor and runtime can do some downrigh wierd things to the order in which operations appear to execute
- it's not safe to use nonvolatile *long* and *double* values as 64 bit read can be trated as two indpendent 32 bit operations that can represent two different states (state can change between these tow operation - so the first 32 bits can be from old state and another 32 bits from the new state)
- when threads *synchronize* on a common lock they all see the most up-to-date values of shared mutable vairiable
- *volatile* variable notifies compiler and runtime that this variable is shared and operations on it should not be reordered with other memory operations. Read of a volatile varialbe always returns the most recent write by any thread. Use cases for *volatile* are completion, interruption or status flags.
- keep objects confined to single thread
- stack confinement is a special case of thread confinement in which an object can only be reached through local variables. It happens by default when we use primitively typed local vairables and for objects we need to store references in local variables and not let them to escape. 
- ThreadLocal<T> allows you associate a per-thread value and can be conceptually treated as Map<Thread, T>. *get* always returns value recently passed to *set* from the currently executing thread. Commonly used to keep transaction state in different frameworks.
- *final* fields cannot be modified, it is a good practice to make all fields *final* unless they need to be mutable 
- to provide atomicity we can wrap data we want to operate on atomically with immutable data class e.g. to provide atomic write to two variables put them into signle immutable object and read/write consistent values from it
- thread-safe collection ensures that every element written by thread A is visible in subsequent read from thread B