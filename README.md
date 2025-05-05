# LAPORAN ANALISA KEBUTUHAN SERVER

**Nama :**
Joni Yoga Kusuma

**Judul Kasus:**
Analisa Kebutuhan Server untuk Aplikasi Pemesanan Makanan Online Lokal

**Tanggal Pengumpulan:**
05 Mei 202590t
## 2. Pendahuluan

Analisa kebutuhan server merupakan langkah krusial dalam perencanaan infrastruktur teknologi informasi untuk memastikan sebuah layanan atau aplikasi dapat berjalan optimal, handal, dan mampu mengakomodasi pertumbuhan di masa depan. Kesalahan dalam menentukan spesifikasi server, baik *under-provisioning* (spesifikasi terlalu rendah) maupun *over-provisioning* (spesifikasi terlalu tinggi), dapat berdampak negatif. *Under-provisioning* menyebabkan performa aplikasi menurun, waktu respons lambat, dan bahkan kegagalan layanan saat beban puncak, yang berujung pada pengalaman pengguna yang buruk dan potensi kehilangan pelanggan. Sebaliknya, *over-provisioning* mengakibatkan pemborosan sumber daya dan biaya operasional yang tidak perlu, baik dari segi perangkat keras maupun konsumsi energi.

Oleh karena itu, melakukan analisa yang cermat terhadap beban kerja yang diharapkan, karakteristik aplikasi, dan proyeksi pertumbuhan pengguna adalah fondasi penting sebelum melakukan pengadaan atau penyewaan server. Laporan ini bertujuan untuk menyajikan hasil analisa kebutuhan server yang sistematis untuk studi kasus Aplikasi Pemesanan Makanan Online Lokal. Analisa ini mencakup identifikasi beban kerja, estimasi spesifikasi perangkat keras dan perangkat lunak server, pertimbangan skalabilitas untuk pertumbuhan di masa depan, serta perancangan topologi deployment yang sesuai. Hasil analisa ini diharapkan dapat menjadi panduan teknis yang akurat dalam menentukan infrastruktur server yang tepat guna mendukung kelancaran operasional dan pengembangan aplikasi tersebut.
## 3. Deskripsi Layanan

Layanan yang dianalisa adalah sebuah Aplikasi Pemesanan Makanan Online Lokal. Aplikasi ini dirancang untuk menghubungkan pengguna (pelanggan) dengan berbagai restoran atau penjual makanan lokal dalam suatu area geografis tertentu. Fungsi utamanya adalah memfasilitasi proses pemesanan makanan secara daring, mulai dari pemilihan menu, penentuan jumlah pesanan, hingga proses pembayaran (meskipun detail sistem pembayaran tidak disebutkan secara eksplisit dalam skenario, asumsi dasar adalah adanya fitur ini).

Jenis aplikasi ini merupakan platform *multi-sided*, yang melayani setidaknya dua kelompok pengguna utama:
1.  **Pelanggan (Pengguna Akhir):** Menggunakan aplikasi mobile (kemungkinan besar tersedia untuk platform Android dan iOS) untuk menjelajahi daftar restoran, melihat menu, melakukan pemesanan, melacak status pesanan, dan melakukan pembayaran.
2.  **Admin/Restoran:** Menggunakan *admin dashboard* (kemungkinan berbasis web) untuk mengelola informasi restoran, menu, harga, menerima pesanan masuk, memperbarui status pesanan, dan mungkin melihat laporan penjualan.

Interaksi pengguna dengan sistem akan bervariasi. Pelanggan akan sering melakukan *browsing* menu, menambahkan item ke keranjang, dan menyelesaikan transaksi. Aktivitas ini melibatkan pembacaan data (menu, gambar makanan, informasi restoran) dan penulisan data (pesanan baru, data pengguna). Pihak admin/restoran akan lebih banyak berinteraksi untuk memperbarui data (menu, status pesanan) dan memantau aktivitas transaksi melalui dashboard. Arsitektur aplikasi kemungkinan besar melibatkan *frontend* (aplikasi mobile dan dashboard web) yang berkomunikasi dengan *backend* server melalui API (Application Programming Interface). Server backend inilah yang akan mengelola logika bisnis, otentikasi pengguna, interaksi dengan database, dan mungkin integrasi dengan layanan pihak ketiga (misalnya, gateway pembayaran atau layanan peta).
## 4. Analisa Beban Kerja

