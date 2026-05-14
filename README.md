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

### 5. Why Android Prefers Interfaces (Official Context)
According to official Android architectural patterns and Kotlin best practices, interfaces are preferred for the following reasons:

#### 🚀 Reason 1: The "Fragile Base Class" Problem
In Android, if you rely heavily on deep abstract class hierarchies (Inheritance), a small change in a base class (like a BaseFragment) can accidentally break logic in dozens of child classes.

The Interface Solution: Interfaces provide a "flat" structure. Changing one interface doesn't ripple through a whole family tree, making your app much easier to maintain.  

#### 🧩 Reason 2: Multiple Contract Implementation
Since Java and Kotlin do not support Multiple Inheritance (you can't extend two classes), using an abstract class locks you in.

The Interface Solution: A single ViewModel or Repository can implement IAnalytics, ILogger, and IDataSource all at once. This flexibility is core to Modularization.  

#### 🧪 Reason 3: Testing & Mocking
Android's testing documentation heavily emphasizes Dependency Injection (DI).

The Interface Solution: It is significantly easier for testing frameworks to create a "Mock" of an interface than to mock a complex abstract class that might have hidden logic or lifecycle dependencies.


#### The Golden Rule: 
Start with an Interface. Move to an Abstract Class only if you require a shared constructor or need to maintain private internal state.
      







# Tech Notes: Android R8, Obfuscation, and De-obfuscation Workflow

**Date:** May 14, 2026  
**Author:** Aryan Dhankar

## 1. Overview of R8 in Android
R8 is the default compiler for Android that converts Java bytecode into optimized DEX code. It handles four critical tasks in a single step:

* **Code Shrinking (Tree Shaking):** Analyzes the code and removes unreachable classes, fields, and methods.
* **Resource Shrinking:** Identifies unused resources in the app and removes them from the APK/Bundle.
* **Obfuscation:** Renames classes and members with short, meaningless names (e.g., `ln.c`) to reduce file size and increase security.
* **Optimization:** Rewrites code to improve runtime performance (e.g., method inlining, dead code removal).

## 2. Project Configuration
To enable R8 and full obfuscation for testing, update the `app/build.gradle` file:

          ```gradle
          android {
              buildTypes {
                  release {
                      minifyEnabled true
                      shrinkResources true
                      proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
                  }
              }
          }


### Advanced Scrambling Settings
To force R8 to remove line numbers and source file attributes (useful for testing manual retrace), add these to your proguard-rules.pro:

#Remove attributes that make logs readable
-keepattributes !SourceFile,!LineNumberTable

#Force R8 to rename all methods/classes not explicitly kept
-repackageclasses ''
-allowaccessmodification

## 3. Generating the Signed Release APK
Android devices reject unsigned release builds. You must sign the APK to install it:

Go to Build > Generate Signed Bundle / APK....

Select APK and provide your Keystore credentials.

Select the release variant.

Ensure V1 (Jar Signature) and V2 (Full APK Signature) are both checked to ensure compatibility with older and newer Android versions.

## 4. Capturing Obfuscated Crash Logs
Once the signed app is installed and crashes, you can view the logs in two ways:

Android Studio Logcat:

Connect the device and filter for Error level logs.

You will see obfuscated identifiers (e.g., at ln.c(...)).

Copy these logs into a crash_log.txt for manual retracing.

Firebase Crashlytics:

If integrated, Firebase will capture the crash automatically.

Crucial Step: You must upload your mapping.txt to the Firebase Console.

Once uploaded, Firebase uses the mapping file to show you readable method names directly in the dashboard.

## 5. De-obfuscation (Retracing) Workflow :The Retracing Command (Manual)
If you are analyzing local logs, run this command in your terminal:

When a release build crashes, the stack trace is obfuscated. Use the mapping.txt file to translate it.

### Step A: Locate the Mapping File
R8 generates the mapping file during every release build at:

app/build/outputs/mapping/release/mapping.txt

### Step B: Capture and Save the Crash
Trigger the crash on your device.

Filter for errors in Logcat.

Save the raw logs to a file: C:\\Users\\PC\\Desktop\\crash_log.txt.

### Step C: Execute Retrace via CMD
Run the retrace utility and redirect the output to a readable file for analysis:

#### Command structure for Windows
C:\\Users\\PC\\AppData\\Local\\Android\\Sdk\\cmdline-tools\\latest\\bin\\retrace.bat ^
"C:\\Path\\To\\Your\\Project\\app\\build\\outputs\\mapping\\release\\mapping.txt" ^
"C:\\Users\\PC\\Desktop\\crash_log.txt" > "C:\\Users\\PC\\Desktop\\crash_log_readable.txt"

#### Command Breakdown:

- retrace.bat: The executable script that performs the de-obfuscation logic.
- mapping.txt: The "translation key" generated during the release build.   
- crash_log.txt: The raw, obfuscated file containing the "alphabet soup" logs (e.g., at ln.c).   
- >: The redirection operator that sends the output to a file instead of the screen.
- crash_log_readable.txt: The final output file containing your original method names and line numbers. [cite: 1]


## 5. Summary & Best Practices
Unique Mappings: Every single APK/Bundle build has its own unique mapping.txt. Never mix them up.

No Git for Mappings: Do not commit mapping.txt to version control. Store it as a build artifact.

Auto-Retrace: Always upload mapping.txt to the Google Play Console or Firebase Crashlytics during release for automatic dashboard de-obfuscation.

SDK Tools: Ensure "Android SDK Command-line Tools" are installed via the SDK Manager to access retrace.bat
