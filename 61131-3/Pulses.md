# Pulses in PLC
PLC’de ayarlanabilir pulse (genlik değil; süre/period ayarlanabilir darbe üretimi) üç farklı şekilde kodlanır:

- Zamanlayıcı (TON/TOF) ile manual toggle
- TP (Pulse Timer) ile
- Duty Cycle + Period hesabıyla tam PWM benzeri çözüm
- (İleri Seviye) High-speed task / PTO / PWM donanım modülü ile

Aşağıda bunlar PLC standard ST diliyle, adım adım ve örneklerle gösterilmiştir.

1. Basit Ayarlanabilir Pulse (TON Toggle yöntemi)

Bu yöntemle “X ms açık, X ms kapalı” tarzında pulse üretilir.

En temel, her PLC’de çalışan yöntem budur.
```pascal
// =================
//  Değişkenler:    
// =================

FUNCTION_BLOCK BasitPulse
VAR
    PulseOut : BOOL;
    T        : TON;
    Toggle   : BOOL := FALSE;
    OnTime   : TIME := T#500ms;
    OffTime  : TIME := T#500ms;
END_VAR

// =================
//  Döngü mantığı:  
// =================

IF NOT T.Q THEN
    T(IN := TRUE, PT := SEL(Toggle, OffTime, OnTime));
ELSE
    T(IN := FALSE);
    Toggle := NOT Toggle;
END_IF;
PulseOut := Toggle;
```
MAIN'den çağırma:
```pascal
PROGRAM MAIN
VAR
    MyPulse     : BasitPulse;   // FB örneği
    OutputLamp  : BOOL;
END_VAR
// ======================
// Pulse üreticiyi çalıştır
MyPulse();  
// PulseOut değerini MAIN'de kullan
OutputLamp := MyPulse.PulseOut;
```
- Toggle = TRUE iken “ON süresi”
- Toggle = FALSE iken “OFF süresi”
- ON/OFF süreleri değiştirilebilir
- Bu pulse tam program cycle hızında güncellenir

Bu, PLC’de “ayarlanabilir kare dalga üretimi”nin en sade halidir.

2. TP (Pulse Timer) kullanarak ayarlanabilir 

genişlik

 pulse

TP = Tek Pulse Zamanlayıcısı
Giriş tetiklendiğinde PT kadar süreyle pulse verir.

```pascal
VAR
    TP1 : TP;
    Trigger : BOOL;
    PulseWidth : TIME := T#250ms; // Ayarlanabilir
    PulseResult : BOOL;
END_VAR

TP1(IN := Trigger, PT := PulseWidth);
PulseResult := TP1.Q;
```
Ne işe yarar?

- “Trigger TRUE olduğu anda 250 ms pulse ver”
- Pulse width ayarlanabilir
- Sürekli kare dalga olmaz, sadece tetik geldiğinde çıkış verir.

Trigger yerine
```pascal
PROGRAM MAIN
VAR
    MyPulse     : BasitPulse;   // FB örneği
    OutputLamp  : BOOL;
	  TP1 : TP;
    Trigger : BOOL;
    PulseWidth : TIME := T#1000MS; // Ayarlanabilir
	  PulseResult : BOOL;
END_VAR


// Pulse üreticiyi çalıştır
MyPulse();  

// PulseOut değerini MAIN'de kullan
OutputLamp := MyPulse.PulseOut;

TP1(IN := OutputLamp, PT := PulseWidth);
PulseResult := TP1.Q;
```









3. Period ve Duty Cycle ile PWM benzeri ayarlanabilir pulse

Bu yöntem profesyoneldir; örneğin:

- Periyot: 100 ms
- Duty: %30

Çıkış 30 ms ON, 70 ms OFF olacak.
```pascal
VAR
    Tcycle : TON;
    PulseOut : BOOL;
    Period : TIME := T#100ms;     // Ayarlanabilir
    Duty   : REAL := 0.3;         // %30
    OnTime : TIME;
END_VAR
// ================================
OnTime := REAL_TO_TIME(REAL(Period) * Duty);
Tcycle(IN := TRUE, PT := Period);
IF Tcycle.ET <= OnTime THEN
    PulseOut := TRUE;
ELSE
    PulseOut := FALSE;
END_IF;
// Periyot bittiğinde yeniden başlat
IF Tcycle.Q THEN
    Tcycle(IN := FALSE);
END_IF;
``` 
Avantajları:

- Duty cycle ve period runtime ayarlanabilir
- PWM tarzı davranış sağlar
- Analog kontrol (fan hızı, valf, heater) için kullanılabilir










4. Micro-second hassasiyette PTO/PWM (Gerçek donanımsal pulse)





Bu, PLC’nin motion/HS modülünde yapılır.





Örneğin Codesys’te:



MC_PulseGen(

    Execute        := Enable,

    Frequency      := PulseHz,

    DutyCycle      := DutyPercent,

    Output         := AxisOutput

);

Beckhoff TwinCAT:

PwmGen(

    Enable     := Enable,

    Frequency  := Hz,

    DutyCycle  := Duty

);

Siemens TIA’da:



PTO (Pulse Train Output)
PWM (Pulse Width Modulation)




Bu donanımsal çıkışlar CPU scan time’dan bağımsız çalışır.









5. Hangi yöntem hangi durumda?



Yöntem

Kullanım

Özellik

TON/Toggle

Basit kare dalga

En yaygın, her PLC’de çalışır

TP Timer

Trigger bazlı pulse

Sadece tek pulse, sürekli değil

Duty Cycle + Period

PWM benzeri kontrol

Fan/valf/ısıtıcı kontrolünde iyi

PTO/PWM Donanım

Servo/Stepper / hızlı kontrol

Mikro-saniye hassas









6. Sana Uygun Final Örnek (Ayarlanabilir pulse FB)





Profesyonel bir çözüm:
```pascal
FUNCTION_BLOCK FB_PulseGenerator
VAR_INPUT
    Enable   : BOOL;
    Period   : TIME;
    Duty     : REAL;   // 0.0 - 1.0
END_VAR
VAR_OUTPUT
    Q : BOOL;
END_VAR
VAR
    T  : TON;
    OnTime : TIME;
END_VAR

IF NOT Enable THEN
    T(IN := FALSE);
    Q := FALSE;
    RETURN;
END_IF;
// Duty hesapla
OnTime := REAL_TO_TIME(TO_REAL(Period) * Duty);
// Timer çalıştır
T(IN := TRUE, PT := Period);
// Output üret
IF T.ET <= OnTime THEN
    Q := TRUE;
ELSE
    Q := FALSE;
END_IF;
// Periyot bitince sıfırla
IF T.Q THEN
    T(IN := FALSE);
END_IF;

//   Kullanım:

PulseGen(
    Enable := TRUE,
    Period := T#200ms,
    Duty   := 0.4,   // %40
    Q      => PulseOut
);
```
Bu FB:

- Ayarlanabilir
- Period + Duty ile PWM sağlar
- Her PLC’de çalışır
- State machine içinde kullanılabilir
- Analog kontrol için uygundur