Analisa beban kerja (workload) bertujuan untuk mengestimasi seberapa besar sumber daya komputasi yang akan dibutuhkan oleh aplikasi berdasarkan aktivitas pengguna yang diharapkan. Berdasarkan skenario kasus Aplikasi Pemesanan Makanan Online Lokal, data berikut digunakan sebagai dasar analisa:

*   **Target Pengguna Harian:** 2000 pengguna.
*   **Target Transaksi Harian:** 500 - 1000 transaksi (pesanan).
*   **Komponen Aplikasi:** Aplikasi mobile untuk pelanggan dan dashboard admin berbasis web.

Dari data ini, kita dapat menguraikan lebih lanjut jenis aktivitas dan potensi beban yang dihasilkan:

1.  **Aktivitas Pengguna (Pelanggan via Mobile App):**
    *   **Browsing/Pencarian:** Pengguna akan menjelajahi daftar restoran, melihat menu, dan mencari makanan spesifik. Aktivitas ini cenderung *read-heavy*, membutuhkan query ke database untuk mengambil data restoran, menu, dan gambar. Dengan 2000 pengguna harian, jumlah sesi browsing bisa jauh lebih tinggi, mungkin mencapai 5-10 kali lipat jumlah pengguna, tergantung pada frekuensi penggunaan aplikasi oleh setiap pengguna.
    *   **Pemesanan:** Meliputi penambahan item ke keranjang, checkout, dan pembayaran. Aktivitas ini bersifat *write-heavy* karena menciptakan data pesanan baru di database. Jumlah transaksi diperkirakan 500-1000 per hari.
    *   **Pelacakan Status:** Pengguna akan memeriksa status pesanan mereka. Ini adalah aktivitas *read* yang relatif ringan tetapi bisa sering terjadi per pesanan.
    *   **Login/Autentikasi:** Terjadi saat pengguna masuk ke aplikasi.
    *   **Upload/Download Data:** Meskipun tidak secara eksplisit disebutkan, mungkin ada fitur upload bukti pembayaran atau download struk/invoice.

2.  **Aktivitas Admin/Restoran (via Admin Dashboard):**
    *   **Manajemen Menu/Restoran:** Memperbarui informasi menu, harga, jam buka, dll. Aktivitas ini bersifat *write*.
    *   **Manajemen Pesanan:** Menerima notifikasi pesanan baru, memperbarui status pesanan (diproses, dikirim, selesai). Aktivitas ini melibatkan *read* dan *write*.
    *   **Melihat Laporan:** Mengakses data penjualan dan ringkasan transaksi. Aktivitas *read-heavy*.

3.  **Beban Puncak (Peak Load):**
    *   Aplikasi pemesanan makanan cenderung memiliki jam sibuk, terutama menjelang waktu makan siang (misalnya 10:00 - 13:00) dan makan malam (misalnya 17:00 - 20:00). Beban pada jam-jam ini bisa signifikan lebih tinggi dibandingkan rata-rata harian.
    *   Transaksi harian 500-1000 berarti rata-rata sekitar 20-42 transaksi per jam jika tersebar merata. Namun, pada jam sibuk, angka ini bisa melonjak beberapa kali lipat, mungkin mencapai 100-200 transaksi per jam atau lebih, disertai dengan aktivitas browsing yang juga meningkat tajam.

4.  **Estimasi Trafik Data:**
    *   Aktivitas browsing menu dengan gambar akan mengkonsumsi bandwidth yang signifikan.
    *   Transaksi dan pembaruan status relatif lebih kecil dalam ukuran data.
    *   Perlu dipertimbangkan ukuran data yang disimpan, terutama gambar menu dan data historis transaksi.

