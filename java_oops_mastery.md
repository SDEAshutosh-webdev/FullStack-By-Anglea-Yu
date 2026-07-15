# Java OOPs Mastery: 5 Key Programs

To prove you are a strong Java developer, you must demonstrate a deep understanding of Object-Oriented Programming (OOP). These 5 programs demonstrate core OOP pillars (Encapsulation, Inheritance, Polymorphism, Abstraction, and Composition) plus real-world design patterns.

---

## Program 1: Encapsulation & Validation (Secure Bank Account)
Encapsulation hides the internal state of an object and forces all interaction through public methods (getters/setters). This allows you to validate data and protect integrity.

### Implementation:
```java
import java.util.ArrayList;
import java.util.List;

public class BankAccount {
    // 1. Private fields prevent direct external modification (Encapsulation)
    private final String accountNumber;
    private double balance;
    private final double overdraftLimit;
    private final List<String> transactionHistory;

    // Constructor to initialize state safely
    public BankAccount(String accountNumber, double initialBalance, double overdraftLimit) {
        this.accountNumber = accountNumber;
        this.balance = Math.max(0, initialBalance); // Prevent negative initial balance
        this.overdraftLimit = Math.max(0, overdraftLimit);
        this.transactionHistory = new ArrayList<>();
        logTransaction("Account created with balance: $" + this.balance);
    }

    // 2. Read-only getters
    public String getAccountNumber() {
        return accountNumber;
    }

    public double getBalance() {
        return balance;
    }

    // 3. Methods to manipulate state with strict business logic/validation
    public void deposit(double amount) {
        if (amount <= 0) {
            throw new IllegalArgumentException("Deposit amount must be positive.");
        }
        this.balance += amount;
        logTransaction("Deposited: $" + amount + " | New Balance: $" + this.balance);
    }

    public void withdraw(double amount) {
        if (amount <= 0) {
            throw new IllegalArgumentException("Withdrawal amount must be positive.");
        }
        // Validate against overdraft limit
        if (this.balance - amount < -overdraftLimit) {
            throw new IllegalStateException("Transaction declined: Overdraft limit exceeded.");
        }
        this.balance -= amount;
        logTransaction("Withdrew: $" + amount + " | New Balance: $" + this.balance);
    }

    public List<String> getTransactionHistory() {
        // Return a copy to prevent external modification of the internal list
        return new ArrayList<>(transactionHistory);
    }

    private void logTransaction(String detail) {
        transactionHistory.add(detail);
    }
}
```

### Why it makes you a pro:
*   It protects internal data by making variables `private` and lists read-only or copy-on-return.
*   It uses business validation inside mutator methods (`deposit`/`withdraw`) rather than trusting the caller.

---

## Program 2: Inheritance & Polymorphism (Payment Processor)
*   **Inheritance (IS-A relationship)**: Reuses code by deriving specific classes from a base class.
*   **Polymorphism (Dynamic Binding)**: Allows a parent class reference to call overridden methods in subclasses at runtime.

### Implementation:
```java
// Base Class
abstract class PaymentMethod {
    protected String transactionId;

    public PaymentMethod(String transactionId) {
        this.transactionId = transactionId;
    }

    // Abstract method to be overridden by subclasses (Polymorphism)
    public abstract void processPayment(double amount);

    // Overloaded method (Compile-time Polymorphism)
    public void processPayment(double amount, String currency) {
        System.out.println("Converting currency to " + currency);
        processPayment(amount); // Calls the overridden method
    }
}

// Subclass 1
class CreditCardPayment extends PaymentMethod {
    private String cardNumber;

    public CreditCardPayment(String transactionId, String cardNumber) {
        super(transactionId);
        this.cardNumber = cardNumber;
    }

    @Override
    public void processPayment(double amount) {
        System.out.println("Processing credit card payment of $" + amount + 
                           " for Card " + cardNumber + " [TxId: " + transactionId + "]");
    }
}

// Subclass 2
class PayPalPayment extends PaymentMethod {
    private String email;

    public PayPalPayment(String transactionId, String email) {
        super(transactionId);
        this.email = email;
    }

    @Override
    public void processPayment(double amount) {
        System.out.println("Processing PayPal payment of $" + amount + 
                           " from " + email + " [TxId: " + transactionId + "]");
    }
}

// Client Code running Polymorphism
public class PaymentSystem {
    public static void main(String[] args) {
        // Parent class references pointing to subclass objects (Upcasting)
        PaymentMethod card = new CreditCardPayment("TXN100", "1234-5678-9876");
        PaymentMethod paypal = new PayPalPayment("TXN200", "user@example.com");

        // The JVM dynamically decides which method to run at runtime (Runtime Polymorphism)
        process(card, 150.0);
        process(paypal, 80.5);
    }

    public static void process(PaymentMethod pm, double amount) {
        pm.processPayment(amount);
    }
}
```

