# PEMODELAN-STOKASTIK-DALAM-SISTEM-ANTRIAN-KANTIN-RUMAH-KAYU

# Analisis Deret Waktu dan Sistem Antrian Kantin Rumah Kayu ITERA  
### Studi Pengaruh Cuaca (Hujan vs Tidak Hujan) terhadap Jumlah Pelanggan

Repositori ini berisi analisis lengkap terhadap pola kunjungan pelanggan ke Kantin Rumah Kayu ITERA berdasarkan data pengamatan lapangan. Studi ini menggunakan pendekatan analisis deret waktu sederhana untuk menemukan tren, volatilitas, stabilitas, dan dampak cuaca terhadap jumlah pelanggan.

Seluruh kode menggunakan data yang ditulis manual di dalam script, sehingga **100% anti error** dan tidak bergantung pada file eksternal.

---

## 1. Struktur Dataset  
Dataset dibuat langsung di R:

- **Tanggal**  
- **Slot waktu (5 menit)**  
- **Jumlah pelanggan yang berhasil membayar di kasir**  
- **Kondisi (Hujan vs Tidak Hujan)**

Data diringkas menjadi **total pelanggan per hari**, lalu dianalisis secara deret waktu.

---

## 2. Alur Analisis

### **Langkah 1 — Membuat dataset manual**
```r
library(tidyverse)

data <- tribble(
  ~Tanggal, ~Slot, ~Jumlah, ~Kondisi,
  "11/11/2025","11.50-11.55",6,"Tidak Hujan",
  ... (seluruh data dimasukkan manual seperti laporan)
)
```

### **Langkah 2 — Mengubah ke format tanggal & merangkum**
```r
data <- data %>% 
  mutate(Tanggal = as.Date(Tanggal, format = "%m/%d/%Y"))

ts_daily <- data %>%
  group_by(Tanggal, Kondisi) %>%
  summarise(total = sum(Jumlah), .groups = "drop")
```

### **Langkah 3 — Moving Average 3 Hari**
```r
library(zoo)
ts_daily <- ts_daily %>%
  arrange(Tanggal) %>%
  mutate(MA_3 = rollmean(total, k=3, fill=NA, align="right"))
```

### **Langkah 4 — Perubahan Antar Hari**
```r
ts_daily <- ts_daily %>%
  mutate(change = c(NA, diff(total)))
```

### **Langkah 5 — ACF (Autocorrelation)**
```r
acf(ts(ts_daily$total, frequency = 1))
```

### **Langkah 6 — Dampak Cuaca (Elastisitas Sederhana)**
```r
D_normal <- mean(ts_daily$total[ts_daily$Kondisi == "Tidak Hujan"])
D_hujan  <- mean(ts_daily$total[ts_daily$Kondisi == "Hujan"])

elasticity <- (D_hujan - D_normal) / D_normal
```

---

## 3. Visualisasi  
Visualisasi meliputi:

1. Deret waktu total pelanggan (`output_ts1.png`)  
2. Moving Average 3-Hari (`output_ma.png`)  
3. Perubahan harian (Δ) (`output_change.png`)  
4. ACF (`output_acf.png`)  
5. Perbandingan hujan vs tidak hujan (`output_compare.png`)

---

## 4. Insight Utama

- Jumlah pelanggan **menurun 30% pada hari hujan** (elastisitas = –0.30).  
- Pola kunjungan **stabil tetapi cenderung fluktuatif**, terutama menjelang tanggal 18–19.  
- ACF menunjukkan **tidak ada autokorelasi kuat**, artinya kunjungan tiap hari berdiri sendiri.  
- Moving Average menghaluskan tren dan mengungkap bahwa **tren jangka pendek sedikit menurun**.

---

## 5. Tujuan Repository  
Proyek ini dapat digunakan sebagai:

- Bahan Tugas Besar Pemodelan Stokastik  
- Dasar simulasi queueing model (M/M/1, M/M/s)  
- Analisis operasional untuk UMKM kampus  
- Bukti empiris untuk rekomendasi kepada pihak institut (Rektorat)

---

## 6. Lisensi  
Bebas digunakan untuk keperluan akademik.

