# TwinCAT Object-Oriented Programming -- Complete Reference Guide

### DUT · ABSTRACT · INTERFACE · INHERITANCE · FINAL · SEALING · POLYMORPHISM · OOP ARCHITECTURE

------------------------------------------------------------------------

# 1. Introduction

This document provides a complete, unified reference for **TwinCAT OOP
development**, combining:

-   **DUT (Data Unit Types)**\
-   **ABSTRACT concepts**\
-   **INTERFACE usage**\
-   **INHERITANCE and EXTENDS**\
-   **FINAL / SEALED classes**\
-   **POLYMORPHISM**\
-   **Advanced OOP patterns for PLC development**

All content is written **entirely in English** with **no special
characters**, ensuring compatibility with all Markdown processors.

------------------------------------------------------------------------

# 2. DUT -- Data Unit Types

A **DUT** (Data Unit Type) defines a user-specific data type inside a
TwinCAT PLC project.\
TwinCAT supports the following DUT types:

-   Structure\
-   Enumeration\
-   Alias\
-   Union

------------------------------------------------------------------------

## 2.1 Creating a DUT

Steps:

1.  Select a folder inside Solution Explorer.\
2.  Right-click → **Add \> DUT...**\
3.  Enter name and choose a data type.\
4.  Click **Open**.\
5.  The DUT is added to the project tree.

------------------------------------------------------------------------

## 2.2 Structure Type

A structure combines variables of different data types into one logical
unit.

### EXTENDS in Structures

TwinCAT allows structure inheritance:

``` pascal
TYPE ST_Struct1 :
STRUCT
    nVar1 : INT;
    bVar2 : BOOL;
END_STRUCT
END_TYPE

TYPE ST_Struct2 EXTENDS ST_Struct1 :
STRUCT
    nVar3 : DWORD;
    sVar4 : STRING;
END_STRUCT
END_TYPE
```

------------------------------------------------------------------------

## 2.3 Enumeration

Enumerations group integer constants under a single logical name.

Text List Support allows:

-   Localization\
-   Displaying text instead of numeric values in visualization

Example:

``` pascal
TYPE E_TrafficSignal :
(
    eRed,
    eYellow,
    eGreen := 10
);
END_TYPE
```

------------------------------------------------------------------------

## 2.4 Alias Type

Aliases assign an alternative name to an existing data type.

``` pascal
TYPE T_Message : STRING[50];
END_TYPE
```

------------------------------------------------------------------------

## 2.5 Union Type

A union stores different components in the same memory space.

``` pascal
TYPE U_Name :
UNION
    fA : LREAL;
    nB : LINT;
    nC : WORD;
END_UNION
END_TYPE
```

------------------------------------------------------------------------

# 3. ABSTRACT Concept

The keyword **ABSTRACT** is available for:

-   Function Blocks\
-   Methods\
-   Properties

It is used to implement abstraction layers, a key concept in
object-oriented programming.

------------------------------------------------------------------------

## 3.1 Rules for ABSTRACT

-   Abstract FBs cannot be instantiated.\
-   Abstract FBs may contain abstract and non-abstract members.\
-   Abstract methods contain declarations only.\
-   If any abstract method exists, the FB must be abstract.\
-   Derived FBs must implement abstract members or also be abstract.

------------------------------------------------------------------------

## 3.2 Example -- Abstract Base Class

``` pascal
FUNCTION_BLOCK ABSTRACT FB_System_Base

PROPERTY nSystemID : UINT

METHOD ABSTRACT Execute
```

------------------------------------------------------------------------

## 3.3 Derived Class Implementation

``` pascal
FUNCTION_BLOCK FB_StackSystem EXTENDS FB_System_Base

METHOD Execute
    // specific implementation
END_METHOD
```

------------------------------------------------------------------------

# 4. INTERFACE Concept

An interface defines a contract of methods and properties without
implementation.

------------------------------------------------------------------------

## 4.1 Interface Example

``` pascal
INTERFACE ISystem
    METHOD Execute : BOOL;
    METHOD Reset;
END_INTERFACE
```

------------------------------------------------------------------------

## 4.2 Implementing Interfaces

A Function Block can implement multiple interfaces:

``` pascal
FUNCTION_BLOCK FB_Motor IMPLEMENTS ISystem, ILogging
```

All interface members must be implemented by the FB.

------------------------------------------------------------------------

# 5. INHERITANCE and EXTENDS

TwinCAT supports single inheritance through the **EXTENDS** keyword.

Example:

``` pascal
FUNCTION_BLOCK FB_Child EXTENDS FB_Parent
```

Benefits:

-   Code reuse\
-   Specialization of general behavior

------------------------------------------------------------------------

# 6. FINAL and SEALING

TwinCAT uses **FINAL** to seal classes and methods.

### Rules:

-   A FINAL FB cannot be extended.\
-   A FINAL method cannot be overridden.

### Example

``` pascal
FUNCTION_BLOCK FINAL FB_DriveController
```

Final Method:

``` pascal
METHOD FINAL CalculateChecksum
```

------------------------------------------------------------------------

# 7. POLYMORPHISM

Polymorphism allows calling the same method on different objects with
different behavior.

------------------------------------------------------------------------

## 7.1 Abstract + Override Example

Base class:

``` pascal
FUNCTION_BLOCK ABSTRACT FB_System_Base
METHOD ABSTRACT Execute
```

Derived classes override "Execute":

``` pascal
METHOD Execute
    // custom behavior
END_METHOD
```

Polymorphic usage:

``` pascal
VAR
    system : FB_System_Base;
    stack  : FB_StackSystem;
END_VAR

system := stack;
system.Execute();   // Executes FB_StackSystem.Execute
```

------------------------------------------------------------------------

# 8. OVERRIDE Keyword

Used in derived classes to replace the implementation from a parent
class.

``` pascal
METHOD OVERRIDE Execute
```

------------------------------------------------------------------------

# 9. Advanced OOP Architecture Example

### Interface

``` pascal
INTERFACE ISystem
    METHOD Execute : BOOL;
END_INTERFACE
```

### Abstract Base Class

``` pascal
FUNCTION_BLOCK ABSTRACT FB_System_Base IMPLEMENTS ISystem
VAR
    nSystemID : UINT;
END_VAR

METHOD ABSTRACT Execute : BOOL
```

### Concrete Class

``` pascal
FUNCTION_BLOCK FB_RobotSystem EXTENDS FB_System_Base
METHOD OVERRIDE Execute : BOOL
END_METHOD
```

### Sealed Class

``` pascal
FUNCTION_BLOCK FINAL FB_RobotPro EXTENDS FB_RobotSystem
METHOD OVERRIDE Execute : BOOL
END_METHOD
```

Usage:

``` pascal
system := robotPro;
system.Execute();
```

------------------------------------------------------------------------

# 10. Conclusion

This document unifies all major TwinCAT OOP concepts:

-   Data Unit Types (DUT)\
-   Abstract classes and methods\
-   Interface-based development\
-   Inheritance and extension\
-   Final / sealed constructs\
-   Polymorphism\
-   Full OOP architecture patterns

It serves as a complete reference for building scalable, maintainable
TwinCAT PLC software.
