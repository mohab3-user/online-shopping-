import java.util.ArrayList;
import java.util.Scanner;

public class Main {
    private static final ArrayList<Product> products = new ArrayList<>();
    private static final ArrayList<Account> accounts = new ArrayList<>();
    private static final Scanner scanner = new Scanner(System.in);
    private static final String ADMIN_PASSWORD = "admin123";

    public static void main(String[] args) {
        initializeProducts();

        System.out.println("***** Welcome to our online store *****");
        while (true) {
            System.out.println("\nMain menu:");
            System.out.println("1. Login");
            System.out.println("2. Create a new account");
            System.out.println("3. Exit");
            System.out.print("Your choice: ");

            String choice = scanner.nextLine().trim();
            if (choice.isEmpty()) {
                continue;
            }
            int option;
            try {
                option = Integer.parseInt(choice);
            } catch (NumberFormatException e) {
                System.out.println("Please enter a number.");
                continue;
            }

            switch (option) {
                case 1:
                    Account user = login();
                    if (user != null) {
                        userMenu(user);
                    }
                    break;
                case 2:
                    register();
                    break;
                case 3:
                    System.out.println("Thank you for using our online shopping. Goodbye!");
                    return;
                default:
                    System.out.println("Incorrect choice, please try again.");
            }
        }
    }

    private static void initializeProducts() {
        products.add(new Product("Laptop", 15000.0, 5));
        products.add(new Product("Smartphone", 8000.0, 10));
        products.add(new Product("Headphones", 1500.5, 7));
        products.add(new Product("Camera", 12000.0, 4));
    }

    private static void register() {
        System.out.println("\n*** Create a new account ***");
        System.out.print("Username: ");
        String username = scanner.nextLine().trim();
        if (username.isEmpty()) {
            System.out.println("Username cannot be empty.");
            return;
        }
        for (Account acc : accounts) {
            if (acc.getUsername().equals(username)) {
                System.out.println("The username is already taken. Please choose another one.");
                return;
            }
        }
        System.out.print("Password: ");
        String password = scanner.nextLine().trim();
        System.out.print("Email: ");
        String email = scanner.nextLine().trim();

        if (password.isEmpty() || email.isEmpty()) {
            System.out.println("The password or email cannot be empty.");
            return;
        }

        Account newAccount = new Account(username, password, email);
        accounts.add(newAccount);
        System.out.println("The account has been created successfully! You can now log in.");
    }

    private static Account login() {
        System.out.println("\n*** Login ***");
        System.out.print("Username: ");
        String username = scanner.nextLine().trim();
        System.out.print("Password: ");
        String password = scanner.nextLine().trim();

        for (Account acc : accounts) {
            if (acc.getUsername().equals(username) && acc.getPassword().equals(password)) {
                System.out.println("Logged in successfully. Welcome, " + username + "!");
                return acc;
            }
        }
        System.out.println("Login failed: The username or password is incorrect.");
        return null;
    }

    private static void userMenu(Account user) {
        ArrayList<CartItem> cart = new ArrayList<>();
        while (true) {
            System.out.println("\nUser Menu:");
            System.out.println("1. Show the product list");
            System.out.println("2. Add a product to the cart");
            System.out.println("3. Show cart contents");
            System.out.println("4. Complete the purchase (payment)");
            System.out.println("5. Show purchase history");
            System.out.println("6. Add a new product (admin only)");
            System.out.println("7. Delete a product (admin only)");
            System.out.println("8. Log out");
            System.out.print("Choose an option: ");

            String choice = scanner.nextLine().trim();
            if (choice.isEmpty()) {
                continue;
            }
            int option;
            try {
                option = Integer.parseInt(choice);
            } catch (NumberFormatException e) {
                System.out.println("Please enter a valid number.");
                continue;
            }
            switch (option) {
                case 1:
                    displayProducts();
                    break;
                case 2:
                    addProductToCart(cart);
                    break;
                case 3:
                    displayCart(cart);
                    break;
                case 4:
                    checkout(cart, user);
                    break;
                case 5:
                    user.displayPurchaseHistory();
                    break;
                case 6:
                    addNewProduct();
                    break;
                case 7:
                    removeProduct();
                    break;
                case 8:
                    System.out.println("Logged out of account " + user.getUsername() + ".");
                    return;
                default:
                    System.out.println("Invalid option. Please try again.");
            }
        }
    }

    private static void displayProducts() {
        System.out.println("\n*** List of available products ***");
        if (products.isEmpty()) {
            System.out.println("There are currently no products available.");
            return;
        }
        System.out.printf("%-5s %-20s %10s %10s%n", "Number", "Product Name", "Price", "Stock");
        System.out.println("-------------------------------------------------------");
        for (int i = 0; i < products.size(); i++) {
            Product p = products.get(i);
            System.out.printf("%-5d %-20s %10.2f %10d%n", (i+1), p.getName(), p.getPrice(), p.getStock());
        }
    }

