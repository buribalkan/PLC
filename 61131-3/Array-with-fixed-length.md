# 📘 TwinCAT – Sabit Uzunluklu Diziler (ARRAY) – Tam Kapsamlı Eğitim Dokümanı

Bu doküman, TwinCAT’te **sabit uzunluklu dizileri (ARRAY)** en temel seviyeden başlayarak çok boyutlu yapılar, STRUCT ve FB içeren diziler, initialization yöntemleri, CheckBounds ve monitoring davranışları ile birlikte **tam kapsamlı** olarak açıklar.

---

# # 🎯 1. ARRAY Nedir?

TwinCAT'te ARRAY, aynı tipte çok sayıda veriyi tek bir isim altında toplamayı sağlayan bir veri yapısıdır.

ARRAY sayesinde:

- Birden fazla veriye **tek isim** üzerinden erişirsin.  
- Her elemana **index** ile ulaşılır.  
- Tüm elemanlar **aynı tiptedir**.

Örnek:
```pascal
aNumbers : ARRAY[0..9] OF INT;
```
Bu dizide **10 adet INT** vardır.

---

# # 🎯 2. Tek Boyutlu ARRAY Tanımı (1D)

## ✔ Söz Dizimi
```pascal
<isim> : ARRAY[<başlangıç> .. <bitiş>] OF <veri tipi>;
```

Örnek (10 elemanlı INT dizi):
```pascal
aCounter : ARRAY[0..9] OF INT;
```

## ✔ Eleman Erişimi
```pascal
nLocal := aCounter[2];   // 3. eleman
```

---

# # 🎯 3. ARRAY Initialization

```pascal
aCounter : ARRAY[0..9] OF INT :=
[0,10,20,30,40,50,60,70,80,90];
```

`aCounter[2] = 20` olur.

---

# # 🎯 4. CONSTANT ile Boyut Tanımı

Boyutlar sabitlerle belirlenebilir.

```pascal
VAR CONSTANT
    cMin : INT := 0;
    cMax : INT := 5;
END_VAR

VAR
    aSample : ARRAY[cMin..cMax] OF BOOL;
END_VAR
```

---

# # 🎯 5. Dizi Sınırı Kontrolü (CheckBounds)

TwinCAT, index erişiminde sınır kontrolü yapabilir.

```pascal
IF nIndex >= cMin AND nIndex <= cMax THEN
    bValue := aSample[nIndex];
END_IF
```

Bu kontrol **runtime hatalarını engeller**.

---

# # 🎯 6. Çok Boyutlu Diziler (2D, 3D…)

TwinCAT birden fazla boyutlu dizileri destekler.

## ✔ 2 Boyutlu ARRAY (2D)

Tanım:
```pascal
aCardGame : ARRAY[1..2, 3..4] OF INT;
```

Short initialization:
```pascal
aCardGame := [2(10), 2(20)];
// Yani: 10,10,20,20
```

Erişim:
```pascal
a := aCardGame[1,3];   // 10
b := aCardGame[2,4];   // 20
```

---

## ✔ 3 Boyutlu ARRAY (3D)

```pascal
aCardGame : ARRAY[1..2, 3..4, 5..6] OF INT;
```

Toplam eleman:
```
2 × 2 × 2 = 8
```

Initialization:
```pascal
aCardGame := [10,20,30,40,50,60,70,80];
```

Erişim:
```pascal
a := aCardGame[1,3,5]; // 10
b := aCardGame[2,4,6]; // 80
```

Short notation:
```pascal
aCardGame := [2(10),2(20),2(30),2(40)];
```

---

# # 🎯 7. STRUCT İçeren Diziler

User-defined data type:

```pascal
TYPE ST_Data :
STRUCT
    n1 : INT;
    n2 : INT;
    n3 : DWORD;
END_STRUCT
END_TYPE
```

3D ARRAY:
```pascal
aData : ARRAY[1..3, 1..3, 1..10] OF ST_Data;
```

