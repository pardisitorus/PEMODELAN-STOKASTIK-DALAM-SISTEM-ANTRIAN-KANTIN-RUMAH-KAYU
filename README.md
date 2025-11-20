# Tugas Besar Pemodelan Stokastik  
## Model Antrian Kantin Rumah Kayu ITERA  
### Penerapan Proses Poisson, Poisson Non-Homogen, dan Proses Kelahiran-Kematian pada Kasus Riil

Repository ini berisi implementasi komprehensif Pemodelan Stokastik berdasarkan data nyata jumlah pelanggan di Kantin Rumah Kayu ITERA. Studi ini secara eksplisit menghubungkan materi kuliah dari pertemuan I–XV dengan kasus riil, khususnya:

- **Proses Poisson & Non-Homogen (Pertemuan VI–X)**
- **Rantai Markov Waktu Kontinu (Pertemuan XI–XII)**
- **Birth–Death Processes sebagai model antrian M/M/1 dan M/M/s (Pertemuan XI–XII)**
- **Renewal Phenomena (Pertemuan XIV)**
- **Implementasi ke Tugas Besar (Pertemuan XV)**

Seluruh kode bersifat **anti-error** karena dataset dimasukkan secara manual ke dalam R script.

---

# 1. Judul
**Model Antrian Kantin Rumah Kayu ITERA Menggunakan Proses Poisson, Poisson Non-Homogen, dan Proses Kelahiran-Kematian**

---

# 2. Latar Belakang (Ringkas)
Kantin Rumah Kayu ITERA merupakan lokasi makan siang utama mahasiswa. Antrean panjang sering terjadi, terutama pada jam sibuk. Selain itu, kondisi cuaca (hujan vs tidak hujan) memengaruhi pola kedatangan mahasiswa.

Materi Pemodelan Stokastik yang dipelajari selama semester memberikan kerangka ilmiah untuk:

- Menganalisis **laju kedatangan (λ)**  
- Membangun model **antrian M/M/1 dan M/M/s**  
- Mengidentifikasi stabilitas sistem (ρ)  
- Memprediksi panjang antrean & waktu tunggu  
- Menilai dampak *Poisson Non-Homogen* saat hujan  

Tugas Besar ini menggabungkan seluruh konsep tersebut dalam studi nyata.

---

# 3. Struktur Dataset

Dataset berisi:

- **Tanggal**
- **Slot waktu 5 menit**
- **Jumlah pelanggan yang berhasil membayar**
- **Kondisi cuaca: Hujan / Tidak Hujan**

Data diringkas menjadi **laju kedatangan per hari** untuk dianalisis dengan model stokastik.

---

# 4. Alur Analisis Model Stokastik

## **Langkah 1 — Memasukkan dataset manual (anti-error)**
```r
library(tidyverse)

data <- tribble(
  ~Tanggal, ~Slot, ~Jumlah, ~Kondisi,
  "11/11/2025","11.50-11.55",6,"Tidak Hujan",
  ... (data lengkap dimasukkan manual)
)
```

## **Langkah 2 — Format tanggal & agregasi**
```r
data <- data %>%
  mutate(Tanggal = as.Date(Tanggal, format="%m/%d/%Y"))

ts_daily <- data %>%
  group_by(Tanggal, Kondisi) %>%
  summarise(total = sum(Jumlah), .groups="drop")
```

---

# 5. Estimasi Proses Poisson (Pertemuan VI–X)

Karena kedatangan pelanggan secara alami acak:

- kedatangan per satuan waktu → model **Poisson**  
- waktu antar kedatangan → berdistribusi **Eksponensial**

Estimasi λ harian:
```r
lambda_daily <- ts_daily %>%
  mutate(lambda = total / 12)    # 12 interval (5 menit) per jam
```

### Temuan:
- λₙₒₙ₋ₕᵤⱼₐₙ ≈ **119 pelanggan/hari**
- λₕᵤⱼₐₙ ≈ **83 pelanggan/hari**

Dampak hujan:  
\[
\text{Elastisitas} = \frac{\lambda_\text{hujan} - \lambda_\text{normal}}{\lambda_\text{normal}} \approx -0.3025
\]
Hujan menurunkan kedatangan sebesar **30.25%**.

---

# 6. Poisson Non-Homogen (Pertemuan IX–X)

Hujan menyebabkan perubahan λ secara mendadak → **Poisson Non-Homogen**:

\[
\lambda(t) = 
\begin{cases}
119, & \text{cuaca normal}\\
83, & \text{hujan}
\end{cases}
\]

Implikasi:

- Varians meningkat  
- Sistem antrean lebih tidak stabil  
- Service rate μ harus menyesuaikan kondisi

---

# 7. Proses Kelahiran-Kematian (Pertemuan XI–XII)

Sistem antrian:

- **Kelahiran = kedatangan pelanggan (λ)**
- **Kematian = pelanggan selesai dilayani (μ)**

Model dasar:

### **M/M/1 Model**
\[
\rho = \frac{\lambda}{\mu}
\]

### Contoh asumsi μ = 15 transaksi/jam:

- ρ\_normal ≈ 119/180 ≈ 0.66  
- ρ\_hujan ≈ 83/180 ≈ 0.46  

Kondisi **tidak hujan** mendekati jenuh pada jam puncak.

---

# 8. Analisis Tambahan (Deret Waktu Sederhana)

Digunakan untuk mendukung interpretasi stokastik.

### **Moving Average**
Menunjukkan tren penurunan menjelang hari hujan.

### **Differencing**
Perubahan ekstrem: +22 hingga –51 pelanggan.

### **ACF**
Tidak ada autokorelasi signifikan → **kedatangan independen**  
→ mendukung asumsi “Poisson Arrival”.

---

# 9. Perbandingan Hujan vs Tidak Hujan

| Kondisi | Rata-rata Pelanggan | Dampak |
|--------|----------------------|--------|
| Tidak hujan | 119 | Stabil tinggi |
| Hujan | 83 | –30% kedatangan |

---

# 10. Insight & Kebijakan Kampus (High-Impact)

### **1. Membangun Kanopi/Jalur Lindung ke Kantin**
Mengurangi gangguan hujan → λ meningkat → antrean lebih stabil.

### **2. Menambah 1 Mobile Counter pada Jam Puncak**
Mengubah sistem dari **M/M/1 → M/M/2**, menurunkan:

- Wq (waktu tunggu)
- Panjang antrean
- Probabilitas kekecewaan pelanggan

### **3. Sistem Pre-Order Digital**
Mengurangi λ\_kasir → meningkatkan stabilitas sistem.

### **4. Dashboard Real-Time untuk Kantin ITERA**
Pemodelan stokastik dapat:

- memprediksi puncak antrean  
- memantau utilisasi server (ρ)  
- menyarankan jumlah tenaga kasir optimal  

### **5. Simulasi Monte Carlo untuk Perencanaan Event**
Dapat memprediksi kebutuhan staf pada acara besar kampus.

---

# 11. Potensi Pengembangan Akademik

- Simulasi **M/G/1** (service time non-eksp.)
- **CTMC keluarga besar** untuk multi-counter
- Simulasi **Renewal Theory** untuk periode stok bahan
- Estimasi **NHPP (Non-Homogeneous Poisson Process)** yang lebih kompleks

---

# 12. Lisensi  
Bebas digunakan untuk keperluan akademik.
