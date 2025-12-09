# TwinCAT PLC – Method / Action / Transition / Property Eğitim Notları  
## C# ile Karşılaştırmalı Anlatım

---

## 1. Kavramlara Genel Bakış

TwinCAT PLC (ST – Structured Text) yapı olarak C#’a benzese de **gerçek zamanlı çalışma**, **deterministik davranış** ve **IEC 61131-3 standartlarını** temel alır.

| Kavram | TwinCAT PLC | C# Karşılığı | Açıklama |
|-------|-------------|--------------|----------|
| **Method** | FB içinde fonksiyonel alt yapı | Class metodu | Bir FB’nin davranışlarını parçalar. |
| **Action** | Bir POU’nun (özellikle SFC) içinde ayrı davranış blokları | C#’ta doğrudan yok | SFC’de belirli adım kodunu izole eder. |
| **Transition** | SFC’de adımlar arası geçiş koşulu | State machine geçiş ifadeleri | SFC’de adımları bağlayan koşuldur. |
| **Property** | FB içinde get/set kontrollü değişken | C# property | Veri kapsülleme sağlar. |

---

## 2. Method – Ne? Neden? Nasıl?

### TwinCAT PLC’de Method
Bir Function Block içinde tanımlanan fonksiyonel alt yapıdır. Modüler kod yazmayı sağlar.

### Örnek (PLC)
```pascal
FUNCTION_BLOCK FB_Motor
VAR
    Speed : INT;
END_VAR

METHOD SetSpeed : BOOL
VAR_INPUT
    NewSpeed : INT;
END_VAR
Speed := NewSpeed;
SetSpeed := TRUE;
```

### Örnek (C#)
```csharp
class Motor
{
    public int Speed;

    public bool SetSpeed(int newSpeed)
    {
        Speed = newSpeed;
        return true;
    }
}
```

---

## 3. Action – Ne? Neden? Nasıl?

### Action Nedir?
SFC içinde bir adımın davranışını temsil eden kod bloklarıdır.

### Örnek (PLC)
```pascal
ACTION Act_OpenValve
ValveCmd := TRUE;
END_ACTION
```

### C#’ta karşılığı
State’e özel metod yazmaya benzer:
```csharp
void State_Open()
{
    ValveCmd = true;
}
```

---

## 4. Transition – Ne? Neden? Nasıl?

### Transition Nedir?
SFC adımlarının birbirine geçiş koşuludur.

#### Örnek (PLC)
```pascal
Step_Open -> Step_Close WHEN ValveIsOpen;
```

### C# karşılığı
```csharp
if (ValveIsOpen)
    currentState = State.Close;
```

---

## 5. Property – Ne? Neden? Nasıl?

### PLC’de Property
C# ile aynı felsefede kapsülleme sağlar.

#### Örnek (PLC)
```pascal
PROPERTY Speed : INT
GET
    Speed := _Speed;
END_GET
SET
    _Speed := Value;
END_SET
```

#### Örnek (C#)
```csharp
public int Speed
{
    get { return _speed; }
    set { _speed = value; }
}
```

---

## 6. Senaryo – Motor Kontrolü (Method + Action + Transition + Property)

### SFC Adımları
- Idle  
- Starting  
- Running  
- Stopping  

### C# karşılığı state machine yapısı:

```csharp
switch(state)
{
    case MotorState.Idle:
        if(StartCmd) state = MotorState.Starting;
        break;
}
```

---

## Özet

| PLC Kavramı | C# Karşılığı | Amaç |
|------------|--------------|------|
| Method | Method | Fonksiyonel davranış bölme |
| Action | State’e özel metod | SFC adım davranışı |
| Transition | if koşulu | Adımlar arası geçiş |
| Property | Property | Veri kapsülleme |

---


