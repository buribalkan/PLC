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

# Singleton Ne İşe Yarar?

**Bir şeyin program içinde yalnızca 1 tane bulunmasını garanti eder.
Herkes o tek nesneyi kullanır.**

------------------------------------------------------------------------

## Örnekler

### 🔌 1. Tek Modem

Evde **1 tane modem** vardır → herkes ona bağlanır.

### 📝 2. Tek Log Sistemi

Uygulamada **1 tane logger** vardır → tüm modüller logları buraya yazar.

### 🔢 3. Tek Sayaç

Sistemde **1 sayaç** vardır → herkes aynı sayacı artırır.

### 🌐 4. Tek Haberleşme Yöneticisi

PLC'de **1 iletişim yöneticisi** vardır → tüm FB'ler aynı bağlantıyı
kullanır.

------------------------------------------------------------------------

Singleton'ın özü:
- > **"Tek bir nesne, herkes tarafından ortak kullanılsın."**
  


# Neden Tek Bir Nesne (Singleton) Kullanılır?

Singleton'ın özü şudur:

> **Bazı nesnelerin programda yalnızca 1 tane olması gerekir, çünkü
> birden fazla olursa sistem bozulur.**

Aşağıda bunun *neden zorunlu olduğunu* en net şekilde açıklıyorum.

------------------------------------------------------------------------

# 1️⃣ Kaynak Çakışmasını Önlemek İçin

Bazı şeylerin fiziksel veya mantıksal olarak **birden fazla örneği
olamaz**.

### Örnekler:

-   Seri port
-   Modbus master
-   TCP bağlantısı
-   Dosya yazıcı
-   Donanım sürücüleri

Eğer iki FB aynı donanıma bağlanmaya çalışırsa:

-   *"Port already in use"*
-   Bağlantı çakışması
-   Mesaj kaybı
-   Cihazın cevap vermemesi

Bu nedenle:

> **Tek bir bağlantı yöneticisi olmalıdır.**
> → Singleton

------------------------------------------------------------------------

# 2️⃣ Veriyi Merkezi ve Tutarlı Tutmak İçin

Bazı bilgiler tek bir merkezde bulunmalıdır:

-   Config (ayarlar)
-   User permissions
-   Makine durumu (FSM)
-   Reçete bilgileri

Eğer bunlar farklı FB'lere dağılırsa:

-   Veri uyumsuzluğu
-   Güncellenmeyen bölümler
-   Güvensiz davranış
-   Hatalı süreç yönetimi

Bu yüzden:

> **Merkezi veri kaynağı = Singleton**

------------------------------------------------------------------------

# 3️⃣ Bellek ve Performans Tasarrufu İçin

Her modül kendi bağlantısını, logger'ını, buffer'ını açarsa:

-   Gereksiz bellekte yer kaplar
-   Gereksiz işlemci kullanılır
-   Gereksiz bağlantı açılır
-   Sistem karmaşıklaşır

Tek bir instance kullanmak:

-   Daha hızlı
-   Daha temiz
-   Daha ekonomik

------------------------------------------------------------------------

# 4️⃣ Tüm Sistem Aynı Davranışı Paylaşsın Diye

Bazı görevler **ortak ve merkezi** olmalıdır:

-   Tek Logger → herkes buraya yazar
-   Tek State Machine → tüm modüller aynı makine durumunda
-   Tek Reçete yöneticisi → herkes aynı veriyi kullanır
-   Tek Watchdog → tek heartbeat mekanizması

Bunların çoğaltılması sistemin davranışını bozar.

Bu yüzden Singleton şarttır.

------------------------------------------------------------------------

# 🧠 Özet

> **Singleton "tek olsun" diye değil, "çok olunca bozuluyor" diye
> vardır.**

Çünkü bazı nesneler:

-   ✔ Tek olmalıdır
-   ✔ Merkezi olmalıdır
-   ✔ Paylaşımlı olmalıdır
-   ✔ Çoğaltılması tehlikelidir

Singleton, bu problemi çözen tasarım desenidir.

------------------------------------------------------------------------

# 🔥 10 Saniyelik Hayat Benzetmesi

-   Evde bir tane modem vardır → herkes ona bağlanır
-   Bir şirkette bir tane muhasebe vardır → her departman onunla
    çalışır
-   Ülkede bir tane nüfus müdürlüğü vardır → herkes buraya gider

**Neden?**
Çünkü çok olursa düzen bozulur.

Programlama dünyasında da ⇒ Singleton.

  
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

# PLC'de Config Manager Neden Singleton Olmalıdır?

Aşağıda **Config Manager'ın neden Singleton olması gerektiği**,
**TwinCAT'te nasıl uygulanacağı** ve **C# karşılığıyla bire bir
ilişkisi** eksiksiz şekilde açıklanmıştır.

------------------------------------------------------------------------

# 1. Neden Config Manager Singleton Olmalı?

Makine yapılandırması tek bir merkezden yönetilir:

-   hız parametreleri
-   zamanlayıcı ayarları
-   limitler
-   PID ayarları
-   IO offsetleri
-   reçete varsayılan değerleri
-   proses parametreleri

Eğer Config Manager'ın birden fazla instance'ı olursa:

-   farklı modüller farklı config okur → **davranış tutarsızlığı**
-   değişiklik bir modüle gider diğerine gitmez → **parametre
    uyuşmazlığı**
-   proses kontrolü güvenilmez hâle gelir

Bu nedenle **Config Manager her PLC projesinde tek bir adet olmalıdır.**

Bu tam anlamıyla Singleton gereksinimidir.

------------------------------------------------------------------------

# 2. Genel Tasarım

TwinCAT ST tarafında Config Manager Singleton şu yapı ile kurulur:

-   `FB_ConfigManager` → config değerlerini yöneten FB
-   `GVL_Config` → tek instance
-   `ConfigManagerInstance()` → C#'taki `ConfigManager.Instance`
    karşılığı
-   Tüm modüller bu instance'a referans alır

Bu yapı **C# Singleton'ın PLC karşılığıdır**.

------------------------------------------------------------------------

# 3. Config Manager FB -- `FB_ConfigManager`

Bu FB config parametrelerini saklar, yükler, değiştirir.

``` iecst
FUNCTION_BLOCK FB_ConfigManager
VAR
    nSpeedLimit : INT := 1000;
    nTimeoutMs  : INT := 500;
    rKp         : REAL := 1.0;
    rKi         : REAL := 0.5;
    rKd         : REAL := 0.1;

    bInitialized : BOOL := FALSE;
END_VAR
```

## Init metodu (isteğe bağlı)

``` iecst
METHOD PUBLIC Init
IF NOT bInitialized THEN
    // Dosyadan / remanent memory'den / ADS'ten config yüklenebilir
    bInitialized := TRUE;
END_IF
```

## Okuma Metodları

``` iecst
METHOD PUBLIC GetSpeedLimit : INT
GetSpeedLimit := nSpeedLimit;
END_METHOD

METHOD PUBLIC GetTimeout : INT
GetTimeout := nTimeoutMs;
END_METHOD
```

## Değiştirme Metodu

``` iecst
METHOD PUBLIC SetSpeedLimit
VAR_INPUT
    speed : INT;
END_VAR
nSpeedLimit := speed;
```

------------------------------------------------------------------------

# 4. Tekil Instance -- GVL içinde tanımlanır

C# karşılığı:

``` csharp
private static ConfigManager _instance;
```

TwinCAT karşılığı:

``` iecst
VAR_GLOBAL
    g_ConfigManager : FB_ConfigManager;  // Singleton instance
END_VAR
```

Bu instance program boyunca **tek bir tanedir**.

------------------------------------------------------------------------

# 5. C#'taki Instance Property Karşılığı

