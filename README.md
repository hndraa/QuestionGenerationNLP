# Automatic Question Generation for Reading Comprehension

> **Generate Pertanyaan untuk Mengukur Pemahaman Bacaan bagi Siswa Sekolah Dasar**

Proyek ini merupakan proyek **Tugas Akhir/Skripsi** yang mengembangkan sistem **Automatic Question Generation (AQG)** untuk menghasilkan pertanyaan literal secara otomatis dari teks bacaan berbahasa Indonesia bagi siswa sekolah dasar.

Sistem mengimplementasikan dan membandingkan dua pendekatan, yaitu **rule-based** dan **machine learning berbasis Transformer (IndoBART)**. Pendekatan rule-based digunakan untuk membentuk pertanyaan berdasarkan struktur linguistik kalimat, sedangkan hasil pertanyaan yang telah dievaluasi digunakan sebagai dataset untuk melatih model machine learning agar dapat menghasilkan pertanyaan dengan struktur yang lebih fleksibel dan natural.

---

## 🎯 Project Overview

Kemampuan memahami bacaan merupakan salah satu keterampilan penting dalam pembelajaran siswa sekolah dasar. Salah satu bentuk latihan pemahaman literal adalah pertanyaan yang jawabannya dapat ditemukan secara langsung pada teks.

Namun, penyusunan pertanyaan secara manual dapat membutuhkan waktu dan menghasilkan variasi pertanyaan yang terbatas.

Oleh karena itu, proyek ini mengembangkan sistem yang dapat:

* Membaca dan memproses teks berbahasa Indonesia.
* Menganalisis struktur linguistik setiap kalimat.
* Mengekstraksi unsur **Subjek, Predikat, Objek, dan Keterangan (S-P-O-K)**.
* Menghasilkan pertanyaan literal secara otomatis.
* Mengklasifikasikan pertanyaan berdasarkan jenis informasi yang ditanyakan.
* Menggunakan hasil pertanyaan rule-based sebagai dataset untuk model Transformer.
* Membandingkan kualitas pertanyaan dari pendekatan rule-based dan machine learning.

---

## 🧠 Approach

Penelitian menggunakan dua pendekatan utama:

### 1. Rule-Based Question Generation

Pendekatan rule-based menggunakan aturan linguistik berdasarkan hasil analisis NLP.

Tahapan utamanya:

```text
Input Text
    ↓
Sentence Segmentation
    ↓
Tokenization
    ↓
Lemmatization
    ↓
POS Tagging
    ↓
Dependency Parsing
    ↓
S-P-O-K Extraction
    ↓
Linguistic Rules
    ↓
Literal Question
```

Sistem menggunakan informasi dependency relation seperti:

* `nsubj` → Subject
* `root` → Predicate
* `obj` → Object
* `obl`
* `nmod`
* `advmod`
* `amod`
* `xcomp`
* `acl:relcl`

untuk membantu mengidentifikasi struktur kalimat.

Jenis pertanyaan yang dihasilkan antara lain:

| Informasi | Kata Tanya |
| --------- | ---------- |
| Subjek    | Siapa      |
| Objek     | Apa        |
| Waktu     | Kapan      |
| Lokasi    | Di mana    |
| Tujuan    | Ke mana    |
| Asal      | Dari mana  |

Contoh:

**Kalimat**

> Rina merapikan tempat tidur di kamarnya.

**Pertanyaan**

> Apa yang Rina rapikan di kamarnya?

---

### 2. Machine Learning Question Generation

Pertanyaan hasil rule-based yang telah melalui proses evaluasi digunakan sebagai dasar pembentukan dataset machine learning.

Model Transformer yang diuji:

* **mT5-small**
* **IndoBART**

Berdasarkan pengujian awal, **IndoBART** digunakan sebagai model utama karena menghasilkan pertanyaan yang lebih stabil dan natural pada dataset penelitian.

Model menggunakan pendekatan **text-to-text generation** dengan instruction prefix.

Contoh:

```text
Input:
generate siapa: Pagi itu Rina bangun lebih awal.

Target:
Siapa yang bangun pagi itu lebih awal?
```

Instruction prefix digunakan untuk membantu model memahami jenis pertanyaan yang harus dihasilkan.

---

## 📊 Dataset

Dataset dikumpulkan dari beberapa sumber, dengan sumber utama berupa:

* Buku Bahasa Indonesia Kurikulum Merdeka kelas 3–6.
* Cerita anak dari majalah Bobo.
* Teks sederhana dari sumber internet.
* Kalimat tambahan yang dibuat menggunakan AI dan kemudian diseleksi secara manual.

