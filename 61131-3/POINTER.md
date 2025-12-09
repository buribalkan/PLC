# 🚀 TwinCAT REFERENCE & POINTER – Tam Kapsamlı Eğitim Paketi (Profesyonel Seviye)

Bu eğitim paketi, TwinCAT ortamında **REFERENCE** ve **POINTER** kavramlarını *sıfırdan profesyonel seviyeye kadar* öğreten eksiksiz bir kaynaktır.  
İçerik tamamen uygulama odaklıdır ve gerçek dünya örnekleri ile desteklenmiştir.

---

# # 📘 İçindekiler

1. REFERENCE Nedir?  
2. REFERENCE Operatörleri (REF= vs :=)  
3. REFERENCE’ın Gerçek Dünya Kullanımları  
4. Çarpıcı Örnek: 12 Bölgeli Fırın Kontrol Sistemi  
5. POINTER Nedir?  
6. Pointer Operasyonları (ADR, ^, Index Access)  
7. STRING/WSTRING Pointer Erişimi  
8. POINTER – REFERENCE Karşılaştırma Tablosu  
9. Online Change Sırasında Pointer/Reference Güncellemeleri  
10. Runtime Güvenlik (CheckPointer)  
11. Büyük Veri Yapıları ile Pointer Kullanımı  
12. Sonuç: Ne Zaman Pointer? Ne Zaman Reference?

---

# # 1️⃣ REFERENCE Nedir?

REFERENCE, bir değişkenin adresini tutan ve **dereference gerektirmeden** doğrudan değerine erişmeni sağlayan güvenli bir mekanizmadır.

```pascal
refInt : REFERENCE TO INT;
refInt REF= nA;
refInt := 10;   // nA = 10
```

REFERENCE, pointer'a göre:

✔ Daha güvenlidir  
✔ Daha temiz sözdizimine sahiptir  
✔ Tür güvenliği sağlar  

---

# # 2️⃣ REFERENCE Operatörleri

## 🔹 REF= (Adres Atama)

```pascal
refA REF= stData;   // refA artık stData’yı gösteriyor
```

## 🔹 := (Değer Atama)

```pascal
refA := refB;   // refB'nin değeri refA’nın gösterdiği yere yazılır
```

---

# # 3️⃣ REFERENCE’ın Gerçek Dünya Kullanımları

- HMI üzerinde dinamik veri yönlendirme  
- Çok bölgeli PID kontrol sistemleri  
- Sensör alias yönetimi  
- Multi-motor kontrol algoritmaları  
- Debug / test için değişken yönlendirme  

---

# # 4️⃣ Çarpıcı Örnek – 12 Bölgeli Fırın Kontrol Sistemi

Bu örnek REFERENCE’ın gerçek hayatta neden vazgeçilmez olduğunu gösterir.

## 🔸 Master FB

```pascal
FUNCTION_BLOCK FB_ZoneControl
VAR_INPUT
    refTemp : REFERENCE TO REAL;
    refSet  : REFERENCE TO REAL;
END_VAR
VAR_OUTPUT
    heaterOut : REAL;
END_VAR

heaterOut := (refSet - refTemp) * 0.5;
```

## 🔸 MAIN Programı

```pascal
VAR
    fbZoneCtrl : FB_ZoneControl;
    Temp : ARRAY[1..12] OF REAL;
    Setpoint : ARRAY[1..12] OF REAL;
    HeaterOut : ARRAY[1..12] OF REAL;
    SelectedZone : INT := 7;
END_VAR

fbZoneCtrl.refTemp REF= Temp[SelectedZone];
fbZoneCtrl.refSet  REF= Setpoint[SelectedZone];
fbZoneCtrl();
HeaterOut[SelectedZone] := fbZoneCtrl.heaterOut;
```

✔ Kod tekrar etmez  
✔ Değişken yönlendirme runtime’da yapılır  
✔ Modüler ve genişletilebilir  

REFERENCE bu noktada **eksiksiz bir mühendislik çözümüdür.**

---

# # 5️⃣ POINTER Nedir?

Pointer, bir değişkenin **hafıza adresini** tutar.

```pascal
pVal : POINTER TO INT;
pVal := ADR(nValue);
```

Pointer değerine erişmek için:

```pascal
nResult := pVal^;
```

---

# # 6️⃣ Pointer Operasyonları

### ✔ ADR() – Adres Alma  
```pascal
pA := ADR(nA);
```

### ✔ ^ Dereference  
```pascal
nX := pA^;
```

### ✔ Index Access  
```pascal
pArr[i]    // (pArr + i * SIZEOF(BaseType))^
```

---

# # 7️⃣ STRING Pointer Erişimi

