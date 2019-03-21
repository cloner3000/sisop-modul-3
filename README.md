# Thread dan IPC

## Objectives
1. Peserta mengetahui thread
2. Peserta memahami bagaimana thread bekerja
3. Peserta memahami bagaimana cara membuat thread


## 1. Thread 
### 1.1 Thread
Thread merupakan unit terkecil dalam suatu proses yang dapat dijadwalkan oleh sistem operasi. Thread biasanya terbentuk oleh `fork` yang berjalan pada suatu script atau program untuk sebuah proses. Minimal terdapat sebuah thread yang berjalan dalam suatu proses, walau biasanya terdapat lebih dari satu thread dalam proses tersebut. Thread akan berbagi memori dan menggunakan informasi (nilai) dari variabel-variabel pada suatu proses tersebut. Penggambaran thread pada sebuah proses dapat dilihat sebagai berikut.

![thread](thread2.png)

Untuk melihat thread yang sedang berjalan, gunakan perintah :
```bash
top -H
```

Thread dapat dibuat menggunakan fungsi pada program berbahasa C :
```c
#include <pthread.h> //library thread

int pthread_create(pthread_t *restrict tidp,
                   const pthread_attr_t *restrict attr,
                   void *(*start_rtn)(void *),
                   void *restrict arg);

/* Jika berhasil mengembalikan nilai 0, jika error mengembalikan nilai 1 */
```
Penjelasan Syntax:
- The memory location pointed to by `tidp` is set to the thread ID of the newly created thread when pthread_create returns successfully.
- The attr argument is used to customize various thread attributes (detailed in Section 12.3). This chapter sets this to NULL to create a thread with the default attributes.
- The newly created thread starts running at the address of the start_rtn function.
- The arg is a pointer to the single argument passed to the start_rtn. If you need to pass more than one argument to the start_rtn function, then you need to store them in a structure and pass the address of the structure in arg.

Contoh membuat program tanpa menggunakan thread [playtanpathread.c](playtanpathread.c):

```c
#include<stdio.h>
#include <unistd.h>
#include <stdlib.h>
int main()
{
	int i,length=5;
	system("clear");
	for(i=length;i>0;i--)
	{
		printf("%d",i);
		fflush(stdout);
		sleep(1);
		system("clear");
	}
	system("xlogo");
}

```

Contoh membuat program menggunakan thread [playthread.c](playthread.c) :
> compile dengan cara `gcc -pthread -o [output] input.c`

```c
#include<stdio.h>
#include<string.h>
#include<pthread.h>
#include<stdlib.h>
#include<unistd.h>

pthread_t tid[2]; //inisialisasi array untuk menampung thread dalam kasusu ini ada 2 thread

int length=5; //inisialisasi jumlah untuk looping
void* playandcount(void *arg)
{
	unsigned long i=0;
	pthread_t id=pthread_self();
	int iter;
	if(pthread_equal(id,tid[0])) //thread untuk menjalankan counter
	{
		system("clear");
		for(iter=length;iter>0;iter--)
		{
			printf("%i",iter);
			fflush(stdout);
			sleep(1);
			system("clear");
		}
		system("pkill xlogo");
	}
	else if(pthread_equal(id,tid[1]))
	{
        system("xlogo");
	}
	return NULL;
}

int main(void)
{
	int i=0;
	int err;
	while(i<2) //looping membuat thread 2x
	{
		err=pthread_create(&(tid[i]),NULL,&playandcount,NULL); //membuat thread
		if(err!=0) //cek error
		{
			printf("\n can't create thread : [%s]",strerror(err));
		}
		else
		{
			printf("\n create thread success\n");
		}
		i++;
	}
	pthread_join(tid[0],NULL);
	pthread_join(tid[1],NULL);
	exit(0);
	return 0;
}

```

**Kesimpulan** :

Terlihat ketika program menggunakan thread dapat menjalankan dua task secara bersamaan (task menampilkan gam dan task untuk timer lagu).

### 1.2 Join Thread
Fungsi untuk melakukan penggabungan dengan thread lain yang telah di-terminasi (telah di exit).Bila thread yang ingin di-join belum diterminasi,Maka fungsi ini akan menunggu hingga thread yang diinginkan telah terminated.

Contoh program C Join_Thread [thread_join.c](thread_join.c):

