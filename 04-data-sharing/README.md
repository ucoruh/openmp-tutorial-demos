# 💾 OpenMP Data Sharing

This project demonstrates OpenMP data sharing mechanisms and variable scoping clauses essential for correct parallel programming.

## 🎯 Overview

When working with shared memory parallel programming, understanding how variables are shared or privatized among threads is crucial for preventing race conditions and ensuring correct results. This module explores the various data sharing clauses available in OpenMP.

## 📊 OpenMP Data Sharing and Variable Scoping

The following diagram illustrates how variables with different data sharing attributes are handled in OpenMP:

![OpenMP Data Sharing](./assets/openmp_data_sharing.png)

## 🧩 Key Data Sharing Clauses

### 1. `shared`

Variables declared as `shared` are accessible by all threads in the team. Changes made by one thread are visible to other threads.

```cpp
int count = 0;
#pragma omp parallel shared(count)
{
    // All threads access the same copy of count
    count++;  // Potential race condition!
}
```

### 2. `private`

Each thread gets its own uninitialized copy of the variable.

```cpp
int var = 10;
#pragma omp parallel private(var)
{
    // Each thread has its own copy of var (uninitialized)
    var = omp_get_thread_num();  // Safe, each thread modifies its own copy
}
// var retains its original value after the parallel region
```

### 3. `firstprivate`

Similar to `private`, but each thread's copy is initialized with the value from the master thread.

```cpp
int var = 10;
#pragma omp parallel firstprivate(var)
{
    // Each thread has its own copy of var initialized to 10
    var += omp_get_thread_num();  // Safe modification
}
// var retains its original value after the parallel region
```

### 4. `lastprivate`

Each thread has its own copy, and the value from the last iteration or section is copied back to the original variable.

```cpp
int var;
#pragma omp parallel for lastprivate(var)
for (int i = 0; i < 100; i++) {
    var = i;  // Safe, thread-local
}
// var now equals 99 (the value from the last iteration)
```

### 5. `threadprivate`

Global or static variables that maintain their values between parallel regions within the same thread.

```cpp
int thread_id;
#pragma omp threadprivate(thread_id)

#pragma omp parallel
{
    thread_id = omp_get_thread_num();
}
// Each thread now has its own persistent copy of thread_id

#pragma omp parallel
{
    // thread_id value from previous parallel region is preserved
    printf("Thread %d has thread_id = %d\n", omp_get_thread_num(), thread_id);
}
```

### 6. `default(shared|none)`

Sets the default data-sharing attribute for all variables in the parallel region.

```cpp
// All variables referenced in the parallel region must be explicitly scoped
#pragma omp parallel default(none) shared(a) private(b)
{
    // Code that uses a and b
}
```

## 💻 Examples in This Project

This project includes the following examples:

1. **Basic Data Sharing**: Demonstrates the differences between `shared`, `private`, and `firstprivate`
2. **Race Conditions**: Shows how race conditions occur and how to prevent them
3. **Thread-Private Variables**: Illustrates the use of `threadprivate` for persistent thread-local storage
4. **Default Clause**: Demonstrates the use of `default(none)` for safer parallel programming

## 🚀 Running the Examples

Use the provided scripts to configure, build, and run the examples:

1. Run `configure.bat` to set up the CMake project
2. Run `build_all.bat` to compile all examples
3. Run `run.bat` to execute the examples

Example usage:

```bash
run.bat --debug --example race_condition
```

## 🔍 Common Pitfalls

1. **Race Conditions**: Multiple threads modifying the same shared variable without synchronization
2. **Uninitialized Private Variables**: Forgetting that `private` variables are uninitialized
3. **Incorrect Scoping**: Using the wrong data-sharing clause for a variable
4. **Hidden Sharing**: Unintentional sharing of variables in nested function calls
5. **Array Elements**: Not properly scoping array elements in loops

## 📚 Additional Resources

