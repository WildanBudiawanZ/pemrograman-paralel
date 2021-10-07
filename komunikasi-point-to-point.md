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

### Demo Opearasi Blocking MPI

Pada demo ini, komunikasi MPI diperkaya dengan memanfaatkan operasi blocking. Pada ilustrasi demo ini kita akan aplikasi PING yang dilakukan pada rank 0 dan 1.

```cpp
#include <mpi.h>
#include <stdio.h>
#include <string.h>

int main(int argc, char* argv[]) 
{	
	int rank, dest, source, ret, total, tag=1;  
	char inmsg[5], outmsg[5];
	MPI_Status status;

	strcpy(outmsg,"mpi");

	MPI_Init(&argc,&argv);	
	MPI_Comm_rank(MPI_COMM_WORLD, &rank);

	if (rank == 0) 
	{
		dest = 1;
		source = 1;
		ret = MPI_Send(&outmsg, strlen(outmsg)+1, MPI_CHAR, dest, tag, MPI_COMM_WORLD);
		ret = MPI_Recv(&inmsg, 5, MPI_CHAR, source, tag, MPI_COMM_WORLD, &status);
	} 
	else if (rank == 1) 
	{
		dest = 0;
		source = 0;
		ret = MPI_Recv(&inmsg, 5, MPI_CHAR, source, tag, MPI_COMM_WORLD, &status);
		ret = MPI_Send(&outmsg, strlen(outmsg)+1, MPI_CHAR, dest, tag, MPI_COMM_WORLD);
	}

	ret = MPI_Get_count(&status, MPI_CHAR, &total);
	printf("Rank %d: Total karakter yg diterima %d dari rank %d dengan nilai tag %d \n",
			rank, total, status.MPI_SOURCE, status.MPI_TAG);

	MPI_Finalize();
	return 0;
}
```

## Operasi Non-Blocking MPI

Operasi ini sangat berguna ketika kita bekerja pada lingkungan asynchronous terutama pada aplikasi berbasis multi-thread. Operasi non-blocking MPI dikenali dengan nama operasinya dimana dapat dikategorikan menjadi empat bagian yakni:
1. Synchronous (S/s)
2. Immediate (I/i)
3. Buffer (B/b)
4. Ready (R/r)

### MPI_Isend()

Operasi ini digunakan untuk mengirim data secara non-blocking dan bersifat immmediate.

| Parameter | Keterangan  |
| ------------- |:-------------:|
| buf | buffer data yang dikirim |
| count| jumlah buffer data |
| datatype| tipe data buffer |
| dest| tujuan rank |
| tag| message tag 0-32767 |
| comm| communicator |
| request| output dari komunikasi |

### MPI_Ibsend()

Operasi ini digunakan untuk mengirim data secara non-blocking dan bersifat immediate dan menggunakan buffer.

| Parameter | Keterangan  |
| ------------- |:-------------:|
| buf | buffer data yang dikirim |
| count| jumlah buffer data |
| datatype| tipe data buffer |
| dest| tujuan rank |
| tag| message tag 0-32767 |
| comm| communicator |
| request| output dari komunikasi |

### MPI_Issend()

Operasi ini digunakan untuk mengirim data secara non-blocking, synchronous, dan bersifat immediate.

| Parameter | Keterangan  |
| ------------- |:-------------:|
| buf | buffer data yang dikirim |
| count| jumlah buffer data |
| datatype| tipe data buffer |
| dest| tujuan rank |
| tag| message tag 0-32767 |
| comm| communicator |
| request| output dari komunikasi |

### MPI_Irsend()

Operasi ini digunakan untuk mengirim data secara non-blocking dan bersifat immediate serta menggunakan mode ready.

| Parameter | Keterangan  |
| ------------- |:-------------:|
| buf | buffer data yang dikirim |
| count| jumlah buffer data |
| datatype| tipe data buffer |
| dest| tujuan rank |
| tag| message tag 0-32767 |
| comm| communicator |
| request| output dari komunikasi |



