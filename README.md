# README — Tim LogLoss | DSC MCF ITB 2026

## Deskripsi Proyek
Proyek ini merupakan pipeline prediksi klaim asuransi kesehatan untuk kompetisi **Data Science Competition — Mathematical Challenge Festival (MCF) ITB 2026**. Fokus utama notebook adalah membangun sistem prediksi untuk tiga target utama:

- **Claim Frequency**
- **Claim Severity**
- **Total Claim**

Pendekatan yang digunakan menggabungkan:

- **Preprocessing dan data cleaning**
- **Exploratory Data Analysis (EDA)**
- **Feature engineering berbasis deret waktu dan segmentasi**
- **Baseline structural model**
- **Hybrid model: Exponential Smoothing + LightGBM + Optuna**
- **Recursive forecasting untuk periode prediksi**
- **Penyimpanan model final ke file `model.joblib`**

---

## Dataset
Notebook ini menggunakan dataset berikut:

- `Data_Klaim.csv`
- `Data_Polis.csv`
- `sample_submission.csv`

Struktur path default pada notebook:

```python
BASE_PATH = "/kaggle/input/datasets/dimaspashaakrilian/dsc-itb/"
```

Pastikan ketiga file tersebut tersedia pada direktori input sebelum notebook dijalankan.

---

## Tujuan Notebook
Notebook ini disusun untuk:

1. Membersihkan dan menyiapkan data klaim dan polis.
2. Melakukan analisis deskriptif terhadap pola klaim.
3. Membangun fitur temporal dan fitur segmentasi.
4. Melatih model baseline dan model hybrid.
5. Mengevaluasi performa model menggunakan MAPE.
6. Menghasilkan prediksi akhir untuk format submission kompetisi.
7. Menyimpan model final dalam satu file artifact agar mudah dipakai ulang.

---

## Alur Notebook

### 1. Preprocessing & Data Cleaning
Tahap awal mencakup:

- Load data klaim dan polis
- Pembersihan nama kolom
- Penghapusan duplikasi
- Parsing kolom tanggal
- Pembersihan nilai penting
- Winsorization ringan pada nominal klaim
- Penggabungan data klaim dan polis
- Pembentukan variabel `year_month`

Output utama tahap ini adalah dataframe gabungan `df` yang sudah siap untuk analisis lanjutan.

### 2. Exploratory Data Analysis
EDA dilakukan untuk memahami karakteristik utama data, meliputi:

- Missing value summary
- Ringkasan statistik numerik dan kategorik
- Agregasi bulanan
- Visualisasi tren frequency, severity, dan total claim
- Analisis segmentasi berdasarkan plan code, care type, dan lokasi rumah sakit
- Analisis high-value claims
- Ringkasan otomatis untuk rekomendasi bisnis

### 3. Feature Engineering
Tahap ini membangun fitur untuk level agregat dan level segmen.

#### Fitur agregat bulanan
- `frequency`
- `total_claim`
- `exposure`
- `severity`
- `claim_rate`

#### Fitur transformasi
- `log_total`
- `log_freq`
- `log_sev`
- `log_rate`

#### Fitur volatilitas
- `roll6`
- `std6`
- `vol_ratio`
- `high_vol_regime`

#### Fitur musiman dan waktu
- `month`
- `month_sin`
- `month_cos`
- `month_index`

#### Fitur historis
- lag 1, 2, dan 3
- rolling mean 3 periode

#### Fitur segmentasi
Segmentasi dibangun berdasarkan:

- `plan_code`
- `care_type`
- `is_cashless`
- `rs_bucket`

Kemudian dibuat juga fitur tambahan seperti:

- `momentum_total`
- `seg_weight`

### 4. Model Training
Notebook melatih dua pendekatan utama:

#### Baseline Structural Model
- `total_claim` diprediksi menggunakan **Exponential Smoothing**
- `frequency` diperkirakan dari rata-rata `claim_rate`
- `severity` dihitung dari rasio `total_claim / frequency`

#### Hybrid ETS + LightGBM + Optuna
- ETS digunakan untuk menangkap pola deret waktu pada `total_claim`
- LightGBM memanfaatkan fitur hasil feature engineering
- Optuna digunakan untuk mencari parameter terbaik seperti:
  - `alpha`
  - `learning_rate`
  - `num_leaves`
  - `shrink`

### 5. Model Evaluation
Evaluasi menggunakan **MAPE (Mean Absolute Percentage Error)**.

Model dibandingkan pada data validasi untuk melihat:

- MAPE Frequency
- MAPE Severity
- MAPE Total Claim
- Estimated Score

### 6. Forecasting & Submission
Tahap akhir digunakan untuk:

- Melakukan recursive forecasting untuk horizon prediksi
- Membentuk prediksi sesuai format `sample_submission.csv`
- Menyimpan file akhir `submission.csv`

---

## Model Final
Versi final model disimpan dalam satu file:

```bash
model.joblib
```

Isi artifact `model.joblib` mencakup:

- LightGBM model final
- ETS model final
- Best hyperparameters
- Daftar fitur
- Metadata training penting

load model:

```python
import joblib

artifact = joblib.load("model.joblib")

lgb_model = artifact["lgb_model"]
ets_model = artifact["ets_model"]
best_params = artifact["best_params"]
features = artifact["features"]
```

---

## Dependensi
Notebook ini menggunakan library utama berikut:

```python
pandas
numpy
lightgbm
optuna
statsmodels
matplotlib
joblib
pickle
json
os
```

Jika dijalankan di luar Kaggle, instal dependensi yang diperlukan terlebih dahulu.


```bash
pip install pandas numpy lightgbm optuna statsmodels matplotlib joblib
```

---

## Cara Menjalankan
Urutan eksekusi yang disarankan:

1. Jalankan bagian **Preprocessing & Data Cleaning**
2. Jalankan bagian **Exploratory Data Analysis**
3. Jalankan bagian **Feature Engineering**
4. Jalankan bagian **Model Training**
5. Jalankan bagian **Model Evaluation**
6. Jalankan bagian **Forecasting & Submission**
7. Jalankan bagian **Model Save** jika ingin menyimpan model final

Notebook bersifat berurutan, sehingga beberapa tahap membutuhkan object dari tahap sebelumnya seperti `df`, `monthly`, `monthly_opt`, `BEST`, dan `features`.

---

## Output Utama
Beberapa output penting dari notebook ini adalah:

- `df` — data klaim dan polis yang sudah dibersihkan dan digabung
- `monthly` — tabel agregasi bulanan utama
- `seg_model` — data segmentasi siap model
- `study` — hasil optimasi Optuna
- `BEST` — parameter terbaik hasil tuning
- `submission.csv` — file prediksi akhir untuk kompetisi
- `model.joblib` — artifact model final

---

## Catatan
- Notebook ini dirancang untuk lingkungan **Kaggle Notebook**.
- Pastikan kolom penting seperti `nomor_polis`, `tanggal_pasien_masuk_rs`, dan `nominal_klaim_yang_disetujui` tersedia.
- Beberapa bagian notebook menggunakan object global yang dihasilkan dari cell sebelumnya.
- Jika ingin deployment atau inferensi ulang, gunakan file `model.joblib` agar tidak perlu melatih ulang model dari awal.

---

## Penulis
**Tim LogLoss**

- Dimas Pasha Akrilian
- Shata Alwan Jalaluddin
- Aaron Christian Daniel

Untuk kebutuhan kompetisi DSC MCF ITB 2026.
