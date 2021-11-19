# Manajemen Communicator dan Group

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

