class ScannerTask implements Runnable {

    private int stationNumber;
    private int scripts;

    ScannerTask(int stationNumber, int scripts) {
        this.stationNumber = stationNumber;
        this.scripts = scripts;
    }

    public void run() {
        try {
            System.out.println("Station " + stationNumber +
                               " started scanning.");

            Thread.sleep(1000 + stationNumber * 300);

            System.out.println("Station " + stationNumber +
                               " scanned " + scripts + " scripts.");

        } catch (InterruptedException e) {
            System.out.println("Station interrupted.");
        }
    }
}

public class ScannerRunnable {

    public static void main(String[] args) {

        Thread[] stations = new Thread[4];

        stations[0] = new Thread(new ScannerTask(1, 10));
        stations[1] = new Thread(new ScannerTask(2, 15));
        stations[2] = new Thread(new ScannerTask(3, 20));
        stations[3] = new Thread(new ScannerTask(4, 25));

        for (Thread station : stations) {
            station.start();
        }

        for (Thread station : stations) {
            System.out.println("Is station alive? " +
                               station.isAlive());
        }

        try {
            for (Thread station : stations) {
                station.join();
            }
        } catch (InterruptedException e) {
            System.out.println("Main thread interrupted.");
        }

        int total = 10 + 15 + 20 + 25;

        System.out.println("All stations completed.");
        System.out.println("Total scripts scanned: " + total);
    }
}