Partial initialization:
```pascal
aData :=
[
 (n1:=1, n2:=10, n3:=16#00FF),
 (n1:=2, n2:=20, n3:=16#FF00),
 (n1:=3, n2:=30, n3:=16#FFFF)
];
```

Geri kalan elemanlar **otomatik 0** ile doldurulur.

Erişim:
```pascal
x := aData[1,1,1].n1;  // 1
y := aData[1,1,3].n3;  // 16#FFFF
```

---

# # 🎯 8. ARRAY of Function Block

Dizi elemanları FB instance olabilir.

FB:
```pascal
FUNCTION_BLOCK FB_Object
VAR
    nCounter : INT;
END_VAR
```

Dizi:
```pascal
aObjects : ARRAY[1..4] OF FB_Object;
```

FB çağırma:
```pascal
aObjects[2]();
```

---

# # 🎯 9. FB_init ile Dizilerin Initialize Edilmesi

FB:
```pascal
FUNCTION_BLOCK FB_Sample
VAR
    nId : INT;
    fIn : LREAL;
END_VAR
```

FB_init:
```pascal
METHOD FB_init : BOOL
VAR_INPUT
    bInitRetains : BOOL;
    bInCopyCode : BOOL;
    nIdInit : INT;
    fInInit : LREAL;
END_VAR

nId := nIdInit;
fIn := fInInit;
```

Array initialization:
```pascal
aSample : ARRAY[0..1,0..1] OF FB_Sample
[
 (nId := 12, fIn := 11.22),
 (nId := 13, fIn := 22.33),
 (nId := 14, fIn := 33.55),
 (nId := 15, fIn := 11.22)
];
```

---

# # 🎯 10. Online Monitoring Range (Large Arrays)

TwinCAT online izleme sırasında:

- Varsayılan limit → **1000 eleman**
- Maksimum görüntüleme → **20,000 eleman**

Ayarlamak için:

1. Diziye online modda çift tıkla  
2. Görüntüleme aralığı (start–end index) belirle  

Not: Çok büyük aralık seçilirse IDE yavaşlayabilir.

---

# # 🎯 11. Özet Bilgiler (ARRAY 10 saniyede)

- ARRAY sabit boyutludur  
- Boyutlar CONSTANT ile belirlenebilir  
- 1D, 2D, 3D ve daha fazlası desteklenir  
- STRUCT ve FB array’de kullanılabilir  
- initialization kısa veya tam formatta yapılabilir  
- CheckBounds ile güvenli erişim sağlanır  
- Büyük diziler online görüntülenirken limit uygulanır  

---

# 🚀 TwinCAT ARRAY – Gerçek Dünya Kullanım Örnekleri (Tam Kapsamlı Eğitim Dokümanı)

Bu doküman, TwinCAT’te **ARRAY (sabit uzunluklu diziler)** kullanımına yönelik tüm gerçek dünya senaryolarını kapsayan tam kapsamlı eğitim materyalidir.  
Aşağıdaki örnekler, fabrika otomasyonu, proses kontrol, robotik, paketleme, depolama ve veri işleme sistemlerinde **en çok kullanılan** array uygulamalarıdır.

---

# # 📘 1. Sensör Verisi Kaydetme (Ring Buffer – Historian Mini)

Bir sensörün son 100 okumasını hafızada tutmak için:

```pascal
VAR
    aTempHistory : ARRAY[0..99] OF REAL;   // Son 100 sıcaklık değeri
    nIndex       : INT := 0;
    rTemp        : REAL;
END_VAR

// Her çevrim sıcaklığı kaydet
aTempHistory[nIndex] := rTemp;

// Ring buffer döngüsü
nIndex := nIndex + 1;
IF nIndex > 99 THEN
    nIndex := 0;
END_IF
```

**Kullanım alanları:**  
- PID kontrol optimizasyonu  
- Trend / zayıflama analizi  
- Hata teşhis sistemi  

---

# # 📘 2. Çoklu Motor Yönetimi (20 Motor İçin Tek Kod)

