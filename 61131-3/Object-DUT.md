# Object DUT -- TwinCAT PLC Data Unit Types

## 1. Object DUT Nedir?

Bir **DUT (Data Unit Type)**, kullanıcıya özel veri tiplerini tanımlamak
için kullanılır. TwinCAT PLC projelerinde yapı (structure),
numaralandırma (enumeration), birlik (union) ve alias gibi yeni veri
türleri oluşturmayı sağlar.

TwinCAT'te iki tür Enumeration DUT bulunur:

-   **Object DUT 1:** Metin listesi desteği *olmayan* DUT\
-   **Object DUT 2:** Metin listesi desteği *olan* Enumeration DUT

------------------------------------------------------------------------

## 2. Object DUT Oluşturma

1.  PLC project tree'de **Solution Explorer** içinde bir klasör seçin.\
2.  Sağ tıklayın → **Add \> DUT...**\
3.  Açılan **Add DUT** diyalogunda bir **isim** ve **veri tipi** seçin.\
4.  **Open** butonuna basın.\
5.  DUT artık PLC project tree'de görünür ve editörde açılır.

------------------------------------------------------------------------

## 3. Add DUT Diyaloğu Açıklamaları

### Name

Oluşturulacak yeni DUT'un adı.

### Data type

#### Structure

Birden fazla farklı tipte değişkeni tek bir mantıksal yapı altında
toplar.\
Yapı içindeki değişkenlere *component* denir.

**Advanced:**\
Yeni yapı, mevcut bir yapıyı genişletebilir (**extends**).\
Bu durumda eski yapının tüm değişkenleri yeni yapıda otomatik olarak
bulunur.

------------------------------------------------------------------------

#### Enumeration

Sabit tamsayı değerlerinin mantıksal olarak bir araya getirildiği veri
tipidir.\
İçindeki değerlere *enumeration values* denir.

**Text List Support:**\
Bu özellik, enumeration elemanlarının dil bağımsızlaştırılmasına olanak
tanır.\
Bir görselleştirmede sayısal değer yerine o dildeki karşılığı görünür.

Bir visualization elementine text list destekli bir enumeration
verilirse:

    MAIN.eVar <E_myEnum>

**Text list ekleme veya kaldırma:**\
Context menu →\
- *Add text list support*\
- *Remove text list support*

------------------------------------------------------------------------

#### Alias

Bir temel tip, veri tipi veya fonksiyon blok için alternatif isim
oluşturmaya yarar.

------------------------------------------------------------------------

#### Union

Farklı veri türlerini aynı hafıza alanında birleştirir.\
Bellek boyutu en büyük bileşene göre belirlenir.

------------------------------------------------------------------------

## 4. DUT Deklare Etme

### Söz Dizimi

    TYPE <identifier> : <data type declaration with optional initialization>
    END_TYPE

------------------------------------------------------------------------

## 5. Örnekler

### 5.1 Structure Tanımı

    TYPE ST_Struct1 :
    STRUCT
        nVar1 : INT;
        bVar2 : BOOL;
    END_STRUCT
    END_TYPE

    TYPE ST_Struct2 EXTENDS ST_Struct1 :
    STRUCT
        nVar3 : DWORD;
        sVar4 : STRING;
    END_STRUCT
    END_TYPE

### 5.2 Enumeration Tanımı

    TYPE E_TrafficSignal :
    (
        eRed,
        eYellow,
        eGreen := 10
    );
    END_TYPE

### 5.3 Alias Tanımı

    TYPE T_Message : STRING[50];
    END_TYPE

### 5.4 Union Tanımı

    TYPE U_Name :
    UNION
        fA : LREAL;
        nB : LINT;
        nC : WORD;
    END_UNION
    END_TYPE


# Object DUT -- TwinCAT PLC Data Unit Types (Genişletilmiş Dokümantasyon)

## 1. Object DUT Nedir?

Bir **DUT (Data Unit Type)**, TwinCAT PLC projelerinde özel veri
tiplerinin tanımlanmasını sağlayan kullanıcı tanımlı veri türüdür. Bu
yapı, projede tekrar eden veri tiplerini standartlaştırmak,
okunabilirliği artırmak ve özellikle büyük ölçekli projelerde veri
yönetimini kolaylaştırmak için kullanılır.

TwinCAT'te Enumeration DUT'lar iki çeşittir:

-   **Object DUT 1:** Metin listesi desteği *olmayan* enumeration\
-   **Object DUT 2:** Metin listesi desteği *olan* enumeration

------------------------------------------------------------------------

## 2. DUT Oluşturma Adımları

1.  **Solution Explorer** içinde bir klasör seçilir.\
2.  Sağ tıklanır → **Add \> DUT...**\
3.  Açılan pencerede bir **isim** ve **veri tipi** seçilir.\
4.  **Open** butonuna tıklanır.\
5.  Oluşturulan DUT proje ağacına eklenir ve editörde açılır.

------------------------------------------------------------------------

## 3. Add DUT Penceresi Açıklamaları

### Name

Yeni oluşturulacak DUT'un adı.

### Data type türleri

------------------------------------------------------------------------

## 3.1 Structure

Birden fazla farklı veri tipini tek bir mantıksal yapı altında toplar.
STRUCT içindeki elemanlara **component** denir.

### Advanced (EXTENDS)

Yeni yapı, mevcut bir yapıyı genişletebilir:

-   Üst yapının tüm değişkenleri otomatik olarak devralınır.
-   Kod tekrarını azaltır.

