Aim

To design and implement a concurrent, fault-tolerant Java program using ExecutorService, Callable, and Future to process 10,000 student result files efficiently while handling corrupt and missing files without stopping the entire process.

Stage 1 – Interpret the Problem (Joint) 

1. Tasks that can run in parallel

The following tasks can be executed simultaneously:

•	Reading individual student score files.

•	Validating the marks present in each file.

•	Calculating the total marks and student results.

•	Recording successful and failed file processing details.

Shared data:

•	A concurrent collection to store successful student results.

•	A concurrent collection to store failed file names and error messages.

•	A shared summary containing the total number of successful and failed files.

2. Possible failures and their handling

•	Missing file: Catch FileNotFoundException and record the file as failed.

•	Corrupt file: Catch IOException and record the error.

•	Invalid marks: Handle invalid or non-numeric marks using NumberFormatException.

•	Invalid score range: Reject marks outside the permitted range.

•	Thread interruption: Handle InterruptedException and restore the interrupt status.

•	Unexpected processing error: Catch exceptions within the individual task so that one failed file does not stop other tasks.

3. Measurable requirements

•	Process exactly 10,000 input file paths.

•	Every file must be classified as either successful or failed.

•	One failed file must not stop the remaining processing.

•	The final success count and failure count must add up to 10,000.

•	Calculate and record total processing time.

•	Compare execution time using thread pools of 1, 2, 4 and 8 threads.

•	Verify the results using a small hand-checked test set.

Stage 2 – Design (Student A)

1. Design using ExecutorService, Callable and Future

•	ExecutorService: Creates and manages a fixed number of worker threads.

•	Callable: Processes one student score file and returns the result.

•	Future: Stores the result returned by a Callable task and allows the main thread to retrieve it.

•	ConcurrentHashMap: Stores successful student results safely.

•	ConcurrentLinkedQueue: Stores failure details safely.

2. Concurrent collections and justification

ConcurrentHashMap:

•	Stores student IDs and their calculated results.

•	Allows multiple threads to update the results safely.

•	Reduces the need for manual synchronization.



ConcurrentLinkedQueue:

•	Stores the names of failed files and their error messages.

•	Supports safe insertion by multiple worker threads.

•	Prevents data corruption during concurrent updates.

3. Exception handling design

•	Each file is processed independently.

•	Exceptions are caught inside the Callable task.

•	Failed files are recorded in the failure collection.

•	The task returns a failure result instead of stopping the entire processing system.

•	The main thread collects all completed task results using Future.

•	The final summary is generated after all tasks finish.

4. Class diagram

              +----------------------+
              |   ResultProcessor    |
              +----------------------+
              | ExecutorService      |
              | Successful Results  |
              | Failed Files        |
              +----------------------+
                        |
                        |
                Submits Callable Tasks
                        |
                        v
              +----------------------+
              |   FileProcessor      |
              +----------------------+
              | call()               |
              | Read File            |
              | Validate Marks       |
              | Calculate Result     |
              +----------------------+
                        |
                        |
                  Returns Result
                        |
                        v
              +----------------------+
              |       Future         |
              +----------------------+
              | get()                |
              | Retrieves Result     |
              +----------------------+
                        |
                        v
              +----------------------+
              |    Final Summary     |
              +----------------------+
              | Successful Count     |
              | Failed Count         |
              | Total Processing Time|
              +----------------------+



5. Pseudocode

START

Create ExecutorService with fixed thread pool

Create concurrent collections:
   
    Successful results
    
    Failed files

FOR each student score file:
    
    Submit a Callable task

    Inside Callable:
        Try:
    
            Read the file
            
            Validate the marks
            
            Calculate the result
            
            Store successful result
            
            Return success status

        Catch file or validation exceptions:
            
            Record failure details
            
            Return failure status

Collect all Future results

Wait for all submitted tasks to complete

Calculate:
   
    Total successful files
    
    Total failed files
    
    Total processing time

Print final summary

Shutdown ExecutorService

END

2. Injected failures