```pascal
VAR
    aMotorRun    : ARRAY[1..20] OF BOOL;
    aMotorError  : ARRAY[1..20] OF BOOL;
    aMotorSpeed  : ARRAY[1..20] OF INT;
    i            : INT;
END_VAR

FOR i := 1 TO 20 DO
    IF aMotorRun[i] THEN
        aMotorSpeed[i] := aMotorSpeed[i] + 1;
    END_IF
END_FOR
```

**Avantajları:**  
✔ Tek döngü → çok ekipman  
✔ Kodu çoğaltma ihtiyacı yok  
✔ Hızlı bakım  

---

# # 📘 3. Konveyörde Ürün Takibi (Shift Register)

Konveyörden geçen ürünlerin pozisyonunu takip etmek için:

```pascal
VAR
    aProduct : ARRAY[0..50] OF BOOL;   // Ürün var/yok bilgisi
    bSensorEntry : BOOL;
    i : INT;
END_VAR

// Diziyi sağa kaydır
FOR i := 50 DOWNTO 1 DO
    aProduct[i] := aProduct[i-1];
END_FOR

// Yeni ürün ekle
aProduct[0] := bSensorEntry;
```

**Gerçek kullanım:**  
- Paketleme makineleri  
- Metal dedektör → Reject sistemleri  
- Etiketleme pozisyon kontrol  

---

# # 📘 4. Reçete (Recipe) Yönetimi – 2D Array

Her ürün tipi için farklı parametreler:

```pascal
VAR
    aRecipe : ARRAY[1..5, 1..10] OF REAL; // 5 ürün × 10 parametre
    nProductID : INT := 3;
    nParamIndex : INT;
    rCurrentParam : REAL;
END_VAR

FOR nParamIndex := 1 TO 10 DO
    rCurrentParam := aRecipe[nProductID, nParamIndex];
END_FOR
```

✔ Ürün çeşitlerini yönetmek çok kolay  
✔ Parametre grupları düzenli  

---

# # 📘 5. 3D Array ile Depo / Raf / Bölme Yönetimi

```pascal
VAR
    aWarehouse : ARRAY[1..3, 1..5, 1..20] OF BOOL; 
    // 3 raf × 5 sütun × 20 bölme
END_VAR

IF aWarehouse[2,4,10] THEN
    // Ürün bu rafta bulunuyor
END_IF
```

**Kullanım:**  
- Otomatik depo sistemleri  
- Shuttle / ASRS çözümleri  

---

# # 📘 6. STRUCT Dizisi ile Tarif Yönetimi

```pascal
TYPE ST_Recipe :
STRUCT
    Temperature : REAL;
    Pressure    : REAL;
    Speed       : INT;
END_STRUCT
END_TYPE

VAR
    aRecipes : ARRAY[1..20] OF ST_Recipe;
END_VAR

aRecipes[5].Temperature := 180.0;
aRecipes[5].Pressure := 2.5;
aRecipes[5].Speed := 1200;
```

✔ 20 tarif bir arada  
✔ Okunabilirlik ve düzen çok yüksek  

---

# # 📘 7. Function Block Dizisi (Çoklu Makine Kontrolü)

Her makine için bir FB varsa ve makine sayısı 50 ise:

```pascal
FUNCTION_BLOCK FB_Motor
VAR
    bRun : BOOL;
    nSpeed : INT;
END_VAR

PROGRAM MAIN
VAR
    aMotors : ARRAY[1..50] OF FB_Motor;
    i : INT;
END_VAR

FOR i := 1 TO 50 DO
    aMotors[i]();   // Her motorun FB'si çalışır
END_FOR
```

✔ Çok makineli tesislerde standart yöntem  
✔ Kod tasarımı ölçeklenebilir  

---

# # 📘 8. Hata Kodları Yönetimi

