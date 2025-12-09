# 📘 TwinCAT – ARRAY OF ARRAYS (İç İçe Diziler)
### *Tam Kapsamlı Eğitim Dokümanı*

---

# 🔍 1. Array of Arrays Nedir?

TwinCAT’te **array of arrays**, çok boyutlu dizilerin alternatif bir gösterimidir.  
Normal 2D array şöyle tanımlanır:

```pascal
aPoints : ARRAY[1..2, 1..3] OF INT;
```

Array of arrays ise bunu iç içe diziler ile ifade eder:

```pascal
a2Boxes : ARRAY[1..2] OF ARRAY[1..3] OF INT;
```

Her eleman başka bir array içerir.  
✔ Daha okunabilir  
✔ Daha esnek  
✔ Her türlü derinlikte iç içe yapı kurabilir

---

# 🧠 2. Neden Array of Arrays Kullanılır?

### ✔ Hiyerarşik veri modellemek için
Örneğin:
- Kutu → raf → kat → bölüm  
- 3D konum bilgisi  
- Depo veya robot haritaları  

### ✔ Initialization (başlangıç) daha düzenlidir
```pascal
[ [1,2,3], [4,5,6] ]
```

### ✔ Çok boyutlu veri analizinde hepsine açıktır

---

# 🔧 3. Sözdizimi (Syntax)

```pascal
<variable> : ARRAY[first_lower..first_upper]
             OF ARRAY[next_lower..next_upper]
             OF <data type>;
```

Örnek:

```pascal
a2D : ARRAY[1..5] OF ARRAY[1..10] OF INT;
```

Bunun eşdeğeri:

```pascal
a2D : ARRAY[1..5, 1..10] OF INT;
```

---

# 📌 4. Veri Erişimi (Data Access)

### Multidimensional array:

```pascal
aPoints[1,2] := 1200;
```

### Array of arrays:

```pascal
a2Boxes[1][2] := 1200;
```

---

# 📦 5. Örnekler

## ✔ Örnek 1 – 2D Array vs Array of Arrays

```pascal
aPoints : ARRAY[1..2,1..3] OF INT := [1,2,3,4,5,6];

a2Boxes : ARRAY[1..2] OF ARRAY[1..3] OF INT := [
    [1, 2, 3],
    [4, 5, 6]
];
```

Erişim:

```pascal
aPoints[1,2] := 1200;
a2Boxes[1][2] := 1200;
```

---

## ✔ Örnek 2 – 3D Array of Arrays

```pascal
a3Boxes : ARRAY[1..2] OF 
          ARRAY[1..3] OF 
          ARRAY[1..4] OF INT := [
    [
        [1, 2, 3, 4],
        [5, 6, 7, 8],
        [9, 10, 11, 12]
    ],
    [
        [13, 14, 15, 16],
        [17, 18, 19, 20],
        [21, 22, 23, 24]
    ]
];
```

Erişim:

```pascal
nValue := a3Boxes[2][3][4];   // 24
```

---

## ✔ Örnek 3 – 4D Gerçek Dünya Kullanımı (Depo Modeli)

```pascal
aWarehouse : ARRAY[1..5] OF 
             ARRAY[1..10] OF 
             ARRAY[1..3] OF 
             ARRAY[1..20] OF INT;
```

Erişim:

```pascal
nItem := aWarehouse[3][7][2][14];
```

Bu:
- 3. Bölüm  
- 7. Raf  
- 2. Kat  
- 14. Kutu  

demektir.

---

# 🎯 6. Avantajlar ve Dezavantajlar

## ✔ Avantajlar
- Daha düzenli veri yapısı  
- Çok boyutlu veriyi rahat modeller  
- Initialization daha okunabilir  
- Hiyerarşik sistemler için ideal  
- Performansı klasik array ile aynıdır  

## ❌ Dezavantajlar
- Çok derine inince okunabilirlik zorlaşabilir  
- Yeni başlayanlar için ilk etapta karışık gelebilir  

---

# 🧲 7. Kullanım Senaryoları

Array of Arrays şu durumlarda özellikle tercih edilir:

- Depo / raf / kutu modellemesi  
- 2D / 3D grid sistemleri  
- Görüntü işleme piksel dizileri  
- Robot alan haritaları  
- Çok katmanlı sistem veri yapıları  
- Parça → alt parça → bileşen modelleri  

---

# 🎉 Özet

Array of arrays:

- Çok boyutlu dizilerin alternatif gösterimidir  
- Her eleman kendi başına bir dizi içerir  
- Initialization daha okunaklıdır  
- Hiyerarşik veri yapıları için mükemmeldir  
- 2D, 3D ve daha derin veri modelleri kurulabilir  
- Performans olarak klasik multidimensional array ile aynıdır  

---



# 📘 TwinCAT – ARRAY OF ARRAYS (Gerçek Dünya Kullanım Örnekleri)
### *Tam Kapsamlı Eğitim Dokümanı (.md)*

Bu doküman, TwinCAT’te **Array of Arrays** yapısının *gerçek endüstriyel senaryolarda* nasıl kullanıldığını açıklayan tam kapsamlı bir eğitim paketidir.  
Her bölümde hem kullanım amacı hem de çalıştırılabilir Structured Text örnekleri bulunmaktadır.

---

# # 🔍 1. Depo / Raf / Kutu Yönetimi (4D Warehouse System)

Gerçek dünya karşılığı:  
- Bölüm (Zone)  
- Raf (Rack)  
- Kat (Level)  
- Kutu (Box)

