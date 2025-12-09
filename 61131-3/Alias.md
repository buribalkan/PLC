# TwinCAT ALIAS TYPES MASTERCLASS  
_Tam Kapsamlı Profesyonel Eğitim Dokümanı (Markdown Formatında)_

---

## 📘 1. Alias Type Nedir?

TwinCAT’te **Alias**, mevcut bir veri tipine yeni bir isim vererek:

- Kod okunabilirliğini artırmak  
- Veri yapıları için standartlar oluşturmak  
- Belirli boyut, aralık veya başlangıç değeri zorlamak  
- Function block türleri için alternatif ad üretmek  

amacıyla kullanılan **kullanıcı tanımlı veri tipidir**.

Alias, bir **DUT (Data Unit Type)** dosyasında tanımlanır.

---

## 🧱 2. Alias Type Sözdizimi

```pascal
TYPE <AliasName> : <ExistingType>;
END_TYPE
```

✔ Basit  
✔ Temiz  
✔ Tamamen derleyici destekli

---

## 🧩 3. Alias Hangi Tipler Üzerinde Kullanılabilir?

Aşağıdaki türler alias olarak kullanılabilir:

### ✔ Basic Types  
BOOL, INT, REAL, BYTE, WORD, STRING, vb.

### ✔ Data Types  
STRUCT, ENUM, ARRAY, vb.

### ✔ Function Block Types  
Bir FB’nin alias olarak yeniden adlandırılması mümkündür.

---

## 📝 4. En Basit Alias Örneği

```pascal
TYPE T_Message : STRING[50];
END_TYPE

PROGRAM MAIN
VAR
   sMessage : T_Message;
END_VAR

sMessage := 'This is a message';
```

Bu tanımla artık:

- Tüm mesajlar **en fazla 50 karakterdir**
- Bu sınır alias tarafından **zorlanmış olur**

---

# 🧰 5. Alias – Gerçek Kullanım Senaryoları

---

## 📌 5.1 Belirli Boyutta Veri Paketleri İçin

Örneğin bir network frame 1500 byte’tır:

```pascal
TYPE FRAME : ARRAY[0..1499] OF BYTE;
END_TYPE
```

Kullanım:

```pascal
aFrame : FRAME;
```

---

## 📌 5.2 Belirli Boyutta String İçin

```pascal
TYPE SYMBOL : STRING(512);
END_TYPE

sSymbol : SYMBOL;
```

Her SYMBOL değerinin maksimum 512 karakter olacağı garanti edilir.

---

# 🎯 6. Alias ile Varsayılan Başlangıç Değeri Verme

Alias en büyük avantajlarından biri:  
**Derleyicinin default değerinin yerine kendi default’unu tanımlayabilmektir.**

### Örnek — index tipi varsayılan -1 olsun

```pascal
TYPE INDEX : DINT := -1;
END_TYPE
```

Kullanım:

```pascal
VAR nIdx : INDEX; END_VAR
// nIdx = -1
```

---

# 🚦 7. Alias + Subrange (Alt Aralık) Kullanımı

Alias, tip üzerinde **zorunlu aralık** belirlemek için kullanılabilir.

### Örnek — Unicode kod noktaları için RUNE tipi

```pascal
{attribute 'qualified_only'}
VAR_GLOBAL CONSTANT
    cMaxRune : DINT := DINT#16#0010FFFF;
END_VAR

TYPE RUNE : DINT(0..GVL.cMaxRune);
END_TYPE
```

Bunun anlamı:

- RUNE değişkeni 0 ile 0x0010FFFF arasında olmalıdır  
- Bu aralık dışında atamalar derleyici hatası oluşturur  

Kullanım:

```pascal
VAR symbol : RUNE; END_VAR
symbol := 1200;       // ✔ geçerli
symbol := -5;         // ❌ derleyici hatası
symbol := 999999999;  // ❌ aralık dışı
```

---

# 🧬 8. Alias Type — FUNCTION BLOCK İçin Kullanımı

Bazen bir FB’ye daha anlamlı bir ad vererek yeniden kullanmak istersiniz:

```pascal
TYPE Mixer : FB_MachineMotor;
END_TYPE

PROGRAM MAIN
VAR mx : Mixer; END_VAR
```

---

# 🏗 9. Alias Kullanmanın Faydaları

| Fayda | Açıklama |
|-------|----------|
| ✔ Okunabilirliği artırır | Karmaşık tiplere anlamlı ad verilir |
| ✔ Güvenli veri modelleri | Subrange kontrolü veri hatalarını azaltır |
| ✔ Standardizasyon | Büyük projelerde veri tipleri iletişim standardı hâline getirilebilir |
| ✔ Default initialization eklenebilir | Özellikle index, parametre, ID tiplerinde kullanışlı |
| ✔ FB'lere alternatif isim verebilme | API tasarımında esneklik sağlar |

---

# ⚠ 10. Alias Kullanırken Dikkat Edilecekler

- Alias sadece **türü yeniden isimlendirir**, yeni bir davranış eklemez  
- STRUCT yerine alias kullanmak bütün veri modelini etkiler  
- Subrange kullanımı → derleyici kontrolü arttırır  
- STRING aliasları, bellek boyutunu belirler  

---

# 🧠 11. Gerçek Dünya Alias Kullanım Pattern’ları

---

## 📌 11.1 Network frame buffer

```pascal
TYPE T_Frame : ARRAY[0..1499] OF BYTE;
END_TYPE
```

---

## 📌 11.2 Recipe / Parameter ID

```pascal
TYPE RECIPE_ID : DINT := -1;
END_TYPE
```

---

## 📌 11.3 Machine State Code (numeric)

```pascal
TYPE STATE_CODE : UINT(0..255);
END_TYPE
```

---

## 📌 11.4 Analog input scaling değeri

```pascal
TYPE SCALER : LREAL := 1.0;
END_TYPE
```

---

# 🧪 12. Alias Kullanımı – FULL ÖRNEK

```pascal
TYPE INDEX : DINT := -1; END_TYPE
TYPE SYMBOL : STRING(128); END_TYPE
TYPE FRAME  : ARRAY[0..1023] OF BYTE; END_TYPE

PROGRAM MAIN
VAR
    nSelected : INDEX;     // default = -1
    sName     : SYMBOL;    // max 128 chars
    aBuffer   : FRAME;     // 1024-byte data
END_VAR

nSelected := 5;
sName := 'TestSymbol';
aBuffer[0] := 16#AA;
```

---

# 🏆 13. Alias Masterclass — Özeti

| Konu | Kapsandı |
|------|----------|
| Alias Type nedir? | ✔ |
| Syntax ve permitted types | ✔ |
| Base types, arrays, strings | ✔ |
| Varsayılan başlangıç değerleri | ✔ |
| Subrange alias | ✔ |
| Function block aliasing | ✔ |
| Network, recipe, ID örnekleri | ✔ |

Bu doküman, TwinCAT projelerinde Alias kullanımını **tam profesyonel seviyede** öğretmek için tasarlanmıştır.

---



