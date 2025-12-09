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

# TwinCAT PLC – Adım Adım Egzersiz Seti  
## C# Karşılaştırmalı Uygulamalı Eğitim

---

# 1. Başlangıç Seviyesi: Method Temelleri

## Egzersiz 1 – Basit Bir Method Yaz
**Görev:**  
Bir `FB_Counter` adlı Function Block oluştur.  
İçinde:
- `Count : INT`
- `Increment()` Methodu

**Increment methodu:**
- Count’u 1 arttırır
- TRUE döndürür

**C# benzeri:**
```csharp
public bool Increment()
{
    Count++;
    return true;
}
```

---

## Egzersiz 2 – Parametre Alan Method
**Görev:**  
`SetValue(value : INT)` methodu ekle.

Kurallar:
- Count = value
- value negatifse FALSE döndür

---

## Egzersiz 3 – Methodları Kullan
Program:
1. Counter FB örneği oluştur  
2. `Increment()` 3 kez çağır  
3. `SetValue(10)` çalıştır  
4. Sonucu sakla  

---

# 2. Orta Seviye: Property Kullanımı

## Egzersiz 4 – Get/Set Property Oluştur
**Görev:**  
`Speed` PROPERTY'si oluştur.

Kurallar:
- Speed 0–3000 arasında olmalı  
- Set geçersizse `_Speed` değişmesin  

**C# benzeri:**
```csharp
public int Speed
{
    get { return _speed; }
    set { if(value >= 0 && value <= 3000) _speed = value; }
}
```

---

## Egzersiz 5 – Property + Method İlişkisi
**Görev:**  
`IncreaseSpeed(step : INT)` methodu yaz.

Kurallar:
- Speed = Speed + step  
- Speed > 3000 ise Speed = 3000  

---

# 3. Orta Seviye: Actions (SFC) Kullanımı

## Egzersiz 6 – Basit SFC Oluştur
Adımlar:
- Step_Init  
- Step_Heat  
- Step_Ready  

Actions:
- Act_Init → Heater OFF  
- Act_Heat → Heater ON  
- Act_Ready → LED ON  

---

## Egzersiz 7 – Transition Koşulları
Geçişler:
- Init → Heat : `StartCmd = TRUE`
- Heat → Ready : `Temp >= 80`
- Ready → Init : `ResetCmd = TRUE`

**C# karşılığı:**
```csharp
if(StartCmd) state = Heat;
if(Temp >= 80) state = Ready;
if(ResetCmd) state = Init;
```

---

# 4. İleri Seviye: Method + Action + Transition Birlikte Kullanımı

## Egzersiz 8 – Motor Kontrol FB’si
FB değişkenleri:
- `_Speed : INT`
- `Running : BOOL`
- `Error : BOOL`

Methodlar:
1. `Start()` → Running=TRUE, Speed=500  
2. `Stop()` → Running=FALSE, Speed=0  
3. `SetSpeed(NewSpeed)` → 0–3000 dışıysa Error=TRUE  

---

## Egzersiz 9 – Motor İçin SFC
Adımlar:
- Step_Stopped  
- Step_Starting  
- Step_Running  
- Step_Stopping  

Transitionlar:
- Stopped → Starting : StartCmd  
- Starting → Running : Speed >= 500  
- Running → Stopping : StopCmd  
- Stopping → Stopped : Speed = 0  

---

# 5. Uzman Seviyesi: Tam Sistem Projesi

## Egzersiz 10 – Konveyör Sistemi Tasarımı

### Function Block: `FB_Conveyor`
Properties:
- Speed (0–100)
- Loaded (BOOL)
- Fault (BOOL)

Methodlar:
- Start()
- Stop()
- Load()
- Unload()
- SetSpeed()

### SFC Adımları:
- Idle  
- Loading  
- Running  
- Unloading  
- Fault  

### Transitionlar:
- Idle → Loading : ProductDetected  
- Loading → Running : Loaded=TRUE  
- Running → Unloading : DestinationReached  
- Unloading → Idle : Loaded=FALSE  
- Any → Fault : FaultDetected  

