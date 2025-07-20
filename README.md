# Capstone Project 2 - Purwadhika  
## Analisis Platform Airbnb Bangkok

# Latar Belakang

Airbnb Bangkok mengalami penurunan jumlah pengguna dalam beberapa tahun terakhir. Salah satu penyebab utamanya adalah meningkatnya persaingan dari platform alternatif seperti **Booking.com**, **ThaiApartment**, dan **RentHub** yang secara strategis menargetkan ceruk pasar berbeda. 

- **Booking.com** menarik wisatawan jangka pendek dengan variasi hotel dan proses pemesanan yang mudah.  
- **ThaiApartment** unggul di kalangan penyewa jangka menengah dan bulanan karena menawarkan apartemen lokal.  
- **RentHub** berhasil menarik segmen berbiaya rendah dengan harga kompetitif dan kemudahan akses.

Dalam kondisi ini, **divisi Pemasaran dan Digital Marketing Airbnb Bangkok** dihadapkan pada tantangan ganda: 
1. Mengembalikan visibilitas merek  
2. Meningkatkan konversi pengguna  

Kedua hal tersebut harus dicapai di tengah lanskap kompetitor yang semakin terfragmentasi dan tersegmentasi. Oleh karena itu, dibutuhkan strategi yang **berbasis data**, yang mampu **mengidentifikasi elemen-elemen penawaran yang paling memengaruhi preferensi pelanggan** dalam memilih jenis akomodasi di platform Airbnb dibandingkan pesaing.

---

## Rumusan Masalah

Fokus analisis ini ditujukan untuk membantu **Manajer Pemasaran dan Digital Marketing** dalam menjawab tiga pertanyaan utama:

1. **Analisis Berdasarkan Jenis Kamar**  
   Bagaimana performa masing-masing tipe kamar dalam memengaruhi minat, preferensi, dan tingkat keterlibatan pengguna pada platform Airbnb Bangkok?

2. **Analisis Berdasarkan Segmentasi Harga**  
   Bagaimana peran segmentasi harga dalam membentuk persepsi pelanggan terhadap nilai akomodasi serta kecenderungan mereka dalam memberikan review?

3. **Analisis untuk Meningkatkan Keterlibatan Pelanggan (Customer Engagement)**  
   Strategi apa yang paling efektif untuk meningkatkan tingkat partisipasi pelanggan, khususnya dalam memberikan review, serta memperkuat loyalitas melalui optimalisasi interaksi digital?

---

Penjabaran lebih lanjut dari tiap pertanyaan akan dijelaskan dalam bagian analisis eksploratif dan hasil visualisasi data.

## 2. Data Cleaning

Proses pembersihan data dilakukan untuk memastikan kualitas data yang digunakan dalam analisis layak dan representatif. Berikut ini adalah langkah-langkah data cleaning yang dilakukan secara sistematis dan berbasis logika bisnis serta distribusi statistik.

### 2.1 Identifikasi Missing Value dan Duplikasi

- Kolom `name` dan `host_name` memiliki jumlah nilai kosong yang sangat kecil (masing-masing 8 dan 1 baris). Hal ini kemungkinan disebabkan oleh kelalaian saat input data.
- Kolom `last_review` dan `reviews_per_month` memiliki lebih dari 5.000 nilai kosong. Ini menunjukkan bahwa banyak properti belum pernah menerima review, sehingga missing value bersifat *missing by design*, bukan akibat kesalahan input atau sistem.
- Tidak ditemukan baris duplikat dalam dataset, sehingga tidak ada penghapusan duplikasi yang diperlukan.

### 2.2 Penanganan Missing Value

- **Kolom `name`**: Karena hanya 8 baris yang kosong dan kolom ini tidak digunakan dalam analisis utama, nilai kosong diisi dengan `"Missing Title"`.
- **Kolom `host_name`**: Hanya terdapat 1 missing value. Karena atribut ini relevan dengan analisis host, baris tersebut dihapus untuk menjaga integritas analisis.
- **Kolom `last_review` dan `reviews_per_month`**: Missing value di dua kolom ini terjadi secara serempak dan mencerminkan properti yang belum pernah menerima review. Tidak dilakukan imputasi. Sebagai gantinya, ditambahkan kolom baru `last_review_status` (berisi `Yes`/`No`) untuk menandai apakah listing tersebut pernah menerima review atau tidak.

### 2.3 Penanganan Outlier

#### 2.3.1 Outlier pada `price`

