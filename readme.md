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