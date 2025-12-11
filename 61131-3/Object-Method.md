# 🧠 SEVİYE 3 ULTRA PROFESYONEL MASTERCLASS  
# **TwinCAT OBJECT METHOD — DERİNLEME İNCELEME**

---

## 📌 İçindekiler
1. Method Nedir?  
2. Method vs Function vs Action  
3. Method Bellek Modeli (Stack-Based Execution)  
4. Method Oluşturma  
5. Method Çağrım Kuralları  
6. Input/Output Davranışı  
7. THIS Pointer ile Instance Erişimi  
8. VAR_INST Kullanımı  
9. REFERENCE TO ile Tek Eleman Erişimi  
10. Recursive Methods  
11. Access Modifiers (PUBLIC, PRIVATE, PROTECTED, INTERNAL, FINAL, ABSTRACT)  
12. FB_init, FB_reinit, FB_exit Özel Metodları  
13. VAR_IN_OUT Erişim Riskleri  
14. C0371 Warning Yönetimi  
15. Best Practices  
16. Sonuç  

---

# 1. Method Nedir?

TwinCAT’te **Method**, bir program (PRG) veya function block (FB) altında tanımlanan,  
**kendi deklarasyonu ve implementasyonu olan fakat bağımsız olmayan** bir kod birimidir.

✔ Fonksiyon gibi dönüş değeri olabilir  
✔ Ek giriş/çıkış alabilir  
✔ FB’nin tüm değişkenlerine erişebilir  
✔ THIS pointer sayesinde instance’a doğrudan erişebilir  

❌ Method bağımsız bir POU değildir  
❌ Global olarak çağrılamaz  
❌ Yalnızca bağlı olduğu FB/PRG üzerinden çağrılır  

---

# 2. Method vs Function vs Action

| Özellik | Method | Function | Action |
|--------|--------|----------|--------|
| Bağımsız POU | ❌ | ✔ | ❌ |
| RETURN değeri | ✔ | ✔ | ❌ |
| FB değişkenlerine erişim | ✔ | ❌ | ✔ |
| Stack üzerinde local değişken | ✔ | ✔ | yok |
| Çok satırlı kod | ✔ | ✔ | ✔ |
| OOP destekler | ✔ | ❌ | sınırlı |

---

# 3. Bellek Modeli: Stack Variables

Bir method içindeki tüm değişkenler **stack üzerinde** yaşar.

- Her çağrıda yeniden oluşturulur  
- Her çağrı sonunda tamamen yok edilir  
- Kalıcı değildir  

VAR_INST hariç → o FB instance’ına bağlıdır ve yeniden oluşturulmaz.

---

# 4. Method Oluşturma

Solution Explorer:

```
Add → Method
```

Dialog:

- Name  
- Return Type  
- Implementation Language  
- Access Modifier  

Editor açıldığında üstte deklarasyon, altta implementasyon bulunur.

---

# 5. Method Çağrım Sözdizimi

```st
returnValue := fb.MethodName(
    input1 := value1,
    input2 := value2,
    out1 => localOut1,
    out2 => localOut2
);
```

Örnek:

```st
bReturn := fbSample.Method1(
    nIn1  := nLocalInput1,
    bIn2  := bLocalInput2,
    fOut1 => fLocalOutput1,
    sOut2 => sLocalOutput2
);
```

Girişler sırasız verilebilir.

---

# 6. Input/Output Davranışı

TwinCAT 3.1.4026 sonrası:

- **Varsayılan başlangıç değeri olmayan** input → çağrıda atanmak zorunda  
- **Varsayılan başlangıç değeri olan** input → opsiyonel  

---

# 7. THIS Pointer ile Instance Erişimi

✔ Bir method’un kendi FB’sine erişmesi için:

```st
THIS^.nCounter := THIS^.nCounter + 1;
```

THIS, methodun bağlı olduğu FB instance’ını temsil eder.

