# TwinCAT **FB_reinit MASTERCLASS**  
_Tam Kapsamlı Profesyonel Eğitim Dokümanı (Markdown Formatı)_  

---

# 📘 1. FB_reinit Nedir?

**FB_reinit**, TwinCAT’te *sadece belirli durumlarda* çalışan bir yeniden-initialization metodudur.  

Bu metodun kullanılma amacı:  
Bir **Online Change** sırasında, FB’nin hafıza düzeni (signature) değiştiğinde:

- Yeni FB instance hafızası oluşturulur  
- Eski instance copy code içine taşınır  
- Bu yeni instance'ın doğru şekilde yeniden başlatılması gerekir  

İşte **FB_reinit tam bu anda otomatik olarak çağrılır.**

Bu nedenle FB_reinit:

- ✔ Normal çalışma sırasında çalışmaz  
- ✔ Sadece Online Change + signature change durumunda devreye girer  
- ✔ Kullanıcı isterse explicit tanımlayabilir  
- ✔ Ama explicit çağrılması **tavsiye edilmez**  

---

# ⚙ 2. FB_reinit Ne Zaman Otomatik Çağrılır?

FB_reinit şu durumda **implicit olarak çalışır**:

### 🔥 **Online Change → Function Block Signature Değişikliği**

Örneğin FB değiştiyse:

- Yeni bir değişken eklendi  
- Var olan değişken tipi değişti  
- Yapı boyutu değişti  
- Miras alınan FB’de değişiklik oldu  

TwinCAT yeni instance oluşturur → copy code işlemi yapar → **FB_reinit çağrılır**.

---

# ⚠ 3. Explicit FB_reinit Çağırmak Neden Sakıncalıdır?

TwinCAT uyarır:

> “Explicit calling is NOT recommended.”

Çünkü explicit çağrı şunları tetikler:

- Implicit initialization tekrar eder  
- Hafıza blokları yeniden oluşturulur  
- Alt seviye FB instance’ları yeniden initialize edilir  
- Dinamik hafıza yönetimi çöker  
- Sayaçlar, zamanlayıcılar, referanslar bozulur  

FB_reinit yalnızca sistem tarafından çağrılmak üzere tasarlanmıştır.

---

# 🧨 4. FB_reinit İçinde Hata Olursa → Core Dump!

TwinCAT 3.1 Build 4024.25 ve sonrası:

FB_reinit içinde exception olursa TwinCAT **otomatik core dump** üretir.

### Core dump klasörleri:

```
TC < 4026:
    C:\TwinCAT.1\Boot\Plc

TC >= 4026:
    C:\ProgramData\Beckhoff\TwinCAT.1\Boot\Plc\CoreDump
```

Bu dosyalar TwinCAT’in Crash Analyzer araçları ile açılabilir.

---

# 🧩 5. FB_reinit Metodunun Arayüzü

```pascal
METHOD FB_reinit : BOOL
```

Dikkat:

- ✔ Parametre yok  
- ✔ Return value implicit call’da *sistem tarafından kullanılmaz*  
- ✔ Explicit çağrı yapılırsa → return value manuel işlenebilir  

---

# 🧬 6. FB_reinit ile Inheritance (Kalıtım) Kullanımı

Derived FB → Base FB'in FB_reinit metodunu çağırmak **programcının sorumluluğundadır**.

Yani:

```pascal
SUPER^.FB_reinit();
```

Bu çağrı:

- ✔ Beklenen davranıştır  
- ✔ Derived DP init → Base DP init sırası sağlanır  

FB_init’te böyle bir zorunluluk yoktu, FB_reinit’te ise vardır.

---

# 🧪 7. Basit FB_reinit Örneği

```pascal
FUNCTION_BLOCK FB_Device
VAR
    bActive : BOOL;
    nValue  : INT;
END_VAR

METHOD FB_reinit : BOOL
bActive := FALSE;
nValue := 0;
```

Bu metod, Online Change sonrası yeni FB instance'da çalışır.

---

# 🧪 8. Derived FB Örneği (SUPER^.FB_reinit)

### Base FB:

```pascal
FUNCTION_BLOCK FB_Base
VAR
    nBaseValue : INT;
END_VAR

METHOD FB_reinit : BOOL
nBaseValue := 100;
```

### Derived FB:

```pascal
FUNCTION_BLOCK FB_Child EXTENDS FB_Base
VAR
    nChildValue : INT;
END_VAR

METHOD FB_reinit : BOOL
SUPER^.FB_reinit();       // Base behavior preserved
nChildValue := 200;
```

Çalışma sırası:

1. Base.FB_reinit  
2. Child.FB_reinit  

---

# 🧠 9. FB_init ve FB_reinit Arasındaki Fark

| Özellik | FB_init | FB_reinit |
|--------|---------|-----------|
| Çalışma zamanı | Her instance creation | Online Change + signature change |
| Input parametreleri | Var | Yok |
| SUPER^ kullanım durumu | Yasak | Gereklidir |
| Initialization türü | İlk başlangıç | Yeniden-initialization |
| Explicit çağrı | Tavsiye edilmez | Tavsiye edilmez |

---

# 🎯 10. FB_reinit Kullanım Amaçları

FB_reinit şu amaçla kullanılmalıdır:

- Online Change sonrası bağlantıları yeniden kurmak  
- Donanım handle’larını yeniden almak  
- File handler, socket, serial port vb. yeniden başlatmak  
- Internal cache yapılarını resetlemek  
- Custom memory structure rebuild etmek  

Pro kullanıcılar bu mekanizmayı:

✔ Online program update sırasında state korunması  
✔ Non-retain variable rebuild  
✔ Reconnection logic  

için kullanır.

---

# 🧪 11. Gerçek Dünya Full Örnek – Haberleşme Handler Yeniden Bağlantı

```pascal
FUNCTION_BLOCK FB_CommHandler
VAR
    bConnected : BOOL;
    hSocket    : DINT;
END_VAR

METHOD FB_reinit : BOOL
// Online change sonrası bağlantı kopacağı için:
bConnected := FALSE;
hSocket := -1;
// Bağlantıyı yeniden açmak üst katmanda yapılır
```

Bu kod:

- Online Change sonrası socket handle artık geçersiz olduğu için  
- Yeni bir instance yaratıldığında bağlantıyı sıfırlar  

---

# 🏆 12. FB_reinit Masterclass Özeti

✔ Sadece Online Change + signature değişiminde çalışır  
✔ Explicit çağrı **önerilmez**  
✔ Derived FB’lerde SUPER^.FB_reinit zorunludur  
✔ Exception durumunda core dump oluşur  
✔ Return value implicit çağrıda kullanılmaz  
✔ Parametresiz yöntemdir  
✔ Yeni instance’un tam initialization’ını sağlar  

Bu doküman FB_reinit konusunu **mühendislik seviyesinde tam kapsamlı** sunar.

---



formatlarında dışa aktarabilirim.