Secara keseluruhan, aplikasi ini memiliki karakteristik beban campuran (*mixed workload*) antara operasi baca (browsing, melihat status, laporan) dan tulis (pemesanan, update data). Beban puncak pada jam-jam makan perlu menjadi perhatian utama dalam penentuan spesifikasi server untuk memastikan ketersediaan dan responsivitas layanan.
## 5. Estimasi Spesifikasi Server

Berdasarkan analisa beban kerja yang telah dilakukan, berikut adalah estimasi spesifikasi dasar server yang direkomendasikan untuk menangani kebutuhan awal Aplikasi Pemesanan Makanan Online Lokal. Estimasi ini mempertimbangkan target 2000 pengguna harian, 500-1000 transaksi per hari, serta potensi beban puncak pada jam sibuk.

**Tabel Estimasi Spesifikasi Dasar Server:**

| Komponen      | Spesifikasi Rekomendasi | Alasan Singkat                                                                 |
|---------------|-------------------------|--------------------------------------------------------------------------------| 
| **CPU**       | 4 - 6 Cores             | Cukup untuk menangani logika aplikasi, request API, dan query database dasar.    |
| **RAM**       | 16 GB                   | Menampung proses aplikasi, koneksi database, caching data (misal: menu populer). |
| **Storage**   | 500 GB SSD (NVMe)       | Kecepatan SSD penting untuk I/O database & akses cepat data aplikasi/gambar.   |
| **Network**   | 1 Gbps                  | Mendukung traffic dari 2000 pengguna harian dan transfer data gambar menu.      |
| **OS**        | Linux (Ubuntu Server LTS) | Stabilitas, keamanan, dukungan komunitas luas, dan biaya lisensi (gratis).     |

**Penjelasan Tambahan:**

*   **CPU:** Jumlah core yang moderat dipilih untuk menyeimbangkan antara kemampuan pemrosesan konkurensi (banyak pengguna akses bersamaan) dan biaya. Beban diperkirakan lebih banyak I/O-bound (akses disk/network) daripada CPU-bound murni.
*   **RAM:** Kapasitas 16GB memberikan ruang yang cukup untuk sistem operasi, web server (misal: Nginx/Apache), application server (misal: Node.js, Python/Django/Flask, PHP/Laravel), database server (misal: PostgreSQL/MySQL), dan mekanisme caching untuk mempercepat respons.
*   **Storage:** Penggunaan SSD, terutama NVMe, sangat direkomendasikan karena kecepatan baca/tulisnya yang tinggi sangat krusial untuk performa database dan waktu muat aplikasi, terutama saat menampilkan gambar menu. Kapasitas 500GB memberikan ruang awal yang cukup untuk OS, aplikasi, database, log, dan aset gambar, dengan asumsi ukuran gambar dioptimalkan.
*   **Network:** Bandwidth 1 Gbps menjadi standar umum dan seharusnya memadai untuk traffic awal. Monitoring penggunaan bandwidth tetap diperlukan.
*   **OS:** Distribusi Linux seperti Ubuntu Server Long-Term Support (LTS) dipilih karena reputasinya dalam stabilitas, keamanan, ketersediaan paket software yang luas, dukungan komunitas yang besar, dan tidak memerlukan biaya lisensi.

Estimasi ini merupakan titik awal. Spesifikasi aktual mungkin perlu disesuaikan berdasarkan hasil monitoring performa setelah sistem berjalan dan data penggunaan riil terkumpul.
## 6. Pertimbangan Pertumbuhan

Sebuah aplikasi yang sukses cenderung mengalami pertumbuhan, baik dari sisi jumlah pengguna, volume data, maupun penambahan fitur. Oleh karena itu, penting untuk mempertimbangkan potensi pertumbuhan dalam 1-2 tahun ke depan saat merancang infrastruktur server agar sistem dapat diskalakan secara efektif tanpa memerlukan perombakan besar.

Untuk Aplikasi Pemesanan Makanan Online Lokal ini, beberapa area pertumbuhan yang perlu diantisipasi adalah:

1.  **Pertumbuhan Pengguna:** Jika aplikasi diterima dengan baik di pasar lokal, jumlah pengguna harian bisa meningkat secara signifikan melebihi target awal 2000 pengguna. Mungkin mencapai 5000-10000 pengguna dalam 1-2 tahun.
2.  **Peningkatan Transaksi:** Seiring bertambahnya pengguna dan popularitas layanan, jumlah transaksi harian juga akan meningkat, berpotensi melebihi 1000 transaksi per hari.
3.  **Penambahan Restoran/Merchant:** Semakin banyak restoran yang bergabung akan menambah volume data (menu, gambar) dan kompleksitas pencarian/filter.
4.  **Volume Data:** Data historis transaksi, data pengguna, log aktivitas, dan aset gambar akan terus bertambah seiring waktu, memerlukan kapasitas penyimpanan yang lebih besar.
5.  **Penambahan Fitur:** Pengembangan di masa depan mungkin mencakup fitur baru seperti sistem rating/review, promosi, program loyalitas, integrasi dengan layanan pengiriman pihak ketiga, atau analitik yang lebih canggih. Fitur-fitur ini dapat menambah beban komputasi dan kebutuhan sumber daya.

**Implikasi terhadap Spesifikasi Server:**

*   **Skalabilitas CPU & RAM:** Perlu ada rencana untuk menambah core CPU atau kapasitas RAM (vertical scaling) atau menambah jumlah server (horizontal scaling) jika beban meningkat signifikan.
*   **Kapasitas Storage:** Pertumbuhan data memerlukan penambahan kapasitas storage atau penggunaan solusi penyimpanan yang lebih scalable (misalnya, object storage untuk gambar, database yang bisa di-shard).
*   **Bandwidth Jaringan:** Peningkatan traffic pengguna dan data transfer akan membutuhkan bandwidth jaringan yang lebih besar.
*   **Arsitektur Aplikasi:** Arsitektur harus dirancang untuk mendukung skalabilitas, misalnya dengan memisahkan komponen aplikasi (web server, application server, database server) ke server yang berbeda atau menggunakan microservices.

Mempertimbangkan potensi pertumbuhan ini sejak awal memungkinkan pemilihan teknologi dan desain arsitektur yang lebih fleksibel dan siap untuk masa depan.
## 7. Draft Spesifikasi Final

Mempertimbangkan estimasi awal dan potensi pertumbuhan yang telah dianalisa, berikut adalah draft spesifikasi final yang direkomendasikan untuk server Aplikasi Pemesanan Makanan Online Lokal. Spesifikasi ini dirancang untuk memberikan kinerja yang baik pada beban awal sambil menyediakan jalur peningkatan (upgrade path) yang jelas untuk mengakomodasi pertumbuhan di masa depan.

**Spesifikasi Final Server:**

*   **CPU:** 6 Cores (Generasi modern, misal: Intel Xeon Scalable Gen 3/4 atau AMD EPYC Gen 3/4).
    *   **Alasan:** Dipilih sedikit lebih tinggi dari estimasi awal (4-6 core) untuk memberikan headroom yang lebih baik dalam menangani lonjakan traffic pada jam sibuk dan pemrosesan latar belakang (misalnya, notifikasi, update status). Jumlah core yang lebih banyak juga membantu dalam menjalankan beberapa layanan (web server, app server, database) pada satu mesin di tahap awal, meskipun pemisahan dianjurkan untuk skalabilitas jangka panjang.
*   **RAM:** 32 GB DDR4 ECC.
    *   **Alasan:** Ditingkatkan dari 16GB menjadi 32GB. Kapasitas RAM yang lebih besar sangat penting untuk kinerja database (in-memory caching), menjalankan application server dengan efisien (terutama jika menggunakan platform seperti Java atau Node.js yang bisa intensif memori), dan menyediakan ruang yang cukup untuk caching data aplikasi (menu, sesi pengguna) guna mengurangi latensi akses.
