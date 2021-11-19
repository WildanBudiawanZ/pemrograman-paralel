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

### Split Communicator

Kadang kala kitaingin membagi beberapa group di dalam suatu communicator dimana masing-masing hasil pembagian ini ditandai dengan sebuat identitas, yaitu color dan key. Realisasinya, kita menggunakan ``MPI_Comm_split()`` yang dideklarasikan sebagai berikut:
```c++
int MPI_Comm_split()
```

| Parameter | Keterangan  |
| ------------- |:-------------:|
| comm | communicator |
| color | identitas color |
| key | identitas key |
| newcomm | communicator baru |

```c++
comm_split.c

#include <mpi.h>
#include <stdio.h>

int main(int argc, char* argv[] )
{
    int rank, numprocs;
	int comm_split,Zero_one,new_rank,new_nodes;
	MPI_Comm NEW_COMM;

	MPI_Init(&argc,&argv);
	MPI_Comm_size(MPI_COMM_WORLD,&numprocs);
	MPI_Comm_rank(MPI_COMM_WORLD,&rank);

	comm_split=rank % 2;
	MPI_Comm_split(MPI_COMM_WORLD,comm_split,rank,&NEW_COMM );
	MPI_Comm_rank(NEW_COMM, &new_rank);
	MPI_Comm_size(NEW_COMM, &new_nodes);

	Zero_one = -1;
	if(new_rank==0)
		Zero_one = comm_split;
	MPI_Bcast(&Zero_one,1,MPI_INT,0, NEW_COMM);

	if(Zero_one==0)
		printf("Rank %d,Bagian genap processor communicator \n",rank);
	if(Zero_one==1)
		printf("Rank %d,Bagian ganjil processor communicator \n",rank);

	printf("id lama= %d id baru= %d\n", rank, new_rank);
   
    MPI_Finalize();
    return 0;
}


```

## Manajemen Group

### Informasi Group
Setiap communicator yang dieksekusi akan mempunyai satu atau lebih group. Untuk memperoleh informasi group yang dimiliki sebuah komunikator, kita dapat menggunakan ``MPI_Comm_group()`` dengan parameter sebagai berikut:

| Parameter | Keterangan  |
| ------------- |:-------------:|
| comm | communicator |
| group | output group dari comm |

Group yang telah diperoleh dapat diketahui size ataupun rank dengan menggunakan ``MPI_Group_size()`` dan ``MPI_Group_rank()``.

```c++
comm_group_info.c

#include <mpi.h>
#include <stdio.h>
#include <stdlib.h>
 
int main( int argc, char **argv )
{
    MPI_Group basegroup;
	MPI_Comm comm;
    int grp_rank, rank, grp_size, size;
 
    MPI_Init( &argc, &argv );
    comm = MPI_COMM_WORLD;
    MPI_Comm_group(comm, &basegroup );
    MPI_Comm_rank(comm, &rank );
    MPI_Comm_size(comm, &size );
 
    MPI_Group_rank(basegroup, &grp_rank );
    if (grp_rank != rank) 
	{        
        printf("group rank %d != comm rank %d\n", grp_rank, rank );
    }
    MPI_Group_size(basegroup, &grp_size );
    if (grp_size != size) 
	{        
        printf("group size %d != comm size %d\n", grp_size, size );
    }
	printf("rank %d, size=%d, grp_rank=%d, grp_size=%d\r\n",rank,size,grp_rank,grp_size);
  
    MPI_Finalize();
    return 0;
}


```

