# Tipe data MPI banyak digunakan untuk melakukan pengiriman dan penerimaan data dari node cluster pada mesin paralel

MPI_Datatype mydata;

Penggunaan MPI_Datatype yang dilakukan untuk mengirim kumpulan data array ke sebuah node:
float a[SIZE][SIZE] = 
 {1.0, 2.0, 3.0, 4.0
  5.0, 6.0, 7.0, 8.0
  9.0, 10.0, 11.0, 12.0
  13.0, 14.0, 15.0, 16.0
 }
