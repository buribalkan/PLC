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

# 📘 TwinCAT – FULL SENSOR HANDLING PIPELINE  
### *Gerçek Dünya İçin Tam Kapsamlı Mühendislik Eğitimi (Uçtan Uca Sensör İşleme)*  
---

# 🏭 1. Giriş – Sensor Handling Nedir?

Endüstriyel otomasyon sistemlerinde sensörler:

- Gürültülü veri üretir  
- Bazen hatalı çalışır  
- Bazı durumlarda donma (stuck-at value) gösterir  
- Zamanla drift (kayma) oluşur  
- Ölçüm aralıkları doğrusal değildir  
- Saha kablosu kopabilir veya kısa devre olabilir  

Bu nedenle sensörlerden gelen ham veri doğrudan kullanılmaz.  
Gerçek dünyada sensör verisi, **çok katmanlı bir işleme pipeline’ından** geçirilerek güvenli hâle getirilir.

Bu doküman, TwinCAT üzerinde **profesyonel makine mühendisliği standardında** bir *FULL SENSOR HANDLING PIPELINE* öğretir.

---

# 🎯 2. Sensor Pipeline Bileşenleri

Aşağıdaki adımlar uçtan uca bir veri işleme sistemi oluşturur:

1. **Raw Data Acquisition** (Ham veri alma)  
2. **Debounce / Signal Stabilization**  
3. **Clamping & Limiting**  
4. **Scaling**  
5. **Offset + Gain Calibration**  
6. **Lookup Table Linearization**  
7. **Filtering**  
   - Moving Average  
   - EMA (Exponential Moving Average)  
   - Median Filter  
8. **Fault Detection**  
   - Cable break  
   - Sensor short  
   - Stuck-at value  
   - Out-of-Range detection  
9. **Drift Compensation**  
10. **Rate-of-Change (ROC) Safety Check**  
11. **Hysteresis-Based Alarm Logic**  
12. **Structured Output Data Packaging**  

Bu pipeline günümüz PLC yazılımlarında standarttır.

---

# 🧱 3. Sonuç Yapısı – ST_SensorPacket

Tüm işlenmiş veriler tek bir struct içinde tutulur.

```pascal
TYPE ST_SensorPacket :
STRUCT
    RawValue        : LREAL;
    StableValue     : LREAL;
    ScaledValue     : LREAL;
    CalibratedValue : LREAL;
    LinearizedValue : LREAL;
    FilteredValue   : LREAL;

    IsShortCircuit  : BOOL;
    IsCableBreak    : BOOL;
    IsFrozen        : BOOL;
    IsOutOfRange    : BOOL;

    AlarmActive     : BOOL;
    ROC_Exceeded    : BOOL;

    FilterHistory   : ARRAY[1..10] OF LREAL;
END_STRUCT
END_TYPE
```

---

# 🧰 4. Lookup Table Calibration

```pascal
VAR_GLOBAL CONSTANT
    aLUT : ARRAY[0..100] OF LREAL := [
        0.0, 0.4, 0.9, 1.5, 2.2, 3.0 (* ... *)
    ];
END_VAR
```

Linearization, sensörün üreticiden gelen karakteristiğini doğrultmak için kullanılır.

---

# 🔧 5. FULL SENSOR HANDLING FUNCTION BLOCK

Tüm pipeline tek FB içinde uygulanır.

```pascal
FUNCTION_BLOCK FB_SensorHandler
VAR_INPUT
    rInput                   : LREAL;
    rMin                     : LREAL := 4.0;   // 4-20mA sensör alt sınır
    rMax                     : LREAL := 20.0;  // üst sınır
    rScaleMin                : LREAL := 0.0;
    rScaleMax                : LREAL := 100.0;
    rAlarmThreshold          : LREAL := 80.0;
    rROC_Limit               : LREAL := 15.0;
END_VAR

VAR_OUTPUT
    stOut : ST_SensorPacket;
END_VAR

VAR
    rPrevValue : LREAL;
    i          : INT;
END_VAR
```

---

# 🟦 6. Adım 1 – Ham Veri Alma

```pascal
stOut.RawValue := rInput;
```

---

# 🟩 7. Adım 2 – Debounce / Stabilization

Sinyal belirli bir döngü boyunca değişmezse kararlı (stable) kabul edilir.

```pascal
IF ABS(stOut.RawValue - stOut.StableValue) < 0.01 THEN
    // sabit
ELSE
    stOut.StableValue := stOut.RawValue;
END_IF
```

---

# 🟥 8. Adım 3 – Clamping & Limiting

```pascal
IF rInput < rMin THEN
    stOut.IsOutOfRange := TRUE;
    stOut.StableValue := rMin;
ELSIF rInput > rMax THEN
    stOut.IsOutOfRange := TRUE;
    stOut.StableValue := rMax;
END_IF
```

---

# 🟨 9. Adım 4 – Scaling (4–20mA → 0–100 arası)

```pascal
stOut.ScaledValue :=
    (stOut.StableValue - rMin) / (rMax - rMin) * (rScaleMax - rScaleMin)
    + rScaleMin;
```

---

