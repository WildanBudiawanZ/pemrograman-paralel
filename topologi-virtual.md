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
