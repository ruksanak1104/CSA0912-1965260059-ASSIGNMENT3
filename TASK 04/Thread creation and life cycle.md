Aim

To implement parallel answer-script scanning using four scanner stations as separate threads, understand the thread life cycle, and compare thread creation using the Thread class and Runnable interface.

2. Thread Creation Using the Runnable Interface
In this approach, a class implements the Runnable interface and defines its task inside the run() method. A Thread object is then created using that Runnable object.

Explanation of states:

1.	NEW: A thread object is created but start() has not been called.

2.	RUNNABLE: The thread is ready to run or is currently executing.

3.	BLOCKED: The thread is waiting to acquire a monitor lock.

4.	WAITING: The thread waits indefinitely for another thread to perform an action.

5.	TIMED_WAITING: The thread waits for a specified time, such as during sleep().

6.	TERMINATED: The thread has completed its execution.

Life cycle of scanner stations:

•	Thread objects are created in the NEW state.

•	Calling start() moves them to RUNNABLE.

•	Calling sleep() temporarily moves them to TIMED_WAITING.

•	After the sleep period, they return to RUNNABLE.

•	After completing run(), they enter TERMINATED.

4. Difference Between Thread and Runnable

Extending Thread:

•	A class extends the Thread class.

•	The task and thread are combined in one class.

•	The class cannot extend another class because Java supports single inheritance.

•	Suitable for simple programs where a custom Thread class is required.

Implementing Runnable:

•	A class implements the Runnable interface.

•	The task is separate from the Thread object.

•	The class can extend another class while implementing Runnable.

•	Preferred when separating task logic from thread management or sharing a task design.

Conclusion: Implementing Runnable is generally preferred because it provides better flexibility and separates the task from the thread.

5. Why start() Is Used Instead of run()

•	start() creates a new thread of execution and internally invokes the run() method.

•	run() called directly behaves like a normal method.

•	Calling run() does not create a separate thread.

•	Therefore, start() is required for parallel execution.

Example:

Thread t = new Thread(() ->

    System.out.println(Thread.currentThread().getName())
);


t.run();

t.start();

Expected observation:

•	When run() is called directly, the output is main.

•	When start() is called, the output is a separate thread name, such as Thread-0.

This provides evidence that run() executes in the current thread, whereas start() creates a new thread.

6. Role of join() and isAlive()

join():

•	The join() method makes the main thread wait until the specified thread finishes execution.

•	It ensures that all scanner stations complete their work before the final total is displayed.

•	Without join(), the main thread may print the total before all stations finish.

isAlive():

•	The isAlive() method checks whether a thread has started and has not yet terminated.

•	It returns true if the thread is still alive.

•	It returns false if the thread has not started or has terminated.

7. Behavioural Experiments

A. Running the Program Ten Times

Run the Runnable version ten times and record the order in which the stations finish.

The following are illustrative observations, not actual measured results:

•	Run 1: 1 → 2 → 3 → 4

•	Run 2: 1 → 2 → 3 → 4

•	Run 3: 1 → 2 → 3 → 4

•	Run 4: 1 → 2 → 3 → 4

•	Run 5: 1 → 2 → 3 → 4

•	Run 6: 1 → 2 → 3 → 4

•	Run 7: 1 → 2 → 3 → 4

•	Run 8: 1 → 2 → 3 → 4

•	Run 9: 1 → 2 → 3 → 4

•	Run 10: 1 → 2 → 3 → 4

Observation:

•	The program uses different sleep durations for each station.

•	Therefore, the stations will usually finish in the order shown.

•	Actual scheduling can still affect the observed order.

B. Experimenting with Thread Priorities

Java supports thread priorities from 1 to 10.

•	Thread.MIN_PRIORITY = 1

•	Thread.NORM_PRIORITY = 5

•	Thread.MAX_PRIORITY = 10

Example:

stations[0].setPriority(Thread.MAX_PRIORITY);

stations[1].setPriority(Thread.MIN_PRIORITY);

stations[2].setPriority(Thread.NORM_PRIORITY);

stations[3].setPriority(Thread.NORM_PRIORITY);

Observation:

•	Higher-priority threads may receive preference from the scheduler.

•	Lower-priority threads may receive less preference.

•	Thread priority does not guarantee execution or completion order.

•	The actual behaviour depends on the JVM and operating-system scheduler.

C. Converting One Station into a Daemon Thread

Add this statement before starting the fourth station:

stations[3].setDaemon(true);

Observation:

•	A daemon thread is a background thread.

