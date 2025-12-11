# TwinCAT OOP -- Advanced Concepts

### SEALING · FINAL · INTERFACE · POLYMORPHISM · INHERITANCE · ABSTRACT · OVERRIDE

------------------------------------------------------------------------

## 1. Overview

TwinCAT 3, IEC 61131-3 standardını genişleterek modern OOP
(Object-Oriented Programming) özelliklerini PLC geliştirme ortamına
taşır. Bu sayede:

-   Modüler\
-   Yeniden kullanılabilir\
-   Genişletilebilir\
-   Bakımı kolay

kod yazmak mümkün olur.

Bu doküman TwinCAT'in sunduğu ileri seviye OOP özelliklerini açıklar.

------------------------------------------------------------------------

## 2. SEALING Concept

`SEALING`, bir **class/function block'ın daha fazla türetilmesini
engellemek** için kullanılır.

### Ne İşe Yarar?

-   Güvenlik gerektiren modüllerde miras alınmasını engeller.\
-   Bir sınıfın artık *tamamlanmış* olduğunu belirtir.

### Kullanım

``` pascal
FUNCTION_BLOCK FINAL FB_DriveController
```

Bu FB başka bir FB tarafından **EXTENDS** ile genişletilemez.

------------------------------------------------------------------------

## 3. FINAL Keyword

TwinCAT'te `FINAL` aynı zamanda **sealed class** anlamına gelir.

Kurallar:

-   FINAL bir FB genişletilemez.\
-   FINAL bir method override edilemez.

### Örnek

``` pascal
METHOD FINAL CalculateChecksum
```

------------------------------------------------------------------------

## 4. INTERFACE Concept

`INTERFACE`, tamamen soyut bir davranış sözleşmesidir.

### Interface Tanımı

``` pascal
INTERFACE ISystem
    METHOD Execute : BOOL;
    METHOD Reset;
END_INTERFACE
```

### Interface Kullanımı

``` pascal
FUNCTION_BLOCK FB_Motor IMPLEMENTS ISystem, ILogging
```

Bir FB, birden fazla interface'i uygulayabilir.

------------------------------------------------------------------------

## 5. POLYMORPHISM (Çok Biçimlilik)

Polymorphism, aynı methodun farklı sınıflarda farklı şekilde
uygulanabilmesini sağlar.

TwinCAT'te polymorphism şu yollarla oluşur:

-   Abstract methods\
-   Override methods\
-   Interface implementation\
-   Base class reference üzerinden derived class nesnesine erişim

### Örnek --- Abstract + Override

``` pascal
FUNCTION_BLOCK ABSTRACT FB_System_Base
METHOD ABSTRACT Execute
```

#### Derived Class 1

``` pascal
FUNCTION_BLOCK FB_StackSystem EXTENDS FB_System_Base
METHOD Execute
    // Stack behavior
END_METHOD
```

#### Derived Class 2

``` pascal
FUNCTION_BLOCK FB_ConveyorSystem EXTENDS FB_System_Base
METHOD Execute
    // Conveyor behavior
END_METHOD
```

### Kullanım

``` pascal
VAR
    fbSystem : FB_System_Base; 
    fbStack : FB_StackSystem;
END_VAR

fbSystem := fbStack;
fbSystem.Execute(); // Polymorphic behavior
```

------------------------------------------------------------------------

## 6. OVERRIDE Keyword

Bir derived class, base class methodunu yeniden tanımlamak için
`OVERRIDE` kullanır.

``` pascal
METHOD OVERRIDE Execute
```

------------------------------------------------------------------------

## 7. OOP Feature Comparison Table

  Özellik          ABSTRACT   FINAL   INTERFACE   OVERRIDE   EXTENDS
  ---------------- ---------- ------- ----------- ---------- ---------
  Instancing       ❌         ✔       ❌          ✔          ✔
  Implementation   Kısmi      Tam     Yok         ✔          ✔
  Inheritance      ✔          ❌      Uygulanır   --         ✔
  Polymorphism     ✔          ✔       ✔           ✔          ✔

------------------------------------------------------------------------

## 8. Complete OOP Hierarchy Example

### 8.1 Interface

``` pascal
INTERFACE ISystem
    METHOD Execute : BOOL;
END_INTERFACE
```

------------------------------------------------------------------------

### 8.2 Abstract Base Class

``` pascal
FUNCTION_BLOCK ABSTRACT FB_System_Base IMPLEMENTS ISystem
VAR
    nSystemID : UINT;
END_VAR

METHOD ABSTRACT Execute : BOOL
```

------------------------------------------------------------------------

### 8.3 Concrete Derived Class

``` pascal
FUNCTION_BLOCK FB_RobotSystem EXTENDS FB_System_Base
METHOD OVERRIDE Execute : BOOL
    // robot behavior
END_METHOD
```

------------------------------------------------------------------------

### 8.4 Sealed Derived Class (FINAL)

``` pascal
FUNCTION_BLOCK FINAL FB_RobotPro EXTENDS FB_RobotSystem
METHOD OVERRIDE Execute : BOOL
    // improved robot behavior
END_METHOD
```

------------------------------------------------------------------------

### 8.5 Kullanım

``` pascal
VAR
    system    : ISystem;
    robot     : FB_RobotSystem;
    robotPro  : FB_RobotPro;
END_VAR

system := robot;
system.Execute();

system := robotPro;
system.Execute();
```

------------------------------------------------------------------------

## 9. Conclusion

Bu doküman, TwinCAT'teki tüm gelişmiş nesne yönelimli programlama
yapılarını kapsayan kapsamlı bir referans sağlar:

-   ABSTRACT\
-   SEALING / FINAL\
-   INTERFACE\
-   POLYMORPHISM\
-   OVERRIDE\
-   INHERITANCE

TwinCAT OOP yapısı, büyük ölçekli ve sürdürülebilir PLC yazılımları
oluşturmayı kolaylaştırır.
