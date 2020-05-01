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

[error] (#error-1)


---
## Soal 1
Source Code : [source](https://github.com/DSlite/SoalShiftSISOP20_modul4_T08/blob/master/ssfs.c)

**Deskripsi**  
Soal meminta kami untuk membuat sebuah metode enkripsi caesar chipher yang berjalan jika sebuah directory **dibuat** atau **direname** dengan awalan "encv_1" dengan key ```bash9(ku@AW1[Lmvgax6q`5Y2Ry?+sF!^HKQiBXCUSe&0M.b%rI'7d)o4~VfZ*{#:}ETt$3J-zpc]lnh8,GwP_ND|jO```
Yang dimana setiap directory yang sudah terenkripsi akan ter-dekrip jika namanya di rename. Setiap penggunaan "mkdir" atau "rename" maka akan tercatat kedalam log file, metode enkripsi disini haarus mengabaikan "/" pada penamaan file dan metode ini berlaku untuk setiap directory yang ada di dalam suatu directory


**Asumsi Soal**  
Kami mengasumsikan bahwa

**Pembahasan**  
Pertama untuk membuat sebuah metode enkripsi seperti yang sudah di terangkan pada soal. Kami membuat tiga systemcall yaitu `mkdir`,
`create` dan `write` yang masing masingnya memiliki fungsi untuk handle kondisi jika ada pembuatan directory dengan nama yang 
ditentukan `(encv1_)`,kondisi jika ada directory yang di rename seusai dengan nama yang sudah di tentukan, dan melakukan write

Setiap system call akan menggunakan fungsi `changePath()` `getDirAndFile()` `decrypt()` yang berfungsi untuk dekripsi dan melakukan 
pengambilan juga pengecekan untuk setiap pathfile extension dan directory  

Untuk melakukan handle dari enkripsi dan dekripsi itu sendiri kami menggunakan fungsi sebagai berikut:  
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
dekripsi sesuai kondisi yang berjalan, fungsi ini akan men dekrip dengan `int index = (ptr-key+strlen(key)-10)%strlen(key)` dimana index 
ini akan ditambah dengan panjang keynya karena agar proccess decrypt tidak menghasilnya nilai (-) kemudian `10` sebagai banyak karakter 
yang di shift dan terkahir akan di moduka dengan panjang key **cursor** akan di increment

Jika kondisi yang berjalan **(path, 1)** maka `if()`kedua akan berjalan dimana akan melakukan enkripsi dengan
`index = (ptr-key+strlen(key)+10)%strlen(key)` yang kurang lebih sama, tapi disina akan di `+10` karena enkripsi dan value cursor akan 
diganti dengan key + index
  

Sebelum dilakukan pengecekan karakter `encv1_` pada proses rename atau membuat directory, kami mendefinisikan fungsi untuk mengambil file dandirectory dari path yang ditentukan yang nantinya akan di pergunakan dalam proses enkripsi dekripsi, sebagai berikut:  
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
Fungsi ini akan membagi path menjadi beberapa token yaitu *directory* dan *file* berdasarkan "/" menggunakan fungsi `strtok()`, `while()` 
pertama disini, berfungsi untuk mengambil dan memasukan setiap token sampai terakhir ke buffer **file** yang sudah di definisikan 
terlebih dahulu dengan memory tertentu menggunakan `memset(file, 0, 1000)` 

`if()` akan berjalan untuk kondisi token belum habis dan masih ada next entry setelah "/" berarti "file" disini merupakan sebuah 
directory dan akan dimasukan kedalam `dir`  yang sudah diset terlebih dahulu dengan memory tertentu menggunakan `memset(dir, 0, 1000)`.  
  
Selanjutnya adalah fungsi untuk melakukan pengecekan karakter `encv1_` pada proses rename atau membuat directory, sebagai berikut:  
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
berjalan untuk kondisi dekripsi dan enkripsi untuk file. Yang akan mendefinisikan dan mengisi buffer untuk `directory` dan 
`encv1_`dimana buffer `pathEncvDirBuff` akan menyimpan setiap karakter sebelum karakter `encv1_` dan `pathEncryptedBuff` akan menyimpan 
setiap karakter setelah `encv1_` yang sudah di enkripsi atau di dekripsi

```bash
if (isWriteOper) {
      char pathFileBuff[1000];
      char pathDirBuff[1000];
      getDirAndFile(pathDirBuff, pathFileBuff, pathEncryptedBuff);
      decrypt(pathDirBuff, 0);
      sprintf(fixPath, "%s%s/%s", pathEncvDirBuff, pathDirBuff, pathFileBuff);
```
Dalam kondisi (1.0) atau hanya **isWriteOper** yang memiliki nilai akan melakukan encrypt dari directory, agar pembuatan file baru pada 
directory yang sudah di enkripsi bisa menggunakan nama yang tidak terenkrip, `if()` disini akan mendefinisikan buffer untuk *file* dan 
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
menggunakan fungsi `strrchr()` yang akan menunjukan lokasi letak extension pada `filebuff` kemudian akan dilakukan pengecekan karakter 
`(.)` karena ada kondisi dimana file tidak memiliki extension (**diawali** dengan`(.)`) menggunakan 
`if (whereIsExtension-pathFileBuff <= 1)`

Jika kondisi yang terjadi `whereIsExtension-pathFileBuff = 0` atau tidak ada karakter didepan `(.)` maka akan dilakukan dekripsi untuk 
`pathDirBuff` dan `pathFileBuff` kemudia dimasukan ke `fixPath`nya

`else()` disini adalah untuk kondisi file yang memilik extention, dengan pendefinisian buffer `pathJustFileBuff` untuk menyimpan nama 
file saja menggunakan, kemudian nama file dan `pathDirBuff` akan di dekrip dan dimasukan kedalam `fixPath`

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
`else` pertama disini adalah untuk kondisi bukan file `(0.0)` atau untuk `directory` sehingga isi dalam buffer `pathEncryptedBuff` akan langsung di dekrip dan di store kedalam `fixpath`

`else` kedua adalah untuk kondisi `state` == 0 atau tidak ada karakter `encv1_` maka isi dari `fixPath` akan sama dengan `path` karena 
tidak ada metode enkripsi dan dekripsi yang akan dijalankan

selanjutnya adalah kedalam `fpath` dengan parameter "/" yang ada di path. `fixpath` yang sudah didapatkan berdasarkan kondisi yang diberikan oleh system call akan dimasukan kedalam `fpath` **jika "/" pada path > 1**  setelah digabungkan dengan `dirpath` atau lokasi mount pointnya disini mengindikasikan ada nya file/directory yang dituju setelah `encv1_`, atau ada proses enkripsi dan dekripsi.  

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

**Implementasi SystemCall**
Beberapa pengeimplementsian system call yang kami gunakan untuk nomor 1 diantaranya:  
**_getattr**
```bash
static int _getattr(const char *path, struct stat *stbuf)  
{
	char fpath[1000];
  changePath(fpath, path, 0, 1);
  if (access(fpath, F_OK) == -1) {
    memset(fpath, 0, sizeof(fpath));
    changePath(fpath, path, 0, 0);
  }
	int res;

	res = lstat(fpath, stbuf);

  const char *desc[] = {path};
  logFile("INFO", "GETATTR", res, 1, desc);

	if (res == -1) return -errno;

	return 0;
}
```
System call ini akan menjalankan fungsi `changePath()` dengan kondisi **(0.1)** karena `_getattr` bukan merupakan write operation.
Selanjutnya akan dilakukan pengecekan eksistensi path yang diinputkan sebagai *file* atau *directory* dan sudah terenkripsi atau belum
menggunakan `(access(fpath, F_OK) == -1)` jika tidak dapat di akses sebagai *file* maka akan dijalankan fungsi `changePath` untuk *path* 
kedalam *fpath* dengan kondisi **(0.0)** atau kondisi untuk direktori

Kemudian akan dijalankan fungsi `lstat()` yang akan mengambil symbolic link status dari path yang diberikan dan dimasukan kedalam `stbuf`,
Pada setiap penggunaan system call akan ada sebuah log yang di write kedalam log file menggunakan fungsi `logFile`, untuk system call ini
pendefinisian log adalah dengan level *info*, dengan nama system call *GETATTR*, *result* dari lstat, *1* dan *des* yang menunjuk kepada
path yang tentukan

**_access**  
```bash
static int _access(const char *path, int mask)
{
	char fpath[1000];
	changePath(fpath, path, 0, 1);
  if (access(fpath, F_OK) == -1) {
    memset(fpath, 0, 1000);
    changePath(fpath, path, 0, 0);
  }
	int res;

	res = access(fpath, mask);

  const char *desc[] = {path};
  logFile("INFO", "ACCESS", res, 1, desc);

	if (res == -1) return -errno;

	return 0;
}
```
System call ini akan menjalankan fungsi `changePath()` dengan kondisi **(0.1)** karena `_getattr` bukan merupakan write operation.
Selanjutnya akan dilakukan pengecekan eksistensi path yang diinputkan sebagai *file* atau *directory* dan sudah terenkripsi atau belum
menggunakan `(access(fpath, F_OK) == -1)` jika tidak dapat di akses sebagai *file* maka akan dijalankan fungsi `changePath` untuk *path* 
kedalam *fpath* dengan kondisi **(0.0)** atau kondisi untuk direktori

Kemudian akan dijalankan fungsi `access()` yang akan melakukan cek permisi pada user terhadap suatu path dengan mode *mask*, Pada setiap 
penggunaan system call akan ada sebuah log yang di write kedalam log file menggunakan fungsi `logFile`, untuk system call ini 
pendefinisian log adalah dengan level *info*, dengan nama system call *ACCESS*, *result* dari access, *1* dan *des* yang menunjuk kepada
path yang tentukan

**_mkdir** 
```bash
static int _mkdir(const char *path, mode_t mode)
{
	char fpath[1000];
	changePath(fpath, path, 1, 0);

  char *ptr = strrchr(path, '/');
  char *filePtr = strstr(ptr, "/encv1_");  
  if (filePtr != NULL) {  
    if (filePtr - ptr == 0) {
      const char *desc[] = {path};
      logFile("SPECIAL", "ENCV1", 0, 1, desc);
    }
  }
```
System call ini berfungsi untuk **menghandle pembuatan direktori** sesuai nama yang ditentukan(encv1_&encv2juga??), pertama tama system call akan melakukan pengecekan jika ada pembuatan direktori dengan nama `encv1_` dengan menggunakan fungsi `strstr(ptr, "/encv1_")` yang menyimpan path setelah karakter `encv1_`

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
Bagian ini berfungsi untuk menjalankan fungsi `mkdir()` itu sendiri kedalam `fpath` dengan `mode` yang sudah di tentukan dan
untuk mendapatkan orignal path yang akan di simpan kedalam buffer `syncOrigPath`. original path ini didapat dari fungsi `getDirAndFile()` 
yang mengisi buffer `syncOrigPath` dengan path directory dan filenya yang akan di update menggunakan fungsi 
`strcpy`dengan isi dari buffer `syncDirpath`

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
Pada bagian ini kami membuat infinite loop yang akan melakukan sinkronisasi pada setiap buffer `syncDirpath` dengan menggunakan fungsi nextSync yang sudah dijelaskan (diatas)

pada `if()` pertama buffer `syncOrigPath` yang berisi path asli akan di bandingkan dengan buffer `syncDirpath` yang menyimpan path yang 
sudah tersingkronisasi, dan jika kondisi `if()` terpenuhi dimana path dari `syncDirpath` sudah sinkron maka akan melakukan `break` dan 
loop akan berhenti, namun selama belum tersingkronisasi maka fungsi `changePath` akan dijalankan pada `syncDirPath` dan 
`syncOrigPath`dengan kondisi **(1.0)**  

`if()` kedua akan melakukan pengecekan pembukaan path yang ada di `syncPath` dengan fungsi `access`, jadi selama path ini belum bisa 
terakses maka loop akan tetap berjalan dan `syncPath` akan di update dengan ditambahkan `syncFilePath` dan directory akan dibuat dengan 
nama yang sudah tersinkronisasi yang ada pada buffer `syncPath`

```bash
 const char *desc[] = {path};
  logFile("INFO", "MKDIR", res, 1, desc);

	if (res == -1) return -errno;

	return 0;
```
Pada bagian ini kami membuat system call untuk memebuat log file dengan fungsi `logFile()` yang sudah di definisikan di (soal4) dengan
format `("INFO", "MKDIR", res, 1, desc)` dimana `desc` akan menunjuk kepada path sebagai keterangannya  
  
**_rename**  
System call berfungsi untuk **menghandle rename directory** sesuai dengan nama yang ditentukan (encv1_&encv2_) 

```bash
static int _rename(const char *from, const char *to)
{
	char ffrom[1000];
	char fto[1000];
	changePath(ffrom, from, 0, 1);
	changePath(fto, to, 0, 1);
```
Pertama tama system call ini akan menjalankan `changePath()` pada dengan fpath `ffrom` & `fto` dan `from` & `to` 
dengan kondisi **(0.1)** yang berarti  hanya `isFileAsked` pada fungsi `changePath()` yang berjalan 

```bash
  if (access(ffrom, F_OK) == -1) {
    memset(ffrom, 0, sizeof(ffrom));
    changePath(ffrom, from, 0, 0);
  }
  if (access(fto, F_OK) == -1) {
    memset(fto, 0, sizeof(fto));
    changePath(fto, to, 0, 0);
  }
```
`if()` pertama disini akan berjalan jika path yang ada di buffer `ffrom` terbuka, maka akan di set memory untuk `ffrom` itu sendiri dan 
akan di jalankan fungsi `changePath()` dengan kondisi **(0.0)** yang berarti path file tidak akan di return, jadi isi dari `fpath` disini hanya `pathEncvDirBuff` dan `pathEncryptedBuff` dengan kondisi tertentu yang ada di fungsi `changepath()`  

`if()` KEDUA disini akan berjalan jika path yang ada di buffer `fto` terbuka, maka akan di set memory untuk `fto` itu sendiri dan 
akan di jalankan fungsi `changePath()` dengan kondisi **(0.0)** yang berarti path file tidak akan di return, jadi isi dari `fto` disini hanya `pathEncvDirBuff` dan `pathEncryptedBuff` dengan kondisi tertentu yang ada di fungsi `changepath()`  

```bash
  char *toPtr = strrchr(to, '/');
  char *fromPtr = strrchr(from ,'/');
  char *toStartPtr = strstr(toPtr, "/encv2_");
  char *fromStartPtr = strstr(fromPtr, "/encv2_");
```
menunjuk setiap karakter dari `/encv2_` hingga null pada `toPtr` dan `fromPtr` yang sebelumnya sudah berisi setiap karakter dari `/` 
terkahir hingga null

```bash
  if (toStartPtr != NULL) {
    if (toStartPtr - toPtr == 0) {
      splitter(ffrom);
      const char *desc[] = {fto};
      logFile("SPECIAL", "ENCV2", 0, 1, desc);
    }
  }
```
`if()` disini akan berfungsi untuk pengecekan jalannya fungsi splitter yang sudah dijelaskan (diatas), 
pada kondisi `(toStartPtr - toPtr == 0)`  fungsi `splitter()` akan berjalan dan men-split setiap file yang ada pada path `ffrom` termasuk semua isi subdirektorinya secara recursive. 

kemudian log akan di write kedalam log file dengan memanggil fungsi `logfile` dengan format (`logFile("SPECIAL", "ENCV2", 0, 1, desc`) dimana desc menunjuk isi dari buff `fto`

```bash
  if (fromStartPtr != NULL) {
    if (fromStartPtr - fromPtr == 0) {
      unsplitter(ffrom);
    }
  }
  toStartPtr = strstr(toPtr, "/encv1_");
  if (toStartPtr != NULL) {
    if (toStartPtr - toPtr == 0) {
      const char *desc[] = {fto};
      logFile("SPECIAL", "ENCV1", 0, 1, desc);
    }
  }
```

`if()` ketiga dan keempat adalah pengecekan untuk kondisi rename dengan karakter `encv1_` yang sebelumnya sudah di cari menggunakan 
fungsi `strtsr()`dan akan melakukan write ke log file dengan fungsi `logfile()` dengan `desc` yang menunjuk isi dari buff `fto` jika 
kondisi `(toStartPtr - toPtr == 0)`terpenuhi


```bash
	int res;

	res = rename(ffrom, fto);

  const char *desc[] = {from, to};
  logFile("INFO", "RENAME", res, 2, desc);

	if (res == -1) return -errno;

	return 0;
}
```
Pada bagian ini variable `res` akan diisi dengan new name(`fto`) yang didapatkan dari fungsi `rename()`, kemudian akan dilakukan write
kedalam log file dengan fungsi `(logFile("INFO", "RENAME", res, 2, desc))`. Fungsi `rename()` akan mereturn -1 jika terjadi error dan
akan mereturn error handling  


**_create**
```bash
static int _create(const char *path, mode_t mode,
struct fuse_file_info *fi)
```
Pada system call ini kami menggunakan fuse_file_info Struct Reference yang ada pada library fuse yang memungkinkan untuk menggunakan 
filehandler

```bash
{
	char fpath[1000];
  changePath(fpath, path, 1, 0);

	int res;

	res = open(fpath, fi->flags, mode);
```
Pertama tama kami menjalankan fungsi `changepath()` dengan kondisi **(1.0)** yang mendfinisikan sebagai process `write`. System call ini 
akan menyimpan path yang ada pada buffer `fpath` menggunakan field documentation `flags` kedalam variabel `res` yang nantinya akan dirubah
dengan file handle id `fh` setelah proccess mebuat log, singkronisasi dan membuat original path


```bash
  char syncOrigPath[1000];
  char syncDirPath[1000];
  char syncFilePath[1000];
  memset(syncOrigPath, 0, sizeof(syncOrigPath));
  strcpy(syncOrigPath, path);
  getDirAndFile(syncDirPath, syncFilePath, syncOrigPath);
  memset(syncOrigPath, 0, sizeof(syncOrigPath));
  strcpy(syncOrigPath, syncDirPath);
```
Bagian ini berfungsi untuk mendapatkan orignal path yang akan di simpan kedalam buffer `syncOrigPath`. original path ini didapat dari 
fungsi `getDirAndFile()` yang menggisi buffer `syncOrigPath` dengan path *directory* dan *filenya* yang akan di update menggunakan fungsi 
`strcpy`dengan isi dari buffer `syncDirpath`

```bash
  do {
    char syncPath[1000];
    memset(syncPath, 0, sizeof(syncPath));
    nextSync(syncDirPath);
    if (strcmp(syncDirPath, syncOrigPath) == 0) break;
    changePath(syncPath, syncDirPath, 0, 1);
    if (access(syncPath, F_OK) == -1) continue;
    sprintf(syncPath, "%s/%s", syncPath, syncFilePath);
    close(open(syncPath, fi->flags, mode));
  } while (1);
```
Pada bagian ini kami membuat infinite loop yang akan melakukan sinkronisasi pada setiap buffer `syncDirpath` dengan menggunakan fungsi nextSync yang sudah dijelaskan (diatas)

pada `if()` pertama buffer `syncOrigPath` yang berisi path asli akan di bandingkan dengan buffer `syncDirpath` yang menyimpan path yang 
sudah tersingkronisasi, dan jika kondisi `if()` terpenuhi dimana path dari `syncDirpath` sudah sinkron maka akan melakukan `break` dan 
loop akan berhenti, namun selama belum tersingkronisasi maka fungsi `changePath` akan dijalankan pada `syncDirPath` dan 
`syncOrigPath`dengan kondisi **(0.1)**  

`if()` kedua akan melakukan pengecekan pembukaan path yang ada di `syncPath` dengan fungsi `access`, jadi selama path ini belum bisa 
terakses maka loop akan tetap berjalan dan `syncPath` akan di update dengan ditambahkan `syncFilePath` dan `syncFilePath`ini akan dibuka dan diset dengan menggunakan flags kemudia di tutup. 

```bash
  const char *desc[] = {path};
  logFile("INFO", "CREAT", res, 1, desc);

	if (res == -1) return -errno;

	fi->fh = res;
	return 0;
}
```
Pada bagian ini kami membuat system call untuk memebuat log file dengan fungsi `logFile()` yang sudah di definisikan di (soal4) dengan
format `("INFO", "CREAT", res, 1, desc)` dimana `desc` akan menunjuk kepada path sebagai keterangannya

Lalu kami menggunakan file handle id `fh` yang akan menyimpan file id dari `res`  
  
**_write**
```bash
static int _write(const char *path, const char *buf, size_t size,
off_t offset, struct fuse_file_info *fi)
```
Pada system call ini kami menggunakan fuse_file_info Struct Reference yang ada pada library fuse yang memungkinkan untuk menggunakan 
filehandler, disini juga menggunakan `off_t offset` dan `size_t size ` untuk fungsi `pwrite()`  

```bash
{
	char fpath[1000];
	changePath(fpath, path, 1, 0);

	int fd;
	int res;

	(void) fi;
	if(fi == NULL) fd = open(fpath, O_WRONLY);
	else fd = fi->fh;

	if (fd == -1) return -errno;

	res = pwrite(fd, buf, size, offset);
```
Pertama tama kami menjalankan fungsi `changepath()` dengan kondisi **(1.0)** yang mendfinisikan sebagai process `write`. Selanjutnya adalah pendefinisan variabel sebagai file descriptor dan `res`. Disini kami juga medefinisikan unused variable `fi` yang ada kemungkinan
terisi saat system call `open` berjalan

`if()` disini akan menset varible fd dengan isi dari `fpath` dan permission write **jika** systemcall `open` dan `create` **belum** me 
return nilai kepada `fi`

res disini akan berisi buffer `fd` yang sudah di write dair buffer `buf` dengan size dan offset yang sudah di tentukan

```bash
  char syncOrigPath[1000];
  char syncDirPath[1000];
  char syncFilePath[1000];
  memset(syncOrigPath, 0, sizeof(syncOrigPath));
  strcpy(syncOrigPath, path);
  getDirAndFile(syncDirPath, syncFilePath, syncOrigPath);
  memset(syncOrigPath, 0, sizeof(syncOrigPath));
  strcpy(syncOrigPath, syncDirPath);
```
Bagian ini berfungsi untuk mendapatkan orignal path yang akan di simpan kedalam buffer `syncOrigPath`. original path ini didapat dari 
fungsi `getDirAndFile()` yang menggisi buffer `syncOrigPath` dengan path directory dan filenya yang akan di update menggunakan fungsi 
`strcpy`dengan isi dari buffer `syncDirpath`

```bash
  do {
    char syncPath[1000];
    int syncFd;
    memset(syncPath, 0, sizeof(syncPath));
    nextSync(syncDirPath);
    if (strcpyrcmp(syncDirPath, syncOrigPath) == 0) break;
    changePath(syncPath, syncDirPath, 0, 1);
    if (access(syncPath, F_OK) == -1) continue;
    sprintf(syncPath, "%s/%s", syncPath, syncFilePath);
    syncFd = open(syncPath, O_WRONLY);
    if (syncFd == -1) return -errno;
    pwrite(syncFd, buf, size, offset);
    close(syncFd);
  } while (1);
```
Pada bagian ini kami membuat infinite loop yang akan melakukan sinkronisasi pada setiap buffer `syncDirpath` dengan menggunakan fungsi 
nextSync yang sudah dijelaskan (diatas)

pada `if()` pertama buffer `syncOrigPath` yang berisi path asli akan di bandingkan dengan buffer `syncDirpath` yang menyimpan path yang 
sudah tersingkronisasi, dan jika kondisi `if()` terpenuhi dimana path dari `syncDirpath` sudah sinkron maka akan melakukan `break` dan 
loop akan berhenti, namun selama belum tersingkronisasi maka fungsi `changePath` akan dijalankan pada `syncDirPath` dan 
`syncOrigPath`dengan kondisi **(0.1)**  

`if()` kedua akan melakukan pengecekan pembukaan path yang ada di `syncPath` dengan fungsi `access`, jadi selama path ini belum bisa 
terakses maka loop akan tetap berjalan dan `syncPath` akan di update dengan ditambahkan `syncFilePath`, selanjutnya `syncPath` akan di buka dan di set ke `syncFd`

`if()` ketiga disini akan melakukan pengecekan untuk nilai `syncFd` dimana nilai tidak boleh -1 dan akan melakukan write dari `buf` kedalam `syncFd` dengan size dan offset yang sudah di tentukan, kemudian `syncFd` akan di close


```bash
  const char *desc[] = {path};
  logFile("INFO", "WRITE", res, 1, desc);

	if (res == -1) res = -errno;

	if(fi == NULL) close(fd);
	return res;
}
```
Pada bagian ini kami membuat system call untuk memebuat log file dengan fungsi `logFile()` yang sudah di definisikan di (soal4) dengan
format `("INFO", "WRITE", res, 1, desc)` dimana `desc` akan menunjuk kepada path sebagai keterangannya

kemudian `if()` disini akan memasukan error handling kedalam `res` jika nilainya `-1` dan akan menutup `fd` jika nilai `fi` == NULL atau
belum ada nilai yang di return kedalam variabel `fi` 

**_rmdir**
```bash
static int _rmdir(const char *path)
{
	char fpath[1000];
	changePath(fpath, path, 0, 0);
	int res;

	res = rmdir(fpath);

  char syncOrigPath[1000];
  char syncDirPath[1000];
  char syncFilePath[1000];
  memset(syncOrigPath, 0, sizeof(syncOrigPath));
  strcpy(syncOrigPath, path);
  getDirAndFile(syncDirPath, syncFilePath, syncOrigPath);
  memset(syncOrigPath, 0, sizeof(syncOrigPath));
  strcpy(syncOrigPath, syncDirPath);
  do {
    char syncPath[1000];
    memset(syncPath, 0, sizeof(syncPath));
    nextSync(syncDirPath);
    if (strcmp(syncDirPath, syncOrigPath) == 0) break;
    changePath(syncPath, syncDirPath, 0, 0);
    if (access(syncPath, F_OK) == -1) continue;
    sprintf(syncPath, "%s/%s", syncPath, syncFilePath);
    rmdir(syncPath);
  } while (1);
  const char *desc[] = {path};
  logFile("WARNING", "RMDIR", res, 1, desc);

	if (res == -1) return -errno;

	return 0;
}
```
System call ini berfungsi untuk menghandle proccess delete directory  
```bash
	char fpath[1000];
	changePath(fpath, path, 0, 0);
	int res;

	res = rmdir(fpath);
```
Pertama adalah system call ini akan memanggil fungsi `changePath` untuk `path` kedalam `fpath` dengan kondidi **(0.0)** atau kondisi 
`changePath` untuk *directory*

Kemudian menjalankan fungsinya `rmdir()` sendiri kepada `fpath` dan disimpan di varibael result

```bash
  char syncOrigPath[1000];
  char syncDirPath[1000];
  char syncFilePath[1000];
  memset(syncOrigPath, 0, sizeof(syncOrigPath));
  strcpy(syncOrigPath, path);
  getDirAndFile(syncDirPath, syncFilePath, syncOrigPath);
  memset(syncOrigPath, 0, sizeof(syncOrigPath));
  strcpy(syncOrigPath, syncDirPath); 
```
Bagian ini berfungsi untuk mendapatkan orignal path yang akan di simpan kedalam buffer `syncOrigPath`. original path ini didapat dari 
fungsi `getDirAndFile()` yang menggisi buffer `syncOrigPath` dengan path *directory* dan *filenya* yang akan di update menggunakan fungsi 
`strcpy`dengan isi dari buffer `syncDirpath`

```bash
  do {
    char syncPath[1000];
    memset(syncPath, 0, sizeof(syncPath));
    nextSync(syncDirPath);
    if (strcmp(syncDirPath, syncOrigPath) == 0) break;
    changePath(syncPath, syncDirPath, 0, 0);
    if (access(syncPath, F_OK) == -1) continue;
    sprintf(syncPath, "%s/%s", syncPath, syncFilePath);
    rmdir(syncPath);
  } while (1);
```
Pada bagian ini kami membuat infinite loop yang akan melakukan sinkronisasi pada setiap buffer `syncDirpath` dengan menggunakan fungsi nextSync yang sudah dijelaskan (diatas)

pada `if()` pertama buffer `syncOrigPath` yang berisi path asli akan di bandingkan dengan buffer `syncDirpath` yang menyimpan path yang 
sudah tersingkronisasi, dan jika kondisi `if()` terpenuhi dimana path dari `syncDirpath` sudah sinkron maka akan melakukan `break` dan 
loop akan berhenti, namun selama belum tersingkronisasi maka fungsi `changePath` akan dijalankan pada `syncDirPath` dan 
`syncOrigPath`dengan kondisi **(1.0)**  

`if()` kedua akan melakukan pengecekan pembukaan path yang ada di `syncPath` dengan fungsi `access`, jadi selama path ini belum bisa 
terakses maka loop akan tetap berjalan dan `syncPath` akan di update dengan ditambahkan `syncFilePath` dan directory akan dibuat dengan 
nama yang sudah tersinkronisasi yang ada pada buffer `syncPath`

```bash
const char *desc[] = {path};
  logFile("WARNING", "RMDIR", res, 1, desc);

	if (res == -1) return -errno;

	return 0;
```
Pada bagian system call membuat log file dengan fungsi `logFile()` yang sudah di definisikan di (soal4) dengan format 
`("INFO", "MKDIR", res, 1, desc)` dimana `desc` akan menunjuk kepada path sebagai keterangannya. 

**_link**
```bash
static int _link(const char *from, const char *to)
{
	char ffrom[1000];
	char fto[1000];
	changePath(ffrom, from, 0, 1);
	changePath(fto, to, 0, 1);
	int res;

	res = link(ffrom, fto);

  const char *desc[] = {from, to};
  logFile("INFO", "LINK", res, 2, desc);
	if (res == -1) return -errno;

	return 0;
}
```
Pertama tama system call ini akan menjalankan `changePath()` pada dengan fpath `ffrom` & `fto` dan `from` & `to` dengan kondisi **(0.1)** 
yang berarti  hanya `isFileAsked` pada fungsi `changePath()` yang berjalan 

Selanjutnya bagian ini berfungsi untuk menjalankan fungsi `link()` itu sendiri yang berfungsi untuk membuat sebuah link (directory entry) 
untuk file yang sudah ada pada (`ffrom`)

Pada setiap penggunaan system call akan ada sebuah log yang di write kedalam log file menggunakan fungsi `logFile`, untuk system call ini
pendefinisian log adalah dengan level *info*, dengan nama system call *LINK*, *result* dari link, *2* dan *des* yang menunjuk kepada
path yang tentukan.  
  
**chmod**
```bash
static int _chmod(const char *path, mode_t mode)
{
	char fpath[1000];
	changePath(fpath, path, 0, 1);
  if (access(fpath, F_OK) == -1) {
    memset(fpath, 0, sizeof(fpath));
    changePath(fpath, path, 0, 0);
  }
	int res;
	res = chmod(fpath, mode);

  char modeBuff[100];
  sprintf(modeBuff, "%d", mode);
  const char *desc[] = {path, modeBuff};
  logFile("INFO", "CHMOD", res, 2, desc);

	if (res == -1) return -errno;

	return 0;
}
```
Pertama tama System call ini akan menjalankan fungsi `changePath()` pada `fpath` dengan kondisi **(0.1)** karena `_chmod` bukan 
merupakan write operation. Selanjutnya akan dilakukan pengecekan eksistensi path yang diinputkan sebagai *file* atau *directory* dan 
sudah terenkripsi atau belum menggunakan `(access(fpath, F_OK) == -1)` jika tidak dapat di akses sebagai *file* maka akan dijalankan 
fungsi `changePath`untuk *path* kedalam *fpath* dengan kondisi **(0.0)** atau kondisi untuk direktori

Selanjutnya adalah menjalankan fungsi `chmod()` itu sendiri yang berfungsi untuk merubah permission pada file kepada `fpath` dengan mode 
yang sudah ditentukan  

Pada setiap penggunaan system call akan ada sebuah log yang di write kedalam log file menggunakan fungsi `logFile`, untuk system call ini
pendefinisian log adalah dengan level *info*, dengan nama system call *CHMOD*, *result* dari `chmod()`, *2* dan *des* yang menunjuk 
kepada path dan mode yang sudah di tentukan.  

**_chown**
```bash
static int _chown(const char *path, uid_t uid, gid_t gid)
{
	char fpath[1000];
  changePath(fpath, path, 0, 1);
  if (access(fpath, F_OK) == -1) {
    memset(fpath, 0, sizeof(fpath));
    changePath(fpath, path, 0, 0);
  }
	int res;
	res = lchown(fpath, uid, gid);

  char uidBuff[100];
  char gidBuff[100];
  sprintf(uidBuff, "%d", uid);
  sprintf(gidBuff, "%d", gid);
  const char *desc[] = {path, uidBuff, gidBuff};
  logFile("INFO", "CHOWN", res, 3, desc);

	if (res == -1) return -errno;

	return 0;
}
```
Pertama tama System call ini akan menjalankan fungsi `changePath()` pada `fpath` dengan kondisi **(0.1)** karena `_chown` bukan 
merupakan write operation. Selanjutnya akan dilakukan pengecekan eksistensi path yang diinputkan sebagai *file* atau *directory* dan 
sudah terenkripsi atau belum menggunakan `(access(fpath, F_OK) == -1)` jika tidak dapat di akses sebagai *file* maka akan dijalankan 
fungsi `changePath` untuk *path* kedalam *fpath* dengan kondisi **(0.0)** atau kondisi untuk direktori

Selanjutnya adalah menjalankan fungsi `lchown(fpath, uid_t, gid_t)` itu sendiri yang berfungsi untuk merubah owner dari sebuah froup 
file kepada `fpath` serta owner(uid) dan group(gid) nya yang sudah di tentukan, kemudian kami membuat dua buah buffer untuk menyimpan 
owner dan groupnya yang nantinya akan digunakan sebagai desc dari fungsi `logFile()`

Pada setiap penggunaan system call akan ada sebuah log yang di write kedalam log file menggunakan fungsi `logFile`, untuk system call ini
pendefinisian log adalah dengan level *info*, dengan nama system call *CHOWN*, *result* dari `chown()`, *3* dan *des* yang menunjuk 
kepada path yang sudah berisi dua buffer untuk owner dan group tadi.  
  
**truncate**
```bash
static int _truncate(const char *path, off_t size)
{
	char fpath[1000];
  changePath(fpath, path, 0, 1);
  if (access(fpath, F_OK) == -1) {
    memset(fpath, 0, sizeof(fpath));
    changePath(fpath, path, 0, 0);
  }
	int res;
	res = truncate(fpath, size);

  const char *desc[] = {path};
  logFile("INFO", "TRUNCATE", res, 1, desc);

	if (res == -1) return -errno;

	return 0;
}
```
Pertama tama System call ini akan menjalankan fungsi `changePath()` pada `fpath` dengan kondisi **(0.1)** karena `_truncate` bukan 
merupakan write operation. Selanjutnya akan dilakukan pengecekan eksistensi path yang diinputkan sebagai *file* atau *directory* dan 
sudah terenkripsi atau belum menggunakan `(access(fpath, F_OK) == -1)` jika tidak dapat di akses sebagai *file* maka akan dijalankan 
fungsi `changePath`untuk *path* kedalam *fpath* dengan kondisi **(0.0)** atau kondisi untuk direktori

Selanjutnya adalah menjalankan fungsi `truncate()` itu sendiri yang berfungsi untuk mengecilkan atau memperbesar ukuran dari suatu file 
dengan ukuran yang sudah di tentukan kepada `fpath` dan dengan ukuran `size`

Pada setiap penggunaan system call akan ada sebuah log yang di write kedalam log file menggunakan fungsi `logFile`, untuk system call ini
pendefinisian log adalah dengan level *info*, dengan nama system call *TRUNCATE*, *result* dari `truncate()`, *1* dan *des* yang 
menunjuk kepada path yang sudah ditentukan.    
    
**utimens**
```bash
static int _utimens(const char *path, const struct timespec ts[2])
{
	char fpath[1000];
  changePath(fpath, path, 0, 1);
  if (access(fpath, F_OK) == -1) {
    memset(fpath, 0, sizeof(fpath));
    changePath(fpath, path, 0, 0);
  }
	int res;
	res = utimensat(0, fpath, ts, AT_SYMLINK_NOFOLLOW);

  const char *desc[] = {path};
  logFile("INFO", "UTIMENSAT", res, 1, desc);

	if (res == -1) return -errno;

	return 0;
}
```
Pertama tama System call ini akan menjalankan fungsi `changePath()`pada `fpath` dengan kondisi **(0.1)** karena `_utimens` bukan 
merupakan write operation. Selanjutnya akan dilakukan pengecekan eksistensi path yang diinputkan sebagai *file* atau *directory* dan 
sudah terenkripsi atau belum menggunakan `(access(fpath, F_OK) == -1)` jika tidak dapat di akses sebagai *file* maka akan dijalankan 
fungsi `changePath` untuk *path* kedalam *fpath* dengan kondisi **(0.0)** atau kondisi untuk direktori

Selanjutnya adalah menjalakan fungsi `utimensat(0, fpath, ts, AT_SYMLINK_NOFOLLOW)` itu sendiri yang berfungsi untuk mengubah timestamp 
sebuah file dengan presisi nano second kepada `fpath` dengan `dirfd` = 0, `ts` sebagai struct timespec dan `AT_SYMLINK_NOFOLLOW` sebagai 
flags

Pada setiap penggunaan system call akan ada sebuah log yang di write kedalam log file menggunakan fungsi `logFile`, untuk system call ini
pendefinisian log adalah dengan level *info*, dengan nama system call *TRUNCATE*, *result* dari `utimensat()`, *1* dan *des* yang 
menunjuk kepada path yang sudah ditentukan.  
  
**open**
```bash
static int _open(const char *path, struct fuse_file_info *fi)
{
	char fpath[1000];
  changePath(fpath, path, 0, 1);

	int res;
	res = open(fpath, fi->flags);

  const char *desc[] = {path};
  logFile("INFO", "OPEN", res, 1, desc);

	if (res == -1) return -errno;

	fi->fh = res;
	return 0;
}
```
Pada system call ini kami menggunakan `struct fuse_file_info` yang berfungsi sebagai file handline dengan pointer `fi`.
Pertama tama System call ini akan menjalankan fungsi `changePath()` pada `fpath` dengan kondisi **(0.1)** karena `_open` bukan merupakan 
write operation.

Selanjutnya adalah menjalankan fungsi `open(fpath, fi->flags)` itu pada `fpath` dengan mengguankan `flags`

Pada setiap penggunaan system call akan ada sebuah log yang di write kedalam log file menggunakan fungsi `logFile`, untuk system call ini
pendefinisian log adalah dengan level *info*, dengan nama system call *OPEN*, *result* dari `open()`, *1* dan *des* yang menunjuk 
kepada path yang sudah ditentukan.  

**read**
```bash
static int _read(const char *path, char *buf, size_t size, off_t offset,
		    struct fuse_file_info *fi)
{
	char fpath[1000];
	changePath(fpath, path, 0, 1);

	int fd;
	int res;

	if(fi == NULL) fd = open(fpath, O_RDONLY);
	else fd = fi->fh;

	if (fd == -1) return -errno;

	res = pread(fd, buf, size, offset);

  const char *desc[] = {path};
  logFile("INFO", "READ", res, 1, desc);

	if (res == -1) res = -errno;

	if(fi == NULL) close(fd);
	return res;
}
```
Pada system call ini kami menggunakan `struct fuse_file_info` yang berfungsi sebagai file handline dengan pointer `fi`
Pertama tama System call ini akan menjalankan fungsi `changePath()` pada `fpath` dengan kondisi **(0.1)** karena `_read` bukan merupakan 
write operation.

selanjutnya disini kami menambahkan satu variabel `fd` untuk menyimpan `fpath` yang sudah di `open()` jika `fi` masih bernilai NULL,
`fi` disini merupakan unusef variabel yang didefinisikan pada system call `_write`, namun jika `fi` tidak NULL maka `fd` disini akan 
di update dengan `i->fh` yang sudah ada.  

Selanjutnya adalah mmenjalankan fungsi `pread(fd, buf, size, offset)` itu sendiri yang berfungsi membaca sebuah file descriptor dengan 
offset yang sudah di tentukan.  

Pada setiap penggunaan system call akan ada sebuah log yang di write kedalam log file menggunakan fungsi `logFile`, untuk system call ini
pendefinisian log adalah dengan level *info*, dengan nama system call *READ*, *result* dari `pread()`, *1* dan *des* yang menunjuk 
kepada path yang sudah ditentukan. Terkahir `fd` akan di `close()` jika nilai `fi` NULL

**statfs**
```bash
static int _statfs(const char *path, struct statvfs *stbuf)
{
	char fpath[1000];
	changePath(fpath, path, 0, 1);
	int res;

	res = statvfs(fpath, stbuf);

  const char *desc[] = {path};
  logFile("INFO", "STATFS", res, 1, desc);

	if (res == -1) return -errno;

	return 0;
}
```
Pertama tama System call ini akan menjalankan fungsi `changePath()` pada `fpath` dengan kondisi **(0.1)** karena `_statfs` bukan 
merupakan write operation.

Selanjutnya adalah menjalankan fungsi `statvfs(fpath, stbuf)` itu sendiri yang berfungsi untuk mengambil statistic filesystem

Pada setiap penggunaan system call akan ada sebuah log yang di write kedalam log file menggunakan fungsi `logFile`, untuk system call ini
pendefinisian log adalah dengan level *info*, dengan nama system call *STATFS*, *result* dari `statvfs()`, *1* dan *des* yang menunjuk 
kepada path yang sudah ditentukan.


**Kesulitan:**  
**ScreenShot**  
**Contoh logging**  


## Soal 2
Source Code : [source](https://github.com/DSlite/SoalShiftSISOP20_modul4_T08/blob/master/ssfs.c)

**Deskripsi**
Soal meminta kami untuk membuat sebuah metode enkripsi V2 yang berjalan jika sebuah directory dibuat atau direname dengan 
awalan "encv2_" Yang dimana setiap directory yang sudah terenkripsi akan ter-dekrip jika namanya di rename. Setiap file yang ada pada 
direktori asli akan dijadikan bagian-bagian kecil sebesar 1024 bytes dan menjadi normal ketika diakses melalui filesystem rancangan 
jasir. Sebagai contoh, file File_Contoh.txt berukuran 5 kB pada direktori asli akan menjadi 5 file kecil yakni: File_Contoh.txt.000, 
File_Contoh.txt.001, File_Contoh.txt.002, File_Contoh.txt.003, dan File_Contoh.txt.004. Dan berlaku untuk setiap isi dari subfolder

**Asumsi Soal**
**Pembahasan**

***Fungsi 





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
pertama kami melakukan pendefisinian path dari file log yang akan di lanjutkan dengan fungsi logFilenya sendiri

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
  for (int i = 0; i < lenDesc; i++) {	
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
