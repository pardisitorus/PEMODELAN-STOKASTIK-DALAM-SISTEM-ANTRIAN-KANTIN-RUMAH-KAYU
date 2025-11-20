# Analisis Sistem Antrian Kantin Rumah Kayu ITERA  
### Pendekatan Pemodelan Stokastik dan Evaluasi Dampak Cuaca terhadap Laju Kedatangan Pelanggan

Repositori ini berisi analisis model antrian berdasarkan data nyata Kantin Rumah Kayu ITERA. Analisis dilakukan dengan pendekatan **Pemodelan Stokastik (Proses Poisson, M/M/1, dan respons laju kedatangan)** serta didukung oleh analisis tambahan berupa tren sederhana untuk memperjelas interpretasi.

Seluruh analisis menggunakan data yang ditulis manual ke dalam script sehingga **100% anti error** dan tidak bergantung pada file eksternal.

---

# 1. Struktur Dataset  
Dataset berisi:

- **Tanggal**
- **Slot waktu (5 menit)**
- **Jumlah pelanggan sukses membayar**
- **Kondisi cuaca: Hujan / Tidak Hujan**

Data diringkas menjadi **laju kedatangan per hari (λ)** untuk dianalisis menggunakan teori **antrian stokastik**.

---

# 2. Alur Analisis

## Langkah 1 — Membuat dataset manual
```r
library(tidyverse)

data <- tribble(
  ~Tanggal, ~Slot, ~Jumlah, ~Kondisi,
  "11/11/2025","11.50-11.55",6,"Tidak Hujan",
  ... (seluruh data dimasukkan manual)
)
```

## Langkah 2 — Format tanggal & ringkas total per hari
```r
data <- data %>% 
  mutate(Tanggal = as.Date(Tanggal, format = "%m/%d/%Y"))

ts_daily <- data %>%
  group_by(Tanggal, Kondisi) %>%
  summarise(total = sum(Jumlah), .groups = "drop")
```

## Langkah 3 — Estimasi *laju kedatangan λ* harian (Poisson)
Karena slot waktu = 5 menit:

```r
lambda_daily <- ts_daily %>%
  mutate(lambda = total / (12))   # 12 slot per jam (60/5)
```

## Langkah 4 — Moving Average 3-Hari (untuk membantu interpretasi)
```r
library(zoo)
ts_daily <- ts_daily %>%
  arrange(Tanggal) %>%
  mutate(MA_3 = rollmean(total, k=3, fill=NA, align="right"))
```

## Langkah 5 — Perubahan antar hari (volatilitas antrean)
```r
ts_daily <- ts_daily %>%
  mutate(change = c(NA, diff(total)))
```

## Langkah 6 — Autocorrelation (ACF)
```r
acf(ts(ts_daily$total, frequency = 1))
```

## Langkah 7 — Dampak Cuaca terhadap Kedatangan (Elastisitas Stokastik)
```r
D_normal <- mean(ts_daily$total[ts_daily$Kondisi == "Tidak Hujan"])
D_hujan  <- mean(ts_daily$total[ts_daily$Kondisi == "Hujan"])

elasticity <- (D_hujan - D_normal) / D_normal
```

---

# 3. Visualisasi  
Visualisasi eksplorasi:

- Deret waktu total pelanggan (`output_ts1.png`)
- Moving Average 3-Hari (`output_ma.png`)
- Perubahan antar hari (`output_change.png`)
- ACF (`output_acf.png`)
- Perbandingan hujan vs tidak hujan (`output_compare.png`)

---

# 4. Insight Utama Berdasarkan Pemodelan Stokastik

### **1. Laju kedatangan (λ) hari hujan turun drastis**
- λ\_tidak\_hujan ≈ **119 pelanggan/hari**
- λ\_hujan ≈ **83 pelanggan/hari**
- Turun **30.25%**

Dalam konteks **Sistem Antrian M/M/1 atau M/M/s**, penurunan λ sebesar ini:

- Mengurangi panjang antrean
- Mengurangi waktu tunggu rata-rata
- Berdampak pada pemanfaatan server (ρ = λ / μ)

