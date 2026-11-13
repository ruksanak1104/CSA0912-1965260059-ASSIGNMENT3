import java.util.*;

public class ATMWithoutException {

    public static void main(String[] args) {

        Scanner sc = new Scanner(System.in);

        int balance = 5000;

        System.out.print("Enter Account Number: ");

        if (!sc.hasNextInt()) {
            System.out.println("Invalid account number");
            return;
        }

        int acc = sc.nextInt();

        System.out.print("Enter Withdrawal Amount: ");

        if (!sc.hasNextInt()) {
            System.out.println("Invalid amount");
            return;
        }

        int amount = sc.nextInt();

        int result = withdraw(balance, amount);

        if (result == -1) {
            System.out.println("Invalid withdrawal amount");
        }
        else if (result == -2) {
            System.out.println("Insufficient balance");
        }
        else {
            System.out.println("Withdrawal successful!");
            System.out.println("Remaining balance: " + result);
        }

        sc.close();
    }

    static int withdraw(int balance, int amount) {

        if (amount < 0)
            return -1;

        if (amount > balance)
            return -2;

        return balance - amount;
    }
}

