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

