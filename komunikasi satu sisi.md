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

```c++
int MPI_Put(void *origin_addr, int origin_count, MPI_Datatype origin_datatype, int target_rank, MPI_Aint target_disp, int target_count, MPI_Datatype target_datatype, MPI_Win win)

int MPI_Get(void *origin_addr, int origin_count, MPI_Datatype origin_datatype, int target_rank, MPI_Aint target_disp, int target_count, MPI_Datatype target_datatype, MPI_Win win)

int MPI_Accumulate(void *origin_addr, int origin_count, MPI_Datatype origin_datatype, int target_rank, MPI_Aint target_disp, int target_count, MPI_Datatype target_datatype, MPI_Op op, MPI_Win win)
```

| Parameter | Keterangan  |
| ------------- |:-------------:|
|origin_addr|alamat awal asal buffer|
|origin_count|jumlah data pada origin addr|
|origin_datatype|tipe data origin addr|
|target_rank|rank target|
|target_disp|perpindahan dari window ke target buffer|
|target_count|jumlah target buffer|
|target_datatype|tipe data target buffer|
|win|objek window|
|op|operasi reduksi|

## Sinkronisasi

Seperti kita ketahui komunikasi satu sisi terjadi pada operasi non blocking, maka sinkronisasi merupakan hal yang sangat menentukan keberhasilan operasi ini. Salah satu caranya adalah fungsi ``MPI_Win_fence(int assert, MPI_Win wn)``. Variabel assert adalah program assertion. Jika nilai assert= 0 ini adalah nilai yang selalu valid.

### Sinkronisasi Target Aktif

Untuk melakukan sinkronisasi pada target yang aktif, kita dapat memanfaatkan fungsi ``MPI_Win_start(MPI_Group group, int assert, MPI_Win win)``.
Setelah memanggil fungsi ``MPI_Win_start()``, kita harus mengakhiri sinkronisasi dengan memanggil fungsi ``MPI_Win_complete(MPI_Win win)``. Fungsi ini akan selalu menunggu hingga ada proses yang melakukan pemanggilan fungsi ``MPI_Win_post(MPI_Group group, int assert, MPI_Win win)``. Fungsi ini berguna untuk mengekpos ke target proses.
Apabila kita ingin mengetahui apakah proses eksekusi fungsi ``MPI_Win_post(MPI_Win win, int *flag)``. Jika nilai flag = true, maka proses ini sudah selesai.

### Lock

Proses sinkronisasi pada MPI menyediakan proses locking terhadap resource. Tujuan lock ini untuk memastikan tidak ada proses yang menunggunakan resource ini. Realisasinya kita dapat menggunakan fungsi ``MPI_Win_lock(int lock_type, int rank, int assert, MPI_Win win)``. Guna melepas proses lock, kita dapat menggunakan fungsi ``MPI_Win_unlock(int rank, MPI_Win win)``

## Demo

### Demo 1

Pada sesi ini akan didemokan bagaimana memanfaatkan komunikasi satu sisi untuk saling kirim data pada rank 0 dan rank 1.

