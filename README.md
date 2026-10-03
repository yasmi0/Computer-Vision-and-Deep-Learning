# ResNet-50 Transfer Learning

Transfer Learning ResNet-50 dengan 3 mode eksperimen: Feature Extraction, Fine-Tuning Partial, dan Fine-Tuning Full.

## 📊 Dataset

- **Total gambar**: 100
- **Kelas**: 2 (`landing_pad`, `not_landing_pad`)
- **Split**: Train 70 (70%) | Val 20 (20%) | Test 10 (10%)
- **Stratified split**: Ya (proporsi kelas seimbang di setiap split)
- **Random seed**: 42

> ⚠️ **Catatan**: Dataset berukuran kecil (100 gambar). Akurasi 1.0 pada test set (10 gambar) tidak dapat digeneralisasi secara langsung ke populasi yang lebih luas. Hasil validasi harus diverifikasi lebih lanjut pada kumpulan data yang lebih besar.

> Dataset tersedia di Google Drive: [Link](https://drive.google.com/drive/folders/1Bz9DVRyVrGYNLZfTsYn2sjf2WTBG2dIZ?usp=sharing)

## 🧠 Metodologi

- **Model dasar**: ResNet-50 pre-trained ImageNet (23 juta parameter)
- **Input**: 224×224 piksel, kanal RGB
- **Preprocessing**: Resize 224×224, normalisasi ImageNet (mean=[0.485, 0.456, 0.406], std=[0.229, 0.224, 0.225])
- **Model selection**: Berdasarkan kombinasi **Validation Accuracy** dan **Validation Loss**, untuk menghindari kebocoran data test set
- **Test set**: Hanya digunakan sekali di akhir untuk evaluasi akhir

### Detail 3 Mode

| Mode | Layer yang Dilatih | LR Awal | Epoch Max | Augmentasi |
|---|---|---|---|---|
| Feature Extraction | Hanya head classifier | 1e-3 | 15 | Tidak |
| Fine-Tuning Partial | Head + 30 layer terakhir | 5e-4 | 25 | Ya |
| Fine-Tuning Full | Seluruh layer backbone | 1e-4 | 25 | Ya |

## 📈 Hasil Eksperimen

### Tabel Hasil Validasi

| Mode | Train Acc | Val Acc | Train Loss | Val Loss | Best Epoch | Total Epochs |
|---|---|---|---|---|---|---|
| Feature Extraction | 0.728571 | 1.000000 | 0.523622 | 0.249589 | 2 | 7 |
| **Fine-Tuning Partial** | 0.685714 | 1.000000 | 0.559109 | **0.025070** | 1 | 6 |
| Fine-Tuning Full | 0.685714 | 1.000000 | 0.612435 | 0.560713 | 1 | 6 |

### Tabel Final (Validasi + Test)

| Mode | Val Acc | Val Loss | Test Acc | Test Loss |
|---|---|---|---|---|
| Feature Extraction | 1.000000 | 0.249589 | 1.000000 | 0.243441 |
| **Fine-Tuning Partial** | 1.000000 | **0.025070** | 1.000000 | **0.027675** |
| Fine-Tuning Full | 1.000000 | 0.560713 | 1.000000 | 0.549323 |

### Grafik Perbandingan Model
![Grafik Akurasi](https://github.com/yasmi0/Computer-Vision-and-Deep-Learning/blob/main/grafik_akurasi.png)

### 🏆 Model Terbaik

Dipilih berdasarkan kombinasi **Validation Accuracy** dan **Validation Loss**:

- **Mode**: Fine-Tuning Partial
- **Val Accuracy**: 1.000000
- **Val Loss**: 0.025070 *(paling rendah dari ketiga mode)*
- **Test Accuracy**: 1.000000
- **Test Loss**: 0.027675 *(paling rendah dari ketiga mode)*
- **Latensi**: **19.58 ms/gambar (± 2.31 ms)**

**Alasan pemilihan**: Meskipun ketiga mode memiliki akurasi yang sama (1.0), Fine-Tuning Partial memiliki **loss paling rendah**, yang berarti model Fine-Tuning Partial ini paling **yakin** dengan prediksinya.

## 🔬 Analisis

### 1. Ketiga Mode Berhasil Mencapai Akurasi Sempurna (1.0)

Setelah perbaikan hyperparameter, ketiga mode mencapai **Val Acc = 1.0** dan **Test Acc = 1.0**. Hal ini menunjukkan bahwa ResNet-50 pre-trained ImageNet mampu mengekstraksi fitur yang relevan untuk membedakan kelas `landing_pad` dan `not_landing_pad`, bahkan pada dataset kecil (100 gambar).

**Perbaikan kunci yang membuat Mode 2 & 3 berhasil:**
- Learning rate dinaikkan dari `1e-5` → `5e-4` (Mode 2) dan `1e-4` (Mode 3).
- Epoch dinaikkan dari 10 → 25.
- Augmentasi data (flip, rotation, zoom) ditambahkan.
- `ReduceLROnPlateau` membantu konvergensi halus.
- `EarlyStopping` patience dinaikkan dari 3 → 5.

### 2. Fine-Tuning Partial adalah Model Terbaik Secara Kualitas

Meskipun **ketiga mode punya akurasi yang sama (1.0)**, **Fine-Tuning Partial unggul secara kualitas prediksi**:

| Mode | Val Loss | Test Loss | Interpretasi |
|---|---|---|---|
| Feature Extraction | 0.249589 | 0.243441 | Cukup yakin |
| **Fine-Tuning Partial** | **0.025070** | **0.027675** | **Sangat yakin** |
| Fine-Tuning Full | 0.560713 | 0.549323 | Kurang yakin |

**Loss yang rendah** berarti model tidak hanya benar dalam klasifikasi, tapi juga **yakin** dengan prediksinya (probabilitas output mendekati 1.0 untuk kelas yang benar).

**Fine-Tuning Partial lebih baik dari Fine-Tuning Full Karena:**
- Fine-Tuning Partial membuka hanya 30 layer terakhir (blok conv5) — cukup untuk menyesuaikan fitur tingkat tinggi tanpa overfitting.
- Fine-Tuning Full membuka **seluruh layer backbone** (termasuk BatchNorm dan Activation, sekitar 175 layer operasi) — terlalu banyak parameter untuk dataset 70 gambar, sehingga model **overfitting** (Val Loss = 0.56 meskipun akurasi 1.0).

Hal ini sesuai dengan teori transfer learning: **semakin kecil dataset, semakin sedikit layer yang boleh di-unfreeze**.

### 3. Anomali: Train Acc (0.69) < Val Acc (1.0)

Hal ini terjadi di ketiga mode dan **bukan tanda model jelek**. Penyebabnya:

- **Dropout 0.5** aktif saat training (mempersulit prediksi), nonaktif saat validasi.
- **Augmentasi data** aktif saat training (menambah variasi), nonaktif saat validasi.
- **Val set hanya 20 gambar** — val set yang kecil ini kebetulan mudah diklasifikasi oleh model.

Artinya: model sebenarnya **belajar dengan baik**, hanya saja evaluasi training dilakukan dalam kondisi yang lebih sulit. Ini adalah **perilaku normal** ketika dropout + augmentasi digunakan.

### 4. Best Epoch = 1 untuk Mode 2 & 3

Mode 2 dan 3 mencapai akurasi terbaik di **epoch pertama**. Ini menandakan:

- Model dengan cepat menemukan solusi yang "cukup baik".
- Dataset kecil membuat model konvergen sangat cepat.
- Dengan dataset lebih besar, biasanya best epoch akan lebih tinggi (5-15).

### 5. Keterbatasan Evaluasi pada Dataset Kecil

Meskipun hasil terlihat sempurna (akurasi 1.0), ada beberapa keterbatasan yang perlu diperhatikan:

- **Test set hanya 10 gambar**. Akurasi 1.0 dengan 10 gambar memiliki **confidence interval lebar** (untuk n=10, akurasi 1.0 memiliki 95% CI sekitar ±0.30).
- **Satu gambar salah** akan menurunkan akurasi menjadi 0.9.
- Hasil **tidak dapat digeneralisasi secara langsung** ke populasi yang lebih luas tanpa validasi tambahan.

## 📦 Dataset & Model

- **Dataset** (`dataset_raw/`): Tersedia di repositori ini (3 MB)
- **Model terbaik (Fine-Tuning Partial)**: [Google Drive Link](https://drive.google.com/file/d/1E5Or7mtRVsWhAAyKgsan3hd7znc2ZLDd/view?usp=sharing)
- **Model Fine-Tuning Full**: [Google Drive Link](https://drive.google.com/file/d/1RN3L705o7q83BY9HUhIkZFIcSy2XBK-3/view?usp=sharing)
- **Model Feature Extraction**: [Google Drive Link](https://drive.google.com/file/d/1GLaqgHof9kSpgHkNzBnEPgV7YJpQEeD7/view?usp=sharing)
- **Semua model (3 mode)**: [Google Drive Link](https://drive.google.com/file/d/1d5V2RjagQUioZL0Y6ivFEDBwE_Tjipb8/view?usp=sharing)

> ⚠️ File model (`.keras`) tidak disertakan dalam repositori karena melebihi batas ukuran GitHub (100 MB per file). Model tersedia melalui Google Drive.

## 📂 Struktur Repositori
