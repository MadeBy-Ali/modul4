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
awalan "encv_1" dengan key 
`9(ku@AW1[Lmvgax6q`5Y2Ry?+sF!^HKQiBXCUSe&0M.b%rI'7d)o4~VfZ*{#:}ETt$3J-zpc]lnh8,GwP_ND|jO`
Yang dimana setiap directory yang sudah terenkripsi akan ter-dekrip jika namanya di rename. Setiap penggunaan "mkdir" atau "rename" maka akan tercatat kedalam log file, metode enkripsi disini haarus mengabaikan "/" pada penamaan file dan metode ini berlaku untuk setiap directory yang ada di dalam suatu directory


**Asumsi Soal**
Kami mengasumsikan bahwa

**Pembahasan**
Pertama untuk membuat sebuah metode enkripsi seperti yang sudah di terangkan pada soal. Kami membuat tiga systemcall yaitu mkdir,create dan write yang masing masingnya memiliki fungsi untuk handle kondisi jika ada pembuatan directory dengan nama yang ditentukan, kondisi
jika ada directory yang di rename seusai dengan nama yang sudah di tentukan, dan write untuk

Setiap system call akan menggunakan fungsi yang bernama `changePath()` yang berfungsi untuk dekripsi dan melakukan pengecekan untuk 
setiap pathfile extension dan directory   


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

```bash
if (isWriteOper) {
      char pathFileBuff[1000];
      char pathDirBuff[1000];
      getDirAndFile(pathDirBuff, pathFileBuff, pathEncryptedBuff);
      decrypt(pathDirBuff, 0);
      sprintf(fixPath, "%s%s/%s", pathEncvDirBuff, pathDirBuff, pathFileBuff);
```

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
