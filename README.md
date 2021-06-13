# soal-shift-sisop-modul-4-E05-2021

# Anggota kelompok
- Nur Hidayati  05111940000028
- Adrian        05111940000130
- Ahmad Aunul   05111940000164

## Soal nomor 1
Di suatu jurusan, terdapat admin lab baru yang super duper gabut, ia bernama Sin. Sin baru menjadi admin di lab tersebut selama 1 bulan. Selama sebulan tersebut ia bertemu orang-orang hebat di lab tersebut, salah satunya yaitu Sei. Sei dan Sin akhirnya berteman baik. Karena belakangan ini sedang ramai tentang kasus keamanan data, mereka berniat membuat filesystem dengan metode encode yang mutakhir. Berikut adalah filesystem rancangan Sin dan Sei :  

Note : 
```
Semua file yang berada pada direktori harus ter-encode menggunakan Atbash cipher(mirror).
Misalkan terdapat file bernama kucinglucu123.jpg pada direktori DATA_PENTING
“AtoZ_folder/DATA_PENTING/kucinglucu123.jpg” → “AtoZ_folder/WZGZ_KVMGRMT/pfxrmtofxf123.jpg”
Note : filesystem berfungsi normal layaknya linux pada umumnya, Mount source (root) filesystem adalah directory /home/[USER]/Downloads, dalam penamaan file ‘/’ diabaikan, dan ekstensi tidak perlu di-encode.
Referensi : https://www.dcode.fr/atbash-cipher
```

**a.** Jika sebuah direktori dibuat dengan awalan “AtoZ_”, maka direktori tersebut akan menjadi direktori ter-encode.  

**b.** Jika sebuah direktori di-rename dengan awalan “AtoZ_”, maka direktori tersebut akan menjadi direktori ter-encode.

**c.** Apabila direktori yang terenkripsi di-rename menjadi tidak ter-encode, maka isi direktori tersebut akan terdecode.

**d.** Setiap pembuatan direktori ter-encode (mkdir atau rename) akan tercatat ke sebuah log. Format : /home/[USER]/Downloads/[Nama Direktori] → /home/[USER]/Downloads/AtoZ_[Nama Direktori]  

**e.** Metode encode pada suatu direktori juga berlaku terhadap direktori yang ada di dalamnya.(rekursif)

## Soal nomor 4
Untuk memudahkan dalam memonitor kegiatan pada filesystem mereka Sin dan Sei membuat sebuah log system dengan spesifikasi sebagai berikut. 

**a.** Log system yang akan terbentuk bernama “SinSeiFS.log” pada direktori home pengguna (/home/[user]/SinSeiFS.log). Log system ini akan menyimpan daftar perintah system call yang telah dijalankan pada filesystem.

**b.** Karena Sin dan Sei suka kerapian maka log yang dibuat akan dibagi menjadi dua level, yaitu INFO dan WARNING.

**c.** Untuk log level WARNING, digunakan untuk mencatat syscall rmdir dan unlink.

**d.** Sisanya, akan dicatat pada level INFO.

**e.** Format untuk logging yaitu:

[Level]::[dd][mm][yyyy]-[HH]:[MM]:[SS]:[CMD]::[DESC :: DESC]

Level : Level logging, dd : 2 digit tanggal, mm : 2 digit bulan, yyyy : 4 digit tahun, HH : 2 digit jam (format 24 Jam),MM : 2 digit menit, SS : 2 digit detik, CMD : System Call yang terpanggil, DESC : informasi dan parameter tambahan

INFO::28052021-10:00:00:CREATE::/test.txt
INFO::28052021-10:01:00:RENAME::/test.txt::/rename.txt


## Penjelasan penyelesaian soal
### No 1
System call yang dipakai dalam soal ini yaitu :
```
static const struct fuse_operations _oper = {
    .getattr = _getattr,
    .readdir = _readdir,
    .mkdir = _mkdir,
    .unlink = _unlink,
    .rmdir = _rmdir,
    .rename = _rename,
    .utimens = _utimens,
    .read = _read,
    .write = _write,
    .statfs = _statfs,
    .open = _open,
    .create = _create,
    .truncate = _truncate,
};
```

