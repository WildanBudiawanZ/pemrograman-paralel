# Bekerja Dengan I/O

Sumber : Agus Kurniawan. 2010. Pemrograman Paralel dengan MPI dan C. Andi (197-226)

## Manipulasi file

Pada kasus tertentu program yang kita buat dengan memanfaatkan MPI ingin mengakses data pada suatu file, baik untuk membaca ataupun menulis. File merupakan salah satu media untuk menyimpan data selain pada menggunakan memori. Memanipulasi file pada MPI secara garis besar konsepnya sama seperti pada manipulasi file pada aplikasi bukan MPI. Manipulasi file pada MPI adalah serangkaian proses untuk melakukan pengolahan data yang meliputi:
1. membuka dan menutup file
2. menulis dan membaca file
3. menghapus file

### Membuka dan menutup file

MPI menyediakan mekanisme membuka dan menutup file yakni menggunakan ``MPI_File_open()`` dan menutupnya menggunakan ``MPI_File_close()``.
```c++
int MPI_File_open(MPI_Comm comm, char *filename, int amode, MPI_Info info, MPI_File *fh)

int MPI_File_close(MPI_File *fh)
```

| Parameter | Keterangan  |
| ------------- |:-------------:|
|comm  |coomunicator |
|filename|nama file|
|amode|mode file akses|
|info|info objek|
|fh|out file handle|

pada fungsi ``MPI_File_open()`` kita memerlukan parameter mode file akses sebagai berikut:
| Parameter | Keterangan  |
| ------------- |:-------------:|
|MPI_MODE_RDONLY|hanya membaca|
|MPI_MODE_RDWR|baca dan tulis|
|MPI_MODE_WRONLY|hanya menulis|
|MPI_MODE_CREATE|membuat file jika tidak ada file|
|MPI_MODE_EXCL|error ketika pembuatan file dimana file sudah ada sebelumnya|
|MPI_MODE_DELETE_ON_CLOSE|hapus file ketika operasi menutup file|
|MPI_MODE_UNIQUE_OPEN|file tidak bisa dibuka secara paralel|
|MPI_MODE_SEQUENTIAL|file hanya dapat diakses secara sequential|
|MPI_MODE_APPEND|menentukan posisi semua pointer file pada akhir data file|


```c++
manipulasi_file.c


#include <mpi.h>
#include <stdio.h>

int main( int argc, char *argv[] )
{   
    int rc;
    int rank;
    MPI_File hdFile;
	MPI_Offset offset;

    MPI_Init(&argc,&argv);
    MPI_Comm_rank(MPI_COMM_WORLD, &rank);
    rc = MPI_File_open( MPI_COMM_WORLD, 
			"c:/temp/data.txt", 
			MPI_MODE_RDONLY,	
			MPI_INFO_NULL, &hdFile);

    if (rc) 
		printf("tidak dapat membuka c:/temp/data.txt\r\n");
    else 
	{
		MPI_File_get_size(hdFile,&offset);
		printf("Rank %d: File dapat dibuka dengan size %d bytes\r\n",rank,offset);
		MPI_File_close(&hdFile);
	}

    MPI_Finalize();
    return 0;
}


```

### Menghapus file

## Informasi file

## Akses data file

### Metode Eksplisit Offset
