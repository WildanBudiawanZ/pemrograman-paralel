# Komunikasi Kolektif

Sumber : Agus Kurniawan. 2010. Pemrograman Paralel dengan MPI dan C. Andi (121-169)

## Mengenal Komunikasi Kolektif

Komunikasi kolektif dapat didefinisikan sebagai sebuah komunikasi yang melibatkan sebuah group atau kumpulan group dari proses. MPI menjamin message yang dibuat oleh komunikasi kolektif tidak akan tercampur denga message yang dibuat pada komunikasi point-to-point.

## Sinkronisasi Barrier

Komunikasi yang terjadi di MPI sangat kompleks terlebih bila kita bekerja pada asinkronus. Oleh karena itu, kita memerlukan sinkronisasi antar proses yang terjadi. Hal ini dapat dilakukan dengan menggunakan ``MPI_Barrier()`` yang dideklarasikan sebagai berikut:

```c
int MPI_Barrier(MPI_Comm comm)
```

Variabel ``comm`` adalah communicator dari sebuah group/

``MPI_Barrier()`` akan melakukan blocking semua proses yang terjadi sampai semua proses pada group comm sudah mencapai ``MPI_Barrier()``. Setelah semua proses mencapai ``MPI_Barrier()``, proses akan dilanjutkan seperti biasa.

```c
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

## Broadcast

Operasi broadcast terjadi ketika suatu proses dalam suatu spesifik group mengirim ke seluruh proses pada spesifik group. Operasi broadcast dapat dilakukan di MPI dengan memanfaatkan ``MPI_Bcast()`` yang dideklarasikan sebagai berikut:
```c
int MPI_Bcast(void*, buffer, int count, MPI_Datatype datatype, int root, MPI_Comm comm)
```

| Parameter | Keterangan  |
| ------------- |:-------------:|
| buffer | Data yang akan di-broadcast pada suatu group comm |
| count | Jumlah data |
| datatype | Tipe data |
| root | Rank dari broadcast |
| comm | Communicator yang digunakan |

```c
collective_bcast.c
#include <mpi.h>
#include <stdio.h>

int main(int argc, char* argv[]) 
{	
	int rank,val;
	
	MPI_Init( &argc, &argv );
	MPI_Comm_rank( MPI_COMM_WORLD, &rank );
	printf("Proses rank: %d \r\n", rank);
	
	if(rank==0)
		val = 100; 

	MPI_Bcast( &val, 1, MPI_INT, 0, MPI_COMM_WORLD);	
	printf("Rank %d, Total val = %d\r\n",rank,val);
	
	MPI_Finalize();
	return 0;
}
```

## Gather and Scatter

Gather adalah proses pengambilan data dari semua proses ke suatu proses pada spesifik komunikator. Sedangkan Scatter adalah proses pengiriman data dari satu proses ke semua proses pada spesifik komunikator.
Gather:
```c
int MPI_Gather(void* sendbuf, int sendcount, MPI_Datatype sendtype, void* recvbuf, int recvcount, MPI_Datatype recvtype, int root, MPI_Comm comm)
```

| Parameter | Keterangan  |
| ------------- |:-------------:|
| sendbuf | Buffer yang akan dikirim |
| sendcount | Jumlah data buffer yang dikirim |
| sendtype | Tipe data buffer yang akan dikirim |
| recvbuf | Buffer untuk menerima |
| recvcount | Jumlah buffer yang diterima |
| recvtype | Tipe buffer yang diterima |
| root | Rank yang menerima |
| comm | Communicator yang digunakan |

Scatter:
```c
int MPI_Scatter(void* sendbuf, int sendcount, MPI_Datatype sendtype, void* recvbuf, int recvcount, MPI_Datatype recvtype, int root, MPI_Comm comm)
```

| Parameter | Keterangan  |
| ------------- |:-------------:|
| sendbuf | Buffer yang akan dikirim |
| sendcount | Jumlah data buffer yang dikirim |
| sendtype | Tipe data buffer yang akan dikirim |
| recvbuf | Buffer untuk menerima |
| recvcount | Jumlah buffer yang diterima |
| recvtype | Tipe buffer yang diterima |
| root | Rank yang menerima |
| comm | Communicator yang digunakan |

```c
collective_scatter_gather.c

#include <mpi.h>
#include <stdio.h>
#include <stdlib.h>

