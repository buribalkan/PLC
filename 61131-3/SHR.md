# 🧠 SEVİYE 3 ULTRA PROFESYONEL MASTERCLASS  
# **SHR — BITWISE SHIFT RIGHT OPERATÖRÜ DERİN TEKNİK EĞİTİMİ**

---

# 📌 İçindekiler  
1. SHR Nedir?  
2. Logical Right Shift — TwinCAT’in Varsayılan Davranışı  
3. Veri Tipi Genişliği ve SHR Etkileşimi  
4. n Değeri Tip Genişliğini Aşarsa Ne Olur?  
5. Zero Padding vs Modulo Shift Davranışı  
6. Signed / Unsigned Türlerde SHR  
7. Derin Örnek Analizleri  
8. BYTE / WORD / DWORD / LWORD Bit Modeli  
9. Endüstriyel Kullanım Senaryoları  
10. Sık Yapılan Hatalar  
11. Örnek Kod  
12. Sonuç  

---

# 1. SHR Nedir?

`SHR(in, n)` → Bitleri **sağa kaydırır**, soldan boşalan bitleri **0** ile doldurur.

Özellikler:
- Logical shift right uygulanır  
- Arithmetic shift değildir  
- Signed türlerde bile işaret biti korunmaz  

Sadece bit pattern üzerinde işlem yapılır.

---

# 2. Logical Right Shift — TwinCAT’in Varsayılan Davranışı

TwinCAT’te SHR:

- Sadece bit kaydırır  
- Signed/unsigned fark etmeksizin **sola gelen yeni bitler = 0’dır**  
- İçerik sağa doğru kayar  
- Sağdan düşen bitler kaybolur  

---

# 3. Veri Tipi Genişliği SHR Sonucunu Belirler

Her data type kendi bit genişliğine göre shift işlemi yapar:

| Veri Tipi | Genişlik |
|-----------|----------|
| BYTE | 8 bit |
| WORD | 16 bit |
| DWORD | 32 bit |
| LWORD | 64 bit |

Aynı başlangıç değeri farklı sonuçlar üretir:

```
BYTE 0100 0101 >> 2 = 0001 0001 (0x11)
WORD 0000 0000 0100 0101 >> 2 = 0000 0000 0001 0001 (0x0011)
```

---

# 4. n Değeri Tip Genişliğini Aşarsa Ne Olur?

TwinCAT dokümantasyonuna göre:

> Bu durum hedef sisteme göre değişir (target dependent).

İki olası davranış:

### ✔ Zero Padding (En yaygın)
Tüm sonuç sıfırlanır.

Örnek:
```
SHR(0x80, 10) BYTE → 0
```

### ✔ Modulo Behavior
```
effectiveShift = n MOD bitWidth
```

Örnek:
```
10 mod 8 = 2 → SHR(0x80, 2) = 0x20
```

---

# 5. Zero Padding vs Modulo Shift Davranışı

| Donanım Türü | Tipik Davranış |
|--------------|----------------|
| TwinCAT x86/x64 | Zero padding |
| Bazı gömülü CPU’lar | Modulo |
| FPGA tabanlı kontrolörler | Modulo |

Gerçek cihazda doğrulamak önemlidir.

---

# 6. Signed / Unsigned Türlerde SHR

SHR her zaman logical shift’tir, arithmetic değildir.

Örnek:

```
INT#-2 = 1111 1111 1111 1110
SHR(-2,1)
= 0111 1111 1111 1111 (32767)
```

Signed bir sayı logical işlemle pozitif büyük sayıya dönüşebilir.

---

# 7. Derin Örnek Analizi (Dokümantasyondaki)

Girdi:

```
nInByte = 0x45   → 0100 0101
nInWord = 0x0045 → 0000 0000 0100 0101
nVar = 2
```

### BYTE:

```
0100 0101 >> 2 =
0001 0001 = 0x11
```

### WORD:

```
0000 0000 0100 0101 >> 2 =
0000 0000 0001 0001 = 0x0011
```

Bit genişliği farkından dolayı sonuçlar farklıdır.

---

# 8. BYTE / WORD / DWORD / LWORD Bit Modeli

| Tip | Yaygın Kullanım |
|------|------------------|
| BYTE | Bitfield, IO flags |
| WORD | Status/control words |
| DWORD | Drive telegramları |
| LWORD | Protokol çözümleme |

SHR, özellikle büyük telegramlarda bit çözümleme için tercih edilir.

---

# 9. Endüstriyel Kullanım Senaryoları

✔ EtherCAT / CANopen / Profinet frame parsing  
✔ Status Word / Control Word bit analizleri  
✔ CRC & checksum algoritmalarının parçaları  
✔ Bit mask çözme  
✔ Çok byte’lı register değer ayrıştırma  
✔ Device profile (DS402 vb.) analizleri  

---

# 10. Sık Yapılan Hatalar

### ❌ Veri tipini unutarak shift yapmak
BYTE ve WORD tamamen farklı sonuç üretir.

### ❌ Arithmetic shift beklemek
SHR asla arithmetic değildir.

### ❌ n genişliği aşınca sonucu tahmin etmek
Her cihaz zero padding yapmaz.

### ❌ Signed türlerde işaret korunduğunu sanmak

---

# 11. Örnek Kod

```st
PROGRAM Shr_st
VAR
    nInByte  :  BYTE := 16#45;      (* 0100 0101 *)
    nInWord  :  WORD := 16#0045;    (* 0000 0000 0100 0101 *)
    nResByte :  BYTE;
    nResWord :  WORD;
    nVar     :  BYTE := 2;
END_VAR

nResByte := SHR(nInByte, nVar);    (* 0x11 *)
nResWord := SHR(nInWord, nVar);    (* 0x0011 *)
```

---

# 12. Sonuç

SHR:

- Bit çözümlemede kritik bir operatördür  
- Logical shift uygular (arithmetic değildir)  
- Veri tipi genişliği sonucu belirler  
- Signed türlerde tamamen farklı numerik sonuçlar doğurabilir  
- Fieldbus ve register analizlerinin temelidir  

Bu masterclass, SHR operatörünün profesyonel kullanımını tüm ayrıntılarıyla açıklamaktadır.

---

