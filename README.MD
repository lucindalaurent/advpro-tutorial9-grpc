## REFLECTION
1. What are the key differences between unary, server streaming, and bi-directional streaming RPC (Remote Procedure Call) methods, and in what scenarios would each be most suitable?
* Unary gRPC merupakan jenis protokol dengan interaksi di mana client mengirimkan satu buah request ke server dan mendapatkan satu buah response. Unary cocok digunakan untuk melakukan _fetch_ satu bagian data dari _database_, atau untuk mengautentikasi user.
* Server streaming gRPC merupakan jenis protokol dengan interaksi di mana client mengirimkan satu buah request, sedangkan server dapat mengirim 1 aliran respon ke client. Server streaming digunakan jika server perlu mengirimkan sejumlah besar data atau update secara terus-menerus ke client. Server streaming cocok digunakan untuk mengirim update secara real-time, misalnya harga saham, berita cuaca, atau ketika server ingin mengirimkan data berukuran besar yang dibagi menjadi potongan-potongan data.
* Bidirectional streaming merupakan jenis protokol dengan interaksi di mana client maupun server bisa mengirimkan banyak pesan kepada satu sama lain dalam aliran yang kontinu. Bidirectional streaming memungkinkan komunikasi dua arah secara real-time antara server dan client. Bidirectional streaming cocok digunakan pada aplikasi chat/text messaging, collaborative editing, atau game interaktif.

2. What are the potential security considerations involved in implementing a gRPC service in Rust, particularly regarding authentication, authorization, and data encryption? <br>
* Autentikasi <br>
Autentikasi saat menggunakan layanan gRPC di Rust memastikan hanya client yang sah yang dapat mengakses layanan gRPC. Pengecekan tersebut tentu dilakukan sebelum memberikan akses ke layanan. Autentikasi dapat melibatkan penggunaan TLS (transport layer security) untuk memastikan saluran komunikasi yang aman. Autentikasi juga dapat menggunakan mekanisme autentikasi berbasis token seperti JWT(JSON Web Tokens), atau integrasi dengan sistem OAuth untuk mengautentikasi pengguna.
* Otorisasi <br>
Selain memastikan hanya client yang terautentikasi yang dapat mengakses layanan, kita juga perlu mengelola kontrol akses client berdasarkan peran dan izin yang ditetapkan. Biasanya kita memerlukan RBAC (Role Based Access Control), di mana peran/role yang diberikan pada pengguna menentukan izin atau akses yang dimilikinya. Kita bisa menggunakan middleware untuk memeriksa izin ini sebelum memberikan akses ke metode gRPC tertentu.
* Enkripsi Data <br>
Enkripsi data diperlukan untuk melindungi kerahasiaan informasi yang dikirim antara client dan server. Kita bisa mengenkripsi data menggunakan protokol seperti TLS untuk mengamankan informasi sensitif dari ancaman penyadapan, pemalsuan, atau manipulasi. <br>
<br>
gRPC menggunakan protokol HTTP2 yang stateful, di mana koneksi tetap terbuka selama stream data berlangsung. Artinya autentikasi, otorisasi, dan enkripsi data harus dilakukan untuk setiap data yang dikirimkan dalam stream. Proses berulang untuk autentikasi, otorisasi, dan enkripsi data meningkatkan keamanan secara keseluruhan. Hal ini berbeda dengan REST yang menggunakan protokol HTTP stateless. Setiap request dianggap independen sehingga autentikasi, otorisasi, dan enkripsi data hanya perlu dilakukan sekali per request. Setelah response dikirimkan, koneksi akan ditutup.

3. What are the potential challenges or issues that may arise when handling bidirectional streaming in Rust gRPC, especially in scenarios like chat applications?<br>
Tantangan atau isu dalam implementasi bidirectional streaming di gRPC Rust, khususnya untuk aplikasi obrolan, antara lain:
* Kompleksitas implementasi<br>
Bidirectional streaming melibatkan dua aliran data simultan yang tentinya lebih kompleks untuk diimplementasikan maupun dikelola dibandingkan dengan streaming searah. Kita membutuhkan pemahaman yang lebih mendalam tentang protokol gRPC dan cara menangani dua aliran data secara bersamaan. 
* Penanganan kesalahan(error) dan kegagalan koneksi<br>
Dalam aplikasi chat di mana obrolan terjadi secara real-time, error dan kegagalan koneksi harus ditangani dengan tepat untuk memastikan kelancaran komunikasi. Implementasi mekanisme untuk menangani error pada konteks bidirectional streaming mungkin akan rumit.
* Sinkronisasi data dan pesan <br>
Dalam aplikasi chat, penting untuk memastikan bahwa pesan disinkronkan dengan benar antara klien dan server untuk menghindari kebingungan dan inkonsistensi. Diperlukan mekanisme yang tepat untuk melacak status pesan dan memastikan bahwa pesan diterima dan diproses dengan benar di kedua sisi. 
* Performa dan skalabilitas <br>
Aplikasi chat, apabila memiliki banyak pengguna tentu memiliki beban traffic yang tinggi. Penting untuk mengoptimalkan implementasi bidirectional streaming yang digunakan untuk memastikan performa dan skalabilitas yang baik.
* Keamanan <br>
Implementasi bidirectional streaming harus mempertimbangkan aspek keamanan untuk melindungi data sensitif dan mencegah akses yang tidak sah. Penting untuk menerapkan mekanisme autentikasi dan otorisasi yang tepat untuk memastikan keamanan komunikasi. <br>

