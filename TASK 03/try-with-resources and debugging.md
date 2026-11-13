Aim

To implement a daily sales report generator using manual resource management with try-catch-finally and automatic resource management using try-with-resources. To compare both approaches based on resource safety, debugging, file handling and time and space complexity.

1. Daily Sales Report Generator Using try-catch-finally

In the manual approach, files are opened using FileReader and BufferedReader. They are closed explicitly using the close() method inside the finally block.

File name: SalesReportManual.java

2. Daily Sales Report Generator Using Try-With-Resources

Try-with-resources automatically closes resources after the try block finishes, even when an exception occurs.

File name: SalesReportAuto.java

Explanation:

•	BufferedReader reads data from each store file.

•	BufferedWriter writes the summary report.

•	The try-with-resources statement automatically closes the reader and writer.

•	Even if an exception occurs, the opened resources are closed automatically.

•	No explicit close() method is required.

Expected output for valid input:

Reading: store1.txt

Reading: store2.txt

Reading: store3.txt

Report generated successfully!

Total Sales: 6000

3. Sample Input Files

For online compiler execution, we can create the store files automatically using Java instead of uploading them separately.

Use the following sample data:

store1.txt

1000

1500

store2.txt

1200

800

store3.txt

500

1000

Expected total:

•	Store 1: 2500

•	Store 2: 2000

•	Store 3: 1500

•	Total sales: 6000

For testing, add an invalid entry such as ABC to store2.txt. This causes a NumberFormatException during processing.

4. Debugging and Experimental Analysis

Student A must identify the resource leak and examine the behaviour when an exception occurs.

A. Locating the Resource Leak

In the manual version, place breakpoints at:
1.	The new FileReader(file) statement.

2.	The readLine() statement.

3.	The Integer.parseInt(line) statement.

4.	The br.close() statement.

5.	The finally block.

Use step-through debugging to observe whether execution reaches the close() statement.

B. Input That Causes Failure

Use the following input:

•	Store 1: 1000, 1500

•	Store 2: 1200, ABC

•	Store 3: 500, 1000

When the program reads ABC, a NumberFormatException occurs.
In a manual implementation where the reader is closed only after the reading loop, execution skips the br.close() statement.
If the reader is not properly closed in the finally block, the file handle remains open.
In try-with-resources, the reader closes automatically even when the exception occurs.

C. Counting Open Files

For the same failing input:

•	Manual version without proper cleanup:

o	Store 1 reader is closed after processing.

o	Store 2 reader remains open when the exception occurs.

o	The writer must also be closed separately.

o	One reader resource leaks.

•	Try-with-resources version:

o	Store 1 reader is closed after processing.

o	Store 2 reader is closed automatically when the exception occurs.

o	The writer is also closed automatically.

o	No reader or writer remains open after cleanup.

D. Logging the File Causing Failure

Add the following statement before reading each file:

System.out.println("Processing file: " + file);

Inside the catch block, use:

System.out.println("Failed while processing: " + file);

This helps identify the store file responsible for the failure.

Example output:

Processing file: store1.txt

Processing file: store2.txt

Failed while processing: store2.txt

Error: For input string: "ABC"

5. Resources Opened and Their Closing Requirements

The daily sales report generator uses the following resources:

•	BufferedReader

o	Used to read sales data from store files.

o	Must be closed after processing each store file.

•	FileReader

o	Opens the store file for reading.

o	Closed automatically when its wrapping BufferedReader is closed.

•	BufferedWriter

o	Used to write the summary report.

o	Must be closed after writing the report.

•	FileWriter

o	Opens the summary file for writing.

o	Closed automatically when its wrapping BufferedWriter is closed.

6. Why Can the Manual Version Leak Resources?

In manual resource management, the programmer must explicitly close every opened resource using close().

A resource leak can occur when:

1.	A file is opened successfully.

2.	An exception occurs while reading or processing the file.

3.	Program execution skips the normal close() statement.

4.	The resource remains open if no suitable cleanup is performed.

This can cause the operating system to reach its maximum file-handle limit and produce a Too many open files error.

7. How Does Try-With-Resources Guarantee Closing?

Try-with-resources automatically closes all declared resources after the try block finishes.

Important points:

•	Resources must implement AutoCloseable or Closeable.

•	Resources are closed automatically after normal execution or an exception.

•	Multiple resources are closed in the reverse order of their declaration.

