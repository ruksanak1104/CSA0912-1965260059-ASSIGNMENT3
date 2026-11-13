import java.util.concurrent.locks.ReentrantLock;

import java.util.concurrent.TimeUnit;

class Account {
    
    int balance;
    
    ReentrantLock lock = new ReentrantLock();

    Account(int balance) {
        this.balance = balance;
    }
}

public class Main {

    static boolean transfer(Account from, Account to, int amount)
            throws InterruptedException {

        if (!from.lock.tryLock(500, TimeUnit.MILLISECONDS)) {
            return false;
        }

        try {
            if (!to.lock.tryLock(500, TimeUnit.MILLISECONDS)) {
                return false;
            }

            try {
                if (from.balance >= amount) {
                    from.balance -= amount;
                    to.balance += amount;
                    return true;
                }

                return false;

            } finally {
                to.lock.unlock();
            }

        } finally {
            from.lock.unlock();
        }
    }

    public static void main(String[] args)
            throws InterruptedException {

        Account A = new Account(1000);
        Account B = new Account(1000);

        Thread t1 = new Thread(() -> {
            try {
                System.out.println(
                    "Transfer 1: " + transfer(A, B, 100));
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });

        Thread t2 = new Thread(() -> {
            try {
                System.out.println(
                    "Transfer 2: " + transfer(B, A, 200));
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });

        t1.start();
        t2.start();
    }
}

