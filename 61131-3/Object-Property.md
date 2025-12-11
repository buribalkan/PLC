# OBJECT PROPERTY – MASTERCLASS LEVEL 3  
TwinCAT 3 | Professional Engineering Series

---

## 1. Giriş  
**Property (Özellik)** IEC 61131-3 standardına TwinCAT tarafından eklenen, tam nesne yönelimli programlamayı mümkün kılan bir yapıdır. Bir property, **Get** ve **Set** olmak üzere iki erişim metodundan oluşur. Bu metotlar TwinCAT tarafından otomatik olarak çağrılır:

- **Get →** Property okunurken otomatik çalışır.  
- **Set →** Property yazılırken otomatik çalışır.  

Property, bir **metodun davranışını** gösteren ama **değişken gibi kullanılan** bir yapıdır.

---

## 2. Property Oluşturma  
### Adımlar
1. Bir **FUNCTION_BLOCK** veya **PROGRAM** üzerinde sağ tıklayın.  
2. **Add > Property…** seçeneğini seçin.  
3. Property adını, türünü, dilini ve access modifier’ını belirleyin.  
4. TwinCAT, property altına otomatik olarak **Get** ve **Set** accessorlarını ekler.

### Kullanılabilir Access Modifier’lar
| Modifier | Erişim Seviyesi |
|---------|------------------|
| **PUBLIC** | Her yerden erişilebilir |
| **PRIVATE** | Sadece aynı FB içinde |
| **PROTECTED** | FB ve türevleri |
| **INTERNAL** | Aynı namespace içinde |
| **FINAL** | Override edilemez |
| **ABSTRACT** | Implementasyon türev FB'de olmalıdır |

---

## 3. Property Yapısı  
Bir property aşağıdaki bileşenlerden oluşur:

```
PROPERTY <Name> : <ReturnType>
```

İki accessor metodu otomatik üretilir:

```
PROPERTY.Get
PROPERTY.Set
```

---

## 4. Örnek — Temel Property Kullanımı

### FB Tanımı
```iecst
FUNCTION_BLOCK FB_Sample
VAR
    nVar : INT;
END_VAR
```

### Property Deklarasyonu
```iecst
PROPERTY PUBLIC nValue : INT
```

### Set Accessor
```iecst
nVar := nValue;
```

### Get Accessor
```iecst
nValue := nVar;
```

### Property Çağrımı
```iecst
PROGRAM MAIN
VAR
    fbSample : FB_Sample;
END_VAR

fbSample();

IF fbSample.nValue > 500 THEN
    fbSample.nValue := 0;
END_IF
```

---

## 5. Local Variables in Property  
Property içinde **local değişken** tanımlanabilir; ancak **ek input veya output** tanımlanamaz.

---

## 6. Property Kullanımında Özel Durumlar  

### 6.1 Read-Only veya Write-Only Property
Accessorlardan biri silinerek yapılır:

- **Sadece Get varsa → Read‑Only property**
- **Sadece Set varsa → Write‑Only property**

---

## 7. REFERENCE Return Type ile Property  
Bir property, **REFERENCE TO <type>** türünde tanımlanırsa, Get çağrısı sırasında **FB içindeki gerçek yapıya referans** döndürülür.

### Örnek
```iecst
TYPE ST_Sample :
STRUCT
    bVar : BOOL;
    nVar : INT;
END_STRUCT
END_TYPE
```

```iecst
FUNCTION_BLOCK FB_Sample
VAR
    stLocal : ST_Sample;
END_VAR
```

### Property
```iecst
PROPERTY MyProp : REFERENCE TO ST_Sample
```

### Get
```iecst
MyProp REF= stLocal;
```

### Set
```iecst
stLocal := MyProp;
```

### Kullanım
```iecst
nSingle := fbSample.MyProp.nVar;   // Doğrudan erişim!
```

### Neden Bu Önemli?
Bu teknik, **gereksiz kopyalamayı ortadan kaldırır** ve tam nesne yönelimli erişim sunar.