    private static void addProductToCart(ArrayList<CartItem> cart) {
        if (products.isEmpty()) {
            System.out.println("There are no products to add. The cart is empty.");
            return;
        }
        displayProducts();
        System.out.print("Enter the product number to add to the cart: ");
        String input = scanner.nextLine().trim();
        int productIndex;
        try {
            productIndex = Integer.parseInt(input) - 1;
        } catch (NumberFormatException e) {
            System.out.println("Invalid input.");
            return;
        }
        if (productIndex < 0 || productIndex >= products.size()) {
            System.out.println("The product number is invalid.");
            return;
        }
        Product selectedProduct = products.get(productIndex);
        System.out.print("Enter the required quantity: ");
        String qtyInput = scanner.nextLine().trim();
        int qty;
        try {
            qty = Integer.parseInt(qtyInput);
        } catch (NumberFormatException e) {
            System.out.println("Invalid input for quantity.");
            return;
        }
        if (qty <= 0) {
            System.out.println("The quantity must be at least 1.");
            return;
        }
        if (qty > selectedProduct.getStock()) {
            System.out.println("The required quantity is not available. Current stock: " + selectedProduct.getStock());
            return;
        }
        for (CartItem item : cart) {
            if (item.getProduct().equals(selectedProduct)) {
                int newQty = item.getQuantity() + qty;
                if (newQty > selectedProduct.getStock()) {
                    System.out.println("The total quantity required exceeds the available stock.");
                    return;
                }
                item.setQuantity(newQty);
                System.out.println("The cart has been updated: " + item.getProduct().getName() + " - Quantity: " + item.getQuantity());
                return;
            }
        }
        CartItem newItem = new CartItem(selectedProduct, qty);
        cart.add(newItem);
        System.out.println("Added " + selectedProduct.getName() + " (Quantity: " + qty + ") to the cart.");
    }

    private static void displayCart(ArrayList<CartItem> cart) {
        System.out.println("\n*** Contents of the cart ***");
        if (cart.isEmpty()) {
            System.out.println("The cart is empty.");
        } else {
            double total = 0.0;
            System.out.printf("%-5s %-20s %10s %10s%n", "Number", "Product Name", "Price", "Quantity");
            System.out.println("-------------------------------------------------------");
            for (int i = 0; i < cart.size(); i++) {
                CartItem item = cart.get(i);
                Product p = item.getProduct();
                System.out.printf("%-5d %-20s %10.2f %10d%n", (i+1), p.getName(), p.getPrice(), item.getQuantity());
                total += p.getPrice() * item.getQuantity();
            }
            System.out.println("-------------------------------------------------------");
            System.out.printf("Total Price: %.2f%n", total);
        }
    }

    private static void checkout(ArrayList<CartItem> cart, Account user) {
        if (cart.isEmpty()) {
            System.out.println("The cart is empty, there is nothing to buy.");
            return;
        }
        double total = 0.0;
        for (CartItem item : cart) {
            total += item.getProduct().getPrice() * item.getQuantity();
        }
        System.out.printf("Total purchase amount: %.2f%n", total);
        System.out.println("Choose a payment method: 1. Visa | 2. Cash");
        System.out.print("Payment method: ");
        String methodInput = scanner.nextLine().trim();
        int method;
        try {
            method = Integer.parseInt(methodInput);
        } catch (NumberFormatException e) {
            System.out.println("Invalid payment option.");
            return;
        }
        boolean paymentSuccess = false;
        switch (method) {
            case 1:
                System.out.print("Enter Visa card number: ");
                String card = scanner.nextLine().trim();
                System.out.print("Enter available balance: ");
                String balInput = scanner.nextLine().trim();
                double balance;
                try {
                    balance = Double.parseDouble(balInput);
                } catch (NumberFormatException e) {
                    System.out.println("Invalid input for balance.");
                    return;
                }
                if (balance >= total) {
                    double newBalance = balance - total;
                    System.out.println("Transaction successful. The amount has been deducted from your card.");
                    System.out.printf("Remaining balance on card: %.2f%n", newBalance);
                    paymentSuccess = true;
                } else {
                    System.out.println("Insufficient balance to complete the transaction.");
                }
                break;
            case 2:
                System.out.print("Enter the amount of cash paid: ");
                String cashInput = scanner.nextLine().trim();
                double cash;
                try {
                    cash = Double.parseDouble(cashInput);
                } catch (NumberFormatException e) {
                    System.out.println("Invalid input for amount.");
                    return;
                }
                if (cash >= total) {
                    double change = cash - total;
                    System.out.println("Cash payment successful.");
                    if (change > 0) {
                        System.out.printf("Change: %.2f%n", change);
                    }
                    paymentSuccess = true;
                } else {
                    System.out.println("Insufficient amount provided to complete the purchase.");
                }
                break;
            default:
                System.out.println("Invalid payment option.");
        }
        if (paymentSuccess) {
            for (CartItem item : cart) {
                Product product = item.getProduct();
                int qty = item.getQuantity();
                product.setStock(product.getStock() - qty);
                if (product.getStock() < 0) {
                    product.setStock(0);
                }
                String record = product.getName() + " (x" + qty + ")";
                user.addPurchase(record);
            }
            cart.clear();
            System.out.println("Purchase completed successfully! Thank you for shopping with us.");
        } else {
            System.out.println("Purchase not completed.");
        }
    }

