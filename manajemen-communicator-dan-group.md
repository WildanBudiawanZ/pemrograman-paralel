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
group_info.c

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

### Operasi Group

Pada group MPI, kita juga dapat melakukan operasi seperti membandingkan suatu group dengan group lainnya menggunakan 
```c++
MPI_Group_compare(MPI_Group group1, MPI_Group group2, int *result )

MPI_Group_translate(MPI_Group group1, int n, int *ranks1, MPI_Group group2, int *ranks2 )
```

| Parameter | Keterangan  |
| ------------- |:-------------:|
| group1 |group pertama |
| group2| group kedua|
| n| jumlah rank pada rank1 dan rank2 dalam bentuk array|
| rank1| array rank1|
| rank2| output array rank2|

Operasi ``MPI_Group_compare()`` digunakan untuk membandingkan kedua group. Output yang diperoleh pada operasi ini adalah:
1. MPI_IDENT, dua group sama
2. MPI_SIMILAR, dua group sama tetapi urutannya berbeda
3. MPI_UNEQUAL, dua group tidak sama

Sedangkan operasi ``MPI_Group_translate_ranks()`` digunakan untuk menentukan relativitas kedua group MPI.

```c++
group_operation.c

#include <mpi.h>
#include <stdio.h>
#include <stdlib.h>
 
int main( int argc, char **argv )
{
    MPI_Group basegroup;
    MPI_Group g1, g2, g3;
    MPI_Comm comm, newcomm, splitcomm, dupcomm;
    int i, rank, size, result;
    int nranks, *ranks, *ranks_out;

 
    MPI_Init( &argc, &argv );
    comm = MPI_COMM_WORLD;
    MPI_Comm_group(comm, &basegroup );
    MPI_Comm_rank(comm, &rank );
    MPI_Comm_size(comm, &size );

	MPI_Comm_split(comm, 0, size - rank, &newcomm );
    MPI_Comm_group(newcomm, &g1 );

    ranks = (int *)malloc( size * sizeof(int) );
    ranks_out = (int *)malloc( size * sizeof(int) );

    for (i=0; i<size; i++) 
		ranks[i] = i;
    nranks = size;
    MPI_Group_translate_ranks( g1, nranks, ranks, basegroup, ranks_out );
    for (i=0; i<size; i++) 
	{
        printf("Translate ranks=  %d Translate ranks original= %d\r\n",
					ranks_out[i], (size-1) - i );
    }

	MPI_Group_compare( basegroup, g1, &result );
    if (result != MPI_SIMILAR) 
		printf("Group tidak sama. Nilai %d\r\n", result);
	else
		printf("Group sama untuk %d\r\n", result);

    MPI_Comm_dup( comm, &dupcomm );
    MPI_Comm_group( dupcomm, &g2 );
    MPI_Group_compare( basegroup, g2, &result );

    if (result != MPI_IDENT) 
		printf("Group tidak sama %d\r\n", result);
	else
		printf("Group sama untuk result= %d\r\n",result);

    MPI_Comm_split(comm, rank < size/2, rank, &splitcomm );
    MPI_Comm_group(splitcomm, &g3 );
    MPI_Group_compare( basegroup, g3, &result );
    if (result != MPI_UNEQUAL) 
		printf("Group tidak unequal. Nilai %d\r\n", result);
	else
		printf("Group unequal. Nilai %d\r\n", result);
  
    MPI_Finalize();
    return 0;
}


```

## Inter Communicator

Inter-communicator adalah komunikasi point-to-point antar proses yang berbeda group. Sedangkan bila komunikasi terjadi pada proses di dalam group yang sama disebut sebagai intra-communicator. Untuk menguji apakah komunikasi bekerja pada inter-communicator atau tidak, kita dapat memanfaatkan fungsi ``MPI_Comm_test_inter()`` dengan parameter sebagai berikut:

| Parameter | Keterangan  |
| ------------- |:-------------:|
| comm|communicator |
| flag| output flag, bernilai 1 jika inter-communication dan sebaliknya merupakan intra-communication|

