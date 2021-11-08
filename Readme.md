# Praktikum Modul 3

- Andika Nugrahanto (0511194000031)
- Muhammad Rayhan Raffi Pratama (05111940000110)
- Fadhil Dimas Sucahyo (05111940000212)

## Pendahuluan

### Setting Topologi

![0.1](imgs/0.1.JPG)

### Edit Knofigurasi Network

#### Foosha

![0.2](imgs/0.2.JPG)
![0.3](imgs/0.3.JPG)

#### EniesLobby

![0.4](imgs/0.4.JPG)

#### Water7

![0.5](imgs/0.5.JPG)

#### Jipangu

![0.6](imgs/0.6.JPG)

#### Loguetown, Alabasta, Totoland, Skypie

![0.7](imgs/0.7.JPG)

## no. 1

Luffy bersama Zoro berencana membuat peta tersebut dengan kriteria EniesLobby sebagai DNS Server, Jipangu sebagai DHCP Server, Water7 sebagai Proxy Server

### Jawab

#### Foosha

Menjalankan command `iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 192.194.0.0/16` yang digunakan supaya dapat terhubung ke jaringan luar pada router `Foosha`
![1.1](imgs/1.1.JPG)

Setelah itu pada EniesLobby, Water7, Jipangu dijalankan command `echo "nameserver 192.168.122.1" > /etc/resolv.conf` untuk setting IP DNS agar dapat terhubung ke jaringan luar.

#### EniesLobby

pada EniesLobby jalankan command `apt-get update` dan `apt-get install bind9 -y` untuk menginstall bind9
![1.2](imgs/1.2.JPG)

#### Jipangu

pada Jipangu jalankan command `apt-get update` dan `apt-get install isc-dhcp-server -y` untuk menginstall isc-dhcp-server
![1.3](imgs/1.3.JPG)

Kemudian setting `INTERFACES` yang digunakan oleh Jipangu pada file `/etc/default/isc-dhcp-server` dengan menambahkan `eth0`
![1.5](imgs/1.5.JPG)

#### Water7

pada Water7 jalankan command `apt-get update` dan `apt-get install squid -y` untuk menginstall squid
![1.4](imgs/1.4.JPG)

## no. 2

Foosha sebagai DHCP Relay

### Jawab

#### Foosha

pada Foosha jalankan command `apt-get update` dan `apt-get install isc-dhcp-relay -y` untuk menginstall isc-dhcp-relay
![2.1](imgs/2.1.JPG)

Kemudian edit file `/etc/default/isc-dhcp-relay` dengan menambahkan `SERVER = "IP Jipangu"` dan `INTERFACES = "eth1 eth2 eth3"`
![2.2](imgs/2.2.JPG)

Lalu jalankan command `service isc-dhcp-relay restart`

## no. 3

Client yang melalui Switch1 mendapatkan range IP dari [prefix IP].1.20 - [prefix IP].1.99 dan [prefix IP].1.150 - [prefix IP].1.169

### Jawab

#### Jipangu

Edit file `/etc/dhcp/dhcpd.conf` dengan menambahkan:

```bash
    subnet 192.194.1.0 netmask 255.255.255.0 {
        range 192.194.1.20 192.194.1.99;
        range 192.194.1.150 192.194.1.169;
        option routers 192.194.1.1;
        option broadcast-address 192.194.1.255;
        option domain-name-servers 192.194.2.2;
        default-lease-time 360;
        max-lease-time 7200;
    }
```

![3.1](imgs/3.1.JPG)

Lalu jalankan command `service isc-dhcp-server restart`

## no. 4

Client yang melalui Switch3 mendapatkan range IP dari [prefix IP].3.30 - [prefix IP].3.50

### Jawab

#### Jipangu

Edit file `/etc/dhcp/dhcpd.conf` dengan menambahkan:

```bash
    subnet 192.194.3.0 netmask 255.255.255.0 {
        range 192.194.3.30 192.194.3.50;
        option routers 192.194.3.1;
        option broadcast-address 192.194.3.255;
        option domain-name-servers 192.194.2.2;
        default-lease-time 720;
        max-lease-time 7200;
    }
```

![4.1](imgs/4.1.JPG)

Lalu jalankan command `service isc-dhcp-server restart`

## no. 5

Client mendapatkan DNS dari EniesLobby dan client dapat terhubung dengan internet melalui DNS tersebut.

### Jawab

#### EniesLobby

Edit file `/etc/bind/named.conf.options` dengan menambahkan

```bash
    forwarders {
        "IP nameserver dari Foosha";
    };

    allow-query{any;};
```

dan mengkomen bagian

```bash
    // dnssec-validation auto;
```

![5.2](imgs/5.2.JPG)

kemudian jalankan command `service bind9 restart`

#### Jipangu

