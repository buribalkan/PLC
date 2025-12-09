# TwinCAT PROPERTY Eğitim Seti  
## Başlangıç → Orta → İleri → Uzman Seviyeleri

---

# 🟦 1. SEVİYE – BAŞLANGIÇ  
## PROPERTY Nedir?

PROPERTY, TwinCAT PLC içinde veri erişimini kontrollü şekilde yönetmek için kullanılan yapıdır.  
C# dilindeki getter–setter karşılığıdır.

### PROPERTY Türleri:
- **Read Only Property** → sadece okunabilir  
- **Write Only Property** → sadece set edilebilir  
- **Read/Write Property** → hem okunabilir hem değiştirilebilir  
- **Computed Property** → hesaplanan değer döndürür

### PROPERTY Neden Kullanılır?
- Kapsülleme (encapsulation)  
- Veri doğrulama  
- Güvenli erişim  
- HMI/SCADA için temiz arayüz  
- Nesne tabanlı FB tasarımı

---

# 🟦 1.1 Başlangıç Senaryosu: Motor FB

Örnek PROPERTY listesi:
- Speed (READ)  
- TargetSpeed (WRITE)  
- IsRunning (READ)  

Bu yapı motorun kontrolünü düzenli ve güvenli hale getirir.

---

# 🟦 1.2 Başlangıç Egzersizi

**Egzersiz 1:**  
Bir ısıtıcı FB tasarla:
- Temperature (read)  
- Setpoint (write)  
- IsReady (read)

---

# 🟩 2. SEVİYE – ORTA SEVİYE  
## PROPERTY Tasarım Kuralları

### ✔ 1. PROPERTY sadece veri erişimi içindir  
İş mantığı METHOD içinde olmalıdır.

### ✔ 2. PROPERTY hızlı ve hafif olmalıdır  
Ağır hesaplamalar property içinde yapılmaz.

### ✔ 3. PROPERTY, public değişkene göre daha güvenlidir  
Doğrudan değişken açmak yerine property kullanılır.

---

# 🟩 2.1 PROPERTY Türleri – Orta Seviye

### ⭐ Read-Only State Properties
- IsRunning  
- HasError  
- IsReady  

### ⭐ Write-Protected Properties
- TargetSpeed  
- LimitValue  

### ⭐ Computed Properties
- FillPercentage  
- PowerConsumption  

---

# 🟩 2.2 Orta Seviye Senaryo: Tank Sistemi

PROPERTY listesi:
- Level (read)  
- TargetLevel (write)  
- LevelPercent (computed/read)  
- IsFull (read)  
- AlarmState (read)

---

# 🟩 2.3 Orta Seviye Egzersizi

**Egzersiz 2:**  
Bir pompa FB tasarla:
- Flow (read)  
- TargetFlow (write)  
- PowerConsumption (computed)  
- Overload (read)

---

# 🟥 3. SEVİYE – İLERİ SEVİYE  
## İleri PROPERTY Kullanım Teknikleri

### ⭐ 1. Lazy Evaluation Property  
Değer sadece property okununca hesaplanır.

### ⭐ 2. Composite Property  
Birden fazla dahili değerden oluşturulur.  
Örn: `MachineStatus`

### ⭐ 3. Engineering Limit Protected Property  
Yazılan değer otomatik doğrulanır.

### ⭐ 4. Debounced Property  
Deadband veya EMA uygulanmış property.

### ⭐ 5. Scaling Property  
ADC → Volt → Bar gibi dönüştürmeler.

---

# 🟥 3.1 İleri Seviye Senaryo: Kalite Kontrol Sistemi

PROPERTY listesi:
- PassCount (read)  
- FailCount (read)  
- TotalCount (computed)  
- Threshold (write)  
- ImageScore (read)  
- RejectRatePercent (computed)

---

# 🟥 3.2 İleri Egzersizler

**Egzersiz 3:** Encoder FB tasarla  
- Position (read)  
- Velocity (computed)  
- RPM (computed)  
- SetPosition (write, limit controlled)

**Egzersiz 4:** HVAC FB tasarla  
- Temperature (read)  
- Setpoint (write)  
- ComfortIndex (computed)

---

# 🟪 4. SEVİYE – UZMAN  
## Kurumsal PROPERTY Standartları

### ✔ Interface + Property  
Standardize FB arayüzleri:
- IMotor  
- ISensor  
- IAxis  

### ✔ Diagnostic Property Sets  
- StatusCode  
- ErrorDescription  
- WarningBits  
- OperationMode  
- Heartbeat  

### ✔ HMI için Property tabanlı API  
HMI sadece property üzerinden veri alır/yazar.

---

# 🟪 4.1 Uzman Egzersizler

**Egzersiz 5:** Machine FB tasarla  
- MachineState (composite)  
- AlarmSummary (composite)  
- IsOperational (computed)  
- IsInCycle (read)  
- Mode (read/write)

**Egzersiz 6:** Vision System FB tasarla  
- Score  
- Threshold  
- PassRate  
- RejectCount  
- LastDefectDescription  

---


