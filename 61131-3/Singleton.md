# Singleton Pattern

✅ 1. Tekil bir kaynağı yönetmek için

Bazı fonksiyonlar sistemde yalnızca bir kere bulunmalıdır. Örneğin:

- Logger (log yazan yapı)
- Haberleşme yöneticisi (Modbus, TCP/IP, ADS…)
- Veri tabanı ya da dosya yöneticisi
- PID Controller gibi tekil kontrol yapıları

Bu yapılar çoğaltılırsa çakışmalar, hatalı veri erişimi veya istenmeyen davranışlar ortaya çıkabilir.

✅ 2. Global veriye kontrollü erişim sağlamak

Singleton sayesinde:

- Global değişken kullanmadan,
- Veri bütünlüğünü koruyarak,
  
Tüm programdan erişilebilen ortak bir yapı elde edilir.

✅ 3. Bellek ve kaynak yönetimini kolaylaştırmak

PLC’de gereksiz FB instance’ları oluşturmak bellek kullanımını artırır.
Singleton ile yalnızca tek instance yaratılır → kaynak kullanımı optimize olur.

🧩 TwinCAT’te Singleton Nasıl Kullanılır?
```pascal
FUNCTION_BLOCK FB_Singleton
VAR
    // Instance data
END_VAR

METHOD PUBLIC GetInstance : POINTER TO FB_Singleton
VAR
    static pInst : POINTER TO FB_Singleton;
END_VAR

IF pInst = 0 THEN
    pInst := ADR(This^);
END_IF

GetInstance := pInst;
```

Bu yapı ile FB’in yalnızca tek bir örneği kullanılmış olur.

🎯 Özet:

TwinCAT’te Singleton pattern:

- ✔ Tek bir FB instance’ının kullanılmasını sağlar

- ✔ Global servisler veya yöneticiler için idealdir

- ✔ Bellek ve kaynak kullanımını azaltır

- ✔ Veri tutarlılığını ve kontrolü artırır


# C# Singleton Tasarım Deseni --- Basit ve Net Açıklama

## 1. Normal (Singleton olmayan) sınıf

Önce sıradan bir sınıf düşün:

``` csharp
public class Logger
{
    public void Log(string message)
    {
        Console.WriteLine(message);
    }
}
```

Bunu projede her yerde şöyle kullanabilirsin:

``` csharp
var logger1 = new Logger();
var logger2 = new Logger();

logger1.Log("Mesaj 1");
logger2.Log("Mesaj 2");
```

Burada önemli nokta:

-   `logger1` ve `logger2` **iki farklı nesne**
-   Yani RAM'de **iki ayrı Logger instance'ı** var

Bazı sınıfların tekil olması gerekir:

-   Tek bir config yöneticisi
-   Tek bir database bağlantısı
-   Tek bir donanım sürücü yöneticisi

İşte Singleton burada devreye girer:

> **"Bu sınıftan uygulama boyunca sadece 1 tane oluşturulsun."**

------------------------------------------------------------------------

## 2. Mantık: "Kimse `new` yapamasın"

Singleton'ın ilk kuralı: Constructor gizli olmalı.

``` csharp
public class Logger
{
    private Logger() { }   // Artık dışarıdan new YASAK
}
```

Artık:

``` csharp
var l = new Logger(); // HATA — erişilemez
```

------------------------------------------------------------------------

## 3. Tek örneği içeride static olarak tutmak

``` csharp
public class Logger
{
    private static Logger _instance;  // Tek örneği saklar
    private Logger() { }
}
```

`static` olduğu için tüm programda **tek bir alan** vardır.

------------------------------------------------------------------------

## 4. Erişim noktası: `Instance` property

``` csharp
public class Logger
{
    private static Logger _instance;
    private Logger() { }

    public static Logger Instance
    {
        get
        {
            if (_instance == null)
                _instance = new Logger();
            return _instance;
        }
    }

    public void Log(string message)
    {
        Console.WriteLine(message);
    }
}
```

Kullanım:

``` csharp
Logger.Instance.Log("Mesaj A");
Logger.Instance.Log("Mesaj B");
```

### Çalışma mantığı:

-   İlk çağrıda `_instance == null` → `new Logger()` → örnek
    oluşturulur
-   Sonraki çağrılarda yeni örnek oluşturulmaz → hep aynı döner

------------------------------------------------------------------------

## 5. Senaryo ile düşünelim

``` csharp
void A() => Logger.Instance.Log("A başladı");
void B() => Logger.Instance.Log("B hata verdi");
void C() => Logger.Instance.Log("C bitti");
```

A, B, C birbirinden habersizdir ama **aynı Logger**'ı kullanır.

