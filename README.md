# Laporan Resmi Soal Shift Modul 3 Jarkom 2021

## Anggota Kelompok D10
- Mohammad Faderik I H (05111940000023)
- Farhan Arifandi (05111940000061)
- Yusril Zubaydi (05111940000160)


## Prefix IP
Prefix IP untuk kelompok kami adalah `10.26`.

## Soal 1 dan 2
> ![image](https://user-images.githubusercontent.com/70105993/141472155-85410a83-129d-4309-8d61-8655331c9899.png)
> 
> Luffy bersama Zoro berencana membuat peta tersebut dengan kriteria EniesLobby sebagai DNS Server, Jipangu sebagai DHCP Server, Water7 sebagai Proxy Server dan Foosha sebagai DHCP Relay

### Jawaban:
1. Membuat topologi seperti gambar diatas
2. Mengatur `Network Configuration` pada setiap node yang akan digunakan
    * Foosha
        ```
        auto eth0
        iface eth0 inet dhcp

        auto eth1
        iface eth1 inet static
            address 10.26.1.1
            netmask 255.255.255.0

        auto eth2
        iface eth2 inet static
            address 10.26.2.1
            netmask 255.255.255.0

        auto eth3
        iface eth3 inet static
            address 10.26.3.1
            netmask 255.255.255.0
        ```
    * EniesLobby
        ```
        auto eth0
        iface eth0 inet static
            address 10.26.2.2
            netmask 255.255.255.0
            gateway 10.26.2.1
        ```
    * Water7
        ```
        auto eth0
        iface eth0 inet static
            address 10.26.2.3
            netmask 255.255.255.0
            gateway 10.26.2.1
        ```
    * Jipangu
        ```
        auto eth0
        iface eth0 inet static
            address 10.26.2.4
            netmask 255.255.255.0
            gateway 10.26.2.1
        ```
3. Jalankan perintah di bawah ini pada node `Foosha`
    ```
    iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 10.26.0.0/16
    ```
4. Jalankan perintah ini pada node yang `Network Configuration` sudah di setting tadi, kecuali `Foosha`
    ```
    echo nameserver 192.168.122.1 > /etc/resolv.conf
    ```
5. Menjadikan `Jipanggu` sebagai `DHCP Server`
    1. Lakukan update pada dengan perintah :
        ```
        apt-get update
        ```
    2. Install isc-dhcp-server dengan perintah :
        ```
        apt-get install isc-dhcp-server -y
        ```
    3. Periksa apakah sudah terinstall atau belum dengan menggunakan perintah :
        ```
        dhcpd --version
        ```
        ![alt-text](https://media.discordapp.net/attachments/848199470025801749/908930638940340224/unknown.png)
    4. Edit file `/etc/default/isc-dhcp-server` dengan perintah :
        ```
        nano /etc/default/isc-dhcp-server
        ```
    5. Karena interface yang akan diberikan layanan adalah **eth0**, tambahkan line berikut sehingga menjadi seperti gambar
        ```
        INTERFACES = "eth0"
        ```
        ![alt-text](https://media.discordapp.net/attachments/848199470025801749/908933315065700422/unknown.png)
6. Menjadikan `Water7` sebagai `Proxy Server`
    1. Lakukan update pada dengan perintah :
        ```
        apt-get update
        ```
    2. Instal squid dengan perintah :
        ```
        apt-get install squid -y
        ```
    3. Backup file configurasi yang sudah disediakan squid dengan perintah :
        ```
        mv /etc/squid/squid.conf /etc/squid/squid.conf.bak
        ```
    4. Buat file configurasi baru dengan peritah :
        ```
        nano /etc/squid/squid.conf
        ```
    5. Tambahkan line berikut
        ```
        http_port 8080
        visible_hostname Water7
        ```
    6. Restart service squid dengan perintah :
        ```
        service squid restart
        ```
7. Menjadikan `EniesLobby` sebagai `DNS Server`
    1. Lakukan update pada dengan perintah :
        ```
        apt-get update
        ```
    2. Install bind9 dengan perintah :
        ```
        apt-get install bind9 -y
        ```
    3. Edit file `/etc/bind/named.conf.options` dengan perintah :
        ```
        nano /etc/bind/named.conf.options
        ```
    4. Lakukan uncomment pada baris berikut
        ```
        forwarders {
            192.168.122.1;
        };
        ```
    5. Lakukan comment pada baris berikut
        ```
        //dnssec-validation auto;
        ```
    6. Tambahkan
        ```
        allow-query{any;};
        ```
8. Menjadikan `Foosha` sebagai `DHCP Relay`
    1. Lakukan update pada dengan perintah :
        ```
        apt-get update
        ```
    2. Install isc-dhcp-relay dengan perintah :
        ```
        apt-get install isc-dhcp-relay
        ```
    3. Saat melakukan instalasi, akan diminta input SERVERS yang diisi dengan `IP Jipanggu`
        ```
        10.26.2.4
        ```
    4. Kemudian, akan diminta input untuk INTERFACES, masukan seperti di bawah
        ```
        eth1 eth2 eth3
        ```
    5. Settingan di atas juga bisa kita ubah pada file `/etc/default/isc-dhcp-relay` dengan perintah
        ```
        nano /etc/default/isc-dhcp-relay
        ```
    6. Restart service isc-dhcp-relay
        ```
        service isc-dhcp-relay restart
        ```
## Soal 3 - 6
> Ada beberapa kriteria yang ingin dibuat oleh Luffy dan Zoro, yaitu:
> 1. Semua client yang ada HARUS menggunakan konfigurasi IP dari DHCP Server.
> 2. Client yang melalui Switch1 mendapatkan range IP dari [prefix IP].1.20 - [prefix IP].1.99 dan [prefix IP].1.150 - [prefix IP].1.169
> 3. Client yang melalui Switch3 mendapatkan range IP dari [prefix IP].3.30 - [prefix IP].3.50
> 4. Client mendapatkan DNS dari EniesLobby dan client dapat terhubung dengan internet melalui DNS tersebut.
> 5. Lama waktu DHCP server meminjamkan alamat IP kepada Client yang melalui Switch1 selama 6 menit sedangkan pada client yang melalui Switch3 selama 12 menit. Dengan waktu maksimal yang dialokasikan untuk peminjaman alamat IP selama 120 menit.


### Jawaban:
1. Edit file `/etc/dhcp/dhcpd.conf` pada Jipanggu dengan perintah :
    ```
    nano /etc/dhcp/dhcpd.conf
    ```
2. Tambahkan script berikut
    ```
    subnet 10.26.2.0 netmask 255.255.255.0 {
        option routers 10.26.2.1;
    }

    subnet 10.26.1.0 netmask 255.255.255.0 {
        range  10.26.1.20 10.26.1.99;           //Untuk No 3 (switch 1)
        range  10.26.1.150 10.26.1.169;         //Untuk No 3 (switch 1)
        option routers 10.26.1.1;
        option broadcast-address 10.26.1.255;
        option domain-name-servers 10.26.2.2;
        default-lease-time 360;                 //Untuk No 6 (switch 1)
        max-lease-time 7200;                    //Untuk No 6 (switch 1)
    }

    subnet 10.26.3.0 netmask 255.255.255.0 {
        range 10.26.3.30 10.26.3.50;            //Untuk No 4 
        option routers 10.26.3.1;
        option broadcast-address 10.26.3.255;
        option domain-name-servers 10.26.2.2;
        default-lease-time 720;                 //Untuk No 6 (switch 2)
        max-lease-time 7200;                    //Untuk No 6 (switch 2)
    }
    ```
3. Restart service DHCP Server dengan perintah :
    ```
    service isc-dhcp-server restart
    ```
4. Edit `Network Configuration` untuk setiap node cient (Untuk no 5) menjadi
    ```
    auto eth0
    iface eth0 inet dhcp
    ```
5. Kemudian restart setiap node client yang disetting tadi
6. Periksa IP setiap node dengan perintah
    ```
    ifconfig eth0
    ```
    * Longuetown
    ![alt-text](https://media.discordapp.net/attachments/848199470025801749/908944853386293278/unknown.png)
    * Alabasta
    ![alt-text](https://media.discordapp.net/attachments/848199470025801749/908944990913327104/unknown.png)
    * TottoLand
    ![alt-text](https://media.discordapp.net/attachments/848199470025801749/908945369591849010/unknown.png)
    * Skypie
    ![alt-text](https://media.discordapp.net/attachments/848199470025801749/908945489582522399/unknown.png)
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
**EniesLobby**
1. Tambahkan konfigurasi DNS untuk `super.franky.d10.com` pada file `/etc/bind/named.conf.local` seperti berikut.
    ```vim
    zone "super.franky.d10.com" {
        type master;
        file "/etc/bind/kaizoku/super.franky.d10.com";
        allow-transfer { 10.26.3.69; };
    };
    ```
2. Buat folder **/etc/bind/kaizoku** dengan perintah `mkdir /etc/bind/kaizoku`.
3. Salin file **db.local** sebagai template file konfigurasi ke folder kaizoku dengan perintah `cp /etc/bind/db.local /etc/bind/kaizoku/super.franky.d10.com`.
4. Ubah isi file `/etc/bind/kaizoku/super.franky.d10.com` menjadi seperti berikut.
    ```vim
    ;
    ; BIND data file for local loopback interface
    ;
    $TTL    604800
    @       IN      SOA     super.franky.d10.com. root.super.franky.d10.com. (
                         2021100401         ; Serial
                             604800         ; Refresh
                              86400         ; Retry
                            2419200         ; Expire
                             604800 )       ; Negative Cache TTL
    ;
    @       IN      NS      super.franky.d10.com.
    @       IN      A       10.26.3.69
    www     IN      CNAME   super.franky.d10.com.
    ```
    
5. Restart bind9 dengan perintah `service bind9 restart`.

**Skypie**
1. Tambahkan baris berikut di akhir file `/etc/apache2/apache2.conf`.
    ```vim
    ServerName 10.26.3.69
    ```
    
2. Buat folder **/var/www/super.franky.d10.com** dengan perintah `mkdir /var/www/super.franky.d10.com`.
3. Install **wget** dan **unzip** dengan perintah:
    ```bash
    apt-get install wget -y
    apt-get install unzip -y
    ```
    
4. Download dan unzip file zip yang diperlukan untuk web dengan perintah:
    ```bash
    wget https://raw.githubusercontent.com/FeinardSlim/Praktikum-Modul-2-Jarkom/main/super.franky.zip
    unzip -j super.franky.zip -d /var/www/super.franky.d10.com
    ```

5. Buat file baru pada direktori **/etc/apache2/sites-available** dengan nama **super.franky.d10.com.conf** dan isi sebagai berikut
    ```vim
    root@Skypie:~# cat /root/no11/super.franky.d10.com.conf
    <VirtualHost *:80 *:5000>
            ServerName super.franky.d10.com
            ServerAlias www.super.franky.d10.com

            ServerAdmin webmaster@localhost
            DocumentRoot /var/www/super.franky.d10.com

            <Directory /var/www/super.franky.d10.com>
                    Options +Indexes
            </Directory>

            <Directory /var/www/super.franky.d10.com/public>
                    Options +Indexes
            </Directory>

            Alias "/js" "/var/www/super.franky.d10.com/public/js"

            ErrorLog ${APACHE_LOG_DIR}/error.log
            CustomLog ${APACHE_LOG_DIR}/access.log combined

            ErrorDocument 404 /error/404.html
    </VirtualHost>
    ```
    
6. Jalankan konfigurasi webserver yang telah dibuat dengan perintah `a2ensite super.franky.d10.com`.
7. Restart service Apache dengan perintah `service apache2 restart`.

**Water7**
1. Pada file **/etc/resolv.conf**, tambahkan nameserver baru yang mengarah ke IP EniesLobby dan comment nameserver lama seperti berikut.

    ```vim
    # nameserver 192.168.122.1
    nameserver 10.26.2.2
    ```

2. Ubah isi file `/etc/squid/squid.conf` seperti berikut untuk mengaktifkan redirection.

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

    acl BLACKLIST dstdomain google.com
    deny_info http://super.franky.d10.com/ BLACKLIST
    http_reply_access deny BLACKLIST
    ```

3. Restart squid dengan perintah `service squid restart`.

**Loguetown**

Coba akses google.com dengan perintah `lynx google.com` pada waktu akses yang diperbolehkan, maka akan muncul perintah untuk memasukkan username dan password proxy. Jika berhasil terautentikasi maka akan diarahkan menuju super.franky.d10.com.

![recording](https://user-images.githubusercontent.com/70105993/141492389-2b602eaa-1075-463c-9113-6415537adc21.gif)

## Soal 12
> Saatnya berlayar! Luffy dan Zoro akhirnya memutuskan untuk berlayar untuk mencari harta karun di super.franky.yyy.com. Tugas pencarian dibagi menjadi dua misi, Luffy bertugas untuk mendapatkan gambar (.png, .jpg), sedangkan Zoro mendapatkan sisanya. Karena Luffy orangnya sangat teliti untuk mencari harta karun, ketika ia berhasil mendapatkan gambar, ia mendapatkan gambar dan melihatnya dengan kecepatan 10 kbps

### Jawaban:
**Water7**
1. Buat file **/etc/squid/acl-bandwidth.conf** untuk menyimpan konfigurasi bandwidth dengan perintah `touch /etc/squid/acl-bandwidth.conf`.
2. Tambahkan informasi pembatasan bandwidth pada file **/etc/squid/acl-bandwidth.conf** untuk membatasi bandwidth sebesar 1250 Bps (10 kbps) pada url dengan path yang mengandung '.jpg' atau '.png' dan diakses dengan akun luffy seperti berikut.

    ```
    acl images_ext urlpath_regex -i \.jpg$ \.png$

    auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/passwd
    acl luffy proxy_auth luffybelikapald10

    delay_pools 2
    delay_class 1 1
    delay_parameters 1 1250/1250
    delay_access 1 allow luffy images_ext
    delay_access 1 deny all
    ```

3. Tambahkan ACL dari file **acl-bandwidth.conf** pada file `/etc/squid/squid.conf` seperti berikut.

    ```
    include /etc/squid/acl.conf
    include /etc/squid/acl-bandwidth.conf

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

    acl BLACKLIST dstdomain google.com
    deny_info http://super.franky.d10.com/ BLACKLIST
    http_reply_access deny BLACKLIST
    ```
    
4. Restart squid dengan perintah `service squid restart`.

**Loguetown**

Coba akses super.franky.d10.com/public/images dengan perintah `lynx super.franky.d10.com/public/images` menggunakan akun luffy kemudian download salah satu file dengan ekstensi .jpg atau .png, maka akan didapatkan kecepatan download sekitar 1.3 kBps (1.25 kBps dibulatkan ke atas).

![Screenshot_57](https://user-images.githubusercontent.com/70105993/141494457-edd61ba8-ad83-431f-af04-90fd5163ff9d.png)

## Soal 13
> Sedangkan, Zoro yang sangat bersemangat untuk mencari harta karun, sehingga kecepatan kapal Zoro tidak dibatasi ketika sudah mendapatkan harta yang diinginkannya

### Jawaban:
**Water7**
1. Ubah file **/etc/squid/acl-bandwidth.conf** untuk menambahkan konfigurasi untuk tidak membatasi bandwidth situs ketika diakses dengan akun zoro menjadi seperti berikut.

    ```
    acl images_ext urlpath_regex -i \.jpg$ \.png$

    auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/passwd
    acl luffy proxy_auth luffybelikapald10
    acl zoro proxy_auth zorobelikapald10

    delay_pools 2
    delay_class 1 1
    delay_parameters 1 1250/1250
    delay_access 1 allow luffy images_ext
    delay_access 1 deny all

    delay_class 2 1
    delay_parameters 2 -1/-1
    delay_access 2 allow zoro
    delay_access 2 deny luffy
    delay_access 2 deny all
    ```
    
2. Restart squid dengan perintah `service squid restart`.

**Loguetown**

Coba akses super.franky.d10.com/public/images dengan perintah `lynx super.franky.d10.com/public/images` menggunakan akun zoro kemudian download salah satu file, maka akan didapatkan kecepatan download yang sangat cepat.

![recording(2)](https://user-images.githubusercontent.com/70105993/141495875-6362f648-e28c-425f-b1dd-d21cc3db4b94.gif)


## Kendala selama pengerjaan
- Start node dalam project membutuhkan urutan tertentu, dan kadang terkendala jaringan saat instalasi package
- Client sempat tidak bisa mendapatkan IP dari DHCP server
- super.franky.d10.com sempat memberikan error 'Destination Host Unreachable' ketika di-ping ataupun diakses melalui lynx
- Restart squid sempat berkali-kali mengalami kegagalan karena proses dihentikan oleh sistem secara otomatis