Dataset difokuskan pada **kalimat sederhana** agar sesuai dengan kebutuhan pembentukan pertanyaan literal dan mengurangi kompleksitas analisis sintaksis.

Hasil pengumpulan menghasilkan:

* **105 file cerita `.txt`**
* Rata-rata sekitar **25 kalimat sederhana per cerita**
* Dataset kemudian digabungkan menjadi dataset utama dalam format `.csv`.

Contoh struktur dataset:

```text
id_teks | id_kalimat | kalimat
------------------------------------------------
1       | 1          | Pagi itu Rina bangun lebih awal.
1       | 2          | Rina merapikan tempat tidur di kamarnya.
1       | 3          | Ibu memanggil Rina dari dapur.
```

---

## 🔎 NLP Preprocessing

Preprocessing dilakukan menggunakan **Stanza Bahasa Indonesia**.

Tahapan preprocessing meliputi:

### Tokenization

Memisahkan kalimat menjadi token.

```text
Rina merapikan tempat tidur di kamarnya.

↓
Rina
merapikan
tempat
tidur
di
kamar
nya
.
```

### Lemmatization

Mengubah kata menjadi bentuk dasar yang digunakan untuk membantu normalisasi predikat pada pembentukan pertanyaan.

```text
merapikan → rapi
```

Dalam implementasi, dilakukan normalisasi tambahan untuk menghasilkan bentuk pertanyaan yang lebih natural.

### POS Tagging

Memberikan label kelas kata kepada setiap token.

Contoh:

```text
Rina        → PROPN
merapikan   → VERB
tempat      → NOUN
tidur       → NOUN
di          → ADP
kamar       → NOUN
```

### Dependency Parsing

Mengidentifikasi hubungan sintaksis antar kata dalam kalimat.

Contoh:

```text
Rina        → nsubj
merapikan   → root
tempat      → obj
di          → case
kamar       → nmod
```

Informasi ini kemudian digunakan sebagai dasar ekstraksi struktur kalimat pada sistem rule-based.

---

## ⚙️ Rule-Based Implementation

Sistem rule-based tidak hanya menggunakan hasil dependency parsing secara langsung, tetapi juga menggunakan beberapa aturan tambahan untuk menangani variasi hasil parsing.

Beberapa aturan yang diimplementasikan meliputi:

* Pemisahan objek dan keterangan berdasarkan preposisi.
* Penghapusan duplicate phrase.
* Normalisasi frasa keterangan.
* Penggabungan `advmod` dan `amod`.
* Penanganan klausa relatif.
* Fallback subject detection.
* Pemisahan keterangan waktu dari subjek.
* Penanganan predikat tambahan (`xcomp`).
* Normalisasi predikat menggunakan lemma.
* Penanganan objek dengan klausa relatif.
* Penanganan informasi numerik.

Pendekatan ini digunakan untuk meningkatkan konsistensi hasil ekstraksi sebelum pertanyaan dibentuk.

---

## 🤖 Machine Learning

Model utama yang digunakan:

**IndoBART — `indobenchmark/indobart`**

Model dilakukan fine-tuning menggunakan dataset pertanyaan hasil pendekatan rule-based yang telah melalui proses filtering berdasarkan semantic similarity.

Parameter utama pelatihan:

| Parameter             | Value |
| --------------------- | ----: |
| Epoch                 |     5 |
| Learning Rate         |  3e-5 |
| Batch Size            |     8 |
| Maximum Input Length  |   128 |
| Maximum Target Length |    64 |

Pelatihan dilakukan menggunakan **Hugging Face Trainer** pada GPU Google Colab.

---

## 📈 Evaluation

Evaluasi dilakukan menggunakan beberapa tahapan untuk melihat kualitas pertanyaan dari sisi sistem, ahli, dan pengguna akhir.

### Rule-Based

Dari **4.975 pertanyaan** yang dihasilkan:

* **4.631 pertanyaan valid**
* **344 pertanyaan tidak valid**
* **93,09% pertanyaan valid**

### Machine Learning

Pada pengujian menggunakan **25 kalimat baru** yang tidak digunakan dalam proses pelatihan, model menghasilkan **70 pertanyaan**:

* **68 pertanyaan valid**
* **2 pertanyaan tidak valid**
* **97,14% pertanyaan valid**