### MPI_Irecv()

Operasi ini digunakan untuk menerima data secara non-blocking dan bersifat immediate.

| Parameter | Keterangan  |
| ------------- |:-------------:|
| buf | buffer data yang dikirim |
| count| jumlah buffer data |
| datatype| tipe data buffer |
| source | sumber rank |
| tag| message tag 0-32767 |
| comm| communicator |
| request| output dari komunikasi |


### Menunggu Proses Selesai pada Non-Blocking

Semua proses MPI yang memanfaatkan operasi non-blocking adalah operasi asinkronus sehingga perlu mengetahui apakah operasi ini selesai atau belum.

#### MPI_Wait()

Operasi ini digunakan untuk menunggu operasi MPI non-blocking yang dilakukan sebelumnya.

| Parameter | Keterangan  |
| ------------- |:-------------:|
| request | MPI request yang akan ditunggu |
| status | status yang dikeluarkan |

#### MPI_Test()

Operasi ini digunakan untuk menguji apakah operasi MPI itu sudah selesai atau belum.

| Parameter | Keterangan  |
| ------------- |:-------------:|
| request | MPI request yang akan ditest |
| status | status yang dikeluarkan|
| flag | jika true maka operasi sudah selesai else belum selesai |

#### MPI_Request_free()

Operasi ini digunakan untuk memberikan tanda suatu operasi MPI supaya dihapus.

| Parameter | Keterangan  |
| ------------- |:-------------:|
| request | MPI request yang akan dihapus |


#### MPI_Waitany()

Operasi ini digunakan untuk menunggu spesifik atau beberapa operasi non-blocking MPI yang sudah dilakukan sebelumnya.

| Parameter | Keterangan  |
| ------------- |:-------------:|
|count|jumlah operasi non-blocking|
|array_of_requests|kumpulan MPI_Request|
|index|output index dari MPI_Request|
|status| output status|

#### MPI_Waitall()

Operasi ini digunakan untuk menunggu semua operasi non-blocking MPI yang dilakukan sebelumnya.

| Parameter | Keterangan  |
| ------------- |:-------------:|
| count | jumlah operasi non-blocking yang ditunggu |
|array_of_request | kumpulan MPI_Request |
|array_of_status | output kumpulan status|


#### MPI_Testany()

Operasi ini digunakan untuk menguji secara spesifik atau beberapa operasi non-blocking MPI sudah selesai atau belum.

| Parameter | Keterangan  |
| ------------- |:-------------:|
| count | jumlah operasi non-blocking yang ditunggu |
|array_of_request | kumpulan MPI_Request |
|index|output dari index MPI_Request|
|flag| jika true maka sudah selesai else belum selesai|
|status| output status|

#### MPI_Testall()

Operasi ini digunakan untuk menguji semua operasi non-blocking MPI sudah selesai atau belum.

| Parameter | Keterangan  |
| ------------- |:-------------:|
| count | jumlah operasi non-blocking yang ditunggu |
|array_of_request | kumpulan MPI_Request |
|flag| jika true maka sudah selesai else belum selesai|
|array_of_status| kumpulan output status|


### Demo
Pada demo ini, akan dicontohkan penggunaan komunikasi point-to-point dengan menggunakan operasi MPI secara non-blocking. Pada demo ini akan mengirim data dari rank 1 ke rank 0 secara non-blocking yaitu ``MPI_Isend()`` dan menunggu operasi tersebut selesai dengan ``MPI_Wait()``.

