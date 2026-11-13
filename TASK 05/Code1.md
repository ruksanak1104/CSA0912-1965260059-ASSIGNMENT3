public class SeatBookingWithoutSync {

    private final boolean[] booked;
    private int bookingsMade = 0;

    SeatBookingWithoutSync(int seats) {
        booked = new boolean[seats];
    }

    public void book(int seat) {

        if (!booked[seat]) {

            // Simulate delay to increase the chance
            // of a race condition
            try {
                Thread.sleep(10);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                return;
            }

            booked[seat] = true;
            bookingsMade++;

            System.out.println(
                Thread.currentThread().getName() +
                " booked seat " + seat
            );
        }
    }

    public static void main(String[] args)
            throws InterruptedException {

        SeatBookingWithoutSync screen =
            new SeatBookingWithoutSync(10);

        Thread[] users = new Thread[100];

        for (int i = 0; i < users.length; i++) {

            final int seat = i % 10;

            users[i] = new Thread(() -> screen.book(seat));
            users[i].start();
        }

        for (Thread t : users) {
            t.join();
        }

        System.out.println("Seats sold: " +
                           screen.bookingsMade);
    }
}

