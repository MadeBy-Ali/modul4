# Shift 3 SISOP 2020 - T08
Penyelesaian Soal Shift 3 Sistem Operasi 2020\
Kelompok T08
  * I Made Dindra Setyadharma (05311840000008)
  * Muhammad Irsyad Ali (05311840000041)

---
## Table of Contents
* [Soal 3](#soal-3)
* [Soal 4](#soal-4)
  * [soal 4 a](#Soal-4.a.)
  * [soal 4 b](#Soal-4.b.)
  * [soal 4 c](#Soal-4.c.)

 
---

## Soal 3
Source Code : [source](https://github.com/DSlite/SoalShiftSISOP20_modul3_T08/blob/master/soal3/soal3.c)

**Deskripsi:**
Soal meminta kami untuk membuat sebuah program dari C untuk mengkategorikan file. Program ini akan memindahkan 
file sesuai ekstensinya (tidak case sensitive. JPG dan jpg adalah sama) ke dalam folder sesuai ekstensinya 
yang folder hasilnya terdapat di working directory ketika program kategori tersebut dijalankan. Terdapat 3
arguman yang dapat di inputkan yaitu **(-f)**, **(*)** dan **(-d)**. Dengan ketentuan sebagai berikut:  

    * (-f) : 
                 *  user bisa menambahkan argumen file yang bisa dikategorikan sebanyak yang user inginkan  
                 *  Pada program kategori tersebut, folder jpg,c,zip tidak dibuat secara manual,
                    melainkan melalui program c. Semisal ada file yang tidak memiliki ekstensi,
                    maka dia akan disimpan dalam folder “Unknown”.  

    * (-d) :  
                 *  user hanya bisa menginputkan 1 directory saja.
                 *  Hasilnya perintah di atas adalah mengkategorikan file di /path/to/directory dan
                    hasilnya akan disimpan di working directory di mana program C tersebut
                    berjalan (hasil kategori filenya bukan di /path/to/directory).
                 *  Program ini tidak rekursif.
                 *  Setiap 1 file yang dikategorikan dioperasikan oleh 1 thread

    * (*) :  
                 *  mengkategorikan seluruh file yang ada di working directory

**Asumsi Soal:**
Soal meminta kami untuk membuat program c yang mampu mengkategorikan file secara tidak rekursif dengan beberapa 
parameter yang sudah ditentukan. Karena diminta untuk adanya thread pada setiap file yang akan di kategorikan, kami 
mengasumsikan bahwa thread akan menjalankan routinenya pada setiap absolut path yang diambil dari sebuah variabel yang menyimpannya terlebih dahulu dan bukan langsung dari dari sebuah fungsi yang me-return absolut path 

**Pembahasan:**

``` bash
#include <sys/types.h>
#include <sys/stat.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <ctype.h>
#include <dirent.h>
#include <pthread.h>
#include <errno.h>
```
* `#include <sys/types.h>` Library tipe data khusus (e.g. pid_t)
* `#include <sys/stat.h>` LIbrary untuk penddeklarasian fungsi stat() dan semacamnya (e.g.fstat() and lstat())  
* `#include <stdio.h>` Library untuk fungsi input-output (e.g. printf(), sprintf())
* `#include <stdlib.h>` Library untuk fungsi umum (e.g. exit(), atoi())
* `#include <unistd.h>` Llibrary untuk melakukan system call kepada kernel linux (e.g. fork())
* `#include <string.h>` Library untuk pendefinisian berbagai fungsi untuk manipulasi array karakter (e.g. strtok())
* `#include <ctype.h>` Library untuk pendefinisian berbagai fungsi untuk karakter handling(e.g.tolower())
* `#include <dirent.h>` Library untuk merepresentasikan directory stream & struct dirent(e.g. struct dirent *entry)
* `#include <pthread.h>` Library untuk operasi thread (e.g. pthread_create(), ptrhead_exit() )
* `#include <errno.h>` Library untuk error handling (e.g. errno)


Pertama kami melakukan pendefinisian 3 fungsi dan 1 routine(thread) untuk program ini yaitu: `getFileName`,
`getExtension`, `dirChecking` dan `routine`.

**Fungsi *getFileName***
``` bash
char *getFileName(char *fName, char buff[]) {
  char *token = strtok(fName, "/");
  while (token != NULL) {
    sprintf(buff, "%s", token);
    token = strtok(NULL, "/");
  }
}
```
* Fungsi ini didefinisikan menggunakan dua parameter yaitu `*fname` sebagai pointernya dan `buff[]` untuk 
store hasil dari fungsi ini sendiri, dan akan mereturn file name yang masih beserta ekstensinya.
  * Selanjutnya nama dari file akan diambil menggunakan fungsi **strtok()** untuk memecah string dengan 
  dengan delimiter `/` dan akan disimpan di dalam `*token` 
  * Lalu **while** loop akan berjalan selama token belum habis dan file name yang sudah diambil akan di 
  print kedalam `buffer`.
  * Fungsi **strtok()** akan dijalankan lagi dengan parameter pertama = **NULL** untuk mencari token 
  selanjutnya hingga akhir dari input.


**Fungsi *getExtension***
``` bash
char *getExtension(char *fName, char buff[]) {
  char buffFileName[1337];
  char *token = strtok(fName, "/");
  while (token != NULL) {
    sprintf(buffFileName, "%s", token);
    token = strtok(NULL, "/");
  }
```
* Fungsi ini didefinisikan menggunakan dua parameter yaitu `*fname` sebagai pointernya dan `buff[]` untuk store hasil dari fungsi ini sendiri, dan akan mereturn ekstensi dari sebuah file.
* Selanjutnya Fungsi akan melakukan hal yang sama persis seperti fungsi getFile name yang nantinya akan menghasilkan nama file yang masih beserta ekstensinya 

``` bash
 int count = 0;
  token = strtok(buffFileName, ".");
  while (token != NULL) {
    count++;
    sprintf(buff, "%s", token);
    token = strtok(NULL, ".");
  }
```
* Disini file name yang masih beserta ekstensinya akan dipecah kembali menggunakan fungsi **strtok()** dengan delimiter `.` sebagai pemisah antara nama file dengan ekstensi dan disimpan kedalam `token*`
* Karna yang akan pertama di return oleh fungsi **strtok()** adalah `token` pertma/kata pertama seblum delimiter maka **while** loop akan berjalan selama `token` belum habis atau belum sampai ekstensinya dan `counter` akan di increment sebagai variabel untuk checking
* Ekstensi yang sudah didapat akan di print ke dalam `buffer`.

``` bash
 if (count <= 1) {
    strcpy(buff, "unknown");
  }

  return buff;
```
* Pengecekan untuk jumlah `counter` yang kurang atau kondisi dimana file tidak ada ekstensi
* Untuk file yang tidak memiliki ekstensi, `buffer` akan berisi `unknown`

**Fungsi *directory Checking***
``` bash
 void dirChecking(char buff[]) {
  DIR *dr = opendir(buff);
  if (ENOENT == errno) {
    mkdir(buff, 0775);
    closedir(dr);
  }
}
```
* Fungsi didefinisikan menggunakan satu parameter yaitu `buff[]` untuk menyimpan hasil dari fungsi ini sendiri
* Disini pembuatan directory baru akan dilakukan jika ada sebuah error yang dihasilakan oleh fungsi **opendir()**, lalu **if** akan melakukan error handling.
* Fungsi ini melakukan pembuatan directory baru menggunakan fungsi **mkdir()** dengan nama yang di return `buffer` dan permission `0775` atau `read and execute` lalu akan di tutup kembali.

***Routine***
``` bash
void *routine(void* arg) {
  char buffExt[100];
  char buffFileName[1337];
  char buffFrom[1337];
  char buffTo[1337];
  char cwd[1337];
  getcwd(cwd, sizeof(cwd));
  strcpy(buffFrom, (char *) arg);
}
```
* Pada Routine ini kami mendefinisikan lima `buffer` yang masih masingnya akan menghandle: `ext` `fileName`
`path input` `path to` dan `cwd`
* Dimana `buffer` untuk `cwd` akan langsung diisi dengan fungsi **getcwd()** yang mereturn 
current working directory beserta sizenya dan untuk `from` atau argumen path yang diinpukan user 
akan diambil menggunakan **(char *)arg**

``` bash
  if (access(buffFrom, F_OK) == -1) {
    printf("File %s tidak ada\n", buffFrom);
    pthread_exit(0);
  }
  DIR* dir = opendir(buffFrom);
  if (dir) {
    printf("file %s berupa folder\n", buffFrom);
    pthread_exit(0);
  }
  closedir(dir); 
```
* Disini program akan melakukan mengecek eksistensi dan bentuk dari file yang diinputkan oleh user
dari argumen yang diinputkan user menggunakan fungsi: 
    * **access()** dengan source `buffFrom` & `F_OK` sebagai `amode` untuk eksistensi. Jika argumen yang diinputkan
      tidak sesuai makan ditampilkan error message dan `thread` akan diselesaikan menggunakan **pthread_exit(0)**
    * Pengecekan bentuk file yang diinputkan adalah dengan menggunakan **dir** yang dimana jika terbuka sebagai 
      directory maka akan ditampilkan error message dan `thread` akan diselesaikan menggunakan **pthread_exit(0)**

``` bash
  getFileName(buffFrom, buffFileName);
  strcpy(buffFrom, (char *) arg);

  getExtension(buffFrom, buffExt);
  for (int i = 0; i < sizeof(buffExt); i++) {
    buffExt[i] = tolower(buffExt[i]);
  }
  strcpy(buffFrom, (char *) arg);
```
* Selanjutnya kami memanggil fungsi **getFilename()** dengan filename yang akan masuk ke `buffer` 
baru `buffFilename`
* Lalu kami memanggil fungsi **getExtension()** untuk mengambil setiap ext dari filename yang ada di `buffFrom`
dan merubah tiap extension dari file yang ada menjadi `lowercase` menggunakan **for** loop degam fungsi **tolower()** dan `i` sebagai `counter`nya.

``` bash
  dirChecking(buffExt);

  sprintf(buffTo, "%s/%s/%s", cwd, buffExt, buffFileName);
  rename(buffFrom, buffTo);

  pthread_exit(0);
```
* Selanjutnya fungsi **dirChecking()** akan dipanggil yang akan membuat directory baru untuk setiap ekstensi didalam `buffExt` yang belum memiliki directory
* Lalu `buffTo` akan diisi dengan value setiap `buffer` yang sudah di set sebelumnya dengan urutan `cwd`,`buffExt`
dan `buffFilename` menggunakan **sprintf()**. Kemudian file name yang ada di `buffFrom` `(const char *old_filename)`
akan di **rename()** dengan urutan dari `buffTo` `(const char *new_filename)` yang sudah di set.


***main()***
``` bash
int main(int argc, char *argv[]) {
  if (argc == 1) {
    printf("Argument kurang\n");
    exit(1);
  }
  if (strcmp(argv[1], "-f") != 0 && strcmp(argv[1], "*") != 0 && strcmp(argv[1], "-d")) {
    printf("Argument tidak ada\n");
    exit(1);
  }
```
### note input  
*note:  * input untuk argumen `-f`  > 2  
        * input untuk argumen `*`   = 2  
        * input untuk argumen `-d`  = 3  
* Pada main, kami menggunakan dua parameter yaitu `argc` & `*argv[]` untuk jumlah argumen & pointer ke masing masing
argumen tersebut karna akan dibutuhkan beberapa pengecekan untuk argumen yang diinputkan.
* Pertama program akan melakukan pengecekan jumlah pada **if()** pertama yang akan menampilkan error message jika
jumlah input tidak sesuai dengan [note input](#note-input).
* Kedua program akan melakukan pengecekan character argumen yang diinputkan dengan menggunakkan fungsi **strcmpr()**
dengan pointer ke masing masing argumennya menggunakan ***argv[]** sebagai parameter pertama
dan `-f` , `*` serta `-d`sebagi parameter kedua. Dan akan menampilakan error message jika argumen yang diinputkan
tidak sesuai

``` bash
if (strcmp(argv[1], "-f") == 0) {
    if (argc <= 2) {
      printf("Argument salah\n");
      exit(1);
    }

    pthread_t tid[argc-2];
    for (int i = 2; i < argc; i++) {
      pthread_create(&tid[i-2], NULL, &routine, (void *)argv[i]);
    }
    for (int i = 2; i < argc; i++) {
      pthread_join(tid[i-2], NULL);
    }
    exit(0);
  }
```
* Disini program akan mengecek banyak argumen jika yang diinputkan adalah `-f` sekaligus membuat thread dengan 
men-set `thread id` terlebih dahulu dengan nilai `jumlah_argumen - 2` disini **for()** loop akan berjalan sebanyak 
`i` kali, dimana `i` adalah jumlah argumen, **for()** loop akan membuat thread menggunakan **pthread_create()** 
untuk setiap argumennya dengan `tid` `i - 2` dimana `i` tadi akan di increment setiap parameter **for()** terpenuhi
* Jumlah argumen untuk penggunaan `-f` harus sesuai dengan [note input](#note-input), jika tidak maka ditampilkan 
error message dan program akan ditutup dengan **exit(1)**
* Lalu `thread` akan diajalankan dengan `routine` kepada `(void *)argv[i])`
* Selanjutnya program akan men-join setiap `thread` yang sudah dibuat dengan **pthread_join()**

``` bash
 char *directory;
  if (strcmp(argv[1], "*") == 0) {
    if (argc != 2) {
      printf("Argument salah\n");
      exit(1);
    }
    char buff[1337];
    getcwd(buff, sizeof(buff));
    directory = buff;
  }
```
* Disini program akan mengecek banyak argumen jika yang diinputkan adalah `*`. Jumlah argumen untuk penggunaan `*` 
harus sesuai dengan [note input](#note-input), jika tidak maka ditampilkan error message dan program akan ditutup 
dengan **exit(1)**
* Jika hanya **if()** pertama yang terpenuhi(argumen benar) maka program akan menset `cwd` beserta sizenya 
menggunakan **getcwd()** kedalam `buffer` baru yang nantinya akan dimasukkan ke variabel `directory`

``` bash
if (strcmp(argv[1], "-d") == 0) {
    if (argc != 3) {
      printf("Argument salah\n");
      exit(1);
    }
    DIR* dir = opendir(argv[2]);
    if (dir) {
      directory = argv[2];
    } else if (ENOENT == errno) {
      printf("Directory tidak ada\n");
      exit(1);
    }
    closedir(dir);
  }
```
* Disini program akan mengecek banyak argumen jika yang diinputkan adalah `-d`. Jumlah argumen untuk penggunaan `-d`
harus sesuai dengan [note input](#note-input), jika tidak maka ditampilkan error message dan program akan ditutup 
dengan **exit(1)**
* Jika hanya **if()** pertama yang terpenuhi(argumen benar) maka program akan membuka directory sesuai dengan 
argumen kedua yang menggunakan **opendir()** dan memasukan nama directory tersebut kedalam variabel dir, namun
jika directory tidak terbuka maka akan ada error handling di dalam **else if()** yang akan menampilkan error 
message dan exit prgram dengan **exit(1)** 
* Contoh untuk kondisi **else if()** disini adalah file yang tidak memiliki ekstensi, jadi variabel `dir` berisi 
`NULL` dan directory tidak terbuka.  

``` bash
int file_count = 0;
  DIR* dir = opendir(directory);
  struct dirent *entry;
  while ((entry = readdir(dir)) != NULL) {
    if (entry->d_type == DT_REG) {
      file_count++;
    }
  }
  closedir(dir);
```
* Disini kami membuat file counter untuk setiap file yang ada dalam suatu directory, bagian ini adalah handler untuk
argumen `-d` dan `*`
* Pertama-tama directory akan dibuka dengan **opendir()** dan di set ke dalam variabel `dir` untuk nantinya di cek
* `struct dirent *entry;` pendefinisian struct `dirent` untuk penggunaan fungsi **readdir()**
* **while** disini adalah untuk pengecekan tiap filenya di dalam `dir` yang sudah dibuka dengan menggunakan 
`entry->d_type` yang dimana itu adalah field yang berisi indikasi tipe file dan `DT_REG` merupakan macro constant 
dari `d_type` yang berarti tipe file reguler, untuk setiap file reguler yang ditemukan maka nilai `counter` akan di
increment, **while()** akan berjalan sampai tiap file yang ada di dalam directory habis.

```bash 
pthread_t tid[file_count];
  char buff[file_count][1337];
  int iter = 0;

  dir = opendir(directory);
  while ((entry = readdir(dir)) != NULL) {
    if (entry->d_type == DT_REG) {
      sprintf(buff[iter], "%s/%s", directory, entry->d_name);
      iter++;
    }
  }
  closedir(dir);
```
* Sebelumnya kami mendefinisikan sebuah `buffer` untuk menyimpan absolut path dan satu variabel untuk iterasi
* Lalu directory dibuka dengan **opendir()** untuk nantinya pengecekan tiap file didalamnya dilakukan
* Disini kami men-set `tid` sejumlah nilai `counter` (banyak file reguler) di setiap directory untuk thread yang 
akan dibuat
* `sprintf(buff[iter], "%s/%s", directory, entry->d_name);` disini akan memasukan absolut path dari setiap file 
reguler itu sendiri, kondisi ini akan berjalan selama file dari sebuah directory belum habis sesuai parameter **while** loop.
* Size dari `buffer`(iter) akan di increment untuk setiap absolut path yang masuk ke dalam `buffer` tersebut

``` bash
  for (int i = 0; i < file_count; i++) {
    char  *test = (char*)buff[i];
    printf("%s\n", test);
    pthread_create(&tid[i], NULL, &routine, (void *)test);
  }

  for (int i = 0; i < file_count; i++) {
    pthread_join(tid[i], NULL);
  }
```
* **for()** loop disini akan berjalan sebanyak jumlah file reguler yang sudah tersimpan di `buffer` sebelumnya. 
* `*test` disini berfungsi untuk menyimpan terlebih dahulu absolut path dari setiap file sebelum dijalankan di  
dalam thread. Jadi thread disini tidak langusng mengambil argumen keempatnya dari `buffer`  
* Pembuatan thread menggunakan `pthread_create(&tid[i], NULL, &routine, (void *)test)` terlihat disini argumen
keempatnya kami mengguanakan `test` yang tadi sudah menyimpan dahulu absolut path dari setiap file
* **for()** loop kedua akan men-join setiap thread yang sudah dibuat  

**Kesulitan:**  
Tidak ada.

**ScreenShot**  

**Contoh input argumen `-f`**  


**Contoh input argumen `-d`**  


**Contoh input argumen `*`**  



## Soal 4
Source Code : [source](https://github.com/DSlite/SoalShiftSISOP20_modul3_T08/blob/master/soal4/soal4a.c)

**Deskripsi:**
Norland mendapati ada sebuah tiga teka-teki yang tertulis di tiga pilar berbeda. Untuk dapat mengambil batu mulia 
di suatu pilar, Ia harus memecahkan teka-teki yang ada di pilar tersebut. Norland menghampiri setiap pilar secara 
bergantian. 

## **Soal 4.a.**
**Deskripsi:**
Pada teka teki untuk pilar pertama Norland diminta untuk :
*  Membuat program C dengan nama "4a.c", yang berisi program untuk melakukan perkalian matriks. Ukuran matriks 
pertama adalah 4x2, dan matriks kedua 2x5. Isi dari matriks didefinisikan di dalam kodingan. Matriks nantinya akan 
berisi angka 1-20 (tidak perlu dibuat filter angka).
* Dan menampilkan matriks hasil perkaliannya ke layar.

**Asumsis Soal:**
Kami mengasumsikan untuk perkalian matriks ini bahwa akan ada variabel yang bisa mengindikasikan baris pertama dan
kolom pertama pada pengulangan pertama yang dijadikan sebagai counter sesuai dengan syarat perkalian matriks

**Pembahasan:**
```bash
#include <stdio.h>
#include <pthread.h>
#include <sys/ipc.h>
#include <sys/shm.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
```
* `#include <stdio.h>` Library untuk fungsi input-output (e.g. printf(), sprintf())
* `#include <pthread.h>` Library untuk operasi thread (e.g. pthread_create(), ptrhead_exit() )
* `#include <sys/ipc.h>` Library digunakan untuk tiga mekanisme interprocess communication (IPC)(e.g. semaphore)
* `#include <sys/shm.h>` Library untuk mendefinisikan symbolic constants structure seperti(SHM_RDONLY,SHMLBA)
* `#include <stdlib.h>` Library untuk fungsi umum (e.g. exit(), atoi())
* `#include <unistd.h>` Llibrary untuk melakukan system call kepada kernel linux (e.g. fork())
* `#include <string.h>` Library untuk pendefinisian berbagai fungsi untuk manipulasi array karakter (e.g. strtok())


```bash
int matA[4][2] = {
  {1, 1},
  {2, 2},
  {1, 1},
  {2, 2}
};
int matB[2][5] = {
  {1, 4, 1, 2, 1},
  {1, 2, 1, 4, 1}
};
int matC[4][5];
```
* Disini kami melakukan pendefinisian tiga matriks yaitu `matA`(matriks 4*2), `matB`(matriks 2*5) dan `matC`
(matriks 4*5 sebagai matriks hasil perkalian)
* Value dari matriks di set dengan range int 1-20 sesuai soal

``` bash
struct args {
  int i;
  int j;
};
```
* Lalu kami mendefinisikan `struct` dengan member `i` sebgai baris dan `j` sebagai kolom 

***fungsi kali***
``` bash
void *kali(void* arg) {
  int i = ((struct args*)arg)->i;
  int j = ((struct args*)arg)->j;

  for (int k = 0; k < 2; k++) {
    matC[i][j] += matA[i][k] * matB[k][j];
  } 
}
```
* Fungsi pengkalian disini mengguanakan `arg` sebagai argumennya dan `((struct args*)arg)->i`akan memasukan
baris dari tiap matriks kedalam variabel `i` serta `int j = ((struct args*)arg)->j` akan memasukan
kolom dari tiap matriks kedalam variabel `j`
* Pada **for()** loop , kami membuat sebuah variabel `k` yang menjadi nilai kesamaan ordo matriks( 2 kolom
di matriks pertama dan 2 baris di matriks kedua) yang akan mengulang sebanyak 2 kali
* Perkalian dilakukan dengan mengalikan setiap baris di `matirksA` dengan setiap kolom di `matriksB`, jadi `k`
disini mengindikasikan baris pertama dan kolom pertama pada pengulangan pertama, dan setrusnya
* Lalu hasil perkalian akan ditambahkan dan dimasukan ke `matriksC`. Pada case ini ordo matriks hasil adalah (4*5),
karena ordo matriks hasil perkalian dua buah matriks adalah jumlah baris pertama dikali jumlah kolom ke dua. 

``` bash
int main() {

  pthread_t tid[4][5];

  for (int i = 0; i < 4; i++) {
    for (int j = 0; j < 5; j++) {
      struct args *index = (struct args *)malloc(sizeof(struct args));
      index->i = i;
      index->j = j;
      pthread_create(&tid[i][j], NULL, &kali, (void *)index);
    }
  }
  for (int i = 0; i < 4; i++) {
    for (int j = 0; j < 5; j++) {
      pthread_join(tid[i][j], NULL);
    }
  }
```
* Pada **main()**, pertama-tama kami mendefinisikan `tid` dari `thread` dengan jumlah ordo matriks hasil, dan 
sebuah `struct` yang berisi atribut `index`  
* **for()** pertama adalah sebagai looping untuk indikasi baris dan **for()** kedua sebagai loopinh indikasi kolom
yang setiap indikasi baris dan kolom tersbeut akan diset ke `i` dan `j`
* Disini `thread` akan dibuat dengan **pthread_create(&tid[i][j], NULL, &kali, (void *)index)** dan berjalan dengan 
`tid` `i` dan `j` yang di increment setiap perulangannya
* `thread` akan menjalankan fungsi `kali` sebagai routine dengan atribut `index` sebagai variabel yang digunakan   
* Selanjutnya kami men-join setiap `thread` yang sudah dibuat dengan **pthread_join(tid[i][j], NULL)**

``` bash
printf("Matriks :\n");
  for (int i = 0; i < 4; i++) {
    for (int j = 0; j < 5; j++) {
      printf("%4d", matC[i][j]);
    }
    printf("\n");
  }
```
* Lalu kami melakukan printout dengan **printf("%4d", matC[i][j])** untuk setiap baris dan kolom menggunakan 
counter `i` dan `j` pada dua **for()** loops yang masing masingnya untuk counter baris dan kolom yang kan 
diincrement sebanyak jumlah baris(4) dan kolom(5) pada matriks hasil

``` bash
key_t key = 1337;
  int *value;

  int shmid = shmget(key, sizeof(matC), IPC_CREAT | 0666);
  value = shmat(shmid, NULL, 0);

  int* p = (int *)value;

  memcpy(p, matC, 80);

  shmdt(value);
```
* Disini kami membuat shared memory untuk `matriksC` sesuai dengan template pembuatan shared memory yang ada 
pada modul, karena nanti `matriksC `akan digunakan untuk acuan dari soal 4.b   

**Kesulitan:**
Tidak ada.

**Screenshot:**


## **Soal 4.b.**
Source Code : [source](https://github.com/DSlite/SoalShiftSISOP20_modul3_T08/blob/master/soal4/soal4b.c)

**Deskripsi:**
Pada teka teki untuk pilar kedua Norland diminta untuk :
* membuatlah program C kedua dengan nama "4b.c". Program ini akan mengambil variabel hasil perkalian matriks dari 
program "4a.c" (program sebelumnya), dan tampilkan hasil matriks tersebut ke layar.
* Setelah ditampilkan, berikutnya untuk setiap angka dari matriks tersebut, carilah nilai penjumlahannya, dan 
tampilkan hasilnya ke layar dengan format seperti matriks.
*note*: Mengguankaan shared memory dan mengguanakan thread dalam perhitungan penjumlahan


**Asumsi Soal:**
Kami mengasumsikan bahwa akan dibuat matriks yang akan digunakan sebagai shared memory dan akan dibuat 
thread untuk setiap nilai dari matriks hasil yang akan berjalan dengan rutin penjumlahan

**Pembahasan:**

```bash
#include <stdio.h>
#include <pthread.h>
#include <sys/ipc.h>
#include <sys/shm.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
```
* `#include <stdio.h>` Library untuk fungsi input-output (e.g. printf(), sprintf())
* `#include <pthread.h>` Library untuk operasi thread (e.g. pthread_create(), ptrhead_exit() )
* `#include <sys/ipc.h>` Library digunakan untuk tiga mekanisme interprocess communication (IPC)(e.g. semaphore)
* `#include <sys/shm.h>` Library untuk mendefinisikan symbolic constants structure seperti(SHM_RDONLY,SHMLBA)
* `#include <stdlib.h>` Library untuk fungsi umum (e.g. exit(), atoi())
* `#include <unistd.h>` Llibrary untuk melakukan system call kepada kernel linux (e.g. fork())
* `#include <string.h>` Library untuk pendefinisian berbagai fungsi untuk manipulasi array karakter (e.g. strtok())

``` bash
int matrix[4][5];
long hasil[4][5];

struct args {
  int i;
  int j;
};
```
* Pertama kami membuat array dan matriks dengan ordo sesuai dengan output matriks soal 3  
* Mendefinisikan `struct` dengan atribut `i` dan `j` yang nanti digunakan sebagai baris dan kolom

***fungsi Penjumlahan***
``` bash
void *factorial(void* arg) {
  int i = ((struct args*)arg)->i;
  int j = ((struct args*)arg)->j;
  long hasilEl = 1;
  for (int n = 1; n <= matrix[i][j]; n++) hasilEl += (long)n;
  hasil[i][j] = hasilEl;
}
```
* Pendefinisian fungsi `penjumalah` dengan pembuatan `struct` yang akan menset atribut `i` dan `j` nya keadalam 
variable `i` dan `j`
* **For()** loops disini akan berjalan dengan counter `n` yang akan di increment untuk setiap baris dan kolom dari 
matriks, dan akan menjumlahkan counter kedalam `hasilEl` untuk setiap perulangannya

```bash
int main() {
  key_t key = 1337;
  int *value;
  int shmid = shmget(key, 80, IPC_CREAT | 0666);
  value = shmat(shmid, NULL, 0);

  int* p = (int *)value;
  memcpy(matrix, p, 80);

  shmdt(value);
  shmctl(shmid, IPC_RMID, NULL);
```
* Pada **main()**, pertama-tama kami akan membuat shared memory untuk `matrix`, sesuai dengan template pembuatan shared memory yang ada pada modul

``` bash
 for (int i = 0; i < 4; i++) {
    for (int j = 0; j < 5; j++) {
      struct args *index = (struct args *)malloc(sizeof(struct args));
      index->i = i;
      index->j = j;
      pthread_create(&tid[i][j], NULL, &factorial, (void *)index);
    }
  }
  for (int i = 0; i < 4; i++) {
    for (int j = 0; j < 5; j++) {
      pthread_join(tid[i][j], NULL);
    }
```
* **for()** loop pertama adalah sebagai looping untuk indikasi baris dan **for()** kedua sebagai loopinh indikasi 
kolom yang setiap indikasi baris dan kolom tersbeut akan diset ke `i` dan `j`
* Disini `thread` akan dibuat dengan **pthread_create(&tid[i][j], NULL, &kali, (void *)index)** dan berjalan dengan 
`tid` `i` dan `j` yang di increment setiap perulangannya
* `thread` akan menjalankan fungsi `faktorial` sebagai routine dengan atribut `index` sebagai variabel yang 
digunakan
* Selanjutnya kami men-join setiap `thread` yang sudah dibuat dengan **pthread_join(tid[i][j], NULL)**

``` bash
  printf("Matriks :\n");
  for (int i = 0; i < 4; i++) {
    for (int j = 0; j < 5; j++) {
      printf("%20ld", hasil[i][j]);
    }
    printf("\n");
  }
}
```
* Disini program akan menampilkan setiap baris dan kolom dari matriks `hasil[i][j]` menggunakan **for()** loops yang
dengan counter `i` untuk baris dan `j` untuk kolom, menggunakan **printf("%20ld", hasil[i][j])** dan 
menampilkan hasil penjumlahan dari setiap matriks hasil, `%ld` disini adalah untuk banyak karakter long yang akan 
di print dari matriks `hasil[i][j]`

**kesulitan:**
Tidak ada.

**ScreenShot:**


## **Soal 4.c.**
Source Code : [source](https://github.com/DSlite/SoalShiftSISOP20_modul3_T08/blob/master/soal4/soal4c.c)

**Deskripsi:**
Pada teka teki untuk pilar ketiga Norland diminta untuk :
* Memebuat program C ketiga dengan nama "4c.c".
* Pada program ini, Norland diminta mengetahui jumlah file dan folder di direktori saat ini dengan command "ls | wc 
-l"
*note*: menggunakan IPC Pipes

**Asumsi Soal:**
Disini kami mengasumsikan untuk melakukan fork dimana parent process akan menjadi `write end`  dari pipe dan child
akan menjadi `read end` dari output parrent proccess

**Pembahasan:**
```bash
#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>
#include<sys/types.h>
#include<string.h>
#include<sys/wait.h>
```
* `#include <stdio.h>` Library untuk fungsi input-output (e.g. printf(), sprintf())
* `#include <stdlib.h>` Library untuk fungsi umum (e.g. exit(), atoi())
* `#include <unistd.h>` Llibrary untuk melakukan system call kepada kernel linux (e.g. fork())
* `#include <sys/types.h> `Library tipe data khusus (e.g. pid_t)
* `#include <string.h>` Library untuk pendefinisian berbagai fungsi untuk manipulasi array karakter (e.g. strtok())
* `#include<sys/wait.h> `Library untuk pendefinisian symbolic constants untuk penggunaan waitpid(): (e.g. WNOHANG)

```bash
int main() {
  int fd[2];

  pid_t pid;

  pipe(fd);
```
* Pertama kami men-set satu `pid`untuk thread dengan **pid_t** dan satu file descriptor  yaitu `fd[2]` dimana 
fungsinya adalah  

``` bash
 pid = fork();
  if (pid == 0) {
    dup2(fd[1], 1);
    close(fd[0]);
    char *argv[] = {"ls", NULL};
    execv("/bin/ls", argv);
  }
  while(wait(NULL) > 0);
```
* Disini akan dilakukan **fork()** dan untuk parrent proccesnya, dia akan membuat copy `fd[1]` yang berfungsi
sebgai `write` end dari `pipe` 
* Selanjutnya `ls` akan dijalankan pertama untuk menampilkan semua `dir` dan `file` yang ada dan disimpan dalam
pointer `*argv` yang nantinya akan di `execv` pada `/bin/ls` sehingga parrent procces hanya berjalan sekali
* Child Proccess dibuat menunggu hingga parrent selesai menggunakan **while(wait(NULL) > 0)** 

``` bash
dup2(fd[0], 0);
  close(fd[1]);
  char *argv[] = {"wc", "-l", NULL};
  execv("/usr/bin/wc", argv);
}
```
* Disini **dup2** dijalankan kembali dengan `fd[0]` yang artinya akan menjadi sebgai input atau `read` end dari pipe
baru akan dilakukan perintah `wc` `-l` yang berfungsi untuk menampilkan seluruh jumlah file dan folder yang ada 
pada direcotry, dan akan di set terlebih dahulu kedalam pointer `*argv[]` selanjutnya
* `wc``-l` yang ada di dalam pointer `*argv[]` tadi akan di execv menggunakan **execv("/usr/bin/wc", argv)** 
sehingga child proccess hanya berjalan sekali

**Kesulitan:**
Tidak ada.

**Screenshot:**
