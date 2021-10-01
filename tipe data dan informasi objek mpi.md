# Tipe Data dan Informasi Objek MPI

Sumber : Agus Kurniawan. 2010. Pemrograman Paralel dengan MPI dan C. Andi (47-77)

Tipe data MPI banyak digunakan untuk melakukan pengiriman dan penerimaan data dari node cluster pada mesin paralel

``MPI_Datatype mydata;``

Penggunaan MPI_Datatype yang dilakukan untuk mengirim kumpulan data array ke sebuah node:
```cpp
float a[SIZE][SIZE] = 
 {1.0, 2.0, 3.0, 4.0
  5.0, 6.0, 7.0, 8.0
  9.0, 10.0, 11.0, 12.0
  13.0, 14.0, 15.0, 16.0
 }

MPI_Datatype rowtype;
for (i = 0; i < numtasks; i++)
 MPI_Send(&a[i][0], 1, rowtype, i, 1, MPI_COMM_WORLD);
 
```
## Tipe Data Dasar

| Tipe Data MPI | Persamaan Tipe Data pada Bahasa C  |
| ------------- |:-------------:|
| MPI_CHAR | signed char |
| MPI_SHORT | signed short int |
| MPI_INT  | signed int  |
| MPI_LONG  | signed long int  |
| MPI_UNSIGNED_CHAR  | unsigned char  |
| MPI_UNSIGNED_SHORT  | unsigned short int  |
| MPI_UNSIGNED  | unsigned int  |
| MPI_USIGNED_LONG  | unsigned long int  |
| MPI_FLOAT  | float  |
| MPI_DOUBLE  | double  |
| MPI_LONG_DOUBLE  | long double  |
| MPI_BYTE  |   |
| MPI_PACKED  |   |

Contoh penggunaan tipe data dasar untuk mengirim dan menerima data melalui MPI_Recv dan MPI_Send:

```cpp
INT token,total=100;
DOUBLE array[100];

MPI_Recv(&token,1,MPI_INT,1,0,MPI_COMM_WORLD,NULL);
MPI_Send(array,total,MPI_DOUBLE,1,0,MPI_COMM_WORLD); 
```

Pada kode diatas,``MPI Recv`` menerima dan bertipe ``MPI_INT`` sedangkan pada ``MPI_Send`` kode diatas melakukan pengiriman array data bertipe ``DOUBLE`` maka kita memberikan tipe data ``MPI_DOUBLE`` pada ``MPI_Send``. Implementasi paling mudah adalah memanfaatkan data array atau struct,Misalkan mempunyai tipe dasar struct pada bahasa C sebagai berikut. 

```cpp
typedef struct {    
 float x,y,z;    
 float velo;    
 int n,type; 
} particle;
```

Selanjutnya tipe data ini akan digunakan pada aplikasi MPI. Oleh karena itu kita perlu membuat suatu tipe data baru berupa tipe data turunan agar tipe data struct kita dipergunakan sesuai standar MPI.

## Tipe Data Turunan

Beberapa tipe data turunan:
1. Kontinyu
2. Vector
3. Struct
4. Index

Setiap membuat tipe data turunan pada MPI, harus konfirmasi dengan memanggil ``MPI_Type_commit()``. Sedangkan untuk menghpus menggunakan ``MPI_Type_free ()``

```cpp
MPI_Type_commit(MPI_Datatype *datatype)
MPI_Type_free(MPI_Datatype *datatype)
```

datatype tipe data yng mau dicommit atau difree.

### Kontinyu

Memungkinkan untuk membuat tipe data yang direplikasi dengan tipe data yang sama pada bidang data yang kontinyu. Misal:
```{(double, 0), (char, 8)}```

direplikasi sebanyak 3 akan menjadi:
```{(double, 0), (char, 8), (double, 16), (char, 24), (double, 32), (char, 40)}```

untuk keperluan ini, diperlukan method MPI ``MPI_Type_contiguous``:
struktur:
```cpp
MPI_Type_contiguous(count, oldtype, &newtype)
```
contoh:
```cpp
MPI_Type_contiguous(3, MPI_INT, &type)
```

