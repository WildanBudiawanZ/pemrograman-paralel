# Manajemen Communicator dan Group

Sumber : Agus Kurniawan. 2010. Pemrograman Paralel dengan MPI dan C. Andi (121-169)

## Manajemen Communicator

Pada materi sebelumnya banyak menggunakan MPI_COMM_WORLD, maka pada section ini kita akan mencoba membuat komunikator sendiri. MPI menyediakan MPI_COMM_create() untuk komunikator baru.
```c++
int MPI_Comm_create()
```

| Parameter | Keterangan  |
| ------------- |:-------------:|
| comm | Communicator |
| group | Group yang dimasukan ke communicator |
| newcomm | communicator yang baru |

Selain membuat komunikator baru, MPI menyediakan method untuk menduplikasi communicator dengan memanfaatkan ``MPI_Comm_dup()``.
Setiap kali kita membuat communicator baru, kita harus menhapusnya di akhir program dengan memanfaatkan MPI_Comm_free() yang dideklarasikan sebagai berikut:
```c++
int MPI_Comm_free()
```

### Operasi pada Communicator
Operasi yang dapat kita lakukan pada communicator ada 3 macam. Dua diantaranya sering kita gunakan untuk menentukan ukuran proses dan rank yang digunakan. Sedangkan operasi ketiga adalah compare untuk membandingkan antar komunikator.

MPI_Comm_size() //digunakan untuk menenetukan ukuran proses yang digunakan

MPI_Comm_rank() //digunakan untuk menentukan rank yang akan digunakan

MPI_Comm_compare() digunakan untuk membandingkan dua komunikator dimana hasilnya ditunjukan pada variabel result sebagai berikut:
1. MPI_IDENT, dua communicator sama
2. MPI_CONGRUENT, hanya berbeda dalam hal konteks
3. MPI_SIMILAR, anggota group sama tetapi urutannya berbeda
4. MPI_UNEQUAL, dua communicator tidak sama

```c++
comm_demo.c

#include <mpi.h>
#include <stdio.h>

int main(int argc, char* argv[] )
{
    MPI_Comm dup_comm_world, world_comm;
    MPI_Group world_group;
    int world_rank, world_size, rank;

    MPI_Init(&argc, &argv);
    MPI_Comm_rank( MPI_COMM_WORLD, &world_rank );
    MPI_Comm_size( MPI_COMM_WORLD, &world_size );

    MPI_Comm_dup( MPI_COMM_WORLD, &dup_comm_world );    
    MPI_Comm_group( dup_comm_world, &world_group );
    MPI_Comm_create( dup_comm_world, world_group, &world_comm );
    MPI_Comm_rank( world_comm, &rank );
    if (rank != world_rank) 
	{
        printf( "Rank pada world comm tidak sama: %d\n", rank );
        MPI_Abort(MPI_COMM_WORLD, 3001 );
    }
	else
		printf( "Rank world comm sama\r\n");

    MPI_Finalize();
    return 0;
}

```
