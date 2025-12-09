# TwinCAT UNION MASTERCLASS  
_Tam Kapsamlı Profesyonel Eğitim Dokümanı (Markdown Formatında)_

---

## 📘 1. UNION Nedir?

TwinCAT’te **UNION**, birden fazla değişkeni **aynı bellek alanını paylaşacak şekilde** tanımlayan özel bir veri yapısıdır.

Bu şu anlama gelir:

- UNION içindeki tüm bileşenler **aynı offset’ten başlar**  
- Bir bileşene yapılan atama **diğerlerini de etkiler**  
- Aynı veri alanına farklı türlerden erişmek mümkündür  

UNION, özellikle:

- Ham bellek manipülasyonu  
- Bit/byte seviyesinde veri çözümleme  
- Protokol parsing  
- Sensör/fieldbus veri paketlerini yeniden yorumlama  
- Yüksek performanslı bellek dönüşümleri  

için endüstride çok kullanılır.

UNION bir **DUT** dosyasında tanımlanır.

---

## 🧱 2. UNION Sözdizimi

```pascal
TYPE <UnionName> :
UNION
    <Variable declarations>
END_UNION
END_TYPE
```

---

## 🧩 3. UNION’un Temel Mantığı: “Tek Bellek, Birden Çok Yorum”

Bir UNION’u şöyle düşünebilirsin:

> Aynı bellek alanına farklı pencerelerden bakmak.

Bu, özellikle:

- Byte → UINT → WORD → LREAL dönüştürme  
- Bit bazlı analiz  
- Fieldbus veri paketlerini çözme  
- Memory mapping  

gibi alanlarda çok güçlü bir araçtır.

---

## 🧪 4. Örnek 1 — Basit UNION

### UNION Deklarasyonu

```pascal
TYPE U_Name :
UNION
    fA : LREAL;
    nB : LINT;
    nC : WORD;
END_UNION
END_TYPE
```

### Kullanım

```pascal
VAR
    uName : U_Name;
END_VAR

uName.fA := 1;
```

### Sonuç

| Alan | Değer |
|------|--------|
| fA | 1 |
| nB | 16#3FF0000000000000 |
| nC | 0 |

Çünkü **fA**, LREAL formatında belleğe 1.0 değerini yazar.  
Diğer alanlar **o bellek alanını kendi türlerine göre yorumlar**.

---

## 🧠 Neden Böyle Oluyor?

LREAL değeri belleğe IEEE-754 formatında yazılır → Bu byte dizisi LINT olarak yorumlanınca büyük bir HEX sayı görünür.

WORD ise ilk 2 byte’ı alır → çoğunlukla 0 çıkar.

---

## 🧪 5. Örnek 2 — Bit/Byte Seviyesinde UNION

Bu örnek profesyonel projelerde yaygın kullanımdır.

Aşağıdaki UNION sayesinde:

- `UINT` sayı yazarsın  
- Aynı veri hem **2 BYTE array** olarak hem de  
- **2 byte içindeki bitlerin 8+8 BIT olarak** çözümlenmiş hâliyle görülebilir  

---

### 5.1 BIT Yapısı

```pascal
TYPE ST_Bits :
STRUCT
    bBit1 : BIT;
    bBit2 : BIT;
    bBit3 : BIT;
    bBit4 : BIT;
    bBit5 : BIT;
    bBit6 : BIT;
    bBit7 : BIT;
    bBit8 : BIT;
END_STRUCT
END_TYPE
```

---

### 5.2 UNION Deklarasyonu

```pascal
TYPE U_2Byte :
UNION
    nUINT  : UINT;
    a2Byte : ARRAY[1..2] OF BYTE;
    aBits  : ARRAY[1..2] OF ST_Bits;
END_UNION
END_TYPE
```

Bu tek UNION ile:

- UINT → sayı görünümü  
- a2Byte → ham byte görünümü  
- aBits → her byte içindeki bit görünümü  

aynı anda elde edilir.

---

### 5.3 Kullanım

```pascal
VAR
    u2Byte : U_2Byte;
END_VAR
```

---

### 📌 Assignment 1

```pascal
u2Byte.nUINT := 5;
```

**Binary:** `0000 0000 0000 0101`

Sonuç:

- a2Byte[1] = 5  
- a2Byte[2] = 0  
- aBits[1].bBit1 = TRUE  
- aBits[1].bBit3 = TRUE  

---

### 📌 Assignment 2

```pascal
u2Byte.nUINT := 255;
```

**Binary:** `0000 0000 1111 1111`

Sonuç:

- a2Byte[1] = 255  
- a2Byte[2] = 0  
- aBits[1] içindeki tüm bitler TRUE  

---

### 📌 Assignment 3

```pascal
u2Byte.nUINT := 256;
```

**Binary:** `0000 0001 0000 0000`

Sonuç:

- a2Byte[1] = 0  
- a2Byte[2] = 1  
- bit dağılımı **ikinci byte’ta** görünür  

---

## 🧬 6. UNION’un Avantajları

