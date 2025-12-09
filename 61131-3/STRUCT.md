# 📘 TwinCAT – STRUCT (Yapılar) TAM KAPSAMLI MASTERCLASS  
### *Gerçek Dünya İçin Profesyonel .md Eğitim Dokümanı*  

---

# 🏭 1. Giriş – STRUCT Nedir?

TwinCAT’te **STRUCT**, birden fazla değişkeni tek bir mantıksal veri paketi hâline getirmenizi sağlayan *kullanıcı tanımlı veri tipidir*.  

STRUCT, özellikle:

- Sensör veri paketleme  
- HMI–PLC arasında veri gönderme  
- Reçete yönetimi  
- Parametre setleri  
- Log kayıtları  
- Konfigürasyon dosyaları  
- Motion & Axis parametreleri  

gibi tüm profesyonel TwinCAT projelerinin temelidir.

---

# 📦 2. STRUCT Temel Sözdizimi

```pascal
TYPE <StructureName> :
STRUCT
    <Variable declarations>
END_STRUCT
END_TYPE
```

STRUCT bir **DUT (Data Unit Type)** olarak oluşturulur.

---

# 🧱 3. Basit Bir STRUCT Örneği

```pascal
TYPE ST_Point :
STRUCT
    X : LREAL;
    Y : LREAL;
END_STRUCT
END_TYPE
```

Kullanımı:

```pascal
VAR
    ptA : ST_Point := (X := 10.5, Y := 5.3);
END_VAR
```

Erişim:

```pascal
ptA.X := 20;
```

---

# 🧱 4. İç İçe STRUCT – Nested Structures

```pascal
TYPE ST_Position :
STRUCT
    X : LREAL;
    Y : LREAL;
    Z : LREAL;
END_STRUCT
END_TYPE

TYPE ST_RobotPose :
STRUCT
    Position : ST_Position;
    Angle    : LREAL;
END_STRUCT
END_TYPE
```

Kullanım:

```pascal
stPose.Position.X := 50.0;
```

---

# 🎨 5. Initialization (Başlangıç Değerleri)

```pascal
TYPE ST_Limits :
STRUCT
    Min : LREAL := 0.0;
    Max : LREAL := 100.0;
END_STRUCT
END_TYPE
```

STRUCT içinde **değişken** ile initialization yapılamaz (örn. Min := SomeVar → yasak).

---

# 🌐 6. STRUCT Alignment (8-byte Alignment)

TwinCAT 3’te:

- LREAL, LWORD gibi 8-byte tipler hizalama gerektirir.
- STRUCT diğer sistemlerle veri alışverişinde alignment kritik olabilir.

Dış sistemle veri alışverişi varsa *struct packing* dikkatle incelenmelidir.

---

# 🧬 7. STRUCT Genişletme – EXTENDS

Bir STRUCT başka bir STRUCT’tan türetilebilir.

### Örnek – ST_POLYGONLINE → ST_PENTAGON

```pascal
TYPE ST_POLYGONLINE :
STRUCT
    aStart : ARRAY[1..2] OF INT;
    aPoint1 : ARRAY[1..2] OF INT;
    aPoint2 : ARRAY[1..2] OF INT;
    aPoint3 : ARRAY[1..2] OF INT;
    aPoint4 : ARRAY[1..2] OF INT;
    aEnd : ARRAY[1..2] OF INT;
END_STRUCT
END_TYPE

TYPE ST_PENTAGON EXTENDS ST_POLYGONLINE :
STRUCT
    aPoint5 : ARRAY[1..2] OF INT;
END_STRUCT
END_TYPE
```

Kullanım:

```pascal
stPentagon.aPoint5 := [5,5];
```

---

# 🧩 8. STRUCT ile Veri Okuma (Access Components)

Erişim:

```pascal
<variable>.<component>
```

Örnek:

```pascal
nPoint := stPolygon.aPoint1[1];
```

---

# 🧱 9. STRUCT + ARRAY (Array of Struct)

Bu en çok kullanılan TwinCAT veri modelidir.

```pascal
TYPE ST_Sensor :
STRUCT
    Value : LREAL;
    Status : BOOL;
END_STRUCT
END_TYPE

VAR
    aSensors : ARRAY[1..8] OF ST_Sensor;
END_VAR
```

Kullanım:

```pascal
aSensors[3].Value := 12.5;
```

---

# 🧱 10. STRUCT of Array

```pascal
TYPE ST_FilterBuffer :
STRUCT
    Values : ARRAY[1..10] OF LREAL;
END_STRUCT
END_TYPE
```

---

# ⚠ 11. BIT Alanları İçeren STRUCT