```c++
//oneside_demo.c

#include <mpi.h>
#include <stdio.h>
 
#define SIZE1 8
#define SIZE2 15
 
int main(int argc, char *argv[])
{
    int rank, destrank, size, *A, *B, i;
    MPI_Group comm_group, group;
    MPI_Win win;
 
    MPI_Init(&argc,&argv);
    MPI_Comm_size(MPI_COMM_WORLD,&size);
    MPI_Comm_rank(MPI_COMM_WORLD,&rank);
    if (size != 2) 
	{
        printf("Jumlah proses harus 2\r\n");
        MPI_Abort(MPI_COMM_WORLD,1);
    }
 
    i = MPI_Alloc_mem(SIZE2 * sizeof(int), MPI_INFO_NULL, &A);
    if (i) 
	{
        printf("Error alokasi memory\r\n");
        MPI_Abort(MPI_COMM_WORLD, 1);
    }
    i = MPI_Alloc_mem(SIZE2 * sizeof(int), MPI_INFO_NULL, &B);
    if (i) 
	{
        printf("Error alokasi memory\r\n");
        MPI_Abort(MPI_COMM_WORLD, 1);
    }

    MPI_Comm_group(MPI_COMM_WORLD, &comm_group);
 
    if (rank == 0) 
	{
        for (i=0; i<SIZE2; i++) 
			A[i] = B[i] = i;

        MPI_Win_create(NULL, 0, 1, MPI_INFO_NULL, MPI_COMM_WORLD, &win);
        destrank = 1;
        MPI_Group_incl(comm_group, 1, &destrank, &group);

        MPI_Win_start(group, 0, win);
        for (i=0; i<SIZE1; i++)
            MPI_Put(A+i, 1, MPI_INT, 1, i, 1, MPI_INT, win);
        for (i=0; i<SIZE1; i++)
            MPI_Get(B+i, 1, MPI_INT, 1, SIZE1+i, 1, MPI_INT, win);
 
        MPI_Win_complete(win);
 
		printf("Rank %d B= ",rank);
        for (i=0; i<SIZE1; i++)
            printf("%d ",B[i]);
		printf("\r\n\r\n");
    }
    else 
	{ 
		/* rank=1 */
        for (i=0; i<SIZE2; i++) 
			B[i] = (2)*i;
        MPI_Win_create(B, SIZE2*sizeof(int), sizeof(int), MPI_INFO_NULL, 
			MPI_COMM_WORLD, &win);
        destrank = 0;
        MPI_Group_incl(comm_group, 1, &destrank, &group);
        MPI_Win_post(group, 0, win);
        MPI_Win_wait(win);

        printf("Rank %d B= ",rank);
        for (i=0; i<SIZE1; i++)
            printf("%d ",B[i]);
		printf("\r\n\r\n");
    }
 
    MPI_Group_free(&group);
    MPI_Group_free(&comm_group);
    MPI_Win_free(&win);
    MPI_Free_mem(A);
    MPI_Free_mem(B);
 
    MPI_Finalize();
    return 0;
} 
```

### Demo 2

Pada demo 2, kita akan mencoba menggunakan lock pada komunikasi satu sisi.

```c++
//oneside_lock.c


#include <mpi.h>
#include <stdio.h>
#include <stdlib.h>

#define SIZE1 8
#define SIZE2 15

int main(int argc, char *argv[]) 
{ 
    int rank, nprocs, A[SIZE2], B[SIZE2], i;
    MPI_Win win;
 
    MPI_Init(&argc,&argv); 
    MPI_Comm_size(MPI_COMM_WORLD,&nprocs); 
    MPI_Comm_rank(MPI_COMM_WORLD,&rank); 

    if (nprocs != 2) 
	{
        printf("Jumlah proses harus 2\r\n");
        MPI_Abort(MPI_COMM_WORLD,1);
    }

    if (rank == 0) 
	{
        for (i=0; i<SIZE2; i++) 
			A[i] = B[i] = i;
        MPI_Win_create(NULL, 0, 1, MPI_INFO_NULL, MPI_COMM_WORLD, &win); 

        for (i=0; i<SIZE1; i++) 
		{
            MPI_Win_lock(MPI_LOCK_SHARED, 1, 0, win);
            MPI_Put(A+i, 1, MPI_INT, 1, i, 1, MPI_INT, win);
            MPI_Win_unlock(1, win);
        }

        for (i=0; i<SIZE1; i++) 
		{
            MPI_Win_lock(MPI_LOCK_SHARED, 1, 0, win);
            MPI_Get(B+i, 1, MPI_INT, 1, SIZE1+i, 1, MPI_INT, win);
            MPI_Win_unlock(1, win);
        }

        MPI_Win_free(&win);

        printf("Rank %d B= ",rank);
        for (i=0; i<SIZE1; i++)
            printf("%d ",B[i]);
		printf("\r\n\r\n");
    }
    else 
	{  
		/* rank=1 */
        for (i=0; i<SIZE2; i++) 
			B[i] = (2)*i;
        MPI_Win_create(B, SIZE2*sizeof(int), sizeof(int), 
			MPI_INFO_NULL, MPI_COMM_WORLD, &win);

        MPI_Win_free(&win); 
        
        printf("Rank %d B= ",rank);
        for (i=0; i<SIZE1; i++)
            printf("%d ",B[i]);
		printf("\r\n\r\n");
    }

    MPI_Finalize(); 
    return 0; 
} 
```