------------------------------------------------------------------------

## 6. `sealed` neden kullanılır?

``` csharp
public sealed class Logger
{
    ...
}
```

Kimsenin:

``` csharp
class MyLogger : Logger
```

şeklinde miras alıp Singleton düzenini bozmasını engeller.

------------------------------------------------------------------------

## 7. En sade modern hali (önerilen)

``` csharp
public sealed class Logger
{
    public static Logger Instance { get; } = new Logger();
    private Logger() { }

    public void Log(string message)
    {
        Console.WriteLine(message);
    }
}
```

-   Program başlarken **bir kere** oluşturulur
-   Sonra her çağrıda aynı instance döner
-   Thread-safe ve basittir

------------------------------------------------------------------------

## 8. Kafada tutman gereken 3 kural

### ✔ **1. `private constructor`**

Kimse `new` yapamaz.

### ✔ **2. `static` alan**

Tek örnek burada saklanır.

### ✔ **3. `public static Instance`**

Herkes bu kapıdan içeri girer → hep aynı nesne döner.

------------------------------------------------------------------------

Bu üçü yan yana geliyorsa:

> **Bu bir Singleton'dır.**



-------------------------------------------------------------------------



# TwinCAT'te Singleton Mantığının C# Singleton ile Bire Bir Karşılaştırmalı Açıklaması

Aşağıda önce **C# Singleton yapısını**, ardından bunun **TwinCAT ST
üzerindeki karşılığını adım adım** göreceksin.

------------------------------------------------------------------------

# 1. C# Singleton Kodunu Hatırlayalım

``` csharp
public class Logger
{
    private static Logger _instance;
    private Logger() { }

    public static Logger Instance
    {
        get
        {
            if (_instance == null)
                _instance = new Logger();

            return _instance;
        }
    }

    public void Log(string message)
    {
        Console.WriteLine(message);
    }
}
```

Bu yapı şunu sağlar:

-   Constructor **private** → kimse new yapamaz
-   `_instance` **static** → tek örnek saklanır
-   `Instance` property → herkes aynı nesneyi alır

------------------------------------------------------------------------

# TwinCAT'te neden aynı şekilde yazamayız?

Çünkü:

-   TwinCAT'te **private constructor yok**
-   TwinCAT'te **static class member** tam C#'taki gibi yok

Ama fikir aynı olabilir:

> **"Tek bir global FB instance'ı + o instance'ı döndüren bir
> fonksiyon"**

------------------------------------------------------------------------

# 2. Logger FB'sini yazalım (C# sınıfına karşılık)

``` iecst
FUNCTION_BLOCK FB_Logger
VAR
    sLastMessage : STRING(255);
END_VAR

METHOD PUBLIC Log
VAR_INPUT
    sMessage : STRING;
END_VAR
    sLastMessage := sMessage;

    // Gerçek loglama – örnek: TwinCAT log'a yaz
    AdsLogStr(ADSLOG_MSGTYPE_LOG, '%s', sMessage);
END_METHOD
```

Bu FB, C#'taki `public class Logger` ile aynı amaca hizmet eder.

------------------------------------------------------------------------

# 3. Tek "singleton" instance (GLOBAL)

C#'taki:

``` csharp
private static Logger _instance;
```

TwinCAT karşılığı:

``` iecst
VAR_GLOBAL
    g_Logger : FB_Logger;  // Singleton instance
END_VAR
```

Artık **programın her yerinden erişilebilen tek örnek** var.

------------------------------------------------------------------------

# 4. C#'taki `Instance` yerine ST'de accessor fonksiyon

C#:

``` csharp
public static Logger Instance { get { ... } }
```

TwinCAT ST karşılığı:

``` iecst
FUNCTION LoggerInstance : REFERENCE TO FB_Logger
VAR
END_VAR
LoggerInstance REF= g_Logger;
```

Kullanım:

``` iecst
LoggerInstance().Log('Mesaj');
```

Bu tam olarak C#'taki:

``` csharp
Logger.Instance.Log("Mesaj");
```

ile aynı mantıktır.

------------------------------------------------------------------------

# 5. TwinCAT kullanım örneği (C# ile bire bir karşılaştırma)

C#:

``` csharp
Logger.Instance.Log("Mesaj");
```

TwinCAT:

``` iecst
PROGRAM MAIN
VAR
    logger : REFERENCE TO FB_Logger;
END_VAR

logger REF= LoggerInstance();
logger.Log('Mesaj');
```

Hatta daha kısa:

``` iecst
LoggerInstance().Log('Mesaj');
```

Hepsi **aynı tek FB_Logger instance** üzerinden çalışır.

------------------------------------------------------------------------

