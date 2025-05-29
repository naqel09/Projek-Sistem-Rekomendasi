# Laporan Proyek Machine Learning - Andry Septian Syahputra Tumaruk
---
## Project Overview
Dengan semakin banyaknya pilihan film yang tersedia bagi pengguna melalui berbagai platform streaming dan database, menemukan film yang sesuai dengan preferensi individu bisa menjadi tugas yang menantang. Pengguna seringkali dihadapkan pada ribuan judul tanpa panduan yang jelas, yang dapat menyebabkan kelelahan dalam memilih (choice fatigue) dan pengalaman pengguna yang kurang memuaskan. Sistem rekomendasi film hadir untuk mengatasi masalah ini dengan menyarankan film yang kemungkinan besar akan disukai pengguna.

Proyek ini bertujuan untuk membangun sistem rekomendasi film menggunakan pendekatan content-based filtering. Masalah utama yang ingin diselesaikan adalah bagaimana memberikan rekomendasi film yang relevan kepada pengguna berdasarkan konten atau atribut dari film itu sendiri, seperti alur cerita (overview), genre, kata kunci, aktor, dan sutradara. Dengan menganalisis fitur-fitur ini, sistem dapat mengidentifikasi dan merekomendasikan film-film yang memiliki kemiripan konten dengan film yang telah diketahui atau disukai pengguna. Dataset yang digunakan adalah ["TMDB Movie Metadata"](https://www.kaggle.com/datasets/tmdb/tmdb-movie-metadata) dari Kaggle, yang menyediakan informasi metadata kaya untuk ribuan film.

---
## Business Understanding
### Problem Statements

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
 - `budget`: Merupakan jumlah anggaran yang dialokasikan untuk produksi sebuah film, biasanya dalam mata uang dolar AS.
 - `genres`: Kategori atau jenis film tersebut, misalnya 'Action', 'Comedy', 'Drama', 'Science Fiction'. Dalam dataset TMDB, kolom ini awalnya seringkali berupa string JSON yang berisi daftar genre beserta ID-nya.
 - `homepage`: Alamat URL (link) ke situs web atau halaman resmi film tersebut.
 - `id`: Sebuah nomor identifikasi unik yang diberikan kepada setiap film dalam dataset `movies_df`. Dalam notebook yang Anda berikan, kolom ini diubah namanya menjadi `movie_id` untuk keperluan penggabungan data.
 - `keywords`: Kata-kata kunci atau frasa yang mendeskripsikan tema, plot, atau elemen penting dalam film. Seperti `genres`, kolom ini juga seringkali dalam format string JSON di dataset TMDB.
 - `original_language`: Kode bahasa (misalnya 'en' untuk Inggris, 'fr' untuk Prancis) yang menunjukkan bahasa asli film tersebut sebelum adanya dubbing atau subtitle.
 - `original_title`: Judul film dalam bahasa aslinya. Ini bisa berbeda dengan judul rilis internasional atau judul dalam bahasa Inggris.
 - `overview`: Sinopsis atau ringkasan singkat dari alur cerita film.
 - `popularity`: Sebuah skor numerik yang mengindikasikan seberapa populer film tersebut. Metrik ini biasanya dihitung oleh penyedia data (seperti TMDB) berdasarkan berbagai faktor seperti jumlah penonton, rating, dan aktivitas terkait lainnya.
 - `production_companies`: Daftar nama perusahaan produksi yang terlibat dalam pembuatan film. Kolom ini juga seringkali dalam format string JSON.
 - `production_countries`: Daftar negara tempat film tersebut diproduksi. Kolom ini juga seringkali dalam format string JSON.
 - `release_date`: Tanggal ketika film tersebut pertama kali dirilis ke publik.
 - `revenue`: Total pendapatan kotor yang dihasilkan oleh film tersebut dari penayangan di seluruh dunia.
 - `runtime`: Durasi atau lama waktu pemutaran film, biasanya dalam satuan menit.
 - `spoken_languages`: Daftar bahasa yang diucapkan dalam film. Kolom ini juga seringkali dalam format string JSON.
 - `status`: Status rilis film pada saat data dikumpulkan, misalnya 'Released' (Sudah Dirilis), 'Post Production' (Pasca Produksi), atau 'Rumored' (Dirumorkan).
 - `tagline`: Slogan atau frasa pendek yang menarik dan mudah diingat, yang digunakan dalam promosi film.
 - `title`: Judul film yang umum digunakan atau judul rilis utama (seringkali dalam bahasa Inggris untuk dataset internasional).kolom `title_x` dari `movies_df` dipilih dan diubah namanya menjadi `title`.
 - `vote_average`: Rata-rata skor rating yang diberikan oleh pengguna untuk film tersebut, biasanya dalam skala tertentu (misalnya 1-10).
 - `vote_count`: Jumlah total suara atau rating yang telah diterima oleh film tersebut dari pengguna.
 - Jumlah data unik pada `credits_df` (berdasarkan `movie_id`): 4803.
 - `movie_id`: Nomor identifikasi unik untuk film, yang berfungsi sebagai kunci untuk menghubungkan data kredit ini dengan data film di tabel `movies_df`.
 - `title`: Judul film, yang seharusnya konsisten dengan judul di tabel `movies_df` untuk `movie_id` yang sama. Dalam notebook, `title_x` dari `movies_df` yang akhirnya digunakan.
 - `cast`: Daftar aktor dan aktris yang berperan dalam film, beserta karakter yang mereka perankan. Kolom ini awalnya berupa string JSON dan dalam notebook diproses untuk mengambil beberapa nama aktor utama.
 - `crew`: Daftar anggota kru yang terlibat dalam produksi film, seperti sutradara, penulis skenario, produser, dll., beserta jabatan mereka. Kolom ini awalnya berupa string JSON dan dalam notebook diproses untuk mengambil nama sutradara.
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

5. **Pembuatan DataFrame Final untuk Vektorisasi:**
   - DataFrame baru final_df dibuat yang hanya berisi kolom `movie_id`, `title`, dan `tags` yang sudah diproses. DataFrame ini menjadi input untuk tahap ekstraksi fitur numerik.

6. **Ekstraksi Fitur Numerik dengan TF-IDF Vectorizer:**
   - TfidfVectorizer dari library sklearn.feature_extraction.text digunakan untuk mengubah kumpulan teks dalam kolom tags dari final_df menjadi matriks representasi numerik TF-IDF. TF-IDF (Term Frequency-Inverse Document Frequency) adalah sebuah bobot statistik yang digunakan untuk mengevaluasi seberapa penting sebuah kata dalam sebuah dokumen dalam suatu koleksi atau korpus.

   - Parameter yang digunakan pada `TfidfVectorizer`:
     - `max_features=5000`: Membatasi jumlah fitur (kata unik) yang dipertimbangkan menjadi 5000 kata yang paling sering muncul di seluruh dataset. Ini membantu mengurangi dimensi data dan fokus pada kata-kata yang lebih signifikan.

     - `stop_words='english'`: Menghilangkan kata-kata umum dalam bahasa Inggris (misalnya, "the", "is", "in") yang tidak banyak memberikan makna diskriminatif terhadap konten film.

   - Proses fit_transform() pada tfidf_vectorizer diterapkan ke kolom data['tags'] (yang berasal dari final_df['tags']) untuk mempelajari kosakata dari semua tag film dan kemudian mengubah setiap tag menjadi vektor fitur numerik. Hasilnya adalah tfidf_matrix.

   - Ukuran tfidf_matrix yang dihasilkan adalah (4800, 5000), yang berarti ada 4800 film (dokumen) dan 5000 fitur kata unik (term) yang digunakan untuk merepresentasikan setiap film.

Alasan dilakukannya tahapan persiapan data ini adalah untuk memastikan bahwa fitur-fitur yang digunakan bersih, konsisten, dan dalam format yang dapat diproses oleh algoritma machine learning (khususnya text vectorizer). Penggabungan fitur ke dalam satu kolom 'tags' menyederhanakan proses ekstraksi fitur numerik dan memungkinkan model untuk menangkap esensi konten film secara holistik.

---
## Modeling
Content-Based Filtering bekerja dengan merekomendasikan item yang serupa dengan item yang disukai pengguna di masa lalu. Dalam konteks sistem rekomendasi film, ini berarti jika pengguna menyukai film tertentu, sistem akan merekomendasikan film lain yang memiliki karakteristik (genre, kata kunci, aktor, sutradara, deskripsi) serupa.

**Cara Kerja Content-Based Filtering setelah Fitur Diekstraksi:**

1. **Ekstraksi Fitur Teks:** Fitur-fitur tekstual seperti `genres`, `keywords`, `overview`, `cast`, dan `director` dari setiap film digabungkan menjadi satu representasi tekstual (`soup`).

2. **Pembobotan TF-IDF:**

   - **TF-IDF (Term Frequency-Inverse Document Frequency)** adalah teknik pembobotan yang digunakan untuk mencerminkan seberapa penting sebuah kata dalam dokumen relatif terhadap korpus.
   - Term Frequency (TF) mengukur seberapa sering sebuah kata muncul dalam sebuah dokumen.
   - Inverse Document Frequency (IDF) mengukur seberapa jarang sebuah kata muncul di seluruh korpus dokumen. Kata yang jarang muncul di banyak dokumen tetapi sering muncul di satu dokumen tertentu akan memiliki bobot IDF yang tinggi, menunjukkan relevansinya yang unik.
   - Dalam proyek ini, `TfidfVectorizer` dari scikit-learn digunakan untuk mengonversi kumpulan teks `soup` dari semua film menjadi matriks fitur TF-IDF. Setiap baris dalam matriks ini merepresentasikan sebuah film, dan setiap kolom merepresentasikan sebuah kata (fitur), dengan nilai di setiap sel menunjukkan bobot TF-IDF kata tersebut untuk film tersebut.

3. **Penghitungan Kesamaan (Cosine Similarity):**

   - Setelah fitur tekstual dikonversi menjadi representasi numerik (matriks TF-IDF), langkah selanjutnya adalah menghitung tingkat kesamaan antara film-film.
   - **Cosine Similarity** adalah metrik yang umum digunakan untuk mengukur kesamaan antara dua vektor dalam ruang multidimensi. Nilainya berkisar antara 0 (tidak ada kesamaan) hingga 1 (sangat mirip).
   - Rumus Cosine Similarity antara dua vektor A dan B adalah:

     $$ \text{similarity} = \cos(\theta) = \frac{A \cdot B}{|A| |B|} $$ 
     
di mana A⋅B adalah produk dot dari vektor A dan B, dan ∥A∥ serta ∥B∥ adalah magnitudo (panjang) dari masing-masing vektor.

   - cosine_similarity dari scikit-learn digunakan untuk menghitung matriks kesamaan kosinus antara semua film berdasarkan matriks TF-IDF mereka. Matriks kesamaan ini akan memiliki dimensi N×N, di mana N adalah jumlah film, dan setiap sel (i,j) berisi skor kesamaan antara film i dan film j.
  
**Fungsi Rekomendasi (`get_recommendations`)**

Fungsi `get_recommendations` dirancang untuk mengambil judul film sebagai input dan mengembalikan daftar film yang direkomendasikan berdasarkan kemiripan konten.

  1. **Mendapatkan Indeks Film:** Fungsi pertama-tama mencari indeks film yang diberikan dalam dataset berdasarkan judulnya. Ini penting karena matriks kesamaan diindeks berdasarkan posisi film.
  2. **Mendapatkan Skor Kesamaan:** Setelah indeks film ditemukan, fungsi mengambil baris yang sesuai dari matriks kesamaan kosinus. Baris ini berisi skor kesamaan antara film input dan semua film lainnya.
  3. **Mengurutkan Rekomendasi:** Skor kesamaan kemudian diurutkan dalam urutan menurun. Film-film dengan skor kesamaan tertinggi adalah yang paling mirip dengan film input.
  4. **Mengeluarkan Top-N Rekomendasi:** Fungsi mengembalikan judul dari N film teratas (misalnya, 10 film) yang memiliki skor kesamaan tertinggi, tidak termasuk film input itu sendiri.

### contoh Hasil Rekomendasi:
**Rekomendasi untuk film "Avatar"**

|       | Judul Film Rekomendasi                    | Skor Similaritas|
| ----  | ----------------------------------------- | ----------------|
| 3723  | Falcon Rising                             | 0.204           |
| 582   | Battle: Los Angeles                       | 0.194           |
| 3603  | Apollo 18                                 | 0.184           |
| 47    | Star Trek Into Darkness                   | 0.171           |
| 1201  | Predators                                 | 0.166           |
| 539   | Titan A.E.                                | 0.166           |
| 2403  | Aliens                                    | 0.163           |
| 942   | The Book of Life                          | 0.163           |
| 260   | Ender's Game                              | 0.160           |
| 557   | Jarhead                                   | 0.160           |

**Rekomendasi untuk film "The Dark Knight Rises"**

|       | Judul Film Rekomendasi                     |Skor Similaritas|
|-------|--------------------------------------------|----------------|
| 65    | The Dark Knight                            | 0.445          |
| 428   | Batman Returns                             | 0.390          |
| 119   | Batman Begins                              | 0.331          |
| 299   | Batman Forever                             | 0.319          |
| 1359  | Batman                                     | 0.282          | 
| 3853  | Batman: The Dark Knight Returns, Part 2    | 0.249          |
| 9     | Batman v Superman: Dawn of Justice         | 0.242          |
| 210   | Batman & Robin                             | 0.223          |
| 507   | Slow Burn                                  | 0.194          |
| 3818  | Defendor                                   | 0.140          |

Dengan demikian, fungsi rekomendasi dibuat dan digunakan untuk memberikan saran film secara otomatis berdasarkan preferensi implisit pengguna yang ditunjukkan oleh film yang mereka tonton atau sukai.

---
## Evaluation
### Mengapa Metrik Ini Dipilih?
- **Fokus pada Top-N Rekomendasi:** Sistem rekomendasi umumnya memberikan daftar rekomendasi teratas kepada pengguna (misalnya, 10 film teratas). Metrik Precision@K dan Recall@K secara langsung mengevaluasi kualitas rekomendasi dalam daftar "Top-N" ini. Ini sangat relevan dengan pengalaman pengguna akhir, di mana pengguna cenderung hanya melihat beberapa rekomendasi pertama.

- **Precision@K:** Metrik ini sangat penting karena mengukur seberapa akurat rekomendasi yang diberikan. Dalam konteks sistem rekomendasi film, Precision@K memberitahu kita berapa proporsi film yang direkomendasikan dan benar-benar relevan bagi pengguna di antara K film teratas. Pengguna ingin rekomendasi yang mereka lihat di awal daftar adalah yang berkualitas tinggi dan relevan. Precision yang tinggi menunjukkan bahwa sistem tidak "membuang-buang" perhatian pengguna dengan rekomendasi yang tidak relevan.

- **Recall@K:** Sementara Precision@K berfokus pada relevansi dari rekomendasi yang diberikan, Recall@K mengukur seberapa lengkap sistem dalam menemukan semua item relevan yang mungkin disukai pengguna. Dalam hal ini, Recall@K memberitahu kita berapa proporsi film relevan yang berhasil ditemukan oleh sistem di antara K rekomendasi teratas dari total film relevan yang ada. Recall yang tinggi menunjukkan bahwa sistem tidak melewatkan banyak film yang seharusnya relevan bagi pengguna.

- **Keseimbangan antara Akurasi dan Cakupan:** Dalam sistem rekomendasi, seringkali ada trade-off antara Precision dan Recall. Sistem yang sangat presisi mungkin hanya merekomendasikan sedikit item tetapi semuanya relevan, sementara sistem dengan Recall tinggi mungkin merekomendasikan banyak item tetapi beberapa di antaranya mungkin tidak relevan. Dengan menggunakan kedua metrik ini secara bersamaan, kita bisa mendapatkan gambaran yang lebih komprehensif tentang performa sistem dan memahami apakah sistem lebih cenderung memberikan rekomendasi yang akurat tetapi terbatas, atau rekomendasi yang lebih luas tetapi kurang akurat.

- **Relevansi untuk Content-Based Filtering:** Untuk sistem Content-Based Filtering di mana relevansi didefinisikan berdasarkan kesamaan fitur, Precision@K dan Recall@K sangat cocok. Kita dapat dengan jelas menentukan "relevan" berdasarkan ambang batas kesamaan kosinus dan kemudian mengukur seberapa baik sistem menemukan film yang secara konten serupa.
Evaluasi adalah tahap krusial untuk mengukur seberapa baik performa sistem rekomendasi. Dalam proyek ini, evaluasi dilakukan secara kuantitatif menggunakan metrik Precision@K dan Recall@K.

### Metrik Evaluasi
- **Precision@K:** Mengukur proporsi item relevan di antara K item teratas yang direkomendasikan.

  - **Rumus:**

$$ \text{Precision@K} = \frac{\text{Jumlah rekomendasi relevan di } K \text{ teratas}}{K} $$

    
  - Dalam konteks ini, "relevan" didefinisikan berdasarkan kesamaan fitur film. Jika film yang direkomendasikan memiliki kesamaan yang tinggi dengan film yang dievaluasi, itu dianggap relevan.
Recall@K: Mengukur proporsi item relevan yang berhasil ditemukan dari seluruh item relevan yang seharusnya ada.

  - **Rumus:**
    
$$ \text{Recall@K} = \frac{\text{Jumlah rekomendasi relevan di } K \text{ teratas}}{\text{Total jumlah item relevan}} $$

        

  - "Total jumlah item relevan" di sini mengacu pada semua film dalam dataset yang memiliki skor kesamaan di atas ambang batas tertentu dengan film yang sedang dievaluasi.
    
**Proses Evaluasi**
1. **Sampling Film:** 100 film acak diambil dari dataset untuk dievaluasi.
2. **Mendapatkan Rekomendasi:** Untuk setiap film yang diambil sampelnya, 10 rekomendasi teratas (K=10) dihasilkan menggunakan fungsi get_recommendations().
3. **Mengukur Relevansi:** Untuk setiap rekomendasi, relevansinya diukur. Ambang batas kesamaan (misalnya, skor kesamaan > 0.6) digunakan untuk menentukan apakah suatu film yang direkomendasikan dianggap "relevan" dengan film aslinya.
4. **Menghitung Metrik:** Precision@10 dan Recall@10 dihitung untuk setiap film yang dievaluasi.
5. **Rata-rata Hasil:** Rata-rata dari Precision@10 dan Recall@10 dari 100 film yang dievaluasi kemudian disajikan sebagai hasil akhir.

**Hasil Numerik**
Setelah menjalankan proses evaluasi, berikut adalah hasil numerik dari metrik Precision@10 dan Recall@10:
```
------------------------------
Evaluation Results (for K=10 over 100 successfully evaluated movies):
Average Precision@10: 0.2860
Average Recall@10: 0.0057
------------------------------
```
**Analisis Hasil:**

  - **Average Precision@10: 0.2860:** Ini berarti rata-rata, sekitar 28.6% dari 10 rekomendasi teratas yang diberikan oleh sistem adalah relevan. Nilai ini menunjukkan bahwa sistem memiliki kemampuan yang cukup baik dalam merekomendasikan film yang serupa. Namun, masih ada ruang untuk peningkatan untuk merekomendasikan lebih banyak film yang relevan di antara 10 teratas.
  - **Average Recall@10: 0.0057:** Nilai Recall@10 yang rendah (sekitar 0.57%) menunjukkan bahwa sistem hanya berhasil menangkap sebagian kecil dari semua film relevan yang seharusnya ada. Ini bisa berarti bahwa meskipun rekomendasi yang diberikan relevan (sesuai dengan Precision), 
**Implikasi:**

Hasil ini menunjukkan bahwa sistem rekomendasi Content-Based Filtering yang dibangun dapat memberikan rekomendasi yang cukup akurat (dari segi Precision), tetapi mungkin kurang dalam cakupan (dari segi Recall). Potensi perbaikan dapat mencakup:

  - **Penyempurnaan Fitur:** Mengeksplorasi fitur tambahan atau teknik rekayasa fitur yang lebih canggih.
  - **Peningkatan Model:** Mempertimbangkan algoritma lain atau kombinasi algoritma (misalnya, hybrid recommendation systems).
  - **Tuning Parameter:** Mengoptimalkan ambang batas kesamaan atau jumlah rekomendasi (K) untuk mencapai keseimbangan yang lebih baik antara Precision dan Recall.
