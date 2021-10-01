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

Pada kode diatas,``MPI Recv`` menerima dan bertipe ``MPI_INT`` sedangkan pada ``MPI_Send`` kode diatas melakukan pengiriman array data bertipe ``DOUBLE`` maka kita memberikan tipe data ``MPI_DOUBLE`` pada ``MPI_Send``. Implementasi paling mudah adalah memanfaatkan data array atau struct,Misalkan mempunyai tipe dasar struct pada bahasa C sebagai berikut. 

```
typedef struct {    
 float x,y,z;    
 float velo;    
 int n,type; 
} particle;
```

Selanjutnya tipe data ini akan digunakan pada aplikasi MPI. Oleh karena itu kita perlu membuat suatu tipe data baru berupa tipe data turunan agar tipe data struct kita dipergunakan sesuai standar MPI.

## Tipe Data Turunan

Beberapa tipe data turunan:
1. Kontinyu
2. Vector
3. Struct
4. Index

Setiap membuat tipe data turunan pada MPI, harus konfirmasi dengan memanggil ``MPI_Type_commit()``. Sedangkan untuk menghpus menggunakan ``MPI_Type_free ()``

```
MPI_Type_commit(MPI_Datatype *datatype)
MPI_Type_free(MPI_Datatype *datatype)
```

datatype tipe data yng mau dicommit atau difree.

### Kontinyu

Memungkinkan untuk membuat tipe data yang direplikasi dengan tipe data yang sama pada bidang data yang kontinyu. Misal:
```{(double, 0), (char, 8)}```

direplikasi sebanyak 3 akan menjadi:
```{(double, 0), (char, 8), (double, 16), (char, 24), (double, 32), (char, 40)}```

untuk keperluan ini, diperlukan method MPI ``MPI_Type_contiguous``:
struktur:
```
MPI_Type_contiguous(count, oldtype, &newtype)
```
contoh:
```
MPI_Type_contiguous(3, MPI_INT, &type)
```

Full Source Code:
```
#include <mpi.h>
#include <stdio.h>
 
int main(int argc, char *argv[])
{
    int i,rank;
    MPI_Status status;
    MPI_Datatype type;
    int data[15];
 
    MPI_Init(&argc, &argv);
	MPI_Comm_rank(MPI_COMM_WORLD, &rank);

    MPI_Type_contiguous(5, MPI_INT, &type);
    MPI_Type_commit(&type);	
 
    if (rank == 0)
    {
		printf("Rank %d mengirim data= ",rank);
		for(i=0;i<15;i++)
		{
			data[i] = i;
			printf("[%d]:%d ",i,data[i]);
		}
		printf("\r\n");

        MPI_Send(data, 1, type, 1, 99, MPI_COMM_WORLD);		
    }
    else 
	if (rank == 1)
    {
		for(i=0;i<15;i++)		
			data[i] = -1;

        MPI_Recv(data, 1, type, 0, 99, MPI_COMM_WORLD, &status);		

		printf("Rank %d menerima data: ",rank);
		for(i=0;i<15;i++)
			printf("[%d]:%d ",i,data[i]);
		printf("\r\n");
    }

	MPI_Type_free(&type);
 
    MPI_Finalize();
    return 0;
}
```

