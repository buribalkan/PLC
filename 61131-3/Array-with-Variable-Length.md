# 📘 TwinCAT – Array with Variable Length (Değişken Uzunluklu Diziler)
### *Tam Kapsamlı Eğitim Dokümanı (.md)*

---

# 🔍 1. Array with Variable Length Nedir?

TwinCAT’te normal dizilerde boyut **derleme zamanında** bellidir:

```pascal
aData : ARRAY[1..10] OF INT;
```

Ancak bazı fonksiyon veya metodlarda dizinin uzunluğu **çalışma zamanında (runtime)** gelen parametreye göre belirlenir.  
Bunun için TwinCAT özel bir yapı sunar:

👉 **VAR_IN_OUT bölümünde ARRAY[*] ifadesi**

Bu sayede fonksiyona:

- 3 elemanlı array,
- 10 elemanlı array,
- 100 elemanlı array

**aynı kodla işlenebilir.**

Bu, fonksiyonların yeniden kullanılabilirliğini büyük ölçüde artırır.

---

# 🎯 2. Neden Kullanılır?

| Amaç | Açıklama |
|------|----------|
| ✔ Aynı fonksiyonu her boyuttaki dizi için kullanmak | En büyük avantajdır |
| ✔ Kod tekrarını azaltmak | Her boyut için ayrı fonksiyon yazmaya gerek yok |
| ✔ Dinamik veri işleme | Batch, sensör listeleri, reçeteler |
| ✔ Çalışma zamanı esnekliği | Boyut runtime’da belirlenir |

> **Not:** ARRAY[*] sadece `VAR_IN_OUT` bölümünde kullanılabilir.

---

# ⚠ 3. Önemli Kurallar

| Durum | Geçerli / Geçersiz |
|-------|---------------------|
| Statik array geçmek | ✔ Evet |
| `__NEW` ile oluşturulmuş dinamik array geçmek | ❌ Hayır |
| Çok boyutlu variable-length array | ✔ Evet |
| STRUCT / FB içeren array | ✔ Evet |
| BIT tabanlı array | ❌ Hayır |

---

# 🔧 4. Sözdizimi (Syntax)

### ✔ Tek Boyutlu Değişken Uzunluklu Array

```pascal
aData : ARRAY[*] OF INT;
```

### ✔ Çok Boyutlu Değişken Uzunluklu Array

```pascal
aMatrix : ARRAY[*, *] OF LREAL;
```

Her `*` işareti, o boyutun **runtime’da gelen array’e göre belirlendiği** anlamına gelir.

---

# 📐 5. Dizinin Sınırlarını Öğrenmek  
👉 TwinCAT iki özel operatör sağlar:

| Operatör | Açıklama |
|---------|----------|
| **LOWER_BOUND(array, dimension)** | Alt index |
| **UPPER_BOUND(array, dimension)** | Üst index |

Bu operatörler olmadan değişken uzunluklu array işlenemez.

---

# 🧩 6. Örnek Fonksiyon: Array Toplama

TwinCAT tarafından verilen örneğin geliştirilmiş hâli:

```pascal
FUNCTION AddValues : DINT
VAR_IN_OUT
    aData  : ARRAY[*] OF INT;   // Değişken uzunluklu array
END_VAR
VAR
    nIdx   : DINT;
    nSum   : DINT;
END_VAR

nSum := 0;

FOR nIdx := LOWER_BOUND(aData, 1) TO UPPER_BOUND(aData, 1) DO
    nSum := nSum + aData[nIdx];
END_FOR;

AddValues := nSum;
```

---

# 📌 7. Bu Fonksiyona Hangi Diziler Geçilebilir?

### ✔ Boyutları 0..4 olan array:

```pascal
aNumbers : ARRAY[0..4] OF INT := [10, 20, 30, 40, 50];
nResult := AddValues(aNumbers);
```

### ✔ Boyutları 5..10 olan array:

```pascal
aOther : ARRAY[5..10] OF INT := [1,2,3,4,5,6];
nResult := AddValues(aOther);
```

Fonksiyon her seferinde **doğru sınırları kendi hesaplar.**

---

# 🧮 8. Çok Boyutlu Değişken Uzunluklu Array Örneği (Matris Toplama)

