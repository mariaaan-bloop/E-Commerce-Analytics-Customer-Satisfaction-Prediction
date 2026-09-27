# Big Data Architecture for E-Commerce Analytics & Customer Satisfaction Prediction
### (Brazilian Olist Dataset)

## 📌 Overview

Project ini membangun sebuah **arsitektur Big Data end-to-end** untuk mengolah data transaksi e-commerce berskala besar dari **Brazilian E-Commerce Public Dataset by Olist**, lalu menghasilkan **model machine learning prediksi kepuasan pelanggan (customer satisfaction)** serta **dashboard analitik interaktif**.

Fokus utama project ini adalah penerapan **teknologi Big Data**, bukan sekadar analisis data biasa — dataset diproses menggunakan **Hadoop Distributed File System (HDFS)** sebagai storage layer dan **Apache Spark (PySpark)** sebagai processing engine, dijalankan di atas **Cloudera VM environment**.

## 🗂️ Big Data Characteristics (5V)

Dataset Olist memenuhi kelima karakteristik Big Data yang menjadi alasan utama penggunaan Hadoop ecosystem:

| Karakteristik | Penjelasan |
|---|---|
| **Volume** | ±100.000 orders, ±110.000 order items, ±99.000 customers, tersebar di 9 tabel dengan jutaan baris data |
| **Velocity** | Data transaksi historis yang terus bertambah secara periodik (batch) |
| **Variety** | Kombinasi data structured (tabel transaksi), semi-structured, hingga unstructured (teks review pelanggan) |
| **Veracity** | Banyak missing value, duplikasi, outlier finansial, dan anomali waktu yang perlu divalidasi |
| **Value** | Insight untuk strategi logistik, retensi pelanggan, dan performa seller |

## 🏗️ Arsitektur Big Data

Project ini menggunakan **layered batch architecture** yang berpusat pada ekosistem Hadoop:

```
Data Sources (CSV) 
      ↓
Ingestion Layer (Batch ETL)
      ↓
Storage Layer → HDFS (Hadoop Distributed File System)
      ↓
Processing & Analytics Engine → Apache Spark (PySpark)
      ↓
Machine Learning → CatBoost & Random Forest
      ↓
Visualization Layer → Power BI Dashboard
```

**Alasan Menggunakan Hadoop (HDFS)**
- **Horizontal scalability** — bisa menambah node komoditas alih-alih upgrade satu server besar
- **Fault-tolerant** — data direplikasi otomatis ke banyak server
- **High throughput untuk batch processing** — cocok untuk data historis berskala besar seperti Olist
- Seluruh dataset mentah (9 file CSV) disimpan dan dibaca langsung dari HDFS melalui path `hdfs://localhost:8020/user/cloudera/olist/raw/`, dan hasil akhir data yang sudah diproses juga disimpan kembali ke HDFS (`.../olist/processed`) dalam format Parquet.

Apache Spark dipilih sebagai processing engine karena berjalan **di atas Hadoop** dan mampu melakukan distributed processing terhadap data yang tersimpan di HDFS, jauh lebih cepat dibanding pemrosesan sekuensial pada RDBMS tradisional.

## ⚙️ Tech Stack

- **Storage:** Hadoop Distributed File System (HDFS) — Cloudera VM
- **Processing:** Apache Spark (PySpark)
- **Machine Learning:** CatBoost, Random Forest (scikit-learn), ADASYN
- **Visualization:** Power BI
- **File Format:** Parquet (output akhir untuk efisiensi storage & ML)

## 🔄 Alur Data Processing

1. **Data Load** — 9 dataset mentah (orders, customers, order items, reviews, payments, products, geolocation, sellers, product category translation) dibaca dari HDFS menggunakan PySpark
2. **Data Cleaning** — filtering status order, handling missing value (drop & imputasi), penghapusan duplikasi, serta anonymization teks review (masking nama samaran)
3. **Data Transformation** — casting timestamp, standardisasi teks kategorikal, casting numerik
4. **Outlier Handling** — deteksi & capping outlier finansial menggunakan metode IQR
5. **Aggregation & Feature Engineering** — agregasi per `order_id`, pembuatan fitur seperti `delivery_time`, `is_late`, `total_payment_value`, `total_items`, dan label target `is_satisfied`
6. **Data Integration** — join seluruh tabel menjadi satu master table
7. **Load ke HDFS** — hasil akhir disimpan kembali ke HDFS dalam format Parquet

## 🤖 Machine Learning

Target prediksi: **`is_satisfied`** (klasifikasi biner — puas / tidak puas, berdasarkan `review_score`)

- **Model utama:** CatBoost Classifier (akurasi ±81.98%)
- **Model pembanding:** Random Forest + ADASYN oversampling
- Evaluasi menggunakan **10-Fold Stratified Cross Validation**
- Insight utama: **keterlambatan pengiriman** dan **durasi pengiriman** adalah faktor paling berpengaruh terhadap kepuasan pelanggan — lebih dominan dibanding harga atau metode pembayaran

## 📊 Dashboard

Dashboard Power BI (`dasboard.pbix`) terdiri dari 3 halaman:
1. Executive Summary — KPI kepuasan pelanggan, rata-rata waktu pengiriman, total order
2. Delivery & Late Delivery Analysis — hubungan keterlambatan dengan kepuasan pelanggan
3. Regional & Order Complexity Analysis — peta sebaran kepuasan per wilayah & pengaruh jumlah item per order

## 📁 Struktur Folder

```
BDP_Group 3/
├── DATASET/                        # Dataset mentah (CSV)
├── DATA PROCESSING/
│   ├── Code Data Processing.ipynb  # ETL dengan PySpark + HDFS
│   └── PROCESSING OUTPUT/
│       └── df_clean.parquet
├── MODEL ML/
│   ├── Model_Utama_CatBoost.ipynb
│   ├── Model Pembanding_Random_Forest.ipynb
│   └── df_master_processed_single.parquet
├── dasboard.pbix                   # Dashboard Power BI
├── Project Report.pdf
└── Presentation.pdf
```

## 🚀 Future Improvements

- Integrasi **real-time streaming** dengan Apache Kafka + Spark Structured Streaming
- Migrasi ke **Multi-Node Hadoop Cluster** untuk scalability dan fault tolerance yang lebih baik
- Eksplorasi model lanjutan (SVM, XGBoost tuning) untuk mengatasi class imbalance