Ini berarti **cuaca adalah variabel stokastik penting** dalam model antrian kampus.

---

### **2. Sistem antrian bersifat *independent day-to-day***
ACF tidak menunjukkan autokorelasi signifikan:

- Tidak ada “pola minggu”
- Tidak ada ketergantungan hari sebelumnya
- Cocok dengan model **kedatangan Poisson** (asumsi utama M/M/1)

Ini mendukung pemilihan model stokastik **Poisson Arrival Process**.

---

### **3. Hari hampir jenuh terjadi pada kondisi tidak hujan**
Dengan estimasi kecepatan layanan (μ) kasir kampus:

- Jika μ ≈ 15 transaksi/jam, λ ≈ 12–18 slot/jam  
- Maka ρ = λ/μ dapat mendekati atau melewati **0.9** pada jam sibuk  
- Ini mendekati kondisi **tidak stabil** pada teori antrian

Artinya sistem **rentan macet**, terutama saat tidak hujan.

---

### **4. Moving Average mengungkap tren penurunan λ**
Tren jangka pendek menurun mendekati tanggal 18–19:

- Hujan mengganggu pola normal
- Stok makanan perlu menyesuaikan kondisi cuaca

---

### **5. Volatilitas harian cukup besar**
Perubahan antar hari menembus:

- +22 (lonjakan)
- –51 (penurunan tajam)

Untuk model stokastik, ini berarti variabilitas tinggi → *lebih cocok M/M/s daripada M/M/1 pada jam puncak*.

---

# 5. Rekomendasi Kebijakan Kampus (Insight Tingkat Rektor)

### **A. Pembangunan Kanopi/Jalur Lindung Menuju Kantin**
Dampak hujan = penurunan 30%.  
Ini adalah **loss ekonomi signifikan** untuk UMKM.

Kebijakan:

- Bangun jalur tertutup menuju kantin  
- Sediakan halte kecil & shading  

Hasil yang diharapkan:

- λ saat hujan meningkat  
- Pendapatan UMKM stabil  
- Mahasiswa tidak menumpuk di gedung lain  
- Mempercepat antrean karena distribusi kedatangan lebih merata

---

### **B. Tambah 1 Kasir Mobil (Mobile Counter) saat Jam Sibuk**
Karena ρ tinggi, sistem mendekati kondisi jenuh.

Kebijakan:

- Sediakan *kasir tambahan temporer*  
- Berlaku hanya 11.30–12.45

Bukti stokastik:
- penambahan server mengubah model dari **M/M/1 → M/M/2**
- Wq turun drastis:  
  \[
  W_q \sim \frac{\rho^{2}}{\mu(1-\rho)} \quad \text{turun signifikan}
  \]

---

### **C. Sistem Pre-Order / QR Pay untuk Menurunkan λ Aksi**
Jika sebagian transaksi dipindah ke:

- Pre-order
- Mobile ordering
- Pembayaran digital mandiri

Maka:

- λ kasir turun → antrean memendek  
- Variabilitas (var λ) lebih kecil  
- Sistem lebih mendekati kondisi stabil M/M/2

---

### **D. Dashboard Pemodelan Stokastik di Kantin ITERA**
Manfaat:

- Prediksi jumlah pelanggan harian  
- Prediksi waktu jenuh  
- Estimasi panjang antrean real-time  

Ini dapat menjadi **pilot project transformasi digital UMKM kampus**.

---

# 6. Pengembangan Sistem Antrian Kampus ke Depan

Beberapa potensi penelitian lanjutan:

1. **Model M/M/s dengan server berkapasitas berbeda**
2. **Model M/G/1 bila distribusi service time tidak eksponensial**
3. **Simulasi Monte Carlo untuk prediksi antrean saat event kampus**
4. **Agent-based simulation (Simmer R package)**
5. **Optimasi jumlah server minimal dengan SLA waktu tunggu**

---

# 7. Lisensi  
Bebas digunakan untuk kepentingan akademik.

