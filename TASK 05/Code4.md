import java.util.concurrent.atomic.AtomicBoolean;

public class AtomicSeatBooking {

    static AtomicBoolean[] seats = new AtomicBoolean[10];

    public static void main(String[] args) {

        for (int i = 0; i < seats.length; i++) {
            seats[i] = new AtomicBoolean(false);
        }

        int seat = 5;

        if (seats[seat].compareAndSet(false, true)) {
            System.out.println("Seat booked successfully.");
        } else {
            System.out.println("Seat already booked.");
        }
    }
}

