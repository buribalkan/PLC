# 🧠 SEVİYE 3 ULTRA PROFESYONEL MASTERCLASS  
# **ROR — ROTATE RIGHT OPERATÖRÜ DERİN TEKNİK EĞİTİMİ**

---

# 📌 İçindekiler  
1. ROR Nedir?  
2. Rotate vs Shift — Temel Farklar  
3. Veri Tipi Genişliğinin Etkisi  
4. TwinCAT ROR Algoritması  
5. Constant Input → Minimum Data Type Kuralı  
6. Modular Rotation Mekanizması  
7. Derin Örnek Analizi  
8. BYTE / WORD / DWORD / LWORD Rotasyon Yapısı  
9. Endüstriyel Kullanım Alanları  
10. Sık Yapılan Hatalar  
11. Örnek Kod  
12. Sonuç  

---

# 1. ROR Nedir?

`ROR(in, n)` operatörü, operandın bitlerini **sağa döndürür (rotate right)**  
ve sağdan çıkan bit sola geri yazılır.  
Bu işlem **cyclic** bir bit hareketidir ve **veri kaybı olmaz**.

---

# 2. Rotate vs Shift — Temel Farklar

| Operatör | Davranış |
|---------|----------|
| **SHR** | Bitleri sağa kaydırır, düşen bitler kaybolur, MSB → 0 |
| **ROR** | Bitleri sağa döndürür, düşen bit MSB’ye geri döner |

---

# 3. Veri Tipi Genişliğinin Etkisi

TwinCAT rotasyonu **input veri tipinin genişliğiyle** tanımlar:

| Tip | Bit Genişliği |
|------|--------------|
| BYTE | 8 bit |
| WORD | 16 bit |
| DWORD | 32 bit |
| LWORD | 64 bit |

---

# 4. TwinCAT ROR Algoritması

Her iterasyonda:

1. LSB alınır  
2. Operand `SHR` edilir  
3. LSB MSB pozisyonuna yerleştirilir  

---

# 5. Constant Input → Minimum Data Type Kuralı

TwinCAT bir literal gördüğünde:

```
ROR(16#45, 2)
```

→ 16#45 değerini **BYTE** olarak ele alır.

---

# 6. Modular Rotation Mekanizması

```
effectiveShift = n MOD width
```

Örnek: WORD için width=16  
`ROR(x, 18) = ROR(x, 2)`

---

# 7. Derin Örnek Analizi

```
nInByte = 0x45   → 0100 0101
nInWord = 0x0045 → 0000 0000 0100 0101
nVar = 2
```

### BYTE ROR:
```
0100 0101 → ROR1 → 1010 0010
1010 0010 → ROR2 → 0101 0001 = 0x51
```

### WORD ROR:
```
0000 0000 0100 0101 → ROR2 → 0100 0000 0001 0001 = 0x4011
```

---

# 8. BYTE / WORD / DWORD / LWORD Döngüsel Yapısı

| Tip | Döngüsel Bit Yapısı |
|------|---------------------|
| BYTE | 0–7 |
| WORD | 0–15 |
| DWORD | 0–31 |
| LWORD | 0–63 |

---

# 9. Endüstriyel Kullanım Alanları

✔ CRC algoritmaları  
✔ Checksum / hashing fonksiyonları  
✔ Frame encoding/decoding  
✔ Haberleşme protokol bit çözümleme  
✔ PRNG yapıları  
✔ DSP bit manipülasyonu  

---

# 10. Sık Yapılan Hatalar

❌ ROR ile SHR’i karıştırmak  
❌ Constant input → BYTE davranışını unutmak  
❌ Veri tipi genişliğini hesaplamamak  
❌ Çok büyük shift değerlerinde modulo çalıştığını unutmamak  

---

# 11. Örnek Kod

```st
PROGRAM Ror_st 
VAR 
    nInByte  : BYTE := 16#45; 
    nInWord  : WORD := 16#45; 
    nResByte : BYTE; 
    nResWord : WORD;
    nVar     : BYTE := 2; 
END_VAR

nResByte := ROR(nInByte, nVar);   (* Result: 0x51  *)
nResWord := ROR(nInWord, nVar);   (* Result: 0x4011 *)
```

---

# 12. Sonuç

- ROR sağa döngüsel bit rotasyonudur  
- Veri kaybı olmaz  
- Veri tipinin bit genişliği sonucu belirler  
- Constant input → minimum type uygulanır  
- Endüstriyel bit manipülasyonlarında kritik kullanıma sahiptir  

---