•	The JVM does not wait for daemon threads to finish when all user threads have terminated.

•	If the main thread finishes and no other user threads remain, the JVM may terminate while a daemon station is still working.

•	Therefore, a daemon thread should not be used for essential work that must finish.

Note: If main() calls join() on the daemon station, the main thread waits for it, so the daemon does not demonstrate early termination in that experiment. To 
observe daemon behaviour, run a separate experiment without joining the daemon thread.

8. Joint Conclusion

The same multithreaded program can produce different output orders because:

•	Threads execute independently.

•	The thread scheduler decides which thread gets CPU time.

•	Different threads may experience different execution delays.

•	Operating-system scheduling and system load can affect execution.

A correct multithreaded program must not depend on a particular execution order because that order is not guaranteed.

Using join() ensures that all required scanner stations finish before the final result is displayed.

9. Alternative Concept: Using ExecutorService

Instead of creating four Thread objects manually, Java provides ExecutorService to manage a group of threads.

A fixed thread pool creates a specified number of worker threads and reuses them to execute submitted tasks.

Program: ScannerExecutor.java



import java.util.concurrent.*;

class ScannerTask implements Callable<Integer> {

    private int stationNumber;
        private int scripts;

    ScannerTask(int stationNumber, int scripts) {
        this.stationNumber = stationNumber;
        this.scripts = scripts;
    }

    public Integer call() {

        try {
            System.out.println("Station " + stationNumber +
                               " started scanning.");

            Thread.sleep(1000);

            System.out.println("Station " + stationNumber +
                               " scanned " + scripts + " scripts.");

            return scripts;

        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            return 0;
        }
    }
}

public class ScannerExecutor {

    public static void main(String[] args) {

        ExecutorService executor =
            Executors.newFixedThreadPool(4);

        int totalSales = 0;

        try {
            Future<Integer>[] results = new Future[4];

            results[0] = executor.submit(
                new ScannerTask(1, 10));

            results[1] = executor.submit(
                new ScannerTask(2, 15));

            results[2] = executor.submit(
                new ScannerTask(3, 20));

            results[3] = executor.submit(
                new ScannerTask(4, 25));

            for (Future<Integer> result : results) {
                totalSales += result.get();
            }

            System.out.println("All stations completed.");
            System.out.println("Total scripts scanned: "
                               + totalSales);

        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            System.out.println("Main thread interrupted.");

        } catch (ExecutionException e) {
            System.out.println("Task execution failed: "
                               + e.getMessage());

        } finally {
            executor.shutdown();
        }
    }
}

Expected Output:

Station 1 started scanning.

Station 2 started scanning.

Station 3 started scanning.

Station 4 started scanning.

Station 1 scanned 10 scripts.

Station 2 scanned 15 scripts.

Station 3 scanned 20 scripts.

Station 4 scanned 25 scripts.

All stations completed.

Total scripts scanned: 70

The exact output order may vary.

Advantages of ExecutorService:

•	Manages threads automatically.

•	Reuses existing worker threads.

•	Reduces the need to create and manage Thread objects manually.

•	Supports task submission and result collection using submit() and Future.

•	shutdown() prevents new tasks from being submitted and allows submitted tasks to finish.

10. Complexity Analysis

Let:

•	n = Total number of answer scripts.

•	p = Number of available threads.

•	S = Memory required for one thread stack.

Time Complexity:

•	With one thread, processing takes approximately O(n), assuming equal processing time per script.

•	With p threads, the expected processing time is approximately O(n/p), assuming balanced workloads and negligible overhead.

•	Actual execution time may be higher due to scheduling, synchronization and task-management overhead.

Space Complexity:

•	Each thread requires memory for its stack.

•	For p threads, total thread-stack memory is approximately O(p × S).

•	Creating threads manually requires managing individual Thread objects.

•	ExecutorService reuses a fixed number of worker threads, reducing repeated thread creation overhead.

11. Final Conclusion

The parallel answer-script scanning system demonstrates how Java multithreading allows multiple scanner stations to process scripts concurrently.

•	Extending Thread provides a direct way to create threads.

•	Implementing Runnable separates task logic from thread management.

•	start() begins a new thread.

•	sleep() temporarily pauses a thread.

•	join() waits for thread completion.

•	isAlive() checks the execution status of a thread.

•	Thread priorities and daemon threads demonstrate additional thread behaviours.

•	ExecutorService provides a more manageable approach for applications with multiple tasks.

Overall, multithreading improves parallel processing, while proper synchronization and thread management ensure reliable results.