*   **Storage:** 
    *   **Primary/OS/App:** 1 TB NVMe SSD (High IOPS).
    *   **Secondary/Data:** 2 TB SATA SSD (Untuk data historis, log, backup awal).
    *   **Alasan:** Kapasitas 1TB NVMe SSD untuk OS, aplikasi, dan database utama memastikan kecepatan I/O maksimal untuk operasi kritis. Penambahan 2TB SATA SSD memberikan ruang yang lebih besar dan hemat biaya untuk data yang tidak memerlukan akses secepat kilat, seperti log lama atau backup. Pemisahan ini juga memudahkan manajemen dan potensi upgrade di masa depan (misalnya, memindahkan database ke volume/server terpisah).
*   **Network:** 1 Gbps NIC (dengan opsi upgrade ke 10 Gbps).
    *   **Alasan:** 1 Gbps cukup untuk awal, tetapi infrastruktur jaringan (switch, router) sebaiknya mendukung 10 Gbps untuk kemudahan upgrade jika traffic data (terutama gambar dan API calls) meningkat drastis seiring pertumbuhan pengguna dan restoran.
*   **Sistem Operasi (OS):** Ubuntu Server 22.04 LTS (atau versi LTS terbaru saat implementasi).
    *   **Alasan:** Tetap dengan pilihan awal karena stabilitas, keamanan, dukungan jangka panjang, ekosistem software yang kaya, dan tidak ada biaya lisensi. LTS memastikan pembaruan keamanan dan dukungan untuk periode yang lebih lama.
*   **Database:** PostgreSQL (versi stabil terbaru).
    *   **Alasan:** PostgreSQL dikenal dengan kehandalan, fitur set yang kaya (mendukung JSONB untuk data menu yang fleksibel), skalabilitas yang baik, dan lisensi open-source. Cocok untuk aplikasi transaksional seperti pemesanan makanan.
*   **Web Server:** Nginx.
    *   **Alasan:** Nginx unggul dalam menangani koneksi konkuren yang tinggi dan efisien dalam menyajikan konten statis (seperti gambar menu) serta berfungsi sebagai reverse proxy yang handal untuk application server.
*   **Application Server:** Tergantung bahasa pemrograman (misal: Node.js dengan Express, Python dengan Django/Flask, PHP dengan Laravel).
    *   **Alasan:** Pemilihan spesifik tergantung pada tim pengembang, namun platform tersebut umum digunakan, memiliki dukungan komunitas yang baik, dan mampu menangani logika bisnis aplikasi.

Spesifikasi final ini memberikan fondasi yang solid untuk peluncuran awal dan beberapa tingkat pertumbuhan, dengan tetap memperhatikan jalur upgrade untuk skalabilitas jangka panjang.
## 8. Rancangan Topologi Deployment

Topologi deployment menggambarkan bagaimana komponen-komponen sistem (server, database, load balancer, firewall) disusun dan saling terhubung dalam infrastruktur jaringan. Untuk Aplikasi Pemesanan Makanan Online Lokal ini, dengan mempertimbangkan spesifikasi final dan kebutuhan skalabilitas, berikut adalah rancangan topologi awal yang disarankan:

**Diagram Topologi (Menggunakan Mermaid Syntax):**

```mermaid
graph TD
    subgraph Internet
        U[Pengguna Mobile App]
        A[Admin/Restoran via Web]
    end

    subgraph DMZ
        FW[Firewall]
        LB[Load Balancer]
    end

    subgraph Private Network
        subgraph Server Cluster (Initial: Single Server)
            S1[Server Aplikasi & Web (Nginx, App Server)]
            DB[(Database Server<br>(PostgreSQL))]
        end
        
        subgraph Future Scaling Options
            S2[Server Aplikasi 2]
            S3[Server Aplikasi 3]
            DB_Replica[(Database Replica)]
            Cache[Cache Server (Redis/Memcached)]
        end
    end

    U --> FW
    A --> FW
    FW --> LB
    LB --> S1
    
    S1 --> DB

    %% Future connections for scaling
    LB --> S2
    LB --> S3
    S1 --> Cache
    S2 --> Cache
    S3 --> Cache
    S1 --> DB_Replica
    S2 --> DB
    S3 --> DB
    DB --> DB_Replica
```

**Penjelasan Topologi:**

