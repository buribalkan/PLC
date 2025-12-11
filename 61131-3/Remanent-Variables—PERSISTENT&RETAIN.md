# 🧠 SEVİYE 3 ULTRA PROFESYONEL MASTERCLASS  
# **REMANENT VARIABLES — PERSISTENT & RETAIN DERİN TEKNİK EĞİTİMİ**

---

# 📌 İçindekiler
1. Remanent Memory Kavramı  
2. TwinCAT Bellek Modeli  
3. RETAIN vs PERSISTENT — Profesyonel Karşılaştırma  
4. Retain Handler Mekanizması  
5. UPS Gereksinimi & Shutdown İşlemleri  
6. Reset Davranış Matrisi  
7. Download, Online Change & Power Loss Davranışları  
8. PERSISTENT — Derin Teknik İnceleme  
9. RETAIN — Derin Teknik İnceleme  
10. Pointer Kullanımındaki Tehlikeler  
11. AT %I / %Q Kullanım Yasakları  
12. Local vs Global RETAIN/PERSISTENT  
13. TcInitOnReset Pragması  
14. Endüstriyel Kullanım Senaryoları  
15. Hatalı Kullanım Örnekleri  
16. Örnek Kodlar  
17. Sonuç

---

# 1. Remanent Memory Kavramı
Remanent değişkenler, PLC yeniden başlatıldığında veya güç kesildiğinde dahi değerlerini koruyabilen özel değişkenlerdir.

TwinCAT iki tür remanent mekanizma sunar:

- **RETAIN** → güç kesintisine dayanıklı  
- **PERSISTENT** → güç kesintisi + PLC projesi download sonrası bile kalıcı  

Her ikisi de normal RAM’den farklı özel memory alanlarında tutulur.

---

# 2. TwinCAT Bellek Modeli

TwinCAT üç temel bellek alanı kullanır:

| Bellek Alanı | Kullanım |
|--------------|----------|
| Normal PLC RAM | Tüm normal değişkenler (reset ile temizlenir) |
| Retain Memory (NovRAM) | RETAIN değişkenleri |
| Persistent Storage | PERSISTENT değişkenleri (UPS eşliğinde kaydedilir) |

---

# 3. RETAIN vs PERSISTENT — Profesyonel Karşılaştırma

| Davranış | VAR | VAR RETAIN | VAR PERSISTENT |
|----------|-----|-------------|----------------|
| Reset cold | Reinitialize | **Korunur** | **Korunur** |
| Reset origin | Reinitialize | Reinitialize | Reinitialize |
| Download | Reinitialize | **Korunur** | **Korunur** |
| Online change | Korunabilir | **Korunur** | **Korunur** |
| Power loss | Kaybolur | **Korunur** | **Korunur** (önceki shutdown’a göre) |

---

# 4. Retain Handler Mekanizması

Retain Handler, RETAIN değişkenlerini her PLC cycle sonunda NovRAM’e yazar.

Özellikler:

- Yalnızca değişen segmentleri yazar → optimize  
- Donanım türüne göre flash aşınması olabilir  
- Yüksek frekanslı değişkenlerde dikkat edilmelidir  

---

# 5. UPS Gereksinimi & Shutdown İşlemleri

PERSISTENT değişkenler:

- Normalde yalnızca **TwinCAT controlled shutdown** sırasında kaydedilir  
- Bu nedenle makine UPS ile korunmalıdır  
- Aksi takdirde aniden güç kaybında veri henüz kaydedilmemiş olabilir  

İstisna:

```st
FB_WritePersistentData()
```

ile manuel tetikleme yapılabilir.

---

# 6. Reset Davranış Matrisi

### RESET COLD
- RETAIN → ✔ Korunur  
- PERSISTENT → ✔ Korunur  

### RESET ORIGIN
- RETAIN → ❌ Sıfırlanır  
- PERSISTENT → ❌ Sıfırlanır  

### DOWNLOAD
- RETAIN → ✔ Korunur  
- PERSISTENT → ✔ Korunur  

### ONLINE CHANGE
- RETAIN → ✔ Korunur  
- PERSISTENT → ✔ Korunur  

---

# 7. Download, Online Change & Power Loss Davranışları

| Olay | RETAIN | PERSISTENT |
|------|---------|------------|
| Power loss | ✔ Korur | ✔ Korur (son kaydedilen hali) |
| Download | ✔ Korur | ✔ Korur |
| Online change | ✔ Korur | ✔ Korur |
| Manual shutdown | ✔ Korur | ✔ Kesin korur |
| Hard power cut | ✔ Korur | ❌ Son kaydetmeden önce kesintiyse veri kaybolur |

