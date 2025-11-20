############################################################
# TUGAS BESAR PEMODELAN STOKASTIK
# Model Antrian Kantin Rumah Kayu ITERA: M/M/1
# - Kedatangan ~ Proses Poisson (laju λ)
# - Pelayanan ~ Eksponensial (laju μ)
# - Satu kasir  →  Birth–Death Process (M/M/1)
############################################################

library(tidyverse)

############################################################
# 1. DATASET MANUAL (ANTI ERROR)
#    Tanggal, Slot 5 menit, Jumlah ke kasir, Kondisi cuaca
############################################################

data_raw <- tribble(
  ~Tanggal,    ~Slot,           ~Jumlah, ~Kondisi,
  # 11 November 2025 - Tidak Hujan
  "11/11/2025","11.50-11.55",    6,      "Tidak Hujan",
  "11/11/2025","11.55-12.00",    7,      "Tidak Hujan",
  "11/11/2025","12.00-12.05",    12,     "Tidak Hujan",
  "11/11/2025","12.05-12.10",    14,     "Tidak Hujan",
  "11/11/2025","12.10-12.15",    8,      "Tidak Hujan",
  "11/11/2025","12.15-12.20",    9,      "Tidak Hujan",
  "11/11/2025","12.20-12.25",    9,      "Tidak Hujan",
  "11/11/2025","12.25-12.30",    7,      "Tidak Hujan",
  "11/11/2025","12.30-12.35",    13,     "Tidak Hujan",
  "11/11/2025","12.35-12.40",    11,     "Tidak Hujan",
  "11/11/2025","12.40-12.45",    9,      "Tidak Hujan",
  "11/11/2025","12.45-12.50",    7,      "Tidak Hujan",

  # 12 November 2025 - Tidak Hujan
  "11/12/2025","11.50-11.55",    10,     "Tidak Hujan",
  "11/12/2025","11.55-12.00",    11,     "Tidak Hujan",
  "11/12/2025","12.00-12.05",    12,     "Tidak Hujan",
  "11/12/2025","12.05-12.10",    12,     "Tidak Hujan",
  "11/12/2025","12.10-12.15",    11,     "Tidak Hujan",
  "11/12/2025","12.15-12.20",    13,     "Tidak Hujan",
  "11/12/2025","12.20-12.25",    5,      "Tidak Hujan",
  "11/12/2025","12.25-12.30",    5,      "Tidak Hujan",
  "11/12/2025","12.30-12.35",    9,      "Tidak Hujan",
  "11/12/2025","12.35-12.40",    7,      "Tidak Hujan",
  "11/12/2025","12.40-12.45",    9,      "Tidak Hujan",
  "11/12/2025","12.45-12.50",    11,     "Tidak Hujan",

  # 13 November 2025 - Tidak Hujan
  "11/13/2025","11.50-11.55",    6,      "Tidak Hujan",
  "11/13/2025","11.55-12.00",    8,      "Tidak Hujan",
  "11/13/2025","12.00-12.05",    12,     "Tidak Hujan",
  "11/13/2025","12.05-12.10",    13,     "Tidak Hujan",
  "11/13/2025","12.10-12.15",    8,      "Tidak Hujan",
  "11/13/2025","12.15-12.20",    9,      "Tidak Hujan",
  "11/13/2025","12.20-12.25",    13,     "Tidak Hujan",
  "11/13/2025","12.25-12.30",    12,     "Tidak Hujan",
  "11/13/2025","12.30-12.35",    14,     "Tidak Hujan",
  "11/13/2025","12.35-12.40",    13,     "Tidak Hujan",
  "11/13/2025","12.40-12.45",    14,     "Tidak Hujan",
  "11/13/2025","12.45-12.50",    12,     "Tidak Hujan",

  # 19 November 2025 - Tidak Hujan
  "11/19/2025","11.50-11.55",    8,      "Tidak Hujan",
  "11/19/2025","11.55-12.00",    8,      "Tidak Hujan",
  "11/19/2025","12.00-12.05",    4,      "Tidak Hujan",
  "11/19/2025","12.05-12.10",    7,      "Tidak Hujan",
  "11/19/2025","12.10-12.15",    16,     "Tidak Hujan",
  "11/19/2025","12.15-12.20",    11,     "Tidak Hujan",
  "11/19/2025","12.20-12.25",    12,     "Tidak Hujan",
  "11/19/2025","12.25-12.30",    12,     "Tidak Hujan",
  "11/19/2025","12.30-12.35",    7,      "Tidak Hujan",
  "11/19/2025","12.35-12.40",    13,     "Tidak Hujan",
  "11/19/2025","12.40-12.45",    14,     "Tidak Hujan",
  "11/19/2025","12.45-12.50",    3,      "Tidak Hujan",

  # 18 November 2025 - Hujan
  "11/18/2025","11.50-11.55",    15,     "Hujan",
  "11/18/2025","11.55-12.00",    4,      "Hujan",
  "11/18/2025","12.00-12.05",    3,      "Hujan",
  "11/18/2025","12.05-12.10",    2,      "Hujan",
  "11/18/2025","12.10-12.15",    2,      "Hujan",
  "11/18/2025","12.15-12.20",    8,      "Hujan",
  "11/18/2025","12.20-12.25",    15,     "Hujan",
  "11/18/2025","12.25-12.30",    12,     "Hujan",
  "11/18/2025","12.30-12.35",    7,      "Hujan",
  "11/18/2025","12.35-12.40",    6,      "Hujan",
  "11/18/2025","12.40-12.45",    7,      "Hujan",
  "11/18/2025","12.45-12.50",    2,      "Hujan"
)

