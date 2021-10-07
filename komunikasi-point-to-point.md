# Komunikasi Point to Point

Sumber : Agus Kurniawan. 2010. Pemrograman Paralel dengan MPI dan C. Andi (79-119)

Mekanisme dasar sistem komunikasi pada MPI adalah proses bertukaran data pada sepasang proses dimana satu sebagai pengirim dan satunya sebagai penerima. Hampir sebagian besar komunikasi yang terjadi di MPI didasarkan pada komunikasi point-to-point. Sehingga komunikasi ini sangatlah penting dan dasar untuk komunikasi MPI.
MPI menyediakan beberapa fungsi untuk mengirim dan menerima data baik secara blocking atau non-blocking.
Blocking send/receive adalah proses yang tidak akan mengembalikan nilai sampai buffer sudah penuh dengan data yang akan dikirim/diterima. Artinya, kode selanjutnya setelah blocking send/receive tidak akan dieksekusi bila proses blocking send/receive belum selesai.
Sedangkan non-blocking akan mengembalikan nilai walaupun data yang dikirm/diterima belum dieksekusi.

Pada MPI, semua komunikasi dieksekusi oleh ``communicator``. Sebuah ``communicator`` mewakili sebuah komunikasi domain yang merupakan kumpulan proses yang saling tukar-menukar data. ``MPI_COMM_WORLD`` komunikasi domain yang umum digunakan di MPI.

## Model Komunikasi MPI

1. Model standar

Banyak digunakan di MPI, MPI menentukan apakah message yang keluar akan menggunakan buffer di lokal sistem atau tidak. Bagi programmer berarti tidak dapat memaksa untuk menggunakan buffer atau tidak.

2. Model sinkronus
Operasi pengiriman akan dianggap selesai bila operasi penerima telah selesai dieksekusi dan sudah menerima data yang dikirim.

3. Model buffer
Operasi pengiriman yang terjadi dilokal termasuk proses selesainya tidak dipengaruhi oleh kejadian diluar lokal.


## Operasi Blocking MPI

Blocking send/receive tidak akan mengembalikan nilai sampai buffer sudah penuh dengan data yang akan dikirim/diterima.

```cpp
point2point.c

#include <mpi.h>
#include <stdio.h>
#include <string.h>

int main(int argc, char* argv[]) 
{
	char message[30];
	int myrank;
	MPI_Status status;
	MPI_Init( &argc, &argv );
	MPI_Comm_rank( MPI_COMM_WORLD, &myrank );
	printf("Proses rank: %d \r\n", myrank);

	if (myrank == 0) 
	{
		strcpy(message,"Halo ini dari rank 0");
		printf("Data dikirim dari rank 0: %s \r\n", message);
		MPI_Send(message, strlen(message)+1, MPI_CHAR, 1, 99, MPI_COMM_WORLD);
	}
	else if (myrank == 1) 
	{
		MPI_Recv(message, 30, MPI_CHAR, 0, 99, MPI_COMM_WORLD, &status);
		printf("Data yang diterima di rank 1: %s \r\n", message);
	}
	MPI_Finalize();
	return 0;
}
```

### MPI_Send()

| Parameter | Keterangan  |
| ------------- |:-------------:|
| buf | Buffer data yang akan dikirim |
| count | Jumlah buffer data |
| datatype | Tipe data dari buffer data yang akan dikirim |
| dest | Tujuan rank |
| tag | Message tag 0 - 32767 |
| comm | Communicator yang digunakan |

### MPI_Recv()

| Parameter | Keterangan  |
| ------------- |:-------------:|
| buf | Buffer data yang akan diterima |
| count | Jumlah buffer data |
| datatype | Tipe data dari buffer data yang akan diterima |
| source | Sumber rank |
| tag | Message tag 0 - 32767 |
| comm | Communicator yang digunakan |
| status | Status yang telah terjadi pada proses penerimaan data |

### MPI_Status

```cpp
typedef struct MPI_Status {
	int count;
	int cancelled;
	int MPI_Source;
	int MPI_Tag;
	int MPI_Error;
} MPI_Status; 
 ```
 
 ```cpp
 int MPI_Get_count()
 ```
 
| Parameter | Keterangan  |
| ------------- |:-------------:|
| status | Status yang telah terjadi pada proses penerimaan data |
| datatype | Tipe data dari buffer data yang akan dikirim |
| count | Jumlah buffer data |