1.  **Internet:** Sumber traffic berasal dari pengguna aplikasi mobile dan admin/restoran yang mengakses dashboard web melalui internet.
2.  **Firewall (FW):** Lapisan keamanan pertama yang memfilter traffic masuk dan keluar, hanya mengizinkan koneksi pada port yang diperlukan (misalnya, HTTPS port 443).
3.  **Load Balancer (LB):** Berada setelah firewall, bertugas mendistribusikan traffic masuk ke satu atau lebih server aplikasi. Pada tahap awal, mungkin hanya ada satu server aplikasi (S1), tetapi load balancer sudah disiapkan untuk mendukung penambahan server (S2, S3) di masa depan (horizontal scaling) tanpa mengubah konfigurasi di sisi pengguna atau firewall. Load balancer juga dapat menangani terminasi SSL/TLS.
4.  **Server Aplikasi & Web (S1):** Server utama yang menjalankan web server (Nginx) untuk melayani request HTTP/S dan konten statis, serta application server (Node.js/Python/PHP) yang berisi logika bisnis aplikasi. Server ini berkomunikasi langsung dengan database.
5.  **Database Server (DB):** Menjalankan PostgreSQL untuk menyimpan dan mengelola semua data aplikasi (pengguna, restoran, menu, pesanan, dll.). Idealnya, database berada di server terpisah dari server aplikasi untuk keamanan dan performa, tetapi pada tahap awal dengan budget terbatas, bisa dijalankan di server yang sama (S1) dengan resource yang cukup. Namun, diagram ini menunjukkannya sebagai komponen logis yang bisa dipisah.
6.  **Private Network:** Server aplikasi dan database berada dalam jaringan privat yang tidak dapat diakses langsung dari internet, meningkatkan keamanan. Komunikasi antar server di jaringan ini lebih aman dan cepat.
7.  **Future Scaling Options:** Diagram juga mengindikasikan potensi penambahan server aplikasi (S2, S3), replika database (DB_Replica) untuk read scaling dan high availability, serta server cache (Redis/Memcached) untuk mempercepat akses data yang sering digunakan.

Topologi ini memberikan keseimbangan antara kesederhanaan untuk implementasi awal dan fleksibilitas untuk skalabilitas di masa depan. Penggunaan firewall dan load balancer sejak awal merupakan praktik terbaik untuk keamanan dan ketersediaan layanan.
## 9. Kesimpulan

Laporan ini telah menyajikan analisa kebutuhan server yang komprehensif untuk Aplikasi Pemesanan Makanan Online Lokal, berdasarkan skenario yang diberikan. Analisa dimulai dengan pemahaman mendalam tentang fungsi aplikasi, target pengguna (2000 harian), dan volume transaksi (500-1000 harian), yang kemudian diterjemahkan menjadi estimasi beban kerja, termasuk identifikasi potensi beban puncak pada jam-jam sibuk.

Berdasarkan analisa beban kerja dan pertimbangan pertumbuhan untuk 1-2 tahun ke depan, spesifikasi server final yang direkomendasikan adalah CPU 6 core, RAM 32 GB ECC, storage utama 1 TB NVMe SSD ditambah 2 TB SATA SSD, dan jaringan 1 Gbps dengan kesiapan upgrade. Sistem operasi yang disarankan adalah Ubuntu Server LTS, dengan PostgreSQL sebagai database dan Nginx sebagai web server/reverse proxy.

Spesifikasi ini dirancang untuk memberikan kinerja yang handal pada tahap awal peluncuran aplikasi sambil menyediakan fleksibilitas untuk skalabilitas di masa depan. Rancangan topologi deployment yang menyertakan firewall dan load balancer juga telah diusulkan untuk meningkatkan keamanan dan ketersediaan layanan, serta memfasilitasi penambahan sumber daya server seiring dengan pertumbuhan aplikasi.

Implementasi server berdasarkan rekomendasi ini diharapkan dapat mendukung operasional Aplikasi Pemesanan Makanan Online Lokal secara efektif, memberikan pengalaman pengguna yang baik, dan mengakomodasi ekspansi layanan di masa mendatang. Monitoring performa secara berkala setelah sistem berjalan sangat penting untuk validasi dan penyesuaian lebih lanjut jika diperlukan.