Kedua pendekatan diuji pada kondisi data yang berbeda. Evaluasi rule-based dilakukan terhadap seluruh hasil generate dari dataset penelitian, sedangkan machine learning dievaluasi menggunakan kalimat baru sebagai data uji.

---

## 👩‍🏫 Expert Validation

Validasi dilakukan oleh **3 guru sekolah dasar** menggunakan skala Likert 1–5.

Aspek penilaian meliputi:

1. Kejelasan pertanyaan.
2. Kesesuaian pertanyaan dengan teks.
3. Kesesuaian dengan kemampuan siswa SD kelas 3–6.

Hasil:

| Pendekatan       | Rata-rata | Standar Deviasi |
| ---------------- | --------: | --------------: |
| Rule-Based       |      4,49 |            0,03 |
| Machine Learning |      4,51 |            0,10 |

Hasil menunjukkan bahwa kedua pendekatan memperoleh penilaian yang baik dan relatif konsisten dari validator.

---

## 👧 Student Validation

Validasi pengguna dilakukan terhadap **16 siswa kelas 3 SD**.

Masing-masing siswa mengerjakan pertanyaan yang dihasilkan oleh kedua pendekatan.

Hasil:

| Pendekatan       | Jawaban Benar | Persentase |
| ---------------- | ------------: | ---------: |
| Rule-Based       |       254/256 |     99,22% |
| Machine Learning |       254/256 |     99,22% |

Hasil tersebut menunjukkan bahwa sebagian besar siswa mampu memahami dan menjawab pertanyaan yang dihasilkan oleh kedua pendekatan.

---

## 🔬 Rule-Based vs Machine Learning

Perbandingan hasil menunjukkan karakteristik yang berbeda pada kedua pendekatan.

### Rule-Based

**Kelebihan:**

* Struktur pertanyaan lebih terkontrol.
* Pola pertanyaan konsisten.
* Mudah ditelusuri berdasarkan aturan linguistik.
* Tidak membutuhkan dataset pelatihan dalam jumlah besar.

**Keterbatasan:**

* Sangat bergantung pada hasil dependency parsing.
* Membutuhkan aturan tambahan untuk menangani variasi struktur kalimat.
* Kurang fleksibel terhadap kalimat yang kompleks.
* Dapat menghasilkan pertanyaan yang tidak lengkap ketika struktur kalimat gagal diekstraksi.

### Machine Learning

**Kelebihan:**

* Lebih fleksibel terhadap variasi struktur kalimat.
* Menghasilkan pertanyaan yang lebih natural.
* Tidak bergantung langsung pada aturan sintaksis yang dibuat secara manual.
* Dapat mempelajari pola pertanyaan dari dataset.

**Keterbatasan:**

* Bergantung pada kualitas dataset pelatihan.
* Masih dapat menghasilkan pertanyaan yang kurang tepat.
* Dapat menghasilkan *hallucination* atau informasi yang tidak terdapat pada teks sumber.
* Kualitas menurun pada kalimat yang terlalu kompleks.

Contoh perbandingan:

| Kalimat                                                 | Rule-Based          | Machine Learning                             |
| ------------------------------------------------------- | ------------------- | -------------------------------------------- |
| Matahari mulai terbenam di barat.                       | Di mana terbenam?   | Di mana matahari mulai terbenam?             |
| Bayu menggunakan tusuk gigi untuk menggambar pola.      | Apa yang Bayu guna? | Apa yang Bayu gunakan untuk menggambar pola? |
| Bingkai kacamata itu sangat ringan dari bahan titanium. | Dari mana Bingkai?  | Dari mana bingkai kacamata itu?              |

---

## 🛠️ Tech Stack

**Programming Language**

* Python

**NLP**

* Stanza
* Natural Language Processing
* Tokenization
* Lemmatization
* POS Tagging
* Dependency Parsing

**Machine Learning**

* Hugging Face Transformers
* IndoBART
* mT5-small
* Fine-tuning
* Text-to-text generation

**Data Processing**

* Pandas
* NumPy
* CSV
* TXT

**Development Environment**

* Google Colab
* GPU

**Evaluation**

* Cosine Similarity
* Semantic Embedding
* Likert Scale
* Manual Expert Validation
* Student Validation

---

## 📂 Project Structure