Full Source Code:
```cpp
#include <mpi.h>
#include <stdio.h>
 
int main(int argc, char *argv[])
{
    int i,rank;
    MPI_Status status;
    MPI_Datatype type;
    int data[15];
 
    MPI_Init(&argc, &argv);
	MPI_Comm_rank(MPI_COMM_WORLD, &rank);

    MPI_Type_contiguous(5, MPI_INT, &type);
    MPI_Type_commit(&type);	
 
    if (rank == 0)
    {
		printf("Rank %d mengirim data= ",rank);
		for(i=0;i<15;i++)
		{
			data[i] = i;
			printf("[%d]:%d ",i,data[i]);
		}
		printf("\r\n");

        MPI_Send(data, 1, type, 1, 99, MPI_COMM_WORLD);		
    }
    else 
	if (rank == 1)
    {
		for(i=0;i<15;i++)		
			data[i] = -1;

        MPI_Recv(data, 1, type, 0, 99, MPI_COMM_WORLD, &status);		

		printf("Rank %d menerima data: ",rank);
		for(i=0;i<15;i++)
			printf("[%d]:%d ",i,data[i]);
		printf("\r\n");
    }

	MPI_Type_free(&type);
 
    MPI_Finalize();
    return 0;
}
```

## Vektor

TIpe data ini memungkinkan kita melakukan replikasi tipe data ke lokasi yang tersimpan dalam blok. Masing-masing blok diperoleh dengan menggabungkan sejumlah data yang sama dari tipe data lama. Antar blok dipisahkan dengan space bertipe data lama ``MPI_Type_vector``.

```cpp
int MPI_Type_vektor(int count, int blocklength, int stride, MPI_Datatype oldtype, MPI_Datatype &type)
```

| Parameter | Keterangan  |
| ------------- |:-------------:|
| count | jumlah replikasi blok |
| blocklength | panjang blok |
| stride | jumlah elemen antar blok |
| oldtype | tipe data lama |
| &type | tipe data baru |

full code:
```cpp
#include <mpi.h>
#include <stdio.h>
 
int main(int argc, char *argv[])
{
    int i,rank;
    MPI_Status status;
    MPI_Datatype type1,type2;
    int data[24];
 
    MPI_Init(&argc, &argv);
	MPI_Comm_rank(MPI_COMM_WORLD, &rank);

    MPI_Type_contiguous(1, MPI_INT, &type1);
    MPI_Type_commit(&type1);	
	MPI_Type_vector(5, 3, 5, type1, &type2);
    MPI_Type_commit(&type2);

 
    if (rank == 0)
    {
		printf("Rank %d mengirim data= ",rank);
		for(i=0;i<24;i++)
		{
			data[i] = i;
			printf("[%d]:%d ",i,data[i]);
		}
		printf("\r\n");

        MPI_Send(data, 1, type2, 1, 99, MPI_COMM_WORLD);		
    }
    else 
	if (rank == 1)
    {
		for(i=0;i<24;i++)		
			data[i] = -1;

        MPI_Recv(data, 1, type2, 0, 99, MPI_COMM_WORLD, &status);		

		printf("Rank %d menerima data= ",rank);
		for(i=0;i<24;i++)
			printf("[%d]:%d ",i,data[i]);
		printf("\r\n");
    }

	MPI_Type_free(&type2);
	MPI_Type_free(&type1);
 
    MPI_Finalize();
    return 0;
}
```


### Struct

Tipe data struct memungkinkan bekerja dengan berbagai tipe data ``MPI_Type_create_struct()``.

```cpp
int MPI_Type_create_struct(
	int count, 
	int array_of_blocklengths[],
	MPI_Aint array_of_displacemenets[],
	MPI_Datatype array_of_types[],
	MPI_Datatype *newtype
)
```

| Parameter | Keterangan  |
| ------------- |:-------------:|
| count | jumlah replikasi blok |
| array_of_blocklengths | jumlah blok masing-masing element |
| array_of_displacemenets | jarak interval antar blok |
| array_of_types | tipe data elemen masing-masing blok |
| newtype | output tipe data baru |

contoh penggunaan tipe data struct pada MPI:
```cpp
MPI_Datatype type[3] = { MPI_INT, MPI_CHAR, MPI_INT };
int blocklen[3] = { 1, 20, 1 };
MPI_Aint disp[3];

MPI_Type_create_struct(3, blocklen, disp, type, &Customertype);
MPI_Type_commit(&Customertype);
```

