import java.io.*;

import java.util.*;

import java.util.concurrent.*;

public class Main {

    static class Result {
        String fileName;
        int total;
        String status;
        String message;

        Result(String fileName, int total,
               String status, String message) {
            this.fileName = fileName;
            this.total = total;
            this.status = status;
            this.message = message;
        }
    }

    static class FileProcessor implements Callable<Result> {

        private final File file;

        FileProcessor(File file) {
            this.file = file;
        }

        public Result call() {

            try (Scanner sc = new Scanner(file)) {

                int total = 0;
                int count = 0;

                while (sc.hasNextLine()) {

                    String line = sc.nextLine().trim();

                    if (line.isEmpty()) {
                        continue;
                    }

                    int mark = Integer.parseInt(line);

                    if (mark < 0 || mark > 100) {
                        throw new IllegalArgumentException(
                            "Invalid mark: " + mark);
                    }

                    total += mark;
                    count++;
                }

                if (count == 0) {
                    throw new IllegalArgumentException(
                        "Empty score file");
                }

                return new Result(
                    file.getName(),
                    total,
                    "SUCCESS",
                    "Total marks: " + total
                );

            } catch (FileNotFoundException e) {

                return new Result(
                    file.getName(), 0,
                    "FAILED", "File missing"
                );

            } catch (NumberFormatException e) {

                return new Result(
                    file.getName(), 0,
                    "FAILED", "Corrupt or non-numeric marks"
                );

            } catch (IOException | IllegalArgumentException e) {

                return new Result(
                    file.getName(), 0,
                    "FAILED", e.getMessage()
                );
            }
        }
    }

    public static void main(String[] args)
            throws Exception {

        File folder = new File("scores");
        folder.mkdir();

        // Create 10,000 sample file paths
        for (int i = 1; i <= 10000; i++) {

            File file = new File(
                folder, "student" + i + ".txt");

            // Leave every 1000th file missing
            if (i % 1000 == 0) {
                continue;
            }

            try (PrintWriter pw = new PrintWriter(file)) {

                if (i % 500 == 0) {
                    pw.println("ABC");
                } else {
                    pw.println(80);
                    pw.println(75);
                    pw.println(90);
                }
            }
        }

        int[] poolSizes = {1, 2, 4, 8};

        for (int poolSize : poolSizes) {

            ExecutorService executor =
                Executors.newFixedThreadPool(poolSize);

            ConcurrentHashMap<String, Integer> successful =
                new ConcurrentHashMap<>();

            ConcurrentLinkedQueue<String> failed =
                new ConcurrentLinkedQueue<>();

            List<Future<Result>> futures =
                new ArrayList<>();

            long start = System.nanoTime();

            for (int i = 1; i <= 10000; i++) {

                File file = new File(
                    folder, "student" + i + ".txt");

                futures.add(
                    executor.submit(new FileProcessor(file))
                );
            }

            for (Future<Result> future : futures) {

                Result result = future.get();

                if (result.status.equals("SUCCESS")) {

                    successful.put(
                        result.fileName, result.total);

                } else {

                    failed.add(
                        result.fileName + ": " + result.message);
                }
            }

            long end = System.nanoTime();

            executor.shutdown();

            System.out.println("\nThread Pool Size: " + poolSize);

            System.out.println(
                "Successful Files: " + successful.size());

            System.out.println(
                "Failed Files: " + failed.size());

            System.out.println(
                "Total Files: " +
                (successful.size() + failed.size()));

            System.out.printf(
                "Execution Time: %.2f ms%n",
                (end - start) / 1_000_000.0);
        }
    }
}

