# TwinCAT OOP MASTERCLASS  
### Professional PLC Object-Oriented Programming Guide  
TwinCAT 3 · IEC 61131‑3 · OOP Architecture

---

# 1. OOP Nedir? TwinCAT'te Neden Kullanılır?

Nesne yönelimli programlama (OOP), yazılımı **nesnelere (objelere)** böler.  
TwinCAT 3, IEC 61131‑3 üzerine tam OOP desteği sunarak:

- Modüler mimari sağlar  
- Bakım ve genişletilebilirlik kolaylığı sunar  
- Kod tekrarını azaltır  
- Büyük projelerde düzen ve kontrol oluşturur  
- Endüstriyel cihaz sürücüleri için standartlaştırma sağlar  

TwinCAT’te OOP → PLC dünyasında modern ve güçlü bir mimari sağlar.

---

# 2. Temel OOP Yapıları

## 2.1 Function Block = Class (Sınıf)

```pascal
FUNCTION_BLOCK FB_Motor
VAR
    rSpeed : REAL;
    bEnabled : BOOL;
END_VAR
```

---

## 2.2 Instance = Object (Nesne)

```pascal
VAR
    motor1 : FB_Motor;
    motor2 : FB_Motor;
END_VAR
```

---

## 2.3 Methods (Davranışlar)

```pascal
METHOD Enable
bEnabled := TRUE;
```

Çağrı:

```pascal
motor1.Enable();
```

---

## 2.4 Properties (Özellikler)

```pascal
PROPERTY Speed : REAL
GET
    Speed := rSpeed;
END_GET
SET
    rSpeed := Speed;
END_SET
```

---

## 2.5 Encapsulation (Kapsülleme)

| Erişim | Açıklama |
|--------|----------|
| `VAR` | Private |
| `VAR PUBLIC` | Global erişim |
| `VAR PROTECTED` | Sadece inherited FB erişebilir |

---

# 3. Inheritance (Kalıtım)

```pascal
FUNCTION_BLOCK FB_Motor
VAR
    rSpeed : REAL;
END_VAR

FUNCTION_BLOCK FB_Servo EXTENDS FB_Motor
VAR
    rPosition : REAL;
END_VAR
```

Avantajları:

- Kod tekrarını azaltır  
- Ortak davranışları merkezileştirir  
- Yeni FB'ler daha hızlı geliştirilir  

---

# 4. Polymorphism (Çok Biçimlilik)

TwinCAT polymorphism’i **interface tabanlı** kullanır.

---

# 5. Interface Kullanımı

Interface bir FB'nin hangi metodları *zorunlu* olarak içermesi gerektiğini belirler.

### Interface tanımı:

```pascal
INTERFACE IDevice
METHOD Init : BOOL
METHOD Read : BOOL
METHOD Write : BOOL
END_INTERFACE
```

### Uygulayan FB:

```pascal
FUNCTION_BLOCK FB_Sensor IMPLEMENTS IDevice
METHOD Init : BOOL
    RETURN TRUE;

METHOD Read : BOOL
    RETURN TRUE;

METHOD Write : BOOL
    RETURN TRUE;
```

---

# 6. Abstract Yapı (TwinCAT Yaklaşımı)

TwinCAT’te klasik abstract class yoktur.  
Bunun yerine:

- Interface → zorunlu sözleşme  
- Base class → ortak davranış  
- Derived class → özel davranış  

kullanılır.

---

# 7. Constructor / Destructor Sistemi (PLC Versiyonu)

TwinCAT’te klasik constructor yoktur.

| OOP Kavramı | TwinCAT Karşılığı |
|-------------|-------------------|
| Constructor | `FB_init` |
| Re‑constructor | `FB_reinit` |
| Destructor | `FB_exit` |

TwinCAT yaşam döngüsü sistem tarafından otomatik yönetilir.

---

# 8. İleri Seviye OOP

## 8.1 Method Overloading
Aynı isim, farklı parametre listesi ile kullanılabilir.

## 8.2 Property Overloading
Desteklenir.

## 8.3 Polymorphic Call (Interface Üzerinden)

```pascal
VAR
    dev : IDevice;
    sensor : FB_Sensor;
END_VAR

dev := sensor;
dev.Read();
```

---

# 9. Gerçek Dünya Uygulaması  
### Endüstriyel Cihaz Sürücü Mimarisi

```
IDevice
   ↑
DeviceBase
   ↑
 ┌───────────────┬────────────────┐
FB_ModbusDevice  FB_ADSDevice   FB_TempSensor
```

### Interface:

```pascal
INTERFACE IDevice
METHOD Connect : BOOL
METHOD Disconnect : BOOL
METHOD Read : BOOL
END_INTERFACE
```

### Base class:

```pascal
FUNCTION_BLOCK FB_DeviceBase IMPLEMENTS IDevice
VAR
    bConnected : BOOL;
END_VAR

METHOD Connect : BOOL
    bConnected := TRUE;

METHOD Disconnect : BOOL
    bConnected := FALSE;

METHOD Read : BOOL
    RETURN FALSE;
```

### Derived Modbus Device:

```pascal
FUNCTION_BLOCK FB_ModbusDevice EXTENDS FB_DeviceBase
VAR
    hModbus : UDINT;
END_VAR

METHOD Read : BOOL
    // Modbus read
    RETURN TRUE;
```

### Kullanım:

```pascal
dev := modbus;
dev.Connect();
dev.Read();
dev.Disconnect();
```

---

# 10. TwinCAT OOP Mimarisi İçin En İyi Uygulamalar

✔ Her cihaz bir interface implement etsin  
✔ Base class ortak davranışları içersin  
✔ Derived FB özel fonksiyon eklesin  
✔ Yaşam döngüsü FB_init/FB_exit ile yönetilsin  
✔ Properties ile veri gizleme sağlansın  
✔ Modüler tasarım prensibi uygulanmalı  

---

# 11. TwinCAT Universal OOP Template

```pascal
INTERFACE IModule
METHOD Init : BOOL
METHOD Execute : BOOL
METHOD Shutdown : BOOL
END_INTERFACE
```

```pascal
FUNCTION_BLOCK FB_ModuleBase IMPLEMENTS IModule
METHOD Init : BOOL
METHOD Execute : BOOL
METHOD Shutdown : BOOL
```

```pascal
FUNCTION_BLOCK FB_Conveyor EXTENDS FB_ModuleBase
METHOD Execute : BOOL
    // Logic
```

---

# 12. Özet

TwinCAT OOP:

- Modüler  
- Güçlü  
- Endüstriyel ölçekte genişleyebilir  
- Profesyonel PLC yazılım mimarisi sağlar  

Bu masterclass, TwinCAT OOP tasarımının tüm temellerini, mimarisini ve gerçek dünya örneklerini kapsar.

---


