# TwinCAT REFERENCE (Referans) Eğitimi  


---

## 1️⃣ REFERENCE Nedir?

REFERENCE (Referans), başka bir değişkenin **adresini** tutan özel bir değişkendir.  
Bir referansı kullandığında otomatik olarak gösterdiği değere erişirsin.  
Pointer gibi `^` operatörüne gerek yoktur.

---

## 2️⃣ Basit Mantık

```pascal
VAR
    refInt : REFERENCE TO INT;
    nA     : INT;
END_VAR
```

Atama:

```pascal
refInt REF= nA;
```

Anlamı:  
👉 refInt artık nA değişkenini gösteriyor.

---

## 3️⃣ REF= ve := Farkı

### ✔ REF= → Adres atama
Referans hangi değişkeni gösterecekse REF= ile belirtilir.

### ✔ := → Değer atama
Değişkenin değeri, referansın gösterdiği yere yazılır.

Örnek:

```pascal
refA REF= stA;   // adres atama
refA := stB;     // değer atama
```

---

## 4️⃣ Örnek Kullanım

```pascal
refInt REF= nA;  // refInt artık nA'yı gösteriyor
refInt := 12;    // nA = 12
nB := refInt * 2; // nB = 24
refInt REF= nB;   // artık nB'yi gösteriyor
refInt := nA / 2; // nB = 6
```

---

## 5️⃣ Geçersiz Referans (0)

```pascal
refInt REF= 0;
```

Bu referans artık hiçbir şeyi göstermez.

Kontrol:

```pascal
bTest := __ISVALIDREF(refInt);
```

---

## 6️⃣ Geçersiz (Yasak) Tanımlar

```pascal
ARRAY OF REFERENCE TO INT;   // yasak
POINTER TO REFERENCE;         // yasak
REFERENCE TO REFERENCE;       // yasak
REFERENCE TO BIT;             // yasak
```

---

## 7️⃣ VAR_IN_OUT ile REFERENCE Farkı

`VAR_IN_OUT` fonksiyon/method girişleri için kullanılan özel bir referanstır.

Avantajları:
- Daima geçerli olmak zorunda
- Stack değişkenleri atanabilir
- CONSTANT (salt okunur) olabilir

Bu nedenle, mümkünse VAR_IN_OUT önerilir.

---

## 8️⃣ Fonksiyon Bloğunda REFERENCE Kullanımı

```pascal
FUNCTION_BLOCK FB_Sample
VAR_INPUT
    refInput1 : REFERENCE TO INT;
    refInput2 : REFERENCE TO INT;
END_VAR
```

Atama:

```pascal
fbSample.refInput1 REF= n1;
fbSample(refInput2 := n2);
```

---

## 9️⃣ Online Change Sırasında Otomatik Güncelleme

TwinCAT, online change sırasında referansların gösterdiği değişkenleri **otomatik günceller**  
(bellek yeri değişse bile referans bozulmaz).

---

## 📌 Özet Tablosu

| Konu | Açıklama |
|------|----------|
| REFERANS | Bir değişkenin aliası / takma adı |
| REF= | Adres atar |
| := | Değer atar |
| __ISVALIDREF | Referans geçerli mi kontrol eder |
| REFERENCE 0 olabilir | Bu durumda hiçbir şeyi göstermez |
| VAR_IN_OUT | Güvenli ve önerilen giriş referansı |

---

# 🌍 TwinCAT REFERENCE – Gerçek Dünya Örnek Kodları

Bu doküman TwinCAT’te **REFERENCE (Referans)** kullanımını gerçek senaryolar üzerinden öğretmek için hazırlanmıştır.

---

## 1️⃣ Sensör Seçimi – Dinamik Alias Kullanımı

### **Senaryo**
- Normal mod → `sensor1` kullanılacak  
- Bakım modu → `sensor2` kullanılacak  
- Algoritma hangi sensörün kullanıldığını bilmeyecek → sadece `refSensor` üzerinden okuyacak.

### **Kod**

```pascal
VAR
    refSensor : REFERENCE TO INT;
    sensor1   : INT;   
    sensor2   : INT;   
    bMaintenanceMode : BOOL;
END_VAR

IF bMaintenanceMode THEN
    refSensor REF= sensor2;   
Else
    refSensor REF= sensor1;   
END_IF;

// tüm hesaplar referans üzerinden yapılır
nFilteredValue := refSensor * 10;
```

---

## 2️⃣ Motor Kontrolünde Dinamik Referans Seçimi

### **Senaryo**
Bir makinede birden fazla motor var. Aynı FB tüm motorlar için kullanılacak.  
FB her çağrıldığında hangi motor üzerinde çalışacağını `REFERENCE` belirleyecek.

### **FB Motor Kontrol**

```pascal
FUNCTION_BLOCK FB_MotorControl
VAR_INPUT
    refSpeed     : REFERENCE TO INT;
    refDirection : REFERENCE TO BOOL;
END_VAR
VAR
    nCurrentSpeed : INT;
END_VAR

nCurrentSpeed := refSpeed + 10;

IF refDirection THEN
    // ileri
ELSE
    // geri
END_IF;
```

### **MAIN**

```pascal
VAR
    fbCtrl : FB_MotorControl;

    Motor1_Speed : INT;
    Motor2_Speed : INT;

    Motor1_Dir   : BOOL;
    Motor2_Dir   : BOOL;

    bSelectMotor1 : BOOL;
END_VAR

IF bSelectMotor1 THEN
    fbCtrl.refSpeed     REF= Motor1_Speed;
    fbCtrl.refDirection REF= Motor1_Dir;
ELSE
    fbCtrl.refSpeed     REF= Motor2_Speed;
    fbCtrl.refDirection REF= Motor2_Dir;
END_IF;

fbCtrl();
```

---

## 3️⃣ Alarm Sistemi – Dinamik Değişken Gösterme

### **Senaryo**
Birçok ekipmanın sıcaklığı izleniyor.  
Her birine alarm FB’si bağlamak yerine, tek FB farklı ekipmanlara yönlendiriliyor.

### **FB Alarm**

```pascal
FUNCTION_BLOCK FB_HighTempAlarm
VAR_INPUT
    refTemp : REFERENCE TO REAL;
END_VAR
VAR_OUTPUT
    bAlarm : BOOL;
END_VAR

bAlarm := (refTemp > 80.0);
```

### **MAIN**

```pascal
VAR
    fbAlarm : FB_HighTempAlarm;

    tempBoiler   : REAL;
    tempTank     : REAL;
    bAlarmActive : BOOL;
    bSelectTank  : BOOL;
END_VAR

IF bSelectTank THEN
    fbAlarm.refTemp REF= tempTank;
ELSE
    fbAlarm.refTemp REF= tempBoiler;
END_IF;

fbAlarm();
bAlarmActive := fbAlarm.bAlarm;
```

---

## 4️⃣ HMI Üzerinden Dinamik Parametre Değiştirme

### **Senaryo**
HMI’da kullanıcı bir parametre seçiyor ve tek giriş alanı üzerinden farklı hedeflere yazıyor.

### **Kod**

```pascal
VAR
    refSetValue : REFERENCE TO REAL;

    Motor1_Set  : REAL;
    Motor2_Set  : REAL;
    Oven_Set    : REAL;

    nSelection  : INT;  
    UserInput   : REAL;
END_VAR

CASE nSelection OF
    1: refSetValue REF= Motor1_Set;
    2: refSetValue REF= Motor2_Set;
    3: refSetValue REF= Oven_Set;
END_CASE;

// seçilen hedef değişkene yaz
refSetValue := UserInput;
```

---

## 5️⃣ Debug / Test İçin Referans Kullanımı

### **Amaç**
Aynı algoritmayı farklı test değişkenleri ile denemek.

### **Kod**

```pascal
VAR
    refTest : REFERENCE TO INT;

    testA : INT := 10;
    testB : INT := 50;
END_VAR

refTest REF= testA;
TestAlgorithm(refTest);

refTest REF= testB;
TestAlgorithm(refTest);
```

---

## 🎯 Sonuç

REFERENCE gerçek dünyada:

- Dinamik değişken bağlama  
- Çok motorlu/sensörlü sistemlerde ortak FB kullanımında  
- HMI entegrasyonunda  
- Alarm/izleme senaryolarında  
- Debug/test süreçlerinde  

çok güçlü bir araçtır.

---

# 🚀 TwinCAT REFERENCE – Çarpıcı ve Tam Kapsamlı Gerçek Dünya Örneği

Bu doküman, TwinCAT’te REFERENCE kullanımını **gerçek bir endüstriyel senaryo** üzerinden profesyonel seviyede öğretmek için hazırlanmıştır.  
Amaç: *Bu dokümanı okuduktan sonra REFERENCE hakkında bir daha soru sormana gerek kalmaması.*

---

# # 🌍 Senaryo: 12 Bölgeli Endüstriyel Fırın Kontrol Sistemi

Modern bir endüstriyel fırın **12 ayrı sıcaklık bölgesine** sahiptir.  
Her bölgenin kendi:

- Sıcaklık sensörü  
- Set değeri  
- PID kontrol mekanizması  
- Isıtıcı çıkışı  

vardır.

Normalde bu sistem için 12 ayrı kontrol bloğu yazmak gerekir.  
Ancak REFERENCE sayesinde **tek bir kontrol bloğu**, tüm bölgeleri yönetebilir.

---

# # 1️⃣ Problem (REFERENCE Olmadan)

REFERENCE kullanmasaydık:

### ❌ 12 ayrı PID FB  
### ❌ 12 ayrı alarm FB  
### ❌ 12 ayrı kontrol mantığı  
### ❌ 12 CASE ya da IF bloğu  
### ❌ Büyük, karmaşık ve bakım yapılması zor bir kod  

REFERENCE olmadığı durumda:

```pascal
// 12 farklı motor, sensör, heater için tekrarlanan kodlar
fbZone1(Temp[1], Setpoint[1]);
fbZone2(Temp[2], Setpoint[2]);
fbZone3(Temp[3], Setpoint[3]);
// böyle 12 tane devam eder...
```

Kod şişer, okunamaz hale gelir, bakım kabus olur.

---

# # 2️⃣ Çözüm: REFERENCE ile Tek FB → Sınırsız Bölge

REFERENCE sayesinde:

✔ Tek FB → tüm bölgeler  
✔ Dinamik yönlendirme → çalışma zamanında  
✔ HMI seçimine göre değişken bağlama  

---

# # 3️⃣ Master Zone Control FB (Tüm bölgeler için tek blok)

```pascal
FUNCTION_BLOCK FB_ZoneControl
VAR_INPUT
    refTemp : REFERENCE TO REAL;  // sensör değeri
    refSet  : REFERENCE TO REAL;  // set değeri
END_VAR
VAR_OUTPUT
    heaterOut : REAL;             // heater çıkışı
END_VAR

// Basit PID benzeri kontrol
heaterOut := (refSet - refTemp) * 0.5;
```

Bu FB tüm bölgeler için kullanılabilir çünkü REFERENCE ile hedef runtime’da seçilir.

---

# # 4️⃣ MAIN – 12 Bölgenin Değişkenleri

```pascal
VAR
    fbZoneCtrl : FB_ZoneControl;

    Temp : ARRAY[1..12] OF REAL;
    Setpoint : ARRAY[1..12] OF REAL;
    HeaterOut : ARRAY[1..12] OF REAL;

    SelectedZone : INT := 7; // HMI’dan operatör seçiyor
END_VAR
```

---

# # 5️⃣ Dinamik Bağlantı – REFERENCE’ın Gücü

```pascal
fbZoneCtrl.refTemp REF= Temp[SelectedZone];
fbZoneCtrl.refSet  REF= Setpoint[SelectedZone];

fbZoneCtrl();   // kontrol hesaplanır

HeaterOut[SelectedZone] := fbZoneCtrl.heaterOut;
```

### ✔ Kod aynı  
### ✔ FB aynı  
### ✔ Değişen tek şey → REFERENCE hedefi  

Bu sayede:

- İster 12 bölge olsun  
- İster 120 bölge olsun  

**kod değişmez.**

---

# # 6️⃣ HMI Seçimi ile Bağlantı (Gerçek Dünya Kullanımı)

Operatör HMI’dan “Bölge 7”yi seçtiğinde:

- FB sıcaklık okumasını Temp[7] üzerinden yapar  
- Setpoint[7] değerini kullanır  
- Çıkışı HeaterOut[7] olarak üretir  

FB’nin içinde hiçbir CASE veya IF yoktur.

---

# # 7️⃣ Bu Tasarımın Çarpıcı Faydaları

### ✔ Kod boyutu 10 kat küçülür  
### ✔ Bakım maliyeti aşırı düşer  
### ✔ Fırın genişletilirse (12 → 20 bölge)  
→ Kodda **tek satır** bile değişmez  
### ✔ Hata yapma ihtimali minimuma iner  
### ✔ Tasarım tamamen modüler hale gelir  

---

# # 8️⃣ REFERENCE’ın Çekirdeği (Kritik Mantık)

REFERENCE’ın felsefesi:

> "Kodunu değiştirmeden, çalışırken hangi değişkenle çalışacağını seç."

Bu zihniyet:

- Dinamik bağlama
- Abstraction
- Modüler yapı
- Tekrarsız (DRY) kod

kültürünün temelidir.

---

# # 9️⃣ __ISVALIDREF Kullanımı

Eğer yanlışlıkla geçersiz bir bölge seçilirse:

```pascal
IF NOT __ISVALIDREF(fbZoneCtrl.refTemp) THEN
    Error := TRUE;
END_IF;
```

Gerçek sistemlerde güvenlik için kritik bir noktadır.

---

# # 🔟 Uzmanlık Seviyesi Genişletme (İsteğe bağlı)

REFERENCE bu senaryoda:

- Motion Control  
- Recipe yönetimi  
- Alarm routing  
- Modbus / OPC değişken yönlendirme  

gibi sistemlerle entegre edilebilir.

Dilersen bu senaryoyu ileri seviye örneklerle zenginleştirebilirim.

---

# # 🎯 Ultimate Özet

**REFERENCE = Tek kod → sınırsız cihaz yönetimi.**

Bu örnekte:
- 12 bölgeli fırın  
- Tek FB  
- Dinamik referans bağlama  

gerçek hayatta REFERENCE’ın neden vazgeçilmez olduğunu gösterir.

---






