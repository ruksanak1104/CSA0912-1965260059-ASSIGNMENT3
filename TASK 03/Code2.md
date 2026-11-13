import java.io.*;

public class SalesReportAuto {

    public static void main(String[] args) {

        String[] files = {
            "store1.txt",
            "store2.txt",
            "store3.txt"
        };

        int totalSales = 0;

        try (BufferedWriter bw = new BufferedWriter(
                new FileWriter("summary.txt"))) {

            for (String file : files) {

                System.out.println("Reading: " + file);

                try (BufferedReader br = new BufferedReader(
                        new FileReader(file))) {

                    String line;

                    while ((line = br.readLine()) != null) {
                        totalSales += Integer.parseInt(line);
                    }
                }
            }

            bw.write("Total Sales: " + totalSales);
            bw.newLine();

            System.out.println("Report generated successfully!");
            System.out.println("Total Sales: " + totalSales);

        }
        catch (IOException | NumberFormatException e) {
            System.out.println("Error: " + e.getMessage());
        }
    }
}