C#:

``` csharp
public static ConfigManager Instance => _instance;
```

TwinCAT ST:

``` iecst
FUNCTION ConfigManagerInstance : REFERENCE TO FB_ConfigManager
ConfigManagerInstance REF= g_ConfigManager;
```

Bu fonksiyon, Config Manager'ın tek örneğini geri döndürür.

------------------------------------------------------------------------

# 6. Kullanım Örneği

### Modül 1 -- Motion Control

``` iecst
speed := ConfigManagerInstance().GetSpeedLimit();
```

### Modül 2 -- Robot FB

``` iecst
timeout := ConfigManagerInstance().GetTimeout();
```

### Parametre Güncelleme

``` iecst
ConfigManagerInstance().SetSpeedLimit(1200);
```

Tüm modüller aynı instance'a erişir → **tek kaynaktan config okurlar**.

------------------------------------------------------------------------

# 7. Lazy Initialization Gerekiyorsa

TwinCAT'te global FB'ler otomatik oluşturulur.

Ama config'inizi **ilk kullanımda** yüklemek isterseniz:

``` iecst
IF NOT ConfigManagerInstance().bInitialized THEN
    ConfigManagerInstance().Init();
END_IF
```

Bu, C#'taki:

``` csharp
if (!Instance.Initialized) Instance.Init();
```

ile aynı mantıktır.

------------------------------------------------------------------------

# 8. Dosyadan Config Yükleme (İleri Seviye)

Gerçek makinelerde config değerleri genellikle:

-   JSON
-   .txt
-   .ini
-   TwinCAT persistent variable
-   ADS üzerinden SCADA

ile yüklenir.

FB içinde şu tür metotlar olabilir:

``` iecst
METHOD PUBLIC LoadFromFile
VAR_INPUT
    sFilePath : STRING;
END_VAR
// Tc2_Utilities FILE_READ kullanılabilir
```

``` iecst
METHOD PUBLIC SaveToFile
VAR_INPUT
    sFilePath : STRING;
END_VAR
```

İstersen bu metodların **tam çalışan implementasyonunu** da
üretebilirim.

------------------------------------------------------------------------

# 9. Özet -- PLC Config Manager Singleton Mimarisi

 | Yapı                    | İşlev                 | C# Eşdeğeri                                   |
|-------------------------|-----------------------|-----------------------------------------------|
| FB_ConfigManager        | Config iş mantığı     | class ConfigManager                           |
| g_ConfigManager         | Tek instance          | private static ConfigManager _instance        |
| ConfigManagerInstance() | Singleton accessor    | public static ConfigManager Instance          |
| Init / Get / Set        | Config operasyonları  | Class methods                                 |


📌 **PLC'de C# Singleton'ın tam karşılığı:**
→ *Global instance + accessor function + FB içinde iş mantığı*


--------------------------------------------------------------------------------


# PLC / TwinCAT – Singleton Network/Database Client Tasarımı  
**Port/bağlantı çakışmasını engellemek için mimari yaklaşım**

---

## 1. Problem: Aynı kaynağa birden fazla client FB

PLC tarafında “client” dediğimiz şeyler genelde:

- TCP/UDP client (SCADA/PC/veritabanı)
- Modbus TCP Master
- MQTT Client
- HTTP/REST Client
- SQL/ODBC Client
- Seri port sürücüsü (RS232/RS485)

Hepsinin ortak özelliği:

- Aynı IP:Port
- Aynı seri port
- Aynı fieldbus kanalı

üzerinden iletişim kurmasıdır. **Birden fazla FB instance’ı aynı kaynağı açmaya çalışırsa çakışma olur.**

### Tipik sorunlar:
- Bir FB bağlanır, diğeri *“port already in use”* hatası alır.
- Aynı soketten gelen data iki FB tarafından sahiplenilir → protokol bozulur.
- Seri port iki kere açılır → timeout & random hatalar.
- Aynı DB bağlantısına birden fazla FB girer → transaction karmaşası.

**Çözüm:**  
Kaynağı yöneten yalnızca **tek bir global FB** olmalı → *Singleton Client*.

---

## 2. Çözüm: Client FB’yi Singleton yapmak

PC tarafındaki:

```
DatabaseClient.Instance
```

kavramının PLC eşdeğeri:

- Tek bir global instance
- Herkes o instance’a referans ile erişir
- Connect/Disconnect tek noktadan yapılır

Avantaj:

- Aynı portu iki kez açamazsın → çakışma önlenir.
- Hata & retry yönetimi merkezi olur.
- Tüm veri trafiği kontrol altında olur.

---

## 3. TwinCAT – TCP Client Singleton Örneği

### 3.1 Client FB

```iecst
FUNCTION_BLOCK FB_NetClient
VAR
    sIpAddress  : STRING(15) := '192.168.0.10';
    nPort       : UINT := 502;

    bConnected  : BOOL;
    bBusy       : BOOL;
    bError      : BOOL;
    nErrId      : UDINT;

    fbTcpClient : FB_TcpClient;  // temsilî
END_VAR

METHOD PUBLIC Connect
VAR_INPUT
    sIp  : STRING;
    port : UINT;
END_VAR
    sIpAddress := sIp;
    nPort      := port;

    IF NOT bConnected THEN
        bConnected := TRUE; // örnek
    END_IF
END_METHOD

METHOD PUBLIC Disconnect
VAR
END_VAR
    IF bConnected THEN
        bConnected := FALSE;
    END_IF
END_METHOD

METHOD PUBLIC Send
VAR_INPUT
    pData : POINTER TO BYTE;
    nSize : UDINT;
END_VAR
    IF NOT bConnected THEN
        bError := TRUE;
        nErrId := 16#0001;
        RETURN;
    END_IF
END_METHOD
```

---

## 3.2 Global Singleton Instance

```iecst
// GVL_Comm.TcGVL
VAR_GLOBAL
    g_NetClient : FB_NetClient;
END_VAR
```

---

## 3.3 Accessor Function

```iecst
FUNCTION NetClientInstance : REFERENCE TO FB_NetClient
NetClientInstance REF= g_NetClient;
```

Kullanım:

```iecst
NetClientInstance().Connect('192.168.0.10', 502);
NetClientInstance().Send(ADR(Buffer), SIZEOF(Buffer));
```

---

## 4. Port/bağlantı çakışması bu şekilde nasıl engelleniyor?

Örnek FB’ler:

- FB_Robot  
- FB_VisioSystem  
- FB_AlarmReporter  

Her biri SCADA’ya TCP ile veri atıyor.

### ❌ Kötü Tasarım
Her biri kendi FB_TcpClient instance’ını açar → aynı IP/port’a 3 bağlantı denemesi → biri bağlanır, diğerleri hata verir.

### ✔️ İyi Tasarım (Singleton)
- Tek g_NetClient var
- Bütün modüller onun üzerinden Send() çağırır
- Portu sadece *tek yer* açabildiği için çakışma imkansızdır

---

## 5. Database Client için aynı yöntem

Aynı model şu durumlarda uygulanır:

- SQL Client
- MQTT Client
- OPC UA Client
- HTTP/REST Client

### Örnek:

```iecst
VAR_GLOBAL
    g_DbClient : FB_DbClient;
END_VAR
```

Ve kullanım yine:

```
DbClientInstance().Query(...)
```

Sonuç:

- Aynı DB’ye birden fazla bağlantı açılmaz
- Connection pool tek noktadan yönetilir

---

## 6. PLC açısından özet

Singleton Network/DB Client ne sağlar?

- Aynı IP/port/seri port için **tek sahip**
- Port/bağlantı çakışması **mimari seviyede engellenir**
- Retry / timeout / reconnect stratejisi tek merkezden yönetilir
- Her FB’nin kafasına göre bağlantı açması engellenir
- Makinede stabilite ve deterministik davranış ciddi yükselir

