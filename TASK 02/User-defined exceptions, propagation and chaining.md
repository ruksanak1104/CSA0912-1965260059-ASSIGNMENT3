Aim

To design and implement user-defined exceptions in an online examination portal using exception hierarchy, throw, throws, exception propagation and exception chaining in Java.

1. User-Defined Exception Hierarchy

The online examination portal uses custom exceptions to handle different submission failures.

Exception hierarchy:
                 
                  Throwable
                      |
                  Exception
                      |
                 ExamException
                 /     |      \
                /      |       \
 ExamWindowClosed  StudentNot  SubmissionStorage
     Exception     Registered   Exception
                       Exception

Additional unchecked exception:
             
              RuntimeException
                     |
          InvalidSubmissionException

Explanation of exceptions:

•	ExamException: Parent class for all examination-related exceptions. It is a checked exception.

•	ExamWindowClosedException: Occurs when a student submits answers after the examination deadline. It is a checked exception.

•	StudentNotRegisteredException: Occurs when a student is not registered for the course. It is a checked exception.

•	SubmissionStorageException: Occurs when the answer file cannot be saved because of storage-related problems. It is a checked exception.

•	InvalidSubmissionException: Occurs when the program receives invalid submission data, such as a negative file size. It is an unchecked exception.

2. Reason for Making Exceptions Checked or Unchecked

Checked Exceptions:

•	ExamWindowClosedException: The examination deadline can be checked, and the application must inform the student.

•	StudentNotRegisteredException: Registration status must be verified before accepting the submission.

•	SubmissionStorageException: Storage failures must be handled by the application so that the student receives an appropriate message.

Unchecked Exception:

- InvalidSubmissionException: Used for invalid program input or an invalid submission state that should be prevented through validation.

Checked exceptions force the programmer to handle or declare expected application failures, whereas unchecked exceptions do not require mandatory handling.


Explanation of the Program

1.	ExamException is the parent class of the custom examination exceptions.

2.	SubmissionStorageException extends ExamException.

3.	The save() method uses throws SubmissionStorageException to declare that it may produce a storage exception.

4.	If the file size exceeds 500 KB, an IOException is generated using throw.

5.	The catch block catches the IOException.

6.	A new SubmissionStorageException is created with the original exception passed as its cause.

7.	The main method catches the parent ExamException and displays the error message and original cause.

4. Exception Propagation Across Three Layers

The online examination portal contains three layers:

•	Controller Layer: Receives the student's submission and handles exceptions.

•	Service Layer: Checks examination timing and student registration.

•	Storage Layer: Saves the answer file and handles storage-related failures.

Propagation flow:

       Controller Layer
              |
              v
        Service Layer
              |
              v
        Storage Layer
              |
              v
       IOException occurs
              |
              v
  SubmissionStorageException
       is created using
       original IOException
              |
              v
        Service Layer
       propagates exception
              |
              v
      Controller Layer
        catches exception
              |
              v
       Error message shown
         to the student

Explanation:

1.	The controller receives the student's submission.

2.	The service layer checks the exam deadline and registration status.

3.	The storage layer attempts to save the answer file.

4.	If storage fails, an IOException occurs.

5.	The storage layer wraps it inside SubmissionStorageException.

6.	The service layer propagates the exception using throws.

7.	The controller catches the exception and displays a meaningful error message.



5. Demonstrating Exception Propagation Using Three Layers

The following method structure shows how throw and throws are used across the three layers.

Controller Layer:

void submitExam(String studentId, String fileName)

        throws ExamException {

    service.processSubmission(studentId, fileName);
}

Service Layer:

void processSubmission(String studentId, String fileName)
      
        throws ExamException {

    if (!registered) {
        throw new StudentNotRegisteredException(
            "Student is not registered");
    }

    storage.save(fileName, 120);
}

Storage Layer:

void save(String fileName, long sizeKb)
        throws SubmissionStorageException {

    try {
        if (sizeKb > 500) {
            throw new IOException("Disk quota exceeded");
        }

    } catch (IOException e) {
        throw new SubmissionStorageException(
            "Could not store " + fileName, e);
    }
}

6. Exception Chaining Using getCause()

Exception chaining means wrapping one exception inside another while preserving the original exception.

In the given program:

throw new SubmissionStorageException(

    "Could not store " + fileName, e);

Here:

•	SubmissionStorageException is the new exception.

•	IOException is the original exception.

•	The original exception is passed through the constructor using e.

The getCause() method retrieves the original exception.

System.out.println("Cause: " +

                   e.getCause().getMessage());

Output:

Error: Could not store answer02.pdf

Cause: Disk quota exceeded

Full Chained Stack Trace

If the catch block contains:

e.printStackTrace();

The output will contain information similar to:

SubmissionStorageException:

Could not store answer02.pdf

Caused by: java.io.IOException:

Disk quota exceeded

The complete stack trace also contains the class names, method names and line numbers where the exceptions occurred.

Explanation:

1.	SubmissionStorageException: Identifies the custom exception.

