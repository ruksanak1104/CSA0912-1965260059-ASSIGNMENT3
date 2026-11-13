import java.io.File;

import java.util.concurrent.atomic.AtomicInteger;

public class Main {

    public static void main(String[] args)
            throws InterruptedException {

        AtomicInteger success = new AtomicInteger(0);
        AtomicInteger failure = new AtomicInteger(0);

        Thread[] threads = new Thread[10000];

        for (int i = 0; i < 10000; i++) {

            final int id = i + 1;

            threads[i] = new Thread(() -> {

                File file = new File(
                    "scores/student" + id + ".txt");

                try {
                    if (file.exists()) {
                        success.incrementAndGet();
                    } else {
                        failure.incrementAndGet();
                    }

                } catch (Exception e) {
                    failure.incrementAndGet();
                }
            });

            threads[i].start();
        }

        for (Thread t : threads) {
            t.join();
        }

        System.out.println(
            "Successful files: " + success.get());

        System.out.println(
            "Failed files: " + failure.get());
    }
}

