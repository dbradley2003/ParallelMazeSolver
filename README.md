# ParallelMazeSolver

**A high-performance, multithreaded C++ maze solver**

## Performance Analysis

To verify the efficiency of parallel architecture, I benchmarked my solution against optimized single-threaded reference 
implementations (specialized DFS and BFS) adapted from the University of Maryland.

### Benchmarks

* **Hardware:** Apple M1 Pro (Virtualized environment, 8 vCPUs allocated via Parallels)
* **Dataset:** `20,000 x 20,000` Grid (400M Cells), `15,000 x 15,000` Grid (225M Cells)
* **Baseline:** Optimized Serial BFS & BFS (University of Maryland Reference)


| Implementation | 15k x 15k | 20k x 20k | Average Time |
| :--- | :--- | :--- | :--- |
| **SolverBFS** | 7.3264s |  ~24.8525s | 16.0895s |
| **SolverDFS** | 4.92471s  | ~6.5888s | 5.7568s |
| **My Solution** | 0.974s| ~1.4468s | 1.2104s |


> **Note:** The baseline algorithms were already highly optimized for memory locality and pointer arithmetic. The speedup is achieved purely by multithreading.

---

## Technical Specification

### 1. Work-Stealing Thread Pool
* **Structure:** Each worker thread maintains a Chunked Linked Stack. This is a linked list of `Chunks` (Memory blocks),
where each chunk contains a fixed size array of tasks.

* **Load Balancing:** The structure operates as a LIFO stack for the owner, pushing and popping tasks from the hot top chunk.
When a chunk is full, a new one is allocated to the top.

* **Work Stealing:** When a worker thread runs out of work, it randomly requests work from another thread. 
If the victim's workload exceeds threshold `K`, it performs a split operation, detaching the bottom half 
(oldest tasks) to transfer to the thread requesting work.

* **Message Passing:** Message passing is used to facilitate the stealing process, reducing synchronization overhead
on the data structure.

### 2. Parallel Bidirectional Search

* **Frontier Expansion:** Worker Threads search from both ends of the maze (Start & End) concurrently.
The search stops when a worker either reaches the opposite end or detects a collision.

* **Collision Detection:** The Maze is represented as a grid of atomic unsigned integers.
Threads use atomic operations to mark cells with a unique flag, a collision occurs when
a thread attempts to mark a cell that was already visited by the opposing search.

## Attribution
* **Baseline Logic:** The single-threaded reference algorithms were adapted from the University of Maryland.

* **Optimization:** The C++ multithreading architecture, thread pool, and bidirectional search are original implementations.


## Future Roadmap
* [ ] Add CMake support for Linux/MacOS cross-compatibility
* [ ] Implement a graphic visualizer