•	It reduces the possibility of resource leaks.

•	It eliminates the need for most manual close() statements.

Example:

try {
   
    FileReader fr = new FileReader("store1.txt");
    
    BufferedReader br = new BufferedReader(fr)
 
 }{
 
    System.out.println(br.readLine());
}

Closing order:

1.	BufferedReader is closed first.

2.	FileReader is closed next.

This is reverse declaration order.

8. What Is a Suppressed Exception?

A suppressed exception occurs when an exception is thrown while another exception is already being handled during try-with-resources cleanup.

For example:

•	An exception occurs while reading a file.

•	Another exception occurs while closing the file.

•	The reading exception remains the primary exception.

•	The closing exception becomes a suppressed exception.

It can be accessed using:

for (Throwable t : e.getSuppressed()) {

    System.out.println(t);
}


Importance:

•	Preserves the original exception.

•	Keeps information about resource-closing failures.

•	Makes debugging easier.

•	Prevents the closing exception from replacing the main exception.

9. Joint Engineering Decision

Under what conditions is try-with-resources not sufficient on its own to make the report generator robust?

Try-with-resources guarantees resource closure, but it does not prevent every possible program failure.

It is not sufficient in the following situations:

1.	Invalid input data: If a store file contains letters instead of numbers, a NumberFormatException may occur.

2.	Missing files: If a store file does not exist, a FileNotFoundException occurs.

3.	Incomplete report: If an exception occurs while processing a store file, the report may not contain all store data.

4.	Disk space problems: Insufficient storage space may prevent the summary file from being written.

5.	Permission problems: The program may not have permission to read or write certain files.

6.	Resource closing failures: Closing a resource can itself produce an exception.

Conclusion: Try-with-resources manages resource cleanup automatically, but proper validation, exception handling, logging and recovery mechanisms are also 
required to make the report generator reliable.

10. Alternative Concept: Explicit Close() in Finally or No Closing

There are three approaches to resource management.

A. Explicit close() using finally

•	Resources are closed manually inside the finally block.

•	The programmer must ensure that every opened resource is closed.

•	If one closing operation fails, other resources must still be closed using separate cleanup handling.

•	It is more complex than try-with-resources.

B. Try-with-resources

•	Resources are closed automatically.

•	Resources are closed in reverse declaration order.

•	It handles exceptions during processing and cleanup.

•	It reduces the possibility of resource leaks.

C. Without Closing Resources

•	Opened files remain allocated until the operating system or JVM releases them.

•	Repeated file operations may exhaust available file handles.

•	The program may produce a Too many open files error.

•	It is not a recommended approach.





11. Time and Space Complexity Analysis

Let f represent the number of store files and n represent the total number of sales records across all files.

Time Complexity:

•	Reading each file requires processing its records.

•	Processing all records takes O(n).

•	Writing the final summary takes O(1) for a fixed-size summary.

•	Therefore, the overall time complexity is O(n + f), including per-file opening and closing overhead.

Space Complexity:
•	Only a limited amount of data is processed at a time.

•	BufferedReader and BufferedWriter use fixed-size buffers.

•	Therefore, auxiliary space complexity is O(1), excluding the input files and output file storage.

Comparison of Resource Management

1. Manual close() using finally

•	Maximum open file handles during normal processing: 2 (one reader and one writer).

•	During a failure, resources may remain open if cleanup is incomplete.

•	Requires explicit resource management.

2. Try-with-resources

•	Maximum open file handles during normal processing: 2 (one reader and one writer).

•	During a failure, opened resources are closed automatically.

•	No file handles remain open after the try-with-resources block completes, assuming normal JVM cleanup execution.

3. Without closing resources

•	Maximum open file handles can increase with the number of files processed.

•	Resources may accumulate until the operating system limit is reached.

•	Can cause a Too many open files error.

Final Comparison

•	Manual finally: Requires careful cleanup and may leak resources if implemented incorrectly.

•	Try-with-resources: Automatically closes resources and provides safer resource management.

•	No closing: Can exhaust system resources and cause program failure.

Conclusion

Try-with-resources is a safer and more reliable approach for generating daily sales reports because it automatically closes opened resources, even when exceptions 
occur.
Manual resource management requires explicit closing and careful cleanup. Not closing resources can cause resource leaks and system failures.
Therefore, try-with-resources combined with proper exception handling, input validation and logging is the recommended approach for developing a reliable sales 
report generator.









