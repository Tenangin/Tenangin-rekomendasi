# Sistem Rekomendasi Klinik Kesehatan Mental untuk Indonesia

Proyek ini menyediakan sistem rekomendasi untuk klinik kesehatan mental di seluruh Indonesia. Sistem ini memanfaatkan kumpulan data klinik yang mencakup informasi seperti nama, kategori, alamat, situs web, nomor telepon, rating, jumlah ulasan, garis bujur, garis lintang, dan provinsi. Sistem ini dirancang untuk membantu pengguna menemukan klinik yang sesuai berdasarkan lokasi mereka dan karakteristik klinik.

## Daftar Isi

- [Gambaran Umum Proyek](#gambaran-umum-proyek)
- [Sumber Data](#sumber-data)
- [Pra-pemrosesan Data](#pra-pemrosesan-data)
- [Analisis Data Eksplorasi (EDA)](#analisis-data-eksplorasi-eda)
- [Model Rekomendasi](#model-rekomendasi)
- [Cara Menggunakan (Pengaturan Lokal)](#cara-menggunakan-pengaturan-lokal)
- [Inferensi](#Inferensi)

## Gambaran Umum Proyek

Tujuan dari proyek ini adalah membangun sistem rekomendasi yang membantu individu di Indonesia menemukan klinik kesehatan mental. Sistem ini mempertimbangkan kedekatan geografis (garis lintang dan garis bujur) serta rating dan jumlah ulasan klinik untuk memberikan rekomendasi yang dipersonalisasi.

## Sumber Data

Data inti untuk proyek ini diambil dari gmaps dapat diakses pada tautan https://github.com/Xinqi04/Dataset-Klinik-Psikologi-Di-Indonesia. Data ini kemudian digabungkan dan diproses.

## Pra-pemrosesan Data

*Notebook* `Data_Preposesing.ipynb` melakukan langkah-langkah berikut:

1.  **Penggabungan Data**: Membaca beberapa file CSV (satu untuk setiap provinsi) dari folder yang ditentukan, mengekstrak nama provinsi dari nama file, menambahkannya sebagai kolom baru, dan kemudian menggabungkan semua *dataframe* menjadi satu *dataframe* utama (`df_gabungan`).
2.  **Pemeriksaan Duplikat**: Memverifikasi baris duplikat dalam kumpulan data gabungan. Tidak ada baris duplikat yang ditemukan.
3.  **Penanganan Nilai yang Hilang**:
    * Mengidentifikasi nilai yang hilang di setiap kolom.
    * Menghapus baris di mana 'longitude' ATAU 'latitude' hilang.
    * Menghapus baris di mana 'rating' hilang.
    * Mengisi nilai `NaN` yang tersisa dengan "tidak diketahui" di kolom 'website' dan 'phone_number'.
4.  **Pemfilteran Kategori**: Memfilter data untuk hanya menyertakan kategori klinik kesehatan mental yang relevan (misalnya, 'Psychologist', 'Psychotherapist', 'Mental health clinic', 'Psychiatrist', 'Mental health service', 'Health consultant', 'Family counselor', 'Health counselor', 'Psychopedagogy clinic').
5.  **Pembersihan Data**:
    * Mengonversi kolom 'addres_full' menjadi huruf kecil.
    * Mencoba mengekstrak nama provinsi dari 'addres_full' jika kolom 'provinsi' hilang atau tidak akurat, menggunakan kamus kata kunci yang telah ditentukan.
    * Menghapus nama klinik duplikat tertentu dan menghapus entri yang tidak relevan (misalnya, "Tes Psikologi SIM").
    * Mengonversi kolom 'rating' dan 'review_count' ke tipe numerik.
6.  **Ekspor Data**: Menyimpan data yang telah dibersihkan dan dipra-proses ke file CSV baru: `data_klinik_psikologi_Indonesia.csv` dan `data_final.csv`.

## Analisis Data Eksplorasi (EDA)

*Notebook* `EDA.ipynb` melakukan analisis mendalam terhadap data yang telah diproses, termasuk:

1.  **Informasi Umum Data**:
    * Menampilkan `df.info()` dan `df.describe(include='all')` untuk memahami tipe data, jumlah non-null, dan statistik ringkasan.
    * Menunjukkan nilai unik untuk setiap kolom menggunakan `df.nunique()`.
2.  **Distribusi Klinik per Provinsi**:
    * Memvisualisasikan jumlah klinik di setiap provinsi menggunakan grafik batang horizontal. Jawa Timur memiliki jumlah klinik tertinggi (161), diikuti oleh Jawa Tengah (159) dan Jawa Barat (126).
3.  **Distribusi Kategori Klinik**:
    * Mengidentifikasi dan mencantumkan kategori unik.
    * Memvisualisasikan distribusi kategori klinik yang relevan menggunakan grafik batang dan *pie chart*. 'Psychologist' adalah kategori paling populer.
4.  **Distribusi Rating dan Jumlah Ulasan**:
    * Menggunakan histogram dengan KDE untuk menunjukkan distribusi 'rating' dan 'review_count'.
    * Mengidentifikasi *outlier* pada 'rating' dan 'review_count' menggunakan *boxplot*.
5.  **Rata-rata Rating dan Ulasan per Provinsi**:
    * Menghitung dan memvisualisasikan rata-rata rating dan jumlah ulasan untuk klinik di setiap provinsi menggunakan grafik batang.
6.  **Distribusi Geografis**:
    * Mengonversi 'latitude' dan 'longitude' ke float.
    * Membuat peta Folium interaktif (`persebaran_klinik.html`) untuk memvisualisasikan persebaran klinik di seluruh Indonesia menggunakan *marker cluster*.
7.  **Ekspor Data Akhir**: Menyimpan data yang telah disempurnakan ke `data_final.csv`.

## Model Rekomendasi

*Notebook* `model_rekomendasi_final.ipynb` mengimplementasikan sistem rekomendasi:

1.  **Pemuatan Data**: Memuat data yang telah diproses dari `data_psikologi_with_uuid.csv`.
2.  **Rekayasa Fitur**:
    * Mengekstrak 'latitude', 'longitude', 'rating', dan 'review_count' sebagai fitur.
    * Menerapkan `MinMaxScaler` untuk menormalisasi fitur-fitur ini.
3.  **Kombinasi Fitur Berbobot**:
    * Menggabungkan fitur lokasi yang diskalakan dan fitur rating/ulasan yang diskalakan.
    * Menerapkan bobot pada fitur (misalnya, 40% untuk garis lintang, 40% untuk garis bujur, 10% untuk rating, 10% untuk jumlah ulasan) untuk membuat array `weighted_features`.
4.  **Fungsi Rekomendasi**:
    * Mendefinisikan fungsi `recommend_places` yang mengambil garis lintang dan garis bujur pengguna, *dataframe*, dan fitur-fitur lain yang diskalakan sebagai masukan.
    * Mensimulasikan preferensi pengguna yang "ideal" dengan mengatur rating mereka ke rating maksimum dalam kumpulan data dan jumlah ulasan mereka ke rata-rata jumlah ulasan. Ini mendorong rekomendasi untuk klinik dengan rating tinggi dan populer.
    * Menghitung kemiripan kosinus antara vektor berbobot pengguna dan semua vektor fitur berbobot klinik.
    * Mengembalikan N klinik teratas yang paling mirip.
5.  **Contoh Penggunaan**:
    * Menunjukkan cara menggunakan fungsi `recommend_places` dengan contoh lokasi pengguna.
    * Mencetak klinik yang direkomendasikan teratas dalam *DataFrame* pandas.
    * Memvisualisasikan lokasi pengguna dan klinik yang direkomendasikan pada peta Folium interaktif.

## Cara Menggunakan (Pengaturan Lokal)

Untuk menjalankan proyek ini secara lokal, ikuti langkah-langkah berikut:

1.  **Kloning Repositori (jika berlaku)**:
    ```bash
    git clone <repository_url>
    cd <repository_name>
    ```

2.  **Buat Lingkungan Virtual (Direkomendasikan)**:
    ```bash
    python -m venv .venv
    ```

3.  **Aktifkan Lingkungan Virtual**:
    * Di Windows:
        ```bash
        .venv\Scripts\activate
        ```
    * Di macOS/Linux:
        ```bash
        source .venv/bin/activate
        ```

4.  **Instal Dependensi**:
    ```bash
    pip install pandas numpy matplotlib seaborn folium scikit-learn
    ```

5.  **Jalankan Jupyter Notebooks**:
    * Buka `Data_Preposesing.ipynb` dan jalankan semua sel. Ini akan menghasilkan file data yang telah dibersihkan.
    * Buka `EDA.ipynb` dan jalankan semua sel untuk menjelajahi data.
    * Buka `tambahan_id.ipynb` dan jalankan semua sel untuk menambahkan UUID ke data.
    * Buka `model_rekomendasi_final.ipynb` dan jalankan semua sel untuk melihat sistem rekomendasi beraksi. Anda dapat memodifikasi kamus `user_input` di sel terakhir untuk menguji lokasi yang berbeda.
  
## Inferensi
Dalam struktur folder terdapat file bernama inferensi_rekomendasi.html, bisa diunduh dan run di lokal host.
  

*note perhatikan kembali struktur folder dan sesuaikan 
