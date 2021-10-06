# Komunikasi Point to Point

Sumber : Agus Kurniawan. 2010. Pemrograman Paralel dengan MPI dan C. Andi (79-119)

Mekanisme dasar sistem komunikasi pada MPI adalah proses bertukaran data pada sepasang proses dimana satu sebagai pengirim dan satunya sebagai penerima. Hampir sebagian besar komunikasi yang terjadi di MPI didasarkan pada komunikasi point-to-point. Sehingga komunikasi ini sangatlah penting dan dasar untuk komunikasi MPI.
MPI menyediakan beberapa fungsi untuk mengirim dan menerima data baik secara blocking atau non-blocking.
Blocking send/receive adalah proses yang tidak akan mengembalikan nilai sampai buffer sudah penuh dengan data yang akan dikirim/diterima. Artinya, kode selanjutnya setelah blocking send/receive tidak akan dieksekusi bila proses blocking send/receive belum selesai.
Sedangkan non-blocking akan mengembalikan nilai walaupun data yang dikirm/diterima belum dieksekusi.

Pada MPI, semua komunikasi dieksekusi oleh ``communicator``. Sebuah ``communicator`` mewakili sebuah komunikasi domain yang merupakan kumpulan proses yang saling tukar-menukar data. ``MPI_COMM_WORLD`` komunikasi domain yang umum digunakan di MPI.

## Model Komunikasi MPI

1. Model standar

a. Banyak digunakan di MPI
b. MPI menentukan apakah message yang keluar akan menggunakan buffer di lokal sistem atau tidak.
c. Bagi programmer berarti tidak dapat memaksa untuk menggunakan buffer atau tidak.

2. Model sinkronus
Operasi pengiriman akan dianggap selesai bila operasi penerima telah selesai dieksekusi dan sudah menerima data yang dikirim.

3. Model buffer

a. operasi pengiriman yang terjadi dilokal termasuk proses selesainya tidak dipengaruhi oleh kejadian diluar lokal.


