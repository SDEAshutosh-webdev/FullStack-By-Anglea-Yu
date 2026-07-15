# JavaScript Mastery: 5 Key Programs

Since ServiceNow developers script exclusively in JavaScript (both server-side and client-side), having a deep understanding of JS prototypes, ES6 classes, asynchronous processing, and closures is essential. These 5 programs demonstrate these crucial patterns.

---

## Program 1: ES6 Classes & Encapsulation (Private Fields)
Modern JavaScript (ES6+) supports classes. Encapsulation is achieved using private class fields (prefixed with `#`), which cannot be accessed outside the class.

### Implementation:
```javascript
class BankAccount {
    // Private fields (ES2022+ syntax, not accessible outside the class)
    #balance;
    #accountNumber;
    #transactionHistory = [];

    constructor(accountNumber, initialBalance) {
        this.#accountNumber = accountNumber;
        this.#balance = initialBalance >= 0 ? initialBalance : 0;
        this.#logTransaction(`Account created with balance: $${this.#balance}`);
    }

    // Getter for Balance
    get balance() {
        return this.#balance;
    }

    // Deposit method with validation
    deposit(amount) {
        if (amount <= 0) {
            throw new Error("Deposit amount must be positive.");
        }
        this.#balance += amount;
        this.#logTransaction(`Deposited: $${amount} | New Balance: $${this.#balance}`);
    }

    // Withdrawal method with validation
    withdraw(amount) {
        if (amount <= 0) {
            throw new Error("Withdrawal amount must be positive.");
        }
        if (amount > this.#balance) {
            throw new Error("Insufficient funds.");
        }
        this.#balance -= amount;
        this.#logTransaction(`Withdrew: $${amount} | New Balance: $${this.#balance}`);
    }

    // Getter for Transaction History
    getTransactionHistory() {
        // Return a copy to prevent mutation of the private history array
        return [...this.#transactionHistory];
    }

    // Private method
    #logTransaction(detail) {
        this.#transactionHistory.push(detail);
    }
}

// Client Execution
const account = new BankAccount("ACC-9081", 1000);
account.deposit(500);
account.withdraw(200);
console.log(`Current Balance: $${account.balance}`); // 1300
// console.log(account.#balance); // SyntaxError: Private field '#balance' must be declared in an enclosing class
```

### Why it makes you a pro:
*   It protects properties using `#` syntax.
*   It returns copies of arrays using the spread operator `[...]` to preserve immutable internal states.

---

## Program 2: Prototypal Inheritance (Under the Hood JS)
Unlike Java, JavaScript does not have built-in traditional classes; it uses **Prototypal Inheritance**. ES6 classes are just "syntactical sugar" over prototypes. Interviewers will test your understanding of how prototypes work.

### Implementation:
```javascript
// Base constructor function (like a class)
function Vehicle(make, model) {
    this.make = make;
    this.model = model;
}

// Adding methods to the prototype so they are shared across instances
Vehicle.prototype.start = function() {
    console.log(`${this.make} ${this.model} engine started.`);
};

// Sub-constructor function
function ElectricCar(make, model, batteryCapacity) {
    // Call the parent constructor (equivalent to super() in Java/ES6)
    Vehicle.call(this, make, model);
    this.batteryCapacity = batteryCapacity;
}

// Set up the prototype chain (Inheritance)
ElectricCar.prototype = Object.create(Vehicle.prototype);

// Reset the constructor pointer to ElectricCar (otherwise it points to Vehicle)
ElectricCar.prototype.constructor = ElectricCar;

// Overriding/Adding methods to child prototype
ElectricCar.prototype.charge = function() {
    console.log(`Charging the ${this.batteryCapacity} kWh battery.`);
};

// Client Execution
const tesla = new ElectricCar("Tesla", "Model Y", 75);
tesla.start();  // Output: Tesla Model Y engine started. (Inherited method)
tesla.charge(); // Output: Charging the 75 kWh battery. (Child method)
```