Edit file `/etc/dhcp/dhcpd.conf` dengan menambahkan baris `option domain-name-servers "IP EniesLobby"` pada `subnet 192.194.1.0` dan `subnet 192.194.3.0`

![5.1](imgs/5.1.JPG)

## no. 6

Lama waktu DHCP server meminjamkan alamat IP kepada Client yang melalui Switch1 selama 6 menit sedangkan pada client yang melalui Switch3 selama 12 menit. Dengan waktu maksimal yang dialokasikan untuk peminjaman alamat IP selama 120 menit.

### Jawab

#### Jipangu

Edit file `/etc/dhcp/dhcpd.conf` dengan menambahkan baris ini pada `subnet 192.194.1.0`

```bash
        default-lease-time 360;
        max-lease-time 7200;
```

![6.1](imgs/6.1.JPG)

dan menambahkan baris ini pada `subnet 192.194.3.0`

```bash
        default-lease-time 720;
        max-lease-time 7200;
```

![6.2](imgs/6.2.JPG)

## no. 7

Luffy dan Zoro berencana menjadikan Skypie sebagai server untuk jual beli kapal yang dimilikinya dengan alamat IP yang tetap dengan IP [prefix IP].3.69

### Jawab

#### Jipangu

Edit file `/etc/dhcp/dhcpd.conf` dengan menambahkan baris ini

```bash
    host Skypie {
        hardware ethernet "hardware address Skypie";
        fixed-address 192.194.3.69;
    }
```

![7.1](imgs/7.1.JPG)

Lalu jalankan command `service isc-dhcp-server restart`

#### Skypie

Kemudian tambahkan `hwaddress ether "hardware address Skypie"` pada `/etc/network/interfaces` agar hwaddress tidak berubah-ubah ketika project direstart atau diexport

![7.2](imgs/7.2.JPG)

## no. 8

Loguetown digunakan sebagai client Proxy agar transaksi jual beli dapat terjamin keamanannya, juga untuk mencegah kebocoran data transaksi.
Pada Loguetown, proxy harus bisa diakses dengan nama jualbelikapal.yyy.com dengan port yang digunakan adalah 5000

### Jawab

#### Water7

Edit file `/etc/squid/squid.conf` dengan menambahkan

```bash
    http_port 5000
    visible_hostname jualbelikapal.d05.com
    http_access allow all
```

![8.1](imgs/8.1.JPG)

kemudian jalankan command `service squid restart`

#### Skypie

Jalankan `apt-get update` dan `apt-get install lynx`

Aktifkan proxy dengan menjalankan command `export http_proxy="http://192.194.2.3:5000"`. Kemudian jalankan command `env | grep -i proxy` untuk mengecek apakah proxy sudah tersetting.

![8.2](imgs/8.2.JPG)

## no. 9

Agar transaksi jual beli lebih aman dan pengguna website ada dua orang, proxy dipasang autentikasi user proxy dengan enkripsi MD5 dengan dua username, yaitu luffybelikapalyyy dengan password luffy_yyy dan zorobelikapalyyy dengan password zoro_yyy

### Jawab

#### Water7

Jalankan `apt-get update` dan `apt-get install apache2-utils`

Kemudian jalankan command `htpasswd -cm /etc/squid/passwd luffybelikapald05`, option `c` digunakan untuk membuat file baru, sedangkan `m` digunakan supaya enkripsinya menggunakan MD5. Setelah itu masukkan password `luffy_d05`.

Kemudian jalankan command `htpasswd -m /etc/squid/passwd zorobelikapald05`. Setelah itu masukkan password `zoro_d05`.
![9.1](imgs/9.1.JPG)

Setelah itu edit file `/etc/squid/squid.conf` menjadi

```bash
    http_port 5000
    visible_hostname jualbelikapal.d05.com
    auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/passwd
    auth_param basic children 5
    auth_param basic realm Login
    auth_param basic credentialsttl 2 hours
    auth_param basic casesensitive on
    acl USERS proxy_auth REQUIRED
    http_access allow USERS
```

![9.2](imgs/9.2.JPG)

kemudian jalankan command `service squid restart`

## no. 10

Transaksi jual beli tidak dilakukan setiap hari, oleh karena itu akses internet dibatasi hanya dapat diakses setiap hari Senin-Kamis pukul 07.00-11.00 dan setiap hari Selasa-Jum’at pukul 17.00-03.00 keesokan harinya (sampai Sabtu pukul 03.00)

### Jawab

#### Water7

Buat file baru bernama `acl.conf` di folder squid dengan menjalankan command `nano /etc/squid/acl.conf` kemudian diisi dengan

```bash
    acl AVAILABLE_WORKING time MTWH 07:00-11:00
    acl AVAILABLE_WORKING time TWHF 17:00-23:59
    acl AVAILABLE_WORKING time WHFA 00:00-03:00
```

![10.1](imgs/10.1.JPG)

Setelah itu edit file `/etc/squid/squid.conf` menjadi