```text
Automatic-Question-Generation/
│
├── dataset/
│   ├── raw/
│   ├── processed/
│   └── question_dataset.csv
│
├── preprocessing/
│   └── preprocessing_stanza.ipynb
│
├── rule_based/
│   ├── question_generator.py
│   └── evaluation.py
│
├── machine_learning/
│   ├── train.py
│   ├── generate.py
│   └── evaluation.py
│
├── notebooks/
│   ├── preprocessing.ipynb
│   ├── rule_based.ipynb
│   └── machine_learning.ipynb
│
├── results/
│   ├── rule_based_results.csv
│   └── machine_learning_results.csv
│
└── README.md
```

> Struktur folder dapat disesuaikan dengan struktur repository sebenarnya.

---

## 🚀 Workflow

Secara keseluruhan, workflow penelitian dapat diringkas sebagai berikut:

```text
Dataset Collection
        ↓
Data Preprocessing
        ↓
NLP Analysis
        ↓
Rule-Based Question Generation
        ↓
Semantic Similarity Evaluation
        ↓
Question Filtering
        ↓
Machine Learning Dataset
        ↓
IndoBART Fine-Tuning
        ↓
Machine Learning Question Generation
        ↓
Expert Validation
        ↓
Student Validation
        ↓
Performance Analysis
```

---

## 💡 Key Findings

Beberapa temuan utama dari proyek ini:

* Rule-based mampu menghasilkan pertanyaan literal secara konsisten berdasarkan struktur linguistik.
* Penambahan aturan manual diperlukan untuk menangani variasi hasil dependency parsing.
* Machine learning menggunakan IndoBART menghasilkan persentase validitas yang lebih tinggi pada data uji yang digunakan.
* IndoBART mampu menghasilkan pertanyaan yang lebih fleksibel dan natural dibandingkan beberapa hasil rule-based.
* Kedua pendekatan memperoleh **99,22% jawaban benar** pada validasi siswa.
* Sistem bekerja lebih baik pada **kalimat sederhana dengan struktur yang jelas**.
* Kalimat dengan struktur sintaksis kompleks masih menjadi keterbatasan sistem.

---

## 🎓 Project Context

**Project Type:** Undergraduate Thesis / Final Project
**Field:** Natural Language Processing, Artificial Intelligence, Machine Learning
**Application:** Reading Comprehension & Education Technology
**Language:** Bahasa Indonesia

**Main Research Topic:**

> Automatic Question Generation untuk Mengukur Pemahaman Bacaan bagi Siswa Sekolah Dasar.

---

## 👨‍💻 My Contribution

Dalam proyek ini, saya mengerjakan proses penelitian dan pengembangan sistem secara menyeluruh, meliputi:

* Pengumpulan dan seleksi dataset.
* Perancangan struktur dataset.
* Implementasi preprocessing NLP menggunakan Stanza.
* Implementasi dependency parsing.
* Pengembangan aturan linguistik untuk sistem rule-based.
* Pengembangan sistem pembentukan pertanyaan literal.
* Evaluasi menggunakan semantic similarity.
* Pembentukan dan filtering dataset machine learning.
* Fine-tuning model IndoBART.
* Implementasi proses generate pertanyaan.
* Evaluasi dan analisis hasil kedua pendekatan.
* Pelaksanaan validasi guru dan siswa.
* Analisis hasil penelitian.

---

## 📌 Limitations & Future Development

Sistem saat ini difokuskan pada **teks dan kalimat sederhana** yang sesuai dengan kebutuhan pertanyaan literal siswa sekolah dasar.

Pengembangan selanjutnya dapat diarahkan pada:

* Penanganan kalimat majemuk dan struktur sintaksis kompleks.
* Peningkatan kemampuan model dalam mempertahankan informasi literal.
* Pengembangan dataset yang lebih besar dan beragam.
* Penambahan variasi tipe pertanyaan.
* Pengembangan antarmuka pengguna untuk penggunaan oleh guru.
* Evaluasi pada jenjang dan karakteristik siswa yang lebih beragam.

---

## 📎 Repository

Source code, dataset yang dapat dibagikan, notebook, dan hasil eksperimen tersedia pada repository ini.

> **Note:** Dataset atau dokumen tertentu yang berasal dari sumber eksternal tidak seluruhnya disertakan dalam repository karena pertimbangan hak cipta dan penggunaan data penelitian.

---

## 📄 Publication / Thesis

This project is based on my undergraduate thesis in **Informatics Engineering**.

**Title:**
**GENERATE PERTANYAAN UNTUK MENGUKUR PEMAHAMAN BACAAN BAGI SISWA SEKOLAH DASAR**

---

⭐ *This project demonstrates the application of Natural Language Processing, rule-based systems, semantic similarity, and Transformer-based question generation for educational technology.*