### Actions:
- Motor kontrol  
- LED göstergeleri  
- Fault reset işlemleri  

### C# Modelleme:
Sistemi class + state machine ile modelle.

---

# TwinCAT PLC Egzersiz Çözümleri  
## C# Karşılaştırmalı Çözüm Seti

---

# 1. Başlangıç Seviyesi Çözümler

## Çözüm – Egzersiz 1: Increment Method
```pascal
FUNCTION_BLOCK FB_Counter
VAR
    Count : INT := 0;
END_VAR

METHOD Increment : BOOL
Count := Count + 1;
Increment := TRUE;
```

---

## Çözüm – Egzersiz 2: SetValue Method
```pascal
METHOD SetValue : BOOL
VAR_INPUT
    value : INT;
END_VAR

IF value >= 0 THEN
    Count := value;
    SetValue := TRUE;
ELSE
    SetValue := FALSE;
END_IF
```

---

## Çözüm – Egzersiz 3: Method Kullanımı
```pascal
PROGRAM MAIN
VAR
    Cnt : FB_Counter;
    Result : INT;
END_VAR

Cnt.Increment();
Cnt.Increment();
Cnt.Increment();
Cnt.SetValue(10);
Result := Cnt.Count;
```

---

# 2. Orta Seviye: Property Çözümleri

## Çözüm – Egzersiz 4: Speed Property
```pascal
FUNCTION_BLOCK FB_Motor
VAR
    _Speed : INT;
END_VAR

PROPERTY Speed : INT
GET
    Speed := _Speed;
END_GET

SET
IF (Value >= 0) AND (Value <= 3000) THEN
    _Speed := Value;
END_IF
END_SET
```

---

## Çözüm – Egzersiz 5: IncreaseSpeed Method
```pascal
METHOD IncreaseSpeed
VAR_INPUT
    step : INT;
END_VAR

_Speed := _Speed + step;

IF _Speed > 3000 THEN
    _Speed := 3000;
END_IF
```

---

# 3. SFC – Action ve Transition Çözümleri

## Çözüm – Egzersiz 6: Basit SFC Actions

**Act_Init**
```pascal
Heater := FALSE;
```

**Act_Heat**
```pascal
Heater := TRUE;
```

**Act_Ready**
```pascal
LED := TRUE;
```

---

## Çözüm – Egzersiz 7: Transition Koşulları

```pascal
(* Init → Heat *)
Transition_Init_Heat := StartCmd;

(* Heat → Ready *)
Transition_Heat_Ready := (Temp >= 80);

(* Ready → Init *)
Transition_Ready_Init := ResetCmd;
```

---

# 4. İleri Seviye FB + SFC Çözümleri

## Çözüm – Egzersiz 8: Motor FB
```pascal
FUNCTION_BLOCK FB_Motor
VAR
    _Speed : INT;
    Running : BOOL;
    Error : BOOL;
END_VAR

METHOD Start
Running := TRUE;
_Speed := 500;

METHOD Stop
Running := FALSE;
_Speed := 0;

METHOD SetSpeed
VAR_INPUT
    NewSpeed : INT;
END_VAR

IF (NewSpeed >= 0) AND (NewSpeed <= 3000) THEN
    _Speed := NewSpeed;
ELSE
    Error := TRUE;
END_IF
```

---

## Çözüm – Egzersiz 9: Motor SFC

**Actions**

Act_Stopped:
```pascal
Motor.Stop();
```

Act_Starting:
```pascal
Motor.Start();
```

Act_Running:
```pascal
Motor.SetSpeed(1000);
```

Act_Stopping:
```pascal
Motor.Stop();
```

**Transitions**
```pascal
Stopped_Starting := StartCmd;
Starting_Running := (Motor._Speed >= 500);
Running_Stopping := StopCmd;
Stopping_Stopped := (Motor._Speed = 0);
```

---

# 5. Uzman Seviyesi: Konveyör Sistemi Çözümü