```pascal
VAR
    aErrorCodes : ARRAY[1..10] OF INT := [101,102,103,104,105,201,202,301,302,400];
    i : INT;
END_VAR

FOR i := 1 TO 10 DO
    IF aErrorCodes[i] = 201 THEN
        // Kritik hata işlemleri
    END_IF
END_FOR
```

---

# # 📘 9. Menü / HMI Seçenekleri (STRING Array)

```pascal
VAR
    aMenu : ARRAY[1..4] OF STRING := ['Start','Stop','Settings','Exit'];
END_VAR
```

**Kullanım alanları:**  
- HMI menüleri  
- Alarm grupları  
- Reçete isimleri  

---

# # 📘 10. Batching / Dosing Sisteminde Adım Yönetimi

```pascal
VAR
    aStepWeight : ARRAY[1..10] OF REAL; // Her adımın hedef ağırlığı
    aStepTime   : ARRAY[1..10] OF TIME; // Her adımın hedef süresi
END_VAR
```

✔ Dosing, karışım, batch süreçlerinde standarttır.

---

# # 📘 11. Analog Giriş Filtreleme (Moving Average)

```pascal
VAR
    aSamples : ARRAY[0..9] OF REAL;
    nIdx     : INT := 0;
    i        : INT;
    rAvg     : REAL := 0.0;
END_VAR

// Yeni veri ekle
aSamples[nIdx] := AI_Temperature;

nIdx := nIdx + 1;
IF nIdx > 9 THEN nIdx := 0; END_IF;

// Ortalama hesapla
FOR i := 0 TO 9 DO
    rAvg := rAvg + aSamples[i];
END_FOR

rAvg := rAvg / 10.0;
```

✔ Gürültü azaltmak  
✔ PID girişini yumuşatmak  

---

# # 📘 12. Zaman Damgası + Değer Kaydı (STRUCT ARRAY Logger)

```pascal
TYPE ST_Log :
STRUCT
    Timestamp : DT;
    Value     : REAL;
END_STRUCT
END_TYPE

VAR
    aLog : ARRAY[1..100] OF ST_Log;
    nLogIndex : INT := 1;
END_VAR

aLog[nLogIndex].Timestamp := CURRENT_DT();
aLog[nLogIndex].Value := rSensorValue;

nLogIndex := nLogIndex + 1;
IF nLogIndex > 100 THEN nLogIndex := 1; END_IF;
```

---

# # 🎉 Özet

Bu dokümanda ARRAY'lerin gerçek hayatta en sık kullanıldığı alanlar işlendi:

✔ Sensör historisi  
✔ Konveyör ürün takibi  
✔ Çoklu motor kontrolü  
✔ Reçete yönetimi  
✔ Depo raf sistemi  
✔ STRUCT dizileri  
✔ Function block dizileri  
✔ Filtreleme / Ortalama alma  
✔ Logger sistemleri  

TwinCAT’te array’ler **modern PLC mimarisinin temelidir**.

---

# 📘 TwinCAT ARRAY Formülleri – Tam Kapsamlı Eğitim Dokümanı

Bu doküman, TwinCAT'te ARRAY’ler ile kullanılan **tüm yaygın matematiksel, filtreleme, arama ve veri işleme formüllerini** profesyonel ve tam kapsamlı şekilde açıklar.  
Gerçek dünya uygulamalarında kullanılan tüm örnekler dahildir.

---

# 🚀 1. ARRAY Toplamı (Sum)

```pascal
rSum := 0.0;

FOR i := 0 TO nMax DO
    rSum := rSum + aData[i];
END_FOR
```

---

# 🚀 2. Ortalama (Arithmetic Mean)

```pascal
rSum := 0.0;

FOR i := 0 TO nMax DO
    rSum := rSum + aData[i];
END_FOR

rAvg := rSum / (nMax + 1);
```

---

# 🚀 3. Maksimum Değer (MAX)

```pascal
rMax := aData[0];

FOR i := 1 TO nMax DO
    IF aData[i] > rMax THEN
        rMax := aData[i];
    END_IF
END_FOR
```

---

