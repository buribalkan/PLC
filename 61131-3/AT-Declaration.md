
# AT-Declaration — Ultra Masterclass Level 4  
**TwinCAT 3 Special Edition – EtherCAT, CoE, NC/PLC Shared Memory, Safety & Compiler Deep Dive**

---

## 1. Giriş  
`AT` deklarasyonu, TwinCAT’in **doğrudan donanıma bağlı değişken eşlemesi**, **işlenmiş process image manipülasyonu**, **memory overlay**, **driver-level mapping** gibi ileri konularda kullanılan en kritik mekanizmalarından biridir.

Bu Level‑4 Masterclass dokümanı yalnızca AT kullanımını anlatmaz;  
aynı zamanda **EtherCAT PDO/CoE mapping**, **NC/PLC shared memory**, **retargeting**, **boot-time image generation**, **derleyici memory alignment kuralları**, **safety farkları**, **multi-task access riskleri** gibi tüm ileri seviye konuları kapsar.

---

# 2. AT-Declaration Temel Sözdizimi  

```iecst
<identifier> AT <address> : <data type>;
```

Adres formatı:

```
%<Area><Size?> <Position>
```

| Bölüm | Açıklama |
|------|----------|
| `%I` | Input area (process image → PLC) |
| `%Q` | Output area (PLC → process image) |
| `%M` | Memory/Flags (non-I/O mapped) |
| `X/B/W/D` | Bit/Byte/Word/DWord |
| `*` | Otomatik adresleme (önerilir) |

Örnekler:

```iecst
bSensor AT %IX7.5 : BOOL;
wInput  AT %IW10  : WORD;
dwFlag  AT %MD48  : DWORD;
```

---

# 3. AT Kullanımında Derin Kurallar

## 3.1 Boolean için AT kullanırken alignment tuzağı
TwinCAT, bir BOOL değişkenini yalnızca **tek bit adresi verilmişse** bit olarak bağlar:

```iecst
b1 AT %IX0.0 : BOOL;   // 1 bit
```

Bit adres verilmezse:

```iecst
b2 AT %QB0 : BOOL;     // 1 BYTE KAPLANIR → QX0.0–QX0.7 etkilenir!
```

Bu, EtherCAT PDO mapping sırasında **çok kritik hatalara** yol açar.

---

# 4. Bellek Yerleşimi (Memory Layout) — Target CPU Bağımlılığı

TwinCAT çoğu durumda:

- **x86/x64 Windows** → Little endian
- **ARM (CX7000 vb.)** → Little endian
- Alignment CPU mimarisi tarafından belirlenir.

Aşağıdaki yapı:

```iecst
TYPE ST_Test :
STRUCT
    b1 : BOOL;   // 1 byte
    w1 : WORD;   // 2 byte alignment
    ud : UDINT;  // 4 byte alignment
END_STRUCT
END_TYPE
```

Gerçekte:

```
Offset  Size  Field
0       1     b1
1       1     PAD
2       2     w1
4       4     ud
```

---

# 5. AT + STRUCT — Memory Overlay

Bir ADC buffer’ı process image’e overlay etmek:

```iecst
TYPE TAdsInput :
STRUCT
    ai0 : INT;
    ai1 : INT;
    ai2 : INT;
END_STRUCT
END_TYPE

adsInputs AT %IB0 : TAdsInput;
```

EtherCAT slave PDO mapping’i ile tam uyumlu olmak zorundadır.

**Alignment uyuşmazlığı → tamamen bozuk veri okuma** üretir.

---

# 6. AT + Function Block Instances (Static Memory)

Bir FB’nin içindeki field’a AT bağlanırsa **bütün instancelar aynı fiziksel hafızayı kullanır**:

```iecst
FUNCTION_BLOCK FB_Test
VAR
    x1 AT %MX10.0 : BOOL;
END_VAR
```

Kaç tane örnek üretirseniz üretin:

```
FB_Test() → hepsi M10.0 adresini paylaşır.
```

Bu mekanizma **static variable** davranışı üretir.

---

# 7. TwinCAT 3 Derleyici Davranışı (4020–4026)

| Durum | Sonuç |
|-------|-------|
| STRUCT için AT | Memory overlay, compiler alignment korur |
| FB instance için AT | Tüm instance’lar shared memory |
| POINTER ile AT | Yasak (derleyici uyarı verir) |
| RETAIN/PERSISTENT + AT | Yasak (kritik uyarı) |
| AT + VAR_IN_OUT | Son derece tehlikeli, önerilmez |

---

# 8. EtherCAT PDO / CoE Mapping ve AT İlişkisi

## 8.1 EtherCAT PDO offsetleri PLC’de nasıl okunur?

EL1xxx (digital input):

```
%IX0.0 … %IX1.7  → EtherCAT input PDOs
```

EL3xxx (analog input):