---

# 8. PERSISTENT — Derin Teknik İnceleme

Deklarasyon:

```st
VAR_GLOBAL PERSISTENT
    nHours : DINT;
END_VAR
```

Özellikler:

- Download sonrası bile korunur  
- Reset cold sonrası korunur  
- Reset origin sırasında reinitialize edilir  
- En güvenli değer saklama yöntemi  

Kullanım amaçları:

- Çalışma saati sayaçları  
- Makine konfigürasyonu  
- Kullanıcı ayarları  
- Kimlik bilgilerinin saklanması  

---

# 9. RETAIN — Derin Teknik İnceleme

Deklarasyon:

```st
VAR RETAIN
    nCounter : DINT;
END_VAR
```

Özellikler:

- Cycle sonunda otomatik olarak novRAM’e yazılır  
- Güç kesintilerinde değeri korur  
- Reset origin → sıfırlanır  
- Download sonrası korunur  

Kullanım amaçları:

- Üretim sayaçları  
- Mode/state recovery  
- Güç kesintisi sonrası kaldığı yerden devam etme  

---

# 10. Pointer Kullanımındaki Tehlikeler

TwinCAT **uyarı verir**:

```txt
Avoid POINTER TO inside persistent lists.
```

Çünkü:

- Download sonrası adresler değişir  
- Pointer invalid hale gelir  
- Veri tutarsızlığı veya crash oluşabilir  

Profesyonel kural:
> Persistent/retain bellek = sadece sabit boyutlu, adres bağımsız veri türleri.

---

# 11. AT %I / %Q Kullanım Yasakları

Şu kullanım kesinlikle yasaktır:

```st
VAR_GLOBAL RETAIN
    bInput AT %I0.0 : BOOL; // ❌ Yasak
END_VAR
```

Sebep:

- IO process image mapping bozulur  
- Retain handler yanlış adresleri yazabilir  
- Donanımsal conflict çıkabilir  

---

# 12. Local vs Global RETAIN/PERSISTENT

### Local RETAIN (FB veya PRG içinde)
✔ Geçerlidir → retain memory’e yazılır

### Function içindeki RETAIN
❌ Etkisizdir → retain’e kaydedilmez

### Local PERSISTENT
❌ Etkisizdir → sadece GVL seviyesinde kullanılmalıdır

---

# 13. TcInitOnReset Pragması

Reset davranışını özelleştirmek için:

```st
{attribute 'TcInitOnReset := TRUE'}
```

Etkisi:

- Reset cold sırasında bile değişkeni resetleyebilir  
- Reset origin davranışıyla uyumlu hâle getirir  

---

# 14. Endüstriyel Kullanım Senaryoları

✔ Çalışma saati sayacı  
✔ Üretim miktarı sayaçları  
✔ Proses parametreleri  
✔ Kullanıcı ayarlarının saklanması  
✔ Enerji sayaçları  
✔ Üretim batch takibi  

---

# 15. Hatalı Kullanım Örnekleri

### ❌ Yüksek frekansta değişen RETAIN değişkenleri  
NovRAM aşınmasına yol açabilir.

### ❌ POINTER ile PERSISTENT/RETAIN  
Adres değişikliği ile crash riski.

### ❌ RETAIN + AT kombinasyonu  
Process image hataları oluşur.

### ❌ Local persistent  
Etki yok.

---

# 16. Örnek Kodlar

### RETAIN örneği
```st
VAR RETAIN
    nRem1 : INT;
END_VAR
```

### PERSISTENT örneği
```st
VAR_GLOBAL PERSISTENT
    nVarPers1 : DINT;
    bVarPers2 : BOOL;
END_VAR
```

### RETAIN FB içinde
```st
FUNCTION_BLOCK FB_Count
VAR RETAIN
    nCount : DINT;
END_VAR
```

---

# 17. Sonuç

RETAIN ve PERSISTENT değişkenler:

- TwinCAT’in en kritik veri sürekliliği araçlarıdır  
- Yanlış kullanım ciddi veri kaybı doğurabilir  
- UPS ve shutdown süreçleri mutlaka doğru tasarlanmalıdır  
- Donanımın desteklediği remanent memory türüne göre seçim yapılmalıdır  

---