```pascal
sData : STRING := 'HELLO';
myChar := sData[1];   // 'E'
```

BYTE döner → ASCII kodu.

---

# # 8️⃣ WSTRING Pointer Erişimi

```pascal
wsData : WSTRING := "TEST";
c := wsData[2];       // 16-bit Unicode
```

---

# # 9️⃣ POINTER vs REFERENCE Karşılaştırma

| Özellik | POINTER | REFERENCE |
|--------|---------|-----------|
| Dereference gerekir | ✔ | ❌ |
| Tür güvenliği | ❌ | ✔ |
| Okuma/yazma riski | yüksek | düşük |
| Kullanım amacı | düşük seviye | yüksek seviye |
| Hata riski | yüksek | düşük |
| Online Change güncellemesi | ✔ | ✔ |

---

# # 1️⃣0️⃣ Online Change – Pointer/Reference Güncelleme

TwinCAT (4026+) Online Change sırasında:

✔ Pointer/Reference yeni sembole otomatik taşınır  
❌ Sembol yoksa pointer = 0 yapılır  
✔ PLC dışı adres gösteren pointer değiştirilmez  

---

# # 1️⃣1️⃣ CheckPointer – Runtime Güvenlik

TwinCAT pointer erişimini sürekli izler.  
Geçersiz adres → watchdog reset → PLC STOP olabilir.

Pointer’ın tehlikeli kabul edilme nedeni budur.  
REFERENCE bu riski ortadan kaldırır.

---

# # 1️⃣2️⃣ Büyük Veri Yapıları ile Pointer Kullanımı

```pascal
TYPE ST_Point3D :
STRUCT
    X : REAL;
    Y : REAL;
    Z : REAL;
END_STRUCT
END_TYPE

VAR
    pPoint : POINTER TO ST_Point3D;
    P : ST_Point3D := (X:=10, Y:=20, Z:=30);
END_VAR

pPoint := ADR(P);
pPoint^.X := 100;
pPoint^.Z := pPoint^.X + pPoint^.Y;
```

Pointer burada ciddi bir hız avantajı sağlar.

---

# # 🎯 SONUÇ — ULTIMATE ÖĞRENME ÖZETİ

### **REFERENCE → kod yönlendirme, modülerlik ve güvenlik aracıdır.**  
### **POINTER → düşük seviye, hafızaya direkt erişim sağlar.**

Kısa prensip:

- Sistem tasarımı → **REFERENCE**  
- Hafıza işlemleri & veri parsing → **POINTER**  

Bu eğitim paketi ile artık:

✔ REFERENCE’ın gerçek değeri  
✔ POINTER’ın profesyonel kullanım alanları  
✔ Aralarındaki fark  
✔ Gerçek bir makinede nasıl uygulandığı  

tamamen anlaşılmış olur.

---

# 🚀 TwinCAT POINTER – Gerçek Dünya Kullanım Örnekleri (Profesyonel .md Dokümanı)

Bu doküman, TwinCAT içerisinde **POINTER kullanımının gerçek dünyada nerede, neden ve nasıl kullanıldığını** açık, güçlü ve uygulama odaklı örneklerle açıklar.  
REFERENCE ile yapılamayan, gerçek otomasyon mühendisliğinde *pointer zorunluluğu* doğan tüm senaryoları içerir.

---

# # 1️⃣ Modbus / TCP / Binary Protokol Parsing (En Yaygın Kullanım)

Bir Modbus cihazından gelen veri buffer’ı:

```
[ 0x01 ][ 0x03 ][ 0x02 ][ 0x00 ][ 0x64 ][ CRC_L ][ CRC_H ]
```

Bu frame'i byte byte parçalamak pointer ile yapılır.

```pascal
VAR
    pData : POINTER TO BYTE;
    rxBuffer : ARRAY[0..255] OF BYTE;
    deviceAddress : BYTE;
    functionCode  : BYTE;
    payloadValue  : UINT;
END_VAR

pData := ADR(rxBuffer);

deviceAddress := pData^;           // Byte 0
functionCode  := pData[1];         // Byte 1
payloadValue  := SHL(pData[3], 8) OR pData[4];
```

### ✔ Pointer burada ZORUNLU çünkü:
- Frame uzunluğu değişken
- Offset hesaplaması runtime’da değişiyor
- REFERENCE dinamik index ile çalışamaz

---

# # 2️⃣ Donanım Belleği Üzerinde Direkt Okuma (Memory-Mapped IO)

Bazı EtherCAT cihazları PLC memory alanına map edilir.

