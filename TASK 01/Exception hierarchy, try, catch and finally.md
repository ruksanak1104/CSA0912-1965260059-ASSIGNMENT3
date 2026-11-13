Aim

To analyse different failure conditions in an ATM withdrawal system and implement exception handling using try, catch, multi-catch and finally blocks in Java.

1. Throwable Hierarchy

The Java Throwable class is the root class of all exceptions and errors.

Relevant Throwable hierarchy:
                 
                  Throwable
                 /         \
              Error       Exception
                           /       \
                  IOException    RuntimeException
                                      |
                              Unchecked Exceptions

Examples relevant to the ATM program:

•	Error: OutOfMemoryError

•	Checked Exception: IOException, FileNotFoundException

•	Unchecked Exception: InputMismatchException, IllegalArgumentException, ArithmeticException

2. Failure Point and Exception Classification

•	Letters entered instead of account number

o	Exception type: InputMismatchException

o	Classification: Unchecked exception

o	Caught in: Multi-catch block.

•	Letters entered instead of withdrawal amount

o	Exception type: InputMismatchException

o	Classification: Unchecked exception

o	Caught in: Multi-catch block.

•	Negative withdrawal amount

o	Exception type: IllegalArgumentException

o	Classification: Unchecked exception

o	Caught in: IllegalArgumentException catch block.

•	Withdrawal amount greater than balance

o	Exception type: IllegalArgumentException

o	Classification: Unchecked exception

o	Caught in: IllegalArgumentException catch block.

•	Card-reader file missing

o	Exception type: FileNotFoundException

o	Classification: Checked exception

o	Caught in: FileNotFoundException catch block.

•	File reading failure

o	Exception type: IOException

o	Classification: Checked exception

o	Caught in: IOException catch block.

•	Insufficient memory

o	Exception type: OutOfMemoryError

o	Classification: Error

o	Caught in: Not normally caught.

4. Justification for Catch Block Order

Catch blocks must be arranged from more specific exceptions to more general exceptions.

1.	InputMismatchException | NumberFormatException: Handles invalid numeric input.

2.	IllegalArgumentException: Handles negative withdrawal amounts and insufficient balance.

3.	FileNotFoundException: Handles missing card-reader files.

4.	IOException: Handles general file input/output errors.

FileNotFoundException is a subclass of IOException. Therefore, it must appear before IOException.

If a broader exception is placed before a narrower exception, Java generates a compile-time error because the narrower catch block becomes unreachable.

5. Purpose of Finally Block

The finally block executes whether an exception occurs or not, provided the JVM continues normal execution.

In this ATM program, it:

•	Displays the transaction completion message.

•	Closes the Scanner resource.

•	Ensures cleanup happens after successful or unsuccessful transactions.

Important: A finally block is not an absolute guarantee if the JVM terminates abruptly.

6. Testing and Tracing

The following test cases cover different execution paths.

Test Case 1: Valid Input

•	Input: Account number = 101, Amount = 1000, file exists.

•	Expected output: Withdrawal successful.

•	Catch block executed: No catch block.

•	Finally block: Executed.

Test Case 2: Invalid Account Number

•	Input: Account number = ABC.

•	Expected output: Invalid input! Enter numbers only.

•	Catch block executed: InputMismatchException.

•	Finally block: Executed.

Test Case 3: Negative Withdrawal Amount

•	Input: Account number = 101, Amount = -500.

•	Expected output: Negative withdrawal amount is not allowed.

•	Catch block executed: IllegalArgumentException.

•	Finally block: Executed.

Test Case 4: Insufficient Balance

•	Input: Account number = 101, Amount = 6000.

•	Expected output: Insufficient balance.

•	Catch block executed: IllegalArgumentException.

•	Finally block: Executed.

Test Case 5: Missing Card-Reader File

•	Input: Account number = 101, Amount = 1000, file missing.

•	Expected output: Card reader file is missing!

•	Catch block executed: FileNotFoundException.

•	Finally block: Executed.

In all five cases, the finally block executes.

7. Stack Trace for an Uncaught Exception

