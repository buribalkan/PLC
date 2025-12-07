# 🚚 Konveyör State Machine -- TwinCAT (Beckhoff) Örneği

TwinCAT üzerinde bir **konveyör sistemi için state machine
(durum makinesi)** yapısının nasıl oluşturulacağını açıklar.\
Kod örnekleri Structured Text (ST) dilindedir.

------------------------------------------------------------------------

## 🔵 1. State ENUM Tanımı

``` pascal
{attribute 'qualified_only'}
{attribute 'strict'}
{attribute 'to_string'}
TYPE E_ConvState :
(
    Idle := 0,
    Prepare := 10,
    Start := 20,
    Running := 30,
    Stop := 40,
    Error := 99
);
END_TYPE
```

------------------------------------------------------------------------

## 🔵 2. Konveyör Değişkenleri

``` pascal
PROGRAM MAIN

VAR
    ConvState       : E_ConvState := E_ConvState.Idle;

    StartCmd        : BOOL := FALSE;
    StopCmd         : BOOL := FALSE;
    ResetCmd        : BOOL := FALSE;

    MotorOn         : BOOL := FALSE;
    SpeedSetpoint   : REAL := 0;
    SpeedActual     : REAL := 0;

    SensorPartReady : BOOL := FALSE;
    ErrorDetected   : BOOL := FALSE;

    tStartRamp      : TON;
END_VAR
```

------------------------------------------------------------------------

## 🔵 3. Konveyör State Machine (CASE Yapısı)

``` pascal
CASE ConvState OF

    E_ConvState.Idle:
        MotorOn := FALSE;
        SpeedSetpoint := 0;

        IF StartCmd THEN
            ConvState := E_ConvState.Prepare;
        END_IF


    E_ConvState.Prepare:
        (* Sensör ve güvenlik kontrolü *)
        IF NOT SensorPartReady THEN
            ErrorDetected := TRUE;
            ConvState := E_ConvState.Error;
        ELSE
            ConvState := E_ConvState.Start;
        END_IF


    E_ConvState.Start:
        MotorOn := TRUE;

        (* Yumuşak başlangıç için zamanlayıcı *)
        tStartRamp(IN := TRUE, PT := T#10S);

        (* Ramp tamamlandı mı? *)
        IF tStartRamp.Q THEN
            SpeedSetpoint := 100.0;   (* %100 hız örneği *)
            ConvState := E_ConvState.Running;
        END_IF


    E_ConvState.Running:
        (* Normal çalışma *)
        IF StopCmd THEN
            ConvState := E_ConvState.Stop;
        ELSIF ErrorDetected THEN
            ConvState := E_ConvState.Error;
        END_IF


    E_ConvState.Stop:
        (* Yumuşak duruş *)
        SpeedSetpoint := 0;

        IF SpeedActual <= 1.0 THEN  (* durdu kabul *)
            MotorOn := FALSE;
            ConvState := E_ConvState.Idle;
        END_IF


    E_ConvState.Error:
        MotorOn := FALSE;
        SpeedSetpoint := 0;

        IF ResetCmd THEN
            ErrorDetected := FALSE;
            ConvState := E_ConvState.Idle;
        END_IF

END_CASE

(* Speed Simulation *)
IF MotorOn AND (SpeedSetpoint > 0) THEN
    (* Motor hızlanıyor *)
    IF SpeedActual < SpeedSetpoint THEN
        SpeedActual := SpeedActual + 0.1;
    END_IF

ELSIF SpeedSetpoint = 0 THEN
    (* Motor yavaşlıyor / duruyor *)
    IF SpeedActual > 0 THEN
        SpeedActual := SpeedActual - 0.1;
    ELSE
        SpeedActual := 0;
    END_IF

END_IF


```

------------------------------------------------------------------------

## 🔵 4. Çalışma Prensibi -- Özet

### ▶️ **Idle → Prepare**

StartCmd ile başlar.

### ▶️ **Prepare → Start**

Sensör kontrolü tamamlanır.

### ▶️ **Start → Running**

10 saniyelik hız rampası uygulanır. Abartılı Gözlemleyebilmek için

### ▶️ **Running → Stop veya Error**

Kullanıcı StopCmd verirse Stop'a, Hata oluşursa Error'a geçilir.

### ▶️ **Error → Idle**

ResetCmd ile tekrar başa dönülür.

------------------------------------------------------------------------

## 🔵 5. Gelişmiş Özellikler (Opsiyonel)

-   Encoder ile hız geri besleme
-   Emniyet fonksiyonları (STO, SLS)
-   Ürün sayaç sistemi
-   Manuel / Auto kontrol modu
-   Çoklu konveyör senkronizasyonu
-   Function Block tabanlı modüler state machine

------------------------------------------------------------------------
