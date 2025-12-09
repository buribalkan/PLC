# TwinCAT ENUMERATIONS MASTERCLASS  
_Tam Kapsamlı Profesyonel Eğitim Dokümanı (Markdown Formatında)_

---

## 1. ENUMERATION NEDİR?

TwinCAT’te **enumeration**, belirli sayıda sabit değeri isimlendirerek bir araya getiren kullanıcı tanımlı bir veri tipidir.  
Amaç:

- Sabit değerleri daha okunabilir ve güvenli hâle getirmek
- Durum makinelerinde state yönetimini kolaylaştırmak
- Magic-number (örn: 0, 1, 2…) kullanımını ortadan kaldırmak
- Type-safe kodlama sağlamak

Enum bir **DUT (Data Unit Type)** olarak oluşturulur.

---

## 2. ENUMERATION SÖZDİZİMİ

```pascal
{attribute 'strict'}
TYPE <EnumName> :
(
    <ComponentA>,
    <ComponentB>,
    <ComponentC> := <ExplicitValue>
) <BaseType> := <DefaultInit>;
END_TYPE
```

### Açıklamalar:

| Alan | Açıklama |
|------|----------|
| `<EnumName>` | Enum veri tipi adı |
| `<Component>` | Enum bileşenleri |
| `<ExplicitValue>` | İstenirse elle verilen başlangıç değeri |
| `<BaseType>` | Opsiyonel (INT, BYTE, DWORD, LWORD vb.) |
| `<DefaultInit>` | Tüm enum değişkenleri için başlangıç değeri |
| `strict` attribute | Otomatik eklenir; hatalı kullanımları engeller |

---

## 3. ENUM DEĞERLERİ NASIL VERİLİR?

### ✔ Otomatik değer atama  
TwinCAT varsayılan olarak 0'dan başlar:

```pascal
eRed = 0,
eYellow = 1,
eGreen = 2
```

### ✔ Manuel değer atama

```pascal
eRed := 5,
eYellow,        // = 6
eGreen := 10,
eBlue           // = 11
```

TwinCAT otomatik olarak devam eden sayıları hesaplar.

---

## 4. BASE DATA TYPE (Temel Veri Tipi) — Genişletme

Varsayılan: **INT**

Ama özel base tipi atanabilir:

```pascal
TYPE E_Color :
(
    eWhite  := 16#FFFFFF,
    eYellow := 16#FFFF00,
    eGreen  := 16#00FF00,
    eBlue   := 16#0000FF,
    eBlack  := 16#000000
) DWORD := eBlack;
END_TYPE
```

Bu kullanım özellikle:

- Bit maskeleri  
- Renk kodları  
- Haberleşme protokol değerleri  

için idealdir.

---

## 5. STRICT MODE — DERLEYİCİ DAVRANIŞI

TwinCAT 3.1 Build 4026’dan itibaren **tüm enum’lar strict attribute ile gelir**.

Strict devredeyken:

### ❌ Matematiksel işlem YASAK
```pascal
eColor := eColor + 1; // Hata
```

### ❌ Enum FOR döngüsünde sayaç olamaz
```pascal
FOR eState := 0 TO 5 DO // Hata
```

### ❌ Farklı data type atanamaz
```pascal
eColor := 5; // Hata
```

### ✔ Amaç:
Enum değişkenlerinin yanlışlıkla “geçersiz” değer almasını engellemek.

---

## 6. ENUM INITIALIZATION (Başlangıç Değerleri)

### ✔ ENUM içinde default initialization:

```pascal
) INT := eYellow;
```

Tüm değişkenler eYellow ile başlar.

### Eğer default belirtilmezse:

1) Eğer enum içinde **değeri 0 olan** bir bileşen varsa → o seçilir  
2) Yoksa → listenin ilk bileşeni seçilir  

---

### Örnek A — 0 değeri olan bileşen önceliklidir:

```pascal
TYPE E_SampleA :
(
    e1 := 2,
    e2 := 0,
    e3
);
END_TYPE

VAR eVal : E_SampleA; END_VAR
// eVal = e2
```

### Örnek B — 0 yoksa ilk bileşen seçilir:

```pascal
TYPE E_SampleB :
(
    e1 := 3,
    e2 := 1,
    e3
);
END_TYPE

VAR eVal : E_SampleB; END_VAR
// eVal = e1
```

---

## 7. ENUM KULLANIMI

```pascal
PROGRAM MAIN
VAR
    eColorCar  : E_Color;
    eColorTaxi : E_Color := E_Color.eYellow;
END_VAR
```

Erişim:

```pascal
IF eColorCar = E_Color.eBlue THEN
    ...
END_IF
```

Enum bileşenleri globaldir, ancak **daima qualified erişim** kullanılmalıdır:

✔ Doğru:
```pascal
eCar := E_Color.eBlue;
```

❌ Yanlış:
```pascal
eCar := eBlue;
```

Bu nedenle farklı enum’larda aynı bileşen adı kullanılabilir.

---

## 8. ENUM İLE SWITCH/CASE KULLANIMI (Tavsiye edilen yöntem)

```pascal
CASE eState OF
    E_State.eIdle:
        ...
    E_State.eInit:
        ...
    E_State.eRun:
        ...
    E_State.eError:
        ...
END_CASE
```

Enum → durum makineleri için en uygun veri modelidir.

---

## 9. GERÇEK DÜNYA ENUM TASARIM ÖRNEKLERİ

---

### 📌 9.1 Machine State (Durum Makinesi)

```pascal
TYPE E_MachineState :
(
    eStopped,
    eStarting,
    eRunning,
    eStopping,
    eError
) INT := eStopped;
END_TYPE
```

---

### 📌 9.2 Error Codes

```pascal
TYPE E_ErrorCode :
(
    eNone := 0,
    eOverload := 100,
    eCommunicationLost := 200,
    eTemperatureHigh := 300
) UINT := eNone;
END_TYPE
```

---

### 📌 9.3 Operator Modes

```pascal
TYPE E_OpMode :
(
    eManual,
    eSemiAuto,
    eAuto
);
END_TYPE
```

---

## 10. ENUM + FUNCTION BLOCK — Endüstri Seviyesi Örnek

```pascal
FUNCTION_BLOCK FB_Machine
VAR_INPUT
    eCmd : E_Command;
END_VAR

VAR
    eState : E_MachineState := E_MachineState.eStopped;
END_VAR

CASE eState OF

    eStopped:
        IF eCmd = eStart THEN
            eState := eStarting;
        END_IF

    eStarting:
        eState := eRunning;

    eRunning:
        IF eCmd = eStop THEN
            eState := eStopping;
        END_IF

    eStopping:
        eState := eStopped;

    eError:
        // error handling

END_CASE
```

Enum burada FB’nin **state engine** yapısını oluşturur.

---

## 11. ENUM MASTERCLASS — ÖZET

| Konu | İçerik |
|------|--------|
| Enum Temelleri | ✔ |
| strict attribute | ✔ |
| qualified_only | ✔ |
| Base Data Type | ✔ |
| Initialization Kuralları | ✔ |
| Matematiksel işlem neden yasak? | ✔ |
| SWITCH/CASE kullanımına yönlendirme | ✔ |
| Gerçek dünya enum pattern’ları | ✔ |

---