```bash
    include /etc/squid/acl.conf

    http_port 5000
    visible_hostname jualbelikapal.d05.com
    auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/passwd
    auth_param basic children 5
    auth_param basic realm Login
    auth_param basic credentialsttl 2 hours
    auth_param basic casesensitive on
    acl USERS proxy_auth REQUIRED
    http_access allow USERS AVAILABLE_WORKING
    http_access deny all
```

![10.2](imgs/10.2.JPG)

kemudian jalankan command `service squid restart`

## no. 11

Agar transaksi bisa lebih fokus berjalan, maka dilakukan redirect website agar mudah mengingat website transaksi jual beli kapal. Setiap mengakses google.com, akan diredirect menuju super.franky.yyy.com dengan website yang sama pada soal shift modul 2. Web server super.franky.yyy.com berada pada node Skypie

### Jawab

#### EniesLobby

Edit file `nano /etc/bind/named.conf.local` dengan mengisi

```bash
    zone "super.franky.d05.com" {
        type master;
        file "/etc/bind/sunnygo/super.franky.d05.com";
    };

    zone "3.194.192.in-addr.arpa" {
        type master;
        file "/etc/bind/sunnygo/3.194.192.in-addr.arpa";
    };
```

![11.1](imgs/11.1.JPG)

Kemudian buat folder sunnygo menggunakan command `mkdir /etc/bind/sunnygo`. Setelah itu jalankan command `cp /etc/bind/db.local /etc/bind/sunnygo/super.franky.d05.com`. Setelah itu edit file `nano /etc/bind/sunnygo/super.franky.d05.com` dengan mengisi

```bash
    $TTL    604800
    @       IN      SOA     super.franky.d05.com. root.super.franky.d05.com. (
                                2         ; Serial
                            604800         ; Refresh
                            86400         ; Retry
                            2419200         ; Expire
                            604800 )       ; Negative Cache TTL
    ;
    @       IN      NS      super.franky.d05.com.
    @       IN      A       192.194.3.69 ; IP Skypie
    www     IN      CNAME   super.franky.d05.com.
```

![11.2](imgs/11.2.JPG)

Setelah itu jalankan command `cp /etc/bind/db.local /etc/bind/sunnygo/3.194.192.in-addr.arpa`. Setelah itu edit file `nano /etc/bind/sunnygo/3.194.192.in-addr.arpa` dengan mengisi

```bash
    $TTL    604800
    @       IN      SOA     super.franky.d05.com. root.super.franky.d05.com. (
                                2         ; Serial
                            604800         ; Refresh
                            86400         ; Retry
                            2419200         ; Expire
                            604800 )       ; Negative Cache TTL
    ;
    3.194.192.in-addr.arpa.      IN      NS      super.franky.d05.com.
    69       IN      PTR       super.franky.d05.com.
```

![11.3](imgs/11.3.JPG)

kemudian jalankan command `service bind9 restart`

#### Skypie

Pertama, install `apache2`, `php`, `libapache2-mod-php7.0`, `wget`, dan `unzip`.

Kemudian jalankan command `wget https://raw.githubusercontent.com/FeinardSlim/Praktikum-Modul-2-Jarkom/main/super.franky.zip` kemudian `unzip super.franky.zip`

Setelah itu, pindah ke directory `/etc/apache2/sites-available`.Kemudian copy file `000-default.conf` menjadi file `super.franky.d05.com.conf`

![11.4](imgs/11.4.JPG)

Lalu setting file `super.franky.d05.com.conf` agar memiliki line `ServerName super.franky.d05.com`, `ServerAlias www.super.franky.d05.com`, dan `DocumentRoot /var/www/super.franky.d05.com`.

![11.5](imgs/11.5.JPG)

Kemudian bikin directory baru dengan nama `super.franky.d05.com` pada `/var/www/` menggunakan command `mkdir /var/www/super.franky.d05.com`. lalu copy isi dari folder `super.franky` yang telah didownload ke `/var/www/super.franky.d05.com`.

Setelah itu jalankan command `a2ensite super.franky.d05.com` dan `service apache2 restart`
![11.6](imgs/11.6.JPG)

#### Water7

Setelah itu edit file `/etc/squid/squid.conf` menjadi

```bash
    include /etc/squid/acl.conf

    http_port 5000
    visible_hostname jualbelikapal.d05.com
    auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/passwd
    auth_param basic children 5
    auth_param basic realm Login
    auth_param basic credentialsttl 2 hours
    auth_param basic casesensitive on
    acl USERS proxy_auth REQUIRED
    acl google dstdomain www.google.com
    http_access allow USERS AVAILABLE_WORKING
    http_access deny all google
    deny_info http://super.franky.d05.com/ google
    http_access deny all
```

![11.7](imgs/11.7.JPG)

kemudian jalankan command `service squid restart`