Fungsi dari system call di atas yaitu :
- .getattr = untuk mendapatkan stat dari path
- .readdir = untuk membaca direktori
- .mkdir = untuk membuat direktori
- .unlink = untuk menghapus file
- .rmdir = untuk menghapus direktori
- .rename = untuk merename path (dari path awal menjadi path yang diinginkan)
- .utimens = untuk mengubah time dari path
- .read = untuk membaca isi dari path
- .write = untuk menulis ke dalam path
- .statfs = untuk mengembalikan informasi mengenai filesystem mount
- .open = untuk membuka path 
- .create = untuk membuka file berdasarkan pathame
- .truncate = untuk melakukan truncate pada path

Fungsi yang akan dipakai yaitu :

Fungsi FileExtension untuk mendapatkan nama file dari sebuah path. Fungsi strrchr untuk mendapatkan posisi terakhir dari sebuah karakter
```c
char *FileExtension(char *filename)
{
    char *temp = strrchr(filename, '/');
    return temp + 1;
}
```

Fungsi getDirAndFile untuk memisahkan file dan direktori dari sebuah filepath. Fungsi strtok untuk memisahkan sebuah string dengan delimmiter tertentu (di sini menggunakan '/'). Nama file akan dimasukkan ke dalam variabel ```file``` dan  nama folder akan dimasukkan ke dalam variabel ```dir```
```c
void getDirAndFile(char *dir, char *file, char *path)
{
    char buff[1000];
    memset(dir, 0, 1000);
    memset(file, 0, 1000);
    strcpy(buff, path);
    char *token = strtok(buff, "/");
    while (token != NULL)
    {
        sprintf(file, "%s", token);
        token = strtok(NULL, "/");
        if (token != NULL)
        {
            strcat(dir, "/");
            strcat(dir, file);
        }
    }
}
```

Fungsi decrypt untuk melakukan enkripsi dan dekripsi suatu path. Metode dekripsi yang kita gunakan yaitu atbash cipher. Syntax untuk melakukan decrypt yaitu ```str[i] = 'Z' + 'A' - str[i]``` atau ```str[i] = 'z' + 'a' - str[i]```
```c
void decrypt(char *path, int isEncrypt)
{
    char *str = path;
    int i = 0;
    while (str[i] != '\0')
    {
        if (!((str[i] >= 0 && str[i] < 65) || (str[i] > 90 && str[i] < 97) || (str[i] > 122 && str[i] <= 127)))
        {
            if (str[i] >= 'A' && str[i] <= 'Z')
                str[i] = 'Z' + 'A' - str[i];
            if (str[i] >= 'a' && str[i] <= 'z')
                str[i] = 'z' + 'a' - str[i];
        }

        if (((str[i] >= 0 && str[i] < 65) || (str[i] > 90 && str[i] < 97) || (str[i] > 122 && str[i] <= 127)))
            str[i] = str[i];

        i++;
    }
}
```

Fungsi changePath untuk mengubah path menjadi sesuai path tujuan (fpath). Terdapat 4 argumen yaitu path untuk path awal, fpath unutk path tujuan, isWriteOper bernilai 1 jika hanya ingin melakukan enkripsi pada nama direktori saja (contoh : ketika membuat file pada direktori yang terenkripsi) , isFileAsked bernilai 1 jika akan melakukan dekripsi pada folder dan file. Pertama, kita mencari tahu apakah dalam path terdapat string 'AtoZ_' atau tidak dengan menggunakan ```strstr```. Jika terdapat, maka kita akan tentukan apakah terdapat direktori lagi di dalam nya atau tidak pada syntax ```if (strstr(ptr + 1, "/") != NULL)```. Jika iya maka state kita ubah ke nilai 1
```c
void changePath(char *fpath, const char *path, int isWriteOper, int isFileAsked)
{
    char *ptr = strstr(path, "/AtoZ_");
    int state = 0;
    if (ptr != NULL)
    {
        if (strstr(ptr + 1, "/") != NULL)
            state = 1;
    }
    char fixPath[1000];
    memset(fixPath, 0, sizeof(fixPath));
```