---



# PLC / TwinCAT — **Recipe Manager (Singleton) Tasarımı**  
**Kayıpsız içerik – Markdown dosyası**

---

## 1. Recipe Manager PLC’de ne işe yarar?

Recipe Manager’ın temel görevi, **ürüne bağlı setpoint’leri ve parametreleri merkezi olarak tutmak** ve tüm makinenin tek bir tarif (recipe) üzerinden çalışmasını sağlamaktır.

Tipik olarak içinde şunlar bulunur:

- Süreler  
- Sıcaklıklar  
- Hızlar  
- Miktarlar  
- Toleranslar  

HMI üzerinden:

1. Operatör Recipe seçer (Recipe 1, Product A, Job 25…)
2. PLC, seçilen recipe'nin parametrelerini yükler
3. Proses FB’leri (dolum, ısıtma, paketleme vb.) bu parametreleri okur

**Kısa özet:**  
> *“Şu an hangi ürün çalışıyor ve o ürünün parametreleri neler?”*  
Bu soruyu yanıtlayan **tek merkez**.

### Tek merkez (Singleton) olmasının nedeni:
- Aynı anda iki farklı aktif recipe istemezsin  
- Tüm makine **tek kaynaktan** parametre almalı  
- Tutarsız ürün çalışması engellenir

---

## 2. Recipe Manager vs Config Manager

Aralarındaki fark çok nettir:

| Yapı | Açıklama | Değişim sıklığı |
|------|----------|------------------|
| **Config Manager** | Makinenin genel ayarları, güvenlik limitleri, hız limitleri, kalibrasyon değerleri | Nadiren değişir |
| **Recipe Manager** | Ürüne göre değişen üretim parametreleri | Sık değişir |

Her ikisi de Singleton yapılır, ancak **rolleri tamamen farklıdır**.

---

## 3. TwinCAT’te Recipe Manager Singleton Tasarımı

Aşağıdaki yapı kullanılır:

- **ST_Recipe** → Bir tarifin (product recipe) datası  
- **FB_RecipeManager** → Tarif listesini, aktif tarifi yöneten FB  
- **GVL_Recipe** → Tek (global) instance  
- **RecipeManagerInstance()** → C#’taki `RecipeManager.Instance` karşılığı  
- Proses FB’leri → Her zaman bu instance üzerinden okur  

---

## 3.1 Recipe Struct (ST_Recipe)

```iecst
TYPE ST_Recipe :
STRUCT
    sName            : STRING(50);
    rTargetWeight    : REAL;
    rTolerance       : REAL;
    tFillTime        : TIME;
    rConveyorSpeed   : REAL;
END_STRUCT
END_TYPE
```

---

## 3.2 Recipe Manager FB

### Temel FB:

```iecst
FUNCTION_BLOCK FB_RecipeManager
VAR
    aRecipes         : ARRAY[1..20] OF ST_Recipe;
    nRecipeCount     : INT := 0;
    nActiveRecipeIdx : INT := 0;
    bInitialized     : BOOL := FALSE;
END_VAR
```

### Init — Default tarifleri yükleme

```iecst
METHOD PUBLIC Init
IF NOT bInitialized THEN
    nRecipeCount := 2;

    aRecipes[1].sName := 'Product A';
    aRecipes[1].rTargetWeight := 100.0;
    aRecipes[1].rTolerance := 2.0;
    aRecipes[1].tFillTime := T#3S;
    aRecipes[1].rConveyorSpeed := 0.5;

    aRecipes[2].sName := 'Product B';
    aRecipes[2].rTargetWeight := 250.0;
    aRecipes[2].rTolerance := 5.0;
    aRecipes[2].tFillTime := T#5S;
    aRecipes[2].rConveyorSpeed := 0.8;

    nActiveRecipeIdx := 1;
    bInitialized := TRUE;
END_IF
```

### Recipe seçme (HMI’den)

```iecst
METHOD PUBLIC SelectRecipe : BOOL
VAR_INPUT
    index : INT;
END_VAR

IF (index >= 1) AND (index <= nRecipeCount) THEN
    nActiveRecipeIdx := index;
    SelectRecipe := TRUE;
ELSE
    SelectRecipe := FALSE;
END_IF
```

### Aktif recipe’i okuma

```iecst
METHOD PUBLIC GetActiveRecipe : ST_Recipe
VAR emptyRecipe : ST_Recipe;
END_VAR

IF (nActiveRecipeIdx >= 1) AND (nActiveRecipeIdx <= nRecipeCount) THEN
    GetActiveRecipe := aRecipes[nActiveRecipeIdx];
ELSE
    GetActiveRecipe := emptyRecipe;
END_IF
```

---

## 3.3 Singleton Instance (GVL)

```iecst
VAR_GLOBAL
    g_RecipeManager : FB_RecipeManager;
END_VAR
```

---

## 3.4 Accessor Function (Instance getter)

```iecst
FUNCTION RecipeManagerInstance : REFERENCE TO FB_RecipeManager
RecipeManagerInstance REF= g_RecipeManager;
```

> TwinCAT için C# `RecipeManager.Instance` karşılığıdır.

---

## 4. Kullanım Senaryoları

### 4.1 PLC Start’ta Init

```iecst
PROGRAM MAIN
VAR
    bInitDone : BOOL := FALSE;
END_VAR

IF NOT bInitDone THEN
    RecipeManagerInstance().Init();
    bInitDone := TRUE;
END_IF
```

---

### 4.2 HMI’nin Recipe seçmesi

```iecst
VAR
    iSelectedRecipeIndex : INT;
    bSelectRecipeCmd     : BOOL;
    bSelectOk            : BOOL;
END_VAR

IF bSelectRecipeCmd THEN
    bSelectRecipeCmd := FALSE;
    bSelectOk := RecipeManagerInstance().SelectRecipe(iSelectedRecipeIndex);
END_IF
```

---

### 4.3 Proses FB’lerinin tarifi kullanması

```iecst
FUNCTION_BLOCK FB_Filler
VAR
    stActiveRecipe : ST_Recipe;
END_VAR

METHOD PUBLIC Cyclic
    stActiveRecipe := RecipeManagerInstance().GetActiveRecipe();
    // Process vars:
    // stActiveRecipe.rTargetWeight
    // stActiveRecipe.rTolerance
    // stActiveRecipe.tFillTime
    // stActiveRecipe.rConveyorSpeed
END_METHOD
```

Bu sayede:

- Tüm modüller **aynı recipe’den beslenir**
- Recipe değişikliği tüm makineye tek noktadan yansır
- Kontrol algoritmaları tutarlı çalışır

---

## 5. Neden Singleton burada kritik?

### Singleton olmazsa ortaya çıkan problemler:
- Her FB kendi "aktif recipe"sini tutabilir → çelişki
- A proses Product A’da, B proses Product B’de kalabilir
- HMI bir yeri günceller, başka FB’ler eski recipe’de kalır
- Makinenin durumu belirsizleşir

### Singleton olduğunda:
- Aktif recipe **tek noktada**
- Tüm modüller aynı tarife bakar
- Değişiklikler anında herkes tarafından görülür
- SCADA / raporlama için sistem tutarlıdır

---

# PLC / TwinCAT — **Alarm Manager (Singleton) Tasarımı**
**Kayıpsız içerik – Markdown dosyası**

---

## 1. Alarm Manager PLC’de ne yapar?

Alarm Manager, PLC tarafında **tüm alarmların merkezi yönetim noktasıdır**.

Görevleri:

- Sistem ve proses alarmlarını toplamak  
- Sensör, motor, güvenlik ve iletişim hatalarını alarm mantığına dönüştürmek  
- Her alarmın:
  - **ID**
  - **Mesaj**
  - **Kaynak (Axis1, Motor3, Filler vb.)**
  - **Durum (aktif/pasif/acked)**
  - **Zaman bilgisi** (opsiyonel)
    gibi özelliklerini saklamak  
- HMI/SCADA alarm ekranına tek veri kaynağı olmak  

**Kısaca:**  
> “Makinede şu anda hangi alarmlar var?” sorusunun tek doğru yanıtı bu modüldedir.

Bu nedenle Alarm Manager modülünün *tek bir merkezi kaynak* olması çok kritiktir.

---

## 2. Neden Singleton olmalı?

Eğer Alarm Manager **tekil (singleton)** olmazsa modüller kendi alarm listelerini tutabilir. Bu şu sorunlara yol açar:

### ❌ Singleton olmazsa:
- HMI aynı anda 5 farklı alarm listesini okumak zorunda kalır  
- Aynı alarm iki farklı yerde takip edilebilir  
- Alarm geçmişi toplamak zorlaşır  
- Ack/clear işlemleri modüller arasında tutarsız olur  
- “Makinenin alarm durumu nedir?” sorusu cevapsız kalır  

### ✔️ Singleton olduğunda:
- Bütün alarmlar **tek listede toplanır**
- HMI sadece **tek listeye** bakar
- Tüm alarm mantığı **tek yerden** yönetilir (raise/clear/ack)
- Event log, SQL, CSV kayıtları tek merkezden yapılır
- Tüm makine için **global HasActiveAlarms()** gibi sorgular mümkün olur

---

## 3. TwinCAT’te Alarm Manager Singleton Mimari

Aşağıdaki yapı önerilir:

- **ST_Alarm** → Alarm verisini tutan struct  
- **FB_AlarmManager** → Alarm mantığını yöneten FB  
- **GVL_Alarm** → Global tek instance (`g_AlarmManager`)  
- **AlarmManagerInstance()** → C#’taki `AlarmManager.Instance` karşılığı  
- Modüller → Alarm üretmek için sadece bu FB’yi kullanır  

---

## 4. Alarm Struct — ST_Alarm

```iecst
TYPE ST_Alarm :
STRUCT
    nId         : UDINT;         // Alarm ID
    sSource     : STRING(50);    // Kaynak: "Axis1", "Motor3" vb.
    sMessage    : STRING(80);    // Alarm açıklaması
    bActive     : BOOL;          // Aktif mi?
    bAcked      : BOOL;          // Operator tarafından ack'lenmiş mi?
END_STRUCT
END_TYPE
```

### Alarm ID constant örneği
```iecst
VAR_GLOBAL CONSTANT
    ALARM_ID_MOTOR_OVERCURRENT : UDINT := 1;
    ALARM_ID_SAFETY_DOOR_OPEN  : UDINT := 2;
    ALARM_ID_COMM_ERROR        : UDINT := 3;
END_VAR
```

---

## 5. Alarm Manager FB — FB_AlarmManager

### Temel yapı

```iecst
FUNCTION_BLOCK FB_AlarmManager
VAR
    aAlarms       : ARRAY[1..100] OF ST_Alarm;
    nAlarmCount   : INT := 0;
    bInitialized  : BOOL := FALSE;
END_VAR
```

### Init — tüm listeyi temizler

```iecst
METHOD PUBLIC Init
VAR i : INT; END_VAR
IF NOT bInitialized THEN
    FOR i := 1 TO 100 DO
        aAlarms[i].nId     := 0;
        aAlarms[i].sSource := '';
        aAlarms[i].sMessage:= '';
        aAlarms[i].bActive := FALSE;
        aAlarms[i].bAcked  := FALSE;
    END_FOR
    nAlarmCount := 0;
    bInitialized := TRUE;
END_IF
```

---

### Yardımcı metod: Alarm ID → index bulma

```iecst
METHOD PRIVATE FindAlarmIndexById : INT
VAR_INPUT nId : UDINT; END_VAR
VAR i : INT; END_VAR

FindAlarmIndexById := 0;
FOR i := 1 TO nAlarmCount DO
    IF aAlarms[i].nId = nId THEN
        FindAlarmIndexById := i;
        RETURN;
    END_IF
END_FOR
```

---

### Alarm Raise (aktif etme)

```iecst
METHOD PUBLIC RaiseAlarm
VAR_INPUT
    nId      : UDINT;
    sSource  : STRING;
    sMessage : STRING;
END_VAR
VAR idx : INT; END_VAR

idx := FindAlarmIndexById(nId);

IF idx = 0 THEN
    IF nAlarmCount < 100 THEN
        nAlarmCount := nAlarmCount + 1;
        idx := nAlarmCount;

        aAlarms[idx].nId     := nId;
        aAlarms[idx].sSource := sSource;
        aAlarms[idx].sMessage:= sMessage;
    ELSE
        RETURN; // liste dolu
    END_IF
END_IF

aAlarms[idx].bActive := TRUE;
aAlarms[idx].bAcked  := FALSE;
```

---

### Alarm Clear (koşul düzelince)

```iecst
METHOD PUBLIC ClearAlarm
VAR_INPUT nId : UDINT; END_VAR
VAR idx : INT; END_VAR

idx := FindAlarmIndexById(nId);

IF idx <> 0 THEN
    aAlarms[idx].bActive := FALSE;
END_IF
```

---

### Alarm Ack (operator onayı)

```iecst
METHOD PUBLIC AckAlarm
VAR_INPUT nId : UDINT; END_VAR
VAR idx : INT; END_VAR

idx := FindAlarmIndexById(nId);

IF idx <> 0 THEN
    IF aAlarms[idx].bActive THEN
        aAlarms[idx].bAcked := TRUE;
    END_IF
END_IF
```

---

### HMI için alarm okuma (index bazlı)

```iecst
METHOD PUBLIC GetAlarmByIndex : ST_Alarm
VAR_INPUT index : INT; END_VAR
VAR emptyAlarm : ST_Alarm; END_VAR

IF (index >= 1) AND (index <= nAlarmCount) THEN
    GetAlarmByIndex := aAlarms[index];
ELSE
    GetAlarmByIndex := emptyAlarm;
END_IF
```

---

### Aktif alarm var mı? (global durum kontrolü)

```iecst
METHOD PUBLIC HasActiveAlarms : BOOL
VAR i : INT; END_VAR

HasActiveAlarms := FALSE;

FOR i := 1 TO nAlarmCount DO
    IF aAlarms[i].bActive THEN
        HasActiveAlarms := TRUE;
        RETURN;
    END_IF
END_FOR
```

---

## 6. Singleton Yapısı (GVL + Accessor)

### Global instance:

```iecst
VAR_GLOBAL
    g_AlarmManager : FB_AlarmManager;
END_VAR
```

### Instance getter:

```iecst
FUNCTION AlarmManagerInstance : REFERENCE TO FB_AlarmManager
AlarmManagerInstance REF= g_AlarmManager;
```

Kullanım:

```iecst
AlarmManagerInstance().RaiseAlarm(...);
AlarmManagerInstance().ClearAlarm(...);
AlarmManagerInstance().AckAlarm(...);
IF AlarmManagerInstance().HasActiveAlarms() THEN ...
```

**C# karşılığı:**  
```csharp
AlarmManager.Instance.RaiseAlarm(...)
```

---

## 7. Kullanım Örneği – Motor FB Alarm Üretimi

