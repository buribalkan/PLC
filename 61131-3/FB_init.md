# TwinCAT **FB_init MASTERCLASS**
_Tam Kapsamlı Profesyonel Eğitim Dokümanı (Markdown)_

---

# 📘 1. FB_init Nedir?

TwinCAT’te **FB_init**, bir Function Block (FB) örneğinin **ilk oluşturulma anında** otomatik olarak çalışan özel bir sistem metodudur.

FB_init:

- ✔ Her FB instance için **otomatik çağrılır**  
- ✔ Değişkenlerin **zero initialization** veya **explicit initialization** işlemi FB içeriğine girmeden önce tamamlanır  
- ✔ Kullanıcı isterse **explicit olarak tanımlayabilir**  
- ❌ **SUPER^.FB_init çağrısı yapılmaz**  
- ⚠ Debug etmek zordur (breakpoint çalışmayabilir)

FB_init, TwinCAT dünyasında bir nevi “constructor gibidir” fakat **tam bir constructor değildir**.

---

# 🧠 2. FB_init Ne Zaman Çağrılır?

TwinCAT ortamı FB_init’i **aşağıdaki durumlarda otomatik olarak çağırır**:

### 🔹 İlk derleme ve PLC’ye download  
(bInitRetains = TRUE, bInCopyCode = FALSE)

### 🔹 Online Change  
(bInitRetains = FALSE, bInCopyCode = TRUE)

### 🔹 TwinCAT Restart (run → stop → run)  
(bInitRetains = FALSE, bInCopyCode = FALSE)

Bu parametreler FB_init içine otomatik aktarılır.

---

# 🔧 3. FB_init Metodunun Sistem Arayüzü

```pascal
METHOD FB_init : BOOL
VAR_INPUT
    bInitRetains : BOOL; // retain değişkenleri initialize edilmeli mi?
    bInCopyCode  : BOOL; // Online change sonrası instans copy code’a taşınacak mı?
END_VAR
```

Bu iki parametre sayesinde FB_init içinde çalışma moduna göre özel davranışlar tanımlanabilir.

---

# 🚫 4. Neden SUPER^.FB_init Çağrısı Yasaktır?

Çünkü:

- FB_init çağrıldığı anda FB örneği **tamamen initialize edilmiştir**  
- Eğer SUPER^.FB_init çağrılırsa FB iki kez initialize edilmiş olur  
- Alt instance seviyelerinde **yanlış hafıza kullanımı** veya **dynamic memory leak** oluşabilir  
- FB_init gerçek bir OOP constructor değildir

Bu yüzden yasaktır.

---

# ⚠ 5. FB_init Debug Zorlukları

FB_init:

- Çalışma **ilk scan** başlamadan önce gerçekleşir  
- Breakpoint koysanız bile çoğu zaman **kaçırırsınız**  
- Debug için en iyi yöntem → **Log mesajı**, **Watch**, veya **flag bit set** etmektir  

TwinCAT bu nedenle FB_init hatalarında otomatik olarak **core dump** üretir.

---

# 🧨 6. Hata Durumunda Core Dump (Auto Crash Dump)

TwinCAT 3.1 Build 4024.25 ve sonrası:

FB_init, FB_reinit veya FB_exit içinde exception oluşursa:

✔ Runtime otomatik core dump oluşturur  
✔ Dosya konumu:

```
<TC < 4026>      :  C:\TwinCAT\3.1\Boot\Plc
<TC >= 4026.0>   :  C:\ProgramData\Beckhoff\TwinCAT\3.1\Boot\Plc\CoreDump
```

Bu dosya TwinCAT Crash Analyzer ile açılabilir.

---

# 🚨 7. FB_init İçinde Dikkat Edilmesi Gereken Altın Kurallar

### ✔ FB_init çalışırken **FB’nin input değerleri hâlâ gelmemiştir**

Input değişkenine göre initialization yapılacaksa → parametre olarak ekleyin:

Örnek:

```pascal
METHOD FB_init : BOOL
VAR_INPUT
    bInitRetains : BOOL;
    bInCopyCode  : BOOL;
    nInitParam   : INT;
END_VAR
```

---

