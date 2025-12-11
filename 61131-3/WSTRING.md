# 🧠 SEVİYE 3 ULTRA PROFESYONEL MASTERCLASS  
# **WSTRING — TwinCAT UCS-2 GENİŞ KARAKTER DİZİSİ MİMARİSİ**

---

## 📌 İçindekiler
1. WSTRING Nedir?  
2. UCS-2 Kodlaması  
3. Bellek Modeli: WORD Bazlı Depolama  
4. Karakter Sayısı vs Byte Sayısı  
5. Varsayılan Boyut ve Final Zero WORD  
6. Reset & Initialization Bellek Davranışı  
7. STRING / UTF-8 / WSTRING Karşılaştırması  
8. UCS-2’nin Sınırları (BMP)  
9. TwinCAT’te WSTRING Kullanım Riskleri  
10. Endüstriyel Kullanım Alanları  
11. Örnek Kodlar  
12. Profesyonel Mühendislik Önerileri  
13. Sonuç  

---

## 1. WSTRING Nedir?

TwinCAT’te **geniş karakter dizisi (wide string)** için kullanılan veri tipidir ve:

- **IEC 61131-3 standardına göre UCS‑2 kodlamalıdır.**
- Her karakter *tam olarak* **2 byte (1 WORD)** ile temsil edilir.
- STRING’den farklı olarak çok geniş karakter setini destekler.
- Literaller **çift tırnak** `" "` ile tanımlanır.

Örnek:

```st
wsVar : WSTRING := "Hello World";
```

---

## 2. UCS‑2 Kodlaması

UCS‑2:

- Unicode karakterlerinin **Basic Multilingual Plane (BMP)** aralığını kapsar.
- Aralık:
  - U+0000 … U+D7FF  
  - U+E000 … U+FFFF  
- Sabit uzunlukludur → her karakter = 2 byte.
- **UTF‑16 değildir**, surrogate pair desteklemez (4 byte karakterler geçersizdir).

Bu nedenle:

- 😊 gibi emoji karakterleri WSTRING ile temsil edilemez.

---

## 3. Bellek Modeli: WORD Bazlı Depolama

WSTRING(N) → N adet karakter için yer + 1 WORD final zero.

Örnek:

```
WSTRING(80)
```

Bellek kullanımı:

- 80 karakter × 2 byte = 160 byte  
- 1 WORD terminator = 2 byte  
**Toplam: 162 byte**

STRING ile kıyaslandığında yaklaşık **2 kat** bellek kullanır.

---

## 4. Karakter Sayısı vs Byte Sayısı

