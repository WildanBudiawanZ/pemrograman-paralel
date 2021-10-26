# Komunikasi Kolektif

## Mengenal Komunikasi Kolektif

Komunikasi kolektif dapat didefinisikan sebagai sebuah komunikasi yang melibatkan sebuah group atau kumpulan group dari proses. MPI menjamin message yang dibuat oleh komunikasi kolektif tidak akan tercampur denga message yang dibuat pada komunikasi point-to-point.

## Sinkronisasi Barrier

Komunikasi yang terjadi di MPI sangat kompleks terlebih bila kita bekerja pada asinkronus. Oleh karena itu, kita memerlukan sinkronisasi antar proses yang terjadi. Hal ini dapat dilakukan dengan menggunakan ``MPI_Barrier()`` yang dideklarasikan sebagai berikut:

```
int MPI_Barrier(MPI_Comm comm)
```

Variabel ``comm`` adalah communicator dari sebuah group/

``MPI_Barrier()`` akan melakukan blocking semua proses yang terjadi sampai semua proses pada group comm sudah mencapai ``MPI_Barrier()``. Setelah semua proses mencapai ``MPI_Barrier()``, proses akan dilanjutkan seperti biasa.

```
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
