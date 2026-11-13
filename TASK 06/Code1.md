import java.util.concurrent.CountDownLatch;

class Account {
    
    int balance;

    Account(int balance) {
        this.balance = balance;
    }
}

public class Main {
   
    public static void main(String[] args) throws InterruptedException {

        Account A = new Account(1000);
        Account B = new Account(1000);

        CountDownLatch ready = new CountDownLatch(2);

        Thread t1 = new Thread(() -> {
            synchronized (A) {
                System.out.println("Thread 1 locked Account A");

                ready.countDown();
                try {
                    ready.await();
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                    return;
                }

                synchronized (B) {
                    A.balance -= 100;
                    B.balance += 100;
                }
            }
        });

        Thread t2 = new Thread(() -> {
            synchronized (B) {
                System.out.println("Thread 2 locked Account B");

                ready.countDown();
                try {
                    ready.await();
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                    return;
                }

                synchronized (A) {
                    B.balance -= 200;
                    A.balance += 200;
                }
            }
        });

        t1.setDaemon(true);
        t2.setDaemon(true);

        t1.start();
        t2.start();

        Thread.sleep(1000);

        System.out.println("Thread 1 alive: " + t1.isAlive());
        System.out.println("Thread 2 alive: " + t2.isAlive());
        System.out.println("Both threads are blocked due to deadlock.");
    }
}

