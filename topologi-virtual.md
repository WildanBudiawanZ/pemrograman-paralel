# Topologi Virtual

Sumber : Agus Kurniawan. 2010. Pemrograman Paralel dengan MPI dan C. Andi (121-169)

## Apakah itu Topologi Virtual

Topologi virtual lebih direpresentasi sebagai sebuah graph. Dengan ini, kita dapat melakukan  ilustrasi yang berhubungan demgam graph. Misal kita mengekesekusi sebuah aplikasi MPI dengan 4 proses. Pada keadaan ini kita dapat melakukan virtual koordinat dengan contoh sebagai berikut:
- Koordinat (0,0) = rank 0
- Koordinat (0,1) = rank 1
- Koordinat (1,0)= rank 2
- Koordinat (1,1) = rank 3

Ada banyak model yang dapat dibuat pada topologi virtual. Sedangkan pada konteks MPI, kita dapat membuat virtual dengan 3 tipe, yaitu:
- Topologi Kartesian
- Topologi Graph
- Topologi Distribusi

## Topologi Kartesian

Topologi kartesian merupakan virtual topologi yang berbentuk grid atau bentuknya seperti matrix.

### Membuat topologi kartesian

MPI menyediakan fungsi untuk membuat sebuah virtual topologi kartesian yaitu:

```c++
int MPI_Cart_create(MPI_Comm comm_old, int ndims, int *dims, int *periods, int reorder, MPI_Comm *comm_cart)
```
| Parameter | Keterangan  |
| ------------- |:-------------:|
|comm  |coomunicator |
|ndims  |jumlah dimensi dari grid kartesian |
|dims  |array integer yang berisi ndims untuk masing-masing dimensi |
|periods  | array logical periodic (true) atau tidak pada masing-masing dimensi|
| reorder | ranking yang diurutkan atau tidak|
| comm_cart | communicator baru yang berisi virtual topologi kartesian|

```c++
//cartesian.c

#include<mpi.h>
#include<stdio.h>

 /* 2D torus untuk 12 processes dalam barisxkolom 4x3 grid */
int main(int argc, char *argv[])
{
    int rank, size;
    MPI_Comm comm;
    int dim[2], period[2], reorder;
    int coord[2], id;

    MPI_Init(&argc, &argv);
    MPI_Comm_rank(MPI_COMM_WORLD, &rank);
    MPI_Comm_size(MPI_COMM_WORLD, &size);
    if (size != 12)
    {
        printf("Harus running dengan 12 processes.\r\n");
        MPI_Finalize();
		return 0;
    }

    dim[0]=4; dim[1]=3;
    period[0]=1; period[1]=0;
    reorder=1;
    MPI_Cart_create(MPI_COMM_WORLD, 2, dim, period, reorder, &comm);

	MPI_Cart_coords(comm, rank, 2, coord);
    printf("Rank %d posisi(%d, %d)\r\n", rank, coord[0], coord[1]);

	if(rank==5)
	{
		coord[0]=1; coord[1]=2;
        MPI_Cart_rank(comm, coord, &id);
        printf("Rank posisi (%d, %d): %d\r\n", coord[0], coord[1], id);
	}
    
    MPI_Finalize();
    return 0;
}


```

### Mapping koodinasi topologi kartesian terhadap rank

Setelah kita membuat virtual topologi kartesian, kita dapat memperoleh informasi yang berhubungan dengan topologi ini, yaitu koordinasi. Bagaimana posisi suatu koordinasi terhadap suatu rank proses MPI. Solusi ini adalah kita menggunakan fungsi ``MPI_Cart_coords()`` yang sudah disediakan MPI.

```c++
int MPI_Cart_coords(MPI_Comm comm, int rank, int maxdims, int *coords)
```

| Parameter | Keterangan  |
| ------------- |:-------------:|
| comm |communicator bertopologi kartesian |
|rank|rank proses dalam group comm|
|maxdims|panjang vektor coords|
|coords|array integer berisi koordinasi kartesian|

### Operasi Shift
 
Pada sebuah virtual topologi kartesian kita dapat melakukan pergeseran suatu koordinat. Hal ini sangat mudah dilakukan dengan menggunakan ``MPI_Cart_shift()`` yang dideklarasikan sebagai berikut:
```c++
itn MPI_Cart_shift(MPI_Comm comm, int direction, int disp, int *rank_source, int *rank_dest)
```