4. What are the advantages and disadvantages of using the tokio_stream::wrappers::ReceiverStream for streaming responses in Rust gRPC services? <br>
Kelebihan: 
* Kesederhanaan: Menyederhanakan proses streaming responses dengan menyediakan API yang lebih mudah digunakan.
* Kemampuan konkurensi: Kompatibel dengan library tokio yang populer untuk asynchronous programming di Rust.
* Backpressure: Mendukung mekanisme backpressure untuk mengontrol laju pengiriman responses.
* Error Handling: Menyediakan mekanisme error handling yang mudah digunakan. <br>
<br>
Kekurangan: 
* Overhead: Menambahkan overhead kinerja karena memerlukan layer abstraksi tambahan.
* Kompleksitas: Implementasi internalnya lebih kompleks dibandingkan dengan menggunakan `grpc::ServerStreaming<T>` secara langsung.
* Keterbatasan: Tidak mendukung semua fitur yang tersedia di `grpc::ServerStreaming<T>`.

5. In what ways could the Rust gRPC code be structured to facilitate code reuse and modularity, promoting maintainability and extensibility over time? <br>
* Service definition <br>
Dengan mendefinisikan gRPC service traits yang mengandung method signature untuk RPC, kita bisa mengimplementasikan traits tersebut untuk membuat service yang konkret. Hal ini memungkinkan kita untuk mengubah implementasi service tanpa mengubah kode client.
* Pemisahan kode gRPC client dan server<br>
Memisahkan kode gRPC client dan server ke modul atau crate yang berbeda dapat meningkatkan pengelolaan dan _reusability_ kode client ke berbagai service berbeda.
* Proto Buffers Definition <br>
Mendefinisikan `version control buffer` dapat memastikan kompabilitas antara service dan client. <br>
* Unit and Integration test<br>
Menulis unit test untuk logic dari gRPC service dan membuat integration test untuk interaksi client-server memastikan fungsionalitas kode tetap berjalan dengan baik setiap terjadi perubahan pada kode. <br>
* Dokumentasi <br>
Pemberian dokumentasi yang jelas meningkatkan pemahaman kode dan _maintainability_ untuk developer di masa mendatang.

6. In the MyPaymentService implementation, what additional steps might be necessary to handle more complex payment processing logic? <br>
* Validasi input <br>
Kita perlu memastikan setiap `PaymentRequest` yang diterima memiliki informasi yang valid dan lengkap. Misalnya memastikan user_id tidak kosong dan user dengan id tersebut memang terdaftar di sistem, atau memastikan nominal pembayarannya bilangan positif.
* Autentikasi dan otorisasi <br>
Kita perlu memastikan/memverifikasi bahwa client yang mengirimkan `PaymentRequest` merupakan user yang sudah memiliki akun dan sudah melakukan login ke dalam aplikasi. Hal ini mungkin melibatkan melibatkan pengecekan token autentikasi dalam metadata permintaan dan verifikasi izin akses client.
* Business logic <br>
Kita bisa mengimplementasikan proses sebenarnya dalam mekanisme pembayaran. Seperti mengecek apakah saldo user cukup untuk membayar, mengurangi saldo user sejumlah nominal pembayaran, penambahan saldo ke penjual sejumlah nominal pembayaran, juga pencatatan transaksi yang dilakukan.
* Penanganan Error <br>
Kita perlu menangani semua kemungkinan error yang mungkin terjadi selama proses pembayaran. Misalnya saldo kurang, kegagalan koneksi jaringan saat melakukan pembayaran, maupun masalah-masalah lain dengan memberikan pesan error yang informatif kepada client dan melakukan pemulihan data yang dibutuhkan.

