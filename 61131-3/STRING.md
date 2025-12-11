# 🧠 SEVİYE 3 ULTRA PROFESYONEL MASTERCLASS  
# **STRING — TwinCAT İçin Genişletilmiş Karakter Dizisi Mimarisi**

---

## 📌 İçindekiler
1. STRING Nedir?  
2. TwinCAT STRING Bellek Modeli  
3. Latin-1 ve UTF-8 Arasındaki Temel Farklar  
4. Depolama Boyutu — Byte vs Character  
5. Varsayılan Uzunluk & Null-Termination  
6. Initialization Davranışının İncelikleri  
7. Truncation (Kesilme) Davranışları  
8. UTF-8 Multi-Byte Karakterlerin Etkisi  
9. TcEncoding Attribute  
10. STRING Fonksiyonlarının Kısıtları (1–255 Byte)  
11. Güvenlik Riskleri  
12. Endüstriyel Kullanım Önerileri  
13. Kod Örnekleri  
14. Sonuç  

---

## 1. STRING Nedir?

TwinCAT'te STRING veri tipi, iki farklı biçimde yorumlanabilir:

- **Latin‑1 (ISO‑8859‑1)**
- **UTF‑8**

Aynı veri tipi, encoding ayarına bağlı olarak tamamen farklı davranış gösterebilir.

STRING temel olarak:

- Null‑terminated (C‑style) bir karakter dizisidir.  
- Uzunluğu **byte** ile ifade edilir.  
- İçerdiği karakter sayısı encoding'e göre değişir.

---

## 2. TwinCAT STRING Bellek Modeli

Her STRING değişkeni için:

```
N byte → içerik
1 byte → NULL terminator (0)
```

Örnek:

```
sVar : STRING(80);
```

Bu durumda bellekte:

- 80 byte → içerik
- 1 byte → final zero  
Toplam: **81 byte**

TwinCAT, STRING uzunluğunu **byte bazında** değerlendirir.

---

## 3. Latin-1 ve UTF-8 Arasındaki Temel Farklar

| Özellik | Latin‑1 | UTF‑8 |
|--------|----------|--------|
| Byte / karakter | 1:1 | 1–4 byte |
| ASCII uyumluluğu | ✔ | ✔ |
| Karakter seti | Sınırlı | Çok geniş |
| STRING fonksiyon uyumluluğu | Tümü | Sadece UTF‑8 için özel fonksiyonlar kullanılmalı |
| Bellek kullanımı | Sabit | İçerik karakterlerine göre değişken |

Latin‑1 → Basit, hızlı, tek byte karakterler  
UTF‑8 → Uluslararası karakter desteği, çok byte'lı yapı

---

## 4. Depolama Boyutu — Byte vs Character

**STRING(46)** → 46 byte içerik alanı ayırır.

Latin‑1’de:  
✔ 46 karakter depolar.

UTF‑8’de:  
✔ Maksimum 46 byte depolar  
❗ Bu 46 karakter demek değildir.

Örneğin:

- “A” (ASCII) → 1 byte  
- “Ö” (UTF‑8) → 2 byte  
- “😊” → 4 byte  

---

## 5. Varsayılan Uzunluk & Null‑Termination

Uzunluk belirtilmezse:

```
sVar : STRING;
```

→ TwinCAT otomatik olarak **80 byte + 1 terminatör** ayırır.

TwinCAT tüm STRING’leri **final zero** ile sonlandırır.

---

## 6. Initialization Davranışının İncelikleri

TwinCAT, STRING reset edildiğinde:

- Sadece **ilk değer kadar byte**’ı doldurur.  
- Kalan alanı sıfırlamaz.

Örnek:

```st
sVar : STRING(20) := 'ABC';
```

Bellek görüntüsü reset sonrası:

```
41 42 43 00 [ESKİ BELLEK İÇERİĞİ DEVAM EDER]
```

Bu önemli bir güvenlik konusudur.

---

## 7. Truncation (Kesilme) Davranışları

Literal uzunluğu kapasiteyi aşarsa TwinCAT:

