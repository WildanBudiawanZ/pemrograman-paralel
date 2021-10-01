# Tipe Data dan Informasi Objek MPI

Tipe data MPI banyak digunakan untuk melakukan pengiriman dan penerimaan data dari node cluster pada mesin paralel

``MPI_Datatype mydata;``

Penggunaan MPI_Datatype yang dilakukan untuk mengirim kumpulan data array ke sebuah node:
```
float a[SIZE][SIZE] = 
 {1.0, 2.0, 3.0, 4.0
  5.0, 6.0, 7.0, 8.0
  9.0, 10.0, 11.0, 12.0
  13.0, 14.0, 15.0, 16.0
 }

MPI_Datatype rowtype;
for (i = 0; i < numtasks; i++)
 MPI_Send(&a[i][0], 1, rowtype, i, 1, MPI_COMM_WORLD);
 
```
## Tipe Data Dasar
| Tables        | Are           | Cool  |
| ------------- |:-------------:| -----:|
| col 3 is      | right-aligned | $1600 |
| col 2 is      | centered      |   $12 |
| zebra stripes | are neat      |    $1 |

## Tipe Data Turunan