---

# 8. VAR_INST Kullanımı

Method içinde tekrar çağrıldığında reinitialize olmayan değişkendir.

Örnek:

```st
VAR_INST
    nLastValue : INT := 0;
END_VAR
```

Fonksiyon gibi çalışır fakat durum (state) tutabilir.

---

# 9. REFERENCE TO ile Tek Eleman Erişimi

Bu mekanizma sayesinde:

```st
fb.MethodReturningStruct().Member
```

şeklinde doğrudan elemana erişilebilir.

Bunun çalışması için:

- Return type → `REFERENCE TO ST_MyStruct`
- Return işlemi → `REF=`

Örnek:

```st
MyProp REF= stLocal;
```

---

# 10. Recursive Methods

İki şekilde çağrılır:

## THIS pointer:

```st
result := THIS^.MyMethod(n);
```

## Local FB instance:

```st
fbTemp := THIS^;
result := fbTemp.MyMethod(n);
```

⚠ TwinCAT recursive çağrı için uyarı verir.

Bunu bastırmak için:

```st
{attribute 'estimated-stack-usage' := '256'}
```

---

# 11. Access Modifiers

| Modifier | Erişim | Sembol |
|----------|--------|--------|
| PUBLIC | Her yer | — |
| PRIVATE | Yalnızca FB | 🔒 |
| PROTECTED | FB + türevleri | ★ |
| INTERNAL | Aynı namespace | ♥ |
| FINAL | Override edilemez | — |
| ABSTRACT | Gövdesiz | — |

---

# 12. Özel FB Metodları

| Method | Açıklama |
|--------|----------|
| **FB_init** | Instance oluşturulurken çalışır |
| **FB_reinit** | Instance kopyalanınca (online change) çalışır |
| **FB_exit** | Download/Reset öncesi temizleme metodu |

FB_init implicit olabilir; diğerleri explicit tanımlanmalıdır.

---

# 13. VAR_IN_OUT Erişim Riskleri (C0371 Warning)

Method, transition veya property çağrıldığında:

✔ FB body çalıştırılmamış olabilir  
✔ VAR_IN_OUT henüz geçerli bir referans olmayabilir  
✔ Bu, *runtime crash* riski oluşturur  

TwinCAT bu yüzden uyarı üretir:

```
Warning C0371: Access to VAR_IN_OUT <var> from external context <method>
```

---

# 14. Güvenli Erişim — `__ISVALIDREF`

Önerilen çözüm:

```st
{warning disable C0371}

IF NOT __ISVALIDREF(bInOut) THEN
    RETURN;
END_IF

bInOut := NOT bInOut;

{warning restore C0371}
```

Bu kontrol yapılırsa uyarı kapatılabilir.

---

# 15. Best Practices

### ✔ Method kullan:
- Modüler davranış blokları için  
- FB içi mantık parçalara ayrılacaksa  
- Dönüş değeri gerekiyorsa  

### ✔ VAR_INST kullan:
State tutması gereken methodlar için  

### ✔ REFERENCE TO kullan:
Performans kritik büyük struct dönüşlerinde  

### ✔ Access modifiers kullan:
OOP mimari kontrolü için  

### ❌ Yapılmamalı:
- Method içinde ağır işlem + recursion → stack overflow riski  
- VAR_IN_OUT korumasız erişim  
- Methodu action gibi kullanmak  

---

# 16. Sonuç

TwinCAT Methods:

- OOP’nin temel yapı taşıdır  
- FB’nin tüm verilerine güvenli biçimde erişebilir  
- Çok güçlü fakat dikkat gerektiren bir araçtır  
- REFERENCE TO, VAR_INST, THIS gibi gelişmiş özelliklerle birlikte çok esnek bir mimari sunar  

Bu masterclass, TwinCAT METHOD mekanizmasını en ince detayına kadar profesyonel düzeyde kapsar.

