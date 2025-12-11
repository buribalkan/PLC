# ABSTRACT Concept -- TwinCAT PLC Object-Oriented Programming

## 1. Overview

The `ABSTRACT` keyword is available for:

-   **Function Blocks (FB)**
-   **Methods**
-   **Properties**

It enables the implementation of abstraction levels within a TwinCAT PLC
project, forming a foundational part of **object-oriented programming
(OOP)** in IEC 61131-3's extended model.

------------------------------------------------------------------------

## 2. Version Information

**ABSTRACT**\
***Available starting from TwinCAT 3.1 Build 4024***

------------------------------------------------------------------------

## 3. Purpose of Abstraction

Abstraction allows developers to define:

-   **General behaviors** in base classes\
-   **Specific implementations** in derived classes

This promotes:

-   Code reusability\
-   Cleaner architecture\
-   Maintainability

The concept is similar to interfaces, with one key difference:

  Concept              Contains Implementation?   Allows Non-Abstract Members?
  -------------------- -------------------------- ------------------------------
  **Interface**        ❌ No                      ❌ No
  **Abstract Class**   ✔ Yes                      ✔ Yes

An abstract class can contain both:

-   **Abstract members** → declaration only\
-   **Concrete members** → full implementation

------------------------------------------------------------------------

## 4. Rules for Using `ABSTRACT`

A clear set of rules governs the use of abstract constructs:

### ✔ 1. Abstract Function Blocks Cannot Be Instantiated

They serve only as templates and cannot be used directly with
`VAR … : FB_Name;`.

------------------------------------------------------------------------

### ✔ 2. Abstract FBs Can Contain Both:

-   Abstract methods/properties\
-   Non-abstract (fully implemented) methods/properties

------------------------------------------------------------------------

### ✔ 3. Abstract Methods/Properties Contain No Implementation

They include:

-   Declaration\
-   Signature

...but **no code** in the implementation part.

------------------------------------------------------------------------

### ✔ 4. If an FB Contains Any Abstract Member, the FB Must Be Abstract

This ensures consistency and prevents accidental instantiation.

------------------------------------------------------------------------

### ✔ 5. Abstract FBs Must Be Extended to Be Used

Only **derived (child) function blocks** can implement abstract members.

------------------------------------------------------------------------

### ✔ 6. A Derived FB Must Either:

-   Implement **all** abstract methods/properties from the parent\
    **OR**
-   Also be declared `ABSTRACT`

------------------------------------------------------------------------

## 5. Example Implementation

### 5.1 Abstract Base Class

``` pascal
FUNCTION_BLOCK ABSTRACT FB_System_Base
```

This abstract base class contains:

-   A **concrete property** → `nSystemID`
-   An **abstract method** → `Execute`

#### Property Implementation

``` pascal
PROPERTY nSystemID : UINT
```

#### Abstract Method Declaration

``` pascal
METHOD ABSTRACT Execute
```

Explanation:

-   All systems share the same `nSystemID` logic → implemented here.\
-   Each system behaves differently when executed → defined abstract.

------------------------------------------------------------------------

## 6. Implementing the Abstract Base Class in a Derived Class

### 6.1 Non-Abstract Sub-Class

``` pascal
FUNCTION_BLOCK FB_StackSystem EXTENDS FB_System_Base
```

Since this class is **not abstract**, it **must implement** the abstract
method:

### Execute Method Implementation in Derived Class

``` pascal
METHOD Execute
```

This method defines:

-   The specific stack system execution logic\
-   Behavior unique to this derived implementation

------------------------------------------------------------------------

## 7. Summary

The **ABSTRACT** keyword enables:

-   Hierarchical system design\
-   Separation of general vs. specific logic\
-   Enforcement of structured OOP patterns in TwinCAT PLC development

Using abstract base classes ensures:

-   Reusable architecture\
-   Cleaner and safer code\
-   Avoidance of duplication across multiple system modules
