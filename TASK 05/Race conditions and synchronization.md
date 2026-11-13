Aim

To demonstrate race conditions in a cinema seat-booking system, understand how multiple threads can book the same seat simultaneously, and compare different synchronization techniques to prevent double booking.

1. Booking Without Synchronization

In this approach, multiple threads attempt to book the same seat without using synchronization. Since checking and booking are separate operations, two threads may access the same seat at the same time.

Observation:

•	Ten seats are available.

•	One hundred users attempt to book seats.

•	Ten users compete for each seat.

•	Multiple threads may pass the availability check before another thread marks the seat as booked.

•	Therefore, bookingsMade may become greater than 10.

The exact number of duplicate bookings can vary between runs.

Observation:

•	The synchronized keyword protects the entire booking method.

•	Only one thread can execute the method on the same screen object at a time.

•	Other threads must wait until the current thread releases the lock.

•	Therefore, duplicate booking is prevented.

3. Booking Using a Synchronized Block

A synchronized block protects only the critical section instead of locking the entire method.

Observation:

•	The synchronized block protects the check-and-book operation.

•	The lock is released automatically after the block finishes.

•	Code outside the synchronized block can execute without acquiring that lock.

•	It prevents duplicate booking while allowing more flexibility in method design.

4. Race Condition Explanation

A race condition occurs when multiple threads access and modify shared data at the same time, and the final result depends on the order of execution.

Step-by-step example:

Suppose two users, Thread A and Thread B, attempt to book seat 5.

1.	Thread A checks whether seat 5 is available. It finds that the seat is free.

2.	Before Thread A marks the seat as booked, Thread B checks seat 5.

3.	Thread B also finds that the seat is free.

4.	Thread A marks seat 5 as booked and increases the booking count.

5.	Thread B also marks seat 5 as booked and increases the booking count.

6.	Both threads report a successful booking for the same seat.

This results in double booking.

Exact critical section:

if (!booked[seat]) {

    booked[seat] = true;
    bookingsMade++;
}


This section is critical because it checks the seat availability, changes its booking status and updates the shared booking count. These operations must execute 
atomically.

5. Testing and Recording Double Bookings

Run the unsynchronized program ten times and record the actual number of seats sold in each run.

Record the observations in bullet points:

•	Run 1: Record the displayed number of seats sold.

•	Run 2: Record the displayed number of seats sold.
•	Run 3: Record the displayed number of seats sold.

•	Run 4: Record the displayed number of seats sold.

•	Run 5: Record the displayed number of seats sold.

•	Run 6: Record the displayed number of seats sold.

•	Run 7: Record the displayed number of seats sold.

•	Run 8: Record the displayed number of seats sold.

•	Run 9: Record the displayed number of seats sold.

•	Run 10: Record the displayed number of seats sold.
Observation:

•	The unsynchronized version may produce more than 10 successful booking operations.

•	The result may vary because thread execution order is not predictable.

•	The synchronized method and synchronized block versions should consistently produce exactly 10 successful bookings in this test.

Important: The unsynchronized result is not guaranteed to exceed 10 on every run. Record the actual output from your compiler rather than assuming a particular 
number.

6. Performance Comparison of Synchronization Strategies

To measure the execution time for 10,000 booking attempts, use System.nanoTime().

For a fair comparison:

•	Create 10,000 booking attempts for the same number of available seats in each version.

•	Start the timer immediately before starting the worker threads.

•	Use join() to wait for all threads.

•	Stop the timer after all threads finish.

•	Repeat the experiment several times and calculate the average execution time.

Example timing code:

long startTime = System.nanoTime();

// Start all booking threads

for (Thread t : users) {

    t.start();
}

// Wait for all threads

for (Thread t : users) {
    t.join();
}

long endTime = System.nanoTime();

double timeTaken =
  
    (endTime - startTime) / 1_000_000.0;

System.out.println(
    
    "Execution time: " + timeTaken + " ms"
);

Comparison:

1. Without Synchronization

•	Threads can execute the booking operation simultaneously.

•	It may have less locking overhead.

•	It can produce incorrect results due to race conditions.

•	It is not suitable for a real cinema booking system.

2. Synchronized Method

•	Locks the entire booking method.

•	Only one thread can execute the method on the same object at a time.

•	It provides reliable booking results.

•	Other threads must wait even when they are booking different seats.

3. Synchronized Block

•	Locks only the critical section.

•	It protects the check-and-book operation.

•	It allows code outside the synchronized block to execute without acquiring the lock.

•	It offers greater flexibility than a synchronized method.

Observation: A synchronized block can reduce unnecessary locking when a method contains additional work outside the critical section. However, in this particular 
example, both synchronized versions protect nearly the same amount of code using the same object lock. Therefore, their performance may be very similar.

7. Intrinsic Lock and Static Synchronization

Intrinsic Lock:

•	Every Java object has an intrinsic lock, also called a monitor lock.

•	A synchronized instance method acquires the lock of the current object (this).

•	A synchronized block using synchronized(this) acquires the same object lock.

