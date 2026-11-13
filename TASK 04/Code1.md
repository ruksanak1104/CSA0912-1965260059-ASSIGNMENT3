class ScannerStation extends Thread {

    private int stationNumber;
    
    private int scripts;
    
    ScannerStation(int stationNumber, int scripts) {
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

public class ScannerThread {

    public static void main(String[] args) {

        ScannerStation[] stations = {
            new ScannerStation(1, 10),
            new ScannerStation(2, 15),
            new ScannerStation(3, 20),
            new ScannerStation(4, 25)
        };

        for (ScannerStation station : stations) {
            station.start();
        }

        for (ScannerStation station : stations) {
            System.out.println("Is station alive? " +
                               station.isAlive());
        }

        try {
            for (ScannerStation station : stations) {
                station.join();
            }
        } catch (InterruptedException e) {
            System.out.println("Main thread interrupted.");
        }

        int total = 0;

        for (int scripts : new int[]{10, 15, 20, 25}) {
            total += scripts;
        }

        System.out.println("All stations completed.");
        System.out.println("Total scripts scanned: " + total);
    }
}