# 6. C#'taki Lazy Initialization konusu

C#:

``` csharp
if (_instance == null)
    _instance = new Logger();
```

→ İlk kullanımda nesne oluşturulur.

TwinCAT'te global FB örnekleri zaten sistem başında oluşturulur; yani
pratikte C#'taki şu modele benzer:

``` csharp
public sealed class Logger
{
    public static Logger Instance { get; } = new Logger();
    private Logger() { }
}
```

Ama istersen TwinCAT'te de "ilk kullanımda init" yapabilirsin:

``` iecst
FUNCTION_BLOCK FB_Logger
VAR
    bInitialized  : BOOL := FALSE;
    sLastMessage  : STRING(255);
END_VAR

METHOD PUBLIC Log
VAR_INPUT
    sMessage : STRING;
END_VAR

    IF NOT bInitialized THEN
        // İlk kullanımda yapılacak hazırlıklar
        bInitialized := TRUE;
    END_IF

    sLastMessage := sMessage;
    AdsLogStr(ADSLOG_MSGTYPE_LOG, '%s', sMessage);
END_METHOD
```

Bu mantıksal "lazy init" sağlar.

------------------------------------------------------------------------

# 7. Alternatif: Singleton'ı ayrı bir FB ile sarmalamak

C# Singleton'a daha çok benzetmek için:

``` iecst
FUNCTION_BLOCK FB_LoggerSingleton
VAR
    _logger : FB_Logger;  // Tek gerçek instance
END_VAR

METHOD PUBLIC GetInstance : REFERENCE TO FB_Logger
VAR
END_VAR
    GetInstance REF= _logger;
END_METHOD
```

Globalde:

``` iecst
VAR_GLOBAL
    g_LoggerSingleton : FB_LoggerSingleton;
END_VAR
```

Kullanım:

``` iecst
PROGRAM MAIN
VAR
    logger : REFERENCE TO FB_Logger;
END_VAR

logger REF= g_LoggerSingleton.GetInstance();
logger.Log('Test');
```

Bu da aynı mantıktır:

C#:

``` csharp
Logger.Instance
```

TwinCAT:

``` iecst
g_LoggerSingleton.GetInstance()
```

------------------------------------------------------------------------

# 8. Özet (C# → TwinCAT ST eşlemesi)

 | C#                                      | TwinCAT ST karşılığı                                                     |
|-----------------------------------------|---------------------------------------------------------------------------|
| `private static Logger _instance;`      | `VAR_GLOBAL g_Logger : FB_Logger;`                                       |
| `private Logger()`                      | TwinCAT'te constructor gizlenemez → *tek global instance disiplini*      |
| `public static Logger Instance { get; }`| `FUNCTION LoggerInstance : REFERENCE TO FB_Logger` veya `GetInstance()`   |
| `Logger.Instance.Log("...")`            | `LoggerInstance().Log('...')` veya `g_LoggerSingleton.GetInstance().Log('...')` |

------------------------------------------------------------------------

TwinCAT'in kısıtlarına rağmen, **C# Singleton'ın mantığı bire bir
uygulanmış oluyor**:

-   Tek örnek ✔
-   Global erişim ✔
-   Tüm kod aynı tek FB üzerinden çalışıyor ✔




--------------------------------------------------------------------------

# PLC'de Singleton Kullanım Alanları --- Profesyonel ve Net Açıklama

Aşağıda **PLC'de Singleton'ın nerede, neden ve nasıl kullanıldığını**
sade, net ve profesyonel şekilde listeliyorum.

------------------------------------------------------------------------

# 1. Global Logger (Loglama Sistemi)

PLC'de loglama çoğu zaman sistem geneldir:

-   Alarmlar, uyarılar
-   Durum değişimleri
-   Proses hataları
-   Debug mesajları

Eğer her FB kendi logger'ını oluştursa:

-   RAM ziyan olur
-   Log yapısı dağılır
-   Mesajlar karışır

Bu yüzden **tek bir Logger FB (singleton)** kullanılır.
Sistemdeki tüm FB'ler **aynı logger üzerinden** log yazar.

------------------------------------------------------------------------

# 2. Configuration Manager (Makine Ayarları Yönetimi)

Bir makinenin:

-   Reçete ayarları
-   Hız limitleri
-   Güvenlik parametreleri
-   Kalibrasyon değerleri

genellikle tek merkezde saklanır.

Bu ayarları yöneten FB'nin **bir tane** olması gerekir.

Tüm diğer FB'ler **aynı config kaynağından** okuma yapmalıdır.

Bu nedenle Config Manager çoğu PLC projesinde **singleton mantığıyla
yazılır**.

