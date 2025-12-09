# 🚦 TwinCAT Shading Rules (İsim Gölgeleme Kuralları) – Tam Kapsamlı Eğitim Dokümanı (.md)

Bu doküman, TwinCAT’te **shading rules (isim gölgeleme kuralları)** konusunu *hiçbir şey bilmeyen birine bile anlatacak şekilde* sadeleştirilmiş, ama aynı zamanda profesyonel mühendislik seviyesinde hazırlanmış **tam kapsamlı bir eğitim setidir**.

---

# 🧠 1. Shading (Gölgeleme) Nedir?

TwinCAT’te **aynı ismi birden fazla yerde kullanabilirsin**, örneğin:

- Bir FUNCTION’ın adı: `Sample`
- Bir FB instance’ın adı: `Sample`

TwinCAT bunu **hata olarak görmez**.

Ancak şöyle bir kodda:

```pascal
FUNCTION Sample : INT
FUNCTION_BLOCK FB_Sample

PROGRAM MAIN
VAR
    Sample : FB_Sample;
END_VAR

Sample();
```

Sorun şudur:

> **Sample() çağrıldığında TwinCAT hangi Sample’ı kullandığını senin yerine kendi seçer.**

İşte buna **shading (gölgeleme)** denir.

---

# 🚨 2. Shading Neden Tehlikelidir?

Çünkü:

❌ TwinCAT hata vermez  
❌ TwinCAT uyarı vermez  
❌ Kod çalışır ama yanlış olan çalışır  
✔ Hata bulması çok zordur  

Bu yüzden shading, PLC yazılımında en riskli hata türlerinden biridir.

---

# 🎯 3. TwinCAT İsim Arama Sırası (Search Order)

TwinCAT bir ismi görünce, onu aşağıdaki sırayla arar:

## 🔍 1. Local değişkenler
1. Method içindeki değişkenler  
2. Function block içindeki değişkenler  
3. Program içi değişkenler  
4. Function block’un LOCAL metodları  

**Yerel değişken her zaman en güçlüdür.  
Her şeyi gölgeler.**

---

## 🔍 2. Global değişkenler
Ancak *qualified_only* yoksa.

Örneğin:

```pascal
VAR_GLOBAL
    Speed : INT;
END_VAR
```

TwinCAT bunu doğrudan görebilir.

Ama:

```pascal
{attribute 'qualified_only'}
GVL
VAR_GLOBAL
    Speed : INT;
END_VAR
```

Artık sadece şöyle erişebilirsin:

```pascal
GVL.Speed
```

---

## 🔍 3. Function Block / Type İsimleri

Projede tanımlı:

- TYPE’lar
- Function Block isimleri
- Global variable listesinin isimleri

---

## 🔍 4. Library isimleri

- Kapsam (namespace) seviyesinde library tanımları
- Library içindeki FB ve TYPE’lar

---

# 🏆 SONUÇ:

> **Aynı isimden iki tane varsa → en önce bulunan kazanır, diğerleri gölgede kalır.**

İşte shading bu yüzden tehlikeli.

---

# 🧩 4. Shading Neden Olur? Örnekle Açıklayalım

```pascal
FUNCTION Sample : INT
FUNCTION_BLOCK FB_Sample

PROGRAM MAIN
VAR
    Sample : FB_Sample;  // FUNCTION ile aynı isim
END_VAR

Sample();
```

TwinCAT bu `Sample()` çağrısını:

✔ Local değişken → FB_Sample instance olarak görür  
❌ Function Sample’ı asla görmez  

Bu yüzden program **yanlış Sample’ı çağırır**.

---

# 🧱 5. Shading’den Nasıl Kaçınırız?

## ✔ 1. İsimlendirme kuralları kullan

Örneğin:

- FB_ prefix → function block
- F_ prefix → function
- GVL_ prefix → global variable list
- g_ prefix → global variable
- l_ prefix → local variable

Bu sayede çakışma olmaz.

---

## ✔ 2. qualified_only attribütünü kullan

Böylece global değişkene sadece GVL adıyla erişilir.

```pascal
{attribute 'qualified_only'}
GVL
VAR_GLOBAL
    MotorSpeed : INT;
END_VAR
```

Artık erişim:

```pascal
GVL.MotorSpeed
```

---

## ✔ 3. Library erişimlerini namespace ile yap

```pascal
MyLibrary.FB_Motor();
```

---

## ✔ 4. Bir FB'in kendi değişkenlerine THIS ile eriş

Eğer metoda local bir isim çakışması varsa:

```pascal
THIS.MotorSpeed
```

Bu, FB’in kendi değişkenini garanti eder.

---

# 🚩 6. Ambiguous Access (Belirsiz Erişim)

Bazı durumlarda TwinCAT bile karar veremez.

Örneğin:

İki farklı GVL içinde aynı isimli değişken varsa VE ikisi de unqualified erişime açıksa:

TwinCAT şu hatayı verir:

```
ambiguous use of the name <variable>
```

### Çözüm:

```pascal
GVL1.VarName
GVL2.VarName
```

Qualified access ile adı netleştir.

---

# 🔍 7. Instance Path Kuralları (yy.Component)

Eğer erişim şöyleyse:

```pascal
yy.Component
```

O zaman shading kuralları değişir:

## ✔ Eğer yy → STRUCT/UNION türü değişkense:

Arama sırası:

1. FB local değişkenleri  
2. FB’in base (kalıtım) değişkenleri  
3. FB metodları  
4. Base FB metodları  

---

## ✔ Eğer yy → Global Variable List ise:

Sadece o GVL içinde arar.

---

## ✔ Eğer yy → Library Namespace ise:

Library search order kuralları geçerli olur.

---

# 🧭 8. Shading Hatalarını Bulmanın En Kolay Yolu → Go To Definition

TwinCAT’te bir isim üzerinde sağ tıkla:

👉 **Go To Definition**

TwinCAT sana şu soruların cevabını verir:

- Bu isim nerede tanımlı?  
- Hangi tanım kullanılacak?  
- Hangileri gölgelenmiş?  

Bu araç shading problemi çözerken en güçlü yardımcıdır.

---

# 🎯 9. Kısa Özet (10 Saniyede Shading Rules)

- Aynı ismi iki kez kullanma → karışır  
- TwinCAT en yakındaki tanımı seçer  
- Diğerleri gölgede kalır → shading  
- Bunu çözmek için:  
  ✔ İsimlendirme kuralları  
  ✔ qualified_only  
  ✔ namespace  
  ✔ THIS  
- Sorun olduğunda: Go To Definition

---