The program deliberately creates the following test conditions:

•	Every 1000th file is missing.

•	Every 500th file contains invalid text such as ABC.

•	All other files contain three valid marks: 80, 75 and 90.

The program should classify:

•	Missing files: 10.

•	Corrupt files: 10.

•	Successful files: 9,980.

•	Total files: 10,000.

3. Expected summary counts

•	Total input files: 10,000.

•	Successful files: 9,980.

•	Failed files: 20.

•	Successful and failed files together: 10,000.

For every successful file:

- Total marks = 80 + 75 + 90 = 245.

4. Measuring execution time

The program uses:

long start = System.nanoTime();

// File processing

long end = System.nanoTime();

double time = (end - start) / 1_000_000.0;

The program measures execution time using four thread pool sizes:

•	1 thread: Processes files sequentially.

•	2 threads: Processes files using two workers.

•	4 threads: Processes files using four workers.

•	8 threads: Processes files using eight workers.


5. Recording performance results

Record the actual execution times produced by your computer.

•	Thread pool size 1:

o	Execution time: Record measured value.

o	Processing is mostly sequential.

•	Thread pool size 2:

o	Execution time: Record measured value.

o	Two files can be processed simultaneously.

•	Thread pool size 4:

o	Execution time: Record measured value.

o	Four files can be processed simultaneously.

•	Thread pool size 8:

o	Execution time: Record measured value.

o	Eight files can be processed simultaneously.

The exact execution times depend on the computer, file system, Java environment and current system load. Do not treat estimated values as measured results.

Important measurement note: Run each pool size several times and compare the average. For a more reliable benchmark, use a warm-up run and separate file-
generation time from processing time. The supplied program generates the files before starting the timer.

6. Hand-checking a small test set

Consider five input files:
•	student1.txt: 80, 75, 90 → Success, total = 245.

•	student2.txt: 70, 60, 80 → Success, total = 210.

•	student3.txt: ABC → Failure, corrupt data.

•	student4.txt: Missing file → Failure.

•	student5.txt: 90, 85, 95 → Success, total = 270.

Expected summary:

•	Total files: 5.

•	Successful files: 3.

•	Failed files: 2.

•	Total successful marks: 725.

7. Failure verification

The following failures are handled:

•	Missing files are recorded as failures.

•	Corrupt marks are recorded as failures.

•	Invalid mark ranges are rejected.

•	Empty files are recorded as failures.

•	Other worker tasks continue even when one file fails.

One limitation is that a serious JVM-level failure, such as an out-of-memory error, cannot always be safely recovered from by ordinary exception handling.

Stage 4 – Defend the Design (Joint) 

1. Recommended thread pool size

A pool of 4 threads is a reasonable starting recommendation for this file-processing task.

Reasons:

•	Multiple files can be processed simultaneously.

•	It provides parallelism without creating too many threads.

•	It reduces the overhead of excessive thread creation.

•	It can provide a good balance between execution time and resource usage.

However, the best pool size must be selected using actual measurements.

2. Why more threads do not always help

•	Creating too many threads increases memory usage.

•	Threads compete for CPU time.

•	Context switching creates additional overhead.

•	Multiple threads may compete for file-system resources.

•	Disk input/output can become the bottleneck.

Therefore, increasing the number of threads does not always reduce execution time.

3. How the design guarantees a complete and correct summary

•	Every input file path is submitted as a separate task.

•	Every submitted task returns either a success or failure result.

•	The main thread calls get() on every Future.

•	It waits for every task to complete before producing the summary.

•	Successful results are stored in a thread-safe collection.

•	Failed files are stored separately in another thread-safe collection.

•	The final success count and failure count must add up to 10,000.

This ensures that one failed file does not prevent the remaining files from being processed.

4. Remaining limitation

•	The program stores all Future objects until processing is complete.

•	For a much larger number of files, this can increase memory usage.

•	A better approach would be to process completed tasks incrementally using ExecutorCompletionService and maintain only the required summary information.

