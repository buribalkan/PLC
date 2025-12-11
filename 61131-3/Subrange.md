# 🧠 SEVİYE 3 ULTRA PROFESYONEL MASTERCLASS  
# **SUBRANGE TYPES — TWINCAT’TE ARALIK KISITLI TÜRLER**

---

## 📌 İçindekiler
1. Subrange Type Nedir?  
2. Neden Subrange Kullanılır?  
3. Söz Dizimi  
4. Derleyici Seviyesinde Aralık Doğrulama  
5. Runtime Kontrolleri (CheckRangeSigned / CheckRangeUnsigned)  
6. Signed ve Unsigned Subrange Farkları  
7. Bellek Modeli ve Performans  
8. Derleyici Optimizasyonları  
9. Endüstriyel Kullanım Senaryoları  
10. Sık Yapılan Hatalar  
11. Kod Örnekleri  
12. Sonuç  

---

## 1. Subrange Type Nedir?

**Subrange type**, bir integer veri tipinin yalnızca belirli bir alt aralığını temsil eden kısıtlı bir türdür.

Örnek:

```
Temperature : INT(-50..150);
Speed       : UINT(0..5000);
```

Yalnızca integer tipi baz alınabilir:

- SINT, INT, DINT, LINT  
- USINT, UINT, UDINT, ULINT  
- BYTE, WORD, DWORD, LWORD  

---

## 2. Neden Subrange Kullanılır?

✔ **Derleme zamanında güvenlik:** Aralık dışı değer → compiler error  
✔ **Daha az runtime hatası**  
✔ **Kod okunabilirliği artar**  
✔ **Donanım koruması (örneğin hız/tork sınırları)**  
✔ **Operatör giriş hatalarının engellenmesi**  
✔ **Parametre doğrulama kolaylaşır**

---

## 3. Söz Dizimi

```
<Name> : <IntType> (<LowerBound> .. <UpperBound>);
```

Örnek:

```
nVarA : INT(-4095..4095);
nVarB : UINT(0..10000);
```

### Notlar:
- Alt ve üst sınırlar **dahil** edilir.  
- Sınır değerleri **base type** ile uyumlu constant olmalıdır.

---

## 4. Derleyici Seviyesinde Aralık Doğrulama

Eğer subrange’e atanacak değer sınır dışında ise TwinCAT:

❌ **Compile-time error** üretir.

Örnek:

```st
nVarA : INT(-4095..4095);
nVarA := 5000;   // HATA: Out of subrange
```

Bu, TwinCAT'in sunduğu güçlü bir statik güvenlik mekanizmasıdır.

---

## 5. Runtime Kontrolleri (CheckRange Signed / Unsigned)

TwinCAT runtime, subrange ihlalini otomatik izlemez.  
Gerekirse aşağıdaki fonksiyonlar kullanılmalıdır:

- **CheckRangeSigned**
- **CheckRangeUnsigned**
- **CheckLRangeSigned**
- **CheckLRangeUnsigned**

Örnek:

```st
IF NOT CheckRangeSigned(nVarA, -4095, 4095) THEN
    Error := TRUE;
END_IF
```

Bu mekanizma özellikle dış girdilerden (HMI, Fieldbus) gelen değerler için önemlidir.

---

## 6. Signed ve Unsigned Subrange Farkları

| Base Type | Geçerli Subrange | Açıklama |
|-----------|------------------|-----------|
| INT | INT(-100..50) | Negatif-pozitif aralık |
| UINT | UINT(0..5000) | Alt sınır negatif olamaz |
| UDINT | UDINT(1000..2000000) | Geniş unsigned aralıklar |

Unsigned subrange → asla negatif alt sınır olmaz.

---

## 7. Bellek Modeli ve Performans

✔ Subrange türleri **base type kadar yer kaplar**  
✔ Ek bellek maliyeti yoktur  
✔ Runtime’da **hiçbir performans kaybı olmaz**  
✔ Derleyici subrange sınırlarını optimize eder  
✔ Runtime güvenlik isteniyorsa CheckRange kullanılmalıdır

---

## 8. Derleyici Optimizasyonları

TwinCAT derleyicisi:

- Subrange türünü **tamamen base type** olarak işler  
- Sabit değerlerle sınır kontrolünü compile-time’da çözer  
- Gereksiz runtime kontrollerini kaldırır  
- Kodunuzu daha güvenli ve deterministik hale getirir

---

## 9. Endüstriyel Kullanım Senaryoları

✔ Motor hız limitleri: `Speed : UINT(0..3000)`  
✔ Tork sınırı: `Torque : INT(-100..100)`  
✔ Analog input scaling: `Raw : INT(0..32767)`  
✔ Güvenlik parametreleri  
✔ Proses setpoint doğrulama  
✔ Robotik eksen aralıkları  

Subrange hem **güvenlik** hem **kararlılık** hem de **girdi doğrulama** için kritik bir araçtır.

---

## 10. Sık Yapılan Hatalar

### ❌ 1. Subrange dışına değer atamak (compile error)
```st
Value := 99999; // BASE TYPE uygun ama SUBRANGE dışı → ERROR
```

### ❌ 2. Runtime kontrolünü atlamak
Dış kaynaklardan gelen veri için CheckRange gerekebilir.

### ❌ 3. Signed ↔ Unsigned uyumsuzluğu
`UINT(-1..10)` → geçersiz

### ❌ 4. Subrange’i base type zannetmek
Subrange sadece bir **kısıtlama**, yeni bir veri formatı değildir.

---

## 11. Kod Örnekleri

### ✔ Doğru kullanım
```st
Temperature : INT(-50..150);
Temperature := 120; // OK
```

### ❌ Yanlış kullanım
```st
Temperature := 200; // ERROR: Out of subrange
```

### ✔ Runtime sınır kontrolü
```st
IF NOT CheckRangeSigned(Temperature, -50, 150) THEN
    Alarm := TRUE;
END_IF;
```

---

## 12. Sonuç

Subrange Types:

- Hata yakalama açısından *derleme zamanı güvenliği* sağlar  
- Runtime’da CheckRange ile birleştiğinde tam koruma sunar  
- Kodun anlamını netleştirir  
- Donanım/işletme limitlerini kolayca enforce eder  
- Performans maliyeti yoktur  

TwinCAT uygulamalarında profesyonel, güvenli ve sürdürülebilir PLC yazılımı geliştirmek için vazgeçilmez bir araçtır.

---