```c
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h> //library thread

void *print_message_function( void *ptr );

int main()
{
     pthread_t thread1, thread2;//inisialisasi awal
     const char *message1 = "Thread 1";
     const char *message2 = "Thread 2";
     int  iret1, iret2;

     iret1 = pthread_create( &thread1, NULL, print_message_function, (void*) message1);//membuat thread pertama
     if(iret1)//jika eror
     {
         fprintf(stderr,"Error - pthread_create() return code: %d\n",iret1);
         exit(EXIT_FAILURE);
     }

     iret2 = pthread_create( &thread2, NULL, print_message_function, (void*) message2);//membuat thread kedua
     if(iret2)//jika gagal
     {
         fprintf(stderr,"Error - pthread_create() return code: %d\n",iret2);
         exit(EXIT_FAILURE);
     }

     printf("pthread_create() for thread 1 returns: %d\n",iret1);
     printf("pthread_create() for thread 2 returns: %d\n",iret2);

     //pthread_join( thread1, NULL);
     //pthread_join( thread2, NULL); 

     exit(EXIT_SUCCESS);
}

void *print_message_function( void *ptr )
{
     char *message;
     message = (char *) ptr;
     printf("%s \n", message);
}
```
Keterangan :
- Pada program di atas kita mengcomment baris pthread_join hasilnya tidak akan memunculkan tulisan Thread 1 dan Thread 2 padahal diinisialisasi thread program akan menjalankan fungsi print_message. 
- Sekarang kita mencoba menghapus comment pada Pthread_join. Hasilnya program akan mengeluarkan output Thread 1 dan Thread 2. 

Kesimpulan :
Pada program pertama tidak menjalankan fungsi print_message karena sebelum kedua Thread dijadwalkan, parent_thread telah selesai dieksekusi sehingga tidak menjalankan fungsi bawaan. Pada program kedua pthread_join digunakan untuk menunda eksekusi calling thread hingga target thread selesai dieksekusi, dengan fungsi ini parent_thread akan disuspend hingga target thread selesai dieksekusi.

### 1.3 Mutual Exclusion
Suatu cara yang menjamin jika ada sebuah proses yang menggunakan variabel atau berkas yang sama (digunakan juga oleh proses lain), maka proses lain akan dikeluarkan dari pekerjaan yang sama.