```pascal
FUNCTION SumMatrix : LREAL
VAR_IN_OUT
    aMatrix : ARRAY[*,*] OF LREAL;
END_VAR
VAR
    x, y : INT;
    rSum : LREAL;
END_VAR

FOR x := LOWER_BOUND(aMatrix, 1) TO UPPER_BOUND(aMatrix, 1) DO
    FOR y := LOWER_BOUND(aMatrix, 2) TO UPPER_BOUND(aMatrix, 2) DO
        rSum := rSum + aMatrix[x][y];
    END_FOR
END_FOR

SumMatrix := rSum;
```

✔ 2×2, 3×5, 10×10 matrislerle çalışabilir.  
✔ Kamera verileri veya ölçüm tabloları için uygundur.

---

# 🏭 9. Gerçek Dünya Kullanım Senaryoları

### ✔ 1. Sensör veri toplama  
Makine modeline göre sensör sayısı değişebilir:

```pascal
ProcessSensors(aSensorArray);
```

### ✔ 2. Batch ölçümleri  
Operatör batch uzunluğunu belirler → fonksiyon aynı kalır.

### ✔ 3. Reçete parametre grupları  
Bazı reçetelerde 3 parametre, bazılarında 20 olabilir.

### ✔ 4. TwinCAT Vision ROI listeleri  
Kamera tarafından dönen ROI’ler dinamik uzunlukta olabilir.

### ✔ 5. Ürün kontrol istasyonları  
Her ürün tipi farklı sayıda test noktasına sahip olabilir.

---

# ⚠ 10. Neden `__NEW` ile Oluşturulan Array Geçilemez?

Çünkü：

- Boyutu runtime’da güvenilir değildir  
- Heap yönetimi PLC tarafında garanti edilmez  
- VAR_IN_OUT mekanizması **stack-bazlı çalışır**

Bu nedenle yalnızca **statik array tanımları** geçilebilir.

---

# 🎉 11. Özet

- ARRAY[*] → Değişken uzunluklu array  
- Sadece VAR_IN_OUT içinde tanımlanır  
- LOWER_BOUND / UPPER_BOUND → Mutlaka kullanılmalı  
- Statik array kabul eder, dinamik array etmez  
- Çok boyutlu diziler desteklenir  
- Esnek ve tekrar kullanılabilir fonksiyonlar yazmayı sağlar  

TwinCAT’te **data processing**, **batch operations**, **sensor handling** gibi görevler için en güçlü yapılardan biridir.

---

# 📘 TwinCAT – Full Data Processing Pipeline  
### *Gerçek Dünya İçin Tam Kapsamlı Eğitim Dokümanı (.md)*

---

# 🏭 1. Giriş – Full Data Processing Nedir?

Üretim hatlarında sensörlerden gelen veriler **ham** hâlde gürültülü, dengesiz, doğrusal olmayan ve hatalı olabilir.  
Bu nedenle veriler işlemeye alınmadan:

- Filtrelenir  
- Normalize edilir  
- Linearize edilir (Lookup Table ile)  
- Eşik değerlerine göre kontrol edilir  
- Batch istatistiklerine dönüştürülür  
- Formatlanmış bir sonuç yapısına aktarılır  

Bu dokümanda, endüstriyel otomasyon projelerinde kullanılan **tam kapsamlı bir veri işleme hattı (pipeline)** adım adım TwinCAT kodlarıyla anlatılmıştır.

---

# 🧱 2. Veri Yapısı Tanımı (ST_ProcessedData)

Bu yapı tüm veri işleme çıktılarının tek bir pakette tutulmasını sağlar.

```pascal
TYPE ST_ProcessedData :
STRUCT
    RawValues      : ARRAY[1..16] OF LREAL;
    FilteredValues : ARRAY[1..16] OF LREAL;
    Normalized     : ARRAY[1..16] OF LREAL;
    Linearized     : ARRAY[1..16] OF LREAL;
    MaxValue       : LREAL;
    MinValue       : LREAL;
    AvgValue       : LREAL;
    ThresholdFlags : ARRAY[1..16] OF BOOL;
END_STRUCT
END_TYPE
```

---

# 📈 3. Lookup Table (Linearization Table)

Bu tablo sensör doğrultma için kullanılır.