## Çözüm – Egzersiz 10: FB_Conveyor
```pascal
FUNCTION_BLOCK FB_Conveyor
VAR
    _Speed : INT;
    Loaded : BOOL;
    Fault : BOOL;
END_VAR

PROPERTY Speed : INT
GET
    Speed := _Speed;
END_GET
SET
IF (Value >= 0) AND (Value <= 100) THEN
    _Speed := Value;
END_IF
END_SET

METHOD Start
_Speed := 50;

METHOD Stop
_Speed := 0;

METHOD Load
Loaded := TRUE;

METHOD Unload
Loaded := FALSE;

METHOD SetSpeed
VAR_INPUT
    NewSpeed : INT;
END_VAR
IF (NewSpeed >= 0) AND (NewSpeed <= 100) THEN
    _Speed := NewSpeed;
ELSE
    Fault := TRUE;
END_IF
```

---

# SFC Çözümleri

**Actions:**

Act_Idle
```pascal
Motor := FALSE;
```

Act_Loading
```pascal
Loader := TRUE;
```

Act_Running
```pascal
Motor := TRUE;
```

Act_Unloading
```pascal
Unloader := TRUE;
```

Act_Fault
```pascal
FaultLED := TRUE;
```

**Transitions:**
```pascal
Idle_Loading := ProductDetected;
Loading_Running := Loaded = TRUE;
Running_Unloading := DestinationReached;
Unloading_Idle := Loaded = FALSE;
Any_Fault := FaultDetected;
```

# TwinCAT PLC – Tam Eğitim Paketi  
## Egzersizler + Çözümler + Açıklamalı Şemalar + SFC Diyagramları  
### (C# Karşılaştırmalı Öğrenme Seti)

---

# 1. METHOD – Temel Seviye Egzersizler ve Çözümleri

## Egzersiz 1 – Increment Method
### Görev:
- FB_Counter içinde Count değişkeni ve Increment() methodu yaz.

### Çözüm:
```pascal
FUNCTION_BLOCK FB_Counter
VAR
    Count : INT := 0;
END_VAR

METHOD Increment : BOOL
Count := Count + 1;
Increment := TRUE;
```

### Açıklama Şeması:
```
+---------------------+
|     FB_Counter      |
|---------------------|
| Count : INT         |
|---------------------|
| Increment()         | → Count = Count + 1
+---------------------+
```

---

## Egzersiz 2 – Parametre Alan Method
### Görev:
- SetValue(value) methodu ile Count güncelle.

### Çözüm:
```pascal
METHOD SetValue : BOOL
VAR_INPUT
    value : INT;
END_VAR

IF value >= 0 THEN
    Count := value;
    SetValue := TRUE;
ELSE
    SetValue := FALSE;
END_IF
```

---

## Egzersiz 3 – Method Kullanımı
### Çözüm:
```pascal
PROGRAM MAIN
VAR
    Cnt : FB_Counter;
    Result : INT;
END_VAR

Cnt.Increment();
Cnt.Increment();
Cnt.Increment();
Cnt.SetValue(10);
Result := Cnt.Count;
```

---

# 2. PROPERTY – Kapsülleme Mantığı

## Egzersiz 4 – Speed Property
### Çözüm:
```pascal
FUNCTION_BLOCK FB_Motor
VAR
    _Speed : INT;
END_VAR

PROPERTY Speed : INT
GET
    Speed := _Speed;
END_GET

SET
IF (Value >= 0) AND (Value <= 3000) THEN
    _Speed := Value;
END_IF
END_SET
```

### Açıklamalı Şema:
```
+---------------------------+
|        FB_Motor           |
|---------------------------|
|  _Speed (private)         |
|---------------------------|
|  PROPERTY Speed           |
|   GET → return _Speed     |
|   SET → limit 0–3000      |
+---------------------------+
```

---

## Egzersiz 5 – IncreaseSpeed Method
### Çözüm:
```pascal
METHOD IncreaseSpeed
VAR_INPUT
    step : INT;
END_VAR

_Speed := _Speed + step;

IF _Speed > 3000 THEN
    _Speed := 3000;
END_IF
```