```pascal
PROGRAM WarehouseDemo
VAR
    aWarehouse : ARRAY[1..5] OF 
                 ARRAY[1..10] OF 
                 ARRAY[1..3] OF 
                 ARRAY[1..20] OF INT;

    nZone      : INT := 3;
    nRack      : INT := 7;
    nLevel     : INT := 2;
    nBox       : INT := 14;

    nItem      : INT;
END_VAR

// Ürün ID’sini kutudan oku
nItem := aWarehouse[nZone][nRack][nLevel][nBox];
```

✔ Depo yönetimi ve lojistik otomasyonunda yaygın kullanılır.

---

# # 🔍 2. Konveyör Ürün Takibi (2D Conveyor Grid Tracking)

Çok bantlı üretim hatlarında ürün konumu takip edilir.

```pascal
PROGRAM ConveyorDemo
VAR
    aConveyor : ARRAY[1..4] OF ARRAY[1..10] OF BOOL; // 4 bant × 10 pozisyon
    bDetected : BOOL;
    nLine     : INT := 2;
    nPos      : INT := 5;
END_VAR

// Sensörden ürün tespit edildi
bDetected := TRUE;

// Ürünü matrise işle
aConveyor[nLine][nPos] := bDetected;
```

✔ Üretim hatlarında ürün konumu izleme için idealdir.

---

# # 🔍 3. Makine Reçete Yönetimi (Recipe Management)

Her kategori için ayrı parametre seti tutulur.

```pascal
TYPE ST_Params :
STRUCT
    Speed    : REAL;
    Pressure : REAL;
    Temp     : REAL;
END_STRUCT
END_TYPE

PROGRAM RecipeDemo
VAR
    aRecipes : ARRAY[1..3] OF ARRAY[1..5] OF ST_Params;
    rSpeed   : REAL;
END_VAR

// 2. kategori, 4. reçetenin hız parametresini oku
rSpeed := aRecipes[2][4].Speed;
```

✔ Plastik enjeksiyon, doldurma, paketleme makinelerinde standarttır.

---

# # 🔍 4. Robotik – Nokta Setleri (Waypoint Lists)

Her robot için farklı waypoint listeleri tutulur.

```pascal
TYPE ST_Point :
STRUCT
    X : LREAL;
    Y : LREAL;
    Z : LREAL;
END_STRUCT
END_TYPE

PROGRAM RobotDemo
VAR
    aRobots : ARRAY[1..3] OF ARRAY[1..10] OF ST_Point;
    ptNext  : ST_Point;
END_VAR

// 1. robotun 5. noktasını al
ptNext := aRobots[1][5];
```

✔ Pick-place robotlarında path planlama için zorunludur.

---

# # 🔍 5. Paketleme İstasyon Zamanlamaları

Her istasyonda birden fazla adımın süresi olabilir.

```pascal
PROGRAM PackagingDemo
VAR
    aTiming : ARRAY[1..4] OF ARRAY[1..6] OF TIME; 
    tSealTime : TIME;
END_VAR

// 2. istasyonun 3. adım süresini oku
tSealTime := aTiming[2][3];
```

✔ Dolum–kapama–etiketleme makinelerinde çok kullanılır.

---

# # 🔍 6. Görüntü İşleme – Piksel Matrisleri (Image Processing Grids)

TwinCAT Vision içeren makinelerde:

```pascal
PROGRAM VisionDemo
VAR
    aImage : ARRAY[0..479] OF ARRAY[0..639] OF BYTE; // 480 × 640 piksel
    nPixel : BYTE;
END_VAR

// 200. satır, 300. sütundaki pikseli oku
nPixel := aImage[200][300];
```

✔ Kalite kontrol, OCR, parça doğrulama gibi tüm vizyon uygulamalarında gerekir.

---

# # 🔍 7. HMI Menü Yapıları (Multi-Level Menu Systems)

HMI içinde menü → alt menü → buton metinleri gibi hiyerarşi saklanır.

```pascal
PROGRAM HMIDemo
VAR
    aMenus : ARRAY[1..4] OF ARRAY[1..8] OF STRING(20);
    sText  : STRING(20);
END_VAR

// 3. menünün 6. buton metni
sText := aMenus[3][6];
```

✔ Büyük HMI projelerinde düzen sağlar.

---

# # 🔍 8. Test İstasyonlarında Sensör Eşik Değerleri

Her ürün tipi için sensör eşikleri saklanır.

```pascal
PROGRAM SensorDemo
VAR
    aThresholds : ARRAY[1..5] OF ARRAY[1..10] OF REAL;
    SensorValue : REAL;
    bAlarm      : BOOL;
END_VAR

IF SensorValue > aThresholds[nProductType][nSensorID] THEN
    bAlarm := TRUE;
END_IF
```

✔ Kalite kontrol istasyonlarında yaygın kullanılır.

---

# # 🎯 TOPARLAMA – Array of Arrays Nerede Kullanılır?

| Endüstri | Kullanım Alanı |
|---------|----------------|
| Depo & Lojistik | Bölüm–raf–kat–kutu modelleme |
| Üretim hatları | Çok bantlı grid takibi |
| Robotik | Waypoint listeleri |
| Makine kontrol | Reçete & parametre yönetimi |
| Paketleme | Çoklu istasyon zamanlama |
| Görüntü işleme | Piksel matrisleri |
| HMI | Menü–buton modelleme |
| Test & kalite kontrol | Sensör eşik tabloları |

✔ Array of Arrays, veri hiyerarşisini koruyarak modelleme konusunda *PLC projelerinin en güçlü araçlarından biridir.*

---


  