| Avantaj | Açıklama |
|--------|----------|
| **Ultra hızlı** | Bellek kopyalama yok, reinterpretation yapar |
| **Bit/Byte analizi** | Sensör ve fieldbus protokollerini çözmek için ideal |
| **Düşük seviye veri manipülasyonu** | C/C++ tarzı memory reinterpretation |
| **Tipler arası dönüşüm ihtiyacını azaltır** | UINT → BYTE → BIT otomatik olur |
| **Embedded kontrol projelerinde çok yaygın** | EtherCAT, Modbus, özel frame parsing |

---

## ⚠ 7. UNION Kullanırken Dikkat

- Bileşen boyutları farklıysa **büyük olan diğerlerini tamamen kaplar**  
- Yanlış türde okuma → yanlış veri yorumu doğurur  
- BIT tipi kullanırken ensure struct alignment  
- UNION içinde **pointer/reference kullanımı önerilmez**  
- Bellek içeriği debug modunda farklı görünebilir  

---

## 🏗 8. Gerçek Dünya Kullanım Örnekleri

### 📌 8.1 Protokol Paket Çözme (Frame Parsing)

Bir fieldbus mesajı şöyle çözümlenebilir:

```pascal
TYPE U_Frame :
UNION
    Raw      : ARRAY[0..7] OF BYTE;
    AsWORD   : ARRAY[0..3] OF WORD;
    AsUDINT  : UDINT;
END_UNION
END_TYPE
```

Bir frame geldiğinde:

- `Raw` → ham byte’lar  
- `AsWORD` → 2-byte register’lar  
- `AsUDINT` → 32-bit numara  

aynı anda görülebilir.

---

### 📌 8.2 Sensör Veri Dönüşümü

```pascal
TYPE U_AnalogRaw :
UNION
    RawValue : WORD;
    Bits     : ST_Bits;
END_UNION
END_TYPE
```

ADC ölçümünün bit-tabanlı çözümü kolaylaşır.

---

### 📌 8.3 Device Status Register Analizi

```pascal
TYPE U_Status :
UNION
    Reg : WORD;
    Bits : ST_DeviceBits;
END_UNION
END_TYPE
```

Register’ın her biti bir cihaz durumuna karşılık gelir.

---

## 🧪 9. Full Kullanım Senaryosu

```pascal
PROGRAM MAIN
VAR
    uData  : U_2Byte;
END_VAR

uData.nUINT := 37;

// Artık:
// - Byte görünümü
// - Bit görünümü
// - UINT görünümü
// aynı bellek üzerinde eşzamanlıdır
```

---

## 🏆 10. UNION Masterclass Özeti

| Konu | Durum |
|------|--------|
| UNION nedir? | ✔ |
| Memory reinterpretation | ✔ |
| Bit/byte seviye çözümlü örnekler | ✔ |
| Örnek 1: LREAL → LINT → WORD | ✔ |
| Örnek 2: UINT → BYTE → BIT | ✔ |
| Gerçek dünya kullanım alanları | ✔ |
| Frame parsing, status decoding | ✔ |
| Best practices | ✔ |

---

 
# TwinCAT UNION MASTERCLASS  
_Tam Kapsamlı Profesyonel Eğitim Dokümanı (Markdown Formatında)_

---

## 📘 1. UNION Nedir?

TwinCAT’te **UNION**, farklı veri tiplerinin **aynı hafıza alanını paylaşmasını sağlayan** özel bir veri yapısıdır.  
Bu şu anlama gelir:

- UNION içindeki tüm üyeler **aynı adresi** kullanır  
- Bir üyenin değeri değiştiğinde **diğerleri de değişmiş olur**  
- Veri yorumlama (type reinterpretation) yapılabilir  
- Özellikle **bit/byte seviyesinde veri çözümleme**, **haberleşme protokol çözme**, **low-level memory parsers** için idealdir  

UNION bir **DUT (Data Unit Type)** içinde tanımlanır.

---

## 🧱 2. UNION Sözdizimi

```pascal
TYPE <UnionName> :
UNION
    <Variable1 : Type1>
    <Variable2 : Type2>
    ...
END_UNION
END_TYPE
```

### Önemli Özellik:
✔ Tüm üyeler **offset 0’dan başlar**  
✔ Hepsi aynı belleği paylaşır  
✔ Birinin yazılması diğerlerinin yeniden yorumlanmasına neden olur

---

## 🧩 3. Temel UNION Örneği (Sample 1)

### Declaration

```pascal
TYPE U_Name :
UNION
    fA : LREAL;
    nB : LINT;
    nC : WORD;
END_UNION
END_TYPE
```

### Usage

```pascal
VAR
    uName : U_Name;
END_VAR

uName.fA := 1;
```

### Sonuç:

| Alan | Değer |
|------|--------|
| fA | 1 |
| nB | 16#3FF0000000000000 |
| nC | 0 |

### Neden böyle oldu?

