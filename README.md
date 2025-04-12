import java.awt.*;
import java.awt.event.*;

public class MatrixChainAWT extends Frame {
    private int[] dimensions;
    private int[][] dp, bracket;
    private int numMatrices;
    private int step, iIndex;

    private Label inputLabel, finalAnswerLabel;
    private TextField inputField;
    private Button nextButton;
    private TextArea dpTable, bracketTable, calculationArea;
    private Panel inputPanel, outputPanel, tablePanel;

    public MatrixChainAWT() {
        setTitle("Matrix Chain Multiplication Visualization");
        setExtendedState(MAXIMIZED_BOTH);
        setLayout(new BorderLayout());
        setBackground(new Color(245, 245, 255));

        // Input Panel
        inputPanel = new Panel();
        inputPanel.setLayout(new FlowLayout(FlowLayout.LEFT, 20, 20));
        inputLabel = new Label("Enter the number of matrices:");
        inputLabel.setFont(new Font("SansSerif", Font.BOLD, 18));
        inputField = new TextField(10);
        inputField.setFont(new Font("SansSerif", Font.PLAIN, 18));
        Button submitButton = new Button("Submit");
        submitButton.setFont(new Font("SansSerif", Font.BOLD, 18));
        inputPanel.add(inputLabel);
        inputPanel.add(inputField);
        inputPanel.add(submitButton);

        // Output Panel
        outputPanel = new Panel();
        outputPanel.setLayout(new BorderLayout());

        // Final Answer Label
        finalAnswerLabel = new Label("");
        finalAnswerLabel.setFont(new Font("SansSerif", Font.BOLD, 32));
        finalAnswerLabel.setAlignment(Label.CENTER);
        finalAnswerLabel.setForeground(new Color(0, 102, 153));
        outputPanel.add(finalAnswerLabel, BorderLayout.NORTH);

        // Calculation Area
        calculationArea = new TextArea("Calculations will be shown here");
        calculationArea.setFont(new Font("Monospaced", Font.PLAIN, 16));
        calculationArea.setEditable(false);
        outputPanel.add(calculationArea, BorderLayout.CENTER);

        // Tables Panel
        Panel bottomPanel = new Panel(new BorderLayout());

        tablePanel = new Panel();
        tablePanel.setLayout(new GridLayout(1, 2, 30, 30));

        dpTable = new TextArea(15, 40);
        dpTable.setFont(new Font("Monospaced", Font.PLAIN, 16));
        dpTable.setEditable(false);

        bracketTable = new TextArea(15, 40);
        bracketTable.setFont(new Font("Monospaced", Font.PLAIN, 16));
        bracketTable.setEditable(false);

        tablePanel.add(dpTable);
        tablePanel.add(bracketTable);
        bottomPanel.add(tablePanel, BorderLayout.CENTER);

        // Next Button
        nextButton = new Button("Next Step");
        nextButton.setFont(new Font("SansSerif", Font.BOLD, 18));
        nextButton.setEnabled(false);
        Panel nextPanel = new Panel(new FlowLayout(FlowLayout.CENTER, 20, 20));
        nextPanel.add(nextButton);

        bottomPanel.add(nextPanel, BorderLayout.SOUTH);
        outputPanel.add(bottomPanel, BorderLayout.SOUTH);

        add(inputPanel, BorderLayout.NORTH);
        add(outputPanel, BorderLayout.CENTER);

        submitButton.addActionListener(e -> {
            try {
                numMatrices = Integer.parseInt(inputField.getText());
                if (numMatrices < 2 || numMatrices > 9) {
                    calculationArea.setText("Enter number of matrices between 2 and 9.");
                    return;
                }
                dimensions = new int[numMatrices + 1];
                inputMatrices();
            } catch (NumberFormatException ex) {
                calculationArea.setText("Invalid input. Enter an integer.");
            }
        });

        nextButton.addActionListener(e -> processNextStep());

        addWindowListener(new WindowAdapter() {
            public void windowClosing(WindowEvent we) {
                dispose();
            }
        });

        setVisible(true);
    }