| Parameter | Keterangan  |
| ------------- |:-------------:|
|comm| communicator bertopologi kartesian|
|direction|arah pergeseran|
|disp|perpindahan, jika bernilai > 0 maka perpindahan ke atas jika bernilai < 0 perpindahan ke bawah|
|rank_source| sumber rank|
|rank_dest| tujuan rank|

```c++
//cartesian_shift.c

#include <mpi.h>
#include <stdio.h>
#include <stdlib.h>

int main (int argc, char *argv[])
{
	int rank,size;
	MPI_Comm comm;
	int coord[2],period[2],reorder;
	int up,down,right,left;

	MPI_Init(&argc, &argv);
	MPI_Comm_rank(MPI_COMM_WORLD,&rank);
	MPI_Comm_size(MPI_COMM_WORLD, &size);
    if (size != 12)
    {
        printf("Harus running dengan 12 processes.\r\n");
        MPI_Finalize();
		return 0;
    }

	coord[0]=4; coord[1]=3;
	period[0]=1; period[1]=0;
	reorder=1;
	MPI_Cart_create(MPI_COMM_WORLD,2,coord,period,reorder,&comm);	

	MPI_Cart_coords(comm, rank, 2, coord);
	printf("Rank %d, Dimensi 4x3 posisi(%d, %d)\r\n", rank, coord[0], coord[1]);
	if(rank==7)
	{		
		MPI_Cart_shift(comm,0,1,&up,&down);
		printf("Rank:%d Up:%d Down:%d\r\n",rank,up,down);
		MPI_Cart_shift(comm,1,1,&left,&right);
		printf("Rank:%d Left:%d Right:%d\r\n",rank,left,right);	
	}

	MPI_Finalize();
}

```

### Partisi

Pada umumnya, komunikasi kolektif melibatkan semua proses diperuntukan pada communicator yang sama. Kadang kala kita ingin operasi ini hanya diperlukan pada subset dari proses suatu komunikator. Hal ini dapat dilakukan dengan melakukan memotong topologi kartesian menjadi beberapa bagian. Masing-masing potongan ini akan mempunyai communicator sendiri. Dengan kondisi ini, bagian-bagian ini akan mengeksekusi komunikasi kolektif sendiri-sendiri. Semua proses ini dapat dilakukan dengan operasi ``MPI_Cart_sub()``.

```c++
int MPI_Cart_sub(MPI_Comm comm, int *remain_dims, MPI_Comm *new_comm)
```
| Parameter | Keterangan  |
| ------------- |:-------------:|
|comm|communicator bertopologi kartesian|
|remain_dims|array bagian yang mana akan menjadi mejadi subgrid|
|newcomm|tujuan rank|

```c++
//cartesian_sub.c

#include <mpi.h>
#include <stdio.h>

int main( int argc, char *argv[] )
{
    int size, dims[2], periods[2], remain[2];
    int result;
    MPI_Comm comm, newcomm;

    MPI_Init( &argc, &argv );
    
    periods[0] = 0;
    MPI_Comm_size( MPI_COMM_WORLD, &size );
    dims[0] = size;
    MPI_Cart_create( MPI_COMM_WORLD, 1, dims, periods, 0, &comm );
    
    remain[0] = 0;
    MPI_Cart_sub( comm, remain, &newcomm );
    
    MPI_Comm_compare(MPI_COMM_SELF, newcomm, &result );
    if (result != MPI_CONGRUENT) 
		printf("newcomm bukan MPI_COMM_SELF\r\n");
	else
		printf("newcomm adalah MPI_COMM_SELF\r\n");
    
    MPI_Comm_free( &newcomm );
    MPI_Comm_free( &comm );

    MPI_Finalize();
    return 0;
}

```

## Topologi graph

### Membuat topologi graph

Kita dapat membuat virtual topologi graph dengan memanfaatkan fungsi ``MPI_Graph_create()`` dengan parameter sebagai berikut:
| Parameter | Keterangan  |
| ------------- |:-------------:|
|comm_old|input communicator|
|nnodes|jumlah nodes|
|index|array integer untuk derajat node|
|edges|array integer untuk edge|
|reorder|ranking akan diurutkan atau tidak|
|comm_graph|communicator baru untuk topologi graph|

