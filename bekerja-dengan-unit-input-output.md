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
buat dan sesuaikan nama file dan direktori

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

Kita dapat menghapus file pada lingkungan MPI dengan menggunakan ``MPI_File_delete()``.
```c++
int MPI_File_delete(char *filename, MPI_Info info)
```

```c++
#include <mpi.h>
#include <stdio.h>

int main( int argc, char *argv[] )
{   
    int rc;
    int rank;
    MPI_File hdFile;

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
		printf("Rank %d: File dapat dibuka\r\n",rank);
		MPI_File_close(&hdFile);

		if (rank == 0) 
		{
            rc = MPI_File_delete("c:/temp/data.txt", MPI_INFO_NULL);
            if (rc) 
				printf("tidak dapat menghapus file c:/temp/data.txt\r\n");
			else
				printf("File c:/temp/data.txt sudah dihapus\r\n");
        }

	}

    MPI_Finalize();
    return 0;
}

```

## Informasi file

Setiap file akan mempunyai baik informasi yang diberikan oleh sistem operasi mapupun informasi yang diberikan oleh pengguna. Pada MPI kita memperoleh informasi file dengan memanfaatkan fungsi ``MPI_File_get_info()``. Sedangkan untuk menulis informasi kedalam file dapat dilakukan dengan menggunakan fungsi ``MPI_File_set_info()``. Dalam memberikan nilai pada file, MPI mempunyai data informasi yang memang sudah dipakai sehingga kita tidak boleh membuat informasi yang sama. Informasi itu adalah:
1. access_style
2. collective_buffering
3. cb_block_size
4. cb_buffer_size
5. cb_nodes
6. chunked
7. chunked_item
8. chunked_size
9. filename
10. file_perm
11. io_node_list
12. nb_proc
13. num_io_nodes
14. striping_factor
15. striping_unit

```c++
infofile.c


#include <mpi.h>
#include <stdio.h>

int main( int argc, char *argv[] )
{   
    int rc,flag,rank,i,nkeys;
    MPI_File hdFile;
	MPI_Info infoin, infoout;	
	char key[MPI_MAX_INFO_KEY], value[MPI_MAX_INFO_VAL];


    MPI_Init(&argc,&argv);
    MPI_Comm_rank(MPI_COMM_WORLD, &rank);
	rc = MPI_File_open( MPI_COMM_WORLD, 
			"c:/temp/testfile.txt", 
			MPI_MODE_CREATE | MPI_MODE_RDWR,
			MPI_INFO_NULL, &hdFile);
	if (rc) 
		printf("tidak dapat membuka c:/temp/testfile.txt\r\n");
	else 
	{
		printf("Rank %d: File dapat dibuka\r\n",rank);
		if(rank==0)
		{			
			MPI_File_get_info(hdFile, &infoout);	
			MPI_Info_get_nkeys(infoout, &nkeys);

			printf("Nilai key dan value pada file\r\n");
			for (i = 0; i < nkeys; i++) 
			{
			   MPI_Info_get_nthkey(infoout, i, key);
			   MPI_Info_get(infoout, key, MPI_MAX_INFO_VAL, value, &flag);
			    if (flag)
					 printf("   key = %s, value = %s\n",key, value);
			}
			
			printf("Modifikasi nilai cb_buffer_size\r\n");	
			MPI_Info_create(&infoin);
			MPI_Info_set(infoin, "cb_buffer_size", "4096" );
			MPI_File_set_info(hdFile,infoin);

			printf("Nilai key dan value hasil modifikasi pada file\r\n");
			MPI_File_get_info(hdFile, &infoout);	
			MPI_Info_get_nkeys(infoout, &nkeys);

			for (i = 0; i < nkeys; i++) 
			{
			   MPI_Info_get_nthkey(infoout, i, key);
			   MPI_Info_get(infoout, key, MPI_MAX_INFO_VAL, value, &flag);
			    if (flag)
					 printf("   key = %s, value = %s\n", key, value);
			}
			MPI_Info_free(&infoin);	
			MPI_Info_free(&infoout);
		}		
		
		MPI_File_close(&hdFile);
	}

    MPI_Finalize();
    return 0;
}


```

## File View



## Akses data file

### Metode Eksplisit Offset
