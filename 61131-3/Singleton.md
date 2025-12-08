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




