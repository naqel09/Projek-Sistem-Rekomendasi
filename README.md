# Laporan Proyek Machine Learning - Andry Septian Syahputra Tumaruk
---
## Project Overview
Dengan semakin banyaknya pilihan film yang tersedia bagi pengguna melalui berbagai platform streaming dan database, menemukan film yang sesuai dengan preferensi individu bisa menjadi tugas yang menantang. Pengguna seringkali dihadapkan pada ribuan judul tanpa panduan yang jelas, yang dapat menyebabkan kelelahan dalam memilih (choice fatigue) dan pengalaman pengguna yang kurang memuaskan. Sistem rekomendasi film hadir untuk mengatasi masalah ini dengan menyarankan film yang kemungkinan besar akan disukai pengguna.

Proyek ini bertujuan untuk membangun sistem rekomendasi film menggunakan pendekatan content-based filtering. Masalah utama yang ingin diselesaikan adalah bagaimana memberikan rekomendasi film yang relevan kepada pengguna berdasarkan konten atau atribut dari film itu sendiri, seperti alur cerita (overview), genre, kata kunci, aktor, dan sutradara. Dengan menganalisis fitur-fitur ini, sistem dapat mengidentifikasi dan merekomendasikan film-film yang memiliki kemiripan konten dengan film yang telah diketahui atau disukai pengguna. Dataset yang digunakan adalah ["TMDB Movie Metadata"](https://www.kaggle.com/datasets/tmdb/tmdb-movie-metadata) dari Kaggle, yang menyediakan informasi metadata kaya untuk ribuan film.

---
## Business Understanding
### Problem Statements**

 1. Bagaimana cara memberikan rekomendasi film yang relevan kepada pengguna berdasarkan atribut intrinsik (konten) dari film tersebut?
 2. Bagaimana memanfaatkan fitur-fitur film seperti `overview`, `genres`, `keywords`, `cast`, dan `crew` untuk mengukur kesamaan antar film dan menghasilkan daftar rekomendasi yang dipersonalisasi?
 3. Bagaimana menangani variasi dalam input pengguna, seperti judul film yang mungkin tidak diketik dengan ejaan yang sempurna atau lengkap, agar tetap dapat memberikan rekomendasi yang berguna?

### Goals
 1. Mengembangkan sebuah sistem rekomendasi film yang mampu menyarankan film-film berdasarkan kemiripan konten dengan film yang diberikan sebagai input.
 2. Mengekstraksi dan memproses fitur-fitur tekstual dan kategorikal dari dataset film (overview, genre, kata kunci, pemeran utama, dan sutradara) untuk membuat representasi konten yang komprehensif untuk setiap film.
 3. Mengimplementasikan mekanisme pencarian judul film yang fleksibel untuk mengakomodasi input pengguna yang tidak persis sama dengan judul di database dan tetap memberikan rekomendasi yang relevan.

### Solution Approach
 - **Solution Statement:** Mengimplementasikan sistem rekomendasi berbasis konten (content-based filtering) dengan langkah-langkah sebagai berikut:
    1. Penggabungan data film dan data kredit (pemeran dan kru).
    2. Pra-pemrosesan data untuk membersihkan dan mengubah fitur-fitur relevan (overview, genres, keywords, cast, crew) menjadi format yang seragam dan dapat diolah. Ini termasuk parsing data JSON, ekstraksi nama aktor dan sutradara, serta normalisasi teks (menghilangkan spasi dan mengubah ke huruf kecil).
    3. Pembuatan sebuah "tag" gabungan untuk setiap film yang merepresentasikan kontennya secara keseluruhan.
    4. Transformasi teks "tag" menjadi representasi numerik menggunakan TF-IDF (Term Frequency-Inverse Document Frequency) Vectorizer.
    5. Penghitungan skor kesamaan antar film menggunakan metrik Cosine Similarity berdasarkan vektor TF-IDF mereka.
    6. Pengembangan fungsi yang dapat menerima judul film sebagai input, menemukan film yang paling mirip berdasarkan skor kesamaan kosinus, dan mengembalikan N film teratas sebagai rekomendasi.

---
## Data Understanding
Dataset yang digunakan dalam proyek ini yang bersumber dari Kaggle adalah ["TMDB Movie Metadata"](https://www.kaggle.com/datasets/tmdb/tmdb-movie-metadata). Dataset ini terdiri dari dua file CSV utama:

1. `tmdb_5000_movies.csv`: Berisi informasi detail tentang sekitar 5000 film.
2. `tmdb_5000_credits.csv`: Berisi informasi mengenai pemeran dan kru untuk film-film tersebut.

### Informasi Awal Dataset:
 - Jumlah data unik pada `movies_df` (berdasarkan `id`): 4803.
 - Jumlah data unik pada `credits_df` (berdasarkan `movie_id`): 4803.
 - Setelah penggabungan, dataset awal memiliki 4803 baris.
 - Beberapa kolom memiliki nilai yang hilang (missing values), seperti `homepage`, `tagline`, dan yang paling relevan untuk proyek ini, `overview` (3 nilai hilang). Kolom `release_date` dan `runtime` juga memiliki beberapa nilai hilang namun tidak secara langsung digunakan dalam pembuatan fitur 'tags'.
 - Tipe data didominasi oleh objek (string), integer, dan float.
   
### Variabel (Fitur) yang Digunakan untuk Model Rekomendasi:
Setelah pra-pemrosesan dan seleksi fitur, variabel-variabel inti yang digunakan untuk membangun sistem rekomendasi adalah:
 - `movie_id`: Identifier unik untuk setiap film.
 - `title`: Judul film.
 - `overview`: Ringkasan singkat atau sinopsis film.
 - `genres`: Genre film (misalnya, Action, Adventure, Fantasy). Awalnya dalam format JSON string.
 - `keywords`: Kata kunci yang terkait dengan film (misalnya, culture clash, ocean, spy). Awalnya dalam format JSON string.
 - `cast`: Daftar pemeran film. Dipilih 3 aktor utama. Awalnya dalam format JSON string.
 - `crew`: Daftar kru film. Informasi sutradara diekstrak dari sini. Awalnya dalam format JSON string.

### Exploratory Data Analysis (EDA):
 - Dataset `movies_df` memiliki 20 kolom, sedangkan `credits_df` memiliki 4 kolom.
 - Status film yang ada dalam dataset adalah 'Released', 'Post Production', dan 'Rumored'.
 - Terdapat 1609 nilai unik untuk `vote_count` pada `movies_df`.
 - Kolom `production_companies` dan `production_countries` juga dalam format JSON string dan menunjukkan keragaman sumber produksi film.
 - Kolom `cast` di `credits_df` memiliki 4761 nilai unik, dan `crew` memiliki 4776 nilai unik, mengindikasikan beberapa aktor dan kru terlibat di lebih dari satu film.

---
## Data Preparation
Tahapan persiapan data dilakukan untuk mengubah data mentah menjadi format yang sesuai untuk pemodelan sistem rekomendasi berbasis konten. Langkah-langkah yang dilakukan adalah sebagai berikut:

1. **Penggabungan DataFrames:**
    - DataFrame `movies_df` dan `credits_df` digabungkan menjadi satu DataFrame `df` berdasarkan kolom `movie_id` (setelah kolom `id` di `movies_df` diubah namanya menjadi `movie_id` untuk konsistensi). Ini menyatukan informasi detail film dengan informasi pemeran dan kru.

2. **Seleksi Fitur Relevan:**
    - Kolom-kolom yang dianggap paling relevan untuk content-based filtering dipilih: `movie_id`, `title_x` (yang kemudian diubah namanya menjadi `title`), `overview`, `genres`, `keywords`, `cast`, dan `crew`.

3. **Penanganan Nilai Hilang (Missing Values):**
    - Baris data yang memiliki nilai hilang pada kolom `overview` dihapus menggunakan `dropna()`. Kolom `overview` penting karena berisi deskripsi tekstual film. Jumlah film setelah penghapusan NaN adalah 4800.

4. **Feature Engineering:**
    - **Parsing Kolom JSON-like:**
        - Fungsi `parse_json_like_string(text_data)` dibuat untuk mengurai kolom `genres` dan `keywords` yang awalnya berupa string JSON. Fungsi ini menggunakan `ast.literal_eval()` untuk mengubah string menjadi list of dictionaries, lalu mengekstrak nilai dari kunci 'name' (misalnya, nama genre atau kata kunci).
    - **Ekstraksi Aktor Utama:**
        - Fungsi `get_top_n_actors(text_data, n=3)` dibuat untuk mengurai kolom `cast`, mengekstrak nama dari 3 aktor utama pertama.
    - **Ekstraksi Sutradara:**
        - Fungsi `get_director(text_data)` dibuat untuk mengurai kolom `crew` dan mengekstrak nama anggota kru yang memiliki `job` sebagai 'Director'.
    - **Pembersihan Spasi dan Konversi ke Huruf Kecil:**
        - Fungsi `clean_spaces(item_list)` diterapkan pada hasil parsing `genres`, `keywords`, `cast`, dan `crew`. Fungsi ini menghilangkan spasi antar kata dalam satu item (misalnya, "Science Fiction" menjadi "sciencefiction") dan mengubah semua teks menjadi huruf kecil. Ini bertujuan agar item seperti "Tom Hanks" dan "tom hanks" diperlakukan sama dan "Science Fiction" tidak dianggap dua kata terpisah oleh vectorizer.

    - **Transformasi Kolom `overview:`**
        - Kolom `overview` diubah dari string menjadi list kata-kata dengan membaginya berdasarkan spasi (str(x).split()).
    - **Pembuatan Kolom 'tags':**
        - Sebuah kolom baru bernama `tags` dibuat dengan menggabungkan konten dari kolom `overview` (yang sudah di-split), `genres`, `keywords`, `cast`, dan `crew` (yang sudah diproses). Ini menciptakan satu representasi tekstual gabungan yang kaya untuk setiap film.
    - **Finalisasi Kolom 'tags':**
        - List kata-kata dalam kolom `tags` digabungkan kembali menjadi satu string tunggal per film, dengan semua kata dalam huruf kecil dan dipisahkan oleh spasi (`" ".join(x).lower()`).

    - **Pembuatan DataFrame Final:**
        - DataFrame baru `final_df` dibuat yang hanya berisi kolom `movie_id`, `title`, dan `tags` yang sudah diproses. DataFrame ini menjadi dasar untuk tahap pemodelan.
          
Alasan dilakukannya tahapan persiapan data ini adalah untuk memastikan bahwa fitur-fitur yang digunakan bersih, konsisten, dan dalam format yang dapat diproses oleh algoritma machine learning (khususnya text vectorizer). Penggabungan fitur ke dalam satu kolom 'tags' menyederhanakan proses ekstraksi fitur numerik dan memungkinkan model untuk menangkap esensi konten film secara holistik.

---
## Modeling
Model sistem rekomendasi yang dikembangkan menggunakan pendekatan Content-Based Filtering. Pendekatan ini merekomendasikan item berdasarkan perbandingan antara konten item dengan profil pengguna atau item lain yang telah dilihat/disukai pengguna. Dalam kasus ini, rekomendasi diberikan berdasarkan kemiripan konten antar film.

### Tahapan Pemodelan:
 1. **Ekstraksi Fitur dengan TF-IDF Vectorizer:**
    - Data yang digunakan adalah `final_df` yang berisi kolom `tags` untuk setiap film.
    - `TfidfVectorizer` dari library `sklearn.feature_extraction.text` digunakan untuk mengubah kumpulan teks dalam kolom 'tags' menjadi matriks representasi numerik TF-IDF.
    - Parameter yang digunakan pada `TfidfVectorizer`:
      - `max_features=5000`: Membatasi jumlah fitur (kata unik) yang dipertimbangkan menjadi 5000 kata yang paling sering muncul di seluruh dataset. Ini membantu mengurangi dimensi data dan fokus pada kata-kata yang lebih signifikan.
      - `stop_words='english'`: Menghilangkan kata-kata umum dalam bahasa Inggris (misalnya, "the", "is", "in") yang tidak banyak memberikan makna diskriminatif terhadap konten film.
    - Proses `fit_transform()` pada `tfidf_vectorizer` diterapkan ke kolom `data['tags']` untuk mempelajari kosakata dan menghasilkan `tfidf_matrix`. Ukuran tfidf_matrix yang dihasilkan adalah (4800, 5000), yang berarti ada 4800 film dan 5000 fitur kata unik.

2. **Perhitungan Kesamaan Kosinus (Cosine Similarity):**
    - Setelah mendapatkan representasi vektor TF-IDF untuk setiap film, langkah selanjutnya adalah menghitung kesamaan antar film.
`cosine_similarity` dari `sklearn.metrics.pairwise` digunakan untuk menghitung kesamaan kosinus antara semua pasangan vektor film dalam `tfidf_matrix`.
    - Hasilnya adalah `cosine_sim`, sebuah matriks (4800x4800) di mana setiap elemen (`i, j`) merepresentasikan skor kesamaan kosinus antara film ke-i dan film ke-j. Skor berkisar antara 0 (tidak mirip) hingga 1 (sangat mirip).

3. **Pembuatan Fungsi Rekomendasi:**
    - Sebuah fungsi `get_recommendations(title, cosine_sim_matrix, df, movie_indices, top_n=10)` didefinisikan untuk menghasilkan rekomendasi.
    - Langkah-langkah dalam fungsi:
        1. Pencarian Indeks Film: Mencari indeks film berdasarkan `title` yang diberikan. Terdapat penanganan jika judul tidak ditemukan persis, yaitu dengan mencari judul yang mengandung string input (case-insensitive). Jika ditemukan beberapa judul yang mirip, judul pertama yang cocok akan digunakan.
        2. Pengambilan Skor Similaritas: Mendapatkan baris skor similaritas dari `cosine_sim_matrix` yang sesuai dengan film input.
        3. Pengurutan Skor: Mengurutkan film berdasarkan skor similaritas secara menurun.
        4. Pemilihan Top-N Film: Memilih `top_n` film dengan skor tertinggi (tidak termasuk film input itu sendiri).
        5. Pengembalian Rekomendasi: Mengembalikan DataFrame yang berisi judul film rekomendasi dan skor similaritasnya.

**Output Model:**

 - Untuk film 'Avatar', 10 film teratas yang direkomendasikan antara lain 'Falcon Rising' (skor 0.204), 'Battle: Los Angeles' (skor 0.194), dan 'Apollo 18' (skor 0.184).
 - Untuk film 'The Dark Knight Rises', rekomendasinya adalah 'The Dark Knight' (skor 0.445), 'Batman Returns' (skor 0.390), dan 'Batman Begins' (skor 0.331).
 - Untuk pencarian 'batman' (judul tidak persis), sistem menemukan beberapa judul yang mirip dan menggunakan 'Batman v Superman: Dawn of Justice' sebagai basis, kemudian merekomendasikan film seperti 'Batman Begins' (skor 0.278) dan 'Man of Steel' (skor 0.272).
    
**Kelebihan Pendekatan Content-Based Filtering:**

- Dapat merekomendasikan item yang sangat spesifik atau niche kepada pengguna.
- Tidak memerlukan data dari pengguna lain (mengatasi masalah user cold-start تا حدی).
- Rekomendasi bersifat transparan karena dapat dijelaskan berdasarkan fitur item.

**Kekurangan Pendekatan Content-Based Filtering:**

 - Terbatas pada fitur item yang ada; jika fitur tidak cukup deskriptif, kualitas rekomendasi bisa buruk.
 - Keterbatasan dalam menemukan item yang benar-benar baru atau berbeda dari profil pengguna (limited serendipity), cenderung merekomendasikan item yang mirip dengan apa yang sudah disukai.
 - Membutuhkan feature engineering yang baik.
 - Mengalami masalah item cold-start: item baru tanpa fitur yang cukup tidak dapat direkomendasikan.

---
## Evaluation
evaluasi sistem rekomendasi dilakukan secara kualitatif dengan mengamati relevansi dari top-N film yang direkomendasikan untuk beberapa judul film input. Tidak ada metrik evaluasi kuantitatif formal seperti presisi, recall, F1-score, MAP (Mean Average Precision), atau NDCG (Normalized Discounted Cumulative Gain) yang dihitung.

### Hasil Proyek Berdasarkan Observasi Kualitatif:
 - **Relevansi Rekomendasi:** Berdasarkan contoh-contoh output dalam notebook:
    - Untuk film 'Avatar', yang merupakan film fiksi ilmiah dengan tema luar angkasa dan pertempuran, beberapa rekomendasi seperti 'Battle: Los Angeles' (invasi alien), 'Star Trek Into Darkness' (luar angkasa), dan 'Aliens' tampak cukup relevan dari segi genre atau tema.
    - Untuk film 'The Dark Knight Rises', sebagian besar rekomendasi adalah film-film Batman lainnya atau film superhero DC, yang sangat relevan ('The Dark Knight', 'Batman Begins', 'Batman Returns', 'Batman v Superman: Dawn of Justice'). Ini menunjukkan kemampuan model untuk menangkap kesamaan berdasarkan franchise dan karakter utama.
    - Fungsi pencarian judul yang fleksibel juga berhasil menangani input 'batman' dengan menyarankan film-film Batman yang relevan, meskipun inputnya tidak spesifik.
 - **Skor Similaritas**: Skor similaritas yang menyertai rekomendasi memberikan indikasi seberapa mirip konten film yang direkomendasikan dengan film input. Misalnya, 'The Dark Knight' memiliki skor similaritas 0.445 dengan 'The Dark Knight Rises', menunjukkan kemiripan yang tinggi.

### Diskusi Metrik Evaluasi (Jika Ingin Digunakan):
Meskipun tidak diimplementasikan, jika evaluasi kuantitatif diinginkan untuk sistem content-based filtering seperti ini, beberapa pendekatan bisa dipertimbangkan:
- **Presisi@k:** Proporsi film yang relevan dari k film teratas yang direkomendasikan.
- **Recall@k:** Proporsi film relevan yang berhasil direkomendasikan dari semua film relevan yang ada. Untuk menggunakan metrik ini, diperlukan dataset ground truth yang mendefinisikan film mana yang dianggap "relevan" atau "mirip" untuk setiap film input, yang bisa berasal dari data historis interaksi pengguna atau penilaian ahli.

Secara keseluruhan, berdasarkan analisis kualitatif dari output yang dihasilkan, model yang dikembangkan mampu memberikan rekomendasi yang masuk akal dan relevan berdasarkan konten film yang diolah.