    private void inputMatrices() {
        inputPanel.removeAll();
        inputPanel.setLayout(new BorderLayout(10, 10));

        inputLabel.setText("Enter dimensions one by one:");
        inputLabel.setFont(new Font("SansSerif", Font.BOLD, 18));
        inputPanel.add(inputLabel, BorderLayout.NORTH);

        Panel dimensionPanel = new Panel();
        dimensionPanel.setLayout(new FlowLayout(FlowLayout.LEFT, 15, 10));

        ScrollPane scrollPane = new ScrollPane();
        scrollPane.setPreferredSize(new Dimension(1400, 100));
        scrollPane.add(dimensionPanel);

        TextField[] fields = new TextField[numMatrices + 1];

        for (int i = 0; i <= numMatrices; i++) {
            Panel pairPanel = new Panel();
            pairPanel.setLayout(new BorderLayout(5, 0));

            Label lbl = new Label("Dim " + (i + 1) + ": ");
            lbl.setFont(new Font("SansSerif", Font.PLAIN, 16));

            TextField tf = new TextField(4);
            tf.setFont(new Font("SansSerif", Font.PLAIN, 16));
            fields[i] = tf;

            pairPanel.add(lbl, BorderLayout.WEST);
            pairPanel.add(tf, BorderLayout.CENTER);
            dimensionPanel.add(pairPanel);
        }

        Button confirmButton = new Button("Confirm");
        confirmButton.setFont(new Font("SansSerif", Font.BOLD, 16));
        dimensionPanel.add(confirmButton);

        Button backButton = new Button("Back");
        backButton.setFont(new Font("SansSerif", Font.BOLD, 16));
        dimensionPanel.add(backButton);

        inputPanel.add(scrollPane, BorderLayout.CENTER);
        inputPanel.revalidate();
        inputPanel.repaint();

        confirmButton.addActionListener(e -> {
            try {
                for (int i = 0; i <= numMatrices; i++) {
                    dimensions[i] = Integer.parseInt(fields[i].getText());
                }
                initializeComputation();
            } catch (NumberFormatException ex) {
                calculationArea.setText("Invalid input. Enter integers only.");
            }
        });

        backButton.addActionListener(e -> {
            removeAll();
            new MatrixChainAWT();
            dispose();
        });
    }

    private void initializeComputation() {
        StringBuilder matrixInfo = new StringBuilder("Matrices: ");
        for (int i = 0; i < numMatrices; i++) {
            matrixInfo.append("A").append(i + 1).append(" = ")
                      .append(dimensions[i]).append("x")
                      .append(dimensions[i + 1]).append(", ");
        }
        calculationArea.setText(matrixInfo.toString());

        dp = new int[numMatrices][numMatrices];
        bracket = new int[numMatrices][numMatrices];
        for (int i = 0; i < numMatrices; i++) dp[i][i] = 0;

        step = 1;
        iIndex = 0;
        nextButton.setEnabled(true);
        updateTables();
    }

    private void processNextStep() {
        if (step >= numMatrices) {
            printFinalAnswer();
            nextButton.setEnabled(false);
            return;
        }

        if (iIndex >= numMatrices - step) {
            step++;
            iIndex = 0;
        }

        if (step >= numMatrices) {
            printFinalAnswer();
            nextButton.setEnabled(false);
            return;
        }

        int i = iIndex;
        int j = i + step;
        if (j >= numMatrices) return;

        StringBuilder calculationSteps = new StringBuilder();
        calculationSteps.append("Calculating M[").append(i + 1).append(",").append(j + 1).append("]\n");

        int minCost = Integer.MAX_VALUE;
        int minK = -1;
        for (int k = i; k < j; k++) {
            int cost = dp[i][k] + dp[k + 1][j] + dimensions[i] * dimensions[k + 1] * dimensions[j + 1];
            calculationSteps.append("  - Cost for split at k=").append(k + 1)
                .append(" ➜ M[").append(i + 1).append(",").append(k + 1)
                .append("] + M[").append(k + 2).append(",").append(j + 1)
                .append("] + ").append(dimensions[i]).append("x")
                .append(dimensions[k + 1]).append("x")
                .append(dimensions[j + 1]).append(" = ")
                .append(cost).append("\n");

            if (cost < minCost) {
                minCost = cost;
                minK = k;
            }
        }

        dp[i][j] = minCost;
        bracket[i][j] = minK;
        calculationSteps.append("  ➤ Minimum cost found at k=").append(minK + 1).append(" = ").append(minCost).append("\n");

        calculationArea.setText(calculationSteps.toString());
        updateTables();
        iIndex++;
    }

    private void updateTables() {
        StringBuilder dpContent = new StringBuilder("DP Table:\n");
        StringBuilder bracketContent = new StringBuilder("Bracket Table:\n");

        for (int i = 0; i < numMatrices; i++) {
            for (int j = 0; j < numMatrices; j++) {
                if (j < i) {
                    dpContent.append(" -\t");
                    bracketContent.append(" -\t");
                } else {
                    dpContent.append(dp[i][j] == 0 ? "0\t" : dp[i][j] + "\t");
                    bracketContent.append((i == j || dp[i][j] == 0) ? " -\t" : (bracket[i][j] + 1) + "\t");
                }
            }
            dpContent.append("\n");
            bracketContent.append("\n");
        }

        dpTable.setText(dpContent.toString());
        bracketTable.setText(bracketContent.toString());
    }

    private void printFinalAnswer() {
        String finalParenthesis = getOptimalParenthesization(0, numMatrices - 1);
        int minMultiplications = dp[0][numMatrices - 1];
        finalAnswerLabel.setText("Final Answer: " + finalParenthesis +
            "      |      Min Multiplications: " + minMultiplications);
    }

    private String getOptimalParenthesization(int i, int j) {
        if (i == j) return "A" + (i + 1);
        int k = bracket[i][j];
        return "(" + getOptimalParenthesization(i, k) + " " + getOptimalParenthesization(k + 1, j) + ")";
    }

    public static void main(String[] args) {
        new MatrixChainAWT();
    }
}
