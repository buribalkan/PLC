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


