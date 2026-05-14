# tech-notes


# Engineering Guide: Interfaces vs. Abstract Classes
> **Domain:** Android Architecture & Software Design Patterns

## 1. The Core Philosophy
In modern software engineering (specifically Android/Kotlin), the choice between these two is driven by **Composition over Inheritance**.

*   **Interface:** Defines a *Contract* or *Capability*. It answers "What can this object do?" (e.g., `Playable`, `Draggable`).
*   **Abstract Class:** Defines an *Identity* or *Template*. It answers "What is this object?" (e.g., `BaseActivity`, `Vehicle`).

---

## 2. Technical Comparison

| Feature | Interface | Abstract Class |
| :--- | :--- | :--- |
| **Inheritance** | Multiple (A class can implement many). | Single (A class can extend only one). |
| **State** | No backing fields (cannot hold data). | Can hold state (variables/fields). |
| **Constructors** | Prohibited (Cannot have `init` or params). | Allowed (used to initialize subclasses). |
| **Default Logic** | Allowed (via Kotlin default implementations). | Full implementation and private logic allowed. |

---

## 3. Practical Verification (Kotlin)

### A. Verifying State
An interface cannot "hold" a value in memory; an abstract class can.

     
      ```kotlin
      interface ITask {
          // val id: Int = 1 // ❌ ERROR: Initializer not allowed in interfaces
          val id: Int       // ✅ OK: Subclass must provide this property
      }
      
      abstract class BaseTask {
          val id: Int = 1   // ✅ OK: Abstract class can hold the actual state
      }```

### B. Verifying Constructors
Interfaces cannot be initialized, so they cannot have constructors.


      ```// ❌ ERROR: Interface cannot have a constructor
      // interface Network(val url: String) 
      
      abstract class Network(val url: String) {
          init { 
              println("Configuring connection to: $url") 
          }
      }
      
      // ✅ SUCCESS: Subclass satisfies the abstract constructor
      class ApiService : Network("[https://api.com](https://api.com)")```

### 4. Architectural Decision Matrix
✅ Use an Interface when:
- You want to support multiple behaviors across unrelated classes.  
- You are designing for Dependency Injection.  
- You want to keep your architecture decoupled and easily testable.  

✅ Use an Abstract Class when:
- You have closely related classes that share a lot of common logic or "DNA."  
- You need to manage internal state (properties that change over time).[cite: 1]
- You want to provide a restricted template for how subclasses should be initialized.[cite: 1]

The Golden Rule: Start with an Interface. Move to an Abstract Class only if you require a shared constructor or need to maintain private internal state.
      