```
%IW20 → Channel 1
%IW22 → Channel 2
%IW24 → Channel 3
```

Örnek:

```iecst
ai0 AT %IW20 : INT;
ai1 AT %IW22 : INT;
ai2 AT %IW24 : INT;
```

## 8.2 CoE Explicit PDO Assignment

Eğer slave PDO düzeni değişirse:

- PLC’deki AT adresleri **tamamen bozulur**.
- Process image yeniden oluşturulmalıdır.

Bu nedenle **%I\*** ile `automatic addressing` tavsiye edilir.

---

# 9. Safety (TwinSAFE) ile AT Kullanımı

TwinSAFE Logic (EL6910/EL6900):

- Safety değişkenleri **AT ile bağlanamaz** (çünkü Safe-Process Image ayrıdır).
- PLC → SafeComm → TwinSAFE Logic → Safe I/O zinciri vardır.

Bu nedenle:

```
SAFExxx vars = NOT m12.0  // Mümkün değil
```

Safety I/O’nun PLC ile kullanımı yalnızca **FSoE** üzerinden yapılabilir.

---

# 10. NC/PLC Shared Memory — AT ile İleri Seviye Kullanım

NC ve PLC arasında exchange area:

```
NCToPLC AT %MB100 : ARRAY[0..15] OF BYTE;
PLCToNC AT %MB200 : ARRAY[0..15] OF BYTE;
```

Bu alan:

- Task-cycle ile güncellenir,
- Gerçek zamanlı davranış gerektirir,
- Multi-task access durumunda **race condition** oluşturabilir.

---

# 11. Multi‑Task Access Riskleri  

Aynı AT adresine iki farklı PLC task yazıyorsa:

- Veri yırtılması (data tearing)
- Non-atomic write hataları
- EtherCAT Watchdog tetiklemesi

**Çözüm:** AT adresleri yalnızca **bir yazıcı (single writer)** task tarafından yönetilmelidir.

---

# 12. Derleyici Uyarı / Hata Tablosu

| Kod | Açıklama | Sebep |
|-----|----------|--------|
| **C0500** | Address overlaps | Aynı adrese birden çok AT bildirimi |
| **C0501** | Invalid address | PDO offset uyuşmazlığı |
| **C0511** | RETAIN + AT yasak | Retain handler çalışamaz |
| **C0520** | STRUCT alignment violation | Veri hedef mimariye uymuyor |
| **C0650** | AT used with POINTER | Desteklenmez |
| **C0300** | Bool assigned to byte without bit index | Alignment kaybı |

---

# 13. İleri Seviye Diyagramlar

## 13.1 Process Image UML Diyagramı

```
+--------------------+
|   EtherCAT Slave   |
| PDO Out: 0x6000    |
+--------------------+
           |
           | mapped to
           v
+-------------------------------+
|  TwinCAT Process Image (I/O) |
|  %I  %Q  %M                   |
+-------------------------------+
           |
           v
+-------------------------------+
|        PLC Runtime            |
|   AT variables bind here      |
+-------------------------------+
```

---

# 14. AT ile GENERIC / ANY Kullanımı

AT değişkenleri **compile-time fixed** olduğundan:

- `VAR_GENERIC` ile birlikte **kullanılamaz**.
- `ANY` parametrelerine gönderilemez (pointer conflict).

Örnek hatalı kod:

```iecst
fb.DoSomething(anyIn := ai0); // ai0 AT %IW20 → HATA
```

---

# 15. Doğru / Yanlış Kullanım Özet Tablosu

| Kullanım | Durum |
|----------|--------|
| `%I*` flexible addressing | ✔ En güvenli yöntem |
| `%IX0.0` bit-level address | ✔ Dijital I/O için |
| STRUCT overlay | ✔ Eğer alignment doğruysa |
| FB internal AT | ⚠ Tüm instancelar memory paylaşıyor |
| RETAIN + AT | ❌ Yasak |
| PERSISTENT + AT | ❌ Yasak |
| POINTER + AT | ❌ Desteklenmez |
| Safety I/O + AT | ❌ Mümkün değil |

---

# 16. Büyük Örnek: EtherCAT + NC + PLC AT Mapping

```iecst
// EtherCAT AI
ai AT %IW20 : ARRAY[0..3] OF INT;

// EtherCAT DI
di AT %IX0.0 : ARRAY[0..15] OF BOOL;

// NC Shared Memory
ncIn  AT %MB200 : ARRAY[0..7] OF BYTE;
ncOut AT %MB210 : ARRAY[0..7] OF BYTE;
```

---

# 17. Sonuç

Bu Level‑4 masterclass:

- AT deklarasyonunun tüm gelişmiş mekaniklerini,
- EtherCAT PDO / CoE mapping,
- NC/PLC shared memory,
- Safety sınırlarını,
- Derleyici uyarılarını,
- Alignment kurallarını

tam kapsamıyla açıklamaktadır.

---

