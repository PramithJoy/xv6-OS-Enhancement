# xv6-OS-Enhancement
A full-stack upgrade for the xv6 OS. I plugged in a new lottery scheduler, on-demand paging, kernel threads, a bunch of sync tools (locks, CVs), and even upgraded the filesystem with doubly-indirect blocks.

**Course:** Operating Systems (Prof. Abhisheck Bichhawat, IIT Gandhinagar)
**Timeline:** Aug 2025 - Nov 2025

## 1. Project Overview
This repository contains a full-stack upgrade to the xv6 OS, built over five assignments. The simple kernel was enhanced with a new lottery scheduler, on-demand paging, kernel threads, a full suite of synchronization tools (locks, CVs, semaphores), and an upgraded filesystem with doubly-indirect blocks and symbolic links.

## 2. How to Build and Run

1.  **Clean the build:**
    ```sh
    make clean
    ```
2.  **Build and run in the QEMU emulator:**
    ```sh
    make qemu
    ```
3.  **Run test programs** at the `$` prompt:
    ```sh
    $ t_threads
    $ t_barrier
    $ t_sem1
    $ bigfile
    $ ...etc.
    ```

---

## 3. Core Feature Implementation

### Phase 1: Core Process & Shell Enhancements (Asst. 1)
* **Goal:** Add foundational utilities and improve shell usability.
* **Implementation:**
    * Created `uptime()` and `ps` system calls to report system and process status.
    * Built the `pause` user program using the kernel's `sleep` syscall.
    * Modified the `sh` (shell) to support tab completion for commands.
* **Key Files:** `sysproc.c`, `user/ps.c`, `user/uptime.c`, `user/pause.c`

### Phase 2: Advanced Process Scheduler (Asst. 2)
* **Goal:** Replace the Round Robin scheduler with a fair, efficient lottery scheduler.
* **Implementation:**
    * Implemented a **Boosted Lottery Scheduler**. Processes get "tickets" for CPU time.
    * Added **priority boosting**: if a process sleeps for `x` ticks, its tickets are doubled for the next `x` lotteries to reclaim lost CPU time.
    * Re-engineered `sleep()` to wake up only when the timer expires, not on every tick.
* **Key Files:** `proc.c`, `proc.h`, `sysproc.c` (for `settickets`)

### Phase 3: Modern Virtual Memory (Asst. 3)
* **Goal:** Implement "lazy allocation" so the OS doesn't crash on a page fault.
* **Implementation:**
    * Modified the `dabort_handler` in `trap.c` to catch page faults (type 5 or 7).
    * Instead of panicking, it calls `handle_page_fault` (in `vm.c`).
    * This new handler allocates a new page, maps it into the process's page table, and resumes the user program.
* **Key Files:** `trap.c`, `vm.c`, `trap_asm.S`

### Phase 4: Concurrency & Threading (Asst. 4)
* **Goal:** Build a complete kernel-level threading system and a full suite of synchronization tools.
* **Implementation:**
    1.  **Kernel Threads:**
        * Added `thread_create`, `thread_join`, and `thread_exit` to `proc.c`.
        * `thread_create` **shares the parent's memory** (`pgdir`) but allocates a **new user stack** (`allocuvm`).
        * Modified `exit()` and `wait()` to be "thread-aware" (e.g., `exit` only exits the thread, `wait` skips threads).
    2.  **Synchronization "Stack":**
        * **Kernel:** Built a `barrier` using `sleep`/`wakeup` (`barrier.c`).
        * **Userspace Atomics:** Built `acquireLock` (spinlock) in `ulib.c` using `__sync_lock_test_and_set`.
        * **Kernel Channels:** Added `getChannel`, `sleepChan`, `sigChan` to `proc.c` to let user threads sleep.
        * **Userspace CVs/Semaphores:** Used the locks and channels to build `condWait`, `semUp`, and `semDown` in `ulib.c`.
* **Key Files:** `proc.c`, `proc.h`, `ulib.c`, `barrier.c`

### Phase 5: Advanced Filesystem Features (Asst. 5)
* **Goal:** Upgrade the filesystem to support large files and symbolic links.
* **Implementation:**
    * **Doubly-Indirect Blocks:** Modified the `dinode` structure and the `bmap()` function to support a doubly-indirect block pointer, increasing max file size.
    * **Symbolic Links:** Implemented the `symlink(target, path)` system call and modified the `open()` syscall to "follow" the link.
* **Key Files:** `fs.c`, `fs.h`, `stat.h`, `sysfile.c`
