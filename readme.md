# Shift 4 SISOP 2020 - T08
Penyelesaian Soal Shift 4 Sistem Operasi 2020
Kelompok T08
  * I Made Dindra Setyadharma (05311840000008)
  * Muhammad Irsyad Ali (05311840000041)

---
## Table of Contents
* [Soal 1](#soal-1)
* [Soal 2](#soal-2)
* [Soal 3](#soal-3)
* [Soal 4](#soal-4)


---
## Soal 1
Source Code : [source](https://github.com/DSlite/SoalShiftSISOP20_modul4_T08/blob/master/ssfs.c)

**Deskripsi**  
Soal meminta kami untuk membuat sebuah metode enkripsi caesar chipher yang berjalan jika sebuah directory dibuat atau direname dengan 
awalan "encv_1" dengan key ```bash9(ku@AW1[Lmvgax6q`5Y2Ry?+sF!^HKQiBXCUSe&0M.b%rI'7d)o4~VfZ*{#:}ETt$3J-zpc]lnh8,GwP_ND|jO```
Yang dimana setiap directory yang sudah terenkripsi akan ter-dekrip jika namanya di rename. Setiap penggunaan "mkdir" atau "rename" maka akan tercatat kedalam log file, metode enkripsi disini haarus mengabaikan "/" pada penamaan file dan metode ini berlaku untuk setiap directory yang ada di dalam suatu directory


**Asumsi Soal**  
Kami mengasumsikan bahwa

**Pembahasan**  
Pertama untuk membuat sebuah metode enkripsi seperti yang sudah di terangkan pada soal. Kami membuat tiga systemcall yaitu `mkdir`,
`create` dan `write`,(.....) yang masing masingnya memiliki fungsi untuk handle kondisi jika ada pembuatan directory dengan nama yang 
ditentukan ,kondisi jika ada directory yang di rename seusai dengan nama yang sudah di tentukan, dan melakukan write

Setiap system call akan menggunakan fungsi `changePath()` `getDirAndFile()` `decrypt()` yang berfungsi untuk dekripsi dan melakukan 
pengambilan juga pengecekan untuk setiap pathfile extension dan directory  

**Fungsi changePath**
```bash
void changePath(char *fpath, const char *path, int isWriteOper, int isFileAsked) {
  char *ptr = strstr(path, "/encv1_");
  int state = 0;
  if (ptr != NULL) {
    if (strstr(ptr+1, "/") != NULL) state = 1;
  }
  char fixPath[1000];
  memset(fixPath, 0, sizeof(fixPath));
  if (ptr != NULL && state) {
    ptr = strstr(ptr+1, "/");
    char pathEncvDirBuff[1000];
    char pathEncryptedBuff[1000];
    strcpy(pathEncryptedBuff, ptr);
    strncpy(pathEncvDirBuff, path, ptr-path);
    if (isWriteOper) {
      char pathFileBuff[1000];
      char pathDirBuff[1000];
      getDirAndFile(pathDirBuff, pathFileBuff, pathEncryptedBuff);
      decrypt(pathDirBuff, 0);
      sprintf(fixPath, "%s%s/%s", pathEncvDirBuff, pathDirBuff, pathFileBuff);
    } else if (isFileAsked) {
      char pathFileBuff[1000];
      char pathDirBuff[1000];
      char pathExtBuff[1000];
      getDirAndFile(pathDirBuff, pathFileBuff, pathEncryptedBuff);
      char *whereIsExtension = strrchr(pathFileBuff, '.');
      if (whereIsExtension-pathFileBuff <= 1) {
        decrypt(pathDirBuff, 0);
        decrypt(pathFileBuff, 0);
        sprintf(fixPath, "%s%s/%s", pathEncvDirBuff, pathDirBuff, pathFileBuff);
      } else {
        char pathJustFileBuff[1000];
        memset(pathJustFileBuff, 0, sizeof(pathJustFileBuff));
        strcpy(pathExtBuff, whereIsExtension);
        snprintf(pathJustFileBuff, whereIsExtension-pathFileBuff+1, "%s", pathFileBuff);
        decrypt(pathDirBuff, 0);
        decrypt(pathJustFileBuff, 0);
        sprintf(fixPath, "%s%s/%s%s", pathEncvDirBuff, pathDirBuff, pathJustFileBuff, pathExtBuff);
      }
    } else {
      decrypt(pathEncryptedBuff, 0);
      sprintf(fixPath, "%s%s", pathEncvDirBuff, pathEncryptedBuff);
    }
  } else {
    strcpy(fixPath, path);
  }
  if (strcmp(path, "/") == 0) {
    sprintf(fpath, "%s", dirpath);
  } else {
    sprintf(fpath, "%s%s", dirpath, fixPath);
  }
}
```

```bash
void changePath(char *fpath, const char *path, int isWriteOper, int isFileAsked) {
  char *ptr = strstr(path, "/encv1_");
  int state = 0;
  if (ptr != NULL) {
    if (strstr(ptr+1, "/") != NULL) state = 1;
```
Pertama tama kami mendefinisikan fungsi ini dengan 4 argumen input, Argumen pertama sebagai fpath yang nanti akan didefinisikan berbeda 
pada beberapa system call yang menggunakan fungsi ini, Argumen kedua sebagai path dari file/directory yang dibuat/direname, Argumen 
ketiga dan keempat adalah untuk menentukan kondisi system call yang dijalanakan (kondisi 0.1 , 1.0 & 0.0)

Terdapat dua kondisi `state`, dengan defalut *0*, state ini berfungsi untuk path yang hanya memiliki satu directory setelah `encv_1` 
CONTOH: *encv_/sisop*, dan *1* untuk path yang masih berlanjut setelah `encv_1` CONTOH: *encv1_/document/sisop*


```bash
}
  char fixPath[1000];
  memset(fixPath, 0, sizeof(fixPath));
  if (ptr != NULL && state) {
    ptr = strstr(ptr+1, "/");
    char pathEncvDirBuff[1000];
    char pathEncryptedBuff[1000];
    strcpy(pathEncryptedBuff, ptr);
    strncpy(pathEncvDirBuff, path, ptr-path);
```
Pendefinisian buffer `fixpath[]` yang digunakan untuk menyimpan proccessed path dari tiap kondisi yang berbeda. `if()` disini akan 
berjalan untuk kondisi (1.0 & 0.1) yang akan mendefinisikan dan mengisi buffer untuk `directory` dan `encv1_`

```bash
if (isWriteOper) {
      char pathFileBuff[1000];
      char pathDirBuff[1000];
      getDirAndFile(pathDirBuff, pathFileBuff, pathEncryptedBuff);
      decrypt(pathDirBuff, 0);
      sprintf(fixPath, "%s%s/%s", pathEncvDirBuff, pathDirBuff, pathFileBuff);
```
Dalam kondisi (1.0) atau hanya **isWriteOper** yang memiliki nilai, `if()` disini akan mendefinisikan buffer untuk *file* dan 
*directory* dan menjalankan fungsi `getDirAndFile()` yang akan me-return directory dan file berdasarkan dari path yang ada di 
`pathEncryptedBuff` dan akan men `decrypt()`nya yang selanjutnya akan di store di buffer `fixpath[]` dengan format 
`(encv1_,pathdir,pathfile)`

```bash
} else if (isFileAsked) {
      char pathFileBuff[1000];
      char pathDirBuff[1000];
      char pathExtBuff[1000];
      getDirAndFile(pathDirBuff, pathFileBuff, pathEncryptedBuff);
      char *whereIsExtension = strrchr(pathFileBuff, '.');
      if (whereIsExtension-pathFileBuff <= 1) {
        decrypt(pathDirBuff, 0);
        decrypt(pathFileBuff, 0);
        sprintf(fixPath, "%s%s/%s", pathEncvDirBuff, pathDirBuff, pathFileBuff);
      } else {
        char pathJustFileBuff[1000];
        memset(pathJustFileBuff, 0, sizeof(pathJustFileBuff));
        strcpy(pathExtBuff, whereIsExtension);
        snprintf(pathJustFileBuff, whereIsExtension-pathFileBuff+1, "%s", pathFileBuff);
        decrypt(pathDirBuff, 0);
        decrypt(pathJustFileBuff, 0);
        sprintf(fixPath, "%s%s/%s%s", pathEncvDirBuff, pathDirBuff, pathJustFileBuff, pathExtBuff);
      }
```
`else if()` disini akan berjalan untuk kondisi (0.1) atau hanya **isFileAsked**nya saja yang memiliki nilai, yang akan menstore buffer 
*Dir*, *esxtension* dan *File* yang sebelumnya sudah di definisikan. Yang pertama di store adalah *dir* dan *file* menggunakan 
`getDirAndFile(pathDirBuff, pathFileBuff, pathEncryptedBuff)`. Untuk mencari *extention* dari path yang ditentukan, akan di lakukan 
pengecekan karakter terlebih dulu menggunakan `if (whereIsExtension-pathFileBuff <= 1)`, karena di kondisi ini nama path sudah di enkripsi dan ada kemungkinan munculnya karakter "." yang akan mempersulit proses dekripsi.  

sebelumnya fungsi `strrchr()` sudah digunakan untuk me-return pointer yang menunjuk ke semua karakter setelah ditemukannya karakter "."
sehingga selanjutnya dapat langsung dilakukan dekripsi untuk string yang hanya memiliki satu atau nol karakter "." dan di store ke buffer 
`fixpath[]` 
untuk string yang memiliki lebih dari satu karakter "." akan dilakukan `snprintf()` sebanyak ` whereIsExtension-pathFileBuff+1` dan 
akan di store ke dalam buffer `pathJustFileBuff[]` yang akan berisi extenxion saja dan seperti sebelumnya, akan dimasukan ke fix path
dengan format `(encv1_, dir, file & extension)`

```bash 
    else {
      decrypt(pathEncryptedBuff, 0);
      sprintf(fixPath, "%s%s", pathEncvDirBuff, pathEncryptedBuff);
    }
  } else {
    strcpy(fixPath, path);
  }
  if (strcmp(path, "/") == 0) {
    sprintf(fpath, "%s", dirpath);
  } else {
    sprintf(fpath, "%s%s", dirpath, fixPath);
  }
}
```
`else` pertama disini adalah untuk kondisi `(0.0)` atau  **isWriteOper** dan **isFileAsked** == 0, sehingga isi dalam buffer 
`pathEncryptedBuff` akan langsung di dekrip dan di store kedalam `fixpath` dengan format `(encv1_, decrypted path)`

`else` kedua adalah untuk kondisi `state` == 0 atau hanya ada **satu** directory ataupun file pada path setelah `encv1_`, sehingga akan 
langsung di store kedalam `fixpath`

selanjutnya adalah pengisian `fpath` dengan parameter "/" yang ada di path. `fixpath` yang sudah didapatkan berdasarkan kondisi yang 
diberikan oleh system call akan dimasukan kedalam `fpath` **jika "/" pada path > 1** disini mengindikasikan ada nya file/directory yang 
dituju setelah `encv1_`. Sebaliknya makan `fpath` hanya akan di isi dengan `dirpath` yang sudah di set di awal 
`(*dirpath = "/home/umum/Documents/SisOpLab/FUSE")`

**Fungsi getDirAndfile**
``` bash 
void getDirAndFile(char *dir, char *file, char *path) {   
  char buff[1000];
  memset(dir, 0, 1000);
  memset(file, 0, 1000);
  strcpy(buff, path);
  char *token = strtok(buff, "/");
  while(token != NULL) {
    sprintf(file, "%s", token);
    token = strtok(NULL, "/");
    if (token != NULL) {
      sprintf(dir, "%s/%s", dir, file);
    }
  }
}
```
Fungsi ini akan membagi path menjadi beberapa token berdasarkan "/" menggunakan fungsi `strtok()`, `while()` pertama disini, berfungsi 
untuk mengambil memasukan setiap token sampai terakhir ke buffer **file** yang sudah di definisikan terlebih dahulu dengan memory 
tertentu menggunakan `memset(file, 0, 1000)` 

`if()` akan berfungsi untuk memasukan `dir` yang diberikan oleh system call kedalam kedalam buffer **dir** yang sudah di definisikan 
terlebih dahulu dengan memory tertentu menggunakan `memset(dir, 0, 1000)` yang akan di tambahkan buffer **file** dibelakangnya yang
menjadikannya sebagai absolut path

**Fungsi Decrypt**
```bash
void decrypt(char *path, int isEncrypt) {  
  char *cursor = path;
  while (cursor-path < strlen(path)) {
    char *ptr = strchr(key, *cursor);
    if (ptr != NULL) {
      int index = (ptr-key+strlen(key)-10)%strlen(key);
      if (isEncrypt) index = (ptr-key+strlen(key)+10)%strlen(key);
      *cursor = *(key+index);
    }
    cursor++;
  }
}
```
Pada fungsi decrypt ini, terdapat `isEncrypt` pada argumen kedua yang akan merubah fungsi ini menjadi enkripsi dari path yang dituju jika 
`isEncyrpt` memiliki nilai atau **(path, 1)** 

`while()` disini akan berjalan untuk setiap karakter yang ada di `path` selama panjang dair kursor < panjang path dan melakukan enkripsi/
dekripsi sesuai kondisi yang berjalan, fungsi ini akan men dekrip dengan `int index = (ptr-key+strlen(key)-10)%strlen(key)` dan 
**cursor** akan di increment

Jika kondisi yang berjalan **(path, 1)** maka `if()`kedua akan berjalan dimana akan melakukan enkripsi dengan
`index = (ptr-key+strlen(key)+10)%strlen(key)` dan cursor akan diganti dengan hasil penjumlahan `key` dan `index`
  
**Fungsi nextSync**
```bash
void nextSync(char *syncDirPath) {  //ini buat nomor 3
  char buff[1000];
  char construct[1000];
  memset(construct, 0, sizeof(construct));
  strcpy(buff, syncDirPath);
  int state = 0;
  char *token = strtok(buff, "/");
}
```

```bash
while (token != NULL) {
    char tBuff[1000];
    char get[1000];
    strcpy(tBuff, token);
    if (!state) {
      if (strstr(tBuff, "sync_")-tBuff == 0) {
        strcpy(get, tBuff+5);
      } else {
        sprintf(get, "sync_%s", tBuff);
        state = 1;
      }
    } else {
      strcpy(get, tBuff);
    }
    sprintf(construct, "%s/%s", construct, get);
    token = strtok(NULL, "/");
  }
```

```bash
  memset(syncDirPath, 0, 1000);
  sprintf(syncDirPath, "%s", construct);
```

**System call**
Ketiga fungsi tersebut dipanggil oleh beberapa system call diantaranya 

**_mkdir** 
```bash
static int _mkdir(const char *path, mode_t mode)
{
	char fpath[1000];
	changePath(fpath, path, 1, 0);

  char *ptr = strrchr(path, '/'); // strrchr intinya ngambil / terakhir sampe paling belakang
  char *filePtr = strstr(ptr, "/encv1_");  //nah baru dah, dari "encv1_" kebelakang di tunjuk *fileptr
  if (filePtr != NULL) {    //kalo ada encv1_ if ini jalan artinya user make encv1 masuk ke log
    if (filePtr - ptr == 0) {
      const char *desc[] = {path};
      logFile("SPECIAL", "ENCV1", 0, 1, desc);
    }
  }
```
System call ini berfungsi untuk menghandle pembuatan direktori, pertama tama system call akan melakukan pengecekan jika ada pembuatan direktori dengan nama `encv1_` dengan menggunakan fungsi `strstr(ptr, "/encv1_")` yang menyimpan path setelah karakter `encv1_`

selanjutnya pada fungsi `if()` akan dilakukan pengecekan pembuatan direktori yang diawali `encv1_`, jika tervalidasi adanya pembuatan dengan nama direktori tersebut maka fungsi `logfile()` akan dijalankan dengan ke 5 argumennya dan path akan di tunjuk oleh pointer `*desc[]` yang akan digunakan untuk mengisi kolom ke 5 dari output `logFile` yang akan dijelasakan pada [Soal 4](#soal-4)

```bash
res = mkdir(fpath, mode);

  char syncOrigPath[1000];
  char syncDirPath[1000];
  char syncFilePath[1000];

  memset(syncOrigPath, 0, sizeof(syncOrigPath));
  strcpy(syncOrigPath, path);
  getDirAndFile(syncDirPath, syncFilePath, syncOrigPath);
  memset(syncOrigPath, 0, sizeof(syncOrigPath));
  strcpy(syncOrigPath, syncDirPath);
```
Pendefinisian dan set size buffer untuk **direktori**, **file** dan buffer **syncOrigPath** yang akan menyimpan path asli, path directory 
dan path file yang synchron

```bash
do {
    char syncPath[1000];
    memset(syncPath, 0, sizeof(syncPath));
    nextSync(syncDirPath);
    if (strcmp(syncDirPath, syncOrigPath) == 0) break;
    changePath(syncPath, syncDirPath, 1, 0);
    if (access(syncPath, F_OK) == -1) continue;
    sprintf(syncPath, "%s/%s", syncPath, syncFilePath);
    mkdir(syncPath, mode);
  } while (1);
```


```bash
 const char *desc[] = {path};
  logFile("INFO", "MKDIR", res, 1, desc);

	if (res == -1) return -errno;

	return 0;
```




**Kesulitan:**  
**ScreenShot**  
**Contoh logging**  

## Soal 2

**Deskripsi**
**Asumsi Soal**
**Pembahasan**
**Kesulitan:**  
**ScreenShot**  
**Contoh logging**  

## Soal 3

**Deskripsi**
**Asumsi Soal**
**Pembahasan**
**Kesulitan:**  
**ScreenShot**  
**Contoh logging**  

## Soal 4
Source Code : [source](https://github.com/DSlite/SoalShiftSISOP20_modul4_T08/blob/master/ssfs.c)

**Deskripsi:**
Soal meminta kami untuk membuat sebuah log system yang akan membuat sebuah log file bernama "fs.log" di direktori *home* user yang berguna untuk menyimpan daftar perintah system call yang telah dijalankan.
Disini pencatatan pada log dibagi menjadi dua level yaitu **warning** untuk system call `rmdir`& `unlink`dan **info** untuk system call sisanya

Dengan format logging:
`[LEVEL]::[yy][mm][dd]-[HH]:[MM]:[SS]::[CMD]::[DESC ...]`
* *yy*,*mm*,*dd* akan merepresentasikan tanggal systemcall terpanggil
* *HH*,*MM*, *SS* akan merepresantasikan waktu systemcall terpanggil
* *CMD*, *DESC* akan merepresantasikan system call yang terpanggil dan path file yang di eksekusi dengan system call tersebut

**Asumsi Soal:**
Kami mengasumsikan bahwa akan ada sebuah fungsi yang akan dipanggil setiap untuk pendefinisan atribut dari setiap system call yang akan meretun tanggal, waktu serta system call ayng dipanggil dan path file yang di eksekusi. setiap system call akan mendefiniskan *LEVEL* dan **CMD*nya saat memanggil fungsi ini, dan juga beberapa varible yang akan mendefiniskan path dari file yang dituju

**Pembahasan:**
```bash
static const char *logpath = "/home/umum/Documents/SisOpLab/fuse_log.txt";
```
pertamaa kami melakukan pendefisinian path dari file log yang akan di lanjutkan dengan fungsi logFilenya sendiri

``` bash
void logFile(char *level, char *cmd, int res, int lenDesc, const char *desc[]) {
  FILE *f = fopen(logpath, "a");
  time_t t;	
  struct tm *tmp;
  char timeBuff[100];

  time(&t);
  tmp = localtime(&t);
  strftime(timeBuff, sizeof(timeBuff), "%y%m%d-%H:%M:%S", tmp);

  fprintf(f, "%s::%s::%s::%d", level, timeBuff, cmd, res); 
  for (int i = 0; i < lenDesc; i++) {	
    fprintf(f, "::%s", desc[i]);
  }
  fprintf(f, "\n");

  fclose(f);
}
```
**Fungsi logFile**
Fungsi ini akan menuliskan pada log file sesuai dengan format yang sudah di tentukan, Pertama fungsi ini akan mendefinisikan beberapa argumen yang akan menjadi inputnya, dimana tiap argumen ini akan terdefinisi pada setiap pendefinisian atribut system call yaitu:
* **level** untuk mendefinisikan level dari atribut yang berjalan *(INFO/WARNING)*
* **cmd** untuk menunjukan nama dari atribut yang berjalan
* **res** adalah sebuah variabel ayng ada di tiap atribut yang berfungsi untuk menyimpan status dari file
* **lenDesc** untuk mendifiniskan panjang path dari file yang dituju
* **desc[]** untuk menunjuk absolut path dari file yang dituju


``` bash
FILE *f = fopen(logpath, "a");
```
Pertama fungsi ini akan membuka dan membuat (jika belum ada) log filenya itu sendiri 

``` bash
time_t t;
  struct tm *tmp;
  char timeBuff[100];
```

Pendefinisian variabel dan buffer untuk waktu, struct untuk fungsi `strftime()`

```bash
  time(&t);
  tmp = localtime(&t);	
  strftime(timeBuff, sizeof(timeBuff), "%y%m%d-%H:%M:%S", tmp);	
```

Disini kami membuat fungsi ini menyimpan waktu eksekusi pada `timeBUFF[100]` dengan cara menggunakan fungsi `strftime()`, `strftime()` 
akan me-return sebuah string ke dalam `timeBuff` dengan format yang ada pada argumen 3 sesuai dengan `tmp` pada argumen 4 sebagai variabel yang menyimpan `localtime()` dari time stamp yang didapat dari fungsi `time()`

```bash
 fprintf(f, "%s::%s::%s::%d", level, timeBuff, cmd, res);
  for (int i = 0; i < lenDesc; i++) {	//ini buat ngeprint location dari file yang di gunain sama cmd	
    fprintf(f, "::%s", desc[i]);
  }
  fprintf(f, "\n");

  fclose(f);
```

Selanjutnya kami akan membuat program untuk melakukan write menggunakan `fprintf()` kedalam *logFile* yang di representasikan dengan 
variabel `f` yang akan berisikan format yang sudah di definiskan soal dengan menggunakan `level` yang akan diberikan oleh tiap 
atribut, `timeBuff` yang sudah didapat dari process sebelumnya, `cmd` sebagai nama system call yang akan didapat dari tiap atribut
dan `res` sebagai file status

Kemudian penempatan absolut path file yang dituju dengan menggunakan **for()** loops yang akan berjalan sebanyak langkah untuk 
mencapai nama file tersebut dengan variabel `i` untuk iterasi  dan `lenDesc` akan didapat dari tiap atribut sebagai panjang pathnya, 
selanjutnya penempatan endline dengan `fprintf(f, "\n")` agar tersusun rapih karena log ini tidak hanya berjalan sekali


**Kesulitan:**  
Tidak ada.

**ScreenShot**  

**Contoh logging**  