```iecst
FUNCTION_BLOCK FB_Motor
VAR_INPUT
    sName : STRING(20);
END_VAR
VAR
    bOvercurrent : BOOL;
END_VAR

METHOD PUBLIC Cyclic
IF bOvercurrent THEN
    AlarmManagerInstance().RaiseAlarm(
        nId      := ALARM_ID_MOTOR_OVERCURRENT,
        sSource  := sName,
        sMessage := 'Motor overcurrent'
    );
ELSE
    AlarmManagerInstance().ClearAlarm(ALARM_ID_MOTOR_OVERCURRENT);
END_IF
END_METHOD
```

### Bu ne sağlar?

- Alarm mantığı dağıtılmaz → merkezde toplanır  
- HMI tek yerden okur  
- Hangi modül alarm üretirse üretsin listeye eklenir  

---

## 8. HMI Tarafında Alarm Listeleme

```iecst
PROGRAM PLC_AlarmView
VAR
    i          : INT;
    stAlarm    : ST_Alarm;
    bHasActive : BOOL;
END_VAR

bHasActive := AlarmManagerInstance().HasActiveAlarms();

FOR i := 1 TO 100 DO
    stAlarm := AlarmManagerInstance().GetAlarmByIndex(i);
    IF stAlarm.nId <> 0 THEN
        // HMI’ya aktar:
        // stAlarm.sSource
        // stAlarm.sMessage
        // stAlarm.bActive
        // stAlarm.bAcked
    END_IF
END_FOR
```

---

## 9. Özet

Alarm Manager Singleton olduğunda:

- Alarmlar tek merkezde tutulur  
- Raise/Clear/Ack işlemleri tek noktadan yönetilir  
- HMI/SCADA karmaşadan kurtulur  
- Tüm makinenin alarm durumu **global olarak** izlenir  
- Modüller “alarm saklamak” yerine sadece **bildirim yapar**

**Port/Connection Singleton mantığıyla aynı:**  
> “Kaynak tek, yönetici tek.”

---



# PLC / TwinCAT — **Hardware Driver (Singleton) Tasarımı**
**Kayıpsız içerik – Markdown dokümanı**

---

## 1. PLC’de “Hardware Driver” ne demek?

PLC tarafında “driver”, bir **donanım kaynağıyla doğrudan iletişim kuran modül**dür. Bu kaynak genelde:

- RS232 / RS485 seri port
- Modbus RTU / Modbus TCP master
- EtherCAT özel terminal driver’ı (ağırlık modülü, IO-Link master vb.)
- CANopen master / özel CAN protokol sürücüleri
- Barkod okuyucu / RFID reader / kamera protokolü
- Üreticiye özel servo / robot haberleşme FB’leri

Ortak özellik:

> **Arka planda tek bir fiziksel kaynak vardır.**

- Tek port: COM1 / RS485 hattı  
- Tek CAN ID / tek node  
- Tek EtherCAT terminal  
- Tek robot bağlantısı  

Ama kod yazarken yanlışlıkla **birden fazla FB’yi aynı kaynağa eriştirip çakıştırmak** mümkündür.  

Bu yüzden driver FB’nin **Singleton** olması gerekir.

---

## 2. Neden hardware driver Singleton olmalı?

### Donanım tek olduğu için, driver’ı kontrol eden de tek olmalıdır.

Eğer 2 FB aynı anda:

- Aynı RS485’e bağlanırsa → *port already in use*
- Aynı Modbus hattında master olmaya kalkarsa → *frame collision*
- Aynı EtherCAT terminalini kontrol ederse → *random timeout*
- Aynı cihaza komut yazarsa → cihaz cevap vermez / kilitlenir

Bu durum sahada tipik olarak şöyle raporlanır:

> “Makine arada bir sapıtıyor.”  
> “Bazen iletişim gidiyor, resetleyince düzeliyor.”  

Bunların neredeyse tamamı **driver çakışması**dır.

### Çözüm
- Donanımla konuşan **tek FB** olacak.
- Diğer tüm modüller bu FB’ye istek gönderecek.

Yani:

> “Donanımı yöneten tek bir sahip (owner) var.”

---

## 3. Mimari: Driver FB + Global Singleton + Accessor

Kalip:

### 1) FB_XxxDriver  
- Port açma / kapama  
- Gönderme / okuma  
- Timeout / retry / buffer yönetimi  
- Cihazın tüm low-level mantığı burada

### 2) Global instance (Singleton)
```iecst
g_XxxDriver : FB_XxxDriver;
```

### 3) Accessor function
```iecst
XxxDriverInstance() : REFERENCE TO FB_XxxDriver
```

### 4) Üst seviye FB’ler (tartı, barkod, robot…)
- Donanımı **doğrudan** kullanmaz  
- Sadece DriverInstance() üzerinden konuşur  

---

## 4. Örnek: RS485 için Singleton Driver

Senaryo:  
RS485’e bir tartı ya da genel bir cihaz bağlı. TwinCAT serial library ile haberleşiyorsun.

---

### 4.1 Driver FB – `FB_SerialDriver`

```iecst
FUNCTION_BLOCK FB_SerialDriver
VAR
    sPortName  : STRING(20) := 'COM1';
    nBaudRate  : DINT := 9600;

    bPortOpen  : BOOL;
    bError     : BOOL;
    nErrId     : UDINT;

    fbSerial   : FB_SerialLine; // Temsili low-level FB
END_VAR
```

---

### Port Açma

```iecst
METHOD PUBLIC OpenPort
VAR_INPUT
    sPort : STRING;
    nBaud : DINT;
END_VAR

IF NOT bPortOpen THEN
    sPortName := sPort;
    nBaudRate := nBaud;

    // fbSerial.sPort := sPortName;
    // fbSerial.nBaudRate := nBaudRate;
    // fbSerial.bOpen := TRUE;

    bPortOpen := TRUE;
END_IF
```

---

### Port Kapama

```iecst
METHOD PUBLIC ClosePort
IF bPortOpen THEN
    // fbSerial.bClose := TRUE;
    bPortOpen := FALSE;
END_IF
```

---

### Data Gönderme

```iecst
METHOD PUBLIC Send
VAR_INPUT
    pData : POINTER TO BYTE;
    nSize : UDINT;
END_VAR

IF NOT bPortOpen THEN
    bError := TRUE;
    nErrId := 16#0001;
    RETURN;
END_IF

// fbSerial.pTxBuffer := pData;
// fbSerial.nTxSize := nSize;
// fbSerial.bSend := TRUE;
```

---

### Data Alma

```iecst
METHOD PUBLIC Receive : UDINT
VAR_INPUT
    pBuffer  : POINTER TO BYTE;
    nMaxSize : UDINT;
END_VAR

IF NOT bPortOpen THEN
    RETURN 0;
END_IF

// fbSerial.pRxBuffer := pBuffer;
// fbSerial.nRxMaxSize := nMaxSize;
// fbSerial.bReceive := TRUE;
// RETURN fbSerial.nRxReceived;

RETURN 0;
```

Bu FB’nin ana prensibi:

> Donanım portuna **sadece bu FB** dokunur.

---

## 4.2 Global Singleton Instance

```iecst
// GVL_Driver.TcGVL
VAR_GLOBAL
    g_SerialDriver : FB_SerialDriver;
END_VAR
```

---

## 4.3 Accessor Function

```iecst
FUNCTION SerialDriverInstance : REFERENCE TO FB_SerialDriver
SerialDriverInstance REF= g_SerialDriver;
```

Artık kodun her yerinde:

```iecst
SerialDriverInstance().OpenPort('COM1', 9600);
SerialDriverInstance().Send(ADR(buf), SIZEOF(buf));
```

Dediğinde **aynı driver** ile konuşuyorsun.

---

## 4.4 Üst Seviye FB’nin Driver Kullanması (Tartı örneği)