full code:
```cpp
#include <mpi.h>
#include <stdio.h>
#include <string.h>

struct Customer
{
    int id;
	char name[20];
    int zipcode;    
};

int main(int argc, char *argv[])
{
    struct Customer customers[5],recv_customers[5];
    int i,rank;
    MPI_Status status;
    MPI_Datatype Customertype;
    MPI_Datatype type[3] = { MPI_INT, MPI_CHAR, MPI_INT };
    int blocklen[3] = { 1, 20, 1 };
    MPI_Aint disp[3];
 
    MPI_Init(&argc, &argv);
	MPI_Comm_rank(MPI_COMM_WORLD, &rank);
	disp[0] = 0;
	disp[1] = sizeof(char);
	disp[2] = 20*sizeof(char);

    MPI_Type_create_struct(3, blocklen, disp, type, &Customertype);
    MPI_Type_commit(&Customertype);    

    if (rank == 0)
    {		
		for(i=0;i<5;i++)
		{
			customers[i].id = i + 1;					
			sprintf(customers[i].name,"User %d",i + 1);
			customers[i].zipcode = 1001 + i;
		}		

        MPI_Send(customers, 6, Customertype, 1, 99, MPI_COMM_WORLD);
    }
    else 
	if (rank == 1)
    {
        MPI_Recv(recv_customers, 6, Customertype, 0, 99, MPI_COMM_WORLD, &status);
		for(i=0;i<5;i++)
		{
			printf("Rank %d data %d diterima: id %d name %s zipcode %d\r\n",
				rank,i+1,recv_customers[i].id,recv_customers[i].name,recv_customers[i].zipcode);
		}
		
    }

	MPI_Type_free(&Customertype);
    MPI_Finalize();
    return 0;
}

```

### Index

Tipe data index digunakan untuk replikasi tipe data tertentu menjadi sebuah sekumpulan urutan data blok yang masing-masingnya terdiri dari tipe data yang dialami dengan interval antar bloknya ``MPI_Type_indexed()``.

| Parameter | Keterangan  |
| ------------- |:-------------:|
| count | jumlah replikasi blok |
| array_of_blocklengths | jumlah blok masing-masing element |
| array_of_displacemenets | jarak interval antar blok |
| oldtype | tipe data lama |
| newtype | output tipe data baru |

full code:
```cpp
#include <mpi.h>
#include <stdio.h>

int main(int argc, char *argv[])
{
    int rank, size, i;
    MPI_Datatype type1, type2;
    int blocklen[3] = { 2, 3, 1 };
    int disp[3] = { 0, 3, 8 };
    int data[15];
    MPI_Status status;

    MPI_Init(&argc, &argv);
    MPI_Comm_size(MPI_COMM_WORLD, &size);   
    MPI_Comm_rank(MPI_COMM_WORLD, &rank);

    MPI_Type_contiguous(1, MPI_INT, &type2);
    MPI_Type_commit(&type2);
    MPI_Type_indexed(3, blocklen, disp, type2, &type1);
    MPI_Type_commit(&type1);

    if (rank == 0)
    {
		printf("Rank %d data dikirim: ",rank);
        for (i=0; i<15; i++)
		{
			data[i] = i;
            printf("[%d]=%d ",i,data[i]);
		}
        printf("\r\n");
      
        MPI_Send(data, 1, type1, 1, 99, MPI_COMM_WORLD);
    }

    if (rank == 1)
    {
        for (i=0; i<15; i++)
            data[i] = -1;
        MPI_Recv(data, 1, type1, 0, 99, MPI_COMM_WORLD, &status);

		printf("Rank %d data diterima: ",rank);
        for (i=0; i<15; i++)
            printf("[%d]=%d ",i,data[i]);
        printf("\r\n");
    }

	MPI_Type_free(&type1);
	MPI_Type_free(&type2);
    MPI_Finalize();
    return 0;
}
```

# Duplikasi Tipe Data

Tipe data bawaan C atau turunan dapat diduplikasi ke suatu variabel tertentu dengan ``MPI_Type_dup()``.

```cpp
int MPI_Datatype_dup(
	type,
	newtype
)
```

full code:
```cpp

#include <mpi.h>
#include <stdio.h>
 
int main(int argc, char *argv[])
{
    MPI_Datatype type, type2;
 
    MPI_Init(&argc, &argv);
 
    MPI_Type_contiguous( 100, MPI_CHAR, &type );
    MPI_Type_commit(&type);

    MPI_Type_dup(type, &type2);
    
	
	MPI_Type_free(&type);
	MPI_Type_free(&type2);
    
    MPI_Finalize();
    return 0;
}
```

