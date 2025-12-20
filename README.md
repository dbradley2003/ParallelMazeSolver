# ParallelMazeSolver

**A high-performance, multithreaded C++ maze solver**

## Performance Analysis

I benchmarked my solution against optimized implementations of single-threaded
DFS and BFS to focusing solely on the performance gained from the
multithreaded architecture.

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
* **Structure:** Each worker thread maintains a chunked sequence data structure. This is essentially a linked list of `Chunks` (Memory blocks) and
each chunk mantains a fixed size array of `S` tasks.

* **Load Balancing:** The structure operates as a LIFO stack for the owner, implementing push, pop, and split operations.
The owner will push and pop from the top hot chunk and when the top becomes full, it allocates a new chunk to the top.
Using a linked list of chunks provides an efficient split operation due to the cheap cost of moving pointers.

* **Amortizing Costs:** In order to amortize the costs of communication and of sharing work, a value `K`
determines a threshold value for workers to respond positively to a work-request message.

* **Work Stealing:** When a worker has no more work, it will send a work-request to another thread chosen at random.
If the victim's workload exceeds threshold `K`, it shares its private frontier with the thread by detaching and transferring 
the bottom half (oldest tasks) to the requesting thread's frontier.

* **Message Passing:** Message passing is used to facilitate the stealing process and reduce the costs of synchronization.

### 2. Parallel Bidirectional Search

* **Dual Frontier Expansion**: To further reduce the time taken to find a solution, threads will search from
both ends of the grid in parallel. A solution is determined if a worker 
collides with a worker searching from the opposite end, or if it reaches its opposite end.

* **Collision Detection:** The Maze is represented as a grid of atomic unsigned integers.
Threads use atomic operations to mark cells with a unique flag, a collision occurs when
a thread attempts to mark a cell that was already visited by the opposing search.

## Build & Run Instructions

This project uses **CMake** for cross-compatibility. It supports Linux (GCC/Clang) and Windows (MSVC).

### Prerequisites
* **CMake** (Version 3.15 or higher)
* A compiler that works with C++17

---

### Linux

1. **Clone the repository:**
	```bash
	git clone git@github.com:dbradley2003/ParallelMazeSolver.git
	```

2. **Generate the build files:**
	```bash
	cmake -B build -DCMAKE_BUILD_TYPE=Release
	```

3. **Compile:**
	```bash
	# Build using all available cores
	cmake --build build -j$(nproc)
	```

4. **Run:**
	```bash
	./build/bin/main
	```
---

### Windows (Visual Studio)

1. Open the cloned project folder in Visual Studio.
2. Visual Studio will automatically detect the `CMakeLists.txt` and configure the project.
3. In the top toolbar:
	* Set the **Configuration** to `x64-Release`.
	* Set the **Startup Item**. to `main.exe`.
5. Build and run.


## Test Mazes

Due to the size of the files, I only kept one in the repository.

If you wish to test the program with the larger test mazes.

* Download the mazes from here:
* Place the downloaded maze in the `mazes/` directory.
* Change the file loaded in `main.cpp` at the top where it says: 	
	* `char inFileName[INPUT_NAME_SIZE] = "mazes/Maze100x100.data";`

* Build and run the program.

## Attribution
* **Baseline Logic:** The single-threaded reference algorithms were adapted from the University of Maryland.

* **Optimization:** The C++ multithreading architecture, thread pool, and bidirectional search are original implementations.