```iecst
FUNCTION_BLOCK FB_Scale
VAR
    wLastWeight : REAL;
    aTxBuf      : ARRAY[0..15] OF BYTE;
    aRxBuf      : ARRAY[0..31] OF BYTE;
END_VAR
```

### Cyclic metodunda:

```iecst
METHOD PUBLIC Cyclic
VAR
    nRx : UDINT;
END_VAR

// Komut gönder
aTxBuf[0] := 16#01; // cihaz adresi
aTxBuf[1] := 16#52; // 'R' = read weight gibi varsayalım

SerialDriverInstance().Send(ADR(aTxBuf), 2);

// Cevap al
nRx := SerialDriverInstance().Receive(ADR(aRxBuf), SIZEOF(aRxBuf));

IF nRx > 0 THEN
    // aRxBuf'tan ağırlık parse edilir
END_IF
```

Bu yapıdaki güzellik:

- FB_Scale **driver yazmıyor**
- Sadece SerialDriverInstance() kullanıyor
- Başka bir cihaz (ör. barkod okuyucu) da aynı hat üzerindeyse  
  yine aynı driver kullanılır  
  (adres/protokol ayrımı üst seviyede yapılır)

---

## 5. Bu tasarım başka donanımlara nasıl genellenir?

### 1) Modbus RTU / TCP
```
FB_ModbusMasterDriver
g_ModbusMasterDriver
ModbusMasterInstance()
```

- Tek master, tüm register okuma/yazma buradan

### 2) MQTT Driver
```
FB_MqttClientDriver
g_MqttClient
MqttClientInstance()
```

- Tek broker bağlantısı  
- Tüm publish/subscribe işlemleri buradan

### 3) Robot Controller Driver
```
FB_RobotDriver
RobotDriverInstance()
```

- Robot program yükleme, start/stop, pozisyon sorgulama

### 4) CANopen Master Driver
```
FB_CanDriver
CanDriverInstance()
```

- Tüm CAN node erişimi tek master FB’den

---

## 6. Özet – Hardware Driver Singleton

Hardware driver Singleton olursa:

### ✔ Donanım kaynağı tek kişi tarafından kontrol edilir  
### ✔ Port/bağlantı çakışmaları mimari olarak imkânsız hale gelir  
### ✔ Timeout / frame collision hataları ortadan kalkar  
### ✔ Debug, logging ve retry yönetimi tek yerden yapılır  
### ✔ Üst seviye FB'ler donanım bağımlılığından kurtulur  
### ✔ Uygulama çok daha kararlı ve deterministik çalışır  

TwinCAT’te uygulama kalıbı:

```iecst
FB_Driver
g_Driver : FB_Driver;
DriverInstance();
```

---

# PLC / TwinCAT — **Machine State (State Machine) Singleton Tasarımı**
**Kayıpsız içerik – Markdown dokümanı**

---

## 1. PLC’de State Machine ne yapar?

State machine, makinenin üst seviye çalışma modlarını yönetir:

- **OFF / PowerUp**
- **INIT**
- **IDLE**
- **START / RUN**
- **PAUSE**
- **STOP**
- **ERROR / ESTOP / RECOVERY**

Bu state machine:

- Start / Stop / Reset butonlarını yorumlar
- Safety durumlarını kontrol eder
- Alt proses FB’lerinin *ne zaman çalışacağını / duracağını* belirler
- HMI’de görünen “Makine Durumu”nun tek kaynağıdır

Kısaca:

> **Makinenin ana kontrol akışı (beyni)** state machine’dir.

---

## 2. Neden Singleton olmalı?

Çünkü:

- Bir makinenin **aynı anda iki ana durumu** olamaz  
  Örnek: Hem RUN hem ERROR aynı anda mantıksızdır.
- Birden fazla modülde *ayrı state machine’ler* olursa:
  - A FB’si “IDLE” derken B FB’si “RUN” zannedebilir
  - HMI tutarsız durum gösterir
  - Start/Stop/Reset davranışları bozulur

Doğru çözüm:

> **Makinenin resmi çalışma durumu tek bir merkezden yönetilmelidir.**

Bu, Singleton yaklaşımı ile bire bir aynıdır.

---

## 3. Mimari – TwinCAT’te Machine State Singleton Yapısı

Temel yapı:

- `E_MachineState` → Makine durum enum’u  
- `FB_MachineState` → Durum geçişlerini yöneten FB  
- `g_MachineState` → GVL’de tek instance  
- `MachineStateInstance()` → C#’taki `MachineState.Instance` karşılığı  
- Alt modüller → Çalışma/durma kararlarını bu state’e göre verir  

---

## 4. Makine Durum Enum’u

```iecst
TYPE E_MachineState :
(
    MS_OFF := 0,
    MS_INIT,
    MS_IDLE,
    MS_STARTING,
    MS_RUNNING,
    MS_STOPPING,
    MS_ERROR
);
END_TYPE
```

---

## 5. State Machine FB — `FB_MachineState`

```iecst
FUNCTION_BLOCK FB_MachineState
VAR_INPUT
    bCmdStart   : BOOL;    // HMI Start butonu
    bCmdStop    : BOOL;    // HMI Stop butonu
    bCmdReset   : BOOL;    // HMI Reset
    bSafetyOk   : BOOL;    // Kapak, estop, safety hattı
END_VAR
VAR_OUTPUT
    eState        : E_MachineState;
    bAllowMotion  : BOOL;     // Motion FB’lere izin
    bAllowProcess : BOOL;     // Proses FB’lere izin
    bErrorLatched : BOOL;     // Hata latched
END_VAR
VAR
    eNextState : E_MachineState;
    tStateTimer : TON;
END_VAR
```

---

## 5.1. Cyclic metodunda state machine mantığı

```iecst
METHOD PUBLIC Cyclic
CASE eState OF

    MS_OFF:
        eNextState := MS_INIT;

    MS_INIT:
        IF bSafetyOk THEN
            eNextState := MS_IDLE;
        ELSE
            eNextState := MS_ERROR;
        END_IF

    MS_IDLE:
        bAllowMotion  := FALSE;
        bAllowProcess := FALSE;

        IF NOT bSafetyOk THEN
            eNextState := MS_ERROR;
        ELSIF bCmdStart THEN
            eNextState := MS_STARTING;
        END_IF

    MS_STARTING:
        tStateTimer(IN := TRUE, PT := T#2S);
        IF NOT bSafetyOk THEN
            eNextState := MS_ERROR;
        ELSIF tStateTimer.Q THEN
            eNextState := MS_RUNNING;
        END_IF

    MS_RUNNING:
        bAllowMotion  := TRUE;
        bAllowProcess := TRUE;

        IF NOT bSafetyOk THEN
            eNextState := MS_ERROR;
        ELSIF bCmdStop THEN
            eNextState := MS_STOPPING;
        END_IF

    MS_STOPPING:
        bAllowProcess := FALSE;
        tStateTimer(IN := TRUE, PT := T#1S);
        IF tStateTimer.Q THEN
            eNextState := MS_IDLE;
        END_IF

    MS_ERROR:
        bAllowMotion  := FALSE;
        bAllowProcess := FALSE;
        bErrorLatched := TRUE;

        IF bCmdReset AND bSafetyOk THEN
            bErrorLatched := FALSE;
            eNextState := MS_IDLE;
        END_IF

END_CASE;

eState := eNextState;
```

Bu state machine:

- Makine davranışını *tamamen belirleyen resmî kaynak* olur.
- Tüm modüller bu state’e göre hareket eder.

---

## 6. Global Singleton Instance

```iecst
// GVL_State.TcGVL
VAR_GLOBAL
    g_MachineState : FB_MachineState;
END_VAR
```

---

