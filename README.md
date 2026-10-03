# ResNet-50 Transfer Learning

Proyek ini memakai teknik **transfer learning** dengan model **ResNet-50** untuk mengklasifikasi gambar. Ada 3 cara yang dicoba:

1. **Feature Extraction** — hanya melatih "otak" terakhir model
2. **Fine-Tuning Partial** — melatih sebagian model
3. **Fine-Tuning Full** — melatih seluruh model

---

## 📊 Dataset

- **Jumlah gambar**: 100
- **Jumlah kelas**: 2 (ada `landing_pad` / tidak ada `landing_pad`)
- **Pembagian data**: 
  - 70 gambar untuk **training** (latihan)
  - 20 gambar untuk **validasi** (uji sementara)
  - 10 gambar untuk **testing** (uji akhir)
- **Random seed**: 42 (agar hasilnya dapat diulang)

> ⚠️ **Penting**: Dataset ini kecil (hanya 100 gambar). Jadi meskipun hasilnya terlihat sempurna (100% benar), ini **belum tentu berlaku** untuk gambar-gambar baru di dunia nyata. Diperlukan dataset yang lebih besar untuk memastikan.

> 📁 Dataset lengkap tersedia di [Google Drive](https://drive.google.com/drive/folders/1Bz9DVRyVrGYNLZfTsYn2sjf2WTBG2dIZ?usp=sharing)

---

## 🧠 Cara Kerja (Metodologi)

Secara sederhana, seperti ini alurnya:

1. **Ambil model ResNet-50 yang sudah pintar** — model ini sebelumnya sudah dilatih dengan 1,2 juta gambar (ImageNet). Jadi dia sudah tahu cara mengenali bentuk, tepi, dan tekstur gambar.
2. **Sesuaikan dengan tugas kita** — kita hanya perlu "mengajari ulang" bagian akhirnya agar bisa membedakan `landing_pad` vs `not_landing_pad`.
3. **Latih dengan data kita** — 70 gambar training dipakai untuk belajar.

**Pengaturan teknis:**
- **Ukuran input**: 224×224 piksel (standar ResNet-50)
- **Warna**: RGB (bukan BGR, karena pakai TensorFlow bukan OpenCV)
- **Normalisasi**: Pakai standar ImageNet (mean=[0.485, 0.456, 0.406], std=[0.229, 0.224, 0.225]) — agar gambar yang masuk ke model "terasa familiar" seperti gambar training aslinya
- **Cara pilih model terbaik**: Ambil dari nilai **validasi** (bukan testing!) — agar test set tidak "bocor" ke proses pemilihan model
- **Data testing**: Hanya dipakai **sekali di akhir** untuk laporan

### Detail 3 Mode

| Mode | Bagian yang Dilatih | Learning Rate | Max Epoch | Augmentasi |
|---|---|---|---|---|
| Feature Extraction | Hanya "otak" terakhir | 1e-3 | 15 | Tidak |
| Fine-Tuning Partial | "Otak" + 30 layer terakhir | 5e-4 | 25 | Ya |
| Fine-Tuning Full | Seluruh model | 1e-4 | 25 | Ya |

> 💡 **Augmentasi** adalah teknik memperbanyak variasi gambar dengan cara memutar, membalik, atau memperbesar gambar asli. Gunanya agar model tidak "hafal mati" pada gambar training.

---

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

---

## 🏆 Model Terbaik

Dipilih berdasarkan kombinasi **Validation Accuracy** dan **Validation Loss**:

- **Mode**: Fine-Tuning Partial
- **Val Accuracy**: 1.000000
- **Val Loss**: **0.025070** *(paling rendah dari ketiga mode)*
- **Test Accuracy**: 1.000000
- **Test Loss**: **0.027675** *(paling rendah dari ketiga mode)*
- **Latensi**: **19.58 ms/gambar (± 2.31 ms)**

> 💡 **Kenapa Loss lebih penting dari Akurasi?**
> Ketiga model semuanya memiliki akurasi 100%, sehingga dibutuhkan cara lain untuk membedakan mana yang terbaik. Loss mengukur seberapa **yakin** model — makin kecil loss, makin yakin model. Fine-Tuning Partial memiliki loss paling rendah, yang berarti model ini paling **yakin** dengan prediksinya.

---

## 🔬 Analisis Hasil

### 1. Ketiga Mode Berhasil Mencapai Akurasi Sempurna (1.0)

Setelah beberapa kali perbaikan, ketiga cara akhirnya berhasil mencapai akurasi sempurna (100%). Artinya, ResNet-50 mampu membedakan `landing_pad` dan `not_landing_pad` meskipun hanya dilatih dengan 70 gambar.

**Perbaikan yang membuat Mode 2 & 3 berhasil:**

| Sebelum | Sesudah | Efeknya |
|---|---|---|
| Learning Rate 1e-5 (terlalu kecil) | 5e-4 dan 1e-4 | Model jadi benar-benar belajar |
| Epoch 10 (terlalu sedikit) | 25 | Model punya cukup waktu belajar |
| Tidak ada augmentasi | Ada (flip, rotasi, zoom) | Model tidak mudah hafal |
| EarlyStopping patience 3 | 5 | Training tidak berhenti terlalu cepat |

### 2. Fine-Tuning Partial adalah Model Terbaik Secara Kualitas

Meskipun **ketiga mode punya akurasi yang sama (1.0)**, **Fine-Tuning Partial unggul secara kualitas prediksi:**

| Mode | Val Loss | Test Loss | Interpretasi |
|---|---|---|---|
| Feature Extraction | 0.249589 | 0.243441 | Cukup yakin |
| **Fine-Tuning Partial** | **0.025070** | **0.027675** | **Sangat yakin** |
| Fine-Tuning Full | 0.560713 | 0.549323 | Kurang yakin |

**Fine-Tuning Partial lebih baik dari Fine-Tuning Full karena:**
- Fine-Tuning Partial membuka hanya 30 layer terakhir (blok conv5) — cukup untuk menyesuaikan fitur tingkat tinggi tanpa overfitting.
- Fine-Tuning Full membuka **seluruh layer backbone** (termasuk BatchNorm dan Activation, sekitar 175 layer operasi) — terlalu banyak parameter untuk dataset 70 gambar, sehingga model **overfitting** (Val Loss = 0.56 meskipun akurasi 1.0).

> 📌 **Semakin kecil dataset, semakin sedikit bagian model yang boleh diubah.**

### 3. Anomali: Train Acc (0.69) < Val Acc (1.0)

Hal ini terjadi di ketiga mode dan **bukan tanda model jelek**. Penyebabnya:

1. **Dropout 0.5** — saat training, sebagian neuron sengaja dimatikan agar model tidak hafal. Hal ini membuat training lebih susah. Saat validasi, semua neuron aktif lagi, sehingga lebih mudah.
2. **Augmentasi** — saat training, gambar-gambar diputar/dibalik sehingga lebih susah dikenali. Saat validasi, tidak ada augmentasi, sehingga lebih mudah.
3. **Val set hanya 20 gambar** — val set yang kecil ini kebetulan mudah diklasifikasi oleh model.

**Kesimpulan**: Model sebenarnya **belajar dengan baik**, hanya saja evaluasi training dilakukan dalam kondisi yang lebih sulit. Ini adalah **perilaku normal** ketika dropout + augmentasi digunakan.

### 4. Best Epoch = 1 untuk Mode 2 dan 3

Mode 2 dan 3 mendapatkan akurasi terbaik di **epoch pertama**. Hal ini menunjukkan bahwa:

- Model dengan cepat menemukan solusi yang "cukup baik"
- Dataset kecil membuat proses belajar sangat cepat
- Dengan dataset lebih besar, best epoch akan lebih tinggi (5-15)

### 5. Keterbatasan Evaluasi pada Dataset Kecil

Meskipun hasilnya terlihat sempurna (akurasi 100%), ada beberapa hal yang perlu diperhatikan:

- **Test set cuma 10 gambar**. Kalau 1 gambar salah saja, akurasi langsung turun ke 90%. Jadi angka 100% ini **belum tentu akurat** untuk data lain.
- **Confidence interval lebar** — untuk 10 gambar, akurasi 100% itu rentangnya bisa ±30%. Artinya bisa jadi akurasi sebenarnya antara 70%-100%.
- **Belum diuji dengan gambar baru** di luar dataset — jadi belum tentu bekerja baik di dunia nyata.

### 6. Kesimpulan

| Masalah | Kesimpulan |
|---|---|
| Learning Rate 1e-5 terlalu kecil | Naikkan LR kalau model tidak belajar |
| Augmentasi penting untuk dataset kecil | Aktifkan saat fine-tuning |
| Fine-Tuning Partial lebih baik dari Full | Jangan unfreeze semua layer |
| Akurasi 100% bukan jaminan model bagus | Cek juga nilai Loss |
| Dataset kecil bikin hasil tidak stabil | Perbesar dataset atau pakai cross-validation |

---

## 📦 Dataset & Model

- **Dataset** (`dataset_raw/`): Ada di repo ini (3 MB)
- **Model terbaik (Fine-Tuning Partial)**: [Google Drive](https://drive.google.com/file/d/1E5Or7mtRVsWhAAyKgsan3hd7znc2ZLDd/view?usp=sharing)
- **Model Fine-Tuning Full**: [Google Drive](https://drive.google.com/file/d/1RN3L705o7q83BY9HUhIkZFIcSy2XBK-3/view?usp=sharing)
- **Model Feature Extraction**: [Google Drive](https://drive.google.com/file/d/1GLaqgHof9kSpgHkNzBnEPgV7YJpQEeD7/view?usp=sharing)
- **Semua model (3 mode)**: [Google Drive](https://drive.google.com/file/d/1d5V2RjagQUioZL0Y6ivFEDBwE_Tjipb8/view?usp=sharing)

> ⚠️ File model (`.keras`) **tidak di-upload ke GitHub** karena ukurannya lebih dari 100 MB per file. Silakan download dari Google Drive.