# 🚀 4. Minimum Değer (MIN)

```pascal
rMin := aData[0];

FOR i := 1 TO nMax DO
    IF aData[i] < rMin THEN
        rMin := aData[i];
    END_IF
END_FOR
```

---

# 🚀 5. Hareketli Ortalama (Moving Average)

```pascal
aSamples[nIdx] := rNewValue;

nIdx := nIdx + 1;
IF nIdx > nMax THEN nIdx := 0; END_IF;

rSum := 0.0;

FOR i := 0 TO nMax DO
    rSum := rSum + aSamples[i];
END_FOR

rFiltered := rSum / (nMax + 1);
```

---

# 🚀 6. Ağırlıklı Ortalama (Weighted Average)

```pascal
rWeightedSum := 0.0;
rWeightTotal := 0.0;

FOR i := 0 TO nMax DO
    rWeightedSum := rWeightedSum + (aData[i] * aWeights[i]);
    rWeightTotal := rWeightTotal + aWeights[i];
END_FOR

rResult := rWeightedSum / rWeightTotal;
```

---

# 🚀 7. Varyans (Variance) ve Standart Sapma (Standard Deviation)

### Ortalama
```pascal
rSum := 0;

FOR i := 0 TO nMax DO
    rSum := rSum + aData[i];
END_FOR

rAvg := rSum / (nMax + 1);
```

### Varyans
```pascal
rVariance := 0;

FOR i := 0 TO nMax DO
    rVariance := rVariance + (aData[i] - rAvg) * (aData[i] - rAvg);
END_FOR

rVariance := rVariance / (nMax + 1);
```

### Standart Sapma
```pascal
rStdDev := SQRT(rVariance);
```

---

# 🚀 8. Lookup Table (TABLO ARAMASI)

```pascal
rOutput := aLookup[nIndex];
```

---

# 🚀 9. Dizi Arama (Find Element)

```pascal
bFound := FALSE;

FOR i := 0 TO nMax DO
    IF aData[i] = value THEN
        bFound := TRUE;
        EXIT;
    END_IF
END_FOR
```

---

# 🚀 10. Diziyi Temizleme (Reset)

```pascal
FOR i := 0 TO nMax DO
    aData[i] := 0;
END_FOR
```

---

# 🚀 11. Shift Register (Sağa Kaydırma)

```pascal
FOR i := nMax DOWNTO 1 DO
    aData[i] := aData[i-1];
END_FOR
```

---

# 🚀 12. Shift Left (Sola Kaydırma)

```pascal
FOR i := 0 TO nMax-1 DO
    aData[i] := aData[i+1];
END_FOR
```

---

# 🚀 13. Reverse Array (Tersine Çevirme)

```pascal
FOR i := 0 TO nMax DO
    aTemp[i] := aData[nMax - i];
END_FOR
```

---

# 🚀 14. Dizi Kopyalama (Copy Array)

```pascal
FOR i := 0 TO nMax DO
    aDest[i] := aSource[i];
END_FOR
```

---

# 🚀 15. Normalize Etme (0–1 Aralığına Ölçekleme)

```pascal
rMin := 999999;
rMax := -999999;

// Min-Max bul
FOR i := 0 TO nMax DO
    IF aData[i] < rMin THEN rMin := aData[i]; END_IF
    IF aData[i] > rMax THEN rMax := aData[i]; END_IF
END_FOR

// Normalize et
FOR i := 0 TO nMax DO
    aNorm[i] := (aData[i] - rMin) / (rMax - rMin);
END_FOR
```

---

# 🚀 16. 2D Matris Toplamı

```pascal
rSum := 0;

FOR x := 1 TO nRows DO
    FOR y := 1 TO nCols DO
        rSum := rSum + aMatrix[x,y];
    END_FOR
END_FOR
```

---

# 🚀 17. 2D Satır Toplamı

```pascal
rRowSum := 0;

FOR y := 1 TO nCols DO
    rRowSum := rRowSum + aMatrix[nRow, y];
END_FOR
```

