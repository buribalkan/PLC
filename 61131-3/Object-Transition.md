# 🧠 SEVİYE 3 ULTRA PROFESYONEL MASTERCLASS  
# **TwinCAT Object Transition — Derinlemesine İnceleme**

---

## 📌 İçindekiler
1. Transition Nedir?  
2. Inline vs Multi‑Use Transition  
3. Transition Nesnesinin Yaşam Döngüsü  
4. Transition Oluşturma Adımları  
5. Transition Çağrım Sözdizimi  
6. SFC İçindeki Transition Davranışı  
7. VAR_IN_OUT Değişkenlerine Erişim — Kritik Güvenlik Notu  
8. `__ISVALIDREF` ile Güvenli Referans Kontrolü  
9. Gelişmiş Kod Örnekleri  
10. Best Practices  
11. Sonuç  

---

# 1. Transition Nedir?

Transition, bir **SFC (Sequential Function Chart)** akışında **bir sonraki adımın (Step) aktive edilip edilmeyeceğini belirleyen koşuldur**.

Transition:

- TRUE → sonraki Step çalışır  
- FALSE → mevcut Step devam eder  

Transition iki farklı şekilde tanımlanabilir:

---

# 2. Transition Türleri  
## ✅ (1) Inline Transition (doğrudan koşul)

SFC diyagramında transition yerine **doğrudan bir Boolean ifade** yazılır:

```st
(i < 100) AND bEnable
```

Kısıtlamalar:

❌ Program çağrısı yok  
❌ FB çağrısı yok  
❌ Atama yok  

Sadece **Boolean değer döndüren ifadeler** kullanılabilir.

---

## ✅ (2) Multi‑Use Transition (nesne temelli transition)

Solution Explorer → **Add > Transition…**  
ile bir transition nesnesi oluşturulur.

Bu nesne:

- Tekrar tekrar kullanılabilir  
- İçinde **çok satırlı kod** olabilir  
- Boolean döndüren kompleks mantık içerebilir  
- Tıpkı bir METHOD gibi davranır  

Örnek bir transition dosyası:

```st
Trans1 := (nCount < 100);
```

---

# 3. Transition Nesnesinin Yaşam Döngüsü

Transition:

- Temel PROGRAM veya FUNCTION_BLOCK’un **tüm değişkenlerine erişebilir**  
- Kendi deklarasyon alanı vardır  
- Bir METHOD gibi bağımsız yürütülür  
- Sadece TRUE/FALSE üretir  

---

# 4. Transition Oluşturma Adımları

1. FB veya PROGRAM seçilir  
2. Sağ tık → **Add > Transition…**  
3. Ad verilir  
4. Dil seçilir (ST / IL / FBD / SFC Action Language)  
5. Editör açılır ve transition kodu yazılır  

---

# 5. Transition Çağrım Sözdizimi

### ✔ ST içinde

```st
Trans1 := (nCount <= 100);
```

veya sadece:

```st
(nCount <= 100)
```

Eğer **çok satırlı transition** ise:

```st
Trans1 := (MyConditionVar);
```

### ✔ SFC içinde

Transition ismi doğrudan transition bloğuna yazılır.

---

# 6. SFC İçindeki Transition Davranışı

Transition:

- Her cycle sonunda değerlendirilir  
- TRUE olduğunda step değişir  
- FALSE iken mevcut step çalışmaya devam eder  
- Inline veya Transition POU olabilir  

Transition, SFC’nin akış kontrolünün temel mekanizmasıdır.

---

# 7. VAR_IN_OUT Değişkenlerine Erişim — Kritik Not

Transition içinden bir FB’nin `VAR_IN_OUT` değişkenine erişirken **tehlikeli durum** oluşabilir.

Neden?

Çünkü:

- Bir FB'nin body’si çağrılmadan önce  
- Transition çağrısı yapılabilir  

Bu durumda `VAR_IN_OUT` **geçerli bir referans olmayabilir**.

TwinCAT bu durumda **C0371 Warning** üretir:

```
Warning: Access to VAR_IN_OUT <Var> ... from external context <Transition>
```

---

# 8. `__ISVALIDREF` ile Güvenli Referans Kontrolü

TwinCAT bir referansın geçerli olup olmadığını kontrol etmek için:

```st
__ISVALIDREF(myVar)
```

fonksiyonunu sağlar.

Kullanımı bir zorunluluktur.

### Güvenli örnek:

```st
{warning disable C0371}

IF NOT __ISVALIDREF(bInOut) THEN
    RETURN; // referans geçersiz → çık
END_IF

bInOut := NOT bInOut;

{warning restore C0371}
```

---

# 9. Gelişmiş Örnek

## FB:

```st
FUNCTION_BLOCK FB_Sample
VAR_IN_OUT
    bInOut : BOOL;
END_VAR
```

## Transition:

```st
{warning disable C0371}

IF NOT __ISVALIDREF(bInOut) THEN
    RETURN;
END_IF

TransLogic := NOT bInOut;

{warning restore C0371}
```

## Kullanım:

SFC transition bloğunda:

```
FB_Sample.TransLogic
```

---

# 10. Best Practices

### ✔ Inline transition kullan:
- Basit koşullar için  
- Performans önceliği varsa  

### ✔ Transition object kullan:
- Çok kullanımlı mantık varsa  
- Koşullar çok satırlı ise  
- Aynı transition birden fazla step’te kullanılacaksa  

### ✔ VAR_IN_OUT erişiminde:
- Her zaman `__ISVALIDREF` kullan  
- C0371 uyarısını bastırmadan önce mutlaka referans doğrula  

### ✔ Transition = Method değildir  
Transition sadece TRUE/FALSE üretmek içindir.

### ✔ Kodu sade tut  
SFC’nin gücü, grafiksel akıştan gelir → transition’ı karmaşıklaştırma.

---

# 11. Sonuç

Transition nesneleri:

- SFC akışının omurgasıdır  
- Inline veya reusable olabilir  
- Doğru kullanıldığında SFC mantığını çok güçlü ve okunabilir kılar  
- VAR_IN_OUT erişiminde dikkat ve doğrulama gerektirir (C0371 uyarısı)  

Bu masterclass, Object Transition’ların TwinCAT profesyonel kullanımını tüm yönleriyle açıklar.

---
