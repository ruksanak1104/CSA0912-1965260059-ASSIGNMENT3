import java.util.*;

import java.io.*;

public class ATM {
    
    public static void main(String[] args) {

        Scanner sc = new Scanner(System.in);

        try {
            System.out.print("Enter Account Number: ");
            int acc = sc.nextInt();

            System.out.print("Enter Withdrawal Amount: ");
            int amount = sc.nextInt();

            int balance = 5000;

            if (amount < 0) {
                throw new IllegalArgumentException(
                    "Negative withdrawal amount is not allowed");
            }

            if (amount > balance) {
                throw new IllegalArgumentException(
                    "Insufficient balance");
            }

            FileReader fr = new FileReader("card.txt");

            System.out.println("Account Number: " + acc);
            System.out.println("Withdrawal Successful!");
            System.out.println("Remaining Balance: " +
                               (balance - amount));

            fr.close();

        }
        catch (InputMismatchException | NumberFormatException e) {
            System.out.println("Invalid input! Enter numbers only.");
        }
        catch (IllegalArgumentException e) {
            System.out.println("Transaction Failed: " + e.getMessage());
        }
        catch (FileNotFoundException e) {
            System.out.println("Card reader file is missing!");
        }
        catch (IOException e) {
            System.out.println("File reading error!");
        }
        finally {
            System.out.println("ATM transaction process completed.");
            sc.close();
        }
    }
}