- Nilai `price` yang ekstrim ditemukan mulai dari **฿0 hingga ฿1.100.000** per malam, yang jauh di luar harga wajar pasar.
- Untuk menjaga validitas analisis, diterapkan batas atas realistis **฿60.000**, berdasarkan hasil pencarian dan referensi properti mewah Airbnb di Bangkok.
- Semua data dengan harga > ฿60.000 dihapus.
- Satu entri dengan harga **฿0** juga dihapus karena dianggap tidak valid dalam konteks komersial.

Referensi harga wajar: [Airbnb Bangkok – Properti Mewah](https://www.airbnb.co.id/s/Bangkok--Thailand/homes?room_types%5B%5D=Entire%20home%2Fapt)

#### 2.3.2 Outlier pada `minimum_nights`

Outlier diidentifikasi dan dihapus berdasarkan dua aturan logis:
1. Batas maksimum `minimum_nights` adalah **730 hari** (2 tahun) sesuai praktik realistis penyewaan jangka panjang.
2. Nilai `minimum_nights` tidak boleh melebihi `maximum_nights`. Jika hal ini terjadi, maka entri dianggap tidak valid.

Semua entri yang melanggar aturan di atas telah dihapus dari dataset.

### 2.4 Transformasi Variabel Numerik menjadi Kategorikal

Beberapa kolom numerik dikonversi menjadi kategorikal untuk keperluan segmentasi dan analisis yang lebih interpretatif:

- **Kolom `price`** dikategorikan menjadi:
  - `Budget` : ฿0 – ฿900
  - `Standard` : ฿900 – ฿1.421
  - `Premium` : ฿1.421 – ฿2.417.5
  - `Luxury` : > ฿2.417.5 hingga ฿60.000

  Segmentasi ini berdasarkan kuartil distribusi dan mencerminkan kelas harga di pasar.

- **Kolom `availability_365`** dikategorikan menjadi:
  - `Rendah` : 0 – 138 hari
  - `Sedang` : 138 – 309 hari
  - `Tinggi` : 309 – 365 hari

  Kategori ini digunakan untuk mengevaluasi frekuensi listing aktif secara tahunan.

- **Kolom `number_of_reviews`** digunakan untuk mengukur popularitas properti, dengan segmentasi:
  - `Rendah` : 0 – 2 review
  - `Sedang` : 2 – 13 review
  - `Tinggi` : >13 review

  Karena tidak tersedia skor rating eksplisit (0–5), maka jumlah review digunakan sebagai proksi tingkat popularitas properti.

### 2.5 Ringkasan

Seluruh proses cleaning dilakukan untuk:

- Menghapus data yang **tidak valid atau ekstrem** agar tidak mengganggu analisis.
- Mengganti nilai kosong secara selektif hanya pada kolom yang **tidak krusial** atau **tidak berisiko bias**.
- Menghindari imputasi berlebihan, khususnya untuk kolom review, demi menjaga **integritas makna data asli**.
- Mempersiapkan data untuk **analisis segmentasi dan visualisasi** dengan lebih mudah melalui transformasi kategorikal.

Langkah-langkah cleaning ini memastikan bahwa analisis yang dilakukan dapat merepresentasikan kondisi pasar Airbnb di Bangkok secara lebih akurat dan dapat dipertanggungjawabkan secara statistik maupun logis.

---
## 3. Data Analisis

Analisis data dilakukan dengan membagi fokus ke dalam tiga dimensi utama: **jenis kamar**, **segmentasi harga**, dan **engagement pelanggan**. Ketiganya digunakan sebagai dasar untuk memahami perilaku pasar, preferensi pengguna, dan peluang perbaikan dalam strategi pemasaran digital Airbnb Bangkok.

### 3.1 Analisis Berdasarkan Jenis Kamar

Jenis kamar merupakan atribut utama yang memengaruhi persepsi nilai, kenyamanan, dan privasi. Empat tipe utama dianalisis: `Entire home/apt`, `Private room`, `Shared room`, dan `Hotel room`.

**Pertanyaan kunci:**
- Jenis kamar mana yang paling populer berdasarkan volume review dan jumlah properti?
- Apakah jenis kamar tertentu lebih dominan pada segmen harga tertentu?
- Bagaimana distribusi jenis kamar per distrik, dan adakah konsentrasi spasial yang menarik?

**Temuan awal:**
- Tipe `Entire home/apt` mendominasi volume review, terutama di distrik Vadhana dan Khlong Toei.
- `Private room` banyak ditemukan di segmen harga rendah namun memiliki tingkat engagement rendah.
- `Hotel room` dan `Shared room` berkontribusi sangat kecil terhadap total review, mengindikasikan perlunya reposisi atau penyesuaian strategi.

### 3.2 Analisis Berdasarkan Segmentasi Harga

Harga merupakan salah satu faktor utama dalam pengambilan keputusan pelanggan. Data harga dibagi ke dalam empat segmen: Budget, Standard, Premium, dan Luxury menggunakan kuartil distribusi.

**Pertanyaan kunci:**
- Bagaimana sebaran unit akomodasi di tiap segmen harga?
- Apakah segmen harga tertentu cenderung menghasilkan review lebih banyak?
- Tipe kamar apa yang paling dominan di tiap segmen harga?

**Temuan awal:**
- Segmen `Premium` dan `Luxury` menghasilkan review lebih tinggi per unit, menandakan tingkat kepuasan atau keterlibatan lebih tinggi.
- `Entire home/apt` mendominasi segmen atas, sedangkan `Private room` lebih banyak di segmen `Budget` dan `Standard`.
- Perbedaan engagement antar segmen menunjukkan peluang untuk diferensiasi strategi komunikasi harga.

### 3.3 Analisis Keterlibatan Pelanggan (Customer Engagement)

Volume review digunakan sebagai proksi keterlibatan pelanggan. Semakin tinggi review, semakin besar kemungkinan adanya pengalaman kuat—positif atau negatif—yang mendorong interaksi.

**Pertanyaan kunci:**
- Apa hubungan antara segmen harga, tipe kamar, dan volume review?
- Bagaimana engagement pelanggan tersebar berdasarkan distrik?
- Distrik mana yang potensial untuk ditingkatkan engagement-nya dengan insentif atau promosi?

**Temuan awal:**
- Engagement tertinggi ditemukan di properti `Entire home/apt` pada segmen `Premium` dan `Luxury`.
- Distrik seperti Vadhana dan Khlong Toei konsisten unggul dalam review dan persebaran properti.
- Distrik seperti Bang Rak dan Huai Khwang memiliki potensi besar untuk pertumbuhan review jika diposisikan secara strategis dalam kampanye.

### 3.4 Integrasi Tiga Dimensi

Dengan menggabungkan ketiga dimensi di atas (jenis kamar, harga, engagement), didapatkan pemetaan strategis:

- `Entire home/apt` di segmen `Premium/Luxury` dan distrik seperti **Vadhana** adalah kombinasi paling optimal untuk promosi high-converting.
- `Private room` di segmen `Budget` di distrik seperti **Bang Rak** adalah target untuk peningkatan engagement berbasis insentif.
- `Shared room` dan `Hotel room` perlu diredefinisi strategi—apakah diposisikan ulang, dikemas ulang, atau dikeluarkan dari fokus utama kampanye.

---

## 4. Kesimpulan Strategis

Analisis eksploratif terhadap data Airbnb Bangkok mengungkap beberapa pola dan insight penting yang dapat menjadi dasar pengambilan keputusan strategis bagi tim Pemasaran dan Digital Marketing.

### 4.1 Kesimpulan Berdasarkan Jenis Kamar

- **Tipe "Entire home/apt"** menunjukkan dominasi mutlak di seluruh dimensi analisis, mencakup:
  - Kontribusi lebih dari **75% total review**
  - Tingkat okupansi bulanan tertinggi (**0,96 review/properti**)
  - Ketersediaan tinggi (**82,53%**) dan konsistensi listing aktif (**74,55%** dalam 12 bulan terakhir)
- **Tipe "Private room"** menunjukkan potensi pertumbuhan:
  - Kontribusi yang signifikan (**50.609 review**), namun hanya **14,26% dari total listing**
  - Tingkat review bulanan cukup stabil (**0,58/properti**) dengan **52,4% unit aktif**
- **Tipe "Hotel room" dan "Shared room"** mengalami keterbatasan kompetitif:
  - Tingkat review dan engagement rendah (<0,5 review/bulan)
  - Proporsi listing aktif di bawah **45%**, menandakan risiko stagnasi

### 4.2 Kesimpulan Berdasarkan Segmentasi Harga dan Lokasi

- **"Entire home/apt" mendominasi di seluruh segmen harga**, menegaskan preferensi pelanggan terhadap privasi dan kenyamanan.
- **"Private room" unggul di segmen Budget**, cocok untuk pelanggan berbiaya terbatas yang tetap menginginkan privasi.
- **Khlong Toei dan Vadhana** adalah distrik yang paling fleksibel dan kompetitif, mampu menjangkau seluruh segmen harga dan tipe kamar.
- **Distrik seperti Huai Khwang, Ratchathewi, dan Bang Rak** menunjukkan konsentrasi pada segmen tertentu, sehingga peluang ekspansi segmentatif masih terbuka.

### 4.3 Kesimpulan Berdasarkan Engagement Pelanggan

- **Segmen Premium dan Luxury** menyumbang lebih dari **59% review tahunan**, menunjukkan tingkat keterlibatan dan kepuasan pelanggan yang tinggi.
- **Segmen Budget memiliki jumlah review terendah**, meskipun mencakup jumlah unit terbanyak, menunjukkan rendahnya interaksi pasca-menginap.
- **"Entire home/apt"** menjadi motor utama visibilitas digital Airbnb Bangkok.

### 4.4 Kesimpulan Berdasarkan Perilaku Host

- **Vadhana dan Khlong Toei** menjadi pusat konsentrasi host, terutama host profesional dengan lebih dari lima properti.
- Host profesional lebih memilih **"Entire home/apt"** untuk efisiensi operasional dan kontrol kualitas.
- **Bang Rak** menjadi wilayah yang menonjol untuk segmen komunitas dan low-budget, dengan dominasi **Private room** dan **Shared room**.
- **Distrik pinggiran** seperti Nong Khaem dan Thawi Watthana cocok untuk model investasi rendah dengan strategi jangka panjang.

---

## 5. Rekomendasi Strategis untuk Tim Pemasaran dan Digital Marketing

### 5.1 Fokus pada Tipe "Entire home/apt"

- Prioritaskan pemasaran untuk tipe "Entire home/apt", terutama di segmen **Premium dan Luxury**.
- Tingkatkan SEO dengan kata kunci seperti: `"privasi"`, `"seluruh rumah"`, dan `"akomodasi eksklusif"`.
- Gunakan **review positif dan testimoni visual** dalam kampanye konten.

### 5.2 Optimalkan Segmen Budget Melalui "Private Room"

- Manfaatkan potensi pertumbuhan **Private room** di segmen Budget.
- Jalankan kampanye bertema: **“privat terjangkau”** dan **“nyaman terjangkau”**.
- Dorong partisipasi review dengan sistem insentif, seperti:
  - Diskon pelanggan pertama
  - Insentif review berupa poin atau kupon

### 5.3 Evaluasi dan Reposisi "Hotel room" dan "Shared room"

- Pertimbangkan untuk mengurangi alokasi pemasaran terhadap dua tipe ini.
- Jika tetap digunakan, sesuaikan dengan strategi spesifik:
  - **Hotel room** untuk **event-based** dan wisatawan bisnis
  - **Shared room** untuk **community living** atau akomodasi bersama

### 5.4 Diversifikasi Kampanye Berdasarkan Lokasi

- Jadikan **Vadhana dan Khlong Toei** sebagai target utama kampanye geografis kelas atas.
- Gunakan **geo-targeted ads** untuk segmen Premium dan Luxury.
- Untuk segmen Budget dan Standard, fokuskan pada:
  - **Bang Rak** sebagai pusat komunitas
  - **Huai Khwang dan Ratchathewi** sebagai titik efisiensi lokasi harga

### 5.5 Perkuat Strategi Engagement Review

- Fokuskan peningkatan volume review pada segmen Budget.
- Luncurkan kampanye **"Rate and Review"** pasca check-out.
- Manfaatkan **email marketing** dan **push notifications**.
- Berikan reward atau potongan harga bagi pengguna aktif yang menulis review.

### 5.6 Kembangkan Penawaran Strategis di Distrik Potensial

- Gunakan **Vadhana dan Khlong Toei** untuk meluncurkan:
  - Kampanye merek yang menonjolkan eksklusivitas dan kenyamanan
  - Penawaran khusus seperti bundling premium, promo musiman, atau layanan tambahan untuk pelanggan priorita
- Posisioning **Bang Rak** sebagai:
  - Pusat akomodasi sosial terjangkau bagi wisatawan generasi muda yang mencari keseimbangan antara harga dan kenyamanan.
  - Pilihan strategis bagi pekerja jarak jauh atau wisatawan jangka menengah yang memprioritaskan lokasi sentral, fleksibilitas waktu tinggal, dan akses terhadap kebutuhan kerja digital