## 7. Accessor Function (Singleton erişimi)

```iecst
FUNCTION MachineStateInstance : REFERENCE TO FB_MachineState
MachineStateInstance REF= g_MachineState;
```

Artık her yerden:

```iecst
MachineStateInstance().eState
MachineStateInstance().bAllowMotion
MachineStateInstance().bAllowProcess
```

ile tek bir makine state’i okunur.

---

## 8. Diğer FB’ler state machine’i nasıl kullanır?

---

### 8.1. Motion / Axis FB

```iecst
FUNCTION_BLOCK FB_AxisHandler
VAR
    bMotionEnabled : BOOL;
END_VAR

METHOD PUBLIC Cyclic
bMotionEnabled := MachineStateInstance().bAllowMotion;

IF bMotionEnabled THEN
    // Motion çalışabilir
ELSE
    // Motion durmalı / hold edilmeli
END_IF
END_METHOD
```

---

### 8.2. Process (Dolum / Paketleme / Tartım)

```iecst
FUNCTION_BLOCK FB_Process
VAR
    bProcessEnabled : BOOL;
END_VAR

METHOD PUBLIC Cyclic
bProcessEnabled := MachineStateInstance().bAllowProcess;

IF bProcessEnabled THEN
    // Normal proses
ELSE
    // Bekleme / durdurma
END_IF
END_METHOD
```

---

## 8.3. HMI – Makine durumunu göstermek

```iecst
PROGRAM PLC_HmiState
VAR
    eStateHmi        : E_MachineState;
    bErrorLatchedHmi : BOOL;
END_VAR

eStateHmi        := MachineStateInstance().eState;
bErrorLatchedHmi := MachineStateInstance().bErrorLatched;
```

HMI renk/ikon değişimlerini buna göre yönetir.

---

## 9. Özet — State Machine + Singleton

Makinenin global durumu:

- **Tek bir FB ile yönetilmelidir → FB_MachineState**
- **Bu FB’nin tek örneği olmalıdır → g_MachineState**
- **Herkes aynı kaynaktan state okumalıdır → MachineStateInstance()**

Bunun avantajları:

✔ Tüm modüller tutarlı çalışır  
✔ HMI tek bir doğru kaynağa bakar  
✔ Start/Stop/Reset davranışı deterministik olur  
✔ Safety ve hata mantığı merkezi yönetilir  
✔ Debug ve bakım çok daha kolaylaşır  

---


# Teknik Terimler Sözlüğü  
*(Deterministik — Heuristik — Prognostik ve ilgili kavramlar)*  

---

Bu sözlük, otomasyon, PLC, kontrol teorisi, yapay zekâ ve mühendislik disiplinlerinde sık geçen soyut ve teknik terimlerin **kısa, anlaşılır ve teknik olarak doğru** açıklamalarını içerir.  
Dipnotlar bölüm sonunda verilmiştir.

---

## 1. Deterministik *(Deterministic)* [^1]
Aynı giriş → her zaman aynı çıkış.  
Tesadüf veya belirsizlik yoktur.

---

## 2. Heuristik *(Heuristic)* [^2]
Kesin çözüm üretmeyen; pratik, hızlı ve yaklaşık yöntem.  
Deneyime dayalı kurallar.

---

## 3. Prognostik *(Prognostic)* [^3]
Bir arızanın **ne zaman** oluşacağını veya sistem ömrünün **ne kadar kaldığını** tahmin eder.

---

## 4. Stokastik *(Stochastic)* [^4]
Davranışı rastlantısal değişkenlere bağlı olan süreç.

---

## 5. Probabilistik *(Probabilistic)* [^5]
Çıktılar olasılık dağılımlarına göre tanımlanır; kesinlik yoktur.

---

## 6. Preskriptif *(Prescriptive)* [^6]
Sistemin ne yapması gerektiğini söyleyen karar modelleri; optimizasyon tabanlı öneriler.

---

## 7. Diagnostik *(Diagnostic)* [^7]
Mevcut hatanın nedenini belirleme; alarm analizi.

---

## 8. Semantik *(Semantic)* [^8]
Bir verinin, sembolün veya tag’in **anlam** içeriği.

---

## 9. Sibernetik *(Cybernetics)* [^9]
Geri besleme (feedback) ve kontrol sistemlerinin bilimi.

---

## 10. Deterministik Zamanlama *(Real-Time Determinism)* [^10]
Sistemin bir işlemi tam belirtilen sürede tamamlamayı garanti etmesi.

---

## 11. Optimistik Algoritma *(Optimistic Algorithm)* [^11]
Çakışmaların nadir olduğunu varsayar; hızlıdır ancak risk alır.

---

## 12. Pessimistik Yaklaşım *(Pessimistic)* [^12]
Çakışmaların olacağını varsayar; güvenlidir ama daha yavaştır.

---

## 13. Deterministik Sonlu Durum Makinesi *(Deterministic Finite State Machine — DFSM)* [^13]
Her durumda yalnızca **bir** geçerli sonraki adım vardır.

---

## 14. Non‑Deterministik *(Nondeterministic)* [^14]
Bir durumda birden fazla geçerli olasılık bulunabilir.  
(Teorik modellerde yaygındır.)

---

## 15. Telemetri *(Telemetry)* [^15]
Uzaktaki cihazlardan veri toplama.

---

## 16. Telemetrik Analiz *(Telemetry Analytics)* [^16]
Toplanan verilerin anlamlandırılması ve yorumlanması.

---

## 17. Anomali Tespiti *(Anomaly Detection)* [^17]
Normal davranıştan sapmaları bulur; erken arıza tespiti için kullanılır.

---

## 18. Sensör Füzyonu *(Sensor Fusion)* [^18]
Birden fazla sensörden gelen veriyi birleştirerek daha yüksek doğruluk üretme.

---

## 19. Regresyon *(Regression)* [^19]
Bir değişkenin diğer değişkenlere göre matematiksel tahmini.

---

## 20. Kestirim *(Estimation)* [^20]
Bir büyüklüğün gerçek değeri bilinmiyorsa, ölçüm ve matematiksel modellerle **yaklaşık** hesaplanması.

---

# 📎 Dipnotlar

[^1]: Deterministik sistemler, PLC ve gerçek zamanlı kontrol uygulamalarında temel gerekliliktir.  
[^2]: Heuristik yöntemler, optimum değil *yeterince iyi* sonuç verecek karar kurallarıdır.  
[^3]: Prognostik analiz, titreşim/ısı/ses verilerinden arıza zamanını tahmin eder.  
[^4]: Stokastik süreçler rastgelelik içerir; sensör gürültüsü bunun doğal bir örneğidir.  
[^5]: Probabilistik modeller, risk ve belirsizlik hesaplamalarında kullanılır.  
[^6]: Preskriptif modeller optimizasyona dayanır ve sistem için en iyi eylemi önerir.  
[^7]: Diagnostik yaklaşım mevcut arızayı kök sebep analiziyle belirler.  
[^8]: Semantik veri, sadece sayı değil **anlam** taşır (ör. bEmergencyStop = “acil stop aktif”).  
[^9]: Sibernetik, kontrol teorisinin temel kavramlarını tanımlar (feedback, correction vb.).  
[^10]: EtherCAT ve güvenlik PLC’leri deterministik zamanlama gerektirir.  
[^11]: Optimistik yöntemler hızlıdır; çakışma olursa yeniden denenir.  
[^12]: Pessimistik yaklaşım çakışmayı baştan engeller (lock mekanizmaları gibi).  
[^13]: PLC’de kullanılan state machine’ler genelde deterministiktir.  
[^14]: Non‑deterministik modeller daha çok bilgisayar bilimi teorisinde görülür.  
[^15]: SCADA/IIoT sistemleri telemetri ile veri toplar.  
[^16]: Analiz aşaması, telemetri verisini anlamlı bilgiye dönüştürür.  
[^17]: Anomali tespiti, erken arıza ve güvenlik ihlallerinde kritik rol oynar.  
[^18]: Örnek: IMU + GPS birleşimi ile daha stabil konum hesaplama.  
[^19]: Regresyon, tahmin modellerinin matematiksel temelidir.  
[^20]: Kestirim, filtreleme teknikleri (Kalman vb.) ile daha doğru ölçüm üretir.