# 🟧 10. Adım 5 – Calibration (Offset + Gain)

```pascal
stOut.CalibratedValue := (stOut.ScaledValue + 0.2) * 1.05; // örnek
```

---

# 🟪 11. Adım 6 – Lookup Table Linearization

```pascal
VAR
    idx : INT;
END_VAR

idx := LIMIT(0, INT(stOut.CalibratedValue), 100);
stOut.LinearizedValue := aLUT[idx];
```

---

# 🔵 12. Adım 7 – Moving Average Filter

```pascal
FOR i := 9 DOWNTO 1 DO
    stOut.FilterHistory[i+1] := stOut.FilterHistory[i];
END_FOR

stOut.FilterHistory[1] := stOut.LinearizedValue;

VAR rSum : LREAL := 0;

FOR i := 1 TO 10 DO
    rSum := rSum + stOut.FilterHistory[i];
END_FOR

stOut.FilteredValue := rSum / 10;
```

---

# 🟤 13. Adım 8 – Sensor Fault Detection

### Kablo kopması (4 mA altına düşmüş)
```pascal
stOut.IsCableBreak := (rInput < 3.5);
```

### Kısa devre (20 mA üstüne çıkmış)
```pascal
stOut.IsShortCircuit := (rInput > 21.0);
```

### Donma (değer uzun süre değişmiyor)

```pascal
stOut.IsFrozen := (ABS(stOut.FilteredValue - rPrevValue) < 0.0001);
rPrevValue := stOut.FilteredValue;
```

---

# 🔺 14. Adım 9 – Rate-of-Change (ROC) Protection

```pascal
IF ABS(stOut.FilteredValue - rPrevValue) > rROC_Limit THEN
    stOut.ROC_Exceeded := TRUE;
END_IF
```

---

# 🚨 15. Adım 10 – Hysteresis’li Alarm

```pascal
IF stOut.FilteredValue > rAlarmThreshold THEN
    stOut.AlarmActive := TRUE;
ELSIF stOut.FilteredValue < (rAlarmThreshold - 5.0) THEN
    stOut.AlarmActive := FALSE;
END_IF
```

---

# 🧪 16. Kullanım Örneği

```pascal
PROGRAM MAIN
VAR
    fbSensor : FB_SensorHandler;
    rRawInput : LREAL := 12.5;
END_VAR

fbSensor(rInput := rRawInput);
```

---

# 🏆 17. Full Sensor Pipeline Özeti

| Adım | Açıklama |
|------|----------|
| 1 | Ham veri alma |
| 2 | Debounce & stabilizasyon |
| 3 | Clamping / limit kontrol |
| 4 | Scaling |
| 5 | Offset + gain calibration |
| 6 | Lookup Table linearization |
| 7 | Filtreleme (MA) |
| 8 | Sensor fault detection |
| 9 | Rate of Change kontrolü |
| 10 | Alarm yönetimi |
| 11 | Çıktı struct paketleme |

Bu doküman, gerçek fabrika ortamlarında kullanılan **endüstri standardı bir sensör işleme pipeline’ıdır**.

---

# 📘 TwinCAT – FULL BATCH OPERATIONS PIPELINE  
### *Gerçek Dünya İçin Tam Kapsamlı Mühendislik Eğitimi (Batch Processing)*  
---

# 🏭 1. Giriş – Batch Operations Nedir?

Endüstriyel üretimde **batch**, belirli bir üretim döngüsünde toplanan verilerin  
tek bir işlem grubu (lot) olarak değerlendirilmesidir.

Batch operasyonları aşağıdakilerde yaygın olarak kullanılır:

- Dolum makineleri  
- Karışım (mixing) sistemleri  
- Gıda–kimya prosesleri  
- Test–kalite kontrol istasyonları  
- Ölçüm toplama sistemleri  
- Enerji ve proses analiz hatları  

Bir batch işleminde tipik olarak:

1. Veri toplanır  
2. Filtrelenir  
3. Hesaplanır  
4. İstatistik üretilir  
5. Limit kontrol yapılır  
6. Batch sonuçları paketlenir  
7. Batch kapanır ve bir yenisi başlar  

Bu doküman, TwinCAT üzerinde **uçtan uca profesyonel batch processing pipeline** oluşturan kapsamlı bir mühendislik setidir.

---

# 🎯 2. Batch Pipeline İçeriği

Bu eğitim, aşağıdaki bileşenleri içerir:

- Batch başlatma / bitirme mekanizması  
- Sensörlerden veri toplama  
- Değer filtreleme (opsiyonel)  
- Batch içi istatistik hesaplama  
- Min / max / avg / std dev  
- Limit & alarm kontrolü  
- Batch sonuçlarını saklama  
- Batch ID yönetimi  
- Zaman damgası oluşturma  
- Batch tamamlama raporu  

---

# 🧱 3. Batch Veri Yapısı – ST_BatchResult

```pascal
TYPE ST_BatchResult :
STRUCT
    BatchID        : UDINT;
    StartTime      : DT;
    EndTime        : DT;

    Count          : UDINT;
    Sum            : LREAL;
    Avg            : LREAL;
    Min            : LREAL;
    Max            : LREAL;
    StdDev         : LREAL;

    HighLimitHits  : UDINT;
    LowLimitHits   : UDINT;

    Completed      : BOOL;
END_STRUCT
END_TYPE
```