- `fA` belleğe **LREAL olarak 1.0** değerini yazar  
- Aynı hafıza alanı `nB` tarafından **LINT olarak** okunur  
- Bellek düzeninden dolayı ortaya **IEEE-754 karşılık değeri** çıkar  

Bu UNION davranışının **temel prensibidir**.

---

## 🧬 4. UNION ile Çoklu Görüntüleme (Sample 2)

Bu örnek, UNION’ın en güçlü kullanım alanlarından birini gösterir:

**Bir BYTE/WORD/UINT değerinin hem bit düzeyinde hem byte düzeyinde hem de sayı olarak okunması**

---

### Önce Bit Struct Tanımı

```pascal
TYPE ST_Bits :
STRUCT
    bBit1 : BIT;
    bBit2 : BIT;
    bBit3 : BIT;
    bBit4 : BIT;
    bBit5 : BIT;
    bBit6 : BIT;
    bBit7 : BIT;
    bBit8 : BIT;
END_STRUCT
END_TYPE
```

### Şimdi UNION Tanımı

```pascal
TYPE U_2Byte :
UNION
    nUINT  : UINT;
    a2Byte : ARRAY[1..2] OF BYTE;
    aBits  : ARRAY[1..2] OF ST_Bits;
END_UNION
END_TYPE
```

### Instantiation

```pascal
VAR
    u2Byte : U_2Byte;
END_VAR
```

---

## 🧪 Assignment 1

```pascal
u2Byte.nUINT := 5;
```

### Bellek düzeni:

- 5 = `0000 0000 0000 0101` (binary)
- Byte[1] = 5  
- Byte[2] = 0  

Bits dizisinde:

| Bit | Değer |
|----|--------|
| bBit1 | 1 |
| bBit2 | 0 |
| bBit3 | 1 |
| bBit4–8 | 0 |

---

## 🧪 Assignment 2

```pascal
u2Byte.nUINT := 255;
```

### Bellek düzeni:

- 255 = `11111111 00000000`
- a2Byte[1] = 255  
- a2Byte[2] = 0  

Bits dizisinde tüm bitler 1 olur.

---

## 🧪 Assignment 3

```pascal
u2Byte.nUINT := 256;
```

### Bellek düzeni:

- 256 = `00000001 00000000`  
- Byte[1] = 0  
- Byte[2] = 1  

Bu şekilde UNION:

- Bir sayı → byte dizisine  
- Byte dizisi → bit dizisine  
- Bit dizisi → UINT değerine  

**Anında dönüşebilir.**

---

# 🎯 5. UNION Ne Zaman Kullanılır?

## ✔ 5.1 Binary / Protocol Parsing
Modbus, CAN, EtherCAT, TCP paket çözümü.

## ✔ 5.2 Bitfield Yönetimi
Bir register'ın hem tümünü hem bitlerini okumak için:

```pascal
TYPE U_Register :
UNION
    Full : WORD;
    Bits : ST_RegisterBits;
END_UNION
END_TYPE
```

## ✔ 5.3 Sensor/Device Raw Data Mapping
Bir cihazdan gelen ham veri hem:

- FLOAT  
- DWORD  
- BYTE array  

olarak yorumlanabilir.

## ✔ 5.4 Endianness / Byte-Swap İşlemleri
Byte sırasının kontrol edilmesi.

---

# 🔥 6. Gerçek Dünya — FULL UNION Kullanım Örneği

```pascal
TYPE ST_StatusBits :
STRUCT
    bReady      : BIT;
    bActive     : BIT;
    bWarning    : BIT;
    bError      : BIT;
    bRemote     : BIT;
    bLocal      : BIT;
    bReserved1  : BIT;
    bReserved2  : BIT;
END_STRUCT
END_TYPE

TYPE U_StatusWord :
UNION
    Full   : WORD;
    Bytes  : ARRAY[1..2] OF BYTE;
    Bits   : ST_StatusBits;
END_UNION
END_TYPE
```

### Kullanım

```pascal
VAR
    Status : U_StatusWord;
END_VAR

Status.Full := 16#000A; // 0000 0000 0000 1010
```

Sonuç:

- Bits.bActive = 1  
- Bits.bError = 1  
- Diğer bitler = 0  

---

# 🧠 7. UNION ile STRUCT Arasındaki Fark

| Özellik | STRUCT | UNION |
|--------|--------|--------|
| Bellek düzeni | Her üye ayrı adres | Tüm üyeler aynı adres |
| Kullanım | Veri gruplama | Veri yorumlama |
| Boyut | Tüm alanların toplamı | En büyük elemanın boyutu |
| Amaç | Mantıksal paketleme | Low-level memory reinterpretation |

---

# 🏆 8. Union Masterclass — Özet

Bu eğitimde:

✔ UNION nedir  
✔ Neden kullanılır  
✔ Bellek düzeni  
✔ Bit → Byte → Word → Number dönüşüm zinciri  
✔ Gerçek dünya örnekleri  
✔ Low-level data parsing teknikleri  

**Tam kapsamlı şekilde öğretildi.**

---





