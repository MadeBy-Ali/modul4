# Shift 4 SISOP 2020 - T08
Penyelesaian Soal Shift 4 Sistem Operasi 2020
Kelompok T08
  * I Made Dindra Setyadharma (05311840000008)
  * Muhammad Irsyad Ali (05311840000041)

---
## Table of Contents
* [Soal 4](#soal-4)


---

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
