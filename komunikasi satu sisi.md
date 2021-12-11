# Komunikasi Satu Sisi

Sumber : Agus Kurniawan. 2010. Pemrograman Paralel dengan MPI dan C. Andi (279-292)

## Mengenal Komunikasi Satu Sisi

MPI menyediakan pengembangan hal komunikasi dengan memanfaatkan remote memory access (RMS) di mana satu proses dapat bertindak sebagai sisi penerima dan sisi pengirim. FItur ini hanya tersedia pada MPI 2 ke atas. Desain rancangan seperti ini para pengguna MPI dapat memperoleh keuntungan seperti mekanisme komunikasi dengan berbagai platform. Semua komunikasisatu sisi adalah proses non blocking. Oleh karena itu, kita memerlukan mekanisme sinkronisasi antar prosesnya.

## Membangun Komunikasi

Kita dapat membuat komunikasi satu sisi yang sudah disediakan MPI yaitu ``MPI_Win_create()`` yang dideklarasikan sebgai berikut:
```c++
int MPI_Win_create(void *base, MPI_Aint size, int disp_unit, MPI_Info info, MPI_Comm comm, MPI_Win *win)
```
Keterangan paratemernya dapat dilihat sebagai berikut:
| Parameter | Keterangan  |
| ------------- |:-------------:|
|base|alamat awal window|
|size|ukuran window dalam bytes|
|disp_unit|ukuran unit lokal untuk interval|
|info|objek info|
|comm|communicator|
|win|output objek window|

Setelah membuat komunikasi satu sis, kita harus menghapusdengan menggunakan fungsi ``MPI_Win_free()``. Kita juga dapat memperoleh group yang di dalamnya telah membuat komunikasi satu sisi pada sebuah communicator. Ini dapat dilakukandengna fungsi ``MPI_Win_get_group()``.

## Operasi pada Komunikasi Satu Sisi

Ada tiga operasi yang dapat dilakukan pada komunikasi satu sisi:
- ``MPI_Put()`` digunakan untuk menulis data ke remote proses. Proses yang terjadi di sini adalah proses blocking.
- ``MPI_Get()`` digunakan untuk membaca data dari remote proses.
- ``MPI_Accumulate()`` digunakan untuk mengupdate data. Pada fungsi ini terjadi operasi reduksi.

## Sinkronisasi

## Demo
