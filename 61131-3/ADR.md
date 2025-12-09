# 🚀 TwinCAT ADR Operatörü – Tam Kapsamlı Eğitim Dokümanı (Kayıpsız .md)

Bu doküman, **TwinCAT ADR operatörünü**, POINTER mantığıyla birlikte tamamen açıklayan, gerçek dünya örneklerine dayanan, tam profesyonel bir eğitim paketidir.  
REFERENCE ve POINTER eğitim formatı ile birebir uyumludur.

---

# 📘 1. ADR Nedir?

`ADR(<variable>)` operatörü TwinCAT’te bir değişkenin **memory adresini** döndürür.

### ADR ne işe yarar?

- POINTER değişkenlerine adres atamak için kullanılır  
- Verinin hafıza üzerindeki gerçek konumunu verir  
- Struct, array, buffer, I/O block gibi veri alanlarının binary adresini almayı sağlar  
- Düşük seviye veri manipülasyonlarının temelidir  

---

# 📘 2. ADR'nin Döndürdüğü Veri Tipi

ADR, platforma göre farklı tip döndürür:

| Runtime Mimarisi | ADR Tipi |
|------------------|----------|
| 32-bit | DWORD |
| 64-bit | LWORD |
| TwinCAT önerisi | **PVOID** |
| Gelişmiş sistemler | __XWORD |

### ✔ Neden PVOID önerilir?

Çünkü:

- 32/64 bit fark etmeksizin uyumludur  
- TwinCAT tarafından resmi olarak önerilir  
- POINTER ile aynı yapıda çalışır  

---

# 📘 3. ADR Kullanım Söz Dizimi

```pascal
<address> := ADR(<variable>);
```

Geçerli hedef tipleri:

- PVOID  
- DWORD  
- LWORD  
- __XWORD  
- POINTER TO <type>

---

# 📘 4. Basit Örnek – Bir INT Değişkeninin Adresini Alma

```pascal
VAR
    nVar : INT := 10;
    pNumber : POINTER TO INT;
END_VAR

pNumber := ADR(nVar);   // pointer artık nVar'ın adresini tutuyor
```

Artık:

- `pNumber` → adres  
- `pNumber^` → değer  

---

# 📘 5. ADR ile Çoklu Adres Tipi Örneği

```pascal
FUNCTION_BLOCK FB_Address
VAR
    nVar      : INT := 10;
    pNumber   : POINTER TO INT;
    nAddress1 : PVOID;
    nAddress2 : DWORD;
    nAddress3 : LWORD;
    nAddress4 : __XWORD;
END_VAR

pNumber   := ADR(nVar);
nAddress1 := ADR(nVar);
nAddress2 := ADR(nVar);
nAddress3 := ADR(nVar);
nAddress4 := ADR(nVar);
```

✔ PVOID tüm platformlarda geçerli olduğu için **her zaman en iyi seçimdir**.

---

# 📘 6. Online Change – Adres Kayması Problemi

TwinCAT’te bir **online change** yapıldığında:

- Değişkenlerin hafızada bulunduğu adresler değişebilir  
- Eski ADR sonuçları geçersiz hale gelebilir  
- POINTER artık **çöp adrese** işaret edebilir  

Bu nedenle TwinCAT şunu yapar:

### ✔ Pointer adreslerini otomatik günceller (TC3.1 Build 4026+)  
Ama bunun çalışması için:

- Değişken sembol tablosunda olmalıdır  
- `{attribute 'hide'}` kullanılmış olmamalıdır  
- Pointer PLC dışı memory’e işaret etmemelidir  

---

# 📘 7. ADR'nin Gerçek Dünya Kullanım Senaryoları

Aşağıdaki örnekler pointer kullanımında ADR’nin nasıl kritik rol oynadığını gösterir.

---

# ⭐ 1. Modbus / TCP Frame Parsing – ADR ile Buffer Başlangıcı

```pascal
pData := ADR(rxBuffer);

deviceAddress := pData^;
functionCode  := pData[1];
payload       := SHL(pData[3], 8) OR pData[4];
```

ADR → buffer'ın başlangıç adresini verir.

---

# ⭐ 2. Struct → Byte Stream (Serialization)

```pascal
pByte := ADR(Packet);

Send(pByte[0], SIZEOF(Packet));
```

ADR burada **binary veri gönderimi** için zorunludur.

---

# ⭐ 3. Memory-Mapped IO Erişimi

```pascal
pMem := ADR(EcatBuffer);

// donanım register okuma
status  := pMem[0];
tempRaw := pMem[10] + SHL(pMem[11], 8);
```

---

# ⭐ 4. Ring Buffer / FIFO Yönetimi

```pascal
pWrite := ADR(buffer);
pWrite[index] := newData;
```

ADR → buffer’ın adresini verir → pointer ile hızlı veri yazma sağlanır.

---

# ⭐ 5. Büyük Array Üzerinde Performans Optimizasyonu

```pascal
pArr := ADR(BigArray);

FOR i := 0 TO 9999 DO
    pArr[i] := pArr[i] + 1;
END_FOR
```

ADR + POINTER → yüksek hız.

---

# ⭐ 6. Dinamik Veri Kaynakları için Universal Stream Processor

```pascal
fbProcessor(pStream := ADR(ModbusBuffer), nLength := modbusLen);
fbProcessor(pStream := ADR(UdpRxBuffer),   nLength := udpLen);
fbProcessor(pStream := ADR(FileData),      nLength := fileLen);
```

---

# 📘 8. ADR – POINTER – REFERENCE: Farkların Tam Özeti

| Özellik | ADR | POINTER | REFERENCE |
|--------|-----|----------|-----------|
| Ne yapar? | adres verir | adresi işler | adresi saklar |
| Dereference | yok | ^ gerekli | yok |
| Risk | düşük | yüksek | düşük |
| Protokol parsing | ✔ | ✔ | ❌ |
| Memory I/O | ✔ | ✔ | ❌ |
| Kod kolaylığı | orta | zor | çok kolay |

---

# 📘 9. ADR’nin Mühendislikteki Prensibi

> **ADR → Hafıza adresini verir.**  
> **POINTER → Bu adresteki veriyi işler.**  
> **REFERENCE → Bu mekanizmayı güvenli ve kolay hale getirir.**

ADR, TwinCAT’te pointer tabanlı tüm sistemlerin giriş kapısıdır.

---

# 📘 10. Kısacası ADR’yi böyle hatırla:

### **ADR = PLC hafızasındaki her şeyin koordinat sistemindeki lokasyonudur.**

---



