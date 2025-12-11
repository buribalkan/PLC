# 🧠 SEVİYE 3 ULTRA PROFESYONEL MASTERCLASS  
# **ROL — ROTATE LEFT OPERATÖRÜ DERİN TEKNİK EĞİTİMİ**

---

# 📌 İçindekiler  
1. ROL Nedir?  
2. Rotate vs Shift — Temel Farklar  
3. Veri Tipi Genişliğinin Rotasyona Etkisi  
4. TwinCAT ROL Mekanizması  
5. Constant Input → “Minimum Data Type” Davranışı  
6. Döngüsel Bit Pozisyonu (Modular Rotation)  
7. Derin Örnek Analizi  
8. BYTE / WORD / DWORD / LWORD Rotasyon Yapısı  
9. Endüstriyel Kullanım Alanları  
10. Sık Yapılan Hatalar  
11. Örnek Kod  
12. Sonuç  

---

# 1. ROL Nedir?

`ROL(in, n)` operatörü:

- Operand *in*’in bitlerini sola **rotate** eder  
- Sola taşan bitler **kaybolmaz**, sağdan tekrar girer  
- Bu nedenle **tam döngüsel (cyclic) bit hareketi** gerçekleşir  

Shift (SHL) ile karıştırılmamalıdır.

---

# 2. Rotate vs Shift — Temel Farklar

| Operatör | Davranış |
|----------|-----------|
| **SHL** | Bitler sola kayar, taşan bitler **kaybolur**, sağdan 0 doldurulur |
| **ROL** | Bitler sola döner, taşan bitler **sağdan geri giriş yapar**, veri kaybı olmaz |

ROL = cyclic bit operation  
SHL = destructive bit shift

---

# 3. Veri Tipi Genişliğinin Rotasyona Etkisi

Rotasyon **her zaman input’un veri tipi genişliği** üzerinden hesaplanır:

| Veri Tipi | Bit Sayısı |
|-----------|------------|
| BYTE | 8 bit |
| WORD | 16 bit |
| DWORD | 32 bit |
| LWORD | 64 bit |

Bu nedenle:

```
ROL(BYTE#16#45, 2) ≠ ROL(WORD#16#45, 2)
```

---

# 4. TwinCAT ROL Mekanizması

Rol algoritması:

```
MSB alınır
1 bit sola kaydırılır (SHL)
MSB → LSB’ye eklenir
n kez tekrarlanır
```

Yani veri asla kaybolmaz, sadece pozisyon değiştirir.

---

# 5. Constant Input → “Minimum Data Type” Davranışı

ÖNEMLİ KURAL:

> Eğer giriş *constant* ise, TwinCAT input’u minimum veri tipi olarak kabul eder.

Örnek:

```
ROL(16#45, 2)
```

Burada 16#45 → BYTE tipinde değerlendirilir.

---

# 6. Döngüsel Bit Pozisyonu (Modular Rotation)

Rotate left matematiksel olarak:

```
ROL(x, n) = (x << n) OR (x >> (width – n))
```

Burada `width` operandın bit genişliğidir.

Bu yapı **modüler bit kaymasını** garanti eder.

---

# 7. Derin Örnek Analizi

Girdi:

```
nInByte = 0x45 → 0100 0101
nInWord = 0x0045 → 0000 0000 0100 0101
nVar = 2
```

### BYTE ROL (8 bit döngü):

Adım adım:

```
0100 0101 → ROL1 → 1000 1010
1000 1010 → ROL2 → 0001 0101
```

Sonuç:
```
0x15
```

### WORD ROL (16 bit döngü):

```
0000 0000 0100 0101 → ROL2 → 0000 0001 0001 0100
```

Sonuç:
```
0x0114
```

---

# 8. BYTE / WORD / DWORD / LWORD Döngüsel Yapısı

| Tip | Döngüsel Bit Yapısı | Örnek |
|------|----------------------|--------|
| BYTE | 0–7 | 8-bit cyclic rotation |
| WORD | 0–15 | 16-bit frame masks |
| DWORD | 0–31 | Protokol parsing |
| LWORD | 0–63 | Geniş telemetri/kontrol bitleri |

ROL → bit kayması değil, **bit yeniden konumlandırmasıdır.**

---

# 9. Endüstriyel Kullanım Alanları

ROL özellikle ileri seviye sistemlerde kullanılır:

✔ CRC algoritmaları  
✔ Hashing fonksiyonları  
✔ Frame encoding/decoding  
✔ CANopen / EtherCAT telegram çözümleme  
✔ Pseudo-random number generator (PRNG)  
✔ DSP bit manipülasyonu  
✔ Encoding/obfuscation algoritmaları  

---

# 10. Sık Yapılan Hatalar

### ❌ ROL ile SHL’i karıştırmak  
Rotate → veri kaybı yok  
Shift → veri kaybı var  

### ❌ Constant input → BYTE davranışını gözden kaçırmak  
`ROL(16#45,2)` → 8-bit rotate yapılır.

### ❌ Input tipine göre bit genişliği farkını göz ardı etmek  
BYTE vs WORD sonuçları tamamen değişir.

### ❌ Çok büyük n değerlerinde modulo beklememek  
TwinCAT otomatik olarak `n mod width` uygular.

---

# 11. Örnek Kod

```st
PROGRAM Rol_st 
VAR 
    nInByte  : BYTE := 16#45; 
    nInWord  : WORD := 16#45;
    nResByte : BYTE; 
    nResWord : WORD; 
    nVar     : BYTE := 2; 
END_VAR 

nResByte := ROL(nInByte, nVar);   (* 0x15 *)
nResWord := ROL(nInWord, nVar);   (* 0x0114 *)
```

---

# 12. Sonuç

ROL:

- Veri kaybetmeyen döngüsel bir bit rotasyonudur  
- SHL’den tamamen farklıdır  
- Veri tipine göre bit genişliği sonuç üzerinde belirleyicidir  
- Sabit girişlerde TwinCAT minimum data type uygular  
- Endüstriyel protokol çözümleme, CRC, hashing ve bit manipülasyonlarında kritik bir rol oynar  

Bu doküman ROL operatörünün profesyonel kullanımını tüm detaylarıyla açıklar.

---
