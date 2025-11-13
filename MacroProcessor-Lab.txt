*********************file 1 named MacroProcessor.java ************************

import java.io.*;
import java.util.*;

public class MacroProcessor {
    String[] MDT = new String[50];  // stores macro definitions
    int MDTP = 0;                   // Macro Definition Table pointer
    List<MacroNameTable> MNT = new ArrayList<>();  // Macro Name Table list

    // ---------- PASS 1 ----------
    private void pass1() throws Exception {
        BufferedReader br = new BufferedReader(new FileReader("inputt.txt"));
        BufferedWriter intermediateWriter = new BufferedWriter(new FileWriter("intermediate.txt"));

        boolean insideMacro = false;
        String line;

        while ((line = br.readLine()) != null) {
            line = line.trim();
            if (line.isEmpty()) continue;

            if (line.equalsIgnoreCase("MACRO")) {
                insideMacro = true;
                line = br.readLine().trim();             // macro header line
                String[] parts = line.split("\\s+");
                String macroName = parts[0];
                int paramCount = parts.length - 1;
                MNT.add(new MacroNameTable(macroName, MDTP, paramCount));
                MDT[MDTP++] = line;                      // store header
            } else if (line.equalsIgnoreCase("MEND")) {
                MDT[MDTP++] = line;
                insideMacro = false;
            } else if (insideMacro) {
                MDT[MDTP++] = line;                      // macro body
            } else {
                intermediateWriter.write(line);          // normal code
                intermediateWriter.newLine();
            }
        }

        br.close();
        intermediateWriter.close();

        System.out.println("--- Macro Name Table (MNT) ---");
        for (MacroNameTable entry : MNT)
            System.out.println("Name: " + entry.name + ", Index: " + entry.index + ", Params: " + entry.parameters);

        System.out.println("\n--- Macro Definition Table (MDT) ---");
        for (int i = 0; i < MDTP; i++)
            System.out.println(i + ": " + MDT[i]);
    }

    // ---------- PASS 2 ----------
    private void pass2() throws Exception {
        BufferedReader br = new BufferedReader(new FileReader("intermediate.txt"));
        BufferedWriter bw = new BufferedWriter(new FileWriter("output.txt"));

        String line;
        while ((line = br.readLine()) != null) {
            line = line.trim();
            if (line.isEmpty()) {
                bw.newLine();
                continue;
            }

            String[] parts = line.split("\\s+");
            boolean macroFound = false;

            for (MacroNameTable mntEntry : MNT) {
                if (parts[0].equalsIgnoreCase(mntEntry.name)) {
                    macroFound = true;
                    int mdtIndex = mntEntry.index + 1;
                    Map<String, String> paramMap = new HashMap<>();

                    for (int i = 1; i < parts.length; i++)
                        paramMap.put("&" + i, parts[i].replace(",", ""));

                    while (!MDT[mdtIndex].equalsIgnoreCase("MEND")) {
                        String expandedLine = MDT[mdtIndex];
                        for (Map.Entry<String, String> entry : paramMap.entrySet())
                            expandedLine = expandedLine.replace(entry.getKey(), entry.getValue());
                        bw.write(expandedLine);
                        bw.newLine();
                        mdtIndex++;
                    }
                    break;
                }
            }

            if (!macroFound) {
                bw.write(line);
                bw.newLine();
            }
        }

        br.close();
        bw.close();
    }

    // ---------- MAIN ----------
    public static void main(String[] args) throws Exception {
        MacroProcessor mp = new MacroProcessor();
        System.out.println("Starting Pass 1...");
        mp.pass1();
        System.out.println("\nPass 1 completed.\n");
        System.out.println("Starting Pass 2...");
        mp.pass2();
        System.out.println("Pass 2 completed. Output generated in output.txt");
    }
}

******************************file 2 named MacroNameTable********************

public class MacroNameTable {
    int index;
    int parameters;
    String name;

    public MacroNameTable(String name, int index, int parameters) {
        this.name = name;
        this.index = index;
        this.parameters = parameters;
    }
}

*****************************file 3 namedd outputt.txt **********************

START
MACRO
ADD_TWO &1 &2
MOV A, &1
ADD A, &2
MEND
MACRO
SUBTRACT_TWO &1 &2
MOV B, &1
SUB B, &2
MEND
LDA 100
ADD_TWO 5, 10
STA 200
SUBTRACT_TWO 20, 5
HLT


To run write in terminal ***********************************************

terminal
javac MacroNameTable.java MacroProcessor.java

output is 2 files 

MacroProcessor.class
MacroNameTable.class

Terminal 
java MacroProcessor

output 

Starting Pass 1...
--- Macro Name Table (MNT) ---
Name: ADD_TWO, Index: 0, Params: 2
Name: SUBTRACT_TWO, Index: 4, Params: 2

--- Macro Definition Table (MDT) ---
0: ADD_TWO &1 &2
1: MOV A, &1
2: ADD A, &2
3: MEND
4: SUBTRACT_TWO &1 &2
5: MOV B, &1
6: SUB B, &2
7: MEND

Pass 1 completed.

Starting Pass 2...
Pass 2 completed. Output generated in output.txt


As output we get files created 

intermediate.txt
output.txt