```pascal
VAR
    pMem : POINTER TO BYTE;
END_VAR

pMem := ADR(ECAT_Input_Buffer);

status  := pMem[0];
error   := pMem[1];
tempRaw := pMem[10] + SHL(pMem[11], 8);
```

### ✔ Kullanım alanları:
- Servo sürücü register raw dataları
- Encoder raw pozisyon
- Özel I/O kart memory alanları

REFERENCE burada çalışamaz.

---

# # 3️⃣ FIFO / Ring Buffer Yönetimi

Yüksek hızlı data logging sistemlerinde ring buffer pointer ile yönetilir.

```pascal
VAR
    pWrite : POINTER TO BYTE;
    buffer : ARRAY[0..1023] OF BYTE;
    index  : INT := 0;
END_VAR

pWrite := ADR(buffer);

pWrite[index] := newData;
index := (index + 1) MOD 1024;
```

### ✔ Pointer kullanmanın sebebi:
- Direkt adres işleme
- Yüksek hız gerektiren işlerde O(1) erişim
- Dairesel buffer mantığı pointer ile doğal

---

# # 4️⃣ Struct → Byte Stream (Serialization)

Aşağıdaki struct’ın binary versiyonu network üzerinden gönderilecek.

```pascal
TYPE ST_Packet :
STRUCT
    Id     : UINT;
    Temp   : REAL;
    Status : BYTE;
END_STRUCT
END_TYPE
```

```pascal
VAR
    packet : ST_Packet;
    pByte  : POINTER TO BYTE;
END_VAR

pByte := ADR(packet);

// Paketin tüm binary içeriğini socket'e gönder
Send(pByte[0], SIZEOF(packet));
```

### ✔ Pointer neden şart?
- Struct’ın ham hafıza şeklini (binary) almak için tek yol pointer’dır
- Endian işlemleri yapılabilir
- Ağ protokolleri ile uyumludur

---

# # 5️⃣ Büyük Veri Yapılarında Performans Optimizasyonu

10.000 elemanlı bir array’de hız yükseltmek pointer ile mümkündür.

```pascal
VAR
    pItem : POINTER TO INT;
    Items : ARRAY[1..10000] OF INT;
END_VAR

pItem := ADR(Items);

FOR i := 1 TO 10000 DO
    pItem[i] := pItem[i] + 1;
END_FOR
```

### ✔ Pointer avantajı:
- Hızlı adres hesaplama
- Döngü optimizasyonu
- Compiler pointer erişimini inline eder

REFERENCE bu amaçla uygun değildir.

---

# # 6️⃣ STRING / BYTE Manipülasyonu (Parser, Encoder, Cryptography)

```pascal
VAR
    pChar : POINTER TO BYTE;
    text  : STRING := 'HELLO';
END_VAR

pChar := ADR(text);

FOR i := 0 TO LEN(text) DO
    pChar[i] := pChar[i] + 1; // ASCII shift
END_FOR
```

### ✔ Kullanım alanları:
- Protokol dönüştürücüler
- Dosya formatı işleme
- Cryptographic XOR, shift, mask algoritmaları

---

# # 7️⃣ Universal Stream Processor (Her Türlü Veri Kaynağı İçin Dinamik Çalışan FB)

Pointer ile tek bir FB, birçok tür veri buffer’ını işleyebilir:

```pascal
FUNCTION_BLOCK FB_StreamProcessor
VAR_INPUT
    pStream : POINTER TO BYTE;
    nLength : UINT;
END_VAR
```

Kullanım:

```pascal
fbProcessor(pStream := ADR(ModbusBuffer), nLength := modbusLen);
fbProcessor(pStream := ADR(UdpRxBuffer),   nLength := udpLen);
fbProcessor(pStream := ADR(FileData),      nLength := fileLen);
```

### ✔ Bu esnekliği sadece pointer sağlar.

---

# # 🎯 Özet: POINTER Nerede Zorunlu?

| Gerçek Dünya Kullanımı | POINTER Zorunlu? |
|------------------------|------------------|
| Modbus / CAN / TCP binary parsing | ✔ EVET |
| Memory-mapped I/O erişimi | ✔ EVET |
| Ring buffer / FIFO | ✔ EVET |
| Struct binary serialization | ✔ EVET |
| High‑performance array processing | ✔ EVET |
| String/byte manipulation | ✔ EVET |
| Basit alias / mapping | ❌ REFERENCE daha uygun |

---

# # 📘 Sonuç

POINTER, TwinCAT'te **düşük seviye, yüksek performans ve esneklik gereken** tüm mühendislik uygulamalarında zorunlu bir araçtır.

REFERENCE → yüksek seviye, güvenli ve modüler  
POINTER → düşük seviye, güçlü ve riskli ama *çok gerekli*

---