int numnodes;
int main(int argc, char* argv[]) 
{	
	int rank;
	int *myray,*send_array,*back_array;
	int count;
	int size,i,total;
	
	MPI_Init( &argc, &argv );
	MPI_Comm_rank( MPI_COMM_WORLD, &rank );
	MPI_Comm_size( MPI_COMM_WORLD, &numnodes );
	
	count=10;
	size=count*numnodes;
	myray=(int*)malloc(count*sizeof(int));
	send_array=(int*)malloc(size*sizeof(int));
	back_array=(int*)malloc(numnodes*sizeof(int));

	if(rank==0)
	{		
		for(i=0;i<size;i++)
			send_array[i]=i;
	}
	
	MPI_Scatter(send_array, count,MPI_INT,
			        myray, count,MPI_INT,
	                0, MPI_COMM_WORLD);

	total=0;
	for(i=0;i<count;i++)
		total=total+myray[i];

	printf("Rank= %d Total= %d \r\n",rank,total);

    MPI_Gather(&total, 1, MPI_INT, 
					back_array, 1,  MPI_INT, 
	                0, MPI_COMM_WORLD);
	if(rank == 0)
	{
		total=0;
		for(i=0;i<numnodes;i++)
			total = total + back_array[i];

		printf("Total dari proses= %d \r\n ",total);
	}
	
	MPI_Finalize();
	return 0;
}
```

## GatherV and ScatterV

GatherV dan ScatterV merupakan pengembangan dari MPI Gather dan Scatter dimana data yang dikirim atau diterima dapat bervariasi jumlahnya.


GatherV:
```c
int MPI_Gatherv(void* sendbuf, int sendcount, MPI_Datatype sendtype, void* recvbuf, int *recvcounts, int *displs, MPI_Datatype recvtype, int root, MPI_Comm comm)
```

| Parameter | Keterangan  |
| ------------- |:-------------:|
| sendbuf | Buffer yang akan dikirim |
| sendcount | Jumlah data buffer yang dikirim |
| sendtype | Tipe data buffer yang akan dikirim |
| recvbuf | Buffer untuk menerima |
| recvcounts | Jumlah buffer yang diterima |
| displs | Array integer untuk data yang diterima |
| recvtype | Tipe buffer yang diterima |
| root | Rank yang menerima |
| comm | Communicator yang digunakan |

```c
collective_gatherv,c

#include <mpi.h>
#include <stdio.h>
#include <stdlib.h>

int numnodes,rank;
int main(int argc, char* argv[]) 
{			
	int *myray,*disp,*counts,*allray;
	int size,data,i;
	size=0;	

	MPI_Init( &argc, &argv );
	MPI_Comm_rank( MPI_COMM_WORLD, &rank );
	MPI_Comm_size( MPI_COMM_WORLD, &numnodes );
	
	data=rank+1;
	myray=(int*)malloc(data*sizeof(int));
	for(i=0;i<data;i++)
		myray[i]=rank+1;

	counts=(int*)malloc(numnodes*sizeof(int));
	disp=(int*)malloc(numnodes*sizeof(int));	
	disp[0]=0;

	printf("Rank= %d, data= %d \r\n",rank,data);
	MPI_Gather((void*)myray,1,MPI_INT, 
				counts, 1,MPI_INT, 
				0,MPI_COMM_WORLD);

	if(rank == 0)
	{				
		for( i=1;i<numnodes;i++)
			disp[i]=counts[i-1]+disp[i-1];
				
		for(i=0;i< numnodes;i++)
			size=size+counts[i];
		
	}	
	allray=(int*)malloc(size*sizeof(int));
	MPI_Gatherv(myray, data, MPI_INT, 
	             allray,counts,disp,MPI_INT, 
	             0, MPI_COMM_WORLD);
	                
	if(rank == 0)
	{		
		printf(">> ");
		for(i=0;i<size;i++)
			printf("%d ",allray[i]);
		printf("\r\n");
	}

	MPI_Finalize();
	return 0;
}