Kemudian, jika pointer tidak null dan state bernilai true maka kita isi pathEncryptedBuff dengan variabel ptr sedangkan pathEncvDirBuff kita isi dengna path folder nya. Jika isWriteOper bernilai true, kita pisahkan file dan folder nya dengan fungsi ```getDirAndFile``` dan kita decrpt folder nya saja. Kemudian, kita susun hasilnya sesuai format ke dalam fixPath
```c
if (ptr != NULL && state)
    {
        ptr = strstr(ptr + 1, "/");
        char pathEncvDirBuff[1000];
        char pathEncryptedBuff[1000];
        strcpy(pathEncryptedBuff, ptr);
        strncpy(pathEncvDirBuff, path, ptr - path);

        if (isWriteOper)
        {
            char pathFileBuff[1000];
            char pathDirBuff[1000];
            getDirAndFile(pathDirBuff, pathFileBuff, pathEncryptedBuff);
            decrypt(pathDirBuff, 0);

            strcat(fixPath, pathEncvDirBuff);
            strcat(fixPath, pathDirBuff);
            strcat(fixPath, "/");
            strcat(fixPath, pathFileBuff);
        }
```

Jika isFileAsked bernilai true, kita akan memisahkan direktori dan file dengan fungsi ```getDirAndFile```. Kemudian, kita cari ekstensi dari file dengan fungsi ```strrchr(pathFileBuff, '.')```. Fungsi ```if (whereIsExtension - pathFileBuff < 1)``` untuk file yang tidak mempunyai ekstensi. Di sini kita bisa langsung melakukan dekripsi folder dan file nya .Else nya untuk file yang mempunyai ekstensi. Untuk file yang mempunyai ekstensi, file dan folder akan diencrypt sedangkan ekstensi file tidak akan di decrypt.
```c
else if (isFileAsked)
        {
            char pathFileBuff[1000];
            char pathDirBuff[1000];
            char pathExtBuff[1000];
            getDirAndFile(pathDirBuff, pathFileBuff, pathEncryptedBuff);
            char *whereIsExtension = strrchr(pathFileBuff, '.');

            if (whereIsExtension - pathFileBuff < 1)
            {
                decrypt(pathDirBuff, 0);
                decrypt(pathFileBuff, 0);
                strcat(fixPath, pathEncvDirBuff);
                strcat(fixPath, pathDirBuff);
                strcat(fixPath, "/");
                strcat(fixPath, pathFileBuff);
            }
            else
            {
                char pathJustFileBuff[1000];
                memset(pathJustFileBuff, 0, sizeof(pathJustFileBuff));
                strcpy(pathExtBuff, whereIsExtension);
                snprintf(pathJustFileBuff, whereIsExtension - pathFileBuff + 1, "%s", pathFileBuff);
                decrypt(pathDirBuff, 0);
                decrypt(pathJustFileBuff, 0);

                strcat(fixPath, pathEncvDirBuff);
                strcat(fixPath, pathDirBuff);
                strcat(fixPath, "/");
                strcat(fixPath, pathJustFileBuff);
                strcat(fixPath, pathExtBuff);
            }
        }

```

Selain itu, untuk else pertama yang bukan merupakan isWriteOper, kita langsung melakukan decrpyt pada pathEncyptedBuff. Setelah itu else berikutnya yaitu jika tidak ada 'AtoZ_' maka akan langsung disalin langsung ke fixPath. Jika path hanya '/', maka akan langsung masukkan ke fpath sedangkan jika tidak maka gabungkan dirpath dan fixPath
```c
else
        {
            decrypt(pathEncryptedBuff, 0);
            strcat(fixPath, pathEncvDirBuff);
            strcat(fixPath, pathEncryptedBuff);
        }
    }
    else
    {
        strcpy(fixPath, path);
    }
    if (strcmp(path, "/") == 0)
    {
        sprintf(fpath, "%s", dirpath);
    }
    else
    {
        sprintf(fpath, "%s%s", dirpath, fixPath);
    }
```

Fungsi logFile untuk mencatatkan hasil log ke dalam sebuah file sesuai permintaan soal no 1d. Di mana kita membuka file nya terlebih dahulu, kemudiann tuliskan isi file nya menggunakan ```fprintf``` dan close file nya
```c
void logFile1(char *pathasal, char *pathakhir)
{
    char fileExt[300], fileExt1[300];
    memset(fileExt, 0, sizeof(fileExt));

    strcpy(fileExt, FileExtension(pathasal));
    strcpy(fileExt1, FileExtension(pathakhir));

    FILE *f = fopen(logpath1, "a");
    fprintf(f, "/home/oni/Downloads/%s -> /home/oni/Downloads/%s\n", fileExt, fileExt1);
    fclose(f);
}
```