2.	Could not store answer02.pdf: Displays the custom error message.

3.	Caused by: Indicates that another exception caused the current exception.

4.	IOException: Identifies the original exception.

5.	Disk quota exceeded: Explains the original reason for failure.

7. What Happens if the Original Cause Is Not Passed?

Suppose the constructor is called without passing e:

throw new SubmissionStorageException(

    "Could not store " + fileName, null);
Now 
the original IOException is not preserved.

If the program executes:

System.out.println("Cause: " +

                   e.getCause().getMessage());
It produces a NullPointerException because getCause() returns null.

What information is lost?

•	The original exception type (IOException).

•	The original error message (Disk quota exceeded).

•	The original cause of the storage failure.

•	The original exception's stack-trace information.

Therefore, passing the original exception as the cause is important for debugging.


8. Testing and Observed Behaviour

The following scenarios demonstrate different exception paths.

Test Case 1: Successful Submission

•	Input: File name = answer01.pdf, Size = 120 KB.

•	Expected output: Saved answer01.pdf

•	Exception generated: None.

•	Result: Submission saved successfully.

Test Case 2: File Exceeds Size Limit

•	Input: File name = answer02.pdf, Size = 800 KB.

•	Expected output: Error: Could not store answer02.pdf

•	Exception generated: IOException, wrapped inside SubmissionStorageException.

•	Result: Submission rejected.

Test Case 3: Exam Window Closed

•	Condition: Student submits after the examination deadline.

•	Exception generated: ExamWindowClosedException.

•	Result: Submission rejected with an examination deadline message.

Test Case 4: Student Not Registered

•	Condition: Student is not registered for the course.

•	Exception generated: StudentNotRegisteredException.

•	Result: Submission rejected with a registration error message.

Test Case 5: Invalid Submission Data

•	Condition: File size is negative.

•	Exception generated: InvalidSubmissionException.

•	Result: Submission rejected with an invalid input message.

9. Joint Conclusion

Why is it better for the storage layer to throw a meaningful SubmissionStorageException than to let IOException travel unchanged to the user interface?

•	It provides a meaningful error message related to the examination portal.

•	It hides unnecessary low-level storage implementation details from the user interface.

•	It makes debugging easier by preserving the original exception as the cause.

•	It separates storage-related errors from general application errors.

•	It improves code readability and maintainability.

•	It allows the controller to handle different examination failures appropriately.

Conclusion: Using meaningful user-defined exceptions with exception chaining improves error handling, debugging and communication between different layers of an 
application.

10. Alternative Concept: Making Custom Exceptions Unchecked

In the given program, ExamException extends Exception, so it is a checked exception.
If we change it to extend RuntimeException, it becomes an unchecked exception.

Original checked exception:

class ExamException extends Exception {

    ExamException(String message) {
    
        super(message);
    }
}


Modified unchecked exception:

class ExamException extends RuntimeException {
    ExamException(String message) {
        super(message);
    }
}

Changes in Method Signatures

When using checked exceptions:

•	Methods must either handle the exception using try-catch or declare it using throws.

•	The compiler forces exception handling or declaration.

•	The calling method must handle or propagate the exception.

When using unchecked exceptions:

•	Methods do not need to declare exceptions using throws.

•	The calling method is not forced to handle the exception.

•	Exceptions can propagate automatically until they are caught.

•	The programmer can still use try-catch whenever required.

Comparison

•	Checked Exceptions: Mandatory handling or declaration.

•	Unchecked Exceptions: Optional handling or declaration.

•	Checked Exceptions: Suitable for expected application failures.

•	Unchecked Exceptions: Suitable for programming errors and invalid states.

11. Time and Space Complexity Analysis

Exception Handling:

•	Time complexity: Creating an exception and capturing its stack trace generally takes time proportional to the stack depth, O(d), where d is the number of stack frames.

•	Space complexity: Requires memory for the exception object, stack trace and associated information, generally O(d).

Exception Chaining:

•	Time complexity: Linking the original exception as a cause takes O(1) additional work.

•	Space complexity: The new exception stores a reference to the original exception. Both exception objects and their stack traces remain in memory while reachable.

Comparison:

•	Creating an exception:

o	Requires memory for the exception object.

o	Captures stack-trace information.

•	Throwing an exception:

o	Requires processing to locate the appropriate catch block.

o	Adds execution overhead.

•	Exception chaining:

o	Preserves the original exception.

o	Requires an additional reference to the cause.

o	Retains both exceptions and their associated stack traces.

•	Unchecked exceptions:

o	Also require stack-trace creation when instantiated.

o	Do not eliminate the runtime cost of throwing exceptions.

Conclusion

User-defined exceptions provide a structured way to handle failures in an online examination portal. Exception propagation allows errors to travel between the 
controller, service and storage layers.
Exception chaining preserves the original cause of an error, making debugging easier. Checked exceptions ensure that important application failures are handled, while unchecked exceptions are useful for programming errors.
Therefore, using custom exceptions, throw, throws and exception chaining improves the reliability, readability and maintainability of the examination portal.