Alternative Concept and Complexity Analysis

1. Using parallel streams

Java parallel streams allow collection elements to be processed simultaneously.

Example:

import java.io.File;

import java.util.*;

import java.util.concurrent.ConcurrentHashMap;

public class Main {

    public static void main(String[] args) {

        List<File> files = new ArrayList<>();

        for (int i = 1; i <= 10000; i++) {
            files.add(new File("scores/student" + i + ".txt"));
        }

        ConcurrentHashMap<String, Integer> results =
            new ConcurrentHashMap<>();

        files.parallelStream().forEach(file -> {

            try (Scanner sc = new Scanner(file)) {

                int total = 0;

                while (sc.hasNextLine()) {
                    total += Integer.parseInt(sc.nextLine().trim());
                }

                results.put(file.getName(), total);

            } catch (Exception e) {

                System.out.println(
                    "Failed: " + file.getName());
            }
        });

        System.out.println(
            "Successful files: " + results.size());

        System.out.println(
            "Failed files: " +
            (files.size() - results.size()));
    }
}

Advantages:

•	Requires less code.

•	Automatically performs parallel processing.

•	Uses the common ForkJoinPool by default.

•	Exceptions can be handled individually inside the processing operation.

Limitations:

•	Provides less direct control over thread pool size.

•	Uses a shared common pool by default.

•	An unhandled exception may terminate the overall parallel stream operation.

•	Managing detailed failure information requires additional collections.

Limitations:

•	Creates 10,000 threads.

•	Requires high memory for thread stacks.

•	Increases context-switching overhead.

•	May reduce performance due to excessive thread creation.

•	Can cause resource exhaustion.

•	Is difficult to manage compared with a fixed thread pool.

3. Time complexity analysis

ExecutorService:

•	Each file is read and validated once.

•	If each file contains an average of M marks, total work is O(N × M).

•	With P worker threads, ideal processing time is approximately O(N × M / P).

•	Actual execution time also depends on disk I/O, scheduling and synchronization.

Parallel streams:

•	Total work remains O(N × M).

•	Parallel execution can reduce elapsed time depending on available processor resources.

•	The common pool controls the parallel execution.

•	Disk I/O and common-pool contention can limit performance.

One thread per file:

•	Total processing work remains O(N × M).

•	Creating and managing 10,000 threads adds significant overhead.

•	Execution time may increase because of context switching and resource contention.

•	More threads do not guarantee faster processing.

4. Memory complexity analysis

ExecutorService:

•	Uses P worker threads instead of 10,000 threads.

•	Thread stack memory is approximately P × S, where S is the stack size per thread.

•	The task queue and Future objects require additional memory.

•	The result and failure collections require memory proportional to the number of processed files.

Parallel streams:

•	Uses a limited number of worker threads from the common ForkJoinPool.

•	Thread stack memory depends on the number of pool workers.

•	Additional memory is required for stream processing and result collections.

One thread per file:

•	Creates 10,000 threads.

•	Thread stack memory is approximately 10,000 × S.

•	Additional memory is required for thread objects and task data.

•	This approach can consume a very large amount of memory.

5. How failures are handled

ExecutorService:

•	Each Callable handles its own file-processing exceptions.

•	Failed files are recorded separately.

•	Other tasks continue processing.

•	Future provides the result of each completed task.

Parallel streams:

•	Exceptions can be caught inside each stream operation.

•	Failures can be recorded in a concurrent collection.

•	If exceptions are not handled inside the operation, an exception may escape and terminate the stream processing operation.

One thread per file:

•	Each thread can handle exceptions independently.

•	An exception in one thread does not normally stop other threads.

•	Shared result collections must be thread-safe.

•	Managing 10,000 individual threads is difficult.

6. Most suitable design

•	ExecutorService: Best choice because it provides controlled concurrency, reusable worker threads and manageable exception handling.

•	Parallel streams: Suitable for simpler parallel operations where detailed thread management is not required.

•	One thread per file: Not recommended because of high memory consumption and thread-management overhead.






