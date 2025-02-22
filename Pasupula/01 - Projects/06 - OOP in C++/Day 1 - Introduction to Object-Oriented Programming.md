
### **Objective:**
Understand the fundamental concepts of OOP, how it differs from procedural programming, and learn its four main principles.

---

## **Theory**

### **What is Object-Oriented Programming (OOP)?**
- **Definition**: A programming paradigm that organizes software design around objects, rather than functions and logic.
- Objects are instances of classes, which encapsulate both data (attributes) and functions (methods).
- OOP models real-world entities, making programs more intuitive, modular, and reusable.

---

### **Difference Between Procedural Programming and OOP**

| Feature         | Procedural Programming  | Object-Oriented Programming (OOP)    |
| --------------- | ----------------------- | ------------------------------------ |
| **Focus**       | Functions/Procedures    | Objects and Classes                  |
| **Data**        | Separate from functions | Encapsulated within objects          |
| **Reusability** | Limited                 | High (via inheritance, polymorphism) |
| **Examples**    | C, Pascal               | C++, Java, Python                    |

---

### **Four Principles of OOP**
1. **Encapsulation**:
   - Bundling of data and methods that operate on that data.
   - Protects the internal state of an object.
   - Example: Using private variables and public methods.

2. **Abstraction**:
   - Hiding unnecessary details and showing only essential information.
   - Example: A car's steering wheel vs. its internal mechanisms.

3. **Inheritance**:
   - Deriving a new class from an existing class.
   - Promotes code reusability and logical hierarchy.

4. **Polymorphism**:
   - Ability to take multiple forms.
   - Example: A function with the same name behaving differently based on arguments.

---

### **Why OOP?**
1. **Modularity**: Divide large programs into smaller parts.
2. **Reusability**: Use existing code for new applications.
3. **Maintainability**: Easily update and modify code.
4. **Scalability**: Manage complex applications efficiently.

---

## **Code Example: Basics of OOP**

Here's an example to illustrate OOP in C++:

```cpp
#include <iostream>
using namespace std;

// Define a class
class Example {
public:
    void show() { // Method inside the class
        cout << "Welcome to Object-Oriented Programming in C++!" << endl;
    }
};

int main() {
    Example obj; // Create an object of the class
    obj.show();  // Call the method using the object
    return 0;
}
```

**Explanation**:
	1. **Class**: Blueprint for creating objects.
1. **Object**: Instance of the class `Example`.
2. **Method**: Function `show()` inside the class.

---

## **Practice Problems**
1. Write a program with a class `Student` that:
   - Stores the student's name.
   - Prints a welcome message with their name.

2. Define a class `Car` that:
   - Has a method to display "This is a car."
   - Instantiate an object to call the method.

---

### **Day 1 Quiz**  
Complete this quiz before moving to Day 2.

### **Questions**:
1. Define Object-Oriented Programming in one sentence.
2. State two differences between procedural programming and OOP.
3. What are the four principles of OOP? Give a brief example of one.
4. What is a class and how is it different from an object?
5. Write the output of the following code:
   ```cpp
   class Demo {
   public:
       void display() {
           cout << "Hello, OOP!" << endl;
       }
   };

   int main() {
       Demo d;
       d.display();
       return 0;
   }
   ```