    private static void addNewProduct() {
        System.out.print("Enter admin password to add a new product: ");
        String pass = scanner.nextLine().trim();
        if (!pass.equals(ADMIN_PASSWORD)) {
            System.out.println("Incorrect password. You cannot add products.");
            return;
        }
        System.out.print("New product name: ");
        String name = scanner.nextLine().trim();
        if (name.isEmpty()) {
            System.out.println("Product name cannot be empty.");
            return;
        }
        for (Product p : products) {
            if (p.getName().equalsIgnoreCase(name)) {
                System.out.println("A product with the same name already exists. Cannot add the product.");
                return;
            }
        }
        System.out.print("Product price: ");
        String priceInput = scanner.nextLine().trim();
        double price;
        try {
            price = Double.parseDouble(priceInput);
        } catch (NumberFormatException e) {
            System.out.println("Invalid price.");
            return;
        }
        System.out.print("Available stock quantity: ");
        String stockInput = scanner.nextLine().trim();
        int stock;
        try {
            stock = Integer.parseInt(stockInput);
        } catch (NumberFormatException e) {
            System.out.println("Invalid stock number.");
            return;
        }
        if (stock < 0) stock = 0;
        Product newProduct = new Product(name, price, stock);
        products.add(newProduct);
        System.out.println("New product added successfully: " + name);
    }

    private static void removeProduct() {
        System.out.print("Enter admin password to delete a product: ");
        String pass = scanner.nextLine().trim();
        if (!pass.equals(ADMIN_PASSWORD)) {
            System.out.println("Incorrect password. You cannot delete products.");
            return;
        }
        if (products.isEmpty()) {
            System.out.println("There are no products to delete.");
            return;
        }
        displayProducts();
        System.out.print("Enter the number of the product to delete: ");
        String input = scanner.nextLine().trim();
        int index;
        try {
            index = Integer.parseInt(input) - 1;
        } catch (NumberFormatException e) {
            System.out.println("Invalid input.");
            return;
        }
        if (index < 0 || index >= products.size()) {
            System.out.println("Invalid product number.");
            return;
        }
        Product removed = products.remove(index);
        System.out.println("Deleted product: " + removed.getName());
    }
}

class Product {
    private final String name;
    private final double price;
    private int stock;

    public Product(String name, double price, int stock) {
        this.name = name;
        this.price = price;
        this.stock = stock;
    }

    public String getName() {
        return name;
    }
    public double getPrice() {
        return price;
    }
    public int getStock() {
        return stock;
    }
    public void setStock(int stock) {
        this.stock = stock;
    }

    @Override
    public boolean equals(Object obj) {
        if (obj instanceof Product other) {
            return this.name.equals(other.name);
        }
        return false;
    }

    @Override
    public int hashCode() {
        return name.hashCode();
    }
}

class Account {
    private final String username;
    private final String password;
    private final String email;
    private final ArrayList<String> purchaseHistory;

    public Account(String username, String password, String email) {
        this.username = username;
        this.password = password;
        this.email = email;
        this.purchaseHistory = new ArrayList<>();
    }

    public String getUsername() {
        return username;
    }
    public String getPassword() {
        return password;
    }
    public String getEmail() {
        return email;
    }

    public void addPurchase(String record) {
        purchaseHistory.add(record);
    }

    public void displayPurchaseHistory() {
        System.out.println("\n*** Purchase history for " + username + " ***");
        if (purchaseHistory.isEmpty()) {
            System.out.println("No previous purchases.");
        } else {
            for (int i = 0; i < purchaseHistory.size(); i++) {
                System.out.println((i+1) + ". " + purchaseHistory.get(i));
            }
        }
    }
}

class CartItem {
    private final Product product;
    private int quantity;

    public CartItem(Product product, int quantity) {
        this.product = product;
        this.quantity = quantity;
    }

    public Product getProduct() {
        return product;
    }
    public int getQuantity() {
        return quantity;
    }
    public void setQuantity(int quantity) {
        this.quantity = quantity;
    }
}
