#### **Learning Objectives**:
1. Deepen understanding of the concept of classes and objects.
2. Explore advanced use of access specifiers (`private`, `public`, `protected`), and their role in real-world applications.
3. Understand the importance of the **constructor** and **destructor** in managing object lifecycle.
4. Learn about **dynamic memory allocation** and the **this pointer**.
5. Learn about **friend functions** and how they provide controlled access to private members.

---

#### **Detailed Theory**:

1. **Classes and Objects: Advanced Concepts**:
   - **Class**: A class is a user-defined type that serves as a blueprint for creating objects, which are instances of that class.
   - **Object**: An object is an instance of a class, where each object has its own state and behavior based on the class definition.
   - **Constructor**: A special member function of the class used to initialize objects. Constructors are automatically called when an object is created.
   - **Destructor**: A function invoked when an object is destroyed, often used to free resources or memory dynamically allocated.

   **Advanced Example**:
   
   > [!EXAMPLE] CODE
>```cpp
>   class Car {
>   private:
>       string brand;
>       int speed;
>
>   public:
>       // Constructor
>       Car(string b, int s) : brand(b), speed(s) {
>           cout << "Car object created!" << endl;
>       }
>
>       // Destructor
>       ~Car() {
>           cout << "Car object destroyed!" << endl;
>       }
>
>       void displayDetails() {
>           cout << "Brand: " << brand << ", Speed: " << speed << " km/h" << endl;
>       }
>   };
>
>   int main() {
>       Car car1("Toyota", 150); // Constructor is called here
>       car1.displayDetails();
>       // Destructor is called automatically when car1 goes out of scope
>       return 0;
>   }
>   ```

   **Explanation**:
   - **Constructor** initializes `brand` and `speed` when the object is created.
   - **Destructor** cleans up when the object goes out of scope, typically useful for memory management (e.g., freeing dynamically allocated memory).

2. **Access Specifiers** in Detail:
   - **Private**: Data is accessible only within the class itself. Essential for data encapsulation and securing sensitive information.
   - **Public**: Data is accessible from outside the class, allowing interaction with the object.
   - **Protected**: Data is accessible within the class and derived classes (important for inheritance).
   
   **Example**:
> [!EXAMPLE] CODE
>```cpp
>   class BankAccount {
>   private:
>       double balance;  // Private member
>
>   public:
>       BankAccount(double initial_balance) : balance(initial_balance) {}
>
>       // Public method to access private data
>       double getBalance() {
>           return balance;
>       }
>
>       void deposit(double amount) {
>           if (amount > 0) {
>               balance += amount;
>           }
>       }
>
>       void withdraw(double amount) {
>           if (amount <= balance) {
>               balance -= amount;
>           } else {
>               cout << "Insufficient funds!" << endl;
>           }
>       }
>   };
>   ```

3. **Dynamic Memory Allocation and the `this` Pointer**:
   - **Dynamic Memory Allocation** allows creating objects at runtime, rather than at compile-time, using the `new` keyword.
   - **`this` Pointer**: A special pointer available in non-static member functions that points to the current object.

> [!EXAMPLE] Dynamic Memory Allocation 
>```cpp
>   class Rectangle {
>   private:
>       int length, width;
>
>   public:
>       Rectangle(int l, int w) : length(l), width(w) {}
>
>       int area() {
>           return length * width;
>       }
>   };
>
>   int main() {
>       Rectangle* rect = new Rectangle(10, 5); // dynamically allocated
>       cout << "Area: " << rect->area() << endl;
>       delete rect;  // Don't forget to free the dynamically allocated memory
>       return 0;
>   }
>   ```

4. **Friend Functions**:
   - A **friend function** is a function or class that has access to the private and protected members of the class.
   - Friend functions are not members of the class but can be declared within the class using the `friend` keyword.

   **Example: Friend Function**:
   ```cpp
   class Box {
   private:
       int width;

   public:
       Box() : width(10) {}

       // Friend function can access private members
       friend void printWidth(Box&);
   };

   void printWidth(Box& b) {
       cout << "Width: " << b.width << endl;  // Access private member
   }

   int main() {
       Box box;
       printWidth(box);  // Calling friend function
       return 0;
   }
   ```

---

#### **Problem Questions**:

1. Create a `ComplexNumber` class with private attributes `real` and `imaginary` for a complex number. Write public methods to:
   - Set the real and imaginary parts.
   - Display the complex number.
   - Add two complex numbers and return the result.

2. Design a `Car` class that:
   - Has private attributes: `model`, `year`, `price`.
   - Include methods to:
     - Set and display the values of these attributes.
     - Calculate and return the depreciation of the car, assuming a 5% decrease in value each year since its manufacture.

3. Write a program that dynamically allocates an array of objects of a class `Employee` using the `new` keyword. The class should have attributes like `id`, `name`, and `salary`, and a method to display these details. Don't forget to free the memory at the end.

---

#### **Day 2 Quiz**:

**Instructions**: Answer all questions correctly to unlock Day 3.

1. Explain the difference between a **constructor** and a **destructor**. Provide examples.
2. What is the purpose of the `this` pointer in C++? Provide an example of its use.
3. Correct the following code and explain the error:
   ```cpp
   class BankAccount {
   private:
       double balance;

   public:
       BankAccount(double initial_balance) {
           balance = initial_balance;
       }

       double getBalance() {
           return balance;
       }

       void deposit(double amount) {
           balance += amount;
       }

       void withdraw(double amount) {
           balance -= amount;
       }
   };

   int main() {
       BankAccount* account = new BankAccount(500);
       account.deposit(100);
       account.withdraw(200);
       delete account;  // Correctly free memory
       account.deposit(50); // Error: access after delete
       return 0;
   }
   ```
4. What is a friend function in C++? Why would you use it?