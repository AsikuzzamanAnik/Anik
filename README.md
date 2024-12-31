import java.io.*;
import java.util.*;

public class PharmacyManagementSystem {

    private static final String ADMIN_FILE = "admins.txt";
    private static final String CUSTOMER_FILE = "customers.txt";
    private static final String MEDICINE_FILE = "medicines.txt";
    private static final double TAX_RATE = 0.08;
    private static final Scanner SCANNER = new Scanner(System.in);

    private final List<Medicine> medicines = new ArrayList<>();
    private final List<CartItem> cart = new ArrayList<>();

    static class Medicine {
        String name;
        double price;
        int quantity;

        Medicine(String name, double price, int quantity) {
            this.name = name;
            this.price = price;
            this.quantity = quantity;
        }

        @Override
        public String toString() {
            return name + " - $" + String.format("%.2f", price) + " (Stock: " + quantity + ")";
        }
    }

    static class CartItem {
        String name;
        double price;
        int quantity;

        CartItem(String name, double price, int quantity) {
            this.name = name;
            this.price = price;
            this.quantity = quantity;
        }

        @Override
        public String toString() {
            return name + " - $" + String.format("%.2f", price) + " x" + quantity;
        }
    }

    public static void main(String[] args) {
        PharmacyManagementSystem system = new PharmacyManagementSystem();
        system.run();
    }

    private void run() {
        ensureFileExists(ADMIN_FILE);
        ensureFileExists(CUSTOMER_FILE);
        ensureFileExists(MEDICINE_FILE);

        loadMedicinesFromFile();

        while (true) {
            System.out.println("\nWelcome to the Pharmacy Management System!");
            System.out.println("1. Admin Panel");
            System.out.println("2. Customer Panel");
            System.out.println("0. Exit");
            System.out.print("Enter your choice: ");
            
            int choice = getIntInput();

            switch (choice) {
                case 1 -> adminPanel();
                case 2 -> customerPanel();
                case 0 -> {
                    System.out.println("Exiting the system. Goodbye!");
                    saveMedicinesToFile();
                    return;
                }
                default -> System.out.println("Invalid choice. Please try again.");
            }
        }
    }

    private void ensureFileExists(String fileName) {
        try {
            File file = new File(fileName);
            if (file.createNewFile()) {
                System.out.println("Initialized data file: " + fileName);
            }
        } catch (IOException e) {
            System.err.println("Error initializing file: " + fileName);
        }
    }

    private void adminPanel() {
        System.out.println("\nAdmin Panel");
        System.out.println("1. Register");
        System.out.println("2. Login");
        System.out.print("Enter your choice: ");

        int choice = getIntInput();

        if (choice == 1) {
            registerUser(ADMIN_FILE);
        } else if (choice == 2) {
            if (loginUser(ADMIN_FILE)) {
                System.out.println("Login successful!");
                manageMedicines();
            } else {
                System.out.println("Invalid username or password.");
            }
        } else {
            System.out.println("Invalid choice.");
        }
    }

    private void customerPanel() {
        System.out.println("\nCustomer Panel");
        System.out.println("1. Register");
        System.out.println("2. Login");
        System.out.print("Enter your choice: ");

        int choice = getIntInput();

        if (choice == 1) {
            registerUser(CUSTOMER_FILE);
        } else if (choice == 2) {
            if (loginUser(CUSTOMER_FILE)) {
                System.out.println("Login successful!");
                shopMedicines();
            } else {
                System.out.println("Invalid username or password.");
            }
        } else {
            System.out.println("Invalid choice.");
        }
    }

    private void registerUser(String fileName) {
        System.out.print("Enter username: ");
        String username = SCANNER.nextLine().trim();
        System.out.print("Enter password: ");
        String password = SCANNER.nextLine().trim();

        try (BufferedWriter writer = new BufferedWriter(new FileWriter(fileName, true))) {
            writer.write(username + "," + password + "\n");
            System.out.println("Registration successful!");
        } catch (IOException e) {
            System.err.println("Error saving user information: " + e.getMessage());
        }
    }

    private boolean loginUser(String fileName) {
        System.out.print("Enter username: ");
        String username = SCANNER.nextLine().trim();
        System.out.print("Enter password: ");
        String password = SCANNER.nextLine().trim();

        try (BufferedReader reader = new BufferedReader(new FileReader(fileName))) {
            String line;
            while ((line = reader.readLine()) != null) {
                String[] credentials = line.split(",");
                if (credentials[0].equals(username) && credentials[1].equals(password)) {
                    return true;
                }
            }
        } catch (IOException e) {
            System.err.println("Error reading user file: " + e.getMessage());
        }
        return false;
    }

    private void loadMedicinesFromFile() {
        try (BufferedReader reader = new BufferedReader(new FileReader(MEDICINE_FILE))) {
            String line;
            while ((line = reader.readLine()) != null) {
                String[] details = line.split(",");
                medicines.add(new Medicine(details[0], Double.parseDouble(details[1]), Integer.parseInt(details[2])));
            }
        } catch (IOException e) {
            System.err.println("Error loading medicines: " + e.getMessage());
        }
    }