| Kavram | Açıklama |
|--------|----------|
| Karakter Uzunluğu | WSTRING(80) → 80 karakter |
| Byte Karşılığı | 80 × 2 = 160 byte |
| Terminator | Ek 2 byte (WORD#0) |

WSTRING tamamen *karakter tabanlıdır*, STRING gibi byte sınırına göre hareket etmez.

---

## 5. Varsayılan Boyut ve Final Zero WORD

Uzunluk belirtilmezse:

```st
wsVar : WSTRING;
```

TwinCAT default olarak şunu kabul eder:

- **80 karakter** alanı
- + 1 WORD terminator

Toplam: 162 byte.

STRING ile tutarlı bir default davranıştır.

---

## 6. Reset & Initialization Bellek Davranışı

STRING’de olduğu gibi:

> TwinCAT reset sırasında yalnızca yeni STRING değerini yazar.  
> Null terminatörden sonraki eski WORD değerlerini **temizlemez**.

Bu bellek yapısında “shadow memory” olarak bilinir.

Örneğin:

```st
wsVar : WSTRING(20) := "ABC";
```

Reset sonrası gerçek bellek içeriği:

```
41 00 42 00 43 00 00 00 [ESKİ UCS‑2 BYTE’LARI DEVAM EDER]
```

Bu durum:

- Güvenlik verisi  
- Loglama  
- HMI input  
- Şifre alanları  

gibi senaryolarda kritik olabilir.

---

## 7. STRING / UTF‑8 / WSTRING Karşılaştırması

| Özellik | STRING (Latin‑1) | STRING (UTF‑8) | WSTRING (UCS‑2) |
|---------|------------------|----------------|------------------|
| Byte / karakter | 1 | 1–4 | 2 |
| Unicode kapsamı | Sınırlı | En geniş | BMP ile sınırlı |
| Performans | En hızlı | İçeriğe bağlı | Orta |
| Bellek kullanımı | Düşük | Değişken | Yüksek |
| EEPROM/Retain maliyeti | Düşük | Orta | Yüksek |
| Surrogate destek | ❌ | ✔ | ❌ |

WSTRING’in en büyük avantajı:

✔ **Sabit karakter genişliği** (real-time işlemlerde kolaylık sağlar)

---

## 8. UCS‑2’nin Sınırları (BMP)

WSTRING şu karakterleri destekler:

- Türkçe karakterler (Ç, Ğ, İ, Ş, Ö, Ü)
- Avrupa dilleri
- Arapça, İbranice
- Çin / Japonca karakterlerin büyük bölümü
- Semboller (BMP içinde olanlar)

Desteklemez:

❌ Emoji  
❌ Surrogate pair gerektiren modern Unicode karakterler  
❌ U+10000 üzeri tüm kod noktaları  

---

## 9. TwinCAT’te WSTRING Kullanım Riskleri

### ⚠ 1. Surrogate pair hatası  
UTF‑16 karakterleri UCS‑2 WSTRING içinde kullanılamaz → derleme hatası ya da bozuk bellek oluşur.

### ⚠ 2. Reset sonrası eski veri gölgesi  
TwinCAT WSTRING belleğinin tamamını sıfırlamaz.

### ⚠ 3. UTF‑8 ve STRING fonksiyonları ile uyumsuzluk  
LEN, LEFT, MID gibi STRING fonksiyonları **WSTRING’e uygulanamaz**.

### ⚠ 4. Byte dizileri ile veri transferinde dikkat  
Her karakter 2 byte olduğundan ters mapping hataları yaygındır.

---

## 10. Endüstriyel Kullanım Alanları

✔ Çok dilli HMI metinleri  
✔ OPC UA WSTRING tag’leri  
✔ Etiketleme / ürün bilgisi  
✔ XML / JSON Unicode içerikleri (BMP seviyesinde)  
✔ Sabit genişlik isteyen gerçek zamanlı Unicode işlemleri  

Uygun değildir:

❌ Emoji içeren mesajlar  
❌ Modern Unicode protokolleri (MQTT JSON'da emoji vb.)  
❌ UTF‑32 veya UTF‑16 gerektiren sistemler  

---

## 11. Örnek Kodlar

### Basit WSTRING tanımı

```st
wsMessage : WSTRING := "This is a WString.";
```

### Belirli uzunlukta WSTRING

```st
wsName : WSTRING(20) := "HELLO";
```

### Belleğin manuel temizlenmesi (Güvenlik için önerilir)

```st
MEMSET(ADR(wsName), 0, SIZEOF(wsName));
```

---

## 12. Profesyonel Mühendislik Önerileri

### ✔ WSTRING → BMP Unicode gerektiren durumlarda kullan  
UTF‑8 daha evrenseldir.

### ✔ Bellek maliyetini göz önünde bulundur  
WSTRING, aynı uzunluktaki STRING’e göre *2 kat daha fazla* yer kaplar.

### ✔ Reset sonrası memory cleanup gerekiyorsa  
Manuel temizleme yap:

```st
MEMSET(ADR(wsVar), 0, SIZEOF(wsVar));
```

### ✔ FIELD BUS / OPC UA mapping’de encoding dönüşümlerine dikkat et  
UCS‑2 ≠ UTF‑8 ≠ Latin‑1

### ✔ JSON/MQTT gibi modern protokoller için UTF‑8 STRING daha uygundur  
Emoji ve geniş Unicode setleri gerekebilir.

---

## 13. Sonuç

WSTRING:

- UCS‑2 tabanlı geniş karakter dizisidir  
- Sabit karakter genişliği sayesinde RT uygulamalarında avantaj sağlar  
- BMP dışındaki Unicode karakterlerini desteklemez  
- Bellek kullanımı yüksektir  
- Reset sonrası gölgelenmiş veri bırakabilir  
- STRING’den tamamen farklı bir mimariye sahiptir  

TwinCAT uygulamalarında Unicode’un profesyonel şekilde yönetilmesi için temel yapı taşıdır.

---