7. What impact does the adoption of gRPC as a communication protocol have on the overall architecture and design of distributed systems, particularly in terms of interoperability with other technologies and platforms? <br>
* Platform Independence <br>
gRPC menggunakan protocol buffer untuk serialisasi data, yang bersifat lintas bahasa (language-agnostic). Ini memungkinkan layanan yang ditulis dalam bahasa pemrograman berbeda (misalnya, Go, Java, Rust) untuk berkomunikasi dengan lancar selama mereka berbagi definisi .proto yang sama.
* Framework Agnostic <br>
gRPC bekerja secara independen dari framework tertentu, memungkinkan integrasi dengan berbagai framework backend seperti Spring Boot (Java) atau Actix Web (Rust).
* Code Generation Overhead <br>
Setiap bahasa pemrograman memerlukan pembuatan kode dari file .proto, sehingga menambah langkah dalam proses pengembangan.
* Arsitektur microservices <br>
gRPC sangat cocok untuk arsitektur microservices karena fokusnya pada interaksi layanan yang terperinci dan terdefinisi dengan baik. Independensi bahasa dan platform mendorong modularitas dan komunikasi yang lebih mudah antara layanan independen.
* Streaming dua arah <br>
gRPC mendukung streaming dua arah, memungkinkan pola komunikasi real-time antara layanan, berguna untuk aplikasi seperti obrolan atau feed data.

8. What are the advantages and disadvantages of using HTTP/2, the underlying protocol for gRPC, compared to HTTP/1.1 or HTTP/1.1 with WebSocket for REST APIs? <br>
Kelebihan:
* Multiplexing: Dapat menangani banyak permintaan dan respons secara bersamaan melalui satu koneksi, mengurangi waktu tunggu dan memperbaiki penggunaan bandwidth.
* Server Push: Mendukung pengiriman proaktif resource dari server ke klien, mengurangi latensi dengan menghindari permintaan tambahan, meningkatkan kecepatan pemuatan halaman.
* Protokol Binari: Menggunakan format biner yang lebih efisien untuk komunikasi data dibanding text parsing, mempercepat proses analisis dan pengiriman data.
* Kompresi Header: Mengurangi overhead dengan menyusutkan ukuran header, bermanfaat terutama untuk permintaan dan respons kecil.
* Pengendalian Arus: Memiliki kontrol yang lebih baik atas aliran data, mencegah pengiriman yang berlebihan ke penerima. <br>
<br>
Kekurangan:

* Kompleksitas: Memerlukan pemahaman yang lebih mendalam dan waktu yang lebih lama untuk mengimplementasikan dan memecahkan masalah.
* Enkripsi: Sering memerlukan enkripsi TLS yang mengakibatkan overhead tambahan, terutama jika tidak diimplementasikan secara efisien.
* Interoperabilitas: Tidak semua platform dan aplikasi mendukung HTTP/2, membatasi kemampuan untuk berinteraksi dengan sistem yang lebih tua atau yang tidak kompatibel.

9. How does the request-response model of REST APIs contrast with the bidirectional streaming capabilities of gRPC in terms of real-time communication and responsiveness? <br>
REST APIs umumnya akan memerlukan waktu lebih lama dibanding gRPC untuk real-time communication. 
REST API mungkin kurang ideal untuk komunikasi real-time karena client tidak dapat menerima data dari server kecuali mengirimkan permintaan. Untuk mendapatkan pembaruan real-time, client harus memeriksa server secara berkala, sehingga kurang efisien dan berpotensi meningkatkan beban pada server. Dalam layanan gRPC dengan Bidirectional Streaming, client dan server dapat mengirimkan dan menerima data kapan saja. Hal ini ideal untuk komunikasi real-time karena server dapat mengirim pembaruan ke client segera setelah tersedia. Selain itu untuk REST API, tiap pengiriman request diperlukan pembuatan koneksi baru dari client ke server, sementara untuk gRPC hanya memerlukan 1 koneksi sampai semua request selesai. Hal ini membuat gRPC terasa lebih optimal untuk real-time communication dan lebih responsif dibanding REST APIs karena komunikasinya dapat berlangsung lebih cepat.

10. What are the implications of the schema-based approach of gRPC, using Protocol Buffers, compared to the more flexible, schema-less nature of JSON in REST API payloads? <br>
Terdapat perbedaan yang signifikan antara penggunaan gRPC dengan Protocol Buffers dan penggunaan JSON dalam payload API REST. gRPC dengan Protocol Buffers mewajibkan struktur/format data yang ketat, memastikan konsistensi data antara client dan server. Ini meminimalkan kemungkinan kesalahan saat aplikasi berjalan dan membuat proses pengiriman data lebih efisien. Namun, penggunaan Protocol Buffers memerlukan langkah tambahan dalam pengembangan aplikasi.
<br>
Di sisi lain, penggunaan JSON dalam payload API REST memberikan fleksibilitas yang lebih besar. Tanpa perlu struktur data yang terikat, JSON memungkinkan representasi data yang dinamis dan mudah diubah. Ini berguna saat aplikasi membutuhkan adaptasi cepat terhadap perubahan data. Selain itu, JSON mudah dibaca dan didukung oleh berbagai bahasa pemrograman dan web browser. Namun, penggunaan JSON dapat menghasilkan payload data yang lebih besar dan memakan waktu lebih lama dalam proses pengiriman.