# 🧠 SEVİYE 3 ULTRA PROFESYONEL MASTERCLASS  
# **REAL & LREAL — IEEE 754 FLOATING-POINT TWINCAT MİMARİSİ**

---

## 📌 İçindekiler
1. REAL & LREAL Nedir?  
2. IEEE 754 Floating-Point Yapısı  
3. Değer Aralıkları ve Precision  
4. Normalized & Denormalized Sayılar  
5. Overflow, Underflow, NaN, ±INF  
6. Integer ↔ Floating Dönüşüm Tehlikeleri  
7. TwinCAT Floating Literal Kuralları  
8. Büyük Sayı Literal Ataması  
9. Deterministik Olmayan Hesaplama Problemleri  
10. Endüstriyel Kullanım Senaryoları  
11. Kod Örnekleri  
12. Profesyonel Öneriler  
13. Sonuç  

---

## 1. REAL & LREAL Nedir?

TwinCAT’te iki floating-point veri tipi vardır:

| Veri Tipi | Boyut | Format | Precision |
|-----------|--------|-------------------|------------|
| **REAL**  | 32 bit | IEEE 754 Single   | ~7 digit   |
| **LREAL** | 64 bit | IEEE 754 Double   | ~15–16 digit |

Bu veri tipleri ondalıklı, üstel gösterimli ve bilimsel hesaplamalarda kullanılır.

---

## 2. IEEE 754 Floating-Point Yapısı

Bir floating point sayı üç bileşenden oluşur:

1. **Sign bit (işaret)**  
2. **Exponent (üs)**  
3. **Mantissa (fraction)**  

Bu yapı çok geniş bir sayı aralığı sağlar ancak:

- Precision sınırlıdır  
- Büyük ve küçük sayıların karıştığı hesaplarda hata büyüyebilir  

---

## 3. Değer Aralıkları ve Precision

### REAL (32 bit)
- En küçük negatif: **–3.402823e+38**  
- En büyük pozitif: **3.402823e+38**  
- En küçük pozitif: **1.0e–44**

### LREAL (64 bit)
- En küçük negatif: **–1.7976931348623158e+308**  
- En büyük pozitif: **1.7976931348623158e+308**  
- En küçük pozitif: **4.94e–324**

### PRECISION KARŞILAŞTIRMASI
- REAL hatası ~1e–7 düzeyinde  
- LREAL hatası ~1e–15 düzeyinde  

Bu nedenle TwinCAT projelerinde **REAL yerine LREAL önerilir**.

---

## 4. Normalized & Denormalized Sayılar

Floating sayı bölgeleri:

- **Normal numbers** → exponent normal aralıkta  
- **Denormal numbers** → çok küçük değerler, precision kaybı olur  
- **Zero** → +0 ve –0 farklıdır  

LREAL denormal aralığı REAL’e göre çok daha geniştir.

---

## 5. Overflow, Underflow, NaN, ±INF

### Overflow  
Sayı aralığı aşılır → +INF veya –INF

### Underflow  
Sayı çok küçüktür → 0 veya denormalize

### NaN (Not a Number)
Şu durumlarda oluşur:

- 0/0  
- ∞ – ∞  
- sqrt(negative)  
- Tan(π/2) gibi tanımsız fonksiyonlar

---

## 6. Integer ↔ Floating Dönüşüm Tehlikeleri

TwinCAT uyarısı:  
**Eğer REAL/LREAL sayısı integer aralığını aşıyorsa sonuç “undefined” olur.**

Örnek:

```st
fVal : REAL := 3.4e38;
nVal : DINT;

nVal := DINT(fVal);  // TANIMSIZ (rastgele sonuç)
```

---

## 7. TwinCAT Floating Literal Kuralları

TwinCAT, literal değerleri türüne göre yorumlar.

Örneğin:

```
3400000000000000000000
```

Bu değer integer olarak yorumlanır → overflow olur.

---

## 8. Büyük Sayı Literal Ataması

Doğru kullanım:

```st
fMyReal : REAL := 3400000000000000000000.0;
fMyReal : REAL := REAL#3400000000000000000000;
```

Yanlış kullanım:

```st
fMyReal : REAL := 3400000000000000000000; // Hatalı
```

---

## 9. Deterministik Olmayan Hesaplama Problemleri

Floating point matematik:

- Toplama sırasına bağlıdır  
- Precision hataları birikir  
- Büyük + küçük sayı → küçük sayı kaybolabilir  
- Realtime sistemlerde deterministik değildir

Bu nedenle:

✔ Motion / PID → **LREAL kullan**  
✔ Finansal hesaplar → FLOAT ASLA KULLANMA  

---

## 10. Endüstriyel Kullanım Senaryoları

### LREAL kritik olduğu alanlar:
- Robotik  
- Motion control  
- PID & filtreleme  
- Sensör verisi işleme  
- Bilimsel hesaplamalar  
- Yüksek çözünürlük gerektiren ölçümler  

REAL yalnızca:

- Basit gösterimler  
- Hafıza kısıtlı modüller  

için tercih edilir.

---

## 11. Kod Örnekleri

```st
PROGRAM MAIN
VAR
    fMaxReal     : REAL  := 3.402823E+38;
    fPosMinReal  : REAL  := 1.0E-44;
    fNegMaxReal  : REAL  := -1.0E-44;
    fMinReal     : REAL  := -3.402823E+38;

    fMaxLreal    : LREAL := 1.7976931348623157E+308;
    fPosMinLreal : LREAL := 4.94065645841247E-324;
    fNegMaxLreal : LREAL := -4.94065645841247E-324;
    fMinLreal    : LREAL := -1.7976931348623157E+308;
END_VAR
```

---

## 12. Profesyonel Öneriler

### ✔ REAL yerine her zaman **LREAL** tercih et  
PLC dünyasında precision çok kritiktir.

### ✔ Integer dönüşümlerine dikkat et  
Taşma → undefined behavior

### ✔ Büyük sayılar için `.0` veya `REAL#` kullan

### ✔ Deterministik gerektiren uygulamalarda FLOAT dikkatli kullanılmalı  
Örneğin: güvenlik fonksiyonlarında FLOAT önerilmez.

---

## 13. Sonuç

REAL & LREAL:

- IEEE 754 standardına uygundur  
- Geniş sayı aralığı sunar  
- Precision sınırlıdır  
- LREAL yüksek doğruluk gerektiren tüm endüstriyel uygulamalarda kullanılmalıdır  
- TwinCAT literal kurallarının doğru anlaşılması önemlidir  

Bu masterclass, TwinCAT floating-point mimarisini profesyonel düzeyde anlamak için eksiksiz referans niteliğindedir.

---