```c++
intercomm_test.c

#include <mpi.h>
#include <stdio.h>

int main(int argc, char* argv[] )
{    
	int flag;

    MPI_Init(&argc, &argv);
	MPI_Comm_test_inter(MPI_COMM_WORLD,&flag);

	if(flag==1)
		printf("inter-communicator \r\n");
	else
		printf("intra-communicator \r\n");   

    MPI_Finalize();
    return 0;
}

```

Selain fungsi ``MPI_Comm_test_inter()``, ada juga fungsi untuk memeriksa nilai size dan group dari sebuah remote communicator.
```c++

int MPI_Comm_remote_size(MPI_Comm comm, int *size)
int MPI_Comm_remote_group(MPI_Comm comm, MPI_Group *group)
int MPI_Intercomm_create(MPI_Comm local_comm, int local_leader, MPI_Comm peer_comm, int remote_leader, int tag, MPI_Comm *newintercomm)
```

| Parameter | Keterangan  |
| ------------- |:-------------:|
| local_comm| lokal intra-communicator |
| local_leader| rank lokal group leader di dalam local_comm|
| peer_comm| peer_communicator|
| remote_leader| rank lokal group leader di dalam peer_comm|
| tag| tag|
| newintercomm| communicator baru dengan inter-communication |

```c++
intercomm_create.c

#include <mpi.h>
#include <stdio.h>
#include <stdlib.h>

#define BUFSIZE 2000
int main( int argc, char *argv[] )
{
    MPI_Status status;
    MPI_Comm comm,scomm;
    int a[5], b[10];
    int i, j, rank,rank_old,rank_new, size, color;
	float num;
	int buf[BUFSIZE];

    MPI_Init(&argc, &argv);
    MPI_Comm_rank( MPI_COMM_WORLD, &rank );
	rank_old = rank;
    color = rank % 2;

    MPI_Comm_split( MPI_COMM_WORLD, color, rank, &scomm );
    MPI_Intercomm_create(scomm, 0, MPI_COMM_WORLD, 1-color, 52, &comm);
    MPI_Comm_rank(comm, &rank);
    MPI_Comm_remote_size(comm, &size );
	MPI_Comm_rank(scomm, &rank_new );
    
	MPI_Buffer_attach(buf, BUFSIZE );

	srand(rank);
	printf("Rank lama= %d Rank baru= %d a= ",rank_old,rank_new);
    for (i=0; i<5; i++) 
	{
		num = (float)rand()/RAND_MAX;
		a[i] = (int)(10.0*num)+1;
		printf("%d ",a[i]);
    }
	printf("\r\n\r\n");

    MPI_Bsend(a, 5, MPI_INT, 0, 99, comm );

    if (rank == 0) 
	{
        for (i=0; i<size; i++) 
		{
            MPI_Recv(b, 5, MPI_INT, i, 99, comm, &status);
			printf("Rank lama= %d Rank baru= %d source %d b= ",rank_old,rank_new,status.MPI_SOURCE);
			for (j=0; j<5; j++) 
			{				
				printf("%d ",b[j]);
			}
			printf("\r\n");
        }
		printf("\r\n");
    }



    MPI_Finalize();
    return 0;
}


```

## Caching

MPI menyediakan fasilitas caching yang memungkinkan aplikasi menyimpan suatu data berdasarkan atribut key. Kita dapat membuatnya dengan memanfaatkan fungsi ``MPI_Comm_create_keyval()`` dengan parameter sebagai berikut:

| Parameter | Keterangan  |
| ------------- |:-------------:|
| comm_copy_attr_fn|fungsi copy callback untuk comm_keyval |
| comm_delete_attr_fn|fungsi delete callback untuk comm_keyval |
| comm_keyval|key dan value |
| extra_state|ekstra state untuk fungsi callback |

Setelah membuat data berdasarkan key, hapus menggunakan ``MPI_Comm_free_keyval()``

Operasi yang dapat dilakukan adalah memberikan nilai, mendapatkan nilai, dan menghapus nilai berdasarkan key sebagai menggunakan method berikut:

```c++

int MPI_Comm_set_attr(MPI_Comm comm, int comm_keyval, void *attribute_val)
int MPI_Comm_get_attr(MPI_Comm comm, int comm_keyval, void *attribute_val, int *flag)
int MPI_Comm_delete_attr(MPI_Comm comm, int comm_keyval)
```

```c++
caching.c


#include <mpi.h>
#include <stdio.h>
#include <stdlib.h>

void check_attributes(MPI_Comm comm, int n, int key[], int attrval[]);
void check_no_attributes(MPI_Comm comm, int n, int key[]);

int main(int argc, char* argv[] )
{    
	int key[3], attrval[3];    
	int i,rank;
	float num;
    MPI_Comm comm;

	comm = MPI_COMM_WORLD;
    MPI_Init(&argc, &argv);
	MPI_Comm_rank(comm, &rank );	

	printf("Rank %d (key,val)= ", rank);
	srand(rank);
    for (i=0; i<2; i++) 
	{
		num = (float)rand()/RAND_MAX;
        MPI_Comm_create_keyval(MPI_NULL_COPY_FN, MPI_NULL_DELETE_FN, &key[i], (void *)0 );
        attrval[i] = (int)(10.0*num)+1;
		printf("(%d,%d) ",key[i],attrval[i]);
    }
	printf("\r\n");

    MPI_Comm_set_attr(comm, key[1], &attrval[1]);
    MPI_Comm_set_attr(comm, key[0], &attrval[0]);
    check_attributes(comm, 2, key, attrval);

    MPI_Comm_delete_attr(comm, key[0]);
    MPI_Comm_delete_attr(comm, key[1]);
    check_no_attributes(comm, 2, key);
  
	for (i=0; i<2; i++) 
	{
        MPI_Comm_free_keyval(&key[i]);
    }

    MPI_Finalize();
    return 0;
}

void check_attributes(MPI_Comm comm, int n, int key[], int attrval[])
{
    int i, flag, *val_p;
    for (i=0; i<n; i++) 
	{
        MPI_Comm_get_attr(comm, key[i], &val_p, &flag );
        if (!flag) 
		{           
            printf("Attribute key %d tidak diset\r\n", i);
        }
        else 
			if (val_p != &attrval[i]) 
			{				
				printf("Nilai Atribute key %d tidak benar\r\n", i);
			}
    }
}

void check_no_attributes(MPI_Comm comm, int n, int key[])
{
    int i, flag, *val_p;
    for (i=0; i<n; i++) 
	{
        MPI_Comm_get_attr(comm, key[i], &val_p, &flag );
        if (flag) 
		{
            printf("Attribute key %d diset tetapi seharusnya dihapus\r\n", i);
        }
    }

}

```

## Penamaan Objek

Kadang kala kita ingin mempunyai communicator dan datatype yang dapat disesuaikan. Di dalam MPI, kita dapat melakukan penyesuaian nama MPI communicator dengan fungsi berikut:
```c++
int MPI_Comm_set_name(MPI_Comm comm, char *comm_name)
int MPI_Comm_get_name(MPI_Comm comm, char *comm_name, int *resultlen)
```

sedangkan untuk datatype sebagai berikut:

```c++
int MPI_Type_set_name(MPI_Datatype type, char *type_name)
int MPI_Type_get_name(MPI_Datatype type, char *type_name, int *resultlen)
```

```c++
naming.c

#include <mpi.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main( int argc, char *argv[] )
{
    int rlen;
    char name[MPI_MAX_OBJECT_NAME], nameout[MPI_MAX_OBJECT_NAME];

    MPI_Init( &argc, &argv );

    nameout[0] = 0;
    MPI_Comm_get_name(MPI_COMM_WORLD, nameout, &rlen );
    printf( "Nama MPI_COMM_WORLD: %s\r\n", nameout );    

    strcpy(name, "MPI_COMM_NUSANTARA" );
    MPI_Comm_set_name(MPI_COMM_WORLD, name);

    nameout[0] = 0;
    MPI_Comm_get_name( MPI_COMM_WORLD, nameout, &rlen );
    printf( "Nama MPI_COMM_WORLD baru: %s\r\n", nameout );    

    MPI_Finalize();
    return 0;
}

```
