# Modul 6
Anderson Tirza Liman

Kelas A

2406355893 

## Commit 1 Reflection Notes
Pada commit ini, saya menambahkan fungsi `handle_connection` untuk membaca dan memproses stream TCP yang masuk dari peramban (browser). Berikut adalah rincian mengenai apa yang terjadi di dalam fungsi tersebut:

`BufReader::new(&mut stream)`:

Fungsi ini membungkus `TcpStream` di dalam sebuah `BufReader`. Fungsi membaca data secara langsung dari stream bisa menjadi tidak efisien karena membutuhkan pemanggilan sistem operasi (_system calls_) yang terlalu sering dalam jumlah kecil. `BufReader` menyangga (_buffer_) proses pembacaan tersebut dengan mengambil data dalam potongan yang lebih besar dari sistem operasi, sehingga data bisa dibaca dengan jauh lebih efisien, misalnya membacanya baris demi baris.

`.lines()`:

Metode ini berasal dari trait `BufRead`. Metode ini membuat sebuah _iterator_ yang mengembalikan setiap baris data dari _stream_. Metode ini juga secara otomatis menangani pencarian karakter baris baru (`\r\n` pada protokol HTTP) untuk memisahkan _stream_ tersebut menjadi beberapa _string_ yang terpisah.

`.map(|result| result.unwrap())`:
_Iterator_ `lines()` mengembalikan tipe data `Result<String, std::io::Error>`. Fungsi `map` memproses setiap elemen di dalam _iterator_ tersebut. Dengan memanggil `unwrap()`,program diperintah untuk mengambil nilai `String` jika proses baca berhasil, atau langsung menghentikan program (_panic_) jika terjadi kesalahan saat membaca baris data.

`.take_while(|line| !line.is_empty()):`

Berdasarkan protokol HTTP, metadata (baris permintaan dan _header_) dipisahkan dari isi permintaan (_request body_) oleh sebuah baris kosong (urutan karakter `\r\n\r\n`). Fungsi ini akan terus mengambil baris dari _iterator_ sampai menemukan baris kosong tersebut. Hal ini memastikan bahwa hanya ditangkap baris permintaan HTTP dan _header_-nya saja.

`.collect():`

Metode ini mengeksekusi _iterator_ dan mengumpulkan semua _string_ yang dihasilkan ke dalam sebuah koleksi (_collection_). Karena telah ditentukan tipe data `Vec<_>` pada variabel `http_request`, Rust mengetahui bahwa ia harus mengumpulkan kumpulan _string_ tersebut ke dalam sebuah _Vector_ (struktur data daftar yang mirip dengan _array_).

`println!("Request: {:#?}", http_request):`

Perintah ini mencetak _vector_ yang telah dikumpulkan ke layar konsol. Sintaks `{:#?}` digunakan untuk mencetak hasil _debugging_ dengan format yang rapi (_pretty-printing_), sehingga daftar _header_ HTTP menjadi sangat mudah dibaca karena ditampilkan pada baris-baris yang terpisah.

## Commit 2 Reflection Notes

Pada commit ini, saya memodifikasi `handle_connection` agar tidak hanya membaca permintaan (_request_), tetapi juga mengirimkan balasan (_response_) berupa halaman HTML ke _browser_. Berikut adalah hal baru yang dipelajari dari penambahan kode tersebut:

`let status_line = "HTTP/1.1 200 OK";`:

Ini adalah baris status HTTP standar yang menandakan bahwa permintaan telah berhasil diproses oleh _server_. Versi HTTP yang digunakan adalah 1.1 dan kode status 200 berarti "OK".

`fs::read_to_string("hello.html").unwrap();`:

Fungsi dari modul `std::fs` (_file system_) ini digunakan untuk membaca seluruh isi _file_ `hello.html` dan mengubahnya menjadi tipe data `String`. Pemanggilan `unwrap()` akan menghentikan program (_panic_) jika file tersebut gagal ditemukan atau tidak bisa dibaca.

`let length = contents.len();`:
Untuk menghitung panjang _byte_ dari isi file HTML. Informasi ini dibutuhkan untuk header `Content-Length`.

`format!("{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}");`:

Makro `format!` merangkai teks-teks tadi menjadi satu balasan HTTP yang utuh dan sesuai standar (valid). Sesuai protokol HTTP, dibutuhkan `\r\n` (CRLF) untuk memisahkan baris status dengan_header_ `Content-Length`, dan wajib meletakkan dua baris kosong (`\r\n\r\n`) untuk memisahkan bagian _header_ HTTP dari bagian _body_ (isi konten HTML).

`stream.write_all(response.as_bytes()).unwrap();`:
Setelah balasan diformat dalam bentuk _string_, bentuk tersebut perlu diubah menjadi barisan _byte_ menggunakan .`as_bytes()`. Kemudian, `write_all` digunakan untuk mengirimkan seluruh _byte_ tersebut kembali ke _browser_ melalui _TCP stream_.