- [OpenMP Data Sharing Attributes](https://www.openmp.org/spec-html/5.0/openmpsu26.html)
- [OpenMP Data Scoping Tutorial](https://www.openmp.org/wp-content/uploads/openmp-examples-4.5.0.pdf)
- [Race Condition Prevention Guide](https://hpc-tutorials.llnl.gov/openmp/data_scope/)

## Prerequisites

- Windows 10/11
- Visual Studio 2022 with C++ development tools
- CMake 3.20 or higher
- OpenMP support (included with Visual Studio)

## Building and Running

### Step 1: Configure
Run `configure.bat` to generate the Visual Studio project files:
```
configure.bat
```

### Step 2: Build
Run `build_all.bat` to compile Debug, Release, and Profile configurations:
```
build_all.bat
```

### Step 3: Run
Use the unified `run.bat` script with various options:

```
run.bat                          # Run with default settings (Release mode)
run.bat --debug                  # Run in Debug mode with additional diagnostics
run.bat --release                # Run in Release mode (optimized performance)
run.bat --threads 8              # Run with 8 threads
run.bat --demo private           # Run only the private variables demo
run.bat --verbose                # Run with verbose output
run.bat --help                   # Show all available options
```

For the most comprehensive experience, you can use:

```
run_all.bat                      # Run all demonstrations in sequence
```

This will execute all data sharing demos with various thread counts and performance comparisons.

For specific data sharing demos, we also provide:

```
run.bat --demo shared            # Demonstrate shared variables
run.bat --demo private           # Demonstrate private variables
run.bat --demo firstprivate      # Demonstrate firstprivate variables
run.bat --demo lastprivate       # Demonstrate lastprivate variables
run.bat --demo threadprivate     # Demonstrate threadprivate variables
run.bat --demo reduction         # Demonstrate reduction variables
```

**Note**: For accurate performance measurements, always use the Release build.

### Step 4: Clean
If you want to clean the build files and start from scratch:

```
clean.bat
```

This will remove all build artifacts and temporary files.

## Program Components and Observed Behavior

This demo includes the following examples and observations from actual execution:

1. **Shared Variables (Race Condition)**
   - Demonstrates issues with unprotected shared variable access
   - Shows typical race conditions in parallel code
   - **Observed:** Counter reached only 750,000 instead of expected 1,000,000
   - **Execution Time:** ~0.38ms in Release build

2. **Shared Variables with Protection**
   - Uses `atomic` directive to protect shared updates
   - Shows performance impact of synchronization
   - **Observed:** Counter correctly reached 1,000,000
   - **Execution Time:** ~8.72ms in Release build (approximately 23x slower than the unprotected version)

3. **Private Variables**
   - Demonstrates thread-local storage
   - Shows how private variables prevent race conditions
   - **Observed:** Correct summation with clear division of work between threads
   - Each thread processes a portion of the workload with private accumulators

4. **Firstprivate Variables**
   - Shows initialization of thread-private copies
   - Demonstrates value inheritance from parent scope
   - **Observed:** Each thread correctly received its own copy of the initial value (100)
   - Thread-specific modifications remain isolated

5. **Lastprivate Variables**
   - Demonstrates propagation of final values
   - Shows how to capture results from the last iteration
   - **Observed:** Final value (99) correctly captured from the last iteration

6. **Threadprivate Variables**
   - Shows persistent thread-local storage
   - Demonstrates value preservation across parallel regions
   - **Observed:** Values maintained between parallel regions and correctly incremented
   - Each thread preserved its unique ID across multiple parallel sections

7. **Matrix Multiplication**
   - Complex example with multiple data sharing strategies
   - Shows performance impact of proper data sharing
   - **Matrix Size:** 600x600
   - **Sequential Time:** ~96.6ms (Release build)
   - **Parallel Time:** ~11.6ms (Release build)
   - **Achieved Speedup:** 8.3x with 32 available threads
   - Uses shared matrices with private loop indices and accumulators

## Memory Visualization

The program includes memory visualization that shows:

```
Memory Layout:
+---------------------------+
| Shared Memory             |
| Values:                  0 |
+---------------------------+
| Thread 0 Private Memory    |
+---------------------------+
| Thread 1 Private Memory    |
+---------------------------+
| Thread 2 Private Memory    |
+---------------------------+
| Thread 3 Private Memory    |
+---------------------------+
```

This visualization helps understand:
- How data is organized in memory
- Which values are shared vs. thread-private
- How values change before and after parallel execution

## Performance Insights

From our experiments, we observed:

1. **Race Conditions vs. Protected Operations:**
   - Race conditions are significantly faster but produce incorrect results
   - Atomic operations ensure correctness but introduce substantial overhead (~23x slowdown)

2. **Debug vs. Release Performance:**
   - Release builds are approximately 4-6x faster than Debug builds
   - Matrix multiplication in Debug: ~639ms vs. Release: ~97ms (sequential)

3. **Parallelization Efficiency:**
   - Matrix multiplication achieved 8.3x speedup with 32 available threads
   - Memory access patterns and work distribution significantly impact scalability

## Understanding Data Sharing in OpenMP

### Default Sharing Behavior

Variables in OpenMP have default scoping rules:
- Variables declared outside a parallel region are **shared** by default
- Loop indices in `for` constructs are **private** by default
- Automatic variables declared inside a parallel region are **private**

### Race Conditions

Race conditions occur when multiple threads access the same shared variable without proper synchronization:
- Read-modify-write operations are particularly vulnerable
- The demo shows how race conditions lead to incorrect results
- Protection mechanisms (`atomic`, `critical`) are demonstrated

## Code Components

- `src/main.cpp` - Main implementation with all examples
- `CMakeLists.txt` - CMake configuration for building the project
- Batch files:
  - `configure.bat` - Sets up the CMake project
  - `build_all.bat` - Builds Debug, Release, and Profile configurations
  - `run.bat` - Runs the program with various options
  - `run_all.bat` - Runs all demonstrations in sequence
  - `clean.bat` - Removes build artifacts

## Common Issues and Troubleshooting

### Performance Issues
- Excessive use of atomic or critical sections can cause performance bottlenecks
- Over-privatization can increase memory usage
- Improper data sharing can lead to false sharing

### Race Conditions
- If results are inconsistent across runs, you likely have a race condition
- Use thread sanitizers or parallel debuggers to identify race conditions
- Consider whether variables should be shared, private, or protected

### Build Issues
- If CMake configuration fails:
  - Ensure Visual Studio 2022 is properly installed with C++ components
  - Verify CMake 3.20+ is installed and in your PATH

## Advanced Usage

### Controlling Thread Count
You can set the number of threads by:
- Setting the `OMP_NUM_THREADS` environment variable
- Using `omp_set_num_threads()` in code

### Custom Data Sharing Experiments
Modify the code to experiment with:
- Different protection mechanisms (atomic vs. critical)
- Alternative privatization strategies
- Reduction variables

## Additional Resources

- [OpenMP Documentation](https://www.openmp.org/resources/)
- [OpenMP 5.1 Specification](https://www.openmp.org/spec-html/5.1/openmp.html)
- See the accompanying `DATA_SHARING_GUIDE.md` for detailed explanations of each data sharing concept

## License

This project is licensed under the MIT License - see the LICENSE file for details. 