---

## 8. TwinCAT 4022 ↔ 4026 Uyum Farkları  
### 8.1 REFERENCE Property Set Davranışı Değişikliği
- **4022’den önce:** `:=` → Set çağrılır  
- **4022 ve sonrası:** `:=` → *Get çağrılır!* → Değer atanır  

Bu yüzden referans atamak için her zaman:

```
REF=
```

kullanılmalıdır.

---

## 9. Monitoring Seçenekleri  
Property üzerinde çevrimiçi izleme (monitoring) için iki özel pragma vardır:

### 9.1 Değer Saklama
```iecst
{attribute 'monitoring' := 'variable'}
```

### 9.2 Get Her İzlemede Çalışsın
```iecst
{attribute 'monitoring' := 'call'}
```

**Dikkat:** Eğer Get içinde yan etki varsa, monitoring sırasında bile çalışır.

---

## 10. VAR_IN_OUT Erişim Uyarısı — C0371  
Bir property veya method içinden FB’in **VAR_IN_OUT** değişkenine erişmek risklidir.

### TwinCAT Uyarısı:
```
Warning: Access to VAR_IN_OUT <Var> from external context <Property>
```

Bu durumda yapılması gereken:

### Adım 1 — Referansın geçerli olup olmadığını kontrol et
```iecst
IF NOT __ISVALIDREF(bInOut) THEN
    RETURN;
END_IF
```

### Adım 2 — Uyarıyı devre dışı bırak
```iecst
{warning disable C0371}
```

### Adım 3 — Erişimden sonra uyarıyı geri aç
```iecst
{warning restore C0371}
```

---

## 11. FB_init ile Property Başlatma Örneği

### FB
```iecst
FUNCTION_BLOCK FB_Sample
VAR_INPUT
    nInput : INT;
END_VAR
VAR
    nLocalInitParam : INT;
    nLocalProp      : INT;
END_VAR
```

### FB_init
```iecst
METHOD FB_init : BOOL
VAR_INPUT
    bInitRetains : BOOL;
    bInCopyCode  : BOOL;
    nInitParam   : INT;
END_VAR

nLocalInitParam := nInitParam;
```

### Property
```iecst
PROPERTY nMyProperty : INT
nLocalProp := nMyProperty;
```

### Instantiation
```iecst
fbSample : FB_Sample(nInitParam := 1) := (nInput := 2, nMyProperty := 3);
```

---

## 12. Best Practices (Masterclass Seviyesi)

### ✔ Property, değişken gibi görünmeli ama **davranış içermeli**
- Get → hesaplama yapılabilir  
- Set → doğrulama yapılabilir  

### ✔ Property içinde ağır işlem yapılmaz  
Property hızlı olmalıdır. Ağır işlem → metoda taşınmalıdır.

### ✔ REFERENCE property yalnızca çok büyük veri yapıları için önerilir  
Yoksa gereksiz karmaşıklık yaratır.

### ✔ Monitoring sırasında yan etki oluşturabilecek kodlardan kaçın  

### ✔ Set içinde doğrulama yapılabilir  
Örneğin:

```iecst
IF nValue < 0 THEN
    RETURN;  // Hatalı değer set edilmesin
END_IF
```

---

## 13. Sonuç  
TwinCAT Property mekanizması, PLC dünyasında nesne yönelimli yaklaşımı güçlendiren en önemli yapılardan biridir. Get/Set accessors, REFERENCE return type, access modifier’lar ve monitoring olanakları ile property, yüksek seviye yazılım mimarileri geliştirmek için ideal bir araçtır.

Bu masterclass dokümanı, property’lerin **teknik temellerini**, **ileri seviye kullanım örneklerini** ve **en iyi pratikleri** kapsamlı şekilde sunmaktadır.

---

**Hazırlayan:** *TwinCAT 3 Masterclass Level 3 – Advanced Engineering Guide*  
