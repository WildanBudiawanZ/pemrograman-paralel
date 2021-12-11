# Memory dan Timer

Sumber : Agus Kurniawan. 2010. Pemrograman Paralel dengan MPI dan C. Andi (293-298)

## Bekerja dengan Memori

Salah satu tujuan kita memanfaatkan sejumlah memori pada MPI adalah supaya operasi yang dilakukan berjalan lebih cepat terutama pada komunikasi satu sisi dengan remote memory access (RMA). MPI menyediakan bagaimana kita melakukan alokasi memori dan menghapusnya. Untuk alokasi memory, kita dapat menggunakan fungsi ``MPI_Alloc_mem(MPI_Aint size, MPI_Info info, void *baseptr)``.

| Parameter | Keterangan  |
| ------------- |:-------------:|
|size|ukuran alokasi memory|
|info|info argumen yang digunakan|
|baseptr|output pointer memori|

sedangkan untuk menghapus pemakaian memory kita dapat menggunakan fungsi ``MPI_Free_mem(void *base)``.

```c++
// alokasimem.c

#include <mpi.h>
#include <stdio.h>

int main( int argc, char *argv[] )
{
    int errs = 0, err;
    int j, count;
    char *ap;
	int rank;

    MPI_Init( &argc, &argv );
	MPI_Comm_rank(MPI_COMM_WORLD,&rank);
    MPI_Errhandler_set(MPI_COMM_WORLD, MPI_ERRORS_RETURN);
    for (count=1; count < 20; count *= 2)
    {
        err = MPI_Alloc_mem( count, MPI_INFO_NULL, &ap );
        if (err) 
		{
            int errclass;
            MPI_Error_class( err, &errclass );
            if (errclass != MPI_ERR_NO_MEM) 
			{
                errs++;
                printf("Error alokasi memory.\n");
            }
        }
        else 
		{            
            for (j=0; j<count; j++) 
			{
                ap[j] = (char)(j & 0x7f);
            }
			printf("Rank %d ukuran ap: %d\r\n",rank,sizeof(ap));
            MPI_Free_mem( ap );
        }
    }
    MPI_Finalize();
    return 0;
}
```

## Bekerja Dengan Timer

MPI menyediakan fungsi yang sama seperti timer. Biasanya digunakan untuk mengukur waktu lamanya suatu proses. Terdapat dua fungsi yang dapat digunakan untuk timer, yaitu:
```c++
double MPI_Wtime(void);
double MPI_Wtick(void);
```
Jika kita ingin bekerja dengan presisi yang tinggi, maka kita dapat memanfaatkan fungsi ``MPI_Wtick()``.

```c++
//timer_demo.c

#include <mpi.h>
#include <stdio.h>

int main( int argc, char *argv[] )
{
	double starttime, endtime;
    int errs = 0, err;
    int j, count;
    char *ap;
	int rank;

    MPI_Init( &argc, &argv );
	MPI_Comm_rank(MPI_COMM_WORLD,&rank);
	starttime = MPI_Wtime();
    MPI_Errhandler_set(MPI_COMM_WORLD, MPI_ERRORS_RETURN);
    for (count=1; count < 100; count *= 2)
    {
        err = MPI_Alloc_mem( count, MPI_INFO_NULL, &ap );
        if (err) 
		{
            int errclass;
            MPI_Error_class( err, &errclass );
            if (errclass != MPI_ERR_NO_MEM) 
			{
                errs++;
                printf("Error alokasi memory.\n");
            }
        }
        else 
		{            
            for (j=0; j<count; j++) 
			{
                ap[j] = (char)(j & 0x7f);
            }
            MPI_Free_mem( ap );
        }
    }

	endtime = MPI_Wtime();
	printf("Rank %d: Total waktu %f detik\r\n",rank,endtime-starttime);

    MPI_Finalize();
    return 0;
}
```