---

# 🧱 4. Batch Geçici Hafıza Yapısı – ST_BatchTemp

```pascal
TYPE ST_BatchTemp :
STRUCT
    Values     : ARRAY[1..10000] OF LREAL; // maksimum batch boyutu
    Index      : UDINT;
    Sum        : LREAL;
    SumSq      : LREAL;
END_STRUCT
END_TYPE
```

---

# 🔧 5. FULL BATCH OPERATION FUNCTION BLOCK

```pascal
FUNCTION_BLOCK FB_BatchProcessor
VAR_INPUT
    rInputValue   : LREAL;
    bAddValue     : BOOL;       // batch'e yeni değer ekle
    bStartBatch   : BOOL;       // yeni batch başlat
    bEndBatch     : BOOL;       // batch bitir
    rHighLimit    : LREAL := 90.0;
    rLowLimit     : LREAL := 10.0;
END_VAR

VAR_OUTPUT
    stResult : ST_BatchResult;
END_VAR

VAR
    stTemp : ST_BatchTemp;
END_VAR
```

---

# 🟦 6. Batch Başlatma

```pascal
IF bStartBatch THEN
    stTemp.Index := 0;
    stTemp.Sum   := 0;
    stTemp.SumSq := 0;

    stResult.BatchID := stResult.BatchID + 1;
    stResult.StartTime := DT#1970-01-01-00:00:00 + TOD_TO_DT(TOD());
    stResult.Completed := FALSE;
END_IF
```

---

# 🟩 7. Batch’e Veri Ekleme

```pascal
IF bAddValue THEN
    stTemp.Index := stTemp.Index + 1;
    stTemp.Values[stTemp.Index] := rInputValue;
    stTemp.Sum   := stTemp.Sum + rInputValue;
    stTemp.SumSq := stTemp.SumSq + (rInputValue * rInputValue);

    IF rInputValue > rHighLimit THEN
        stResult.HighLimitHits := stResult.HighLimitHits + 1;
    END_IF
    IF rInputValue < rLowLimit THEN
        stResult.LowLimitHits := stResult.LowLimitHits + 1;
    END_IF
END_IF
```

---

# 🟥 8. Batch Bitirme + İstatistik Hesaplama

```pascal
IF bEndBatch AND stTemp.Index > 0 THEN
    stResult.EndTime := DT#1970-01-01-00:00:00 + TOD_TO_DT(TOD());

    stResult.Count := stTemp.Index;
    stResult.Sum   := stTemp.Sum;
    stResult.Avg   := stTemp.Sum / stTemp.Index;

    // Min / Max hesaplama
    stResult.Min := stTemp.Values[1];
    stResult.Max := stTemp.Values[1];

    FOR i := 2 TO stTemp.Index DO
        IF stTemp.Values[i] < stResult.Min THEN stResult.Min := stTemp.Values[i]; END_IF;
        IF stTemp.Values[i] > stResult.Max THEN stResult.Max := stTemp.Values[i]; END_IF;
    END_FOR

    // Std deviation
    stResult.StdDev :=
        SQRT( (stTemp.SumSq / stTemp.Index) - (stResult.Avg * stResult.Avg) );

    stResult.Completed := TRUE;
END_IF
```

---

# 🔵 9. Kullanım Örneği – MAIN Program

```pascal
PROGRAM MAIN
VAR
    fbBatch : FB_BatchProcessor;
    rSensor : LREAL;
    bNewSample : BOOL;
    bStart : BOOL;
    bEnd   : BOOL;
END_VAR

// sensörden okunan değer
rSensor := 45.3;

// batch başlayacak
bStart := TRUE;

// her döngüde yeni sample ekle
bNewSample := TRUE;

fbBatch(
    rInputValue := rSensor,
    bAddValue   := bNewSample,
    bStartBatch := bStart,
    bEndBatch   := bEnd
);

// batch sonlandır
bEnd := TRUE;
```

---

# 📊 10. Batch Pipeline Özeti

| Adım | Açıklama |
|------|----------|
| 1 | Batch başlatma |
| 2 | Veri toplama |
| 3 | Limit kontrol |
| 4 | Toplam & kareler toplamı |
| 5 | Min / Max bulma |
| 6 | Ortalama hesaplama |
| 7 | Standart sapma hesaplama |
| 8 | Batch kapatma |
| 9 | Sonuç raporunu üretme |

---

# 🧪 11. Gerçek Dünya Kullanım Senaryoları

### ✔ Dolum makineleri batch dolum analizi  
### ✔ Test istasyonlarında batch kalite kontrol  
### ✔ Enerji kayıt sistemlerinde batch ölçüm  
### ✔ Tartım & Loadcell batch operasyonları  
### ✔ ISO 9001 süreçlerinde parti takibi  
### ✔ Pastörizasyon, karıştırma, pişirme prosesleri  

Bu pipeline endüstriyel otomasyon sektöründe standarttır.

---






