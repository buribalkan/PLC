# 🧠 SEVİYE 3 ULTRA PROFESYONEL MASTERCLASS  
# **SHL — BITWISE SHIFT LEFT OPERATÖRÜ DERİN TEKNİK EĞİTİMİ**

---

# 📌 İçindekiler  
1. SHL Nedir?  
2. TwinCAT Derleyici & CPU Bitwise İşlem Mantığı  
3. Veri Tipi Bit Genişliği ve SHL Etkileşimi  
4. Shift Amount (n) Limit Aşımı Davranışı  
5. Zero Padding vs Modulo Behavior  
6. Signed / Unsigned Türlerde SHL  
7. Derin Örnek Analizleri  
8. BYTE / WORD / DWORD / LWORD Bit Modeli  
9. Endüstriyel Kullanım Senaryoları  
10. Sık Yapılan Hatalar  
11. Örnek Kod  
12. Sonuç  

---

# 1. SHL Nedir?

`SHL(in, n)` → Operand *in* değerinin bitlerini *n* kadar **sola kaydırır**.

- Sola kaydırılan bitler düşer (kaybolur)
- Sağdan giren yeni bitler **0** ile doldurulur

---

# 2. TwinCAT Derleyici & CPU Bitwise İşlem Mantığı

TwinCAT, SHL’i **logical shift left** olarak uygular.

- Arithmetic shift değildir  
- Signed veri kullanılsa bile işlem bit pattern’i üzerinde yapılır  
- İşlem mantığı tamamen **data type genişliğine** bağlıdır  

---

# 3. Veri Tipi Bit Genişliği SHL Davranışını Belirler

Aynı sayı, farklı veri tiplerinde **farklı sonuçlar** yaratır:

| Veri Tipi | Bit Genişliği |
|-----------|----------------|
| BYTE | 8 |
| WORD | 16 |
| DWORD | 32 |
| LWORD | 64 |

Bu nedenle:

```st
SHL(BYTE#16#45, 2)   → 0x14
SHL(WORD#16#45, 2)   → 0x0114
```

---

# 4. Shift Amount (n) Limit Aşımı Davranışı

TwinCAT belgelendirmesi:

> Eğer n, veri tipinin bit genişliğini aşarsa davranış hedef sisteme (target system) bağlıdır.

Tipik iki davranış görülür:

### ✔ Zero Padding  
- Birçok sistemde tüm sonuç **0** olur  

### ✔ Modulo Behavior  
Bazı hedeflerde:

```
effectiveShift = n MOD bitWidth
```

---

# 5. Zero Padding vs Modulo Behavior

**Zero Padding:**
```
SHL(1, 100) BYTE → 0
```

**Modulo Behavior (örnek donanımlar):**
```
100 MOD 8 = 4 → SHL(1,4) = 0x10
```

TwinCAT çoğunlukla zero padding kullanır.

---

# 6. Signed / Unsigned Türlerde SHL

SHL her zaman **logical shift** yapar.

Signed türlerde bile arithmetic shift beklenmez.

Örnek:

```st
VAR
    a : INT := 16384;    // 0x4000 = 0100 0000 0000 0000
END_VAR

a := SHL(a,1);           // 0x8000 → -32768
```

Bit pattern değiştiği için işaret etkilenebilir.

---

# 7. Derin Örnek Analizleri

Verilen değerler:

```
nInByte = 0x45   → 0100 0101
nInWord = 0x0045 → 0000 0000 0100 0101
nVar = 2
```

### BYTE SHL:

```
0100 0101 << 2 =
0001 0100 (0x14)
```

### WORD SHL:

```
0000 0000 0100 0101 << 2 =
0000 0001 0001 0100 (0x0114)
```

Aynı değer, **bit genişliği farklı olduğu** için farklı sonuçlar üretmiştir.

---

# 8. BYTE / WORD / DWORD / LWORD Bit Modeli

| Tip | Genişlik | Tipik Kullanım |
|------|-----------|----------------|
| BYTE | 8 bit | IO bayrakları, compact bitfield |
| WORD | 16 bit | Status word, alarm mask |
| DWORD | 32 bit | Drive telegramları |
| LWORD | 64 bit | Geniş protokol bit setleri |

SHL ile bit manipülasyonu yaparken doğru veri tipi **en kritik faktördür**.

---

# 9. Endüstriyel Kullanım Senaryoları

✔ Fieldbus frame parsing (CANopen, EtherCAT)  
✔ Status/Control word decoding  
✔ CRC, checksum ve bitwise algoritmalar  
✔ IO mapping bit mask işlemleri  
✔ Motion control telegram bit analizi  
✔ Register manipülasyonları  

---

# 10. Sık Yapılan Hatalar

### ❌ 1. Veri tipini unutarak SHL yapmak
```
BYTE vs WORD → tamamen farklı sonuç
```

### ❌ 2. Shift amount veri genişliğini aşarsa davranışı öngörmemek

### ❌ 3. Signed türlerde arithmetic shift beklentisi

### ❌ 4. Protokol çözümlemede yanlış maskeleme yapmak

---

# 11. Örnek Kod

```st
PROGRAM Shl_st
VAR
    nInByte  :  BYTE := 16#45;      (* 0100 0101 *)
    nInWord  :  WORD := 16#0045;    (* 0000 0000 0100 0101 *)
    nResByte :  BYTE;
    nResWord :  WORD;
    nVar     :  BYTE := 2;
END_VAR

nResByte := SHL(nInByte, nVar);    (* 0x14 *)
nResWord := SHL(nInWord, nVar);    (* 0x0114 *)
```

---

# 12. Sonuç

SHL operatörü:

- Bit manipülasyonunda temel operatördür  
- Davranışı veri tipinin bit genişliği tarafından belirlenir  
- Signed türlerde arithmetic shift uygulanmaz  
- Fieldbus, alarm maskeleri, IO bit işleme gibi alanlarda kritik öneme sahiptir  

Bu masterclass ile SHL operatörünün tüm profesyonel kullanım detayları kapsamlı şekilde açıklanmıştır.

---