Tangkapan Layar:
![Commit 2 screen capture](/assets/images/commit2.png)

## Commit 3 Reflection Notes
Pada commit ini, saya menambahkan logika untuk memvalidasi permintaan yang masuk dan memberikan respons yang berbeda (selektif) berdasarkan path yang diminta oleh _browser_.

Cara Memisahkan Respons:

Pengguna tidak lagi membaca seluruh _request_, tetapi hanya mengambil baris pertama saja menggunakan `buf_reader.lines().next().unwrap().unwrap()` yang disimpan dalam variabel `request_line`. Baris pertama ini berisi metode dan rute yang diminta, misalnya `GET / HTTP/1.1`.

Telah digunakan blok `if-else` untuk mengecek string tersebut:

- Jika peramban meminta root direktori (`/`), akan disiapkan baris status `HTTP/1.1 200 OK` dan menampilkan `hello.html`.

- Jika peramban meminta rute lain (misalnya `http://127.0.0.1:7878/bad`), kode akan masuk ke blok `else`, kemudian menyiapkan baris status `HTTP/1.1 404 NOT FOUND` dan menampilkan halaman khusus yaitu `404.html`.

Mengapa Refactoring Diperlukan:

Saat menulis logika pembacaan _file_ dan penulisan respons secara langsung di dalam blok `if` dan `else`, terjadi melakukan duplikasi kode yang sangat banyak (menulis `fs::read_to_string`, `format!`, dan `stream.write_all` sebanyak dua kali).

Oleh karena itu, dilakukan _refactoring_ untuk menerapkan prinsip **DRY (_Don't Repeat Yourself_)**. Alih-alih menulis ulang seluruh proses, blok `if-else` dibuat hanya untuk mengembalikan sebuah _tuple_ berisi data yang berbeda saja, yaitu `(status_line, filename)`. Setelah blok kondisional selesai mengevaluasi nilai mana yang harus dipakai, proses membaca _file_, memformat _header_ HTTP, dan mengirimkan barisan _byte_ (`stream.write_all`) hanya perlu ditulis satu kali saja di bagian paling bawah. Hal ini membuat kode menjadi lebih bersih, ringkas, dan mudah untuk dipelihara.

Tangkapan Layar:
![Commit 3 screen capture](/assets/images/commit3.png)

## Commit 4 Reflection Notes

Pada *commit* ini, saya melakukan *refactoring* dengan mengubah struktur kendali dari `if-else` menjadi `match` agar penambahan rute baru menjadi lebih rapi dan berskala. Selain itu, telah ditambahkan sebuah rute pengujian, yaitu `/sleep`, untuk menyimulasikan respons server yang lambat (*slow request*).

Menurut hasil refleksi saya, berikut adalah pembedahan teknis dari hasil simulasi yang telah dilakukan:

* **Cara Kerja Simulasi `/sleep`**:

    Ketika peramban (*browser*) mengirimkan permintaan ke rute `/sleep`, program akan mengeksekusi `thread::sleep(Duration::from_secs(10));`. Perintah ini secara harfiah akan menghentikan sementara (menidurkan) *thread* yang sedang berjalan selama 10 detik penuh sebelum akhirnya mengembalikan respons berupa halaman `hello.html`.

* **Masalah pada Server *Single-Thread***:

    Karena arsitektur server saat ini masih *single-threaded* (berjalan pada satu utas tunggal), ia hanya sanggup memproses satu koneksi dalam satu waktu secara berurutan (*sequential*). Di dalam fungsi `main`, *loop* `for` akan tertahan dan harus menunggu eksekusi `handle_connection` selesai sepenuhnya sebelum ia bisa menerima `stream` (koneksi) dari klien berikutnya.

* **Dampak (*Bottleneck*) yang Terjadi**:

    Jika dibuka `/sleep` di satu *tab browser*, *thread* utama server akan langsung terkunci selama 10 detik. Jika di saat yang bersamaan apabila membuka rute normal (`/`) di *tab* lain, *tab* kedua tersebut tidak akan memuat tampilan apapun meskipun rutenya sangat ringan. *Tab* kedua dipaksa masuk ke dalam antrean dan menggantung (*pending*) sampai proses penahanan 10 detik dari koneksi pertama benar-benar selesai.



Simulasi ini berhasil menyingkap kelemahan fatal dari *web server single-thread*. Satu interaksi yang lambat dapat memblokir seluruh pengguna lain di dalam sistem. Untuk lingkungan dunia nyata (*production*), perilaku ini sangat tidak dapat diterima. Solusi untuk masalah ini adalah dengan beralih ke arsitektur komputasi konkuren (*concurrent*), seperti menerapkan implementasi *multi-threading* menggunakan *Thread Pool*, sehingga server dapat menangani puluhan hingga ribuan permintaan secara bersamaan.