•	Only one thread can hold that particular lock at a time.

•	The lock is automatically released when the synchronized method or block finishes, including when an exception occurs.

Static Synchronization:

•	A static synchronized method locks the class object instead of an individual object.

•	Its lock is associated with SeatBooking.class.

•	It is useful when multiple objects share static booking information that must be protected by the same lock.

•	In a cinema application, it may be needed if all screen objects modify a shared static seat inventory or booking counter.

•	It is not necessary when each screen has its own independent booking array and counter.

Example:

public static synchronized void bookSeat() {

    // Access shared static booking data
}


8. Joint Engineering Decision

For a cinema with 20 screens, I would deploy a synchronized block or a per-seat concurrency mechanism rather than using one global lock for all screens.

Reasons:

•	Each screen should maintain an independent seat inventory.

•	Bookings for different screens should be processed concurrently.

•	Locking only the necessary critical section reduces unnecessary waiting.

•	A per-seat mechanism can allow users to book different seats simultaneously.

•	Synchronization prevents duplicate bookings and maintains data consistency.

Final decision: For a larger cinema booking system, using a separate concurrency control mechanism for each seat or screen provides better scalability than using a single shared lock for the entire cinema.

9. Alternative Concept: Using AtomicBoolean, ReentrantLock and ConcurrentHashMap

Java provides other concurrency mechanisms that can prevent two users from booking the same seat.

A. Using AtomicBoolean[]

AtomicBoolean provides an atomic compareAndSet() operation. It checks the current value and changes it only if the expected value matches.

B. Using ReentrantLock

ReentrantLock provides explicit locking and unlocking operations.

Example:

import java.util.concurrent.locks.ReentrantLock;

public class LockSeatBooking {

    static boolean[] booked = new boolean[10];

    static ReentrantLock lock = new ReentrantLock();

    public static void book(int seat) {

        lock.lock();

        try {
            if (!booked[seat]) {
                booked[seat] = true;
                System.out.println("Seat " + seat +
                                   " booked successfully.");
            } else {
                System.out.println("Seat already booked.");
            }
        }
        finally {
            lock.unlock();
        }
    }

    public static void main(String[] args) {
        book(5);
        book(5);
    }
}

Explanation:

•	lock() acquires the lock before accessing shared data.

•	Only one thread can enter the protected section at a time using this lock.

•	unlock() releases the lock.

•	The finally block ensures that the lock is released even if an exception occurs.

C. Using ConcurrentHashMap with putIfAbsent()

ConcurrentHashMap is a thread-safe collection that supports concurrent operations.

Example:

import java.util.concurrent.ConcurrentHashMap;

public class MapSeatBooking {
        
        static ConcurrentHashMap<Integer, Boolean> seats =
        new ConcurrentHashMap<>();

    public static void main(String[] args) {

        int seat = 5;

        Boolean previous =
            seats.putIfAbsent(seat, true);

        if (previous == null) {
            System.out.println("Seat booked successfully.");
        } else {
            System.out.println("Seat already booked.");
        }
    }
}

Explanation:

•	putIfAbsent() inserts a seat only if it does not already exist.

•	The operation is atomic.

•	If the seat is already present, the existing value is returned.

•	This prevents two users from successfully inserting the same seat.

10. Complexity Analysis

Let:

•	s = Total number of seats.

•	n = Total number of booking requests.

•	p = Number of threads competing for seats.

Time Complexity:

•	AtomicBoolean[]: Each booking attempt takes O(1) time for compareAndSet() under normal conditions. Under contention, threads may experience retries or waiting.

•	ReentrantLock: Each booking attempt takes O(1) time for the seat check and update, excluding lock-waiting time. High contention can increase waiting time.

•	ConcurrentHashMap: putIfAbsent() takes approximately O(1) expected time per booking under normal hash distribution. Contention and hash collisions may increase 
execution time.

Space Complexity:

•	AtomicBoolean[]:

o	Stores one AtomicBoolean object for each seat.

o	Requires O(s) space.

•	ReentrantLock:

o	Uses a lock object and the seat array.

o	Requires O(s) space for the seat array and O(1) additional lock space.

•	ConcurrentHashMap:

o	Stores seat identifiers and their booking status.

o	Requires O(s) space when all seats are represented in the map.

Overall observation:

•	AtomicBoolean is useful when each seat needs independent atomic booking.

•	ReentrantLock is useful when more control over locking is required.

•	ConcurrentHashMap is useful when seat information needs to be managed through a concurrent key-value collection.

11. Final Conclusion

The cinema seat-booking system demonstrates how race conditions occur when multiple threads access shared data simultaneously.

•	Without synchronization, two users may book the same seat.

•	A synchronized method protects the entire booking operation.

•	A synchronized block protects only the critical section.

•	Intrinsic locks ensure that only one thread at a time executes the protected section using the same lock.

•	Static synchronization is useful when multiple objects access shared static data.

•	AtomicBoolean, ReentrantLock and ConcurrentHashMap provide alternative ways to achieve thread safety.