```c++
//graph_create.c

#include <mpi.h>
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char *argv[])
{
    int i, k;
    int size,rank;
    int topo_type;
    int *index, *edges;
    MPI_Comm comm1, comm2;


    MPI_Init(&argc, &argv);
    MPI_Comm_rank(MPI_COMM_WORLD, &rank);
    MPI_Comm_size(MPI_COMM_WORLD, &size);

	index = (int*)malloc(rank*sizeof(int));
	edges = (int*)malloc(rank*2*sizeof(int));
		
	index[0] = 2;
	for (i=1; i<size; i++) 
		index[i] = 2 + index[i-1];

	k=0;
	for (i=0; i<size; i++) 
	{
		edges[k++] = (i-1+size) % size;
		edges[k++] = (i+1) % size;
	}

	MPI_Graph_create(MPI_COMM_WORLD, size, index, edges, 0, &comm1);
	MPI_Comm_dup(comm1, &comm2 );
	MPI_Topo_test(comm2, &topo_type);

	if (topo_type != MPI_GRAPH) 
		printf("Tipe topology bukan MPI_GRAPH\r\n");
	else 
		printf("Tipe topology MPI_GRAPH\r\n");

	    
    MPI_Finalize();

    return 0;
}


```

## Topologi distribusi

Virtual topologi terakhir adalah virtual topologi distribusi. Realisasi untuk membuat topologi distribusi, kita dapat menggunakan fungsi ``MPI_Dist_graph_create()``.

```c++
int MPI_Dist_graph_create(MPI_Comm comm_old, int n, int sources[]. int degrees[], int destinations[], int weights[], MPI_Info info, int reorder, MPI_Comm *comm_dist_graph)
```

| Parameter | Keterangan  |
| ------------- |:-------------:|
|comm_old| input communicator|
|n|jumlah source node|
|sources|array yang berisi n source node dengan spesifik edge|
|degrees| array jumlah destination untuk masing-masing source node|
|destinations|tujan dari source node|
|weights|bobot untuk source node ke destination edge|
|info|informasi optimasiasi dari bobot|
|reorder|proses akan diurutkan atau tidak|
|comm_dist_graph|communicator yang menghasilkan topologi distribusi|

```c++
//distgraph_create.c

// KODE PROGRAM ini menggunakan MPICH2

#include <mpi.h>
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char *argv[])
{
    int size,rank;
    int topo_type;
    MPI_Comm comm1, comm2;
	int  x, y;
	int sources[1], degrees[1];
	int destinations[8], weights[8];

	int P = 2,Q = 2; // size = P*Q


    MPI_Init(&argc, &argv);
    MPI_Comm_rank(MPI_COMM_WORLD, &rank);
    MPI_Comm_size(MPI_COMM_WORLD, &size);

	y=rank%P; x=rank%P;
	destinations[0] = P*y+(x+1)%P; weights[0] = 2;
	destinations[1] = P*y+(P+x-1)%P; weights[1] = 2;

	destinations[2] = P*((y+1)%Q)+x; weights[2] = 2;
	destinations[3] = P*((Q+y-1)%Q)+x; weights[3] = 2;

	destinations[4] = P*((y+1)%Q)+(x+1)%P; weights[4] = 1;
	destinations[5] = P*((Q+y-1)%Q)+(x+1)%P; weights[5] = 1;
	destinations[6] = P*((y+1)%Q)+(P+x-1)%P; weights[6] = 1;
	destinations[7] = P*((Q+y-1)%Q)+(P+x-1)%P; weights[7] = 1;
	sources[0] = rank;
	degrees[0] = 8;
		
	MPI_Dist_graph_create(MPI_COMM_WORLD, 1, sources, degrees, destinations,
			weights, MPI_INFO_NULL, 1, &comm1);
	MPI_Comm_dup(comm1, &comm2 );
	MPI_Topo_test(comm2, &topo_type);

	if (topo_type != MPI_GRAPH) 
		printf("Tipe topology bukan MPI_GRAPH\r\n");
	else 
		printf("Tipe topology MPI_GRAPH\r\n");
	    
    MPI_Finalize();

    return 0;
}


```

## Pengujian dan testing