### Why it makes you a pro:
*   You demonstrate that you understand `Vehicle.call(this)` and `Object.create(Vehicle.prototype)` which is how JavaScript behaves under the hood.

---

## Program 3: Asynchronous JS (Promises & Async/Await)
ServiceNow is heavily dependent on asynchronous actions (like REST integrations and Client-side GlideAjax queries). Understanding Promises is essential.

### Implementation:
```javascript
// Simulating an API call fetching user data
function fetchUserFromServer(userId) {
    return new Promise((resolve, reject) => {
        console.log(`Fetching details for User ID: ${userId}...`);
        
        setTimeout(() => {
            const success = true; // Simulated flag
            if (success) {
                resolve({ id: userId, name: "Alice", role: "Developer" });
            } else {
                reject(new Error("Failed to fetch user data."));
            }
        }, 1500); // 1.5 second delay
    });
}

// Consuming Promise using modern Async/Await syntax
async function getUserProfile(userId) {
    try {
        const user = await fetchUserFromServer(userId);
        console.log("User Profile Loaded:", user);
    } catch (error) {
        console.error("Error occurred:", error.message);
    }
}

// Client Execution
getUserProfile("USR-789");
```

### Why it makes you a pro:
*   Instead of nested callbacks (callback hell), this uses `async/await` and `try/catch` which makes asynchronous code look synchronous, clean, and readable.

---

## Program 4: Closures & The Module Pattern
A **closure** is the combination of a function bundled together with references to its surrounding state. Closures allow JS developers to create private scopes.

### Implementation:
```javascript
const CounterModule = (function() {
    // Private variables inside local function scope (Closure)
    let count = 0;

    function logCount() {
        console.log(`Current Count: ${count}`);
    }

    // Public API returned to the outer scope
    return {
        increment: function() {
            count++;
            logCount();
        },
        decrement: function() {
            count--;
            logCount();
        },
        reset: function() {
            count = 0;
            logCount();
        }
    };
})();

// Client Execution
CounterModule.increment(); // Output: Current Count: 1
CounterModule.increment(); // Output: Current Count: 2
CounterModule.decrement(); // Output: Current Count: 1
// CounterModule.count = 100; // Has no effect, 'count' is inaccessible externally
```

### Why it makes you a pro:
*   This uses the **Immediately Invoked Function Expression (IIFE)** and closures to isolate internal states. This is a common pattern in ServiceNow Client Script libraries to prevent variable contamination in global browser scope.

---

## Program 5: Callbacks & Array Operations
ServiceNow server scripts often query records and process tables. Using high-order functions (`filter`, `map`, `reduce`) with callback functions is the modern JS standard for data manipulation.

### Implementation:
```javascript
const incidents = [
    { number: "INC001", state: "Active", priority: 1, duration: 4 },
    { number: "INC002", state: "Resolved", priority: 3, duration: 12 },
    { number: "INC003", state: "Active", priority: 2, duration: 8 },
    { number: "INC004", state: "Resolved", priority: 1, duration: 6 }
];

// 1. Callback using Array.filter (Get only Active Incidents)
const activeIncidents = incidents.filter(inc => inc.state === "Active");
console.log("Active Incidents:", activeIncidents);

// 2. Callback using Array.map (Extract only incident numbers)
const incidentNumbers = incidents.map(inc => inc.number);
console.log("Incident Numbers:", incidentNumbers);

// 3. Callback using Array.reduce (Calculate total duration of all resolved incidents)
const totalResolvedDuration = incidents
    .filter(inc => inc.state === "Resolved")
    .reduce((total, inc) => total + inc.duration, 0);

console.log("Total Resolved Duration (hours):", totalResolvedDuration); // 18
```

### Why it makes you a pro:
*   Instead of using standard, verbose `for` loops, you use declarative, functional array methods (`filter`/`map`/`reduce`), making record collection parsing highly optimized and clean.