```cpp
point2point_nonblocking.c
#include <mpi.h>
#include <stdio.h>
#include <string.h>

int main(int argc, char *argv[]) 
{
    int rank, size;
	char message[10];

    MPI_Init(&argc, &argv);
    MPI_Comm_size(MPI_COMM_WORLD, &size);
    MPI_Comm_rank(MPI_COMM_WORLD, &rank);
   
    if (size != 2) 
	{
        printf("Minimal proses/processor 2\r\n");
		MPI_Finalize();
        return 0;
    }

	printf("Rank %d\r\n", rank);
    if (rank != 0) 
	{
		MPI_Request request;		
        strcpy(message,"DATA-MPI");		
        
        MPI_Isend(message, 10, MPI_CHAR, 0, 99, MPI_COMM_WORLD, &request);       
        MPI_Wait(&request, MPI_STATUS_IGNORE);
        printf("Node %d telah mengirim data ke rank 0\r\n", rank);
    } 
	else 
	{       
        MPI_Recv(message, 10, MPI_CHAR, 1, 99, MPI_COMM_WORLD, MPI_STATUS_IGNORE);
        printf("Node %d menerima data \"%s\" dari node %d\r\n", rank, message, 1);
    }

    MPI_Finalize();
    return 0;
}
```

## MPI_Sendrecv()

MPI menyediakan operasi pengiriman dan penerimaan data secara bersamaan menggunakan ``MPI_Sendrecv()``.

| Parameter | Keterangan  |
| ------------- |:-------------:|
| sendbuf| data yang dikirim| 
| sendcount| jumlah data yang dikirim| 
| sendtype| tipe data yang dikirim| 
| dest| tujuan rank| 
| sendtag| message tag 0-32767| 
| recvbuf| data yang diterima| 
| recvcount| jumlah data yang diterima| 
| recvtype| tipe data yang diterima| 
| source| sumber rank| 
| recvtag| message tag 0-32767| 
| comm| coomunicator yang digunakan | 
| status| status yang dikeluarkan| 

untuk variabel kirim dan terima yang sama, dapat menggunakan ``MPI_Sendrecv_replace()``
| Parameter | Keterangan  |
| ------------- |:-------------:|
| buf| data yang dikirim dan diterima| 
| count| jumlah data yang dikirim dan diterima| 
| type| tipe data yang dikirim dan diterima| 
| dest| tujuan rank| 
| sendtag| message tag 0-32767| 
| source| sumber rank|
| recvtag| message tag 0-32767|
| comm| communicator yang digunakan|
| status| status yang dikeluarkan|

```cpp
#include <mpi.h>
#include <stdio.h>
#include <string.h>

int main(int argc, char *argv[]) 
{
	MPI_Status status;
    int rank, size;
	char sendMessage[30];
	char recvMessage[30];

    MPI_Init(&argc, &argv);
    MPI_Comm_size(MPI_COMM_WORLD, &size);
    MPI_Comm_rank(MPI_COMM_WORLD, &rank);
   
    if (size != 2) 
	{
        printf("Minimal proses/processor 2\r\n");
		MPI_Finalize();
        return 0;
    }

	printf("Rank %d\r\n", rank);
    if (rank == 0) 
	{		
        strcpy(sendMessage,"DATA-MPI dari rank 0");
		strcpy(recvMessage,"");
        
		MPI_Sendrecv(sendMessage,30,MPI_CHAR,1,99,recvMessage,30,MPI_CHAR,1,99,MPI_COMM_WORLD,&status);
        printf("Node %d telah mengirim data ke rank 1\r\n", rank);
		printf("Data yang diterima \"%s\" dari rank 1\r\n", recvMessage);
    } 
	if (rank == 1) 
	{
		strcpy(sendMessage,"DATA-MPI dari rank 1");
		strcpy(recvMessage,"");
        
		MPI_Sendrecv(sendMessage,30,MPI_CHAR,0,99,recvMessage,30,MPI_CHAR,0,99,MPI_COMM_WORLD,&status);
        printf("Node %d telah mengirim data ke rank 0\r\n", rank);
		printf("Data yang diterima \"%s\" dari rank 0\r\n", recvMessage);
    }

    MPI_Finalize();
    return 0;
}

```