# Cek singkat
nrow(data_raw)  # harus 60
head(data_raw)
tail(data_raw)

############################################################
# 2. RINGKAS PER HARI (TOTAL PELANGGAN / JAM PENGAMATAN)
############################################################

data_daily <- data_raw %>%
  mutate(Tanggal = as.Date(Tanggal, format = "%m/%d/%Y")) %>%
  group_by(Tanggal, Kondisi) %>%
  summarise(
    total = sum(Jumlah),      # total pelanggan dalam 1 jam
    .groups = "drop"
  )

data_daily

############################################################
# 3. ESTIMASI LAJU KEDATANGAN λ (Poisson) PER KONDISI CUACA
#    λ di sini = rata-rata jumlah pelanggan per jam
############################################################

lambda_by_cond <- data_daily %>%
  group_by(Kondisi) %>%
  summarise(
    lambda = mean(total),          # pelanggan per jam
    sd_total = sd(total),
    .groups = "drop"
  )

lambda_by_cond

# Simpan ke variabel
lambda_normal <- lambda_by_cond$lambda[lambda_by_cond$Kondisi == "Tidak Hujan"]
lambda_hujan  <- lambda_by_cond$lambda[lambda_by_cond$Kondisi == "Hujan"]

lambda_normal
lambda_hujan

############################################################
# 4. FUNGSI M/M/1 (MODEL KELAHIRAN-KEMATIAN)
#    Input : λ (kedatangan), μ (pelayanan) per jam
#    Output: ρ, L, Lq, W, Wq
############################################################

mm1_metrics <- function(lambda, mu) {
  if (lambda >= mu) {
    stop("Sistem tidak stabil: lambda >= mu. Pilih mu lebih besar dari lambda.")
  }
  
  rho <- lambda / mu                 # tingkat utilisasi
  L   <- rho / (1 - rho)             # rata-rata pelanggan dalam sistem
  Lq  <- rho^2 / (1 - rho)           # rata-rata dalam antrian
  W   <- 1 / (mu - lambda)           # waktu rata-rata dalam sistem (jam)
  Wq  <- rho / (mu - lambda)         # waktu rata-rata menunggu (jam)
  
  tibble(
    lambda = lambda,
    mu     = mu,
    rho    = rho,
    L      = L,
    Lq     = Lq,
    W      = W,
    Wq     = Wq
  )
}

############################################################
# 5. SETTING μ (LAJU PELAYANAN)
#    Asumsi: satu kasir mampu melayani ~30 pelanggan per jam
#    → rata-rata 2 menit per transaksi
############################################################

mu_kasir <- 30   # pelanggan per jam

############################################################
# 6. HITUNG METRIK M/M/1 UNTUK
#    - KONDISI TIDAK HUJAN
#    - KONDISI HUJAN
############################################################

mm1_normal <- mm1_metrics(lambda_normal, mu_kasir) %>%
  mutate(Kondisi = "Tidak Hujan")

mm1_hujan <- mm1_metrics(lambda_hujan, mu_kasir) %>%
  mutate(Kondisi = "Hujan")

mm1_compare <- bind_rows(mm1_normal, mm1_hujan) %>%
  select(Kondisi, everything())

mm1_compare

############################################################
# 7. INTERPRETASI ANGKA (DALAM KOMENTAR)
############################################################

# - rho (utilisasi) mendekati 1 artinya kasir sangat sibuk.
#   Jika rho_normal jauh lebih tinggi dari rho_hujan,
#   berarti saat tidak hujan sistem hampir jenuh.
#
# - L  : rata-rata jumlah pelanggan dalam sistem (antri + dilayani)
# - Lq : rata-rata jumlah yang menunggu dalam antrian
# - W  : waktu rata-rata pelanggan berada dalam sistem (jam)
# - Wq : waktu rata-rata hanya menunggu (jam)
#
# Untuk mengubah W, Wq ke menit:
mm1_compare_min <- mm1_compare %>%
  mutate(
    W_min  = W  * 60,
    Wq_min = Wq * 60
  )

mm1_compare_min

############################################################
# 8. VISUALISASI SEDERHANA PERBANDINGAN NORMAL VS HUJAN
############################################################

# Barplot utilisasi rho
ggplot(mm1_compare_min, aes(x = Kondisi, y = rho, fill = Kondisi)) +
  geom_col(width = 0.6) +
  ylim(0, 1) +
  labs(
    title = "Utilisasi Kasir (ρ) M/M/1\nKantin Rumah Kayu ITERA",
    x = "Kondisi Cuaca",
    y = "ρ (lambda / mu)"
  ) +
  theme_minimal()

# Barplot waktu tunggu rata-rata dalam menit
ggplot(mm1_compare_min, aes(x = Kondisi, y = Wq_min, fill = Kondisi)) +
  geom_col(width = 0.6) +
  labs(
    title = "Waktu Tunggu Rata-Rata (Wq) dalam Menit\nModel M/M/1 Kantin",
    x = "Kondisi Cuaca",
    y = "Wq (menit)"
  ) +
  theme_minimal()