```cpp
point2point_status.c

#include <mpi.h>
#include <stdio.h>
#include <string.h>

int main(int argc, char* argv[]) 
{
	char message[30];
	int myrank;
	MPI_Status status;
	int total;
	
	MPI_Init( &argc, &argv );
	MPI_Comm_rank( MPI_COMM_WORLD, &myrank );
	printf("Proses rank: %d \r\n", myrank);

	if (myrank == 0) 
	{
		strcpy(message,"Halo ini dari rank 0");
		printf("Data dikirim dari rank 0: %s \r\n", message);
		MPI_Send(message, strlen(message)+1, MPI_CHAR, 1, 99, MPI_COMM_WORLD);
	}
	else if (myrank == 1) 
	{
		MPI_Recv(message, 30, MPI_CHAR, 0, 99, MPI_COMM_WORLD, &status);
		printf("Data yang diterima di rank 1: %s \r\n", message);
	}

	MPI_Get_count(&status, MPI_CHAR, &total);
	printf("Total: %d\r\n",total);
	printf("Tag: %d\r\nSource: %d\r\nError: %d\r\nCount: %d\r\nCancelled: %d\r\n", 
		status.MPI_TAG,status.MPI_SOURCE,status.MPI_ERROR,status.count,status.cancelled);

	MPI_Finalize();
	return 0;
}

```

### MPI_Buffer

Pada saat mengirim data melalui MPI_Send, harus mendefinisikan buffer data dan panjangnya yang akan digunakan. Hanya satu buffer yang dapat digunakan pada proses yang berjalan.
```cpp
MPI_Buffer_attach(buffer, size)
MPI_Buffer_detach(buffer_addr, size)
```


| Parameter | Keterangan  |
| ------------- |:-------------:|
| buffer | alamat buffer data yang akan digunakan |
| size | ukurang jumlah buffer |
| buffer_addr | alamat buffer data yang digunakan |

bila menggunakan ``MPI_Buffer_attach()`` dan ``MPI_Buffer_detach()`` maka proses mengirim datanya memerlukan mode buffer ``MPI_Bsend()``.

```cpp
MPI_Bsend()
```

| Parameter | Keterangan  |
| ------------- |:-------------:|
| buf | buffer data yang akan dikirim |
| count | jumlah buffer data |
| datatype | tipe data dari buffer yang akan dikirim |
| dest | tujuan rank |
| tag | message tag 0-32767 |
| comm | communicator yang digunakan |

full code:
```cpp
point2point_buffered.c
#include <mpi.h>
#include <stdlib.h>
#include <math.h>
#include <stdio.h>

#define N 1000
int main(int argc, char *argv[])
{
	int     numtasks, rank, ret=0, i, 
    dest=1, tag=33, source=0, size;
	double  data[N];
	void    *buffer;
	MPI_Status status;
  
	MPI_Init(&argc,&argv);
	MPI_Comm_size(MPI_COMM_WORLD,&numtasks);
	MPI_Comm_rank(MPI_COMM_WORLD,&rank);
  
	if (numtasks != 2) 
	{
		printf("Jumlah proses harus 2\r\n");
		MPI_Finalize();
		return 0;
	}
	printf ("Rank %d\r\n", rank);
  	
	if (rank == 0) 
	{		
		for(i=0; i<N; i++)
			data[i] =  (double)rand();
      
	
		MPI_Pack_size (N, MPI_DOUBLE,MPI_COMM_WORLD, &size);
		size = size +  MPI_BSEND_OVERHEAD;
		printf("Ukuran buffer= %d Overhead %d\n",size, MPI_BSEND_OVERHEAD);
      		
		buffer = (void*)malloc(size);
		ret = MPI_Buffer_attach(buffer, size);
		if (ret != MPI_SUCCESS) 
		{
			printf("Buffer attach gagal. Code= %d\r\n", ret);
		    MPI_Finalize();
			return 0;
		}
		ret = MPI_Bsend(data, N, MPI_DOUBLE, dest, tag,MPI_COMM_WORLD);
		printf("Message telah dikirim. Code= %d\r\n",ret);
		MPI_Buffer_detach(&buffer, &size);
		free (buffer);		
	}	
	if (rank == 1) 
	{
		MPI_Recv(data, N, MPI_DOUBLE, source, tag,MPI_COMM_WORLD, &status);
		printf("Message diterima. Code= %d. Tag=%d. Source=%d\r\n",
			ret,status.MPI_TAG,status.MPI_SOURCE);
		
	}
	MPI_Finalize();
	return 0;
}


```

