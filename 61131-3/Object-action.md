# 🧠 SEVİYE 3 ULTRA PROFESYONEL MASTERCLASS  
# **TwinCAT Object Action — Derinlemesine İnceleme**

---

## 📌 İçindekiler
1. Object Action Nedir?  
2. Action’ın Yaşam Döngüsü ve Bellek Modeli  
3. Action ve Function Block İlişkisi  
4. Action Nasıl Oluşturulur?  
5. Action Çağrım Kuralları  
6. Farklı Dillerde Action Kullanımı  
7. SFC’de Action Mekanizması  
8. Gelişmiş Örnekler  
9. Best Practices  
10. Sonuç  

---

# 1. Object Action Nedir?

TwinCAT’te **Action**, bir **PROGRAM** veya **FUNCTION_BLOCK** altına eklenebilen özel bir programlama nesnesidir.

Bir Action:

- Kendi başına bir POU değildir  
- **Kendi değişken deklarasyonu yoktur**  
- Tüm değişkenleri temel implementasyondan (FB veya PROGRAM) devralır  
- Aynı FB içinde farklı dillerde (ST, IL, FBD, LD) ek davranışlar yazmaya izin verir  

Bu nedenle Action, FB içindeki alternatif davranış bloklarının kapsülüdür.

---

# 2. Action’ın Yaşam Döngüsü ve Bellek Modeli

✔ Bir Action **temel FB’nin belleğini kullanır**  
✔ Action içinde deklarasyon yapılamaz  
✔ Action çağrısı → temelde yer alan değişkenlerle çalışır  

Bu şu anlama gelir:

- Action çalışırken oluşturulan hiçbir değişken stack üzerinde değildir  
- Action bir FB metoduna göre daha hafiftir  
- Action program kontrol akışına göre çağrılır  

---

# 3. Action ve Function Block İlişkisi

Action doğrudan bir FB’ye bağlıdır.  
Bir Action:

- FB'nin üyelerini (inputs, outputs, VAR, VAR_STAT, vb.) kullanır  
- FB’nin durumunu değiştirebilir  
- Bir FB metoduna benzer, ancak **gerçek bir POU değildir**

Action’ın amacı:

- FB içinde modüler davranış blokları oluşturmak  
- Farklı dillerde ilave kod yazmak  
- SFC içinde adım davranışlarını yönetmek  

---

# 4. Action Nasıl Oluşturulur?

1. Solution Explorer → FB veya PROGRAM seçilir  
2. Sağ tık → **Add > Action…**  
3. Action adı yazılır  
4. Implementasyon dili seçilir  
5. Açılır ve proje ağacına eklenir  

Action hiçbir zaman tek başına çağrılmaz; mutlaka ait olduğu FB/PROGRAM üzerinden çağrılır.

---

# 5. Action Çağrım Kuralları

## Temel Sözdizimi:

### FB içinden:
```st
Reset();
```

### Başka bir POU’dan:
```st
fbCounterA.Reset();
```

### Açık parametre çağrımı (ST):
```st
fbCounterA.Reset(In := FALSE);
```

### IL çağrımı:
```
fbCounterA.Reset
```

### FBD çağrımı:
FBD’de Action, FB instance kutusunun alt davranışı olarak görünür.

---

# 6. Farklı Dillerde Action Kullanımı

TwinCAT Action’lar şu dillerde yazılabilir:

- ST  
- IL  
- Ladder (LD)  
- FBD  
- SFC (özellikle yaygın kullanım)  

Her Action’ın dili, FB’nin kendi dilinden bağımsız olabilir.

Örneğin:

- FB ST ile yazılmış olabilir  
- Ancak bir Action IL ile yazılmış olabilir  

Bu çoklu dil desteği Action’ların en güçlü özelliklerindendir.

---

# 7. SFC’de Action Mekanizması

SFC’de Action nesneleri:

- Adımların davranışlarını tanımlar  
- “Stored Action”, “Transition Action”, “Continuous Action” gibi farklı türlerde davranış modelleri uygulanabilir  
- SFC grafiğinde Action sembolleri doğrudan çağrılır  

TwinCAT SFC editörü Action kullanımını otomatikleştirir.

---

# 8. Gelişmiş Örnek

### FB tanımı:
```st
FUNCTION_BLOCK FB_Counter
VAR_INPUT
    In : BOOL;
END_VAR
VAR_OUTPUT
    nOut : INT;
END_VAR
```

### Action: Reset
```st
nOut := 0;
```

### Action çağrısı
```st
fbCounterA.Reset(In := FALSE);
result := fbCounterA.nOut;
```

Önemli not:  
Action'ın parametre deklarasyonu yoktur — parametreler FB'nin inputs olarak yorumlanır.

---

# 9. Best Practices

### ✔ Action kullanın:
- SFC step behavior tanımlarken  
- FB içinde küçük modüler davranış bloklarına ihtiyaç varsa  
- Farklı dillerde minik görevler yazmanız gerekiyorsa  

### ✔ Metod yerine Action kullanın:
- Ek deklarasyon gerekmiyorsa  
- Legacy kod veya multi-dil mimarisi gerekiyorsa  

### ❌ Action kullanmayın:
- Local değişkenlere ihtiyaç varsa  
- Geniş API tasarlıyorsanız  
- Global veya reusable logic gerekiyorsa → METHOD tercih edilmeli  

---

# 10. Sonuç

Action nesneleri TwinCAT'te:

- FB davranışlarını bölmek  
- SFC step yönetimini sağlamak  
- Çoklu dil desteği ile kodu modülerleştirmek  

için kullanılan güçlü ama çoğu zaman yanlış anlaşılan bir mekanizmadır.

Bu masterclass dokümanı Action’ların profesyonel kullanımını tüm yönleriyle açıklar.

---