# Informasi Objek

Objek yang dibuat pada MPI dapat diset sebuah nilai tertentu dan dapat diakses objek lainnya. Informasi ini dapat dimanipulasi sesuai dengan kebutuhan.
```cpp
typedef int MPI_Info;
```

1. Membuat  Objek MPI_Info
```cpp
int MPI_Info_create(MPI_Info, info1)
```

2. Mengisi dan Edit Nilai

set key dari sebuah info
```cpp
MPI_Info_set(info1, "version", "1.0")
```

display nilai dari sebuah info
```cpp
MPI_Info_get(
	info,
	key,
	value,
	valuelen,
	flag
)
```

3. Menghapus Nilai

```cpp
MPI_Info_delete(info, key)
```

demo code:
```cpp
#include <mpi.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
 
int main( int argc, char *argv[] )
{
	int rank,i,total_key;
    MPI_Info info1, infodup;
    int nkeys, nkeysdup, vallen, flag, flagdup;
    char key[MPI_MAX_INFO_KEY], keydup[MPI_MAX_INFO_KEY];
    char value[MPI_MAX_INFO_VAL], valdup[MPI_MAX_INFO_VAL];
 
    MPI_Init(&argc, &argv);
	MPI_Comm_rank(MPI_COMM_WORLD, &rank);

    MPI_Info_create(&info1);
    MPI_Info_set(info1, "blog", "blog.aguskurniawan.net");
    MPI_Info_set(info1, "file", "mydata.txt"); 
 
    MPI_Info_dup( info1, &infodup );
 
    MPI_Info_get_nkeys(infodup, &nkeysdup);
    MPI_Info_get_nkeys(info1, &nkeys);
    if (nkeys != nkeysdup) 
	{        
        printf("Jumlah key [%d] dan duplikasi key [%d] tidak sama \r\n",nkeys,nkeysdup);

		MPI_Finalize();
		return 0;
    }

    vallen = MPI_MAX_INFO_VAL;
    for (i=0; i<nkeys; i++) 
	{
        MPI_Info_get_nthkey(info1,i,key);
        MPI_Info_get_nthkey( infodup,i,keydup);
        if (strcmp(key, keydup)) 
		{
            printf("Key tidak sama: %s != %s\n", keydup, key);
        }
 
        vallen = MPI_MAX_INFO_VAL;
        MPI_Info_get(info1, key, vallen, value, &flag);
        MPI_Info_get(infodup, keydup, vallen, valdup, &flagdup);
        if (!flag || !flagdup) 
		{            
            printf("Rank %d: Error mengambil key %s\r\n",rank, key);
        }
        else 
		{
            printf("Rank %d: key \"%s\" value \"%s\"\r\n",rank, key,value);
			printf("Rank %d: key duplikasi \"%s\" value \"%s\"\r\n",rank, keydup,valdup);
        }
    } 

	total_key = nkeys;
	MPI_Info_delete(info1,"file");
	MPI_Info_get_nkeys(info1, &nkeys);	
	printf("Rank %d: total_key [%d] key yang telah dihapus [%d]\r\n",rank,total_key,nkeys);

    MPI_Info_free(&info1);
    MPI_Info_free(&infodup);

    MPI_Finalize();
    return 0;
}

```

# MPI Proses

| Fungsi | Kegunaan  |
| ------------- |:-------------:|
| MPI_Comm_size() | Untuk mengetahui ukuran komunikasi |
| MPI_Comm_rank() | Untuk mengetahui rank yang digunakan |
| MPI_Get_version() | Cek versi MPI |
| MPI_get_processor_name | Cek nama processor |

demo code:
```cpp
#include <mpi.h>
#include <stdio.h>
 
int main(int argc, char *argv[])
{
	int rank,size;
	int version, subversion,len;
	char name[MPI_MAX_PROCESSOR_NAME];
 
    MPI_Init(&argc, &argv);
	MPI_Comm_size(MPI_COMM_WORLD, &size);
	MPI_Comm_rank(MPI_COMM_WORLD, &rank);
    
    MPI_Get_version(&version,&subversion);
	MPI_Get_processor_name(name,&len);

	printf("Rank: %d\r\n",rank);
	printf("Size: %d\r\n",size);
	printf("Nama Proses: %s\r\n",name);
	printf("Versi: %d\r\n",version,subversion);	
    
    MPI_Finalize();
    return 0;
}

```
