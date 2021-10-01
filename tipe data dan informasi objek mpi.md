# Tipe Data dan Informasi Objek MPI

Sumber : Agus Kurniawan. 2010. Pemrograman Paralel dengan MPI dan C. Andi (47-77)

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

| Tipe Data MPI | Persamaan Tipe Data pada Bahasa C  |
| ------------- |:-------------:|
| MPI_CHAR | signed char |
| MPI_SHORT | signed short int |
| MPI_INT  | signed int  |
| MPI_LONG  | signed long int  |
| MPI_UNSIGNED_CHAR  | unsigned char  |
| MPI_UNSIGNED_SHORT  | unsigned short int  |
| MPI_UNSIGNED  | unsigned int  |
| MPI_USIGNED_LONG  | unsigned long int  |
| MPI_FLOAT  | float  |
| MPI_DOUBLE  | double  |
| MPI_LONG_DOUBLE  | long double  |
| MPI_BYTE  |   |
| MPI_PACKED  |   |

Contoh penggunaan tipe data dasar untuk mengirim dan menerima data melalui MPI_Recv dan MPI_Send:

```
INT token,total=100;
DOUBLE array[100];

MPI_Recv(&token,1,MPI_INT,1,0,MPI_COMM_WORLD,NULL);
MPI_Send(array,total,MPI_DOUBLE,1,0,MPI_COMM_WORLD); 
```


## Tipe Data Turunan