Pada section ini kita akan melakukan pengujian dan testing terhadap virtual topologi yang telah dibuat. Pengujian pertama adalah menentukan jenir virtual topologi pada sebuah communicator. Caranya dengan menggunakan fungsi ``MPI_Topo_test()``.

```c++
int MPI_Topo_test(MPI_Comm comm, int *status)
```

Nilai status merupakan output dari fugnsi ini, beberapa outptu yang didefinisi sebagai virtual topologi dapat dilihat pada tabel berikut:
| Output | Keterangan  |
| ------------- |:-------------:|
|MPI_GRAPH|Topologi graph|
|MPU_CART|Topologi kartesian|
|MPI_DIST_GRAPH|Topologi distribusi|
|MPI_UNDEFINED|Tidak ada topologi|

### Pengujian topologi kartesian

Pada topologi kartesian, kita dapat melakukan pengujian yang memang sudah disediakan oleh MPI. Pengujian pertama adalah memperoleh jumlah dimensi. Hal ini dapat dilakukan dengan ``MPI_Cartdim_get()`` dan ``MPI_Cart_get()``.

```c++
int MPI_Cartdim_get(MPI_Comm comm, int *ndims)

int MPI_Cart_get(MPI_Comm comm, int maxdims, int *dims, int *periodes, int *coords)
```
Fungsi mengembalikan jumlah dimensi dari sebuah communicator berbasis topologi kartesian.

Pengujian kedua, kita dapat memperoleh rank dari input koordinasi pada topologi kartesian. Ini dapat dilakukan dengan menggunakan fungsi ``MPI_Cart_rank()`` yang dideklarasikan sebagai berikut:

int MPI_Cart_rank(MPI_Comm comm, int *cords, int *rank)

Pengujian terakhir, kita ingin memperoleh koordinasi pada topologi kartesian dari input rank. Hal ini dapat dilakukan ``MPI_Cart_coords()`` yang dideklarasikan sebagai berikut:
```c++
int MPI_Cart_coords(MPI_Comm comm, int rank, int maxdims, int *coords)
```

| Parameter | Keterangan  |
| ------------- |:-------------:|
|comm|input communicator|
|rank|rank|
|maxdims|panjang koordinasi|
|coords|output koordinasi|

```c++
//cartesian_test.c

#include <mpi.h>
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char *argv[])
{
	int i,topo_type;
    int dims[2], periods[2], size,rank;
    int coord[2],outdims[2], outperiods[2], outcoords[2];    
    int *outindex, *outedges;
    MPI_Comm comm1, comm2;

    MPI_Init( &argc, &argv );
	MPI_Comm_rank(MPI_COMM_WORLD, &rank);
    MPI_Comm_size( MPI_COMM_WORLD, &size );
   
    dims[0] = dims[1] = 0;
    MPI_Dims_create(size, 2, dims);

    periods[0] = periods[1] = 0;
    MPI_Cart_create(MPI_COMM_WORLD, 2, dims, periods, 0, &comm1);
    MPI_Comm_dup( comm1, &comm2 );

	MPI_Cart_coords(comm2, rank, 2, coord);
	printf("Rank %d posisi(%d, %d)\r\n", rank, coord[0], coord[1]);

    MPI_Topo_test( comm2, &topo_type );  
	if (topo_type == MPI_CART)
    {
		outindex = (int*)malloc(size*sizeof(int));
		outedges = (int*)malloc(size*2*sizeof(int));

        MPI_Cart_get( comm2, 2, outdims, outperiods, outcoords);
        for (i=0; i<2; i++) 
		{
            if (outdims[i] != dims[i]) 
				printf("Rank %d -->%d = outdims[%d] != dims[%d] = %d\n",rank, outdims[i], i, i, dims[i] );
			else
				printf("Rank %d -->%d = outdims[%d] == dims[%d] = %d\n", rank,outdims[i], i, i, dims[i] );

            if (outperiods[i] != periods[i]) 
				printf("Rank %d -->%d = outperiods[%d] != periods[%d] = %d\n", rank,outperiods[i], i, i, periods[i]);
			else
				printf("Rank %d -->%d = outperiods[%d] == periods[%d] = %d\n", rank,outperiods[i], i, i, periods[i]);
        }
    }
	    
    MPI_Finalize();

    return 0;
}


```

