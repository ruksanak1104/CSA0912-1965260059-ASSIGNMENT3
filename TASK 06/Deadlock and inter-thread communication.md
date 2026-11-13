Aim

To understand deadlock prevention and inter-thread communication in Java by implementing hostel wallet transfers and a producer-consumer token counter using multiple threads.

Part A – Deadlock Analysis and Prevention

1. Program that can cause deadlock

In a hostel wallet system, two students transfer money between two accounts.

•	Thread 1 locks Account A and waits for Account B.

•	Thread 2 locks Account B and waits for Account A.

•	Both threads wait forever because each thread holds a lock required by the other.

2. Four necessary conditions for deadlock

1.	Mutual Exclusion: Only one thread can hold a particular account lock at a time.

2.	Hold and Wait: A thread holds one account lock while waiting for another account lock.

3.	No Preemption: A lock cannot be forcibly taken away from a thread.

4.	Circular Wait: Thread 1 waits for Account B while Thread 2 waits for Account A.

All four conditions are present, resulting in deadlock.

3. Deadlock prevention using consistent lock ordering

To prevent deadlock, all threads must acquire account locks in the same order.

For example, every transfer locks the account with the smaller ID first and the account with the larger ID second.

Explanation:

•	Every thread locks the account with the smaller ID first.

•	The account with the larger ID is locked second.

•	Therefore, threads cannot form a circular waiting chain.

•	This prevents deadlock while allowing multiple transfers to execute safely.

Part B – Producer-Consumer Problem

1. Complete the TODO 1 and TODO 2

The mess counter can hold a maximum of 5 tokens.

•	The producer prepares tokens and adds them to the counter.

•	Consumers collect tokens from the counter.

•	If the counter is full, the producer waits.

•	If the counter is empty, consumers wait.

2. Why must wait() be used inside a while loop?

The wait() method must be used inside a while loop because:

•	A thread may wake up even without receiving the expected notification. This is called a spurious wakeup.

•	Another thread may change the counter before the awakened thread gets the lock again.

•	Therefore, the condition must be checked again before accessing the buffer.

•	The while loop prevents inserting into a full counter or removing from an empty counter.

Example:

while (count == 0) {

    wait();
}


3. Why must wait() be called inside a synchronized block?

•	The wait() method must be called by a thread that owns the object's monitor.

•	The synchronized method provides that monitor ownership.

•	When wait() is called, the thread releases the monitor and enters the waiting state.

•	After notification, it must reacquire the monitor before continuing.

Calling wait() outside synchronized code causes an IllegalMonitorStateException.

4. Difference between notify() and notifyAll()

•	notify() wakes up only one waiting thread.

•	notifyAll() wakes up all threads waiting on the same object's monitor.

•	With multiple producers and consumers, notify() may wake a thread that cannot currently proceed.

•	This can leave the correct thread waiting and may cause the system to stop making progress.

•	notifyAll() allows all waiting threads to recheck their conditions, so it is safer for this producer-consumer design.

Why use notifyAll() in this program?

There is one producer and two consumers. With this particular arrangement, notify() may work because only the consumers wait when the buffer is empty and only the 
producer waits when it is full. However, notifyAll() is more reliable if the program is extended to use multiple producers or additional consumers.



5. Counter size over time

The counter size changes according to the producer and consumer operations.

Example sequence:

•	Producer adds token 1: counter size = 1.

•	Producer adds token 2: counter size = 2.

•	Producer adds token 3: counter size = 3.

•	Consumer removes token 1: counter size = 2.

•	Producer adds token 4: counter size = 3.

•	Producer adds token 5: counter size = 4.

•	Producer adds token 6: counter size = 5.

•	Producer waits because the counter is full.

•	Consumer removes a token: counter size = 4.

The counter size always remains between 0 and 5 because:

•	The producer waits when count == 5.

•	The consumer waits when count == 0.

6. Difference between sleep() and wait()

•	sleep():

o	Pauses a thread for a specified time.

o	Does not release the object's lock.

o	Belongs to the Thread class.

o	Can be called without a synchronized block.

•	wait():

o	Makes a thread wait until notification or interruption.

o	Releases the object's monitor lock.

o	Belongs to the Object class.

o	Must be called inside synchronized code.