```

ScatterV:
```c
int MPI_ScatterV(void* sendbuf, int *sendcounts, int *displs, MPI_Datatype sendtype, void* recvbuf, int recvcount, MPI_Datatype recvtype, int root, MPI_Comm comm)
```

| Parameter | Keterangan  |
| ------------- |:-------------:|
| sendbuf | Buffer yang akan dikirim |
| sendcounts | Jumlah data buffer yang dikirim |
| displs | Array integer untuk data yang dikirim |
| sendtype | Tipe data buffer yang akan dikirim |
| recvbuf | Buffer untuk menerima |
| recvcount | Jumlah buffer yang diterima |
| recvtype | Tipe buffer yang diterima |
| root | Rank yang menerima |
| comm | Communicator yang digunakan |

```c
collective_scatterv.c

#include <mpi.h>
#include <stdio.h>
#include <stdlib.h>

int numnodes,rank;
int main(int argc, char* argv[]) 
{		
	int *sray,*disp,*counts,*allray;
	int size,data,i;
	size=0;

	MPI_Init( &argc, &argv );
	MPI_Comm_rank( MPI_COMM_WORLD, &rank );
	MPI_Comm_size( MPI_COMM_WORLD, &numnodes );
	
	data=rank+1;
	
	counts=(int*)malloc(numnodes*sizeof(int));
	disp=(int*)malloc(numnodes*sizeof(int));	
	sray=(int*)malloc(size*sizeof(int));

	disp[0]=0;	

	printf("Rank= %d data= %d \r\n",rank,data);
	MPI_Gather(&data,1,MPI_INT, 
				counts,1,MPI_INT, 
				0,MPI_COMM_WORLD);

	if(rank == 0)
	{		
		for( i=1;i<numnodes;i++)
			disp[i]=counts[i-1]+disp[i-1];
			
		size = 0;
		for(i=0;i< numnodes;i++)
			size=size+counts[i];
		
		sray=(int*)malloc(size*sizeof(int));
		for(i=0;i<size;i++)
		{
		    sray[i]=i+1;
		}
	}

	allray=(int*)malloc(sizeof(int)*data);
	MPI_Scatterv(sray,counts,disp,MPI_INT, 
	                 allray, data,MPI_INT,
	                 0,MPI_COMM_WORLD);
	         
	if(rank !=0)
	{
		printf("Rank %d >> ",rank);
		for(i=0;i<data;i++)
			printf("%d ",allray[i]);
		printf("\r\n");
	}

	MPI_Finalize();
	return 0;
}

```

## Gather-to-all

Pada MPI Gather-to-all, semua proses akan menerima data.
```c
int MPI_Allgather()
```

| Parameter | Keterangan  |
| ------------- |:-------------:|
| sendbuf | Buffer yang akan dikirim |
| sendcount | Jumlah data buffer yang dikirim |
| sendtype | Tipe data buffer yang akan dikirim |
| recvbuf | Buffer untuk menerima |
| recvcount | Jumlah buffer yang diterima |
| recvtype | Tipe buffer yang diterima |
| comm | Communicator yang digunakan |

```c
collective_allgather.c

#include <mpi.h>
#include <stdio.h>
#include <stdlib.h>

int numnodes;
int main(int argc, char* argv[]) 
{	
	int rank;
	int *back_array;	
	int i,data;
	
	MPI_Init( &argc, &argv );
	MPI_Comm_rank( MPI_COMM_WORLD, &rank );
	MPI_Comm_size( MPI_COMM_WORLD, &numnodes );	
		
	data = rank + 1;
	printf("Rank= %d, Data= %d \r\n",rank,data);

	MPI_Barrier(MPI_COMM_WORLD);
	back_array = (int *)malloc(numnodes*sizeof(int));
    MPI_Allgather(&data, 1, MPI_INT, 
					back_array, 1,  MPI_INT, 
	                MPI_COMM_WORLD);

	MPI_Barrier(MPI_COMM_WORLD);

	printf("Gather>> ");
	for(i=0;i<numnodes;i++)
		printf("%d ",back_array[i]);
	printf("\r\n");	
	
	MPI_Finalize();
	return 0;
}
```

## All-to-all Scatter and Gather

Proses MPI all-to-all merupakan pengembangan dari MPI Gather-to-all, tetapi dapat dipilih data yang akan dikirim.
```c
int MPI_Alltoall()
```

| Parameter | Keterangan  |
| ------------- |:-------------:|
| sendbuf | Buffer yang akan dikirim |
| sendcount | Jumlah data buffer yang dikirim |
| sendtype | Tipe data buffer yang akan dikirim |
| recvbuf | Buffer untuk menerima |
| recvcount | Jumlah buffer yang diterima |
| recvtype | Tipe buffer yang diterima |
| comm | Communicator yang digunakan |

```c
collective_alltoall.c

