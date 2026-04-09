# The Complete Guide to Python Multithreading: Architecting for Concurrency

> *Written from first principles. Every concept is grounded in a real-world scenario. Every line of code is explained.*

-----

## Table of Contents

1. [What Problem Are We Actually Solving?](#1-what-problem-are-we-actually-solving)
1. [The GIL: The Single Microphone Rule](#2-the-gil-the-single-microphone-rule)
1. [Concurrency vs. Parallelism: The Two Kitchens](#3-concurrency-vs-parallelism-the-two-kitchens)
1. [The Syntax Toolkit](#4-the-syntax-toolkit)
- [Manual Thread Control: The Warehouse Worker](#41-manual-thread-control-the-warehouse-worker)
- [Daemon Threads: The Background Janitors](#42-daemon-threads-the-background-janitors)
- [ThreadPoolExecutor: The Bank Task Queue](#43-threadpoolexecutor-the-bank-task-queue)
1. [Synchronization Primitives](#5-synchronization-primitives)
- [Lock: The Restroom Key](#51-lock-the-restroom-key)
- [RLock: The Nested Door Problem](#52-rlock-the-nested-door-problem)
- [Event: The Race Starter Pistol](#53-event-the-race-starter-pistol)
- [Semaphore: The Nightclub Bouncer](#54-semaphore-the-nightclub-bouncer)
- [Queue: The Ticket Window](#55-queue-the-thread-safe-ticket-window)
1. [Nested Threading: Manager and Worker Threads](#6-nested-threading-manager-and-worker-threads)
1. [The Dark Side: Pitfalls 🛑](#7-the-dark-side-pitfalls-)
- [Race Conditions: The Shared Bank Account](#71-race-conditions-the-shared-bank-account)
- [Deadlocks: The Four-Way Stop](#72-deadlocks-the-four-way-stop)
- [Memory Leaks: The Hoarder](#73-memory-leaks-the-hoarder)
1. [Memory Profiling with tracemalloc](#8-memory-profiling-with-tracemalloc)
1. [Decision Framework](#9-decision-framework)
1. [Comparison Table: Threading vs. Multiprocessing vs. Asyncio](#10-comparison-table-threading-vs-multiprocessing-vs-asyncio)

-----

## 1. What Problem Are We Actually Solving?

Imagine a **chef in a restaurant kitchen**. A bad chef does one thing at a time:

```
1. Boil pasta.  (Wait 10 minutes, staring at the pot)
2. Make sauce.  (Wait 8 minutes, staring at the pan)
3. Prep salad.  (Wait 5 minutes, staring at the bowl)
Total: 23 minutes for one table's order.
```

A **good chef** is concurrent:

```
1. Start boiling pasta.   (goes on in the background)
2. While pasta boils → make the sauce.
3. While sauce simmers  → prep the salad.
Total: ~10 minutes. Same tasks, smarter execution.
```

**Multithreading is about being the good chef.** It lets your program do useful work *while waiting* — waiting for a web request to return, a file to load, or a database query to finish. These “waiting” periods are called **I/O-bound** tasks, and they are where threading shines.

-----

## 2. The GIL: The Single Microphone Rule

🧠 **Deep Dive: Why Python threads don’t run truly in parallel**

Picture a **podcast studio** with 4 guests (threads) and **only one microphone** (the CPU). No matter how many guests are present, only the person *holding the microphone* can speak at any moment.

This microphone is Python’s **Global Interpreter Lock (GIL)**.

```
┌──────────────────────────────────────────────────────────┐
│                  THE PYTHON RUNTIME                       │
│                                                           │
│  Thread A ──→ [Wants to run] ──→ WAIT for microphone     │
│  Thread B ──→ [Wants to run] ──→ WAIT for microphone     │
│  Thread C ──→ [Currently holds microphone] ──→ EXECUTING │
│  Thread D ──→ [Wants to run] ──→ WAIT for microphone     │
│                                                           │
│  [ GIL = The Single Microphone ]                         │
└──────────────────────────────────────────────────────────┘
```

**The GIL’s critical implication:**

|Task Type                                         |What it does                                             |Does GIL help or hurt?                                  |
|--------------------------------------------------|---------------------------------------------------------|--------------------------------------------------------|
|**I/O-Bound** (web requests, file reads, DB calls)|Thread *releases* the GIL while waiting for external data|✅ Threading helps a lot                                 |
|**CPU-Bound** (math, image processing, encryption)|Thread *holds* the GIL while crunching numbers           |🛑 Threading adds overhead, use `multiprocessing` instead|

**The key insight:** When a thread is waiting for I/O (e.g., waiting for a website to respond), it voluntarily *releases* the GIL. Another thread immediately picks it up. This is how concurrency is achieved even with the GIL.

-----

## 3. Concurrency vs. Parallelism: The Two Kitchens

These terms are often confused. Let’s settle this with two restaurant scenarios:

**Concurrency (Threading):** One chef, multiple dishes. The chef switches between tasks whenever one task is waiting. Only one task is *active* at any microsecond, but multiple tasks are *in progress*.

**Parallelism (Multiprocessing):** Two chefs, each with their own separate kitchen. Both are literally cooking *at the same time*, simultaneously, on different CPUs.

```
CONCURRENCY (Threading)                PARALLELISM (Multiprocessing)
───────────────────────                ────────────────────────────
One CPU core                           Multiple CPU cores

Time →                                 Time →
Chef: [Task A][Task B][Task A][Task B] Chef 1: [Task A][Task A][Task A]
        ↑ switches when A waits         Chef 2: [Task B][Task B][Task B]
                                                  ↑ running simultaneously
```

-----

## 4. The Syntax Toolkit

### 4.1 Manual Thread Control: The Warehouse Worker

**Scenario:** A warehouse manager needs to dispatch 3 workers to scan inventory in 3 different aisles simultaneously. Each worker takes a different amount of time.

```python
import threading  # Import the threading module — our toolbox for creating threads
import time       # Import time so we can simulate work with sleep()

# ─────────────────────────────────────────────────────────────────────
# THE WORKER FUNCTION
# This is the job each warehouse worker (thread) will execute.
# Each worker receives their aisle name and how long scanning takes.
# ─────────────────────────────────────────────────────────────────────
def scan_warehouse_aisle(aisle_name, scan_duration_seconds):
    # Print that this specific worker has started their task
    print(f"[Worker] Starting scan of aisle: {aisle_name}")

    # Simulate the time it takes to scan the aisle (the actual "work")
    time.sleep(scan_duration_seconds)

    # Print that this specific worker has finished their task
    print(f"[Worker] Finished scanning aisle: {aisle_name} (took {scan_duration_seconds}s)")


# ─────────────────────────────────────────────────────────────────────
# MAIN PROGRAM: THE WAREHOUSE MANAGER
# ─────────────────────────────────────────────────────────────────────

# Define the three aisles and how long each one takes to scan
aisles_to_scan = [
    ("Aisle-A (Electronics)", 3),  # (aisle name, scan time in seconds)
    ("Aisle-B (Clothing)",    2),
    ("Aisle-C (Groceries)",   4),
]

# Create an empty list to hold our thread objects so we can manage them later
active_worker_threads = []  # This list is our "roster" of dispatched workers

# Loop over each aisle and create a dedicated thread (worker) for it
for aisle_name, scan_duration in aisles_to_scan:

    # Create a Thread object. 'target' is the function the thread will run.
    # 'args' is a tuple of arguments passed to that function.
    worker_thread = threading.Thread(
        target=scan_warehouse_aisle,   # The function this worker will execute
        args=(aisle_name, scan_duration),  # Arguments passed to the function
        name=f"Worker-{aisle_name}"    # Give each thread a readable name for debugging
    )

    # .start() sends the worker to their aisle — they begin working NOW
    # The main program does NOT wait here; it continues to the next line immediately
    worker_thread.start()

    # Add this running thread to our roster so we can track it
    active_worker_threads.append(worker_thread)

print("[Manager] All workers dispatched. Waiting for them to finish...")

# Loop through our roster and wait for EACH worker to finish
for worker_thread in active_worker_threads:
    # .join() tells the manager (main thread): "Don't move on until this worker is done."
    # Without join(), the manager would leave the building before workers finished.
    worker_thread.join()

# This line only runs AFTER all threads have finished (all join() calls returned)
print("[Manager] All aisles scanned. Inventory check complete! ✅")
```

**Expected Output:**

```
[Worker] Starting scan of aisle: Aisle-A (Electronics)
[Worker] Starting scan of aisle: Aisle-B (Clothing)
[Worker] Starting scan of aisle: Aisle-C (Groceries)
[Manager] All workers dispatched. Waiting for them to finish...
[Worker] Finished scanning aisle: Aisle-B (Clothing) (took 2s)
[Worker] Finished scanning aisle: Aisle-A (Electronics) (took 3s)
[Worker] Finished scanning aisle: Aisle-C (Groceries) (took 4s)
[Manager] All aisles scanned. Inventory check complete! ✅
```

> ✅ **Best Practice:** Always keep a reference list of your threads so you can `.join()` them. An unjoined thread is like a worker you forgot to check on — you don’t know if they finished or got lost.

-----

### 4.2 Daemon Threads: The Background Janitors

**Scenario:** A building has janitors (daemon threads) that clean up while workers are on shift. When the last worker leaves (main program exits), the janitors don’t finish their sweep — they simply leave too. They exist *only to serve* the main program.

```python
import threading  # Import threading to create and manage threads
import time       # Import time to simulate work with sleep()

# ─────────────────────────────────────────────────────────────────────
# THE JANITOR FUNCTION (Daemon Thread)
# This runs forever — but will be killed when the main program exits.
# ─────────────────────────────────────────────────────────────────────
def building_janitor_routine():
    # This loop runs forever, simulating background cleanup
    while True:
        # Print a status message to show the janitor is working
        print("[Janitor] Sweeping the hallways... 🧹")

        # Pause for 2 seconds between sweeps (simulates periodic background work)
        time.sleep(2)


# ─────────────────────────────────────────────────────────────────────
# THE MAIN WORKER FUNCTION (Normal Thread)
# ─────────────────────────────────────────────────────────────────────
def office_worker_shift(worker_name, shift_duration_seconds):
    # Print that the worker has started their shift
    print(f"[Worker] {worker_name} has started their shift.")

    # Simulate the worker doing their job for the duration of the shift
    time.sleep(shift_duration_seconds)

    # Print that the worker is leaving the building
    print(f"[Worker] {worker_name} has finished and is leaving.")


# ─────────────────────────────────────────────────────────────────────
# MAIN PROGRAM: BUILDING MANAGER
# ─────────────────────────────────────────────────────────────────────

# Create the janitor thread — this is the key difference
janitor_thread = threading.Thread(
    target=building_janitor_routine,
    name="JanitorThread"
)

# ✅ Setting daemon=True is the critical step.
# It tells Python: "Kill this thread automatically when the main program exits."
# Without this, the infinite while-loop would keep the program alive FOREVER.
janitor_thread.daemon = True  # Mark as a daemon (background service) thread

# Start the janitor — they begin cleaning in the background immediately
janitor_thread.start()

# Create and start a normal (non-daemon) worker thread
worker_thread = threading.Thread(
    target=office_worker_shift,
    args=("Alice", 5),  # Alice works for 5 seconds
    name="WorkerThread-Alice"
)
worker_thread.start()  # Alice starts her shift

# Wait for Alice (the only non-daemon thread) to finish her shift
worker_thread.join()   # Main program waits here until Alice leaves

# When this line is reached, Alice has left. Program is about to exit.
# Python sees no more non-daemon threads are running, so it kills the janitor
print("[Building] Last worker has left. Building closing. Janitor will stop mid-sweep.")
# The janitor is automatically terminated here — no cleanup needed
```

> 🧠 **When to use daemon threads:** Background monitoring, log flushing, cache refreshing, heartbeat signals. Any task that *supports* the main program but doesn’t need to be completed before exit.

> 🛑 **Pitfall:** Never use daemon threads for tasks that *must* complete — like writing to a database or saving a file. They will be killed mid-operation without warning.

-----

### 4.3 ThreadPoolExecutor: The Bank Task Queue

**Scenario:** A bank has 3 teller windows (worker threads in a pool). Customers (tasks) arrive and take a number. The pool manages dispatching customers to available tellers — you don’t manually manage each teller.

```python
from concurrent.futures import ThreadPoolExecutor, as_completed  # The high-level thread pool API
import time   # To simulate processing time
import random # To simulate variable processing times

# ─────────────────────────────────────────────────────────────────────
# THE TELLER FUNCTION
# This is the work each teller (thread) does for each customer (task).
# ─────────────────────────────────────────────────────────────────────
def bank_teller_process_customer(customer_name):
    # Record how long this customer's transaction will take (random 1-3 seconds)
    transaction_duration = random.uniform(1, 3)

    # Print that the teller has started serving this customer
    print(f"[Teller] Now serving: {customer_name} (will take {transaction_duration:.1f}s)")

    # Simulate the bank transaction taking time
    time.sleep(transaction_duration)

    # Return a result tuple — the caller can inspect this later
    return (customer_name, transaction_duration)


# ─────────────────────────────────────────────────────────────────────
# MAIN PROGRAM: THE BANK MANAGER
# ─────────────────────────────────────────────────────────────────────

# The list of customers waiting to be served today
waiting_customers_list = [
    "Customer-001 (Maria)",
    "Customer-002 (James)",
    "Customer-003 (Sarah)",
    "Customer-004 (David)",
    "Customer-005 (Linda)",
    "Customer-006 (Carlos)",
]

print(f"[Bank] Opening with 3 teller windows. {len(waiting_customers_list)} customers waiting.\n")

# ✅ 'with' statement ensures the pool is properly shut down when done
# max_workers=3 means at most 3 customers are served simultaneously
with ThreadPoolExecutor(max_workers=3, thread_name_prefix="Teller") as teller_pool:

    # Submit all customers to the pool at once.
    # .submit() returns a "Future" object — a promise that the result will be ready later.
    # We store futures in a dict mapping {future: customer_name} for easy lookup later.
    future_to_customer = {
        teller_pool.submit(bank_teller_process_customer, customer): customer
        for customer in waiting_customers_list
    }

    # as_completed() is a generator that yields futures AS THEY FINISH (not in submission order)
    # This lets us react to results immediately rather than waiting for all to complete
    for completed_future in as_completed(future_to_customer):

        # Look up which customer this completed future belongs to
        customer_name = future_to_customer[completed_future]

        # .result() retrieves the return value from bank_teller_process_customer()
        # If an exception occurred inside the thread, .result() will re-raise it here
        served_customer, duration = completed_future.result()

        # Report the completed transaction
        print(f"[Manager] ✅ Transaction complete for {served_customer} ({duration:.1f}s)")

print("\n[Bank] All customers served. Closing for the day.")
```

**Why use ThreadPoolExecutor over manual threads?**

|Feature          |Manual `threading.Thread`     |`ThreadPoolExecutor`        |
|-----------------|------------------------------|----------------------------|
|Thread creation  |You create each one           |Pool manages it             |
|Reuse threads    |No — new thread per task      |✅ Yes — threads are recycled|
|Get return values|Requires shared state or Queue|✅ via `Future.result()`     |
|Handle exceptions|Must wrap each thread manually|✅ `.result()` re-raises them|
|Best for         |Fine-grained control          |✅ Most production use cases |

-----

## 5. Synchronization Primitives

When multiple threads share data, you need *referees*. These are synchronization primitives.

### 5.1 Lock: The Restroom Key

**Scenario:** A restaurant has one shared restroom. Only one person can use it at a time. There’s a physical key on a hook — you grab the key, use the restroom, return the key.

```python
import threading  # For Thread and Lock
import time       # To simulate work with sleep()

# ─────────────────────────────────────────────────────────────────────
# SHARED STATE: The restaurant's cash register
# This is the "critical resource" — accessed by multiple threads (cashiers)
# ─────────────────────────────────────────────────────────────────────
cash_register_balance = 0   # The shared balance — threads will modify this

# Create the Lock object — this is our "Restroom Key"
# There is only ONE lock; whichever thread holds it has exclusive access
cash_register_lock = threading.Lock()


# ─────────────────────────────────────────────────────────────────────
# THE CASHIER FUNCTION
# Each cashier (thread) will process several transactions
# ─────────────────────────────────────────────────────────────────────
def cashier_process_transactions(cashier_name, number_of_transactions, amount_per_transaction):
    # Reference the shared balance variable (we'll modify it, so we use 'global')
    global cash_register_balance

    # Process each transaction one by one
    for transaction_number in range(1, number_of_transactions + 1):

        # ── CRITICAL SECTION START ──────────────────────────────────
        # Grab the 'Restroom Key' (acquire the lock)
        # If another thread holds the lock, THIS LINE BLOCKS until the key is free
        cash_register_lock.acquire()  # Block here until we have exclusive access

        # Now we are the ONLY thread inside this block — safe to modify shared data
        current_balance = cash_register_balance     # Read the current balance
        time.sleep(0.01)                             # Simulate time to process the transaction
        cash_register_balance = current_balance + amount_per_transaction  # Update balance

        # Print the transaction receipt
        print(f"  [{cashier_name}] Transaction #{transaction_number}: "
              f"+${amount_per_transaction} → Balance: ${cash_register_balance}")

        # Release the 'Restroom Key' so the next thread can enter the critical section
        cash_register_lock.release()  # Other threads can now acquire the lock
        # ── CRITICAL SECTION END ────────────────────────────────────


# ─────────────────────────────────────────────────────────────────────
# ✅ BETTER PATTERN: Use 'with' statement for automatic lock release
# This is safer — if an exception occurs, the lock is ALWAYS released
# ─────────────────────────────────────────────────────────────────────
def cashier_process_with_context_manager(cashier_name, number_of_transactions, amount_per_transaction):
    global cash_register_balance

    for transaction_number in range(1, number_of_transactions + 1):

        # 'with lock' automatically calls .acquire() on entry and .release() on exit
        # Even if an exception is raised inside, the lock will be released — no deadlock
        with cash_register_lock:  # This is equivalent to acquire/release but SAFER
            current_balance = cash_register_balance     # Read shared state safely
            time.sleep(0.01)                             # Simulate work
            cash_register_balance = current_balance + amount_per_transaction  # Modify safely


# ─────────────────────────────────────────────────────────────────────
# MAIN: Run two cashiers simultaneously
# ─────────────────────────────────────────────────────────────────────
cashier_alice = threading.Thread(
    target=cashier_process_transactions,
    args=("Alice", 3, 10),  # Alice processes 3 transactions of $10 each
    name="Cashier-Alice"
)
cashier_bob = threading.Thread(
    target=cashier_process_transactions,
    args=("Bob", 3, 15),    # Bob processes 3 transactions of $15 each
    name="Cashier-Bob"
)

cashier_alice.start()  # Alice starts processing
cashier_bob.start()    # Bob starts processing (concurrently)

cashier_alice.join()   # Wait for Alice to finish all her transactions
cashier_bob.join()     # Wait for Bob to finish all his transactions

# Expected: 3*10 + 3*15 = $75
print(f"\n[Register] Final balance: ${cash_register_balance} (Expected: $75)")
```

-----

### 5.2 RLock: The Nested Door Problem

🧠 **Why a regular Lock breaks with recursion**

**Scenario:** A hotel manager holds a master key to the hotel (outer lock). Inside, they need to also unlock the safe (inner lock). But the master key and the safe key are the same physical key. With a regular Lock, the manager locks themselves out the moment they try to use the key a second time. An **RLock (Reentrant Lock)** solves this.

```python
import threading  # For Thread and RLock

# ─────────────────────────────────────────────────────────────────────
# SHARED STATE: Hotel inventory
# ─────────────────────────────────────────────────────────────────────
hotel_inventory = {"room_101": "available", "room_102": "booked", "safe_cash": 5000}

# RLock = Reentrant Lock. The SAME thread can acquire it multiple times.
# A regular Lock would DEADLOCK if the same thread called acquire() twice.
hotel_master_rlock = threading.RLock()


# ─────────────────────────────────────────────────────────────────────
# INNER FUNCTION: Access the hotel safe
# This function also needs the master key (the same RLock)
# ─────────────────────────────────────────────────────────────────────
def access_hotel_safe(manager_name, amount_to_withdraw):
    # Try to acquire the RLock AGAIN (the same thread is calling this)
    # With a regular Lock, this would DEADLOCK (thread waits for itself forever)
    # With RLock, the same thread can re-acquire it — it keeps an internal counter
    with hotel_master_rlock:  # RLock: counter goes from 1 → 2 (same thread, allowed)
        print(f"  [{manager_name}] Opened the safe. Withdrawing ${amount_to_withdraw}.")
        hotel_inventory["safe_cash"] -= amount_to_withdraw  # Safely modify the safe
        print(f"  [{manager_name}] Safe cash remaining: ${hotel_inventory['safe_cash']}")
    # RLock counter goes from 2 → 1 on exit (not fully released yet)


# ─────────────────────────────────────────────────────────────────────
# OUTER FUNCTION: Full hotel room + safe audit
# This acquires the RLock first, then calls a function that acquires it AGAIN
# ─────────────────────────────────────────────────────────────────────
def perform_hotel_audit(manager_name):
    # Acquire the RLock for the first time (counter: 0 → 1)
    with hotel_master_rlock:
        print(f"[{manager_name}] Starting hotel audit. Checking all rooms...")

        # Check each room's status in the inventory
        for room, status in hotel_inventory.items():
            if room != "safe_cash":  # Skip the safe entry in the room loop
                print(f"  [{manager_name}] Room {room}: {status}")

        # Now call access_hotel_safe — which also tries to acquire the SAME RLock
        # This is the "nested door" scenario. With RLock, the same thread passes through.
        access_hotel_safe(manager_name, 200)  # This internally acquires the RLock again

        print(f"[{manager_name}] Audit complete.")
    # RLock counter goes from 1 → 0 here. NOW it is fully released for other threads.


# ─────────────────────────────────────────────────────────────────────
# MAIN: Two managers run audits concurrently (only one can hold the RLock at a time)
# ─────────────────────────────────────────────────────────────────────
manager_alice = threading.Thread(target=perform_hotel_audit, args=("Manager-Alice",))
manager_bob   = threading.Thread(target=perform_hotel_audit, args=("Manager-Bob",))

manager_alice.start()  # Alice starts her audit
manager_bob.start()    # Bob tries to start — but blocks until Alice releases the RLock

manager_alice.join()   # Wait for Alice's full audit to complete
manager_bob.join()     # Wait for Bob's full audit to complete

print(f"\n[Hotel System] Final safe balance: ${hotel_inventory['safe_cash']}")
```

> 🧠 **RLock vs Lock in one sentence:** A regular `Lock` asks “Is anyone inside?” An `RLock` asks “Is anyone *other than me* inside?”

-----

### 5.3 Event: The Race Starter Pistol

**Scenario:** 5 sprinters (threads) are at the starting blocks. They’re all stretched, ready, waiting. The race official fires the starting pistol (sets the Event). All 5 sprint simultaneously.

```python
import threading  # For Thread and Event
import time       # To simulate preparation time

# ─────────────────────────────────────────────────────────────────────
# Create the "Starting Pistol" Event object
# An Event has two states: "not fired" (clear) and "fired" (set)
# Initially it is in the "not fired" state
# ─────────────────────────────────────────────────────────────────────
starting_pistol_event = threading.Event()  # All threads will listen to this event


# ─────────────────────────────────────────────────────────────────────
# THE SPRINTER FUNCTION
# Each sprinter (thread) prepares, then waits for the pistol
# ─────────────────────────────────────────────────────────────────────
def sprinter_prepare_and_run(sprinter_name, preparation_time_seconds):
    # Simulate each sprinter's warm-up routine (takes different amounts of time)
    print(f"[{sprinter_name}] Warming up... (takes {preparation_time_seconds}s)")
    time.sleep(preparation_time_seconds)  # Each sprinter finishes warm-up at different times

    # Sprinter is ready. Now they wait at the blocks for the pistol.
    print(f"[{sprinter_name}] In position. Waiting for the pistol...")

    # .wait() BLOCKS this thread until starting_pistol_event.set() is called
    # All sprinters are blocked here simultaneously, waiting for the same signal
    starting_pistol_event.wait()  # Do not move until the event is "fired"

    # This line executes the MOMENT the event is set (pistol fires)
    print(f"[{sprinter_name}] 🏃 RUNNING!")


# ─────────────────────────────────────────────────────────────────────
# MAIN: The race official coordinates the start
# ─────────────────────────────────────────────────────────────────────

# Define the 5 sprinters and their warm-up durations
sprinters_roster = [
    ("Sprinter-Alice",   1),
    ("Sprinter-Bob",     2),
    ("Sprinter-Carlos",  1),
    ("Sprinter-Diana",   3),
    ("Sprinter-Erik",    2),
]

# Create and start all sprinter threads
sprinter_threads = []  # List to track all sprinter threads
for sprinter_name, prep_time in sprinters_roster:
    sprinter_thread = threading.Thread(
        target=sprinter_prepare_and_run,
        args=(sprinter_name, prep_time),
        name=sprinter_name
    )
    sprinter_thread.start()    # Thread starts, goes through warm-up, then blocks at .wait()
    sprinter_threads.append(sprinter_thread)  # Track this thread for later join()

# Race official waits for everyone to get into position (4 seconds covers all warm-ups)
print("\n[Official] Waiting for all sprinters to be in position...")
time.sleep(4)  # Pause to allow all threads to finish warm-up and reach .wait()

# Fire the starting pistol! — This unblocks ALL waiting threads simultaneously
print("\n[Official] 🔫 BANG! GO GO GO!\n")
starting_pistol_event.set()  # All threads blocked on .wait() are released AT ONCE

# Wait for all sprinters to complete
for sprinter_thread in sprinter_threads:
    sprinter_thread.join()  # Wait for each sprinter thread to finish

print("\n[Official] Race complete.")
```

> ✅ **Use Events when:** You want to *broadcast* a signal to many threads at once. One `event.set()` unblocks *all* waiting threads simultaneously — unlike a Lock, which only lets one thread through at a time.

-----

### 5.4 Semaphore: The Nightclub Bouncer

**Scenario:** A nightclub has a maximum capacity of 3 people. A bouncer tracks the count. When 3 are inside, new arrivals wait outside. When someone leaves, the bouncer lets the next person in.

```python
import threading  # For Thread and Semaphore
import time       # To simulate time inside the club

# ─────────────────────────────────────────────────────────────────────
# A Semaphore is like a Lock, but allows N threads in simultaneously
# Here, our nightclub allows max 3 people at once
# ─────────────────────────────────────────────────────────────────────
nightclub_capacity_semaphore = threading.Semaphore(3)  # Max 3 concurrent "inside" threads


def nightclub_patron_experience(patron_name, time_inside_seconds):
    # Patron arrives and waits for the bouncer to let them in
    # .acquire() decrements the semaphore counter. Blocks when counter hits 0.
    print(f"[{patron_name}] Waiting outside the club...")
    nightclub_capacity_semaphore.acquire()  # "Bouncer lets me in" — or blocks if full

    # Inside the club! (critical section — only 3 threads here at once)
    print(f"[{patron_name}] 🎉 Inside the club! Enjoying for {time_inside_seconds}s.")
    time.sleep(time_inside_seconds)         # Simulate time enjoying the club

    # Patron leaves. Semaphore counter goes up — one more spot available.
    print(f"[{patron_name}] Leaving the club.")
    nightclub_capacity_semaphore.release()  # "Tell bouncer I've left" — opens a spot


# Send 7 patrons to the club; only 3 can be inside at once
patron_list = [("Patron-" + name, duration) for name, duration in [
    ("Alice", 2), ("Bob", 3), ("Carlos", 1),
    ("Diana", 2), ("Erik", 4), ("Fatima", 1), ("George", 3)
]]

patron_threads = []
for patron_name, duration in patron_list:
    t = threading.Thread(target=nightclub_patron_experience, args=(patron_name, duration))
    t.start()             # Patron heads to the club
    patron_threads.append(t)

for t in patron_threads:
    t.join()  # Wait for everyone to finish their night out

print("\n[Club] Closed for the night.")
```

-----

### 5.5 Queue: The Thread-Safe Ticket Window

**Scenario:** A train station has multiple agents selling tickets (producers) and multiple passengers picking up tickets (consumers). They share a common ticket booth (Queue). The Queue handles all the coordination so agents and passengers don’t collide.

```python
import threading   # For Thread creation
import queue       # queue.Queue is a thread-safe FIFO data structure
import time        # To simulate work

# ─────────────────────────────────────────────────────────────────────
# The Queue is the shared "ticket booth" — thread-safe by design
# maxsize=5 means the booth holds at most 5 tickets at once
# (Producers will block if the booth is full)
# ─────────────────────────────────────────────────────────────────────
ticket_booth_queue = queue.Queue(maxsize=5)


def ticket_agent_produce_tickets(agent_name, tickets_to_issue):
    # This agent will issue a set number of tickets into the booth
    for ticket_number in range(1, tickets_to_issue + 1):
        ticket_id = f"TICKET-{agent_name}-{ticket_number:03d}"  # Format: TICKET-Alice-001

        # .put() adds an item to the queue. Blocks if queue is full (booth is full).
        ticket_booth_queue.put(ticket_id)  # Place ticket into the shared booth
        print(f"  [Agent {agent_name}] Issued: {ticket_id}")
        time.sleep(0.2)  # Simulate ticket printing time

    # Signal that this agent is done by placing a special sentinel value
    # The consumer will see "DONE" and know to stop
    ticket_booth_queue.put("DONE")


def passenger_collect_tickets(passenger_name):
    # This passenger will keep collecting tickets until they see the "DONE" signal
    while True:
        # .get() removes and returns an item from the queue. Blocks if queue is empty.
        ticket = ticket_booth_queue.get()  # Wait for a ticket to be available

        # Check if this is the "done" signal from the producer
        if ticket == "DONE":
            print(f"[Passenger {passenger_name}] Received all tickets. Done collecting.")
            # Put the sentinel back so OTHER consumers also see it (if multiple consumers)
            ticket_booth_queue.put("DONE")
            break  # Stop the loop — no more tickets coming

        # Normal ticket received — process it
        print(f"[Passenger {passenger_name}] Collected: {ticket}")
        time.sleep(0.3)   # Simulate validating the ticket

        # CRITICAL: Always call .task_done() after processing a .get() item
        # This decrements the internal counter used by .join() (if used)
        ticket_booth_queue.task_done()


# Create one agent producing 5 tickets and one passenger collecting them
agent_thread     = threading.Thread(target=ticket_agent_produce_tickets, args=("Alice", 5))
passenger_thread = threading.Thread(target=passenger_collect_tickets, args=("Bob",))

agent_thread.start()      # Agent starts issuing tickets
passenger_thread.start()  # Passenger starts collecting

agent_thread.join()       # Wait for agent to finish
passenger_thread.join()   # Wait for passenger to finish

print("[Station] All tickets processed. ✅")
```

-----

## 6. Nested Threading: Manager and Worker Threads

**Scenario:** A logistics company has a **Manager Thread** responsible for coordinating regional operations. The Manager *spawns* **Worker Threads** for each region. The challenge: cleaning up all workers without leaving “zombie” threads behind.

A **zombie thread** is a thread that has finished executing but hasn’t been properly `.join()`-ed — it lingers in memory, consuming resources, like a ghost in the system.

```python
import threading  # For Thread and Event
import time       # To simulate work durations
import logging    # Structured logging — better than print() for production systems

# Configure logging to show thread names — invaluable for debugging multi-threaded apps
logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s [%(threadName)s] %(message)s",  # Include thread name in every log
    datefmt="%H:%M:%S"
)

# ─────────────────────────────────────────────────────────────────────
# WORKER FUNCTION: Regional Delivery Driver
# Each driver (worker thread) runs until the shutdown_event is set
# ─────────────────────────────────────────────────────────────────────
def regional_delivery_driver(region_name, deliveries_count, shutdown_event):
    logging.info(f"Driver started for region: {region_name}")

    # Process each delivery in this region
    for delivery_number in range(1, deliveries_count + 1):

        # Before each delivery, check if a shutdown was requested
        # This is a "cooperative shutdown" pattern — threads check in periodically
        if shutdown_event.is_set():  # Has the manager signaled us to stop?
            logging.info(f"[{region_name}] Shutdown requested. Stopping after {delivery_number-1} deliveries.")
            return  # Exit the function cleanly — thread ends naturally

        # Simulate the delivery taking time
        logging.info(f"[{region_name}] Making delivery #{delivery_number}/{deliveries_count}")
        time.sleep(0.5)  # Each delivery takes 0.5 seconds

    # All deliveries done — log completion before thread ends
    logging.info(f"[{region_name}] All {deliveries_count} deliveries complete. Driver clocking out.")


# ─────────────────────────────────────────────────────────────────────
# MANAGER FUNCTION: Regional Operations Manager
# The Manager Thread is itself spawned by main(), and it spawns workers
# ─────────────────────────────────────────────────────────────────────
def regional_operations_manager(regions_and_deliveries, manager_shutdown_event):
    logging.info("Manager thread started. Spawning regional drivers...")

    # Create a shared shutdown Event — all workers listen to this one event
    worker_shutdown_event = threading.Event()  # Will be used to signal all workers to stop

    # ── SPAWN WORKER THREADS ──────────────────────────────────────────
    active_worker_threads = []  # Track all worker threads for cleanup

    for region_name, delivery_count in regions_and_deliveries:
        # Create a worker thread for each region
        worker_thread = threading.Thread(
            target=regional_delivery_driver,
            args=(region_name, delivery_count, worker_shutdown_event),
            name=f"Driver-{region_name}",  # Descriptive name for logs/debugging
            daemon=False  # Non-daemon: we WANT to join() these threads explicitly
        )
        worker_thread.start()                    # Launch the regional driver
        active_worker_threads.append(worker_thread)  # Track for later cleanup

    logging.info(f"Manager: {len(active_worker_threads)} drivers dispatched.")

    # ── WAIT FOR WORKERS OR RESPOND TO SHUTDOWN ───────────────────────
    # Poll every second to check if: (a) all workers are done, or (b) manager was told to stop
    while any(t.is_alive() for t in active_worker_threads):  # While any worker is still running
        if manager_shutdown_event.is_set():  # Has main() told the manager to stop?
            logging.info("Manager received shutdown signal. Telling all drivers to stop...")
            worker_shutdown_event.set()  # Signal ALL workers to stop their loops
            break  # Exit the polling loop — proceed to cleanup

        time.sleep(1)  # Check status every second (not too aggressive)

    # ── CLEANUP: JOIN ALL WORKER THREADS ─────────────────────────────
    # This is CRITICAL. Not joining = zombie threads.
    for worker_thread in active_worker_threads:
        # .join(timeout=5) waits up to 5 seconds for the thread to finish
        # Without a timeout, a stuck thread would hang the program forever
        worker_thread.join(timeout=5)  # Give each worker up to 5 seconds to finish

        if worker_thread.is_alive():  # Check if the thread STILL hasn't stopped
            # This should not happen in well-written code, but log a warning if it does
            logging.warning(f"⚠️ Thread {worker_thread.name} did not stop in time! Possible zombie.")
        else:
            logging.info(f"✅ Thread {worker_thread.name} joined cleanly. No zombie.")

    logging.info("Manager: All workers cleaned up. Manager exiting.")


# ─────────────────────────────────────────────────────────────────────
# MAIN PROGRAM: Company Headquarters
# ─────────────────────────────────────────────────────────────────────
if __name__ == "__main__":
    # Define the regions and how many deliveries each has
    delivery_regions = [
        ("North-Zone",  4),   # North zone has 4 deliveries
        ("South-Zone",  3),   # South zone has 3 deliveries
        ("East-Zone",   5),   # East zone has 5 deliveries
    ]

    # Create a shutdown event for the manager thread
    manager_shutdown_signal = threading.Event()  # Main() uses this to signal the manager

    # Create the Manager Thread — it will create its own worker threads inside
    manager_thread = threading.Thread(
        target=regional_operations_manager,
        args=(delivery_regions, manager_shutdown_signal),
        name="RegionalManager",
        daemon=False  # Non-daemon: we WILL join() the manager in main
    )
    manager_thread.start()  # Launch the manager thread

    # Main thread waits for the manager to finish all operations
    # timeout=30 prevents hanging forever if something goes wrong
    manager_thread.join(timeout=30)

    if manager_thread.is_alive():  # Manager didn't finish in 30 seconds
        logging.warning("Main: Manager thread timed out. Sending shutdown signal...")
        manager_shutdown_signal.set()  # Tell manager to wrap up
        manager_thread.join(timeout=10)  # Give 10 more seconds

    logging.info("Main: All operations complete. Program exiting cleanly. ✅")
```

**Thread Hierarchy Diagram:**

```
main() [Main Thread]
  │
  └── RegionalManager [Thread]
        │
        ├── Driver-North-Zone [Thread]
        ├── Driver-South-Zone [Thread]
        └── Driver-East-Zone  [Thread]

Shutdown signal flow (top-down):
main() → manager_shutdown_signal → Manager → worker_shutdown_event → All Drivers
```

-----

## 7. The Dark Side: Pitfalls 🛑

### 7.1 Race Conditions: The Shared Bank Account

🛑 **The most dangerous and subtle bug in concurrent programming.**

**Scenario:** Alice and Bob both check a shared bank account with $1000. *At the exact same microsecond*, both see $1000, both decide to withdraw $800. Both succeed. The bank just lost $600.

```python
import threading  # For Thread
import time       # To create the race condition window

# ─────────────────────────────────────────────────────────────────────
# SHARED STATE — the account both people access
# ─────────────────────────────────────────────────────────────────────
shared_account_balance = 1000  # Both Alice and Bob see this value: $1000

# ─────────────────────────────────────────────────────────────────────
# 🛑 BROKEN VERSION — No lock, race condition possible
# ─────────────────────────────────────────────────────────────────────
def withdraw_without_protection(person_name, withdrawal_amount):
    global shared_account_balance

    # Step 1: Read the current balance (Alice sees $1000, Bob sees $1000)
    current_balance = shared_account_balance      # Both threads read $1000

    # ⚠️ THE GAP: Between reading and writing, the OS can SWITCH threads.
    # Alice reads $1000. OS switches to Bob. Bob reads $1000. OS switches back.
    # Now BOTH think they can withdraw $800 from $1000.
    time.sleep(0.001)  # This tiny sleep simulates the OS thread-switching gap

    # Step 2: Check if sufficient funds (both pass this check!)
    if current_balance >= withdrawal_amount:
        # Step 3: Write the new balance (BOTH threads run this line)
        shared_account_balance = current_balance - withdrawal_amount  # RACE CONDITION HERE

        # Both Alice and Bob believe they succeeded — but the math is wrong
        print(f"[{person_name}] Withdrew ${withdrawal_amount}. "
              f"Balance: ${shared_account_balance}")  # Both may show $200
    else:
        print(f"[{person_name}] Withdrawal denied. Insufficient funds.")


# Run the broken version
print("=== 🛑 BROKEN VERSION (Race Condition) ===")
alice_thread = threading.Thread(target=withdraw_without_protection, args=("Alice", 800))
bob_thread   = threading.Thread(target=withdraw_without_protection, args=("Bob",   800))
alice_thread.start()
bob_thread.start()
alice_thread.join()
bob_thread.join()
print(f"Final balance (broken): ${shared_account_balance}")  # Likely shows $200, not $-600!
# Note: in reality, both withdrew $800 from $1000. Someone got away with fraud.


# ─────────────────────────────────────────────────────────────────────
# ✅ FIXED VERSION — Lock eliminates the race condition
# ─────────────────────────────────────────────────────────────────────
shared_account_balance = 1000  # Reset the balance for the fixed demo
account_lock = threading.Lock()  # One lock for the entire account operation

def withdraw_with_protection(person_name, withdrawal_amount):
    global shared_account_balance

    # The entire read-check-write sequence is inside the lock
    # Only ONE thread can execute this block at a time — the race is impossible
    with account_lock:  # Acquire lock: no other thread enters here until we're done
        current_balance = shared_account_balance  # Read — protected
        time.sleep(0.001)                          # Gap — but we're still holding the lock

        if current_balance >= withdrawal_amount:
            shared_account_balance = current_balance - withdrawal_amount  # Write — protected
            print(f"[{person_name}] Withdrew ${withdrawal_amount}. "
                  f"Balance: ${shared_account_balance}")
        else:
            print(f"[{person_name}] Denied. Only ${current_balance} available.")
    # Lock released — next thread may enter


print("\n=== ✅ FIXED VERSION (With Lock) ===")
alice_thread = threading.Thread(target=withdraw_with_protection, args=("Alice", 800))
bob_thread   = threading.Thread(target=withdraw_with_protection, args=("Bob",   800))
alice_thread.start()
bob_thread.start()
alice_thread.join()
bob_thread.join()
print(f"Final balance (fixed): ${shared_account_balance}")  # Correct: $200 or $1000
```

**The Race Condition Timeline:**

```
WITHOUT LOCK:                          WITH LOCK:
──────────────                         ─────────────
Time  Alice           Bob              Alice      Bob
 1    Read: $1000                       Acquire lock
 2                    Read: $1000       Read: $1000
 3    Write: $200                       Write: $200
 4                    Write: $200       Release lock
                                                   Acquire lock
 Result: Bob steals $800!               Read: $200
                                        Denied: $800 > $200
                                        Release lock
                                        Result: Fair ✅
```

-----

### 7.2 Deadlocks: The Four-Way Stop

🛑 **Deadlock:** Two (or more) threads each hold a lock the other needs, and both wait forever.

```python
import threading  # For Thread and Lock
import time       # To make the deadlock more reproducible

# Two resources (locks) both threads need
oven_lock   = threading.Lock()   # The kitchen oven
pantry_lock = threading.Lock()   # The ingredient pantry

# ─────────────────────────────────────────────────────────────────────
# 🛑 DEADLOCK SCENARIO (DO NOT RUN IN PRODUCTION — demonstrates the bug)
# Chef A grabs the oven, then tries to get the pantry.
# Chef B grabs the pantry, then tries to get the oven.
# Both wait forever.
# ─────────────────────────────────────────────────────────────────────
def chef_A_broken():
    with oven_lock:          # Chef A grabs the oven (step 1)
        time.sleep(0.1)      # Small delay — gives Chef B time to grab the pantry
        print("[Chef A] Has oven. Waiting for pantry...")
        with pantry_lock:    # Chef A waits for the pantry — but Chef B holds it!
            print("[Chef A] Got both resources! Cooking...")

def chef_B_broken():
    with pantry_lock:        # Chef B grabs the pantry (step 1)
        time.sleep(0.1)      # Small delay — gives Chef A time to grab the oven
        print("[Chef B] Has pantry. Waiting for oven...")
        with oven_lock:      # Chef B waits for the oven — but Chef A holds it!
            print("[Chef B] Got both resources! Cooking...")

# ─────────────────────────────────────────────────────────────────────
# ✅ FIX: Always acquire locks in the SAME ORDER across ALL threads
# Both chefs agree: always grab oven FIRST, pantry SECOND.
# ─────────────────────────────────────────────────────────────────────
def chef_A_fixed():
    with oven_lock:        # Chef A grabs oven first (agreed order)
        with pantry_lock:  # Then pantry second (agreed order)
            print("[Chef A - Fixed] Cooking with oven and pantry! ✅")

def chef_B_fixed():
    with oven_lock:        # Chef B also grabs oven first (same agreed order!)
        with pantry_lock:  # Then pantry second — will wait if Chef A has it, but NO deadlock
            print("[Chef B - Fixed] Cooking with oven and pantry! ✅")

# Run the fixed version
print("=== ✅ Fixed Deadlock Demo ===")
thread_a = threading.Thread(target=chef_A_fixed, name="ChefA")
thread_b = threading.Thread(target=chef_B_fixed, name="ChefB")
thread_a.start()
thread_b.start()
thread_a.join()
thread_b.join()
print("Both chefs cooked successfully. No deadlock. ✅")
```

> ✅ **Deadlock Prevention Rule:** Always acquire multiple locks in the **same global order** across all threads. If Thread 1 always acquires Lock-A before Lock-B, then Thread 2 must also acquire Lock-A before Lock-B.

-----

### 7.3 Memory Leaks: The Hoarder

🛑 **Memory Leaks in threads happen when references to objects are kept alive longer than needed — the Garbage Collector can’t clean the room because someone is hoarding a reference to everything in it.**

**Memory Map: How a Leak Forms**

```
┌─────────────────────────────────────────────────────────────┐
│  MEMORY HEAP                                                  │
│                                                               │
│  Thread A  ──→  large_data_object (500MB)                    │
│                        │                                      │
│                        └──→  sub_object_B                     │
│                                    │                          │
│                                    └──→  callback_function    │
│                                               │               │
│                                               └──→  Thread A  │ ← CIRCULAR REFERENCE!
│                                                               │
│  🛑 Garbage Collector sees: Thread A is referenced by        │
│     callback, which is referenced by sub_object_B,           │
│     which is referenced by large_data_object,                 │
│     which is referenced by Thread A.                         │
│     Nobody can be cleaned! The room fills up.                │
└─────────────────────────────────────────────────────────────┘
```

```python
import threading   # For Thread creation
import time        # For sleep
import gc          # Python's Garbage Collector — we can ask it to collect manually

# ─────────────────────────────────────────────────────────────────────
# 🛑 LEAKY PATTERN: Thread holds onto large data forever
# ─────────────────────────────────────────────────────────────────────
leaked_results_list = []  # Global list that threads append to — NEVER CLEARED

def leaky_data_processor(job_name):
    # Simulate processing a large chunk of data (10,000 items)
    large_data_result = [f"result_{i}" for i in range(10_000)]  # Creates a large object

    # 🛑 BAD PRACTICE: Appending to a global list that is never pruned
    # The global reference keeps large_data_result alive FOREVER
    # The GC cannot collect it because leaked_results_list still holds a reference
    leaked_results_list.append(large_data_result)  # Reference is now "hoarded" globally

    print(f"[{job_name}] Processed {len(large_data_result)} items. "
          f"(Total stored results: {len(leaked_results_list)})")


# ─────────────────────────────────────────────────────────────────────
# ✅ CLEAN PATTERN: Thread returns its result, no global accumulation
# ─────────────────────────────────────────────────────────────────────
def clean_data_processor(job_name, result_queue):
    # Simulate processing a large chunk of data
    large_data_result = [f"result_{i}" for i in range(10_000)]

    # ✅ GOOD PRACTICE: Put result in a queue, then let the reference go out of scope
    # Once this function returns, large_data_result has no more references
    # The GC CAN collect it when memory pressure demands
    result_queue.put((job_name, len(large_data_result)))  # Only store the COUNT, not the data

    # large_data_result goes out of scope here — eligible for garbage collection ✅
    print(f"[{job_name}] Processed {len(large_data_result)} items. Result stored in queue.")


# Demonstrate the clean pattern
import queue
results_queue = queue.Queue()  # Thread-safe queue for collecting results

clean_threads = []
for job_number in range(1, 4):  # Process 3 jobs
    job_name = f"DataJob-{job_number:02d}"
    t = threading.Thread(target=clean_data_processor, args=(job_name, results_queue))
    t.start()
    clean_threads.append(t)

for t in clean_threads:
    t.join()  # Wait for all jobs to complete

# Drain the results queue
while not results_queue.empty():
    job_name, result_count = results_queue.get()
    print(f"  [Main] Collected result from {job_name}: {result_count} items processed")

# Request garbage collection to clean up any lingering objects
gc.collect()  # Manually trigger GC — useful after heavy thread workloads
print(f"\n✅ Clean pattern complete. No large objects retained in memory.")
```

-----

## 8. Memory Profiling with tracemalloc

🧠 **`tracemalloc` is Python’s built-in memory profiler. It lets you see which thread is consuming the most RAM.**

```python
import threading   # For Thread creation
import tracemalloc # Python's built-in memory allocation tracer
import time        # For simulating work

# ─────────────────────────────────────────────────────────────────────
# Two different threads with very different memory behaviors
# ─────────────────────────────────────────────────────────────────────

def memory_efficient_image_processor(processor_name):
    # This thread processes images one at a time and discards each after use
    for image_number in range(1, 4):  # Process 3 images
        # Simulate loading a small image (10,000 bytes)
        image_data = b"x" * 10_000         # 10KB per image — loaded into memory
        time.sleep(0.05)                    # Simulate processing
        # image_data goes out of scope here — GC eligible
        del image_data                      # Explicitly release the reference immediately
    print(f"[{processor_name}] Efficient processing complete.")


def memory_hungry_report_builder(builder_name):
    # This thread loads ALL reports into memory at once before processing
    all_reports_accumulated = []  # 🛑 Accumulates everything — never shrinks

    for report_number in range(1, 4):  # Load 3 reports
        # Simulate loading a large report (500,000 bytes = ~0.5MB each)
        report_data = b"R" * 500_000       # 500KB per report — all kept in memory!
        all_reports_accumulated.append(report_data)  # Hoard all reports in a list
        time.sleep(0.05)

    # Only now do we process — but we're holding 1.5MB unnecessarily
    print(f"[{builder_name}] Hungry processing complete. "
          f"Peak memory held: ~{len(all_reports_accumulated) * 500_000 / 1024:.0f} KB")


# ─────────────────────────────────────────────────────────────────────
# PROFILE WITH tracemalloc
# ─────────────────────────────────────────────────────────────────────

# Start tracing memory allocations RIGHT NOW
tracemalloc.start()  # All memory allocs after this line are tracked

# Take a "before" snapshot to establish our baseline
snapshot_before = tracemalloc.take_snapshot()  # Capture current memory state

# Run both threads
efficient_thread = threading.Thread(
    target=memory_efficient_image_processor,
    args=("EfficientProcessor",),
    name="EfficientThread"
)
hungry_thread = threading.Thread(
    target=memory_hungry_report_builder,
    args=("HungryReportBuilder",),
    name="HungryThread"
)

efficient_thread.start()  # Start the efficient processor
hungry_thread.start()     # Start the memory-hungry builder

efficient_thread.join()   # Wait for efficient processor to finish
hungry_thread.join()      # Wait for hungry builder to finish

# Take an "after" snapshot to see what changed
snapshot_after = tracemalloc.take_snapshot()  # Capture memory state after threads ran

# ── ANALYZE THE DIFFERENCE ─────────────────────────────────────────
# Compare the two snapshots to find what grew the most
top_memory_consumers = snapshot_after.compare_to(
    snapshot_before,       # Compare to our "before" snapshot
    key_type="lineno"      # Group by source line number for precise attribution
)

print("\n" + "="*60)
print("📊 TOP MEMORY CONSUMERS (lines that allocated most RAM):")
print("="*60)

# Show the top 5 lines that consumed the most memory
for rank, memory_stat in enumerate(top_memory_consumers[:5], start=1):
    # Each stat shows: size_delta (how much new memory this line added)
    size_kb = memory_stat.size_diff / 1024  # Convert bytes to kilobytes
    print(f"  #{rank}: {size_kb:+.1f} KB | {memory_stat.traceback.format()[0].strip()}")

# Get the peak memory usage recorded during execution
current_memory_bytes, peak_memory_bytes = tracemalloc.get_traced_memory()
print(f"\n  📈 Peak memory used during run: {peak_memory_bytes / 1024:.1f} KB")

# Stop the memory tracer — frees tracer overhead
tracemalloc.stop()  # Always stop tracing when done — it has overhead

print("\n✅ Memory profiling complete.")
```

-----

## 9. Decision Framework

Use this flowchart to choose the right concurrency tool:

```
START: Do I have slow code I want to speed up?
│
├─→ Is it slow because it WAITS for something external?
│   (web requests, file I/O, database, API calls, user input)
│   │
│   └─→ YES: Are there fewer than ~50 concurrent tasks?
│           │
│           ├─→ YES: Use threading.Thread or ThreadPoolExecutor ✅
│           │         (I/O-bound + manageable count)
│           │
│           └─→ NO (hundreds/thousands of connections):
│                   Use asyncio + aiohttp/asyncpg ✅
│                   (I/O-bound + massive scale)
│
├─→ Is it slow because of HEAVY COMPUTATION?
│   (math, image processing, encryption, ML inference, data transforms)
│   │
│   └─→ YES: Use multiprocessing.Pool or ProcessPoolExecutor ✅
│             (CPU-bound: bypasses the GIL entirely)
│
└─→ Is it slow AND I need SHARED MEMORY between workers?
    │
    └─→ YES: Use multiprocessing with Manager() or shared Value/Array ✅
              (True parallelism with shared state)
```

-----

## 10. Comparison Table: Threading vs. Multiprocessing vs. Asyncio

|                     |`threading`                             |`multiprocessing`                            |`asyncio`                                |
|---------------------|----------------------------------------|---------------------------------------------|-----------------------------------------|
|**Best for**         |I/O-bound tasks                         |CPU-bound tasks                              |I/O-bound, high concurrency              |
|**Parallelism**      |No (GIL)                                |✅ Yes (separate processes)                   |No (single thread)                       |
|**Memory**           |Shared (risks race conditions)          |Separate per process                         |Shared (single thread, safe)             |
|**Overhead**         |Low                                     |High (process spawn)                         |Very Low                                 |
|**Communication**    |Shared variables + Lock                 |Queue / Pipe / Manager                       |awaitable coroutines                     |
|**Failure isolation**|Low (one crash can hurt all)            |✅ High (isolated processes)                  |Medium                                   |
|**Complexity**       |Medium                                  |High                                         |High (event loop mental model)           |
|**Use when**         |Downloading files, scraping, DB queries |Data science, video encoding, ML             |Web servers, chat apps, 1000+ connections|
|**Python API**       |`threading.Thread`, `ThreadPoolExecutor`|`multiprocessing.Pool`, `ProcessPoolExecutor`|`async def`, `await`, `asyncio.gather()` |
|**GIL impact**       |🛑 Limited by GIL for CPU work           |✅ Not affected                               |🛑 Single thread, but cooperative         |

-----

## Quick Reference Cheat Sheet

```python
# ── CREATE AND START A THREAD ─────────────────────────────
t = threading.Thread(target=my_function, args=(arg1, arg2), name="MyThread")
t.start()   # Begin execution
t.join()    # Wait for completion

# ── THREAD POOL (recommended for most cases) ──────────────
with ThreadPoolExecutor(max_workers=5) as pool:
    future = pool.submit(my_function, arg1, arg2)
    result = future.result()  # Blocks until done, re-raises exceptions

# ── LOCK (mutual exclusion) ────────────────────────────────
lock = threading.Lock()
with lock:              # Auto acquire + release (even on exception)
    # critical section

# ── RLOCK (reentrant — same thread can acquire multiple times)
rlock = threading.RLock()
with rlock:
    with rlock:         # Same thread — allowed with RLock, deadlock with Lock
        pass

# ── EVENT (broadcast signal to multiple threads) ──────────
event = threading.Event()
event.wait()            # Block until event.set() is called
event.set()             # Unblock ALL waiting threads
event.clear()           # Reset to "not set" state

# ── SEMAPHORE (allow N threads into a section) ────────────
sem = threading.Semaphore(3)  # Max 3 threads at once
with sem:
    # at most 3 threads here simultaneously

# ── THREAD-SAFE QUEUE ─────────────────────────────────────
q = queue.Queue(maxsize=10)
q.put(item)             # Add item (blocks if full)
item = q.get()          # Remove item (blocks if empty)
q.task_done()           # Signal item processing is complete
```

-----

*Guide written with the “First Principles” method: every abstraction is grounded in a real-world scenario, every line is explained, and every pattern is justified.*