STRUCT içinde BIT kullanılarak bit alanları oluşturulabilir.

### Örnek:

```pascal
TYPE ST_ControlBits :
STRUCT
    bitOperationEnabled : BIT;
    bitSwitchOnActive   : BIT;
    bitError            : BIT;
    bitWarning          : BIT;
END_STRUCT
END_TYPE
```

Kullanım:

```pascal
stControl.bitWarning := TRUE;
```

⚠ ARRAY of BIT veya POINTER/REFERENCE TO BIT **yasaktır**.

---

# 🧠 12. REAL WORLD – STRUCT Tasarım Desenleri (Design Patterns)

Aşağıdaki örnekler profesyonel TwinCAT projelerinde %100 kullanılır.

---

## 📌 12.1 Sensor Data Packet Structure

```pascal
TYPE ST_SensorPacket :
STRUCT
    RawValue       : LREAL;
    FilteredValue  : LREAL;
    Timestamp      : DT;
    Valid          : BOOL;
END_STRUCT
END_TYPE
```

---

## 📌 12.2 Motion Axis Configuration Structure

```pascal
TYPE ST_AxisConfig :
STRUCT
    MaxVel : LREAL;
    MaxAcc : LREAL;
    HomePosition : LREAL;
    Inverted : BOOL;
END_STRUCT
END_TYPE
```

---

## 📌 12.3 Error Handling Packet

```pascal
TYPE ST_ErrorInfo :
STRUCT
    ErrorCode  : UDINT;
    Message    : STRING(100);
    Timestamp  : DT;
    Active     : BOOL;
END_STRUCT
END_TYPE
```

---

## 📌 12.4 HMI → PLC Data Exchange Structure

```pascal
TYPE ST_HMI_Command :
STRUCT
    CmdStart : BOOL;
    CmdStop  : BOOL;
    Target   : LREAL;
END_STRUCT
END_TYPE
```

---

## 📌 12.5 Configuration Structure (FB init input)

```pascal
TYPE ST_FB_Config :
STRUCT
    Enabled     : BOOL;
    ScalingMin  : LREAL;
    ScalingMax  : LREAL;
    AlarmHigh   : LREAL;
END_STRUCT
END_TYPE
```

---

# 🎯 13. STRUCT Kullanırken “Best Practices”

| Öneri | Açıklama |
|------|----------|
| ✔ STRUCT isimleri ST_ prefix ile başlatılır | Beckhoff standardı |
| ✔ FB config değerlerini STRUCT ile aktar | Temiz API |
| ✔ HMI komutlarını tek STRUCT içinde grupla | Tag sayısı azalır |
| ✔ ARRAY of STRUCT kullanarak veri paralelliği sağla | Çoklu sensör sistemi için ideal |
| ✔ Struct’ları çok büyük yapma | Bellek ve alignment maliyeti artar |
| ❌ BIT tabanlı pointer/reference kullanma | Yasaktır |

---

# 🚀 14. Uçtan Uca Gerçek Dünya Örneği – Robot Pose Data Packet

```pascal
TYPE ST_RobotPose :
STRUCT
    PosX : LREAL;
    PosY : LREAL;
    PosZ : LREAL;
    AngleA : LREAL;
    AngleB : LREAL;
    AngleC : LREAL;
    Timestamp : DT;
END_STRUCT
END_TYPE

TYPE ST_RobotTelemetry :
STRUCT
    Pose     : ST_RobotPose;
    Speed    : LREAL;
    Load     : LREAL;
    Error    : ST_ErrorInfo;
END_STRUCT
END_TYPE
```

Bu paket:

- Robot kontrolü  
- Telemetry logging  
- Remote monitoring  

için endüstride standarttır.

---

# 🧪 15. Kullanım Örneği

```pascal
PROGRAM MAIN
VAR
    Telemetry : ST_RobotTelemetry;
END_VAR

Telemetry.Pose.PosX := 120.5;
Telemetry.Error.ErrorCode := 102;
Telemetry.Speed := 55.3;
```

---

# 🏆 16. STRUCT Masterclass Sonuç Özeti

| Konu | Açıklama |
|------|----------|
| STRUCT temelleri | ✔ |
| Nested struct | ✔ |
| EXTENDS ile genişletme | ✔ |
| ARRAY of STRUCT | ✔ |
| STRUCT of ARRAY | ✔ |
| BIT field struct | ✔ |
| Alignment | ✔ |
| Gerçek dünya örnekleri | ✔ |
| Design patterns | ✔ |

Bu eğitim ile TwinCAT projelerinde **uzman seviyesi STRUCT tasarımı** oluşturabilirsin.

---



