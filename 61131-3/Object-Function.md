# 🧠 SEVİYE 3 ULTRA PROFESYONEL MASTERCLASS  
# **TwinCAT Object Function (FUNCTION) — Derinlemesine İnceleme**

---

## 📌 İçindekiler
1. FUNCTION Nedir?  
2. Bellek Modeli: Stack Değişkenleri  
3. Deterministik Davranış ve Kısıtlamalar  
4. FUNCTION Sözdizimi  
5. FUNCTION Çağrımı (ST, IL, FBD, SFC)  
6. Ek Çıkışlar (VAR_OUTPUT)  
7. Yapısal Tip Döndüren Fonksiyonlar  
8. Reftype (REFERENCE TO) Kullanarak Tek Eleman Erişimi  
9. Gelişmiş Örnekler  
10. Best Practices  

---

# 1. FUNCTION Nedir?

TwinCAT’te **Function (FUN)**, bir POU türüdür ve:

- **Kesinlikle bir adet dönüş değeri üretir**
- **Global durumu değiştiremez**
- **Her çağrıda aynı input → aynı output** üretmek zorundadır (pure function davranışı)
- **Sadece stack üzerinde yaşayan geçici değişkenlere sahiptir**

Bu nedenle fonksiyonlar **deterministik** kabul edilir.

---

# 2. Bellek Modeli: Stack Değişkenleri

Bir fonksiyon içindeki tüm değişkenler:

- Çağrı sırasında stack üzerinde oluşturulur  
- Fonksiyon bitince tamamen silinir  
- Her fonksiyon çağrısında yeniden oluşturulur  

Bu yüzden:

❌ **RETAIN kullanılamaz**  
❌ **Static behavior yoktur**  
❌ **Global erişim ve adres erişimi yasaktır**

---

# 3. Deterministik Davranış ve Kısıtlamalar

TwinCAT, fonksiyonların "matematiksel fonksiyon" gibi çalışmasını ister.

Bu nedenle fonksiyon:

### ❌ Global değişken kullanamaz  
### ❌ AT %I / %Q gibi adresli değişkenlere erişemez  
### ❌ FB state değiştiremez  
### ❌ IO okuma/yazma yapamaz  
### ✔ Yalnızca input parametrelere göre output üretmelidir

Bu özellik kütüphane tasarımında fonksiyonları çok değerli yapar.

---

# 4. FUNCTION Sözdizimi

```st
FUNCTION F_Sample : INT
VAR_INPUT
    x : INT;
    y : INT;
END_VAR
VAR
    temp : INT;
END_VAR

temp := x + y;
F_Sample := temp;
```

Dönüş değeri fonksiyon adıdır → `F_Sample := ...`

---

# 5. FUNCTION Çağrımı

## ST:

```st
nRes := F_Sample(5, 3);
```

## IL:

```il
LD 5
LD 3
F_Sample
ST nRes
```

## FBD:

Bir FBD kutusu içinde fonksiyon node’u olarak kullanılır.

## SFC:

Yalnızca *step action* veya *transition* ifadelerinde çağrılabilir.

---

# 6. Ek Çıkışlar (VAR_OUTPUT)

IEC 61131-3’e göre fonksiyonlarda ek çıktı değişkenleri kullanılabilir.

Örnek fonksiyon:

```st
FUNCTION F_Fun : INT
VAR_INPUT
    nIn1 : INT;
    nIn2 : INT;
END_VAR
VAR_OUTPUT
    nOut1 : INT;
    nOut2 : INT;
END_VAR

nOut1 := nIn1 + 10;
nOut2 := nIn2 + 20;

F_Fun := nIn1 + nIn2;
```

Çağrı:

```st
F_Fun(nIn1 := 1, nIn2 := 2, nOut1 => nLoc1, nOut2 => nLoc2);
```

Bu sayede:

- Fonksiyon *bir ana değer döndürür*
- Ek değerler *referansla output değişkenlere yazılır*

---

# 7. Yapısal Tip Döndüren Fonksiyonlar

Fonksiyonun dönüş tipi:

- STRUCT
- FB
- ARRAY

gibi kompleks tipler olabilir.

```st
FUNCTION F_GetStatus : ST_Status
```

Bu durumda:

```st
status := F_GetStatus();
```

---

# 8. Reftype (REFERENCE TO) ile Tek Eleman Erişimi

TwinCAT normalde:

```st
F_GetStruct().elem
```

şeklinde doğrudan eleman erişimini desteklemez.

Bu engeli aşmak için dönüş tipi **REFERENCE TO** yapılır.

---

## Örnek

### 1. STRUCT Tanımı

```st
TYPE ST_Sample :
STRUCT
    bVar : BOOL;
    nVar : INT;
END_STRUCT
END_TYPE
```

### 2. FB içinde instance:

```st
FUNCTION_BLOCK FB_Sample
VAR
    stLocal : ST_Sample;
END_VAR
```

### 3. Property tanımı:

```st
PROPERTY MyProp : REFERENCE TO ST_Sample
```

### 4. Get methodu:

```st
MyProp REF= stLocal;
```

### 5. Kullanım:

```st
nSingleGet := fbSample.MyProp.nVar;
```

Artık doğrudan erişim mümkün:

✔ `fbSample.MyProp.nVar`  
❌ local kopya oluşturmaya gerek yok

---

# 9. Gelişmiş Örnek – RETURN Yapısal Tip

```st
FUNCTION F_MakePoint : REFERENCE TO ST_Point
VAR
    p : ST_Point;
END_VAR

p.x := 10;
p.y := 20;

F_MakePoint REF= p;
```

Kullanım:

```st
myY := F_MakePoint().y;
```

---

# 10. Best Practices

### ✔ Fonksiyonları saf (pure) tutun  
Yalnızca input → output ilişkisi olmalı.

### ✔ Global veya I/O erişimi kesinlikle kaçının  
Fonksiyon bir "matematik operatörü" gibi davranmalıdır.

### ✔ FB yerine FUNCTION ne zaman kullanılmalı?
- Hesaplama
- Veri dönüşümü
- Statik durumu olmayan işlemler

### ✔ "REFERENCE TO" ile performansı artırabilirsiniz  
Büyük struct’lar kopyalanmadan erişilebilir.

### ✔ Ek çıkışlar (VAR_OUTPUT) ile fonksiyon esnekliği artırılabilir  

---

# 📌 Sonuç

TwinCAT fonksiyonları:

- Deterministik davranışı garanti eder
- Sadece stack değişkenleri kullanır
- Global state'i etkileyemez
- Yüksek performanslıdır
- Kütüphane geliştirmede kritik önem taşır

Bu masterclass, TwinCAT FUNCTION yapısının tüm profesyonel kullanım detaylarını kapsar.

---