#include <mpi.h>
#include <stdio.h>
#include <stdlib.h>

int numnodes;
int main(int argc, char* argv[]) 
{	
	int rank,i;		
	int *data_send,*data_recv;	
	float num;
	
	MPI_Init( &argc, &argv );
	MPI_Comm_rank( MPI_COMM_WORLD, &rank );
	MPI_Comm_size( MPI_COMM_WORLD, &numnodes );	

	data_send=(int*)malloc(sizeof(int)*numnodes);
	data_recv=(int*)malloc(sizeof(int)*numnodes);
		
	srand(rank);
	for(i=0;i<numnodes;i++)
	{
		num = (float)rand()/RAND_MAX;
		data_send[i]=(int)(10.0*num)+1;
	}

	printf("Rank= %d Data dikirim  =",rank);
	for(i=0;i<numnodes;i++)
		printf("%d ",data_send[i]);
	printf("\r\n");
	
	MPI_Alltoall(data_send,1,MPI_INT,
			      data_recv,1,MPI_INT,
	              MPI_COMM_WORLD);

	printf("Rank= %d Data diterima =",rank);
	for(i=0;i<numnodes;i++)
		printf("%d ",data_recv[i]);
	printf("\r\n");
	
	MPI_Finalize();
	return 0;
}
```


## Operasi All-to-Allv

Operasi all-to-allv sama seperti all-to-all, namun operasi all-to-allv memberikan fleksibilitas terhadap lokasi data yang diterima.
```c
int MPI_Alltoallv()
```

```c
collective_alltoallv.c

#include <mpi.h>
#include <stdio.h>
#include <stdlib.h>

int numnodes;
int main(int argc, char* argv[]) 
{	
	int rank,i;	
	float num;
	int *sray,*rray;
	int *sdisp,*send_data,*rdisp,*recv_data;
	int ssize,rsize;
	
	MPI_Init( &argc, &argv );
	MPI_Comm_rank( MPI_COMM_WORLD, &rank );
	MPI_Comm_size( MPI_COMM_WORLD, &numnodes );	
	
	send_data=(int*)malloc(sizeof(int)*numnodes);
	recv_data=(int*)malloc(sizeof(int)*numnodes);
	sdisp=(int*)malloc(sizeof(int)*numnodes);
	rdisp=(int*)malloc(sizeof(int)*numnodes);
		
	srand(rank);
	for(i=0;i<numnodes;i++)
	{
		num = (float)rand()/RAND_MAX;
		send_data[i]=(int)(10.0*num)+1;
	}

	printf("rank= %d, send_data= ",rank);
	for(i=0;i<numnodes;i++)
		printf("%d ",send_data[i]);
	printf("\r\n");

	MPI_Alltoall(send_data,1,MPI_INT,
					recv_data,1,MPI_INT,
	                MPI_COMM_WORLD);

	printf("rank= %d, recv_data= ",rank);
	for(i=0;i<numnodes;i++)
		printf("%d ",recv_data[i]);
	printf("\r\n");
	sdisp[0]=0;
	for(i=1;i<numnodes;i++)
	{
		sdisp[i]=send_data[i-1]+sdisp[i-1];
	}
	rdisp[0]=0;
	for(i=1;i<numnodes;i++)
	{
		rdisp[i]=recv_data[i-1]+rdisp[i-1];
	}
	ssize=0;
	rsize=0;
	for(i=0;i<numnodes;i++)
	{
		ssize=ssize+send_data[i];
		rsize=rsize+recv_data[i];
	}

	sray=(int*)malloc(sizeof(int)*ssize);
	rray=(int*)malloc(sizeof(int)*rsize);

	for(i=0;i<ssize;i++)
		sray[i]=rank;

	MPI_Alltoallv(sray,send_data,sdisp,MPI_INT,
				    rray,recv_data,rdisp,MPI_INT,
	                MPI_COMM_WORLD);
	                
	printf("Rank= %d, rray= ",rank);
	for(i=0;i<rsize;i++)
		printf("%d ",rray[i]);
	printf("\r\n");
	
	MPI_Finalize();
	return 0;
}
```

