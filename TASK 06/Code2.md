class Account {
    int id;
    
    int balance;

    Account(int id, int balance) {
        this.id = id;
        this.balance = balance;
    }
}

public class Main {

    static void transfer(Account from, Account to, int amount) {

        Account first = from.id < to.id ? from : to;
        Account second = from.id < to.id ? to : from;

        synchronized (first) {
            synchronized (second) {

                if (from.balance >= amount) {
                    from.balance -= amount;
                    to.balance += amount;

                    System.out.println("Transfer successful: " + amount);
                } else {
                    System.out.println("Insufficient balance");
                }
            }
        }
    }

    public static void main(String[] args) {

        Account A = new Account(1, 1000);
        Account B = new Account(2, 1000);

        Thread t1 = new Thread(() -> transfer(A, B, 100));
        Thread t2 = new Thread(() -> transfer(B, A, 200));

        t1.start();
        t2.start();
    }
}

