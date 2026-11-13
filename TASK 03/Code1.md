import java.io.*;

import java.util.*;

public class SalesReportManual {

    public static void main(String[] args) {

        BufferedReader br = null;
        BufferedWriter bw = null;

        try {
            String[] files = {
                "store1.txt",
                "store2.txt",
                "store3.txt"
            };

            bw = new BufferedWriter(
                new FileWriter("summary.txt"));

            int totalSales = 0;

            for (String file : files) {

                System.out.println("Reading: " + file);

                br = new BufferedReader(
                    new FileReader(file));

                String line;

                while ((line = br.readLine()) != null) {
                    totalSales += Integer.parseInt(line);
                }

                br.close();
                br = null;
            }

            bw.write("Total Sales: " + totalSales);
            bw.newLine();

            System.out.println("Report generated successfully!");
            System.out.println("Total Sales: " + totalSales);

        }
        catch (IOException | NumberFormatException e) {
            System.out.println("Error: " + e.getMessage());
        }
        finally {

            try {
                if (br != null) {
                    br.close();
                }

                if (bw != null) {
                    bw.close();
                }

            }
            catch (IOException e) {
                System.out.println("Error closing files: "
                                   + e.getMessage());
            }
        }
    }
}