# 🚫 8. FB_init'i Explicit Çağırmak Neden Kötü Fikirdir?

TwinCAT buna izin verir ama **ŞİDDETLE TAVSİYE EDİLMEZ**.

Çünkü explicit çağrıldığında:

- ✔ Kullanıcı kodunuz tekrar çalışır  
- ❌ Implicit initialization **tekrar yapılır**  
- ❌ Alt FB instance'ları **yeniden initialize edilir**  
- ❌ Dinamik memory allocation kurguları bozulur  
- ❌ Sayaçlar sıfırlanır  
- ❌ Multi-level FB instance şeması bozulur  

Sonuç: Programınız öngörülemez hâle gelir.

---

# 🧩 9. FB_init ile Ek Parametre Kullanımı

TwinCAT FB_init metoduna **ek parametreler tanımlamanıza izin verir**.

Bu sayede:

- FB instance özel parametre alabilir  
- input değişkenleri yerine initialization parametreleri kullanılabilir  

---

# 🧪 ÖRNEK 1 → Seriyel Cihaz İçin Port Numarası

```pascal
METHOD PUBLIC FB_init : BOOL
VAR_INPUT
    bInitRetains : BOOL;
    bInCopyCode  : BOOL;
    nComNum      : INT; // extra parameter
END_VAR
```

Kullanım:

```pascal
fbCom1 : FB_SerialDevice(nComNum := 1);
fbCom0 : FB_SerialDevice(nComNum := 0);
```

---

# 🧪 ÖRNEK 2 → Başlangıç Değeri Parametrik Verme

FB kodu:

```pascal
FUNCTION_BLOCK FB_Sample
VAR
    nStartValue : INT;
END_VAR
```

FB_init:

```pascal
METHOD FB_init : BOOL
VAR_INPUT
    bInitRetains : BOOL;
    bInCopyCode  : BOOL;
    nValue       : INT;
END_VAR

nStartValue := nValue;
```

Kullanım:

```pascal
fbSample1 : FB_Sample(123);  
fbSample2 : FB_Sample(456);
```

---

# 🧪 ÖRNEK 3 → Property + Input + Init Param Bir Arada

FB:

```pascal
FUNCTION_BLOCK FB_Sample
VAR_INPUT
    nInput : INT;
END_VAR
VAR
    nLocalInitParam : INT;
    nLocalProp      : INT;
END_VAR
```

FB_init:

```pascal
METHOD FB_init : BOOL
VAR_INPUT
    bInitRetains : BOOL;
    bInCopyCode  : BOOL;
    nInitParam   : INT;
END_VAR

nLocalInitParam := nInitParam;
```

Property:

```pascal
PROPERTY nMyProperty : INT
nLocalProp := nMyProperty;
```

VB (MAIN):

```pascal
fbSample : FB_Sample(nInitParam := 1) := (nInput := 2, nMyProperty := 3);
```

---

# 🧠 10. Derived Function Blocks (Kalıtım) ve FB_init

Bir FB başka bir FB’den türetilmişse:

### ✔ Base FB_init otomatik olarak çalışır  
### ✔ Derived FB_init tanımlanırsa → Base FB_init **sonra** çalışır  
### ❌ SUPER^.FB_init yoktur  
### ✔ Derived FB_init → base FB_init ile aynı imzayı taşımak ZORUNDADIR  

İsterseniz ek parametreler ekleyebilirsiniz.

---

# 🔍 11. FB_init = Constructor mı?

Hayır.

- FB_init *constructor gibi davranır ama OOP constructor değildir*.  
- FB değişkenleri **FB_init çağrılmadan önce** initialize edilir.  
- FB_init → sadece initialization logic içindir.

---

# 🏆 12. FB_init Masterclass Özeti

| Özellik | Açıklama |
|--------|----------|
| Implicit çalışma | ✔ |
| Explicit çağrı tavsiye edilmez | ✔ |
| Ek parametre desteği | ✔ |
| Debugging zordur | ✔ |
| Core dump desteği | ✔ |
| Inheritance davranışı | ✔ |
| Input’lar FB_init sırasında hazır değildir | ✔ |

Bu doküman FB_init konusunda uzman seviyesinde bilgi sağlar.

---