------------------------------------------------------------------------

## 3.2 Enumeration

Sabit sayısal değerleri mantıksal bir grup altında toplar.

### Text List Support

Localization (çok dilli destek) sağlar:

-   HMI ekranlarında numeric değer yerine metinsel karşılık gösterilir.
-   Örneğin:\
    `MAIN.eVar <E_myEnum>`

### Text list ekleme / kaldırma

Enumeration'a sağ tıklanır →\
- **Add text list support**\
- **Remove text list support**

------------------------------------------------------------------------

## 3.3 Alias

Var olan bir veri tipi için yeni bir isim oluşturur.

Kullanım senaryoları:

-   Projede okunabilirliği artırmak\
-   Karmaşık tipleri sadeleştirmek

------------------------------------------------------------------------

## 3.4 Union

Birden fazla veri tipini **aynı hafıza alanında** tutar.

Özellikler:

-   Bütün elemanların offset'i aynıdır.\
-   Bellek boyutu en büyük bileşenin boyutudur.\
-   Protokol çözme, byte-level veri işleme için idealdir.

------------------------------------------------------------------------

## 4. TwinCAT İçinde DUT Kullanım Senaryoları

-   Büyük projelerde veri tiplerinin standartlaştırılması\
-   Fieldbus cihazları için özel paket tanımları\
-   HMI ile PLC arasındaki veri alışverişinde güçlü tip kontrolü\
-   IoT cihazlarından gelen farklı formatlardaki verilerin yorumlanması\
-   Function Block'lar arasında tutarlı veri paylaşımı

------------------------------------------------------------------------

## 5. DUT Kullanırken Dikkat Edilmesi Gerekenler

-   STRUCT içindeki sıralama hafıza düzenini etkiler.\
-   UNION dikkatli kullanılmalıdır; yanlış yorumlama mümkündür.\
-   STRING boyutları hafıza kullanımını direkt etkiler.\
-   ENUM değerleri otomatik artar; açık manüel değer daha güvenlidir.\
-   Text List destekli ENUM adı değiştirildiğinde HMI güncellenmelidir.

------------------------------------------------------------------------

## 6. Best Practices (TwinCAT Naming Convention Önerileri)

  Veri Tipi        Önerilen Prefix
  ---------------- -----------------
  Structure        `ST_`
  Enumeration      `E_`
  Alias            `T_`
  Union            `U_`
  Değişkenler      lowerCamelCase
  Enum değerleri   `eValueName`

------------------------------------------------------------------------

## 7. DUT Deklarasyon Sözdizimi

    TYPE <identifier> : <data type declaration with optional initialization>
    END_TYPE

------------------------------------------------------------------------

# 8. Örnekler

## 8.1 Structure Örneği ve EXTENDS Kullanımı

    TYPE ST_Struct1 :
    STRUCT
        nVar1 : INT;
        bVar2 : BOOL;
    END_STRUCT
    END_TYPE

    TYPE ST_Struct2 EXTENDS ST_Struct1 :
    STRUCT
        nVar3 : DWORD;
        sVar4 : STRING;
    END_STRUCT
    END_TYPE

------------------------------------------------------------------------

## 8.2 Enumeration Örneği

    TYPE E_TrafficSignal :
    (
        eRed,
        eYellow,
        eGreen := 10
    );
    END_TYPE

------------------------------------------------------------------------

## 8.3 Alias Örneği

    TYPE T_Message : STRING[50];
    END_TYPE

------------------------------------------------------------------------

## 8.4 Union Örneği

    TYPE U_Name :
    UNION
        fA : LREAL;
        nB : LINT;
        nC : WORD;
    END_UNION
    END_TYPE

------------------------------------------------------------------------

# 9. Gelişmiş TwinCAT Örnekleri

## 9.1 IoT Cihaz Durum Yapısı

    TYPE ST_DeviceStatus :
    STRUCT
        bConnected     : BOOL;
        nSignalQuality : USINT;   
        tLastUpdate    : TIME;
        sDeviceName    : STRING(30);
    END_STRUCT
    END_TYPE

------------------------------------------------------------------------

## 9.2 Motor Yapılarında EXTENDS Kullanımı

    TYPE ST_BaseMotor :
    STRUCT
        nRpm      : INT;
        bEnabled  : BOOL;
        rCurrent  : REAL;
    END_STRUCT
    END_TYPE

    TYPE ST_ServoMotor EXTENDS ST_BaseMotor :
    STRUCT
        rPosition : LREAL;
        rVelocity : LREAL;
    END_STRUCT
    END_TYPE

------------------------------------------------------------------------

## 9.3 Text List Destekli ENUM için Çok Dilli Kullanım

  Key        Türkçe      English
  ---------- ----------- ---------
  eIdle      Bekliyor    Idle
  eRunning   Çalışıyor   Running
  eError     Hata        Error

------------------------------------------------------------------------

## 9.4 Byte-Level Veri İşlemede UNION Kullanımı

    TYPE U_DataWord :
    UNION
        wValue : WORD;
        stByte :
        STRUCT
            byLow  : BYTE;
            byHigh : BYTE;
        END_STRUCT
    END_UNION
    END_TYPE

------------------------------------------------------------------------

# 10. Sonuç

Bu doküman, TwinCAT projelerinde DUT kullanımını derinlemesine anlamak
ve projelerde tekrar eden veri yapılarının daha etkili şekilde
yönetilmesi için kapsamlı bir rehber sunmaktadır.