- Sağdan keser  
- Hata vermez  

Örnek:

```st
sVar : STRING(10) := 'This text is too long';
```

Belleğe yazılan değer:

```
"This text"
```

---

## 8. UTF‑8 Multi‑Byte Karakterlerin Etkisi

UTF‑8 encoding aktifken:

```
STRING(10)
```

→ En fazla 10 byte depolar.

Bu şu olabilir:

- 10 ASCII karakter  
- 5 adet 2‑byte karakter  
- 2 adet 4‑byte karakter + 1 ASCII  
- 1 adet 4 byte + 3 adet 2 byte + 1 ASCII  

Yani karakter sayısı, kullanılan karakterlere göre değişir.

---

## 9. TcEncoding Attribute

UTF‑8 string tanımı:

```st
{attribute 'TcEncoding' := 'UTF-8'}
sVar : STRING(46);
```

Bu attribute:

- Monitoring’de UTF‑8 gösterimi sağlar  
- Dump / hafıza görünümünü UTF‑8 formatında gösterir  

Ancak şu gerçek değişmez:

> STRING uzunluğu hâlâ BYTE olarak hesaplanır.

UTF‑8 için **özel fonksiyonlar** kullanılmalıdır:

- UTF8Len()
- UTF8Left()
- UTF8Mid()
- UTF8Find()
- UTF8Delete()

---

## 10. STRING Fonksiyonlarının Kısıtları (1–255 Byte)

TwinCAT STRING fonksiyonlarının ortak sınırı vardır:

✔ Maksimum işlenebilir uzunluk → **255 byte**

Örnek:

- LEN()
- LEFT()
- RIGHT()
- CONCAT()
- FIND()
- MID()

Bu nedenle büyük buffer'lar için:

- BYTE array kullan  
- MEM utils (MEMCPY, MEMSET vs.) tercih et  

---

## 11. Güvenlik Riskleri

### 🔹 1. Bellek taşması (overflow)
Literal boyutu büyükse sağdan kesilir → beklenmeyen veri kaybı.

### 🔹 2. Reset sonrası “shadow memory”
TwinCAT string’in gerisini temizlemez → önceki veriler bellek içinde kalır.

Bu, özellikle:

- Log string’lerinde  
- Güvenlik verilerinde  
- HMI input’larında  

kritiktir.

### 🔹 3. UTF‑8 karakterinin yarısının kesilmesi
Çok byte’lı karakter tam kesilirse string bozulur.

---

## 12. Endüstriyel Kullanım Önerileri

✔ OPC UA, MQTT ve JSON iletişimlerinde UTF‑8 kullan  
✔ Fieldbus (ASCII tabanlı) sistemlerde Latin‑1 uygun  
✔ Usulsüz data kesintileri için UTF‑8 fonksiyonları tercih et  
✔ Büyük veri için STRING yerine BYTE array kullan  
✔ Güvenlik kritikli projelerde STRING yerine sabit uzunluklu BYTE buffer önerilir  

---

## 13. Kod Örnekleri

### Latin‑1 STRING

```st
sText : STRING(46) := 'This is a string with memory for 46 characters.';
```

### UTF‑8 STRING

```st
{attribute 'TcEncoding' := 'UTF-8'}
sUtf8 : STRING(46) := 'Türkçe karakterler: ÇĞŞİÜÖ';
```

### UTF‑8 uzunluk ölçümü

```st
nChars := UTF8Len(sUtf8);
```

---

## 14. Sonuç

STRING veri tipi:

- Byte‑tabanlıdır  
- Latin‑1 veya UTF‑8 olarak çalışabilir  
- Multi‑byte karakterlerde sınırlara dikkat ister  
- Reset sonrası bellek temizliği yapmaz  
- TwinCAT STRING fonksiyonlarının 255‑byte sınırı vardır  
- Endüstriyel metin işleme için güçlü ancak dikkat gerektiren bir veri tipidir  

Bu masterclass, STRING veri tipinin TwinCAT mühendisliği açısından tüm kritik detaylarını kapsar.

---