    private void saveMedicinesToFile() {
        try (BufferedWriter writer = new BufferedWriter(new FileWriter(MEDICINE_FILE))) {
            for (Medicine medicine : medicines) {
                writer.write(medicine.name + "," + medicine.price + "," + medicine.quantity + "\n");
            }
        } catch (IOException e) {
            System.err.println("Error saving medicines: " + e.getMessage());
        }
    }

    private void manageMedicines() {
        while (true) {
            System.out.println("\nMedicine Management Panel");
            System.out.println("1. Add Medicine");
            System.out.println("2. View Medicines");
            System.out.println("3. Remove Medicine");
            System.out.println("4. Update Medicine");
            System.out.println("5. Exit");
            System.out.print("Enter your choice: ");

            int choice = getIntInput();

            switch (choice) {
                case 1 -> addMedicine();
                case 2 -> viewMedicines();
                case 3 -> removeMedicine();
                case 4 -> updateMedicine();
                case 5 -> {
                    System.out.println("Exiting medicine management.");
                    return;
                }
                default -> System.out.println("Invalid choice.");
            }
        }
    }

    private void addMedicine() {
        System.out.print("Enter medicine name: ");
        String name = SCANNER.nextLine().trim();
        System.out.print("Enter medicine price: ");
        double price = getDoubleInput();
        System.out.print("Enter medicine quantity: ");
        int quantity = getIntInput();

        medicines.add(new Medicine(name, price, quantity));
        System.out.println("Medicine added successfully!");
    }

    private void viewMedicines() {
        System.out.println("\nAvailable Medicines:");
        if (medicines.isEmpty()) {
            System.out.println("No medicines available.");
        } else {
            for (Medicine medicine : medicines) {
                System.out.println(medicine);
            }
        }
    }

    private void removeMedicine() {
        System.out.print("Enter medicine name to remove: ");
        String name = SCANNER.nextLine().trim();
        boolean removed = medicines.removeIf(medicine -> medicine.name.equalsIgnoreCase(name));
        if (removed) {
            System.out.println("Medicine removed successfully.");
        } else {
            System.out.println("Medicine not found.");
        }
    }

    private void updateMedicine() {
        System.out.print("Enter medicine name to update: ");
        String name = SCANNER.nextLine().trim();
        for (Medicine medicine : medicines) {
            if (medicine.name.equalsIgnoreCase(name)) {
                System.out.print("Enter new price: ");
                medicine.price = getDoubleInput();
                System.out.print("Enter new quantity: ");
                medicine.quantity = getIntInput();
                System.out.println("Medicine updated successfully!");
                return;
            }
        }
        System.out.println("Medicine not found.");
    }

    private void shopMedicines() {
        while (true) {
            System.out.println("\nCustomer Shopping Panel");
            System.out.println("1. View Medicines");
            System.out.println("2. Add to Cart");
            System.out.println("3. View Cart");
            System.out.println("4. Checkout");
            System.out.println("5. Exit");
            System.out.print("Enter your choice: ");

            int choice = getIntInput();

            switch (choice) {
                case 1 -> viewMedicines();
                case 2 -> addToCart();
                case 3 -> viewCart();
                case 4 -> checkout();
                case 5 -> {
                    System.out.println("Exiting shopping panel.");
                    return;
                }
                default -> System.out.println("Invalid choice.");
            }
        }
    }

    private void addToCart() {
        System.out.print("Enter medicine name to add to cart: ");
        String name = SCANNER.nextLine().trim();
        System.out.print("Enter quantity: ");
        int quantity = getIntInput();

        for (Medicine medicine : medicines) {
            if (medicine.name.equalsIgnoreCase(name)) {
                if (medicine.quantity >= quantity) {
                    cart.add(new CartItem(medicine.name, medicine.price, quantity));
                    medicine.quantity -= quantity;
                    System.out.println("Added to cart successfully!");
                } else {
                    System.out.println("Not enough stock available.");
                }
                return;
            }
        }
        System.out.println("Medicine not found.");
    }

    private void viewCart() {
        System.out.println("\nYour Cart:");
        if (cart.isEmpty()) {
            System.out.println("Cart is empty.");
        } else {
            for (CartItem item : cart) {
                System.out.println(item);
            }
        }
    }

    private void checkout() {
        if (cart.isEmpty()) {
            System.out.println("Cart is empty. Add items before checkout.");
            return;
        }

        double total = 0;
        for (CartItem item : cart) {
            total += item.price * item.quantity;
        }

        double tax = total * TAX_RATE;
        double finalTotal = total + tax;

        System.out.printf("Total: $%.2f\n", total);
        System.out.printf("Tax (%.2f%%): $%.2f\n", TAX_RATE * 100, tax);
        System.out.printf("Final Total: $%.2f\n", finalTotal);

        cart.clear();
        System.out.println("Checkout complete. Thank you for shopping!");
    }

    private int getIntInput() {
        while (true) {
            try {
                return Integer.parseInt(SCANNER.nextLine().trim());
            } catch (NumberFormatException e) {
                System.out.print("Invalid input. Please enter a number: ");
            }
        }
    }

    private double getDoubleInput() {
        while (true) {
            try {
                return Double.parseDouble(SCANNER.nextLine().trim());
            } catch (NumberFormatException e) {
                System.out.print("Invalid input. Please enter a valid number: ");
            }
        }
    }
}
