# Laporan Resmi Soal Shift Modul 3 Jarkom 2021

## Anggota Kelompok D10
- Mohammad Faderik I H (05111940000023)
- Farhan Arifandi (05111940000061)
- Yusril Zubaydi (05111940000160)


## Prefix IP
Prefix IP untuk kelompok kami adalah `10.26`.

## Soal 1
> ![image](https://user-images.githubusercontent.com/70105993/141472155-85410a83-129d-4309-8d61-8655331c9899.png)
> 
> Luffy bersama Zoro berencana membuat peta tersebut dengan kriteria EniesLobby sebagai DNS Server, Jipangu sebagai DHCP Server, Water7 sebagai Proxy Server

### Jawaban:

## Soal 2
> dan Foosha sebagai DHCP Relay

### Jawaban:

## Soal 3
> Ada beberapa kriteria yang ingin dibuat oleh Luffy dan Zoro, yaitu:
> 1. Semua client yang ada HARUS menggunakan konfigurasi IP dari DHCP Server.
> 2. Client yang melalui Switch1 mendapatkan range IP dari [prefix IP].1.20 - [prefix IP].1.99 dan [prefix IP].1.150 - [prefix IP].1.169


### Jawaban:

## Soal 4
> 3. Client yang melalui Switch3 mendapatkan range IP dari [prefix IP].3.30 - [prefix IP].3.50

### Jawaban:

## Soal 5
> 4. Client mendapatkan DNS dari EniesLobby dan client dapat terhubung dengan internet melalui DNS tersebut.

### Jawaban:

## Soal 6
> 5. Lama waktu DHCP server meminjamkan alamat IP kepada Client yang melalui Switch1 selama 6 menit sedangkan pada client yang melalui Switch3 selama 12 menit. Dengan waktu maksimal yang dialokasikan untuk peminjaman alamat IP selama 120 menit.

### Jawaban:

## Soal 7
> Luffy dan Zoro berencana menjadikan Skypie sebagai server untuk jual beli kapal yang dimilikinya dengan alamat IP yang tetap dengan IP [prefix IP].3.69

### Jawaban:
**Skypie**

Dapatkan hwaddress milik Skypie dengan perintah `ip a`, lalu lihat pada bagian `link/ether` sesuai dengan interface yang terhubung dengan router (dalam hal ini **eth0**). Yang didapatkan adalah `f6:11:75:ac:47:fc`.

![image](https://user-images.githubusercontent.com/70105993/141478652-e3c0e032-d809-47d9-9a5c-a976482ea1b2.png)

**Jipangu**

Tambahkan pengaturan seperti berikut pada file `/etc/dhcp/dhcpd.conf`

```vim
host Skypie {
        hardware ethernet f6:11:75:ac:47:fc;
        fixed-address 10.26.3.69;
}
```

![Screenshot_43](https://user-images.githubusercontent.com/70105993/141481022-52dbc2ef-d1ef-48a9-bfab-6c60e0ee8a9c.png)

**Skypie**

1. Edit network configuration pada node Skypie seperti berikut.
    ```
    auto eth0
    iface eth0 inet dhcp
    hwaddress ether f6:11:75:ac:47:fc
    ```

    ![image](https://user-images.githubusercontent.com/70105993/141478822-7e69e74e-511f-4d2a-97bc-55149dfa4dd9.png)

2. Cek dengan perintah `ip a` apakah IP sudah sesuai ketentuan yaitu `[prefix IP].3.69`.

    ![image](https://user-images.githubusercontent.com/70105993/141479085-53a87530-fdca-40d9-b7f2-0aed13ba72b1.png)

## Soal 8
> Loguetown digunakan sebagai client Proxy agar transaksi jual beli dapat terjamin keamanannya, juga untuk mencegah kebocoran data transaksi.
> Pada Loguetown, proxy harus bisa diakses dengan nama jualbelikapal.yyy.com dengan port yang digunakan adalah 5000

### Jawaban:
**Water7**

1. Tambahkan konfigurasi berikut pada file `/etc/squid/squid.conf` untuk mengatur nama dan port proxy serta memperbolehkan akses http untuk semua kondisi.
    ```
    http_port 5000
    visible_hostname jualbelikapal.d10.com

    http_access allow all
    ```
    
   ![Screenshot_45](https://user-images.githubusercontent.com/70105993/141481206-6a111c66-0ffc-4089-bab8-7721ef6d5b76.png)

2. Restart squid dengan perintah `service squid restart`.

**Loguetown**

1. Aktifkan proxy dan cek konfigurasi proxy dengan perintah berikut.
    ```bash
    export http_proxy=http://10.26.2.3:5000
    env | grep -i proxy
    ```

    ![image](https://user-images.githubusercontent.com/70105993/141480776-89c00d94-84db-49ef-9304-1467a0ef65dc.png)


2. Setelah proxy terpasang, coba akses http://its.ac.id dengan perintah `lynx http://its.ac.id`.

    ![image](https://user-images.githubusercontent.com/70105993/141479916-2172f938-a78e-47b3-87b2-578d1bceaf08.png)

## Soal 9
> Agar transaksi jual beli lebih aman dan pengguna website ada dua orang, proxy dipasang autentikasi user proxy dengan enkripsi MD5 dengan dua username, yaitu luffybelikapalyyy dengan password luffy_yyy dan zorobelikapalyyy dengan password zoro_yyy

### Jawaban:
**Water7**
1. Buat file **/etc/squid/passwd** untuk menyimpan username dan password yang akan digunakan dengan perintah `touch /etc/squid/passwd`.
2. Jalankan perintah berikut untuk menginstall package yang dibutuhkan.
    ```bash
    apt-get update
    apt-get install apache2-utils -y
    ```
    
3. Tambahkan informasi username dan password pada file **/etc/squid/passwd** sesuai format yang diminta pada soal dengan perintah berikut.
    ```bash
    htpasswd -m /etc/squid/passwd luffybelikapald10
    luffy_d10
    luffy_d10

    htpasswd -m /etc/squid/passwd zorobelikapald10
    zoro_d10
    zoro_d10
    ```

4. Ubah isi file `/etc/squid/squid.conf` seperti berikut untuk mengaktifkan user login menggunakan file **passwd** yang sudah dibuat sebelumnya.

    ```
    http_port 5000
    visible_hostname jualbelikapal.d10.com
    
    auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/passwd
    auth_param basic children 5
    auth_param basic realm Proxy
    auth_param basic credentialsttl 2 hours
    auth_param basic casesensitive on
    acl USERS proxy_auth REQUIRED
    http_access allow USERS
    ```
    
5. Restart squid dengan perintah `service squid restart`.

**Loguetown**

Coba akses kembali http://its.ac.id dengan perintah `lynx http://its.ac.id`, maka akan muncul perintah untuk memasukkan username dan password proxy.

![image](https://user-images.githubusercontent.com/70105993/141483788-9e32311d-adab-4ef8-824b-523fdd4985fc.png)

## Soal 10
> Transaksi jual beli tidak dilakukan setiap hari, oleh karena itu akses internet dibatasi hanya dapat diakses setiap hari Senin-Kamis pukul 07.00-11.00 dan setiap hari Selasa-Jum’at pukul 17.00-03.00 keesokan harinya (sampai Sabtu pukul 03.00)

### Jawaban:
**Water7**
1. Buat file **/etc/squid/acl.conf** untuk menyimpan konfigurasi ACL berdasarkan waktu akses yang akan digunakan dengan perintah `touch /etc/squid/acl.conf`.
2. Tambahkan informasi waktu akses pada file **/etc/squid/acl.conf** sesuai yang diminta pada soal seperti berikut.
    ```bash
    acl AVAILABLE1 time MTWHF 07:00-11:00
    acl AVAILABLE2 time TWHF 17:00-24:00
    acl AVAILABLE3 time WHFA 00:00-03:00
    ```

    ![Screenshot_46](https://user-images.githubusercontent.com/70105993/141484431-17a16e2d-3801-4d1e-9a20-127d67b1848b.png)


3. Ubah isi file `/etc/squid/squid.conf` seperti berikut untuk memperbolehkan akses secara terautentikasi dengan waktu akses sesuai yang sudah didefinisikan pada file **acl.conf**, dan menolak akses di luar itu.

    ```
    include /etc/squid/acl.conf

    http_port 5000
    visible_hostname jualbelikapal.d10.com

    auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/passwd
    auth_param basic children 5
    auth_param basic realm Proxy
    auth_param basic credentialsttl 2 hours
    auth_param basic casesensitive on
    acl USERS proxy_auth REQUIRED

    http_access allow AVAILABLE1 USERS
    http_access allow AVAILABLE2 USERS
    http_access allow AVAILABLE3 USERS

    http_access deny all
    ```
    
    ![Screenshot_47](https://user-images.githubusercontent.com/70105993/141484608-6caf5afa-8b62-42e8-816f-de059570d695.png)
    
4. Restart squid dengan perintah `service squid restart`.

**Loguetown**

Pengujian pembatasan waktu akses bisa dilakukan dengan mengganti tanggal sistem dengan perintah:
- `date -s "12 NOV 2021 20:00:00"` (di dalam waktu akses yang diperbolehkan)
- `date -s "12 NOV 2021 14:00:00"` (di luar waktu akses yang diperbolehkan)

kemudian akses kembali http://its.ac.id dengan perintah `lynx http://its.ac.id`.

- Jika akses dilakukan di dalam waktu yang diperbolehkan, maka akan muncul layar login, dan jika berhasil terautentikasi maka website akan ditampilkan.

    ![image](https://user-images.githubusercontent.com/70105993/141479916-2172f938-a78e-47b3-87b2-578d1bceaf08.png)

- Jika akses dilakukan di luar waktu yang diperbolehkan, akan muncul alert 403 Forbidden, dan pesan error seperti berikut.

    ![image](https://user-images.githubusercontent.com/70105993/141485902-740111d8-8c83-4b7f-8c4e-04963f120dec.png)

## Soal 11
> Agar transaksi bisa lebih fokus berjalan, maka dilakukan redirect website agar mudah mengingat website transaksi jual beli kapal. Setiap mengakses google.com, akan diredirect menuju super.franky.yyy.com dengan website yang sama pada soal shift modul 2. Web server super.franky.yyy.com berada pada node Skypie

### Jawaban:

## Soal 12
> Saatnya berlayar! Luffy dan Zoro akhirnya memutuskan untuk berlayar untuk mencari harta karun di super.franky.yyy.com. Tugas pencarian dibagi menjadi dua misi, Luffy bertugas untuk mendapatkan gambar (.png, .jpg), sedangkan Zoro mendapatkan sisanya. Karena Luffy orangnya sangat teliti untuk mencari harta karun, ketika ia berhasil mendapatkan gambar, ia mendapatkan gambar dan melihatnya dengan kecepatan 10 kbps

### Jawaban:

## Soal 13
> Sedangkan, Zoro yang sangat bersemangat untuk mencari harta karun, sehingga kecepatan kapal Zoro tidak dibatasi ketika sudah mendapatkan harta yang diinginkannya

### Jawaban:

