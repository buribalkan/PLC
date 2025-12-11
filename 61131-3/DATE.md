# 🧠 SEVİYE 3 ULTRA PROFESYONEL MASTERCLASS  
# **TwinCAT DATE, DATE_AND_TIME (DT), TIME_OF_DAY (TOD)  
ve LDATE, LDATE_AND_TIME (LDT), LTIME_OF_DAY (LTOD) Veri Tipleri**  
### (UDINT & ULINT Tabanlı 32‑bit / 64‑bit Zaman Mimarisi)

---

## 📌 İçindekiler
1. Giriş  
2. 32‑bit Zaman Tipleri (DATE, DT, TOD → UDINT)  
3. 64‑bit Zaman Tipleri (LDATE, LDT, LTOD → ULINT)  
4. Epoch (1970) ve Unix Time ilişkisi  
5. Veri tipi karşılaştırmaları  
6. Zaman çözünürlüğü farkları  
7. Tarih üst sınırlarının hesaplanması  
8. Reset & runtime davranışı  
9. Zaman literal sözdizimi  
10. Endüstriyel kullanım alanları  
11. Kod örnekleri  
12. Profesyonel öneriler  
13. Sonuç  

---

# 1. Giriş

TwinCAT, tüm tarih & zaman veri tiplerini **UDINT (32‑bit)** veya **ULINT (64‑bit)** olarak saklar.  
Bu veri tipleri dışarıdan DATE, DT, TOD gibi görünse de içeride sayısal zaman damgası (timestamp) olarak işlem görür.

32‑bit zaman tipleri:  
- **DATE**  
- **DATE_AND_TIME (DT)**  
- **TIME_OF_DAY (TOD)**  

64‑bit uzun zaman tipleri (TwinCAT 3.1.4026+):  
- **LDATE**  
- **LDATE_AND_TIME (LDT)**  
- **LTIME_OF_DAY (LTOD)**

---

# 2. 32‑bit Zaman Tipleri (UDINT Tabanlı)

Bu tipler maksimum **4294967295 saniye** (2³²‑1) kapasitelidir.

## 📌 DATE
- Dahili çözünürlük: **saniye**  
- Gösterim: YYYY‑MM‑DD  
- Aralık: **1970‑01‑01 → 2106‑02‑07**

Örnek:
```st
dDate : DATE := DATE#2024-02-07;
```

---

## 📌 DATE_AND_TIME (DT)
- Hem tarih hem zaman içerir  
- Çözünürlük: **1 saniye**  
- Üst limit: **2106‑02‑07‑06:28:15**

Örnek:
```st
dtEvent : DT := DT#2024-02-07-12:55:01.234;
```

(Not: Milisaniye gösterilse de dahili çözünürlük saniyedir. Milisaniye dışarıdan format içindir.)

---

## 📌 TIME_OF_DAY (TOD)
- Gün içi zaman  
- Çözünürlük: **milisaniye**  
- Aralık: **00:00:00.000 → 23:59:59.999**

Örnek:
```st
todNow : TOD := TOD#12:03:04.567;
```

---

# 3. 64‑bit Zaman Tipleri (ULINT Tabanlı)

Bu tipler **nanosecond resolution** sunar ve modern motion/robotics için gereklidir.

## 📌 LDATE
- Dahili çözünürlük: **ns**, gösterimi sadece gün  
- Aralık: **1970‑01‑01 → 2554‑07‑21**

Örnek:
```st
ldDate : LDATE := LDATE#2024-02-07;
```

---

## 📌 LDATE_AND_TIME (LDT)
- Tam timestamp  
- Nanosecond çözünürlük  
- Üst limit: **2554‑07‑21‑23:34:33.709551615**

Örnek:
```st
ldtEvent : LDT := LDT#2024-02-07-12:55:01.234567891;
```

---

## 📌 LTIME_OF_DAY (LTOD)
- Gün içi zaman  
- Çözünürlük: **nanosecond**  
- Aralık: **00:00:00.000000000 → 23:59:59.999999999**

Örnek:
```st
ltodNow : LTOD := LTOD#12:03:04.567890123;
```

---

# 4. Epoch (1970) ve Unix Time İlişkisi

Tüm 32‑bit zaman tipleri **Unix epoch (1970‑01‑01)** tabanlıdır.  
Fark:  
Unix time → signed 32‑bit → 2038'de taşar  
TwinCAT DATE/DATETIME → unsigned 32‑bit → **2106’ya kadar taşma yapmaz**

64‑bit zaman tipleri (ULINT) → pratikte taşması imkansıza yakındır.

---

# 5. Veri Tipi Karşılaştırmaları

| Tip | Gösterim | Dahili Depolama | Çözünürlük | Tipik Kullanım |
|-----|----------|------------------|-------------|----------------|
| DATE | YYYY‑MM‑DD | UDINT | 1 s | Tarih |
| DT | Timestamp | UDINT | 1 s | Log, event kayıtları |
| TOD | HH:MM:SS.MMM | UDINT | 1 ms | Saat, scheduler |
| LDATE | YYYY‑MM‑DD | ULINT | 1 ns | Uzun süreli tarih ölçümü |
| LDT | Timestamp | ULINT | 1 ns | High‑precision logging |
| LTOD | Gün içi zaman | ULINT | 1 ns | Motion timing |

---

# 6. Zaman Çözünürlükleri

| Veri Tipi | Çözünürlük |
|-----------|-------------|
| DATE | Saniye |
| DT | Saniye |
| TOD | Milisaniye |
| LDATE | Nanosecond |
| LDT | Nanosecond |
| LTOD | Nanosecond |

---

# 7. Tarih Üst Sınırları

### UDINT (32‑bit)
```
Max: 4294967295 s → 2106‑02‑07
```

### ULINT (64‑bit)
```
Max: 2^64‑1 ns → 2554‑07‑21
```

---

# 8. Reset & Runtime Davranışı

Tarih-zaman tipleri:

- Online change sırasında değerini koruyabilir  
- Reset warm → değer korunabilir  
- Reset cold/origin → reinitialize edilir  
- PERSISTENT yapılırsa kalıcı olur  

---

# 9. Zaman Literal Sözdizimi

### DATE
```
DATE#2024-2-7
D#2024-2-7
```

### DATETIME
```
DT#2024-2-7-12:55:1.234
DATE_AND_TIME#1970-1-1-0:0:0
```

### TIME_OF_DAY
```
TOD#23:59:59.999
```

### LDATE
```
LDATE#2024-2-7
```

### LDT
```
LDT#2024-2-7-12:55:1.234567891
```

### LTOD
```
LTOD#12:03:04.567890123
```

---

# 10. Endüstriyel Kullanım Alanları

## DATE / DT
- Alarm kayıtları  
- Batch raporlama  
- Event timestamp  
- Scheduler başlangıç tarihleri  

## TOD
- Gün içi çalışma döngüleri  
- Güneş doğuş/batış hesaplamaları  
- HMI saat göstergesi  

## LDATE / LDT / LTOD
- High‑resolution logging  
- Jitter analizi  
- Motion control zaman senkronizasyonu  
- EtherCAT DC tabanlı timestamping  

---

# 11. Kod Örnekleri

```st
VAR
    dLower : DATE := DATE#1970-1-1;
    dUpper : DATE := DATE#2106-2-7;

    dtApp : DT := DT#2024-2-7-12:55:1.234;

    tdApp : TOD := TOD#12:3:4.567;

    ldApp : LDATE := LDATE#2024-2-7;

    ldtApp : LDT := LDT#2024-2-7-12:55:1.234567891;

    ltodApp : LTOD := LTOD#12:3:4.567890123;
END_VAR
```

---

# 12. Profesyonel Öneriler

### ✔ Motion, robotics veya EtherCAT DC kullanıyorsan LDT / LTOD kullan
Nanosecond çözünürlük gereklidir.

### ✔ Log saklama ve raporlama için DT yeterlidir
Saniye çözünürlüğü endüstride çoğu görev için yeterlidir.

### ✔ TOD günlük döngü işlemlerinde idealdir
24‑saat döngülü sayaç mantığıyla çalışır.

### ✔ 2106 sınırında overflow riski olan sistemlerde DATE yerine LDATE kullan
Uzun süreli kullanılacak tesislerde kritik önem taşır.

### ✔ Zaman literal’lerinde gereksiz ms/us/ns eklememeye çalış
Okunabilirlik + performans artar.

---

# 13. Sonuç

TwinCAT tarih & zaman mimarisi:

- **DATE / DT / TOD → UDINT (32‑bit)**  
- **LDATE / LDT / LTOD → ULINT (64‑bit)**  
- L-serisi zaman tipleri nanosecond çözünürlüklüdür  
- DT günlük operasyonlarda yeterlidir  
- LDT yüksek hassasiyet gerektiren modern otomasyon sistemlerinde zorunludur  

Bu masterclass dokümanı TwinCAT tarih & zaman veri tiplerini ileri seviye mühendislik düzeyinde anlaman için tam kapsamlı rehber niteliğindedir.

---