Fungsi readdir untuk membaca direktori. Pertama kita memanggil fungsi ```changePath``` untuk mendapatkan path yang diinginkan. Kemudian kita buka path nya dengan ```opendir```. Jika error akan mengeluarkan error. 
```c
static int _readdir(const char *path, void *buf, fuse_fill_dir_t filler, off_t offset, struct fuse_file_info *fi)
{
    char fpath[1000];
    changePath(fpath, path, 0, 0);

    DIR *dp;
    struct dirent *de;

    (void)offset;
    (void)fi;

    dp = opendir(fpath);
    if (dp == NULL)
    {
        const char *desc[] = {path};
        logFile("INFO", "READDIR", -1, 1, desc);
        return -errno;
    }
```
Kemudian, jika dari path terdapat "/AtoZ_" maka kita akan menentukan lagi apakah itu termasuk folder atau file. Jika termasuk file maka kita akan tentukan terlebih dahulu apakah mempunyai extension atau tidak sedangkan jika termasuk folder maka kita akan melakukan decrypt. Hasil nya akan kita langsung masukkan ke dalam buff dengan fungsi ```filler```
```c
if (strstr(path, "/AtoZ_") != NULL)
        {
            char encryptThis[1000];
            strcpy(encryptThis, de->d_name);
            if (de->d_type == DT_REG)
            {
                char *whereIsExtension = strrchr(encryptThis, '.');
                if (whereIsExtension - encryptThis < 1)
                {
                    decrypt(encryptThis, 1);
                }
                else
                {
                    char pathFileBuff[1000];
                    char pathExtBuff[1000];
                    strcpy(pathExtBuff, whereIsExtension);
                    snprintf(pathFileBuff, whereIsExtension - encryptThis + 1, "%s", encryptThis);
                    decrypt(pathFileBuff, 1);
                    memset(encryptThis, 0, sizeof(encryptThis));
                    strcat(encryptThis, pathFileBuff);
                    strcat(encryptThis, pathExtBuff);
                }
            }
            else if (de->d_type == DT_DIR)
            {
                decrypt(encryptThis, 1);
            }

            if (filler(buf, encryptThis, &st, 0))
                break;
        }

        else
        {
            if (filler(buf, de->d_name, &st, 0))
                break;
        }
```

Isi file log untuk no 1d  
![1d](https://user-images.githubusercontent.com/65168221/121800639-4f561b00-cc5d-11eb-9ae6-174b17fe9106.jpg)

Sebelum testing  
![seblum](https://user-images.githubusercontent.com/65168221/121800661-685ecc00-cc5d-11eb-9e82-4e45763eabf7.jpg)


Sesudah testing  
![sesudah](https://user-images.githubusercontent.com/65168221/121800679-7ad90580-cc5d-11eb-82d2-dae1401a037f.jpg)


Kendala :
- File pernah tidak bisa dibuka pada filesystem mount 
- Kesusahan untuk melakukan debugging
- Pernah tidak bisa membuat file di dalam filesystem mount

### No 4
Fungsi logFile untuk menuliskan log ke dalam file. Di sini, kita gunakan ```time(&t)``` karena kita perlu menggunakan waktu dalam penulisan log. Serta, kita melakukan for untuk melakukan perluangan pada description.
```c
void logFile(char *level, char *cmd, int res, int lenDesc, const char *desc[])
{
    FILE *f = fopen(logpath, "a");
    time_t t;
    struct tm *tmp;
    char timeBuff[100];

    time(&t);
    tmp = localtime(&t);
    strftime(timeBuff, sizeof(timeBuff), "%d%m%Y-%H:%M:%S", tmp);

    fprintf(f, "%s::%s::%s::%d", level, timeBuff, cmd, res);
    for (int i = 0; i < lenDesc; i++)
    {
        fprintf(f, "::%s", desc[i]);
    }
    fprintf(f, "\n");

    fclose(f);
}
```

Isi file log untuk no4
![1623573108987](https://user-images.githubusercontent.com/65168221/121800687-84626d80-cc5d-11eb-9204-81de70054342.jpg)

Kendala : 
- Agak bingung dengan description 