### Why it makes you a pro:
*   The `process()` method does not care about the type of payment (CreditCard or PayPal); it operates purely on the `PaymentMethod` abstraction, making the codebase highly extendable.

---

## Program 3: Abstraction & Interfaces (Decoupled Notification Engine)
Abstraction hides complex implementation details. In Java, **Interfaces** define a contract of *what* a class should do, not *how* it does it, decoupling components.

### Implementation:
```java
// Contract Interface
interface MessageSender {
    void sendMessage(String recipient, String content);
}

// Secondary Interface for logging capabilities
interface Loggable {
    void logStatus(String status);
}

// Implementation 1
class EmailSender implements MessageSender, Loggable {
    @Override
    public void sendMessage(String recipient, String content) {
        System.out.println("Sending Email to " + recipient + ": " + content);
        logStatus("Email Sent successfully.");
    }

    @Override
    public void logStatus(String status) {
        System.out.println("[EMAIL LOG]: " + status);
    }
}

// Implementation 2
class SmsSender implements MessageSender, Loggable {
    @Override
    public void sendMessage(String recipient, String content) {
        System.out.println("Sending SMS to " + recipient + ": " + content);
        logStatus("SMS Sent successfully.");
    }

    @Override
    public void logStatus(String status) {
        System.out.println("[SMS LOG]: " + status);
    }
}

// Decoupled Notification Service
class NotificationService {
    private final MessageSender sender; // Dependency Injection (Coding to an Interface)

    public NotificationService(MessageSender sender) {
        this.sender = sender;
    }

    public void notifyUser(String user, String message) {
        sender.sendMessage(user, message);
    }
}
```

### Why it makes you a pro:
*   This uses **Dependency Injection** by declaring variables of interface types (`MessageSender`) rather than concrete classes. This allows you to swap `EmailSender` with `SmsSender` without modifying `NotificationService` code.

---

## Program 4: Composition vs. Inheritance (The HAS-A Rule)
*   **Inheritance**: `IS-A` relationship (e.g., A Dog IS-A Animal).
*   **Composition**: `HAS-A` relationship (e.g., A Car HAS-A Engine). 
*   **Rule of Thumb**: Favor Composition over Inheritance to keep classes loosely coupled.

### Implementation:
```java
// Component class 1
class Engine {
    private final String type;

    public Engine(String type) {
        this.type = type;
    }

    public void start() {
        System.out.println(type + " Engine starting... Vroom!");
    }
}

// Component class 2
class GPS {
    public void navigateTo(String destination) {
        System.out.println("Routing coordinates to: " + destination);
    }
}

// Composite Class (Car has-a Engine and has-a GPS)
class Car {
    private final String model;
    private final Engine engine; // Composition
    private final GPS gps;       // Composition

    // Injecting components via constructor
    public Car(String model, Engine engine, GPS gps) {
        this.model = model;
        this.engine = engine;
        this.gps = gps;
    }

    public void startJourney(String destination) {
        System.out.println("Starting journey in " + model);
        engine.start();
        gps.navigateTo(destination);
    }
}
```

### Why it makes you a pro:
*   Instead of creating nested, rigid inheritance structures (e.g., `GpsNavigationSystemCar` extends `EngineCar`), you compose complex systems by combining small, reusable classes.

---

## Program 5: Factory Design Pattern (Dynamic Object Creation)
A design pattern combining polymorphism and abstraction. It delegates object creation logic to a subclass/factory class, preventing object instantiation details from leaking into the client code.

### Implementation:
```java
// 1. Common Product Interface
interface Document {
    void open();
    void save();
}

// 2. Concrete Products
class PdfDocument implements Document {
    @Override
    public void open() { System.out.println("Opening PDF Document..."); }
    @Override
    public void save() { System.out.println("Saving PDF Document..."); }
}

class WordDocument implements Document {
    @Override
    public void open() { System.out.println("Opening Word Document..."); }
    @Override
    public void save() { System.out.println("Saving Word Document..."); }
}

// 3. Factory Creator Class
class DocumentFactory {
    public static Document createDocument(String type) {
        if (type == null) {
            return null;
        }
        if (type.equalsIgnoreCase("PDF")) {
            return new PdfDocument();
        } else if (type.equalsIgnoreCase("WORD")) {
            return new WordDocument();
        }
        throw new IllegalArgumentException("Unknown document type: " + type);
    }
}

// 4. Client Code
public class Client {
    public static void main(String[] args) {
        // Instantiate products dynamically using the Factory
        Document doc1 = DocumentFactory.createDocument("PDF");
        doc1.open(); // Output: Opening PDF Document...

        Document doc2 = DocumentFactory.createDocument("WORD");
        doc2.save(); // Output: Saving Word Document...
    }
}
```

### Why it makes you a pro:
*   It separates creation logic from execution logic. If you need to add a new document type (e.g., `ExcelDocument`), you only modify the `DocumentFactory` class, leaving the client application code completely untouched.