7. What happens if while is changed to if?

Original code:

while (count == 0) {
    wait();

}

Modified code:

if (count == 0) {

    wait();
}

Similarly, for the producer:

if (count == buffer.length) {
         wait();
}

Possible problems:
•	A thread may wake up without the condition becoming true.

•	Another consumer may remove the available token before the awakened consumer continues.

•	The awakened consumer may then attempt to remove a token from an empty counter.

•	A producer may also attempt to insert into a full counter.

•	This can result in incorrect data, overwritten tokens, or invalid counter values.

Conclusion: Using while is necessary because it checks the condition again whenever a thread wakes up. Changing it to if may cause errors, although the errors may 
not appear in every execution.

Alternative Concepts and Complexity Analysis

1. Using ArrayBlockingQueue
Instead of manually implementing wait() and notifyAll(), Java provides ArrayBlockingQueue.
It is a thread-safe bounded queue that automatically handles producer-consumer synchronization.

Example code:

import java.util.concurrent.ArrayBlockingQueue;

public class Main {

    public static void main(String[] args)
            throws InterruptedException {

        ArrayBlockingQueue<Integer> counter =
                new ArrayBlockingQueue<>(5);

        int[] collected = new int[2];

        Thread cook = new Thread(() -> {
            try {
                for (int i = 1; i <= 20; i++) {
                    counter.put(i);
                    System.out.println("Produced: " + i);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });

        Thread[] students = new Thread[2];

        for (int s = 0; s < 2; s++) {

            final int id = s;

            students[s] = new Thread(() -> {
                try {
                    for (int k = 0; k < 10; k++) {
                        int token = counter.take();
                        collected[id] += token;

                        System.out.println("Consumed: " + token);
                    }
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            });
        }

        cook.start();

        for (Thread t : students) {
            t.start();
        }

        cook.join();

        for (Thread t : students) {
            t.join();
        }

        System.out.println("Sum of tokens collected: "
                + (collected[0] + collected[1])
                + " (expected 210)");
    }
}

Advantages:

•	Automatically handles synchronization.

•	Prevents the counter from exceeding its capacity.

•	put() waits when the queue is full.

•	take() waits when the queue is empty.

•	Reduces manual coding errors.

3. Time and space complexity analysis

Producer-consumer using wait() and notifyAll():

•	put() operation: O(1) when space is available.

•	take() operation: O(1) when a token is available.

•	Waiting time depends on other threads and their execution.

•	Space complexity: O(5), because the buffer stores at most 5 tokens.

Producer-consumer using ArrayBlockingQueue:

•	put() operation: O(1) under normal conditions.

•	take() operation: O(1) under normal conditions.

•	Waiting may occur when the queue is full or empty.

•	Space complexity: O(5), because the queue has a fixed capacity of 5.

Wallet transfer using synchronized locks:

•	Time complexity: O(1) for a transfer involving two accounts, excluding lock waiting time.

•	Space complexity: O(1) additional space per transfer.

Wallet transfer using tryLock():

•	Time complexity: O(1) for each lock acquisition attempt, excluding timeout and retries.

•	Repeated retries may increase total execution time.

•	Space complexity: O(1) additional space per transfer.

4. Which design is less error-prone?

•	ArrayBlockingQueue is less error-prone for producer-consumer problems because it handles synchronization internally.

•	tryLock() with timeout helps prevent indefinite waiting and allows the application to handle unsuccessful lock acquisition.

•	Consistent lock ordering is also a simple and effective way to prevent deadlock.

•	Manual wait() and notifyAll() require careful condition checking and synchronization.

Joint Conclusion

Deadlock and producer-consumer problems both involve multiple threads competing for shared resources.

•	Deadlock occurs when threads wait indefinitely for locks held by one another.

•	Producer-consumer problems occur when threads coordinate access to a shared buffer.

•	Deadlock is prevented by using a consistent lock-ordering rule.

•	Producer-consumer coordination is achieved using wait(), notifyAll(), and condition checking with while loops.

•	ArrayBlockingQueue provides a simpler and safer alternative for managing shared buffers.

Therefore, proper synchronization, lock management, and inter-thread communication ensure data consistency and help threads make progress without deadlock or 
buffer errors.