---

# 🚀 18. 2D Sütun Toplamı

```pascal
rColSum := 0;

FOR x := 1 TO nRows DO
    rColSum := rColSum + aMatrix[x, nCol];
END_FOR
```

---

# 🚀 19. En Yakın Değer (Closest Value)

```pascal
rClosest := aData[0];

FOR i := 1 TO nMax DO
    IF ABS(aData[i] - rTarget) < ABS(rClosest - rTarget) THEN
        rClosest := aData[i];
    END_IF
END_FOR
```

---

# 🚀 20. Threshold Üstü Eleman Sayısı (Threshold Count)

```pascal
nCount := 0;

FOR i := 0 TO nMax DO
    IF aData[i] > rThreshold THEN
        nCount := nCount + 1;
    END_IF
END_FOR
```

---

# 🎉 Özet

Bu doküman ARRAY yapıları üzerinde kullanılan tüm matematiksel ve kontrol algoritmalarını içerir:

- Toplama, ortalama, max/min  
- Moving average, weighted average  
- Variance & std deviation  
- Shift register  
- Lookup table  
- Normalize  
- 2D matris işlemleri  
- Threshold analizleri  

TwinCAT ile veri işleme, kontrol algoritmaları ve endüstriyel sistemlerde ARRAY formülleri kritik rol oynar.

---

# 📘 TwinCAT Lookup Table (LUT) – Tam Kapsamlı Eğitim Dokümanı

Bu doküman, TwinCAT’te **Lookup Table (LUT)** kullanımını sıfır bilgi seviyesinden ileri mühendislik seviyesine kadar eksiksiz şekilde açıklar.  
LUT; sensör linearizasyonundan kontrol algoritmalarına kadar endüstride en çok kullanılan hızlandırma tekniklerinden biridir.

---

# # 🔍 1. Lookup Table (LUT) Nedir?

Lookup Table (LUT), bir fonksiyonun çıktılarını **önceden hesaplayıp bir diziye kaydetme**, çalışma anında ise hesaplama yapmak yerine **tablodan doğrudan okuma** işlemidir.

> ✔ *"Hesaplamak yerine okumak."*  
> Bu sayede PLC CPU yükü büyük oranda azalır.

---

# # 🚀 2. LUT Neden Kullanılır? (Gerçek Dünya Uygulamaları)

### ✔ 1. Yüksek maliyetli hesaplamaları hızlandırmak  
- Sinüs, kosinüs  
- Logaritma  
- Üstel fonksiyon  
- Polinom çözümü  

PLC’de bu işlemler **yavaştır** → LUT **çok hızlıdır**.

---

### ✔ 2. Sensör linearizasyonu  
Gerçek sensörler genelde **doğrusal değildir**.

Örn:  
0–4095 → ADC  
0–250°C → Gerçek sıcaklık

Linearizasyon LUT ile yapılır.

---

### ✔ 3. Kalibrasyon eğrileri  
Üretici tabloları doğrudan LUT’a aktarılır.

---

### ✔ 4. Motor, pompa, servo karakteristik haritaları  
Örn:  
PWM → Tork  
Akım → Hız  
Sıcaklık → Kompanzasyon

---

### ✔ 5. Hızlı kontrol algoritmaları  
PID feedforward (FFWD) tabloları.

---

# # 🔧 3. TwinCAT’te Lookup Table Tanımlama

### Basit LUT:

```pascal
VAR
    aLUT : ARRAY[0..9] OF REAL := [0.0, 1.2, 2.7, 4.5, 6.8, 9.5, 12.8, 16.7, 21.3, 26.6];
END_VAR
```

### Erişim:

```pascal
rOutput := aLUT[nIndex];
```

---

# # 🧩 4. Sensör Linearizasyonu Örneği

Ham sensör okuması (0–4095):

```pascal
rRaw := AI_Temp;
```

Index'e dönüştürme:

```pascal
nIndex := LIMIT(0, INT(rRaw / 410), 9);
```