Consider the following program:

public class Test {
   
    public static void main(String[] args) {
    
        int a = 10;
        
        int b = 0;

        int c = a / b;

        System.out.println(c);
    }
}

Output:

Exception in thread "main"

java.lang.ArithmeticException: / by zero

    at Test.main(Test.java:6)

Explanation:

1.	Exception in thread "main": Indicates that the exception occurred in the main thread.

2.	java.lang.ArithmeticException: Identifies the exception type.

3.	/ by zero: Describes the reason for the exception.

4.	at Test.main(Test.java:6): Shows that the exception occurred in the main method at line 6.

Since no catch block handles this exception, the JVM prints the stack trace and terminates the program.

8. What Happens if a Broader Catch Block Comes First?

Example:

try {
   
    FileReader fr = new FileReader("card.txt");
}

catch (IOException e) {

    System.out.println("File error");
}

catch (FileNotFoundException e) {

    System.out.println("File missing");
}

Result: Compile-time error.

Reason: FileNotFoundException is a subclass of IOException. The first catch block already handles it, making the second catch block unreachable.

Correct order:

catch (FileNotFoundException e) {
    
    System.out.println("File missing");
}

catch (IOException e) {

    System.out.println("File error");
}

9. Joint Design Question

Should the ATM program catch every exception using a single catch (Exception e)?
No. Catching every exception using a single catch block is generally not recommended.
When is it acceptable?

•	When a general fallback mechanism is required.

•	When logging unexpected exceptions.

•	At the top level of an application to prevent an unhandled exception from crashing the entire service.

What is lost by using a single catch block?

1.	Specific error messages cannot be provided easily.

2.	Different exceptions cannot be handled using different recovery methods.

3.	Debugging becomes difficult.

4.	Programming errors may be hidden.

5.	The program may not respond appropriately to different failure conditions.

Conclusion: Specific catch blocks are better for an ATM because they allow different failures to be handled safely and clearly.

10. Alternative Concept: Exception-Free ATM Program

If exceptions are avoided, input validation can be performed using Scanner.hasNextInt() and error codes.

11. Comparison Between Exception Handling and Error Codes

•	Readability

o	Exception Handling: Separates normal logic from error handling.

o	Error Codes: Requires multiple validation checks.

•	Execution

o	Exception Handling: Exception objects are created and thrown.

o	Error Codes: Returns an error value.

•	Performance

o	Exception Handling: Throwing exceptions is relatively expensive.

o	Error Codes: Usually faster for routine validation.

•	Debugging

o	Exception Handling: Provides stack traces.

o	Error Codes: Requires manual error tracking.

•	Code Complexity

o	Exception Handling: Uses try-catch-finally.

o	Error Codes: Uses if-else conditions.

•	Best Suited For

o	Exception Handling: Unexpected or exceptional failures.

o	Error Codes: Expected validation failures.


12. Time and Space Complexity Analysis

Exception Handling:

•	Time complexity: Creating an exception takes time proportional to the stack trace depth in typical JVM implementations. Throwing and catching also adds overhead.

•	Space complexity: Requires memory for the exception object and its stack trace.

Error Code Approach:

•	Time complexity: Each validation check generally takes O(1) time. Returning an error code is also O(1).

•	Space complexity: O(1) additional space for simple validation and error codes.

Comparison:

•	Normal execution

o	Exception approach: Low overhead when no exception occurs.

o	Error-code approach: Low overhead.

•	Error execution

o	Exception approach: Higher overhead.

o	Error-code approach: Lower overhead.

•	Additional memory

o	Exception approach: Exception object and stack trace.

o	Error-code approach: Small integer value.

•	Validation

o	Exception approach: Can use exception-based handling.

o	Error-code approach: Requires explicit checks.


Conclusion

Exception handling provides a structured way to manage unexpected failures in an ATM system. Using specific catch blocks, multi-catch and finally ensures better error management and resource cleanup.

The error-code approach is simpler and more efficient for predictable validation failures. Therefore, a combination of normal validation checks and exception handling is suitable for developing a reliable ATM application.










