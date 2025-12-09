
# TwinCAT FB_exit MASTERCLASS  
Profesyonel PLC OOP Eğitim Dokümanı  
TwinCAT 3 – Yaşam Döngüsü Metotları

---

## 1. FB_exit Nedir?

`FB_exit`, TwinCAT tarafından **otomatik (implicit)** olarak çağrılan bir fonksiyon blok yaşam döngüsü metodudur.  
Görevi:

- FB örneği hafızadan kaldırılmadan önce güvenli kapanışı yapmak  
- Donanım bağlantılarını serbest bırakmak  
- Pointer/dinamik bellek temizlemek  
- Online Change öncesi instance kopyalanmadan önce son temizliği yapmak  

---

## 2. Ne Zaman Çağrılır?

| Durum | FB_exit Çağrılır mı? | Açıklama |
|-------|------------------------|----------|
| Run → Config | ✔ | PLC durdurulurken tüm FB’ler kapatılır |
| Online Change | ✔ | Eski instance kapanır → yenisi oluşturulur |
| TwinCAT kapanışı | ✔ | Shutdown sırasında temizleme yapılır |
| Warm/Cold Start | ✖ | Başlangıçta exit tetiklenmez |
| Explicit delete | ✔ | Instance kaldırılırken |

---

## 3. FB_exit'i Asla Manuel Çağırmamalısın

```
fbSample.FB_exit();   // ❌ ÖNERİLMEZ
```

Neden?

- Yaşam döngüsü bozulur  
- Çift kapanış → donanım kilitlenmesi  
- Core dump oluşabilir  
- Bellek yönetimi zarar görebilir  

TwinCAT bunu **otomatik yönetir**, elle çağırmak tehlikelidir.

---

## 4. Core Dump Oluşumu

FB_exit içinde hata oluşursa TwinCAT otomatik olarak core dump oluşturur.

Konum:

**TwinCAT ≥ 3.1.4026.0**
```
C:\ProgramData\Beckhoff\TwinCAT\3.1\Boot\Plc\CoreDump
```

---

## 5. FB_exit Metodunun Arayüzü

```pascal
METHOD FB_exit : BOOL
VAR_INPUT
    bInCopyCode : BOOL;
END_VAR
```

### Parametre

| Parametre | TRUE Durumu | Anlamı |
|----------|--------------|--------|
| bInCopyCode | Online Change sırasında | Instance kopyalanmadan önce kapatma yapılır |

### Return Value
- **Implicit çağrıda değerlendirilmez**
- Explicit çağrıda (önerilmez) değerlendirilebilir

---

## 6. Kalıtım (Inheritance)

Bir FB başka bir FB’den türetilmişse:

1. Derived FB_exit varsa:  
   **Derived → Base** sırasıyla çalışır  
2. Derived FB_exit yoksa:  
   Base FB_exit otomatik çalışır

---

## 7. FB_exit’in Profesyonel Kullanım Alanları

### ✔ Donanım bağlantılarını kapatma
```pascal
IF hCom <> 0 THEN
    SerialClose(hCom);
    hCom := 0;
END_IF
```

### ✔ Pointer / Dinamik bellek temizliği
```pascal
IF pBuffer <> 0 THEN
    __DELETE(pBuffer);
    pBuffer := 0;
END_IF
```

### ✔ Timer / Task durdurma
```pascal
fbTimer.Stop();
```

### ✔ Log dosyası flush / close
```pascal
fbLog.Close();
```

### ✔ Online Change için özel davranış
```pascal
IF bInCopyCode THEN
    Log('Instance replaced during Online Change');
END_IF
```

---

## 8. Gerçek Dünya Örneği: Endüstriyel Cihaz Sürücüsü

```pascal
FUNCTION_BLOCK FB_ModbusDevice
VAR
    hModbus      : UDINT;
    bConnected   : BOOL;
    pRxBuffer    : POINTER TO BYTE;
END_VAR

METHOD FB_exit : BOOL
VAR_INPUT bInCopyCode : BOOL;
END_VAR

IF bConnected THEN
    LogMessage('Disconnecting Modbus device...');
END_IF

IF hModbus <> 0 THEN
    ModbusClose(hModbus);
    hModbus := 0;
END_IF

IF pRxBuffer <> 0 THEN
    __DELETE(pRxBuffer);
    pRxBuffer := 0;
END_IF

bConnected := FALSE;

FB_exit := TRUE;
```

---

## 9. Profesyonel Kodlama Kuralları

| Kural | Açıklama |
|------|----------|
| Yalnızca cleanup işlemleri yap | Exit hızlı olmalıdır |
| Donanım bağlantılarını düzgün kapat | Port/Soket kilitlenmesini önler |
| Pointer serbest bırakmayı unutma | Memory leak engellenir |
| bInCopyCode değerlendir | Online Change ile ilgili mantığı ayrıştır |

---

## 10. Profesyonel FB_exit Template

```pascal
METHOD FB_exit : BOOL
VAR_INPUT bInCopyCode : BOOL;
END_VAR

IF bInCopyCode THEN
    Log('FB_exit: Online Change cleanup');
END_IF

bRunning := FALSE;

IF hDevice <> 0 THEN
    DeviceClose(hDevice);
    hDevice := 0;
END_IF

fbTask.Stop();

IF pData <> 0 THEN
    __DELETE(pData);
    pData := 0;
END_IF

Log('FB_exit completed.');

FB_exit := TRUE;
```

---

## 11. Özet

Doğru FB_exit kullanımı:

✔ Donanım kaynaklarının güvenli kapanışı  
✔ Memory leak önlenmesi  
✔ Sorunsuz Online Change  
✔ Stabil çalışma  

Yanlış kullanım:

❌ Kilitlenen portlar  
❌ Memory leak  
❌ Core dump  
❌ Bozuk instance yaşam döngüsü  

---


---