Contoh program Simple Mutual_Exclusion [threadmutex.c](https://github.com/desyrahmi/sisop-modul-3/blob/master/threadmutex.c):
```c
#include<stdio.h>
#include<string.h>
#include<pthread.h>
#include<stdlib.h>
#include<unistd.h>
 
pthread_t tid1;
pthread_t tid2;
int status;
int nomor;
 
void* tulis(void *arg)
{
    status = 0;
 
    printf("Masukan nomor ");
    scanf("%d", &nomor);
 
    status = 1;
 
    return NULL;
}


void* baca(void *arg)
{
    while(status != 1)
    {

    }

    printf("Nomor %d\n", nomor);
}
 
int main(void)
{

    pthread_create(&(tid1), NULL, &tulis, NULL);
    pthread_create(&(tid2), NULL, &baca, NULL);
 
    pthread_join(tid1, NULL);
    pthread_join(tid2, NULL);
 
    return 0;
}

```
Keterangan :
- Variabel status adalah contoh simple untuk mengendalikan jalannya thread. 

Kesimpulan :
Kegunaan dari Mutex adalah untuk menjaga sumber daya suatu thread tidak digunakan oleh thread lain.


## 2. IPC (Interprocess Communication)
### 2.1 IPC
IPC (*Interprocess Communication*) adalah cara atau mekanisme pertukaran data antara satu proses dengan proses lain, baik pada komputer yang sama atau komputer jarak jauh yang terhubung melalui suatu jaringan.

### 2.2 Pipes
Pipe merupakan komunikasi sequensial antar proses yang saling terelasi. Kelemahannya, hanya dapat digunakan untuk proses yang saling berhubungan dan secara sequensial.
Terdapat dua jenis pipe:
- unnamed pipe : Komunikasi antara parent dan child proses.
- named pipe : Biasa disebut sebagai FIFO, digunakan untuk komunikasi yang berjalan secara independen. **Hanya bisa digunakan jika kedua proses menggunakan *filesystem* yang sama.**
```
$ ls | less
```

### 2.3 Sockets
*Socket* merupakan sebuah *end-point* dalam sebuah proses yang saling berkomunikasi. Biasanya *socket* digunakan untuk komunikasi antar proses pada komputer yang berbeda, namun dapat juga digunakan dalam komputer yang sama.

Example : [socket-server.c](https://github.com/desyrahmi/sisop-modul-3/blob/master/socket-server.c) [socket-client.c](https://github.com/desyrahmi/sisop-modul-3/blob/master/socket-client.c)

Server
```c
#include <stdio.h>
#include <sys/socket.h>
#include <stdlib.h>
#include <netinet/in.h>
#include <string.h>
#include <unistd.h>
#define PORT 8080

int main(int argc, char const *argv[]) {
    int server_fd, new_socket, valread;
    struct sockaddr_in address;
    int opt = 1;
    int addrlen = sizeof(address);
    char buffer[1024] = {0};
    char *hello = "Hello from server";
      
    if ((server_fd = socket(AF_INET, SOCK_STREAM, 0)) == 0) {
        perror("socket failed");
        exit(EXIT_FAILURE);
    }
      
    if (setsockopt(server_fd, SOL_SOCKET, SO_REUSEADDR | SO_REUSEPORT, &opt, sizeof(opt))) {
        perror("setsockopt");
        exit(EXIT_FAILURE);
    }

    address.sin_family = AF_INET;
    address.sin_addr.s_addr = INADDR_ANY;
    address.sin_port = htons( PORT );
      
    if (bind(server_fd, (struct sockaddr *)&address, sizeof(address))<0) {
        perror("bind failed");
        exit(EXIT_FAILURE);
    }

    if (listen(server_fd, 3) < 0) {
        perror("listen");
        exit(EXIT_FAILURE);
    }

    if ((new_socket = accept(server_fd, (struct sockaddr *)&address, (socklen_t*)&addrlen))<0) {
        perror("accept");
        exit(EXIT_FAILURE);
    }

    valread = read( new_socket , buffer, 1024);
    printf("%s\n",buffer );
    send(new_socket , hello , strlen(hello) , 0 );
    printf("Hello message sent\n");
    return 0;
}
```
Client
```c
#include <stdio.h>
#include <sys/socket.h>
#include <stdlib.h>
#include <netinet/in.h>
#include <string.h>
#include <unistd.h>
#define PORT 8080
  
int main(int argc, char const *argv[]) {
    struct sockaddr_in address;
    int sock = 0, valread;
    struct sockaddr_in serv_addr;
    char *hello = "Hello from client";
    char buffer[1024] = {0};
    if ((sock = socket(AF_INET, SOCK_STREAM, 0)) < 0) {
        printf("\n Socket creation error \n");
        return -1;
    }
  
    memset(&serv_addr, '0', sizeof(serv_addr));
  
    serv_addr.sin_family = AF_INET;
    serv_addr.sin_port = htons(PORT);
      
    if(inet_pton(AF_INET, "127.0.0.1", &serv_addr.sin_addr)<=0) {
        printf("\nInvalid address/ Address not supported \n");
        return -1;
    }
  
    if (connect(sock, (struct sockaddr *)&serv_addr, sizeof(serv_addr)) < 0) {
        printf("\nConnection Failed \n");
        return -1;
    }

    send(sock , hello , strlen(hello) , 0 );
    printf("Hello message sent\n");
    valread = read( sock , buffer, 1024);
    printf("%s\n",buffer );
    return 0;
}
```
Jalankan proses server dulu, kemudian jalankan clientnya. Dan amati apa yang terjadi.

### 2.4 Message Queues
Merupakan komunikasi antar proses dimana proses tersebut menciptakan internal linked-list pada alamat kernel Sistem Operasi. Pesannya disebut sebagai *queue* sedangkan pengenalnya disebut *queue* ID. *Queue* ID berguna sebagai *key* untuk menandai pesan mana yang akan dikirim dan tujuan pengiriman pesannya.

### 2.5 Semaphores
Semaphore berbeda dengan jenis-jenis IPC yang lain. Pada pengaplikasiannya, semaphore merupakan sebuah counter yang digunakan untuk controlling resource oleh beberapa proses secara bersamaan.
- Jika suatu counter block memory memiliki nilai positif, semaphore dapat menggunakan resource untuk prosesnya, dan mengurangi nilai counter block dengan 1 untuk menandai bahwa suatu block memory tengah digunakan.
- Sebaliknya, jika semaphore bernilai 0, proses akan masuk pada mode sleep sampai semaphore bernilai lebih besar dari 0.

### 2.6 Shared Memory
Sebuah mekanisme *mapping area(segments)* dari suatu blok *memory* untuk digunakan bersama oleh beberapa proses. Sebuah proses akan menciptakan *segment memory*, kemudian proses lain yang diijinkan dapat mengakses *memory* tersebut. *Shared memory* merupakan cara yang efektif untuk melakukan pertukaran data antar program.

Example: [Proses 1](https://github.com/desyrahmi/sisop-modul-3/blob/master/proses1.c) [Proses 2](https://github.com/desyrahmi/sisop-modul-3/blob/master/proses2.c)

Proses 1
```c
#include <stdio.h>
#include <sys/ipc.h>
#include <sys/shm.h>
#include <unistd.h>

void main()
{
        key_t key = 1234;
        int *value;

        int shmid = shmget(key, sizeof(int), IPC_CREAT | 0666);
        value = shmat(shmid, NULL, 0);

        *value = 10;

        printf("Program 1 : %d\n", *value);

        sleep(5);

        printf("Program 1: %d\n", *value);
        shmdt(value);
        shmctl(shmid, IPC_RMID, NULL);
}
```
Proses 2
```c
#include <stdio.h>
#include <sys/ipc.h>
#include <sys/shm.h>
#include <unistd.h>

void main()
{
        key_t key = 1234;
        int *value;

        int shmid = shmget(key, sizeof(int), IPC_CREAT | 0666);
        value = shmat(shmid, NULL, 0);

        printf("Program 1 : %d\n", *value);
	*value = 30;

        sleep(5);

        printf("Program 1: %d\n", *value);
        shmdt(value);
        shmctl(shmid, IPC_RMID, NULL);
}
```
Jalankan proses 1 terlebih dahulu, lalu proses 2. 
Hasilnya
Proses 1
```
Program 1 : 10
Program 1 : 30
```
Proses 2
```
Program 1 : 10
Program 1 : 30
```

## Appendix
### Libraries documentation (and functions) :sparkles: :sparkles: :camel:
```
$ man {anything-you-want-to-know}
$ man mkfio
$ man fcntl.h
$ man unistd.h
```

## Soal Latihan 

#### Latihan 1
Buatlah sebuah program multithreading yang dapat menyalin isi file baca.txt ke dalam file salin1.txt. Kemudian menyalin isi dari file salin1.txt ke dalam file salin2.txt!
<!---#### Latihan 2
Buatlah sebuah program multithreading yang dapat menampilkan N bilangan prima pertama. program akan dieksekusi menggunakan thread sebanyak T dimana setiap thread akan melakukan print sebanyak N/T bilangan prima.
Input : N = banyak bilangan prima; T = banyak thread yang digunakan 
--->
#### Revisi Latihan 2
Buatlah sebuah program multithreading yang dapat menampilkan bilangan prima dari 1-N. program akan dieksekusi menggunakan thread sebanyak T dimana setiap thread akan melakukan pencarian bilangan prima dengan range N/T (range tiap thread berbeda), kemudian tiap thread akan menampilkan hasilnya.

misalkan N = 100 dan T=2; jadi thread 1 akan mencari bilangan prima dari 1-50 dan thread 2 akan mencari dari 51-100
#### Latihan 3
Ohan adalah seorang network administrator, dia bekerja menggunakan linux server. Suatu ketika Ohan
merasa jenuh dengan pekerjaannya dia ingin mendengarkan lagu, tetapi linux server tidak memiliki GUI
sehingga Ohan harus memutar musik menggunakan konsol/terminal. Bantulah Ohan membuat pemutar
musik berbasis konsol.
Pemutar musik memiliki spesifikasi sebagai berikut :
1. Perintah help untuk menampilkan daftar perintah yang dapat digunakan.
2. Memiliki fitur list untuk menampilkan semua lagu pada folder playlist
3. Memiliki fitur play untuk menjalankan lagu
4. Memiliki fitur pause setelah t detik
5. Memiliki fitur continue setelah t detik
6. Memiliki fitur stop setelah t detik

### References 
https://notes.shichao.io/apue/
https://www.gta.ufrj.br/ensino/eel878/sockets/index.html
http://advancedlinuxprogramming.com/alp-folder/alp-ch05-ipc.pdf
