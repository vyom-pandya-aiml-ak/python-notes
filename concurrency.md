# Python Concurrency: The Definitive Master Document

> **Threading · Multiprocessing · AsyncIO · The GIL**
> 
> A complete reference synthesized from authoritative sources and CPython internals.
> Covers Python 3.9 through 3.13, including PEP 684 (sub-interpreters) and PEP 703 (free-threaded builds).

-----

## Table of Contents

- [Preface: How to Read This Document](#preface-how-to-read-this-document)
- [Module 1: The Global Interpreter Lock (GIL)](#module-1-the-global-interpreter-lock-gil)
  - [1.1 What Is the GIL and Why Does It Exist?](#11-what-is-the-gil-and-why-does-it-exist)
  - [1.2 Under the Hood: CPython’s GIL Implementation](#12-under-the-hood-cpythons-gil-implementation)
  - [1.3 GIL and the OS: Kernel vs. User Space](#13-gil-and-the-os-kernel-vs-user-space)
  - [1.4 The GIL’s Impact: CPU-Bound vs. I/O-Bound](#14-the-gils-impact-cpu-bound-vs-io-bound)
  - [1.5 Annotated Code: Observing the GIL](#15-annotated-code-observing-the-gil)
  - [1.6 The Gilectomy and PEP 703: A History of Removal Attempts](#16-the-gilectomy-and-pep-703-a-history-of-removal-attempts)
  - [1.7 Danger Zone: GIL Pitfalls](#17-danger-zone-gil-pitfalls)
- [Module 2: Threading](#module-2-threading)
  - [2.1 What Is a Thread? OS-Level Architecture](#21-what-is-a-thread-os-level-architecture)
  - [2.2 The threading Module: Core API](#22-the-threading-module-core-api)
  - [2.3 Synchronization Primitives](#23-synchronization-primitives)
  - [2.4 Race Conditions: The Silent Killer](#24-race-conditions-the-silent-killer)
  - [2.5 ThreadPoolExecutor: The Modern High-Level API](#25-threadpoolexecutor-the-modern-high-level-api)
  - [2.6 Danger Zone: Threading Pitfalls](#26-danger-zone-threading-pitfalls)
- [Module 3: Multiprocessing](#module-3-multiprocessing)
  - [3.1 True Parallelism: The Process Model](#31-true-parallelism-the-process-model)
  - [3.2 Process Start Methods: fork, spawn, forkserver](#32-process-start-methods-fork-spawn-forkserver)
  - [3.3 The multiprocessing Module: Core API](#33-the-multiprocessing-module-core-api)
  - [3.4 Inter-Process Communication (IPC)](#34-inter-process-communication-ipc)
  - [3.5 ProcessPoolExecutor: The High-Level API](#35-processpoolexecutor-the-high-level-api)
  - [3.6 Danger Zone: Multiprocessing Pitfalls](#36-danger-zone-multiprocessing-pitfalls)
- [Module 4: Asynchronous I/O (asyncio)](#module-4-asynchronous-io-asyncio)
  - [4.1 The Concurrency Model: Cooperative Multitasking](#41-the-concurrency-model-cooperative-multitasking)
  - [4.2 The Event Loop: Under the Hood](#42-the-event-loop-under-the-hood)
  - [4.3 Coroutines, Tasks, and Awaitables](#43-coroutines-tasks-and-awaitables)
  - [4.4 Annotated Code: Core asyncio Patterns](#44-annotated-code-core-asyncio-patterns)
  - [4.5 TaskGroup: Structured Concurrency (Python 3.11+)](#45-taskgroup-structured-concurrency-python-311)
  - [4.6 async/await Internals: Generator-Based State Machines](#46-asyncawait-internals-generator-based-state-machines)
  - [4.7 Running Blocking Code Inside asyncio](#47-running-blocking-code-inside-asyncio)
  - [4.8 Danger Zone: asyncio Pitfalls](#48-danger-zone-asyncio-pitfalls)
- [Module 5: Comparative Analysis](#module-5-comparative-analysis)
  - [5.1 The Decision Tree: Which Tool for Which Problem?](#51-the-decision-tree-which-tool-for-which-problem)
  - [5.2 Performance: I/O-Bound Tasks](#52-performance-io-bound-tasks)
  - [5.3 Performance: CPU-Bound Tasks](#53-performance-cpu-bound-tasks)
  - [5.4 Memory and Resource Cost](#54-memory-and-resource-cost)
  - [5.5 ThreadPoolExecutor vs. ProcessPoolExecutor](#55-threadpoolexecutor-vs-processpoolexecutor)
  - [5.6 asyncio vs. ThreadPoolExecutor](#56-asyncio-vs-threadpoolexecutor)
  - [5.7 The asyncio + ProcessPool Hybrid Pattern](#57-the-asyncio--processpool-hybrid-pattern)
- [Module 6: Advanced Topics and Python 3.12/3.13 Features](#module-6-advanced-topics-and-python-312313-features)
  - [6.1 Sub-Interpreters and PEP 684 (Python 3.12)](#61-sub-interpreters-and-pep-684-python-312)
  - [6.2 Free-Threaded Python 3.13 (PEP 703)](#62-free-threaded-python-313-pep-703)
  - [6.3 Async Generators, Comprehensions, and Context Managers](#63-async-generators-comprehensions-and-context-managers)
  - [6.4 asyncio.Semaphore for Rate Limiting](#64-asynciosemaphore-for-rate-limiting)
- [Module 7: Quick-Reference Summary Tables](#module-7-quick-reference-summary-tables)
  - [7.1 Master Comparison: All Three Models](#71-master-comparison-all-three-models)
  - [7.2 Synchronization Primitives Reference](#72-synchronization-primitives-reference)
  - [7.3 When to Use What: The One-Page Cheat Sheet](#73-when-to-use-what-the-one-page-cheat-sheet)

-----

## Preface: How to Read This Document

This document is structured as a progressive deep-dive. Each of the four primary modules — the GIL, Threading, Multiprocessing, and Asyncio — begins with foundational architecture, then advances through implementation details, annotated code, comparative analysis, and finally a **Danger Zone** covering real-world failure modes. You do not need to read linearly; each module is self-contained, with explicit cross-references to related concepts.

The consistent analogy used throughout is a **restaurant kitchen**: the kitchen (process) has a single head chef (GIL) who must sign off on every action, multiple line cooks (threads) who share the same workspace and ingredients, and a waiter (the event loop) who takes many orders simultaneously and dispatches tasks asynchronously without ever blocking the dining room floor.

A quick orientation on terminology before diving in. **Concurrency** means dealing with multiple things at once — tasks may overlap in time but not necessarily run simultaneously. **Parallelism** means executing multiple things simultaneously on multiple CPU cores. Python’s threading gives you concurrency but not parallelism (for pure Python code). Multiprocessing gives you both. Asyncio gives you concurrency with neither threading overhead nor multiple cores.

-----

## Module 1: The Global Interpreter Lock (GIL)

### 1.1 What Is the GIL and Why Does It Exist?

The **Global Interpreter Lock** is a mutex — a mutual exclusion lock — embedded in the CPython interpreter itself. Its rule is brutally simple: *only one thread may execute Python bytecode at any point in time*. Every thread, before it can run a single instruction, must first acquire this lock. The moment it releases the lock, another waiting thread may acquire it and take its turn.

To understand why this exists, you need to understand CPython’s memory management strategy: **reference counting**. Every Python object carries an integer field, `ob_refcnt`, embedded directly in the `PyObject` C struct. This field tracks how many live references point to the object. When that count reaches zero, the memory is freed immediately. This is simple, deterministic, and fast — but it is emphatically **not thread-safe** without protection.

Consider what happens when two threads both hold a reference to the same list object and both execute `del my_ref` simultaneously. On a multi-core CPU, both threads may decrement `ob_refcnt` concurrently. Because the decrement operation is not a single atomic CPU instruction (it involves a load, a subtract, and a store — three separate steps), a race condition can cause the count to reach zero prematurely, triggering a premature free and a use-after-free bug. Or the opposite: the count might never reach zero, causing a permanent memory leak. Either way, the program is silently corrupted.

The naive fix would be to add a lock to every Python object. But this creates two new problems: **deadlocks** (if any two locks can be acquired in different orders by different threads) and **performance degradation** from the constant acquisition and release of potentially millions of per-object locks. The GIL is the elegant — if controversial — solution: one single lock over the entire interpreter. Deadlocks become impossible (you can’t deadlock with a single lock), and the per-object locking overhead vanishes entirely.

> **Key Insight:** The GIL is not a design flaw that was accidentally introduced. It was a deliberate engineering tradeoff in the late 1980s that made CPython simpler to implement, single-threaded code faster (no per-object lock overhead), and C extension integration far easier. The cost is that CPU-bound multi-threaded Python programs cannot use multiple cores.

### 1.2 Under the Hood: CPython’s GIL Implementation

In CPython’s C source (`Python/ceval_gil.c`), the GIL is implemented not as a simple OS mutex but as a combination of a **mutex**, a **condition variable**, and several **atomic flags**. This design allows sleeping threads to be awakened efficiently by a signal rather than busy-waiting in a spin loop.

The sequence of a thread acquiring the GIL works roughly like this. First, the thread checks an atomic `locked` flag. If the flag is clear (GIL is free), the thread sets it atomically using a compare-and-swap instruction and proceeds to execute Python bytecode. If the GIL is held by another thread, the requesting thread increments a `gil_drop_request` counter and parks itself on the condition variable, sleeping until it is signaled.

The currently running thread checks the `eval_breaker` flag between bytecode instructions — this is a combined signal that indicates any of: a GIL drop was requested, a signal needs to be handled, or the interpreter is shutting down. When `gil_drop_request` is set and the switch interval has elapsed, the running thread releases the GIL and sleeps, allowing a waiting thread to acquire it.

In Python 3.2 and later, the GIL uses a **timer-based forced-switch** mechanism. The default switch interval is 5 milliseconds, configurable via `sys.setswitchinterval()`. Before Python 3.2, the switch mechanism was instruction-count-based: every 100 bytecode instructions, the thread would check if a switch was needed. The modern timer-based approach is fairer and avoids a pathological case where a thread executing 100 very fast instructions would reacquire the GIL before a waiting I/O thread ever got a chance.

```python
import sys

# Inspect and tune the GIL switch interval
print(sys.getswitchinterval())    # 0.005 (5ms default)

# Reduce to 1ms for I/O-heavy applications with many short tasks
# More frequent switches = more overhead but better I/O responsiveness
sys.setswitchinterval(0.001)

# Increase to 50ms for CPU-bound code that rarely yields
# Fewer switches = less overhead, but I/O threads may starve
sys.setswitchinterval(0.05)

# Always restore to default in production unless you have measured justification
sys.setswitchinterval(0.005)
```

### 1.3 GIL and the OS: Kernel vs. User Space

This is the most commonly misunderstood aspect of the GIL. Python threads are **real OS-level threads** — POSIX pthreads on Unix, Win32 threads on Windows. The OS kernel is fully aware of them and will schedule them across multiple CPU cores. The GIL, however, lives entirely in **user space** — the kernel knows nothing about it.

This creates a deep paradox. You can have 8 Python threads running on a machine with 8 CPU cores. The OS, doing its job correctly, schedules all 8 threads across all 8 cores simultaneously. But the GIL, sitting in user space, ensures that at the Python bytecode level, only one of those threads is executing Python code at any moment. The other 7 cores are running threads that are blocked on the GIL — waiting in the condition variable sleep, consuming a tiny amount of CPU for the kernel’s bookkeeping, but not doing any useful Python work.

This is not the whole story, though. When a thread makes a **blocking system call** — reading from a network socket, writing to disk, querying a database, or calling `time.sleep()` — CPython releases the GIL *before* entering the kernel call and reacquires it upon return. The key sequence is:

1. Thread A calls `socket.recv()`.
1. CPython executes the `Py_BEGIN_ALLOW_THREADS` macro, which releases the GIL.
1. Thread A is now in kernel space, blocked waiting for data. It does **not** hold the GIL.
1. Thread B, which was waiting for the GIL, immediately acquires it and begins executing Python code.
1. When the network data arrives, Thread A returns from the kernel call and executes `Py_END_ALLOW_THREADS`, blocking until it can reacquire the GIL.

This GIL-release-around-syscalls behavior is precisely why threading is effective for I/O-bound tasks: the total waiting time is parallelized even though the Python execution is not.

Many C extension modules (NumPy array operations, pandas, OpenCV, compression libraries) explicitly release the GIL around their compute-heavy C code using the same `Py_BEGIN_ALLOW_THREADS` / `Py_END_ALLOW_THREADS` macros. This is why a NumPy matrix multiplication can genuinely utilize multiple cores even though it’s invoked from Python — the actual computation happens in C with the GIL released.

### 1.4 The GIL’s Impact: CPU-Bound vs. I/O-Bound

|Scenario                                  |GIL Held?              |Multi-Thread Effective?          |Recommended Tool         |
|------------------------------------------|-----------------------|---------------------------------|-------------------------|
|CPU-bound Python code (loops, math)       |Yes, continuously      |No — serial execution            |`multiprocessing`        |
|I/O-bound (network, disk, DB)             |Released during syscall|Yes — threads run during I/O wait|`threading` or `asyncio` |
|C extension (NumPy, pandas, OpenCV)       |Released by extension  |Yes — true parallelism in C code |`threading` is sufficient|
|Mixed (Python CPU + I/O)                  |Intermittently held    |Partially — depends on ratio     |`asyncio` + `ProcessPool`|
|High-concurrency I/O (10,000+ connections)|Released during I/O    |Threads work but are wasteful    |`asyncio`                |

### 1.5 Annotated Code: Observing the GIL

The following code demonstrates the GIL’s serializing effect on CPU-bound threads. The critical observation is that two threads performing a CPU-bound task in parallel will be no faster — and often slower — than the same task run sequentially.

```python
import threading
import time

# ── DEMONSTRATING GIL IMPACT ON CPU-BOUND TASKS ──────────────────────────────

ITERATIONS = 50_000_000  # 50 million pure-Python arithmetic iterations

def cpu_task(label: str) -> None:
    """
    A tight Python loop. This is the worst case for the GIL because the thread
    never releases it voluntarily (no I/O, no sleep). The GIL is only released
    at the 5ms switch interval, forcing a context switch with overhead.
    """
    total = 0
    for i in range(ITERATIONS):   # Each iteration = multiple bytecode operations
        total += i                 # LOAD, ADD, STORE — GIL held throughout
    print(f"{label} done: {total}")

# ── SINGLE-THREADED BASELINE ──────────────────────────────────────────────────
start = time.perf_counter()
cpu_task("Single-thread")           # Run once, sequentially
single_time = time.perf_counter() - start
print(f"Single thread: {single_time:.2f}s")

# ── MULTI-THREADED — expect NO speedup due to GIL ────────────────────────────
t1 = threading.Thread(target=cpu_task, args=("Thread-1",))
t2 = threading.Thread(target=cpu_task, args=("Thread-2",))

start = time.perf_counter()
t1.start()    # Both threads are created and scheduled by the OS on real cores...
t2.start()    # ...but the GIL ensures only one executes Python bytecode at a time
t1.join()     # Wait for thread 1 to complete
t2.join()     # Wait for thread 2 to complete
multi_time = time.perf_counter() - start

# Expected: multi_time ≈ single_time, or even SLOWER due to GIL contention.
# On Python 3.12, you often see 10-30% SLOWDOWN vs single thread because
# OS context switches + GIL acquisition overhead add up to pure waste.
print(f"Multi-thread:  {multi_time:.2f}s")
print(f"Overhead:      {(multi_time / single_time - 1) * 100:.1f}%")
```

### 1.6 The Gilectomy and PEP 703: A History of Removal Attempts

The desire to remove the GIL is nearly as old as Python itself. Understanding the history is important because it explains why the GIL is still present in 2025 and what the path forward looks like.

The most famous removal attempt was **Larry Hastings’ Gilectomy** (2016), a fork of CPython that replaced reference counting’s GIL dependency with fine-grained per-object locks and biased reference counting. While it achieved genuine multi-core parallelism for CPU-bound code, single-threaded performance degraded by 50–60%. This was an unacceptable regression: the entire Python ecosystem — web frameworks, data science tools, scripts — runs primarily single-threaded. No community would accept a 50% slowdown for the benefit of better multi-threaded scaling.

The breakthrough came with **PEP 703** (authored by Sam Gross, 2023), accepted for Python 3.13 as an optional, experimental feature. The approach is more sophisticated, using three complementary techniques. **Immortalization** converts certain objects (small integers, interned strings, `None`, `True`, `False`) into immortal objects whose `ob_refcnt` is never modified, eliminating the main source of reference count contention. **Biased reference counting** uses a fast, non-atomic path for the thread that “owns” an object, reserving atomic operations only for cross-thread interactions. **Deferred reference counting** buffers `Py_DECREF` operations through a per-thread mechanism, reducing the frequency of atomic memory operations. The result: the optional `--disable-gil` build of Python 3.13 achieves near-linear multi-core scaling for CPU-bound code with only approximately 10% single-thread regression — a transformational improvement over the Gilectomy.

**PEP 684** (Python 3.12) introduced **per-interpreter GILs**, a stepping stone toward a GIL-free world. Each sub-interpreter instance gets its own GIL, allowing true parallelism between sub-interpreters within a single process. This is accessible via the `_interpreters` module (promoted to `interpreters` in 3.13).

> **Python 3.12/3.13 GIL Status:**
> 
> - Python 3.12: PEP 684 — per-interpreter GIL; sub-interpreters in a single process can run in parallel (experimental C API, `_interpreters` module).
> - Python 3.13: PEP 703 — build CPython with `--disable-gil` for free-threaded mode (`python3.13t` in some distributions). This is experimental and not yet production-ready.
> - Extension module ecosystem compatibility for free-threaded 3.13 is still being resolved (e.g., the `Py_GIL_DISABLED` build tag).
> - The goal is that Python 3.15–3.16 will ship a fully stable, opt-in free-threaded mode.

### 1.7 Danger Zone: GIL Pitfalls

> #### ⚠ Danger: GIL Contention Makes CPU-Bound Threads Slower
> 
> If you spawn N threads for a CPU-bound task, performance is often *worse* than single-threaded execution. Each context switch forces the OS to save and restore thread state, and the GIL acquisition/release cycle adds 1–5 microseconds of overhead per switch. With 8 threads fighting over the GIL on a pure CPU-bound task, you can measure a 3× **slowdown** compared to running the 8 tasks sequentially in a single thread. The OS does real work scheduling those threads on multiple cores, but all that work is wasted because the GIL serializes the execution anyway. **Always use `multiprocessing` for CPU-bound work.**

> #### ⚠ Danger: Assuming C Extensions Are Always GIL-Safe
> 
> Not all C extensions release the GIL. If a C extension holds the GIL throughout its computation (either intentionally for thread safety or by mistake), it will block all other Python threads for its entire duration. Always verify in the library’s documentation whether its functions release the GIL, and benchmark with multiple threads before assuming parallelism.

-----

## Module 2: Threading

### 2.1 What Is a Thread? OS-Level Architecture

A **thread** is the smallest unit of execution that the operating system’s scheduler can manage. All threads within a process share the same **virtual address space** — the same heap, the same code segment, the same global data, and the same open file descriptors. Each thread has its own **private stack** (default 8 MB on Linux, 1 MB on Windows), its own program counter tracking which instruction to execute next, and its own set of CPU registers.

This shared memory model is simultaneously threading’s greatest strength and its greatest danger. The strength: sharing a large data structure between two threads requires only passing a reference — zero copying, zero serialization. The danger: if both threads modify the same data structure concurrently without synchronization, the result is undefined behavior.

In CPython, every Python `threading.Thread` is backed by a real OS thread created via `pthread_create()` on Unix or `CreateThread()` on Windows. The OS kernel maintains a **Thread Control Block (TCB)** for each thread containing its stack pointer, program counter, register state, and scheduling metadata. Thread creation costs approximately 10–100 microseconds, which is why thread pools (which reuse threads) are preferred over creating a new thread for every short task.

The operating system uses a **preemptive scheduler** with a hardware timer interrupt (typically every 1–10ms) to force context switches between threads regardless of what the threads are doing. This is fundamentally different from asyncio’s cooperative model, where context switches happen only at explicit `await` points. The preemptive nature of threads means that any shared state mutation that is not atomic is vulnerable to corruption — the scheduler can interrupt a thread between any two instructions.

### 2.2 The threading Module: Core API

Python’s `threading` module provides a high-level, cross-platform wrapper over OS threads. The central class is `threading.Thread`. You instantiate it with a `target` callable and optional `args`/`kwargs`, call `.start()` to hand it to the OS scheduler, and `.join()` to block the calling thread until the target thread finishes.

```python
import threading
import time
import requests  # pip install requests

# ── I/O-BOUND TASK: ideal use case for threading ─────────────────────────────
# While a thread waits for a network response, the GIL is RELEASED,
# allowing other threads to execute Python code concurrently.

URLS = [
    "https://httpbin.org/delay/1",   # Each URL takes ~1 second to respond
    "https://httpbin.org/delay/1",
    "https://httpbin.org/delay/1",
    "https://httpbin.org/delay/1",
]

results = []                          # Shared list — written by multiple threads
results_lock = threading.Lock()       # Protects concurrent writes to 'results'

def fetch(url: str, idx: int) -> None:
    """
    Downloads a URL and appends the status code to the shared results list.
    The network wait happens inside the kernel, so the GIL is released for
    the entire duration of the HTTP round-trip.
    """
    # requests.get() triggers a blocking socket read syscall.
    # CPython releases the GIL before the syscall, other threads run freely.
    response = requests.get(url, timeout=10)

    # After the response arrives, we reacquire the GIL to run Python code.
    # We then immediately acquire our own lock to protect the shared list.
    with results_lock:
        results.append((idx, response.status_code))
        # Lock is automatically released when the 'with' block exits,
        # even if an exception is raised inside the block.

# ── SEQUENTIAL BASELINE: ~4 seconds total ────────────────────────────────────
start = time.perf_counter()
for i, url in enumerate(URLS):
    fetch(url, i)   # One HTTP request at a time, each waits ~1 second
print(f"Sequential: {time.perf_counter() - start:.2f}s")

results.clear()

# ── THREADED: ~1 second total (all 4 requests overlap) ───────────────────────
threads = []
start = time.perf_counter()
for i, url in enumerate(URLS):
    t = threading.Thread(target=fetch, args=(url, i))
    threads.append(t)
    t.start()    # All 4 threads start and immediately block on their I/O syscall

for t in threads:
    t.join()     # Wait for each thread to complete before printing results

print(f"Threaded:   {time.perf_counter() - start:.2f}s")  # ~1.0s
print(f"Results: {sorted(results)}")
```

Key lifecycle properties to know: a thread is a **daemon thread** if `thread.daemon = True` is set before `start()`. Daemon threads are silently killed when the main thread exits. Non-daemon threads (the default) prevent the interpreter from exiting until they complete. `threading.current_thread()` returns the current thread object, and `threading.main_thread()` returns the main thread.

### 2.3 Synchronization Primitives

Python’s `threading` module provides a full suite of synchronization tools. Understanding when to use each is as important as knowing they exist.

**`threading.Lock`** is the fundamental mutual exclusion primitive. Only one thread may hold it at a time. Attempting to acquire an already-held lock blocks the calling thread indefinitely (or until a timeout if specified). The idiomatic usage is always `with lock:` rather than manual `lock.acquire()` / `lock.release()`, which guarantees release even on exceptions.

**`threading.RLock`** (re-entrant lock) is like `Lock` but the same thread may acquire it multiple times without deadlocking. Each acquisition must be matched by a release. This is essential for recursive code where the same function (holding a lock) calls itself again, and for class methods that all require the same lock — an outer method can call an inner method without either deadlocking.

**`threading.Semaphore`** is initialized to a count N and allows up to N threads to proceed concurrently. Each `acquire()` decrements the count; each `release()` increments it. When the count is zero, `acquire()` blocks. The classic use case is a connection pool: `Semaphore(5)` allows at most 5 threads to hold a database connection simultaneously.

**`threading.Event`** is a simple boolean flag with blocking wait semantics. One thread calls `event.set()` to signal; waiting threads unblock. `event.wait(timeout)` optionally times out. Useful for “ready” signals — a producer sets the event when data is available; consumers wait on it.

**`threading.Condition`** combines a lock with `wait()` / `notify()` semantics. A thread acquires the condition’s lock, checks a predicate, and if false, calls `condition.wait()` — which atomically releases the lock and parks the thread. When another thread changes the state and calls `condition.notify()`, the waiting thread wakes up, reacquires the lock, and rechecks the predicate. This is the classic pattern for producer-consumer coordination.

**`threading.Barrier`** synchronizes N threads to a common point. All threads block at `barrier.wait()` until all N have arrived, then all are released simultaneously. This is useful in parallel algorithms where all workers must complete a phase before any proceeds to the next.

```python
import threading
import time

# ── Semaphore example: limiting concurrent database connections ───────────────

db_semaphore = threading.Semaphore(3)  # At most 3 "connections" active at once
connection_log = []
log_lock = threading.Lock()

def db_worker(worker_id: int) -> None:
    """Simulates a worker that needs a limited database connection."""
    # If 3 connections are already active, this blocks until one releases.
    with db_semaphore:
        with log_lock:
            connection_log.append(f"Worker {worker_id} acquired connection")
        time.sleep(0.5)   # Simulate DB query time
        with log_lock:
            connection_log.append(f"Worker {worker_id} released connection")

threads = [threading.Thread(target=db_worker, args=(i,)) for i in range(8)]
for t in threads: t.start()
for t in threads: t.join()

for entry in connection_log:
    print(entry)
# You'll see at most 3 "acquired" lines before any "released" — the semaphore works.
```

### 2.4 Race Conditions: The Silent Killer

A **race condition** occurs when the correctness of a program depends on the relative timing of concurrent operations. Because the OS scheduler can preempt a thread between any two CPU instructions, any shared-state mutation that is not atomic is a potential race condition.

The classic example is incrementing a shared counter. In Python, `counter += 1` compiles to three distinct bytecode operations: `LOAD_GLOBAL counter` (read the current value), `BINARY_ADD 1` (add one), and `STORE_GLOBAL counter` (write back). If the OS preempts Thread A between the LOAD and the STORE, and Thread B then executes its full LOAD-ADD-STORE sequence, Thread A will overwrite Thread B’s result when it resumes — one increment is lost silently.

```python
import threading

# ── FAILURE STATE: Race condition on a shared counter ────────────────────────
# This demonstrates what happens WITHOUT proper synchronization.

counter = 0  # Shared mutable state — NO protection

def unsafe_increment(n: int) -> None:
    global counter
    for _ in range(n):
        # This is NOT atomic. It compiles to three bytecode operations:
        #   LOAD_GLOBAL  counter    ← read into temporary register
        #   BINARY_ADD   1          ← add 1 to register value
        #   STORE_GLOBAL counter    ← write register back to memory
        #
        # The OS scheduler can preempt between ANY of these operations!
        # If Thread B runs its LOAD after Thread A's LOAD but before Thread A's
        # STORE, both threads will store the same (N+1) value — losing one increment.
        counter += 1

N = 100_000
t1 = threading.Thread(target=unsafe_increment, args=(N,))
t2 = threading.Thread(target=unsafe_increment, args=(N,))
t1.start(); t2.start()
t1.join();  t2.join()
# Expected: 200,000. Actual: often 150,000-199,999 (varies per run — nondeterministic!)
print(f"Expected: {2 * N}, Got: {counter}")


# ── CORRECT VERSION: Protect shared state with a Lock ────────────────────────

safe_counter = 0
lock = threading.Lock()   # Exactly one lock guards exactly one piece of state

def safe_increment(n: int) -> None:
    global safe_counter
    for _ in range(n):
        # 'with lock' is equivalent to lock.acquire() ... lock.release(),
        # but is exception-safe and more readable.
        with lock:
            # Inside this block, ONLY this thread can execute.
            # The LOAD-ADD-STORE sequence is now effectively atomic at the Python level.
            safe_counter += 1

safe_counter = 0
t1 = threading.Thread(target=safe_increment, args=(N,))
t2 = threading.Thread(target=safe_increment, args=(N,))
t1.start(); t2.start()
t1.join();  t2.join()
# Now guaranteed to always be 200,000.
print(f"Expected: {2 * N}, Got: {safe_counter}")


# ── Thread-Local Storage: state that should NOT be shared ────────────────────
# threading.local() creates an object where each thread has its own independent
# copy of every attribute. Use this for per-thread state like DB connections,
# request contexts, or session objects.
tls = threading.local()

def set_request_id(rid: str) -> None:
    tls.request_id = rid    # Each thread writes its own value here
    time.sleep(0.01)        # Other threads may set their own tls.request_id
    print(f"Thread {threading.current_thread().name}: {tls.request_id}")

threads = [threading.Thread(target=set_request_id, args=(f"req-{i}",)) for i in range(5)]
for t in threads: t.start()
for t in threads: t.join()
# Each thread prints its OWN request_id — no cross-contamination
```

### 2.5 ThreadPoolExecutor: The Modern High-Level API

Directly managing `threading.Thread` objects is error-prone. Python 3.2 introduced `concurrent.futures.ThreadPoolExecutor`, which manages a pool of reusable worker threads, queues submitted tasks, and returns `Future` objects representing pending results. This is the recommended API for the vast majority of threading use cases.

```python
from concurrent.futures import ThreadPoolExecutor, as_completed
import time

def simulate_io(task_id: int, duration: float) -> str:
    """Simulates an I/O operation: a network call, DB query, or file read."""
    time.sleep(duration)  # time.sleep() releases the GIL — other threads run freely
    return f"Task {task_id} completed in {duration:.1f}s"

tasks = [(i, 0.5) for i in range(10)]  # 10 tasks × 0.5s = 5s if sequential

# ── submit() + as_completed(): process results as they arrive ────────────────
# max_workers guideline: I/O-bound → 2×–5× cpu_count; CPU-bound → cpu_count
# (but for CPU-bound, use ProcessPoolExecutor, not ThreadPoolExecutor!)
with ThreadPoolExecutor(max_workers=5) as executor:
    # submit() returns a Future immediately. The task is queued but not necessarily running yet.
    # The dict maps each Future to its task_id for error reporting.
    futures = {
        executor.submit(simulate_io, task_id, dur): task_id
        for task_id, dur in tasks
    }

    # as_completed() is a generator that yields each Future as it finishes.
    # Results arrive out of submission order — whichever finishes first comes first.
    for future in as_completed(futures):
        task_id = futures[future]
        try:
            result = future.result()   # Blocks until THIS specific future is done
            print(result)
        except Exception as e:
            # Exceptions raised inside the worker are re-raised here, not swallowed.
            print(f"Task {task_id} raised: {e}")

# The 'with' block calls executor.shutdown(wait=True) automatically,
# ensuring ALL pending tasks complete before code after the block runs.

# ── map(): simpler syntax when you don't need error handling per-task ─────────
with ThreadPoolExecutor(max_workers=5) as executor:
    # map() returns an iterator that yields results IN SUBMISSION ORDER,
    # regardless of completion order. It re-raises the first exception encountered.
    results = list(executor.map(lambda args: simulate_io(*args), tasks))
    print(results[:3])
```

### 2.6 Danger Zone: Threading Pitfalls

> #### ⚠ Danger: Deadlocks
> 
> A deadlock occurs when Thread A holds Lock1 and waits for Lock2, while Thread B holds Lock2 and waits for Lock1 — both blocked forever, neither able to proceed. Python’s process will appear to hang with no error message.
> 
> **Prevention rules:** (1) Always acquire multiple locks in a **consistent global order** across all threads. If every thread always acquires Lock1 before Lock2, a deadlock is impossible. (2) Use `lock.acquire(timeout=5.0)` to detect deadlocks — if the timeout expires, log the state and raise an alert rather than hanging forever. (3) Prefer higher-level `queue.Queue` over raw locks wherever possible; it handles all the locking internally. (4) Prefer `threading.RLock` for recursive code to avoid self-deadlock.
> 
> **Diagnosis:** When a process hangs, use `faulthandler.dump_traceback_later(timeout=5)` to print all thread stacks, or run `py-spy dump --pid <PID>` to see what every thread is waiting for.

> #### ⚠ Danger: Daemon Threads and Orphaned Work
> 
> Non-daemon threads (the default) prevent the interpreter from exiting. If the main thread finishes but a non-daemon thread is still running, the program hangs at exit, waiting for it. This is often seen as a mystery hang during testing.
> 
> Daemon threads (set `thread.daemon = True` before calling `thread.start()`) solve the hang but introduce a different danger: they are killed **abruptly** when the main thread exits, with no cleanup. Files may be left unclosed, database transactions uncommitted, and network connections forcibly closed. Never use daemon threads for work that requires guaranteed completion. Always `join()` important threads, or — best practice — use `with ThreadPoolExecutor(...)` which handles shutdown automatically.

> #### ⚠ Danger: Shared Mutable Default Arguments
> 
> Python’s default argument values are evaluated once at function definition time. If a thread-target function has a mutable default argument (e.g., `def f(data=[])`), all threads share the **same** list object — a subtle source of race conditions. Always use `None` as the default and create a new mutable object inside the function body.

-----

## Module 3: Multiprocessing

### 3.1 True Parallelism: The Process Model

A **process** is an isolated instance of a running program. It has its own virtual address space, its own file descriptor table, its own signal handlers, and — crucially — its own CPython interpreter, which means its own GIL. Since each process has a completely independent GIL, N processes can run N Python threads simultaneously on N CPU cores with absolutely no GIL interference. This is the only mechanism for achieving true parallelism with pure Python code in CPython.

The price of this isolation is substantial. Creating a new process involves the OS kernel cloning (`fork()`) or launching (`spawn`) a fresh Python interpreter — a cost of 50–200 milliseconds. Each process carries the full memory footprint of the Python interpreter plus your program’s imported state, typically 20–80 MB per process before any workload is applied. Communication between processes cannot use direct memory references; data must be explicitly serialized (pickled), transmitted through an OS pipe or shared memory region, and deserialized on the other side.

The kitchen analogy is apt here: multiprocessing is like opening N completely separate restaurant kitchens, each with their own independent inventory, stoves, and staff. They can all cook simultaneously without interfering with each other. But if Kitchen 1 needs an ingredient from Kitchen 2, a runner must physically carry it between buildings — there is no shared pantry.

### 3.2 Process Start Methods: fork, spawn, forkserver

Python’s `multiprocessing` module supports three process start methods, chosen via `multiprocessing.set_start_method()`. Understanding the differences is critical for correctness, especially on macOS and Windows.

**`fork`** (default on Linux, formerly default on macOS): The child process is created as an exact memory copy of the parent via the OS `fork()` syscall, using copy-on-write page mapping for efficiency. This is fast (~1ms) but inherits the parent’s entire state, including any mutexes that were held at fork time. In multi-threaded parent processes, this can cause deadlocks in the child: if Thread B was holding a lock when the fork happened, the child process contains that lock in a permanently-locked state with no thread to release it. macOS changed its default to `spawn` in Python 3.12 precisely because of this hazard.

**`spawn`** (default on Windows and macOS 3.12+, available on Linux): A fresh Python interpreter is started from scratch. The target function and its arguments are pickled and sent to the child’s stdin. This is the safest option — the child has no inherited state — but it’s slow (50–200ms) and requires that all top-level module code be guarded by `if __name__ == "__main__":`, because the child re-imports the module to get the target function.

**`forkserver`**: A separate server process is started once at program launch. For each new process request, the client sends a message to the server, which forks a clean single-threaded child. This combines some of fork’s speed with spawn’s safety. It avoids the multi-threaded parent problem and is faster than spawn after the initial server startup.

```python
import multiprocessing as mp

# Set the start method ONCE, at program startup, before any processes are created.
# This call must happen inside 'if __name__ == "__main__":', not at module level.
if __name__ == "__main__":
    # For cross-platform consistency (Linux + macOS + Windows), use 'spawn'
    mp.set_start_method("spawn")   # or "fork" on Linux for speed, "forkserver" for safety
    print(mp.get_start_method())   # Confirm the active method
```

### 3.3 The multiprocessing Module: Core API

```python
from multiprocessing import Process, Queue
import os
import time

# ── Basic Process creation and result collection via Queue ────────────────────

def cpu_intensive(n: int, result_queue: Queue) -> None:
    """
    This function runs in a CHILD PROCESS with its own Python interpreter.
    It has its own GIL, so it runs in true parallel with other child processes.
    os.getpid() returns a different PID in every child, confirming isolation.
    """
    pid = os.getpid()   # Each child process has a unique process ID from the OS
    total = sum(i * i for i in range(n))   # CPU-bound computation — genuinely parallel!

    # Communicate the result back to the parent via the Queue.
    # The Queue object uses OS pipes internally; data is pickled for transmission.
    result_queue.put((pid, total))


if __name__ == "__main__":
    # Queue is a process-safe FIFO backed by OS pipes and a background thread.
    q = Queue()
    processes = []

    # Spawn 4 child processes — on a 4-core machine, all run simultaneously
    for i in range(4):
        # IMPORTANT: target function and args must be picklable under 'spawn' mode.
        # Module-level functions (like cpu_intensive) are always picklable.
        p = Process(target=cpu_intensive, args=(1_000_000, q))
        processes.append(p)
        p.start()    # Fork/spawn the child — returns immediately to the parent

    # IMPORTANT: Join BEFORE draining a large queue to avoid the pipe filling up
    # and blocking the child (which would cause deadlock).
    # For small queues (< ~100 items), join first is safe.
    for p in processes:
        p.join()     # Block the parent until this child exits

    # Now safely drain the results
    while not q.empty():
        pid, total = q.get()
        print(f"PID {pid}: result = {total}")
```

### 3.4 Inter-Process Communication (IPC)

Since processes don’t share memory, Python’s `multiprocessing` module provides several IPC mechanisms, each representing a different point on the speed/flexibility tradeoff spectrum.

**`multiprocessing.Queue`** is a process-safe FIFO queue backed by OS pipes, with a background thread in each process managing serialization. It’s the most convenient and the right default for result collection and task distribution. The important caveat: if you put large objects into a queue that fills its OS pipe buffer, the `put()` call will block. Always drain queues before joining the processes that fill them, or use a separate drainer thread.

**`multiprocessing.Pipe()`** creates a low-level duplex channel and returns a pair of `Connection` objects. It’s faster than Queue for point-to-point communication because there’s no background thread or internal locking overhead. Use it when only two processes need to communicate and you want maximum throughput.

**`multiprocessing.Value` and `multiprocessing.Array`** allocate a primitive type in shared memory (backed by `mmap`) that multiple processes can read and write directly. You must provide a `multiprocessing.Lock` for safe access. These are useful for simple counters or flags shared across workers.

**`multiprocessing.shared_memory.SharedMemory`** (Python 3.8+) allocates a raw buffer in shared memory that any process can attach to by name. Combined with `numpy.ndarray`, this enables true zero-copy data sharing between processes — no pickling overhead for large arrays.

**`multiprocessing.Manager`** creates a server process that hosts Python objects (lists, dicts, Namespaces, Queues, etc.) and provides proxy objects. Any process can access these objects via RPC through the proxy. It’s the most flexible but slowest option — every attribute access is a round-trip IPC call through a socket.

```python
from multiprocessing import Process, shared_memory
import numpy as np

# ── SharedMemory: zero-copy NumPy array sharing between processes ─────────────

if __name__ == "__main__":
    # Step 1: Parent creates a named shared memory block
    shm = shared_memory.SharedMemory(create=True, size=4 * 256 * 1024)  # 1 MB for float32

    # Step 2: Attach a NumPy array as a VIEW onto the shared buffer — no copy!
    arr = np.ndarray((256, 1024), dtype=np.float32, buffer=shm.buf)
    arr[:] = 0.0   # Initialize all elements to zero in the shared region

    def worker(shm_name: str) -> None:
        """Child process: attaches to the EXISTING shared memory by name."""
        # create=False means we're attaching to an existing block, not creating
        existing = shared_memory.SharedMemory(name=shm_name, create=False)
        # Attach a NumPy view — no copy! Both the parent and child view the SAME bytes.
        child_arr = np.ndarray((256, 1024), dtype=np.float32, buffer=existing.buf)
        child_arr[:] = 42.0   # This write is IMMEDIATELY visible in the parent!
        existing.close()      # Detach from shared memory (do NOT unlink from child)

    p = Process(target=worker, args=(shm.name,))
    p.start()
    p.join()

    print(arr[0, 0])   # Prints 42.0 — child's write is visible with no IPC call!

    # CRITICAL cleanup: close detaches; unlink destroys the underlying OS resource.
    # If you forget unlink(), the shared memory block leaks in /dev/shm
    # (on Linux) until the system reboots.
    shm.close()
    shm.unlink()    # Always call unlink() exactly ONCE, in the process that created it
```

### 3.5 ProcessPoolExecutor: The High-Level API

`concurrent.futures.ProcessPoolExecutor` is the recommended high-level API for multiprocessing. It manages a pool of worker processes, handles pickling, and returns `Future` objects with the same interface as `ThreadPoolExecutor`.

```python
from concurrent.futures import ProcessPoolExecutor, as_completed
import math
import time

def is_prime(n: int) -> bool:
    """
    CPU-bound primality test. When submitted to ProcessPoolExecutor, this runs
    in a separate OS process with its own GIL — true parallelism for pure Python.
    This function MUST be defined at module level (not as a lambda or nested
    function) because it needs to be picklable for transmission to worker processes.
    """
    if n < 2:
        return False
    for i in range(2, int(math.sqrt(n)) + 1):
        if n % i == 0:
            return False
    return True

CANDIDATES = list(range(10_000_000, 10_000_200))   # 200 large numbers to test

if __name__ == "__main__":
    # Sequential baseline
    start = time.perf_counter()
    sequential = [is_prime(n) for n in CANDIDATES]
    print(f"Sequential: {time.perf_counter() - start:.3f}s")

    # ProcessPoolExecutor: distributes tasks across worker processes.
    # max_workers=None defaults to os.cpu_count(), which is optimal for CPU-bound tasks.
    start = time.perf_counter()
    with ProcessPoolExecutor(max_workers=4) as executor:
        # chunksize > 1 amortizes the per-task IPC overhead.
        # Instead of pickling each integer individually, we send batches.
        # Rule of thumb: chunksize = len(iterable) // (max_workers * 4)
        results = list(executor.map(is_prime, CANDIDATES, chunksize=25))
    print(f"ProcessPool: {time.perf_counter() - start:.3f}s")   # ~3-4x faster on 4 cores

    primes = [n for n, p in zip(CANDIDATES, results) if p]
    print(f"Primes found: {primes}")
```

### 3.6 Danger Zone: Multiprocessing Pitfalls

> #### ⚠ Danger: Zombie Processes
> 
> A zombie process is a child that has exited but whose exit status has not been collected by the parent via `wait()` or `join()`. The process has finished its work and freed its memory, but its entry in the kernel’s process table persists, consuming a small amount of kernel resources. In Python, forgetting to call `p.join()` leaves zombie entries indefinitely. On systems spawning thousands of processes per hour, this can exhaust the kernel’s PID table, causing `OSError: [Errno 11] Resource temporarily unavailable` when trying to create new processes.
> 
> **Fix:** Always call `p.join()` after `p.start()`, or use `with Pool()` / `with ProcessPoolExecutor()` context managers, which automatically call `terminate()` and `join()` on every worker when the block exits.

> #### ⚠ Danger: Pickling Failures in ProcessPool
> 
> `ProcessPoolExecutor` (and `multiprocessing.Pool`) communicate with workers by pickling the function and its arguments. Lambda functions, locally-defined functions (closures), objects containing `threading.Lock`, open file handles, database connections, and most generator objects **cannot be pickled**, causing `PicklingError` or `AttributeError` at runtime — not at definition time, which makes the bug hard to discover.
> 
> **Fix:** Always define worker functions at module level (top-level, not nested inside other functions). If you need lambda behavior, use `functools.partial` with a named function. For environments that genuinely require closure or lambda pickling (e.g., interactive notebooks), use the `cloudpickle` or `pathos` libraries.

> #### ⚠ Danger: Fork + Threads = Deadlock
> 
> If you use `multiprocessing` with the `fork` start method in a parent process that also uses threads, you risk deadlocks in the child. The child inherits the parent’s memory, including any locks held by threads that don’t exist in the child. If a background thread in the parent was holding `malloc`‘s internal lock during the fork, the child’s `malloc` will deadlock the first time it tries to allocate memory. On macOS, this was common enough that Python 3.12 switched the default start method to `spawn`. On Linux, be aware of this risk and prefer `forkserver` or `spawn` in multi-threaded programs.

-----

## Module 4: Asynchronous I/O (asyncio)

### 4.1 The Concurrency Model: Cooperative Multitasking

`asyncio` implements **cooperative multitasking** within a single OS thread. Unlike the preemptive scheduling of OS threads (where the kernel can interrupt any thread at any time via a hardware timer), asyncio tasks voluntarily yield control by executing an `await` expression. No `await` means no yield — the task holds the event loop and all other tasks are frozen until it returns.

This single-threaded model has profound implications. There is no GIL contention (there’s only one thread). There are no race conditions on Python objects — since only one coroutine runs at a time and it can only be interrupted at explicit `await` points, you always have atomic access to local state between any two awaits. There is no context-switch overhead from the OS. The trade-off: a single blocking call — a synchronous `requests.get()`, an unconstrained CPU loop, even `time.sleep()` — freezes the *entire event loop*, halting every other task. This is the most important property to internalize about asyncio.

The restaurant analogy makes cooperative multitasking tangible. A single waiter (the event loop) manages the entire dining room. When a customer (a coroutine) places an order, the waiter writes it down (registers the I/O interest with the OS), then immediately moves to the next table. When the kitchen (the OS/hardware) signals that order #1 is ready, the waiter delivers it. The waiter is never blocked waiting at a table — they are always moving. But if a customer insists on having a conversation that takes 10 minutes (a blocking CPU task), the waiter is stuck and everyone else waits.

### 4.2 The Event Loop: Under the Hood

The asyncio event loop is a **run-to-completion scheduler** implemented in pure Python (with optional C acceleration via `uvloop`) that operates in a tight polling cycle:

1. Run all **ready callbacks** — coroutines that have been resumed and are ready to execute to their next `await` point.
1. Collect **I/O events** from the OS using the platform’s best I/O multiplexer: `epoll` on Linux, `kqueue` on macOS/BSD, `IOCP` on Windows.
1. Schedule callbacks for any file descriptors that have become readable or writable.
1. Run any **timed callbacks** (from `asyncio.sleep()`, `call_later()`, etc.) that are now due.
1. Repeat indefinitely until the loop is explicitly stopped.

At the OS level, asyncio uses **non-blocking I/O multiplexing**. When you `await socket.recv()`, asyncio does not block waiting for data. Instead, it registers the socket’s file descriptor with `epoll` (or equivalent) and suspends the coroutine. The event loop then processes other ready tasks. When `epoll` reports that data has arrived on that file descriptor, the event loop wakes the suspended coroutine and schedules it to continue from exactly where it paused. No threads are created, no stacks are allocated beyond the one per coroutine frame.

This model is why asyncio can handle hundreds of thousands of concurrent connections that would exhaust a system’s memory or file-descriptor limits if implemented with threads. A coroutine frame — the suspended state of an `async def` function — costs only a few hundred bytes on the heap, compared to 64 KB–8 MB for an OS thread stack.

### 4.3 Coroutines, Tasks, and Awaitables

An **awaitable** is any object that can appear after `await`. Python recognizes three kinds: coroutines, Tasks, and Futures.

A **coroutine** is the object returned by calling an `async def` function. Calling `my_coroutine()` does *not* execute it — it returns a coroutine object that has not yet started. This is a common source of bugs. To actually run it, you either `await` it (which runs it sequentially from the current coroutine) or wrap it in a `Task`.

A **Task** wraps a coroutine and schedules it on the event loop immediately. Creating multiple Tasks allows them to run concurrently — the event loop interleaves them at each `await` point. This is the central mechanism for async concurrency: not `await`ing sequentially, but creating Tasks and gathering them.

A **Future** is a low-level awaitable representing a computation that will produce a result at some point. `asyncio.Task` is a subclass of `Future`. You rarely create raw `Future` objects in application code, but you interact with them through `asyncio.gather()` and similar APIs.

### 4.4 Annotated Code: Core asyncio Patterns

```python
import asyncio
import aiohttp   # pip install aiohttp — async-native HTTP client
import time

# ── CORE PATTERN: concurrent HTTP requests with asyncio.gather ────────────────

async def fetch_url(session: aiohttp.ClientSession, url: str, idx: int) -> tuple:
    """
    A coroutine function. The 'await' keywords are the cooperative yield points —
    they tell the event loop "I'm waiting for I/O; go run someone else."
    The event loop can interleave this coroutine with others at each 'await'.
    """
    # 'async with' calls __aenter__ and __aexit__ as coroutines.
    # session.get() returns a response object; the context manager closes it.
    async with session.get(url) as response:
        # 'await response.text()' suspends this coroutine while the body is read.
        # Other tasks run during this network I/O wait.
        body = await response.text()
    return (idx, response.status, len(body))


async def main() -> None:
    urls = ["https://httpbin.org/delay/1"] * 5   # 5 URLs, each ~1 second response

    # aiohttp.ClientSession manages an async connection pool.
    # CRITICAL: Create ONE session and reuse it — creating a session per request
    # leaks file descriptors and connection resources.
    async with aiohttp.ClientSession() as session:

        # ── WRONG: sequential awaiting — runs tasks one after another ──────────
        # This takes ~5 seconds because each await pauses the loop until complete.
        # results = [await fetch_url(session, url, i) for i, url in enumerate(urls)]

        # ── CORRECT: asyncio.create_task() starts all tasks concurrently ───────
        # create_task() schedules the coroutine on the event loop IMMEDIATELY.
        # By the time we've created all 5 tasks, all 5 HTTP requests are in flight.
        tasks = [
            asyncio.create_task(fetch_url(session, url, i))
            for i, url in enumerate(urls)
        ]

        # gather() awaits ALL tasks and returns results in CREATION ORDER,
        # regardless of which task finished first.
        # If any task raises an exception, gather() cancels the others by default
        # (when return_exceptions=False, the default).
        results = await asyncio.gather(*tasks)

        for r in results:
            print(r)   # All 5 results arrive in ~1 second total!


# asyncio.run() is the correct entry point for asyncio programs in Python 3.7+.
# It creates a new event loop, runs main() to completion, then closes the loop
# and cleans up all remaining tasks. NEVER call asyncio.run() from inside
# a running event loop — this raises RuntimeError.
asyncio.run(main())
```

### 4.5 TaskGroup: Structured Concurrency (Python 3.11+)

Python 3.11 introduced `asyncio.TaskGroup`, which implements the **structured concurrency** principle: tasks created within a scope are guaranteed to be finished (or cancelled) before that scope exits. This eliminates an entire class of bugs where tasks are created and accidentally “forgotten,” leading to work being silently dropped or exceptions being swallowed.

```python
import asyncio

async def work(task_id: int, duration: float) -> str:
    """A simple I/O-bound coroutine that simulates a wait."""
    await asyncio.sleep(duration)
    if task_id == 2 and duration < 0.5:
        raise ValueError(f"Task {task_id} encountered a simulated error!")
    return f"Task {task_id} completed after {duration:.1f}s"


async def main_with_taskgroup() -> None:
    # TaskGroup is the idiomatic Python 3.11+ pattern.
    # All tasks created inside 'async with tg:' run concurrently.
    # The block does NOT exit until ALL tasks are done (or at least one raises).
    async with asyncio.TaskGroup() as tg:
        t1 = tg.create_task(work(1, 0.5))
        t2 = tg.create_task(work(2, 0.3))
        t3 = tg.create_task(work(3, 0.8))
    # ← Execution resumes here only after ALL three tasks have finished.
    # All task objects are in a 'done' state at this point.

    print(t1.result())   # "Task 1 completed after 0.5s"
    print(t2.result())   # "Task 2 completed after 0.3s"
    print(t3.result())   # "Task 3 completed after 0.8s"


async def main_with_error() -> None:
    """
    Demonstrates TaskGroup error propagation.
    If ANY task raises, TaskGroup cancels all remaining tasks and collects
    all exceptions into an ExceptionGroup (Python 3.11+).
    """
    try:
        async with asyncio.TaskGroup() as tg:
            tg.create_task(work(1, 0.5))
            tg.create_task(work(2, 0.1))  # This will raise ValueError
            tg.create_task(work(3, 0.9))  # Will be cancelled when t2 raises
    except* ValueError as eg:
        # 'except*' is the new Python 3.11 syntax for handling ExceptionGroups.
        # It allows handling specific exception types from a group.
        for exc in eg.exceptions:
            print(f"Caught: {exc}")


asyncio.run(main_with_taskgroup())
```

### 4.6 async/await Internals: Generator-Based State Machines

Understanding how `async/await` works internally illuminates why it behaves the way it does. When the Python compiler processes an `async def` function, it generates bytecode that turns the function into a **coroutine object** — which is, under the hood, a specialized generator. Each `await` expression in the function body becomes a `YIELD_VALUE` bytecode instruction.

When you `await some_coroutine`, the sequence is: call `some_coroutine.__await__()` to get an iterator, then call `next()` on it. If the coroutine reaches an `await asyncio.sleep()` or `await socket.recv()`, it ultimately reaches a raw `Future` object. The Future’s `__await__` method executes `yield self` — yielding the Future up through the call chain to the event loop. The event loop receives the Future, registers the appropriate I/O interest with the OS, stores the coroutine’s frame state (its local variables, which line it was on), and moves on to process other ready tasks.

When the OS signals that the I/O is ready, the event loop calls `future.set_result(value)`, which schedules the coroutine to be resumed. The event loop calls `coroutine.send(value)` on the next iteration of its polling cycle, which resumes execution exactly at the `yield` point inside the `__await__` iterator, returning the value (e.g., the received bytes) to the calling `await` expression.

The beauty of this design is that each suspended coroutine frame — the chunk of memory storing its local variables and resume point — costs only a few hundred bytes on the heap, as opposed to the 64KB–8MB stack that an OS thread requires. This is why asyncio can support hundreds of thousands of concurrent coroutines where threading would run out of memory.

### 4.7 Running Blocking Code Inside asyncio

One of the most common asyncio mistakes is calling synchronous blocking code from inside a coroutine — using the `requests` library instead of `aiohttp`, calling `time.sleep()` instead of `asyncio.sleep()`, or running a CPU-intensive function without offloading it. The solution is `loop.run_in_executor()`.

```python
import asyncio
import time
import requests   # Synchronous — will block the event loop if called directly

# ── WRONG: Blocking the event loop ───────────────────────────────────────────
async def bad_fetch(url: str) -> str:
    # This call enters a blocking socket read in the kernel.
    # The event loop is FROZEN for the entire duration of this call.
    # All other coroutines are stuck. This is the cardinal sin of asyncio.
    response = requests.get(url)  # ← NEVER call blocking functions directly
    return response.text


# ── CORRECT: Offload the blocking call to a thread pool ──────────────────────
async def good_fetch(url: str) -> str:
    loop = asyncio.get_running_loop()   # Get the currently running event loop

    # run_in_executor() submits 'requests.get' to a ThreadPoolExecutor.
    # The event loop AWAITS the result — other coroutines run freely meanwhile.
    # The blocking call happens in a thread; the event loop is never blocked.
    # First argument: None → use the loop's default ThreadPoolExecutor.
    result = await loop.run_in_executor(
        None,          # executor: None = default ThreadPoolExecutor (size: min(32, cpu+4))
        requests.get,  # The blocking callable
        url            # Positional argument passed to requests.get
    )
    return result.text


# ── CORRECT for CPU-bound work: use a ProcessPoolExecutor ────────────────────
from concurrent.futures import ProcessPoolExecutor

def heavy_compute(n: int) -> int:
    """CPU-bound work. Defined at module level so it's picklable."""
    return sum(i * i for i in range(n))

async def run_compute() -> None:
    loop = asyncio.get_running_loop()

    # A ProcessPoolExecutor offloads CPU work to a separate process entirely.
    # This avoids both blocking the event loop AND the GIL.
    # Create the pool ONCE and reuse it — don't create per-call pools.
    with ProcessPoolExecutor(max_workers=4) as pool:
        # The heavy computation runs in a worker process.
        # The event loop awaits the Future — no blocking, no GIL contention.
        result = await loop.run_in_executor(pool, heavy_compute, 1_000_000)
    print(f"Result: {result}")

asyncio.run(run_compute())
```

### 4.8 Danger Zone: asyncio Pitfalls

> #### ⚠ Danger: Nested Event Loops and RuntimeError
> 
> `asyncio.run()` creates a **new** event loop. If you call it from inside a running event loop — inside another coroutine, or in a Jupyter notebook (which runs its own event loop) — you get:
> `RuntimeError: This event loop is already running.`
> 
> **Fix in Jupyter:** `pip install nest_asyncio`, then call `nest_asyncio.apply()` once at the top of your notebook. This monkey-patches asyncio to allow nested event loops.
> 
> **Fix in production code:** Never call `asyncio.run()` from inside async code. Use `asyncio.get_running_loop()` to access the current loop, or `asyncio.get_event_loop()` from non-async code in older patterns.

> #### ⚠ Danger: Fire-and-Forget Tasks Silently Swallowing Exceptions
> 
> `asyncio.create_task()` schedules a coroutine but does not await it. If the task raises an exception and no code ever calls `task.result()` or awaits the task, the exception is **silently discarded** — printed to `stderr` only when the Task object is garbage-collected, by which time the context has been completely lost.
> 
> **Fix:** Never truly “fire and forget.” Store task references and await them later, or use `TaskGroup` which propagates exceptions automatically. For background tasks that must not block, add a done callback: `task.add_done_callback(lambda t: t.result())` which will re-raise stored exceptions when the task completes.

> #### ⚠ Danger: Memory Leaks in Long-Running Loops
> 
> Long-running asyncio services accumulate `Task` objects if tasks are created without ever being awaited or explicitly cancelled. Each Task object references its coroutine frame and all its local variables. Over hours of operation, this can silently exhaust memory. Use `asyncio.all_tasks()` to audit live tasks. Similarly, `aiohttp.ClientSession` must be created once and reused across requests. Creating a new session per request leaks file descriptors and connection pool resources that may not be released until the garbage collector runs.

> #### ⚠ Danger: Forgetting to Await a Coroutine
> 
> Calling `my_coroutine()` without `await` returns a coroutine object and does nothing — it doesn’t execute. Python 3.5+ emits a `RuntimeWarning: coroutine 'my_coroutine' was never awaited` warning, but this warning can be easy to miss in busy logs. Always treat the absence of expected side effects as a signal to check for missing `await` keywords.

-----

## Module 5: Comparative Analysis

### 5.1 The Decision Tree: Which Tool for Which Problem?

The most consequential decision in concurrent Python programming is choosing the right concurrency model. The wrong choice can make things actively worse — GIL contention slowing down threads, event loop blockages in asyncio, unnecessary process overhead in multiprocessing.

```
Is your bottleneck I/O or CPU?

  I/O-BOUND (network, disk, database, subprocess):
  ├── Can you use async-native libraries (aiohttp, asyncpg, httpx)?
  │   └── YES → Use asyncio
  │          Best for high concurrency (100s–10,000s of simultaneous connections).
  │          Lowest memory overhead per concurrent operation.
  └── Must use synchronous libraries (requests, psycopg2, boto3)?
      └── YES → Use ThreadPoolExecutor
             Threads release the GIL during blocking syscalls.
             Simpler to add to existing synchronous codebases.

  CPU-BOUND (math, compression, image processing, ML in pure Python):
  ├── Does your library release the GIL? (NumPy, SciPy, OpenCV, Pillow?)
  │   └── YES → ThreadPoolExecutor is sufficient.
  │          The actual computation happens in C with the GIL released.
  └── Pure Python CPU work (parsing, algorithms, business logic):
      └── Use ProcessPoolExecutor or multiprocessing.Pool
             Each worker process has its own GIL → true parallelism.
             Memory cost: ~50MB per worker; startup cost: ~100ms.

  MIXED (I/O coordination + CPU computation):
  └── asyncio for I/O orchestration
      + loop.run_in_executor(ProcessPoolExecutor) for CPU-bound tasks
      Best of both worlds: non-blocking I/O + true multi-core computation.
```

### 5.2 Performance: I/O-Bound Tasks

This table summarizes the expected wall-clock time and scalability for 10 tasks each requiring 1 second of I/O wait.

|Method                     |10 tasks × 1s I/O|Scalability            |Memory per task      |Notes                             |
|---------------------------|-----------------|-----------------------|---------------------|----------------------------------|
|Sequential (single-thread) |~10 seconds      |O(n)                   |Negligible           |Baseline                          |
|`threading.Thread` (manual)|~1 second        |Good to ~100 threads   |64KB–8MB stack       |Thread creation overhead per task |
|`ThreadPoolExecutor`       |~1 second        |Good to ~500 workers   |64KB–8MB stack       |Pool reuse reduces creation cost  |
|`asyncio` + `aiohttp`      |~1 second        |Excellent to 10,000+   |~2–5 KB per coroutine|Best for high-concurrency I/O     |
|`multiprocessing.Pool`     |~1 second        |Poor (process overhead)|~50MB per worker     |Overkill — processes aren’t needed|

### 5.3 Performance: CPU-Bound Tasks

This table summarizes the expected behavior for 4 CPU-bound tasks on a 4-core machine.

|Method                            |4 CPU tasks on 4 cores|Actual Speedup|Notes                          |
|----------------------------------|----------------------|--------------|-------------------------------|
|Sequential (single-thread)        |~4s baseline          |1×            |Reference                      |
|`threading.Thread`                |~4s or worse          |≤1×           |GIL serializes all execution   |
|`ThreadPoolExecutor`              |~4s or worse          |≤1×           |Same GIL limitation            |
|`multiprocessing.Pool` (4 workers)|~1s                   |~3.5×         |True parallelism; fork overhead|
|`ProcessPoolExecutor` (4 workers) |~1s                   |~3.5×         |Cleaner API; same performance  |
|`asyncio`                         |~4s (blocks loop!)    |<1× (worse!)  |Never use for CPU-bound work   |

### 5.4 Memory and Resource Cost

|Model                    |Per-task memory            |Startup cost|Max recommended concurrent|
|-------------------------|---------------------------|------------|--------------------------|
|`asyncio` coroutine      |~2–5 KB                    |Microseconds|100,000+                  |
|`threading.Thread`       |64KB–8MB (stack)           |0.5–2ms     |500–1,000                 |
|`multiprocessing.Process`|~20–80MB (full interpreter)|50–200ms    |CPU count (4–32)          |

### 5.5 ThreadPoolExecutor vs. ProcessPoolExecutor

|Criterion            |`ThreadPoolExecutor`              |`ProcessPoolExecutor`                            |
|---------------------|----------------------------------|-------------------------------------------------|
|GIL impact           |Serialized for Python bytecode    |No GIL contention — independent interpreters     |
|Best workload        |I/O-bound tasks                   |CPU-bound tasks (pure Python)                    |
|Memory overhead      |Shared — ~0 extra per thread      |Isolated — ~50MB+ per worker process             |
|Data sharing         |Direct reference (zero-copy)      |Must pickle/unpickle (serialization overhead)    |
|Startup cost         |~1ms per thread                   |50–200ms per process                             |
|Crash isolation      |One crash can corrupt shared state|Crash isolated to the worker process             |
|Return values        |Any Python object                 |Must be picklable                                |
|`max_workers` default|`min(32, cpu_count + 4)`          |`os.cpu_count()`                                 |
|`chunksize` parameter|Not applicable                    |Reduces per-task IPC overhead for large iterables|

### 5.6 asyncio vs. ThreadPoolExecutor

|Criterion                 |`asyncio`                                   |`ThreadPoolExecutor`                          |
|--------------------------|--------------------------------------------|----------------------------------------------|
|Concurrency model         |Cooperative (single-threaded)               |Preemptive (multi-threaded)                   |
|GIL impact                |None (single thread)                        |GIL released during I/O syscalls              |
|Memory per concurrent task|~2–5 KB (coroutine frame)                   |64KB–8MB (thread stack)                       |
|Max concurrent tasks      |100,000+ (memory limited)                   |500–1,000 (stack limited)                     |
|Code complexity           |Requires `async/await` throughout call chain|Works with any existing synchronous code      |
|Blocking call risk        |Critical — one block freezes everything     |Low — one blocked thread doesn’t affect others|
|CPU-bound tasks           |Terrible (blocks the event loop)            |Bad (GIL) — use `ProcessPoolExecutor`         |
|Latency overhead          |Sub-millisecond (no context switch)         |1–10ms (OS context switch)                    |
|Library compatibility     |Requires async-native libraries             |Works with any Python library                 |
|Debugging complexity      |More complex (stack traces span event loop) |Familiar threading model                      |

### 5.7 The asyncio + ProcessPool Hybrid Pattern

For real-world production services that need both high-concurrency I/O *and* CPU-intensive computation — a web scraper that downloads pages asynchronously and then parses HTML in parallel, or an API server that handles requests asynchronously but performs heavy ML inference — the optimal architecture combines asyncio with a `ProcessPoolExecutor`.

```python
import asyncio
from concurrent.futures import ProcessPoolExecutor
import aiohttp

# ── Worker function: CPU-bound, runs in a separate process ────────────────────
# Must be at module level for pickling to work under 'spawn' mode.
def parse_and_analyze(data: bytes) -> dict:
    """Expensive CPU work: JSON parsing, hashing, or data analysis."""
    import json, hashlib
    try:
        parsed = json.loads(data)               # CPU: JSON deserialization
    except json.JSONDecodeError:
        parsed = {}
    digest = hashlib.sha256(data).hexdigest()   # CPU: cryptographic hash
    return {"hash": digest, "key_count": len(parsed)}


async def fetch_and_process(
    session: aiohttp.ClientSession,
    url: str,
    pool: ProcessPoolExecutor,
) -> dict:
    """
    Combines async I/O (aiohttp) with CPU parallelism (ProcessPool).
    The event loop is never blocked: it awaits the I/O, then awaits the
    CPU work (which runs in a process and is itself non-blocking to the loop).
    """
    loop = asyncio.get_running_loop()

    # Step 1: Non-blocking async HTTP request
    async with session.get(url) as resp:
        data = await resp.read()   # Returns when all bytes received — non-blocking

    # Step 2: CPU-bound analysis — offloaded to a worker process
    # run_in_executor returns a Future; 'await' suspends this coroutine
    # while the process works, freeing the event loop to handle other tasks.
    result = await loop.run_in_executor(pool, parse_and_analyze, data)
    return result


async def main() -> None:
    urls = ["https://httpbin.org/json"] * 8    # 8 concurrent requests

    # IMPORTANT: Both the ClientSession and the ProcessPoolExecutor are created
    # ONCE and shared. Creating them per-request is a severe performance bug.
    with ProcessPoolExecutor(max_workers=4) as pool:
        async with aiohttp.ClientSession() as session:
            tasks = [
                asyncio.create_task(fetch_and_process(session, url, pool))
                for url in urls
            ]
            results = await asyncio.gather(*tasks)
            for r in results:
                print(r)


if __name__ == "__main__":
    asyncio.run(main())
```

-----

## Module 6: Advanced Topics and Python 3.12/3.13 Features

### 6.1 Sub-Interpreters and PEP 684 (Python 3.12)

Python 3.12 implemented **PEP 684**, which gives each sub-interpreter instance its own independent GIL. Sub-interpreters are isolated Python runtime environments within a single OS process — they have their own module state, their own type objects, their own import system, and now, their own GIL. The new `interpreters` module (available as `_interpreters` in 3.12) provides a Python-level API for creating sub-interpreters and running code in them from threads.

The key architectural advantage over multiprocessing: sub-interpreters share the same OS process, so the OS doesn’t need to allocate a full address space for each one. The overhead is substantially lower than spawning a new process. The key constraint versus threading: sub-interpreters cannot share arbitrary Python objects — they communicate via **channels** that accept only a restricted set of types (strings, bytes, integers, and a few others). This prevents the entire class of race conditions that plague shared-memory threading.

```python
# Python 3.12+ sub-interpreters example
import _interpreters   # Promoted to 'interpreters' in 3.13
import threading

# Create a new interpreter with its OWN GIL
interp_id = _interpreters.create()

results = {}

def run_in_interp(iid, result_dict):
    """Run code in a sub-interpreter from a thread."""
    # Each sub-interpreter runs with its own GIL, so this thread
    # is truly parallel with the main interpreter's GIL.
    _interpreters.run_string(iid,
        "result = sum(i*i for i in range(1_000_000))"
    )
    # Sub-interpreters communicate back via channels (restricted types only)
    # Full channel API available in Python 3.13's 'interpreters' module

# Clean up
_interpreters.destroy(interp_id)
```

### 6.2 Free-Threaded Python 3.13 (PEP 703)

Python 3.13 ships two build variants. The standard build includes the GIL as always. The **free-threaded build** (compiled with `./configure --disable-gil`, distributed as `python3.13t` in some package managers) removes the GIL entirely. Thread safety is maintained through three mechanisms working together: immortalization of singleton objects, biased reference counting for thread-local objects, and deferred reference counting with atomic operations for shared objects.

```python
# Check if you're running a free-threaded Python 3.13 build
import sys

# sys._is_gil_enabled() is available in Python 3.12+
print(sys._is_gil_enabled())   # False in --disable-gil build, True in standard build

# In free-threaded 3.13, CPU-bound threads achieve TRUE multi-core parallelism:
import threading
import time

results = [0] * 4

def cpu_task(idx: int) -> None:
    # In a free-threaded build, four of these running simultaneously will use
    # all four CPU cores, achieving ~4x speedup over sequential execution.
    # CAUTION: The GIL no longer protects you. ALL shared state mutations
    # need explicit locking. Code that appeared "safe enough" under the GIL
    # may now have genuine data races.
    results[idx] = sum(i * i for i in range(5_000_000))

threads = [threading.Thread(target=cpu_task, args=(i,)) for i in range(4)]
start = time.perf_counter()
for t in threads: t.start()
for t in threads: t.join()
elapsed = time.perf_counter() - start
print(f"Free-threaded: {elapsed:.2f}s (vs ~{elapsed * 4:.2f}s sequential)")

# WARNING: In free-threaded mode, audit ALL shared state mutations.
# Use threading.Lock() around any list append, dict update, or counter increment
# that can be reached from multiple threads simultaneously.
```

> #### ⚠ Caution: Free-Threaded Code Requires Thorough Audit
> 
> Code written for standard CPython often has implicit safety from the GIL — operations that appear atomic (short loops, simple attribute reads) are “safe enough” because the GIL’s switch interval makes their window of vulnerability very small in practice. In free-threaded 3.13, this implicit safety disappears entirely. Before adopting free-threaded builds, audit every piece of shared-state code with thread-sanitizer tools (`-fsanitize=thread` in C extensions, `threading.settrace`-based tools in Python) and add explicit locks wherever needed.

### 6.3 Async Generators, Comprehensions, and Context Managers

Python’s `async/await` syntax extends to iterators and context managers, enabling elegant patterns for streaming data and resource management.

```python
import asyncio

# ── Async Generator: yields values with async pauses ─────────────────────────
async def stream_data(source: list, delay: float = 0.05):
    """
    An async generator — an 'async def' function that contains 'yield'.
    Each yield pauses, allows the event loop to run other tasks, then resumes.
    Think of this as a non-blocking data stream: a live feed, paginated API
    response, or file read in chunks.
    """
    for item in source:
        await asyncio.sleep(delay)   # Simulate async I/O between each item
        yield item                   # 'yield' in 'async def' = async generator


async def process_stream() -> None:
    data = list(range(10))

    # 'async for' drives an async generator.
    # It calls __aiter__() once, then __anext__() for each item,
    # which returns an awaitable that the event loop schedules.
    async for value in stream_data(data, delay=0.02):
        print(value, end=" ")
    print()   # Newline after the stream

    # Async list comprehension — 'async for' inside a comprehension
    squared = [v ** 2 async for v in stream_data(data, delay=0.01)]
    print(squared)   # [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]

    # Async dict comprehension and set comprehension work identically
    lookup = {v: v ** 2 async for v in stream_data(range(5), delay=0.01)}
    print(lookup)   # {0: 0, 1: 1, 2: 4, 3: 9, 4: 16}


# ── Async Context Manager: non-blocking resource lifecycle ────────────────────
class AsyncDBConnection:
    """
    Demonstrates an async context manager by implementing __aenter__ and __aexit__.
    These are coroutines, so setup and teardown can include async I/O
    (e.g., TCP connection, TLS handshake, authentication query).
    """
    async def __aenter__(self):
        await asyncio.sleep(0.01)   # Simulate async connection setup
        print("Database connected")
        return self                 # 'as db' in 'async with ... as db' receives this

    async def __aexit__(self, exc_type, exc_val, exc_tb):
        await asyncio.sleep(0.005)  # Simulate async teardown / connection return
        print("Database disconnected")
        return False                # False = do not suppress any raised exception

    async def query(self, sql: str) -> list:
        await asyncio.sleep(0.02)   # Simulate async DB round-trip
        return [{"id": 1, "result": sql}]


async def use_database() -> None:
    # 'async with' calls __aenter__ on entry (non-blocking) and __aexit__ on exit.
    # Compare to 'with open(...)': same cleanup guarantee, but the setup
    # and teardown themselves can perform async I/O.
    async with AsyncDBConnection() as db:
        rows = await db.query("SELECT * FROM users LIMIT 10")
        print(rows)


asyncio.run(process_stream())
asyncio.run(use_database())
```

### 6.4 asyncio.Semaphore for Rate Limiting

Without rate limiting, `asyncio.gather(1000_tasks)` fires 1,000 HTTP requests simultaneously — potentially triggering rate limit responses (HTTP 429), overwhelming a downstream service, or exhausting local connection pool limits. `asyncio.Semaphore` is the canonical solution.

```python
import asyncio
import aiohttp
import time

async def fetch_with_limit(
    sem: asyncio.Semaphore,
    session: aiohttp.ClientSession,
    url: str,
    idx: int,
) -> tuple:
    """
    Wraps a fetch operation in a semaphore. The semaphore ensures that at most
    N coroutines are inside the 'async with sem:' block simultaneously.
    Others are suspended at the 'await sem.acquire()' point until a slot opens.
    This is cooperative rate-limiting — no spinning, no busy-waiting.
    """
    async with sem:   # Blocks (suspends) this coroutine if N slots are taken
        async with session.get(url) as resp:
            body = await resp.text()
        return (idx, resp.status, len(body))
    # sem is released automatically when 'async with' exits, allowing
    # the next waiting coroutine to proceed.


async def main() -> None:
    # Create 20 tasks but allow at most 5 to run concurrently.
    # This respects API rate limits while still achieving significant parallelism.
    sem = asyncio.Semaphore(5)    # Max 5 concurrent connections at any time
    urls = ["https://httpbin.org/get"] * 20

    start = time.perf_counter()
    async with aiohttp.ClientSession() as session:
        tasks = [
            asyncio.create_task(fetch_with_limit(sem, session, url, i))
            for i, url in enumerate(urls)
        ]
        results = await asyncio.gather(*tasks)

    elapsed = time.perf_counter() - start
    # With semaphore(5), 20 tasks complete in ~4 batches × 1s = ~4s
    # Without semaphore, 20 tasks would complete in ~1s but might hit rate limits
    print(f"Completed {len(results)} requests in {elapsed:.2f}s")
    print(f"Status codes: {set(r[1] for r in results)}")


asyncio.run(main())
```

-----

## Module 7: Quick-Reference Summary Tables

### 7.1 Master Comparison: All Three Models

|Property                  |`threading`                            |`multiprocessing`                    |`asyncio`                            |
|--------------------------|---------------------------------------|-------------------------------------|-------------------------------------|
|Concurrency type          |Preemptive (OS threads)                |True parallelism (separate processes)|Cooperative (event loop)             |
|GIL bypassed?             |No (for Python bytecode)               |Yes — each process has its own GIL   |N/A — single thread, no GIL issue    |
|Best workload             |I/O-bound tasks                        |CPU-bound tasks (pure Python)        |I/O-bound, high-concurrency          |
|Memory sharing            |Direct (shared address space)          |Isolated — explicit IPC required     |Direct (single thread)               |
|Data passing              |Direct reference (zero-copy)           |Pickle via Queue/Pipe                |Direct reference (zero-copy)         |
|Startup cost per unit     |~1ms                                   |~50–200ms                            |~microseconds                        |
|Memory per unit           |64KB–8MB (stack)                       |~20–80MB (full interpreter)          |~2–5 KB (coroutine frame)            |
|Max recommended concurrent|500–1,000                              |CPU count (4–32)                     |100,000+                             |
|Error isolation           |Shared — one crash affects all         |Process-level isolation              |Task-level (loop keeps running)      |
|Blocking call risk        |Low (GIL released on I/O)              |None (separate process)              |**Critical** (blocks entire loop)    |
|Python 3.12 news          |PEP 684 sub-interpreters               |No major changes                     |`TaskGroup`, performance improvements|
|Python 3.13 news          |PEP 703 free-threading (experimental)  |No major changes                     |Faster internal event loop           |
|Debugging complexity      |Moderate (thread-aware debugger needed)|Moderate (cross-process tracing)     |Higher (async stack traces)          |

### 7.2 Synchronization Primitives Reference

|Primitive          |Module                         |Thread-safe?  |Process-safe?|Use Case                                    |
|-------------------|-------------------------------|--------------|-------------|--------------------------------------------|
|`Lock`             |`threading` / `multiprocessing`|✓             |✓ (mp.Lock)  |Protect any shared mutable state            |
|`RLock`            |`threading`                    |✓             |✗            |Recursive functions; same-thread re-entry   |
|`Semaphore`        |`threading` / `asyncio` / `mp` |✓             |✓ (mp)       |Connection pools, rate limiting             |
|`Event`            |`threading` / `asyncio` / `mp` |✓             |✓ (mp)       |Start/ready signals, one-time notifications |
|`Condition`        |`threading`                    |✓             |✗            |Producer-consumer coordination              |
|`Barrier`          |`threading`                    |✓             |✗            |Phase synchronization in parallel algorithms|
|`Queue`            |`queue` / `asyncio` / `mp`     |✓             |✓ (mp.Queue) |Task distribution, result collection        |
|`asyncio.Lock`     |`asyncio`                      |Coroutine-safe|✗            |Protecting shared state in async code       |
|`asyncio.Semaphore`|`asyncio`                      |Coroutine-safe|✗            |Rate limiting concurrent coroutines         |
|`asyncio.Event`    |`asyncio`                      |Coroutine-safe|✗            |Signaling between coroutines                |

### 7.3 When to Use What: The One-Page Cheat Sheet

```
TASK TYPE          TOOL                      WHY
─────────────────────────────────────────────────────────────────────
I/O bound,         asyncio                   Lowest overhead; scales to
high concurrency   + async-native libs       100,000+ concurrent tasks.

I/O bound,         ThreadPoolExecutor        Threads release GIL on I/O.
existing sync code (use sync libs as-is)     No code rewrite required.

CPU bound,         ProcessPoolExecutor       Each process = own GIL.
pure Python        or mp.Pool                True multi-core parallelism.

CPU bound,         ThreadPoolExecutor        NumPy/SciPy/OpenCV release
C extension        (threading is fine)       GIL around their C code.

Mixed: I/O +       asyncio (I/O layer)       Async event loop handles
CPU intensive      + ProcessPool (CPU)       concurrency; processes handle
                   via run_in_executor()     parallelism. Best of both.

One-off scripts    concurrent.futures        Unified API for both Thread
                   (either executor)         and Process pools. Simple.

Python 3.13+,      threading with            Free-threaded build removes
experimental       --disable-gil build       GIL. Audit all shared state.

Parallel Python    _interpreters / interps   Own GIL per interpreter.
in one process     (3.12+ PEP 684)          Lower overhead than processes.
```

-----

> **Final Principle:** Concurrency is not inherently faster than sequential code. It is faster *only* when the bottleneck is waiting — waiting for I/O, waiting for a network response, waiting for a database. If your program’s bottleneck is the CPU, only multiprocessing (or a GIL-releasing C extension) will help. The most common performance mistake in Python is reaching for threads when the real solution is either to profile first, use a more efficient algorithm, or move the bottleneck to a C extension. Profile before you parallelize.