Table lookup:

```pascal
rTemp := aTempLUT[nIndex];
```

---

# # 🎚 5. Interpolasyonlu Lookup Table (Daha Hassas)

Gerçek dünyada değer LUT noktalarının arasına düşebilir.  
Bu nedenle **lineer interpolasyon** kullanılır.

### Formül:

```
y = y0 + (x - x0) * (y1 - y0) / (x1 - x0)
```

### TwinCAT kodu:

```pascal
nLow  := nIndex;
nHigh := nIndex + 1;

y0 := aLUT[nLow];
y1 := aLUT[nHigh];

rOutput := y0 + (x - nLow) * (y1 - y0);
```

✔ Sensör linearizasyonunda endüstri standardıdır.

---

# # 🧮 6. 2 Boyutlu Lookup Table (2D LUT)

Bazı fonksiyonlar **iki değişkene bağlıdır**:

- Sıcaklık & basınç → Yoğunluk  
- Hız & tork → Güç  
- Açı & yük → Servo komutları  

### Tanım:

```pascal
aMap : ARRAY[0..10, 0..10] OF REAL;
```

### Kullanım:

```pascal
rOutput := aMap[nTempIdx, nPressIdx];
```

---

# # 🧊 7. 3D Lookup Table (3D LUT)

Örneğin motor haritalarında:

- Hız  
- Tork  
- Sıcaklık  

üç farklı eksendir.

### Tanım:

```pascal
aMap3D : ARRAY[0..10, 0..10, 0..10] OF REAL;
```

---

# # ⚡ 8. Neden LUT Çok Hızlıdır? (Performans Analizi)

| İşlem türü          | Göreceli hız |
|---------------------|--------------|
| sin(x)              | Yavaş        |
| log(x)              | Yavaş        |
| karmaşık polinom    | Orta         |
| **LUT lookup**      | **Çok hızlı** |

PLC tarama süresini azaltır → gerçek zamanlı sistemlerde kritik öneme sahiptir.

---

# # 🏎 9. LUT ile PWM Linearization Örneği

Motorlar genelde doğrusal tepki vermez.  
PWM komutu → gerçek tork eğrisi LUT ile düzeltilir.

```pascal
aPWMfix : ARRAY[0..100] OF REAL := [
    0.0, 0.1, 0.4, 0.8, 1.3, ... , 100.0
];

rCorrected := aPWMfix[nPWMpercent];
```

Sonuç → **daha stabil ve hassas motor kontrolü**

---

# # 📉 10. LUT ile Ters Fonksiyon Hesabı

Bazı fonksiyonlar:

- hesaplanması zor  
- tersinin bulunması daha zor  

Örn:  
Seviye → Debi dönüştürme.

Proses mühendisleri eğriyi verir → PLC içinde LUT olur.

---

# # ☑ 11. Lookup Table Kullanırken Dikkat Edilecekler

### ✔ Index mutlaka sınırlandırılmalı

```pascal
nIndex := LIMIT(0, nIndex, 100);
```

### ✔ Büyük LUT RAM tüketir  
Genelde PLC için sorun değildir, ama bilinmelidir.

### ✔ Tablolar revizyon kontrolüne tabi olmalıdır  
Yanlış LUT → yanlış proses.

### ✔ Interpolation doğruluğu artırır  
Özellikle sensörlerde **çok kritik**.

---

# # 🎉 Özet

Lookup Table:

- CPU yükünü azaltır  
- Hız kazandırır  
- Sensör linearizasyonu için mükemmel çözüm sağlar  
- 1D, 2D, 3D tablolarla çok esnek bir yapıdır  
- Interpolation ile yüksek doğruluk elde edilir  

Endüstride LUT en sık kullanılan optimizasyon tekniklerindendir.

---

# 📁 Dosya Bilgisi
Bu dosya TwinCAT eğitim setine uygun olarak hazırlanmıştır ve diğer modüllerle birebir uyumludur.

---






