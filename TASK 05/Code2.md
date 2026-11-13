public class SeatBookingMethod {

    private final boolean[] booked;
    private int bookingsMade = 0;

    SeatBookingMethod(int seats) {
        booked = new boolean[seats];
    }

    public synchronized void book(int seat) {

        if (!booked[seat]) {

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

        SeatBookingMethod screen =
            new SeatBookingMethod(10);

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