```pascal
VAR_GLOBAL CONSTANT
    aLinearLUT : ARRAY[0..100] OF LREAL := [
        0.0, 0.5, 1.0, 1.6, 2.3, 3.1
        (* ... örnek değerler 0..100 arası *)
    ];
END_VAR
```

---

# 🧠 4. Full Data Processing Function Block

Pipeline adımları tek FB içinde yapılır.

```pascal
FUNCTION_BLOCK FB_DataProcessor
VAR_INPUT
    aInput     : ARRAY[1..16] OF LREAL;
    rThreshold : LREAL := 75.0;
END_VAR

VAR_OUTPUT
    stOut : ST_ProcessedData;
END_VAR

VAR
    i, j   : INT;
    rSum   : LREAL;
    rMin   : LREAL := 99999;
    rMax   : LREAL := -99999;
    nIndex : INT;
END_VAR
```

---

# 🔧 5. Adım 1 – Ham Veriyi Kopyalama

```pascal
FOR i := 1 TO 16 DO
    stOut.RawValues[i] := aInput[i];
END_FOR
```

---

# 🔁 6. Adım 2 – Moving Average Filtreleme

Sensör gürültüsünü azaltmak için geçmiş 10 veri saklanır.

```pascal
VAR
    aHistory : ARRAY[1..16, 1..10] OF LREAL;
    nIdx     : INT := 1;
END_VAR

FOR i := 1 TO 16 DO
    aHistory[i, nIdx] := aInput[i];
END_FOR

nIdx := nIdx + 1;
IF nIdx > 10 THEN nIdx := 1; END_IF;

FOR i := 1 TO 16 DO
    rSum := 0;
    FOR j := 1 TO 10 DO
        rSum := rSum + aHistory[i, j];
    END_FOR
    stOut.FilteredValues[i] := rSum / 10;
END_FOR
```

---

# 🎚 7. Adım 3 – Normalize Etme (0–1 Arası)

Önce minimum ve maksimum değer bulunur:

```pascal
FOR i := 1 TO 16 DO
    IF stOut.FilteredValues[i] < rMin THEN rMin := stOut.FilteredValues[i]; END_IF
    IF stOut.FilteredValues[i] > rMax THEN rMax := stOut.FilteredValues[i]; END_IF
END_FOR
```

Normalize işlemi:

```pascal
FOR i := 1 TO 16 DO
    stOut.Normalized[i] := (stOut.FilteredValues[i] - rMin) / (rMax - rMin);
END_FOR
```

---

# 🔍 8. Adım 4 – Lookup Table ile Linearization

```pascal
FOR i := 1 TO 16 DO
    nIndex := LIMIT(0, INT(stOut.Normalized[i] * 100), 100);
    stOut.Linearized[i] := aLinearLUT[nIndex];
END_FOR
```

---

# 🚨 9. Adım 5 – Threshold Kontrolü

```pascal
FOR i := 1 TO 16 DO
    stOut.ThresholdFlags[i] := (stOut.Linearized[i] > rThreshold);
END_FOR
```

---

# 📊 10. Adım 6 – Batch İstatistikleri (Min, Max, Ortalama)

```pascal
rSum := 0;
FOR i := 1 TO 16 DO
    rSum := rSum + stOut.Linearized[i];
END_FOR

stOut.AvgValue := rSum / 16;
stOut.MaxValue := rMax;
stOut.MinValue := rMin;
```

---

# 🧪 11. Kullanım Örneği (MAIN Program)

```pascal
PROGRAM MAIN
VAR
    fbProc   : FB_DataProcessor;
    aSensors : ARRAY[1..16] OF LREAL := [12.3, 11.0, 9.5, 8.8, 13.2, 14.1];
END_VAR

fbProc(aInput := aSensors, rThreshold := 70.0);
```

---

# 🏆 12. Full Pipeline Özeti

| Adım | Açıklama |
|------|----------|
| 1 | Ham veri alma |
| 2 | Moving average ile filtreleme |
| 3 | Normalize etme |
| 4 | Lookup table ile sensör linearization |
| 5 | Threshold kontrol |
| 6 | İstatistiksel analiz (min–max–avg) |
| 7 | Sonuçları struct içine aktarma |

Bu pipeline, endüstriyel veri işleme için **çok güçlü, ölçeklenebilir ve gerçek dünya ile birebir uyumlu** bir yapıdır.

---