------------------------------------------------------------------------

# 3. Data Layer / Database Arayüzü

PLC'nin dış sistemlerle konuşan FB'leri:

-   SQL client
-   MQTT client
-   REST API client
-   ADS client
-   OPC UA client
-   SQLite FB

Bu tür FB'ler sadece **bir kez** çalışmalıdır.

Aksi takdirde:

-   Port çakışması
-   Çift bağlantı açma
-   Mesaj karışması
-   Kimlik doğrulama sorunları

oluşur.

Bu nedenle haberleşme / database arayüz FB'leri **singleton** olarak
kullanılmalıdır.

------------------------------------------------------------------------

# 4. Donanım Erişim FB'leri (I/O, Seri Hatlar, Fieldbus)

Cihazlarla konuşan FB'ler ikinci kez örneklenemez:

-   RS485/RS232 sürücüsü
-   EtherCAT özel terminal sürücüleri
-   Modbus Master
-   RFID okuyucu
-   CANbus driver
-   Servo/robot kontrol FB'leri

İki instance kullanılmaya çalışılırsa:

-   Aynı port iki kez açılır → **hata**
-   Frame çakışır → **mesaj bozulur**
-   Cihaz cevap vermez

Bu FB'ler **zaten doğal singleton** gibidir.

------------------------------------------------------------------------

# 5. Alarm Manager / Event Manager

PLC'de alarm yönetimi typik olarak **merkezi** çalışır:

-   Tüm FB'ler alarm gönderir
-   Alarm yöneticisi tek noktadır
-   HMI tek bir alarm kaynağına bağlanır

Bu nedenle:

-   AlarmTable
-   ErrorManager
-   EventLogger

gibi yapılar **tek instance** tutulur.

------------------------------------------------------------------------

# 6. State Machine Manager (Merkezi Durum Makinesi)

Bir makinede genellikle:

-   Tüm makineyi yöneten **bir master state machine** bulunur
-   Alt modüller bu FSM ile haberleşir

Makine:

-   Aynı anda iki farklı "RUN" durumunda olamaz
-   İki state machine işletmek kontrol kaosuna yol açar

Bu yüzden FSM'ler genellikle **singleton** olur.

------------------------------------------------------------------------

# 7. Recipe Manager (Reçete Yönetimi)

Üretim makinelerinde:

-   Reçete **bir kez** yüklenir
-   Tüm FB'ler bu reçeteyi okur

Dolayısıyla tek merkez zorunludur → Singleton.

------------------------------------------------------------------------

# 8. Communication Watchdog / Heartbeat Manager

PLC'nin üst sisteme heartbeat göndermesi için:

-   Tek bir yönetici FB gerekir
-   Bunu iki kez çalıştırmak yanlış süre ölçümüne neden olur

Bu FB her yere konulmaz → **bir tane** olur.

------------------------------------------------------------------------

# 9. Machine Time / Clock Provider

TwinCAT zamanı sistemden alır ama özel ihtiyaçlar için:

-   Cycle time hesaplayıcı
-   Makine çalışma süresi
-   Özel zaman motoru

gibi FB'ler tek instance kullanılabilir.

------------------------------------------------------------------------

# 10. Safety Gateway / Permission Manager

Makinede güvenlik ve kullanıcı izinleri genellikle merkezidir:

-   User access level (Login/Logout)
-   Operatör / mühendis / admin hak kontrolü
-   Safety door FB'leri

Bu tür sistemler **tek merkezden yönetilir** → Singleton yapısı ideal
olur.

------------------------------------------------------------------------

# ÖZET: PLC'de Singleton Nerede ve Neden Kullanılır?

  | Kullanım Alanı              | Neden Singleton Gerekir?                          |
|-----------------------------|---------------------------------------------------|
| Logger                      | Tüm modüller aynı log kaynağını kullanmalıdır     |
| Config Manager              | Makine ayarları tek merkezden gelir               |
| Database / Network Client   | Port/bağlantı çakışmalarını engellemek            |
| Hardware Drivers            | Aynı cihazla 2 bağlantı olamaz                    |
| Alarm Manager               | HMI merkezi alarm yapısına ihtiyaç duyar          |
| State Machine               | Makineyi yöneten tek FSM olmalıdır                |
| Recipe Manager              | Tüm proses aynı reçeteyi okur                     |
| Communication Watchdog      | Tek heartbeat mekanizması olmalıdır               |
| Permission / Safety Manager | Kullanıcı ve safety merkezi yönetilir             |


------------------------------------------------------------------------

Bu liste **gerçek endüstriyel PLC projelerinde Singleton'ın nerede
kullanıldığını** en net şekilde özetler.


