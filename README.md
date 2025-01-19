import javax.swing.*;
import java.awt.*;
import java.awt.event.*;
import java.sql.*;
import java.util.Vector;
import javax.swing.table.DefaultTableModel;
import java.sql.DriverManager;
import java.sql.Connection;
import java.sql.SQLException;


public class ExpenseTrackerApp extends JFrame implements ActionListener {

    private JLabel lblDate, lblDescription, lblCategory, lblAmount;
    private JTextField txtDate, txtDescription, txtCategory, txtAmount;
    private JButton btnSave, btnLoad;
    private JTable expenseTable;
    private JScrollPane scrollPane;
    public DefaultTableModel tableModel;

    private Connection conn;
    private PreparedStatement pstmt;
    private ResultSet rs;

    public ExpenseTrackerApp() {
        setTitle("Expense Tracker");
        setSize(800, 400);
        setDefaultCloseOperation(EXIT_ON_CLOSE);

        // Initialize components
        lblDate = new JLabel("Date:");
        lblDescription = new JLabel("Description:");
        lblCategory = new JLabel("Category:");
        lblAmount = new JLabel("Amount:");

        txtDate = new JTextField(10);
        txtDescription = new JTextField(20);
        txtCategory = new JTextField(10);
        txtAmount = new JTextField(10);

        btnSave = new JButton("Save");
        btnLoad = new JButton("Load Expenses");

        btnSave.addActionListener(this);
        btnLoad.addActionListener(this);

        // Table setup
        tableModel = new DefaultTableModel();
        expenseTable = new JTable(tableModel);
        scrollPane = new JScrollPane(expenseTable);
        scrollPane.setPreferredSize(new Dimension(700, 200));

        JPanel inputPanel = new JPanel(new GridLayout(2, 4));
        inputPanel.add(lblDate);
        inputPanel.add(txtDate);
        inputPanel.add(lblDescription);
        inputPanel.add(txtDescription);
        inputPanel.add(lblCategory);
        inputPanel.add(txtCategory);
        inputPanel.add(lblAmount);
        inputPanel.add(txtAmount);

        JPanel buttonPanel = new JPanel();
        buttonPanel.add(btnSave);
        buttonPanel.add(btnLoad);

        JPanel mainPanel = new JPanel(new BorderLayout());
        mainPanel.add(inputPanel, BorderLayout.NORTH);
        mainPanel.add(scrollPane, BorderLayout.CENTER);
        mainPanel.add(buttonPanel, BorderLayout.SOUTH);

        add(mainPanel);
        setVisible(true);

        // Connect to database
        try {
            Class.forName("com.mysql.cj.jdbc.Driver");
            conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/expense_tracker", "root", "Root");
        } catch (ClassNotFoundException | SQLException e) {
            e.printStackTrace();
        }
    }

    public void actionPerformed(ActionEvent e) {
        if (e.getSource() == btnSave) {
            saveExpense();
        } else if (e.getSource() == btnLoad) {
            loadExpenses();
        }
    }

    private void saveExpense() {
        String date = txtDate.getText();
        String description = txtDescription.getText();
        String category = txtCategory.getText();
        double amount = Double.parseDouble(txtAmount.getText());

        try {
            String sql = "INSERT INTO expenses (date, description, category, amount) VALUES (?, ?, ?, ?)";
            pstmt = conn.prepareStatement(sql);
            pstmt.setString(1, date);
            pstmt.setString(2, description);
            pstmt.setString(3, category);
            pstmt.setDouble(4, amount);
            pstmt.executeUpdate();
            JOptionPane.showMessageDialog(this, "Expense saved successfully!");
            clearFields();
        } catch (SQLException ex) {
            ex.printStackTrace();
            JOptionPane.showMessageDialog(this, "Error saving expense.");
        }
    }

    private void loadExpenses() {
        try {
            tableModel.setColumnCount(0);
            tableModel.setRowCount(0);

            pstmt = conn.prepareStatement("SELECT * FROM expenses");
            rs = pstmt.executeQuery();

            ResultSetMetaData rsmd = rs.getMetaData();
            int columnCount = rsmd.getColumnCount();

            // Add columns to table model
            for (int i = 1; i <= columnCount; i++) {
                tableModel.addColumn(rsmd.getColumnName(i));
            }

            // Add rows to table model
            while (rs.next()) {
                Vector<String> row = new Vector<>();
                for (int i = 1; i <= columnCount; i++) {
                    row.add(rs.getString(i));
                }
                tableModel.addRow(row);
            }
        } catch (SQLException ex) {
            ex.printStackTrace();
            JOptionPane.showMessageDialog(this, "Error loading expenses.");
        }
    }

    private void clearFields() {
        txtDate.setText("");
        txtDescription.setText("");
        txtCategory.setText("");
        txtAmount.setText("");
    }
public static void main(String[] args) {
        new ExpenseTrackerApp();
}}