---

# 3. ACTION & TRANSITION – SFC Yapısı

## Egzersiz 6 – SFC Steps: Init → Heat → Ready

### Actions:
```pascal
(* Act_Init *)
Heater := FALSE;

(* Act_Heat *)
Heater := TRUE;

(* Act_Ready *)
LED := TRUE;
```

### SFC Diyagramı:
```
 [Init]
    |
    | StartCmd = TRUE
    v
 [Heat]
    |
    | Temp >= 80
    v
 [Ready]
    |
    | ResetCmd = TRUE
    v
 [Init]
```

---

## Egzersiz 7 – Transition Koşulları
### Çözüm:
```pascal
Transition_Init_Heat := StartCmd;
Transition_Heat_Ready := (Temp >= 80);
Transition_Ready_Init := ResetCmd;
```

---

# 4. İleri Seviye: Motor FB + SFC

## Egzersiz 8 – Motor Function Block

### Çözüm:
```pascal
FUNCTION_BLOCK FB_Motor
VAR
    _Speed : INT;
    Running : BOOL;
    Error : BOOL;
END_VAR

METHOD Start
Running := TRUE;
_Speed := 500;

METHOD Stop
Running := FALSE;
_Speed := 0;

METHOD SetSpeed
VAR_INPUT
    NewSpeed : INT;
END_VAR

IF (NewSpeed >= 0) AND (NewSpeed <= 3000) THEN
    _Speed := NewSpeed;
ELSE
    Error := TRUE;
END_IF
```

### Açıklama Şeması:
```
+--------------------------------+
|           FB_Motor             |
|--------------------------------|
| _Speed : INT                   |
| Running : BOOL                 |
| Error : BOOL                   |
|--------------------------------|
| Start()   → Speed = 500        |
| Stop()    → Speed = 0          |
| SetSpeed()                     |
+--------------------------------+
```

---

## Egzersiz 9 – Motor SFC

### Actions:
```pascal
Act_Stopped:   Motor.Stop();
Act_Starting:  Motor.Start();
Act_Running:   Motor.SetSpeed(1000);
Act_Stopping:  Motor.Stop();
```

### Transitions:
```pascal
Stopped_Starting := StartCmd;
Starting_Running := (Motor._Speed >= 500);
Running_Stopping := StopCmd;
Stopping_Stopped := (Motor._Speed = 0);
```

### SFC Diyagramı:
```
  [Stopped]
      |
      | StartCmd
      v
  [Starting]
      |
      | Speed >= 500
      v
  [Running]
      |
      | StopCmd
      v
  [Stopping]
      |
      | Speed = 0
      v
  [Stopped]
```

---

# 5. Uzman Seviye: Konveyör Sistemi

## Egzersiz 10 – FB_Conveyor

### Çözüm:
```pascal
FUNCTION_BLOCK FB_Conveyor
VAR
    _Speed : INT;
    Loaded : BOOL;
    Fault : BOOL;
END_VAR

PROPERTY Speed : INT
GET
    Speed := _Speed;
END_GET
SET
IF (Value >= 0) AND (Value <= 100) THEN
    _Speed := Value;
END_IF
END_SET

METHOD Start
_Speed := 50;

METHOD Stop
_Speed := 0;

METHOD Load
Loaded := TRUE;

METHOD Unload
Loaded := FALSE;

METHOD SetSpeed
VAR_INPUT
    NewSpeed : INT;
END_VAR
IF (NewSpeed >= 0) AND (NewSpeed <= 100) THEN
    _Speed := NewSpeed;
ELSE
    Fault := TRUE;
END_IF
```

---

## Konveyör SFC Diyagramı

```
     [Idle]
        |
        | ProductDetected
        v
     [Loading]
        |
        | Loaded = TRUE
        v
     [Running]
        |
        | DestinationReached
        v
     [Unloading]
        |
        | Loaded = FALSE
        v
     [Idle]

 ANY STATE → [Fault] (FaultDetected = TRUE)
```

---