---


# Singleton & Teknik Terimler — Soru Bankası  
**Kolaydan Zora, Kitap Formatında Sorular + Dipnot Cevapları**

---

Bu dosya, yukarıdaki Singleton, PLC mimarisi ve teknik terimler bölümlerine dayalı olarak hazırlanmış **kolay → orta → zor** seviyeli bir soru bankasıdır.  
Cevaplar **dipnotlar** bölümünde verilmiştir (kitap formatı).

---

# 📘 1. Kolay Seviye Sorular

### **Soru 1:**  
Singleton deseninin temel amacı nedir?

### **Soru 2:**  
PLC’de neden bir seri port sürücüsü (RS485) Singleton yapılmalıdır?

### **Soru 3:**  
Recipe Manager’ın tekil (Singleton) olması neden önemlidir?

### **Soru 4:**  
Deterministik sistem ne demektir?

### **Soru 5:**  
Heuristik yöntemler kesin çözüm sağlar mı?

---

# 📗 2. Orta Seviye Sorular

### **Soru 6:**  
Alarm Manager neden birden fazla instance’a sahip olmamalıdır?

### **Soru 7:**  
PLC mimarisinde global bir config manager olmasaydı hangi problemler ortaya çıkardı?

### **Soru 8:**  
Machine State (FSM) Singleton olmazsa makinede hangi tür hatalar gözlenebilir?

### **Soru 9:**  
Stokastik bir sistem ile deterministik bir sistem arasındaki fark nedir?

### **Soru 10:**  
Preskriptif bir model ile prognostik bir model arasındaki farkı açıklayın.

---

# 📙 3. Zor Seviye Sorular

### **Soru 11:**  
PLC’de hem Alarm Manager hem Recipe Manager hem de State Machine Singleton değilse sistemde hangi tür *senkronizasyon bozuklukları* oluşabilir? En az üç örnek verin.

### **Soru 12:**  
Singleton Driver (ör. RS485) yerine iki farklı FB ile aynı porta erişilmeye çalışıldığında protokol seviyesinde görülebilecek hataları açıklayın.

### **Soru 13:**  
Deterministik olmayan (non‑deterministic) yapılar PLC’de neden tercih edilmez? Örnek bir senaryo ile açıklayın.

### **Soru 14:**  
Bir makinede hem prognostik hem de diagnostik analizlerin tutulduğu sistemde Singleton yaklaşımı kullanılmazsa veri bütünlüğü nasıl bozulabilir?

### **Soru 15:**  
Aynı makinede State Machine, Alarm Manager ve Hardware Driver Singleton iken Recipe Manager’ın Singleton olmaması neden tehlikeli bir mimari açığıdır? Teknik gerekçe ile açıklayın.

---

# 📎 Dipnot Cevapları (Çözümler)

[^1]: **Singleton’ın amacı**, bir sınıfın/FB’nin tek örneğini oluşturmak ve tüm sistemin bu örneğe erişmesini sağlamaktır.  
[^2]: RS485 donanımı tek fiziksel porttur; iki FB aynı anda portu açmaya çalışırsa çakışma olur.  
[^3]: Tüm makine aynı ürün parametrelerini kullanmalıdır; birden fazla recipe kaynağı tutarsızlığa yol açar.  
[^4]: Deterministik sistemlerde aynı giriş her zaman aynı sonucu üretir.  
[^5]: Hayır. Heuristik yöntemler pratik ve hızlıdır fakat kesin doğruyu garanti etmez.  

[^6]: Birden fazla Alarm Manager olursa HMI hangi listeye bakacağını bilemez; alarmlar tutarsız olur.  
[^7]: Config değerleri farklı modüllerde farklı görünür; davranış tutarsızlaşır; bakım zorlaşır.  
[^8]: Bir modül RUN derken diğeri STOP veya ERROR olabilir; makine davranışı kaotik hâle gelir.  
[^9]: Deterministik → sonuç kesin; Stokastik → sonuç olasılıksal ve rastlantısaldır.  
[^10]: Prognostik → “Ne zaman arıza olur?”  
Preskriptif → “Ne yapılmalı?” veya “En iyi aksiyon nedir?”

[^11]:  
- Alarmlar farklı listelerde oluşur → tutarsız görünür.  
- Recipe farklı FB’lerde farklı görünür → proses bozulur.  
- State Machine farklı FB’lerde farklı durur → kontrol deterministik olmaz.  

[^12]:  
- Frame collision  
- CRC mismatch  
- Donanım cevap vermez  
- Zamanlama bozulur (timeout)  

[^13]: PLC gerçek zamanlı sistemdir; belirsiz geçişler (non‑deterministic) kontrol edilemez durumlara yol açar.  

[^14]: Prognostik → arıza zamanı tahmini  
Diagnostik → mevcut arızanın nedeni  
Singleton olmadığında veriler farklı kaynaklarda farklı tutulur → veri uyumsuzluğu doğar.  

[^15]: Recipe farklı FB’lerde farklı görünebilir → proses modülleri farklı parametrelerle çalışır → üretim tutarsızlığı ve hata oluşur.

---

**Hazırlayan:**  
Otomasyon & PLC Mimarisi — *Singleton Pattern Derin Teknik Soru Seti*




## 📘 Ek Kodlama Soruları (Kolay → Zor)

### 1. Singleton erişim hatası
```iecst
FUNCTION_BLOCK FB_Logger
VAR
    sLast : STRING(80);
END_VAR

METHOD PUBLIC Log
VAR_INPUT s : STRING; END_VAR
sLast := s;
END_METHOD

FUNCTION LoggerInstance : FB_Logger
VAR_GLOBAL
    g_Logger : FB_Logger;
END_VAR

LoggerInstance := g_Logger;
```
**Soru:** Singleton neden doğru çalışmaz ve nasıl düzeltilir?

### 2. State machine geçişi
```iecst
CASE eState OF
    ST_IDLE:
        IF bStart THEN
            eState := ST_RUN;
        END_IF

    ST_RUN:
        IF bStop THEN
            // eksik geçiş
        END_IF
END_CASE
```
**Soru:** ST_RUN → ST_IDLE geçişini yazınız.

### 3. Config Manager senkronizasyonu
**Soru:** Tüm modüllerin güncel değeri görmesi için tek erişim noktasını kodla gösterin.

### 4. Alarm Manager Raise/Clear davranışı
**Soru:** Overload alarmının doğru temizlenmesi için kodu düzeltin.

### 5. RS485 Driver ile çoklu cihaz yönetimi
```iecst
FUNCTION_BLOCK FB_ScaleDevice
VAR_INPUT
    nAddress : BYTE;
END_VAR
VAR
    aTx : ARRAY[0..15] OF BYTE;
    aRx : ARRAY[0..31] OF BYTE;
END_VAR

METHOD PUBLIC ReadWeight
VAR
    nRx : UDINT;
END_VAR

// buraya kod yazılacak
```
**Soru:** Tek driver üzerinden farklı adreslerde çalışan yapıyı tamamlayın.

### 6. Deterministik state machine sırası
**Soru:** Geçişleri deterministik yapmak için doğru öncelik sırasıyla yeniden yazın.










