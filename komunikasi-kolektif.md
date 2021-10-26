# Komunikasi Kolektif

Sumber : Agus Kurniawan. 2010. Pemrograman Paralel dengan MPI dan C. Andi (121-169)

## Mengenal Komunikasi Kolektif

Komunikasi kolektif dapat didefinisikan sebagai sebuah komunikasi yang melibatkan sebuah group atau kumpulan group dari proses. MPI menjamin message yang dibuat oleh komunikasi kolektif tidak akan tercampur denga message yang dibuat pada komunikasi point-to-point.

## Sinkronisasi Barrier

Komunikasi yang terjadi di MPI sangat kompleks terlebih bila kita bekerja pada asinkronus. Oleh karena itu, kita memerlukan sinkronisasi antar proses yang terjadi. Hal ini dapat dilakukan dengan menggunakan ``MPI_Barrier()`` yang dideklarasikan sebagai berikut:

```
int MPI_Barrier(MPI_Comm comm)
```

Variabel ``comm`` adalah communicator dari sebuah group/

``MPI_Barrier()`` akan melakukan blocking semua proses yang terjadi sampai semua proses pada group comm sudah mencapai ``MPI_Barrier()``. Setelah semua proses mencapai ``MPI_Barrier()``, proses akan dilanjutkan seperti biasa.

```c
collective_barrier.c

#include <mpi.h>
#include <stdio.h>

int main(int argc, char* argv[]) 
{	
	int rank, total;
	
	MPI_Init( &argc, &argv );
	MPI_Comm_rank( MPI_COMM_WORLD, &rank );
	printf("Proses rank: %d \r\n", rank);
	
	// komputasi
	total = (rank+1)*25;

	MPI_Barrier(MPI_COMM_WORLD);
	
	printf("Total data pada rank %d: %d\r\n",rank,total);
	
	MPI_Finalize();
	return 0;
}
```

## Broadcast

Operasi broadcast terjadi ketika suatu proses dalam suatu spesifik group mengirim ke seluruh proses pada spesifik group. Operasi broadcast dapat dilakukan di MPI dengan memanfaatkan ``MPI_Bcast()`` yang dideklarasikan sebagai berikut:
```c
int MPI_Bcast(void*, buffer, int count, MPI_Datatype datatype, int root, MPI_Comm comm)
```

| Parameter | Keterangan  |
| ------------- |:-------------:|
| buffer | Data yang akan di-broadcast pada suatu group comm |
| count | Jumlah data |
| datatype | Tipe data |
| root | Rank dari broadcast |
| comm | Communicator yang digunakan |

```c
collective_bcast.c
#include <mpi.h>
#include <stdio.h>

int main(int argc, char* argv[]) 
{	
	int rank,val;
	
	MPI_Init( &argc, &argv );
	MPI_Comm_rank( MPI_COMM_WORLD, &rank );
	printf("Proses rank: %d \r\n", rank);
	
	if(rank==0)
		val = 100; 

	MPI_Bcast( &val, 1, MPI_INT, 0, MPI_COMM_WORLD);	
	printf("Rank %d, Total val = %d\r\n",rank,val);
	
	MPI_Finalize();
	return 0;
}
```
