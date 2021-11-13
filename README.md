# Laporan Praktikum Modul 3

## Anggota  Kelompok D15 :

## Richard Asmarakandi (05111940000017)

## Nurul Izzatil Ulum (05111940000058)



Pertama untuk menyambungkan node2 lainnya ke internet, isikan `iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 192.192.0.0/16` pada roter "Foosha"
Isikan `echo nameserver 192.168.122.1 > /etc/resolv.conf` pada server2 lainnya agar nameservernya terganti dan bisa menerima internet. dan atur konfigurasi nodenya sesuai dengan modul sebelumnya.


## SOAL

### 1. Luffy bersama Zoro berencana membuat peta tersebut dengan kriteria EniesLobby sebagai DNS Server, Jipangu sebagai DHCP Server, Water7 sebagai Proxy Server.

* EniesLobby sebagai DNS Server
Instalasi pada EniesLobby sebagai berikut
```
apt-get update
apt-get install bind9
```
* Jipangu sebagai DHCP Server
Instalasi pada Jipangu sebagai berikut
```
apt-get update
apt-get install isc-dhcp-server
```
Ganti `INTERFACES=` paad file `/etc/default/isc-dhcp-server.` dengan `INTERFACES="eth0"`. maksud dari hal ini adalah untuk menyambungkan klien DHCp yang aman adalah Switch 1 dan 2 melalui eth0 milik Jipangu.
(gambar)

* Water7 sebagai Proxy Server
Instalasi pada Water7 sebagai berikut
```
apt-get update
apt-get install squid
```
Lakukan konfigurasi squid dengan membackup file konfigurasi default squid `mv /etc/squid/squid.conf /etc/squid/squid.conf.bak`
Buat konfigurasi squid pada file `/etc/squid/squid.conf` dengan menambahkan
```
http_port 8080
visible_hostname Water7
```
(gambar)
Restart dengan `service squid restart` dan cek status squid dengan `service squid status`


### 2. Foosha sebagai DHCP Relay.
Instalasi pada Foosha sebagai berikut
```
apt-get update
apt-get install isc-dhcp-relay
```
* Jika muncul seperti gambar, maka masukkan hal2 yang diperlukan yaitu server milik DHCP server (Jipangu) dan Interfaces milih DHCP Server dan Client
```
SERVERS="192.199.2.4"
INTERFACES="eth1 eth3 eth2"
```
(gambar)
* Cara lainnya, Konfigurasi pada file `/etc/default/isc-dhcp-relay` dan isi `SERVER=` dengan IP DHCP server dan isi `INTERFACES=` dengan interface milik DHCP server dan DHCP client.
(Gambar)

Restart dengan perintah `service isc-dhcp-relay restart`
Lakukan konfigurasi pada file `/etc/sysctl.conf` untuk membuka IP Forwarding dengan uncommet pada `net.ipv4.ip_forward=1`
(gambar)

### Jawaban 3,4,5,6
* 3. Semua client yang ada HARUS menggunakan konfigurasi IP dari DHCP Server. Client yang melalui Switch1 mendapatkan range IP dari [prefix IP].1.20 - [prefix IP].1.99 dan [prefix IP].1.150 - [prefix IP].1.169.
* 4. Client yang melalui Switch3 mendapatkan range IP dari [prefix IP].3.30 - [prefix IP].3.50.
* 5. Client mendapatkan DNS dari EniesLobby dan client dapat terhubung dengan internet melalui DNS tersebut.
* 6. Lama waktu DHCP server meminjamkan alamat IP kepada Client yang melalui Switch1 selama 6 menit sedangkan pada client yang melalui Switch3 selama 12 menit. Dengan waktu maksimal yang dialokasikan untuk peminjaman alamat IP selama 120 menit.

#### DHCP Server (Jipangu)
lakukan konfigurasi di file `/etc/dhcp/dhcpd.conf`
* Switch 2
```
subnet 192.199.2.0 netmask 255.255.255.0 {
}
```
* Switch 1
```
subnet 192.199.1.0 netmask 255.255.255.0 {
  range 192.199.1.20 192.192.1.99;
  range 192.199.1.150 192.199.1.169; 
  option routers 192.199.1.1; 
  option broadcast-address 192.199.1.255;
  option domain-name-servers 192.199.2.2; 
  default-lease-time 360; 
  max-lease-time 7200; 
}
```
* Switch 3
```
subnet 192.199.3.0 netmask 255.255.255.0 {
  range 192.199.3.30 192.199.3.50; 
  option routers 192.199.3.1; 
  option broadcast-address 192.199.3.255;
  option domain-name-servers 192.199.2.2; 
  default-lease-time 720; 
  max-lease-time 7200; 
}
```
(gambar)

Restart dengan perintah `service isc-dhcp-server restart`
Kemudian cek dengan perintah `service isc-dhcp-server status`
(gambar)

#### DHCP Client (LogueTown, Alabasta, TottoLand, Skypie)
Komentar konfigurasi lama (IP Statis) pada file /etc/network/interfaces dan tambahkan:
```
auto eth0
iface eth0 inet dhcp
```
Restart dan cek dengan `ip a`
(gambar)
periksa apakah sudah sesuai dengan ip (EniesLobby)

```
### Lakukan pada semua interface DHCP Client.
```

#### DNS Forwader
lakukan konfigurasi pada DNS Server (EniesLobby) `/etc/bind/named.conf.options`tambahkan line seperti dibawah dan beri komentar pada `// dnssec-validation auto;` dan tambahkan `allow-query{any;};`
```
forwarders {
    192.168.122.1; // IP namserver Foosha
};
```
(gambar)
Periksa apakah klien tersambung internet atau tidak.

### 7. Luffy dan Zoro berencana menjadikan Skypie sebagai server untuk jual beli kapal yang dimilikinya dengan alamat IP yang tetap dengan IP [prefix IP].3.69.
Tambahkan pada konfigurasi di DHCP Server (Jipangu) pada file `/etc/dhcp/dhcpd.conf`
```
host Skypie {
    hardware ethernet b6:e7:28:cb:47:a6;
    fixed-address 192.192.3.69;
}
```
(gambar)

Hardware address di Skypie dapat dilihat dengan `ip a`
(gambar)

Restart DHCP Server (Jipangu) dengan perintah `service isc-dhcp-server restart`

Pada DHCP Client Skypie, tambahkan konfigurasi pada file `/etc/network/interfaces`
```
hwaddress ether b6:e7:28:cb:47:a6
```
(gambar)

Restart node Skypie dan cek menggunakan `ip a` dan cek apakah sudah menggunakan IP Address yang dikonfigurasikan sebagai fixed address.
(gambar)

### 8. Loguetown digunakan sebagai client Proxy agar transaksi jual beli dapat terjamin keamanannya, juga untuk mencegah kebocoran data transaksi. Pada Loguetown, proxy harus bisa diakses dengan nama jualbelikapal.yyy.com dengan port yang digunakan adalah 5000.
* Pada node Water7, ubah config squid pada `/etc/squid/squid.conf` menjadi seperti berikut :
![8-A](https://cdn.discordapp.com/attachments/804405775988555776/908619216771497984/unknown.png)
* Setelah itu, lakukan restart pada squid menggunakan command `service squid restart`
* Agar ip Water7 dapat diakses melalui `jualbelikapal.d15.com`, maka kita perlu mengatur dns. Pada modul kali ini, dns server diinstall pada node EniesLobby, sehingga kita ubah config `etc/bind/named.conf.local` pada EniesLobby dan diubah menjadi seperti ini :
![8-B](https://cdn.discordapp.com/attachments/804405775988555776/908616272546242620/unknown.png)
* Setelah itu kita ubah config pada `etc/bind/jualbelikapal/jualbelikapal.d15.com` (sesuai dengan direktori yang telah ditulis pada zone "jualbelikapal.d15.com") menjadi seperti berikut :
![8-C](https://cdn.discordapp.com/attachments/804405775988555776/908616748931113000/unknown.png)
* Lalu kita restart bind9 dengan command `service bind9 restart`.
* Pada node Loguetown(atau client lainnya), kita coba tes konfigurasi. Pertama-tama ketikkan `export http_proxy="http://jualbelikapal.d15.com:5000"
![8-D](https://cdn.discordapp.com/attachments/804405775988555776/908618408013205524/unknown.png)
* Setelah itu kita coba mengakses jualbelikapal.d15.com. Jika berhasil maka akan tampak seperti ini.
![8-E](https://cdn.discordapp.com/attachments/804405775988555776/908618930912895026/unknown.png)
* Akses masih forbidden karena kita belum memasukkan `http_access allow all` pada config squid, sehingga tambahkan pada `/etc/squid/squid.conf`
![8-F](https://cdn.discordapp.com/attachments/804405775988555776/908614278465069126/unknown.png)

### 9. Agar transaksi jual beli lebih aman dan pengguna website ada dua orang, proxy dipasang autentikasi user proxy dengan enkripsi MD5 dengan dua username, yaitu luffybelikapalyyy dengan password luffy_yyy dan zorobelikapalyyy dengan password zoro_yyy.
* Kita generate password menggunakan `htpasswd` pada Water7, selaku pemilik proxy server. Pertama-tama kita install apache2-utils dengan `apt-get install apache2-utils`. setelah itu kita gunakan command `htpasswd -b -c -m /etc/squid/passwd luffybelikapald15 luffy_d15` untuk menambahkan user baru dengan nama luffybelikapald15 dan password luffyd15.
* Lalu untuk menambahkan user zoro, kita gunakan command `htpasswd -b -m /etc/squid/passwd zorobelikapald15 zoro_d15`. kita hilangkan `-c` karena berarti membuat file, sedangkan file telah dibuat saat kita menambahkan user untuk luffybelikapald15.
![9-A](https://cdn.discordapp.com/attachments/804405775988555776/908621368927588352/unknown.png)
* Maka setelah digenerate, file `/etc/squid/passwd` akan terlihat seperti berikut :
![9-B](https://cdn.discordapp.com/attachments/804405775988555776/908987846294118400/unknown.png)
* Setelah itu pada `etc/squid/squid.conf` ubah confignya menjadi seperti berikut :
![9-C](https://cdn.discordapp.com/attachments/804405775988555776/908622077395877929/unknown.png)
* Lalu untuk mengeceknya kita gunakan command `lynx jualbelikapald15.com` pada node Loguetown(atau client lainnya). Hasilnya akan tampak seperti ini :
![9-D](https://cdn.discordapp.com/attachments/804405775988555776/908622511841886218/unknown.png)
![9-E](https://cdn.discordapp.com/attachments/804405775988555776/908622523858563082/unknown.png)


### 10. Transaksi jual beli tidak dilakukan setiap hari, oleh karena itu akses internet dibatasi hanya dapat diakses setiap hari Senin-Kamis pukul 07.00-11.00 dan setiap hari Selasa-Jum’at pukul 17.00-03.00 keesokan harinya (sampai Sabtu pukul 03.00).
* Untuk melakukannya, kita perlu membuat file config di node Water7 selaku proxy server secara manual dengan nama `acl.conf` pada `/etc/squid/acl.conf` dan filenya terlihat seperti ini
![10-A](https://cdn.discordapp.com/attachments/804405775988555776/908693652832919552/unknown.png)
* Setelah itu ubah config pada `/etc/squid/squid.conf` menjadi seperti ini
![10-B](https://cdn.discordapp.com/attachments/804405775988555776/908694000477806694/unknown.png)
* Restart squid dengan command `service squid restart`, lalu untuk mengeceknya gunakan command `lynx its.ac.id` pada client. Maka akan terlihat seperti ini apabila waktu di luar yang ditentukan :
![10-C](https://cdn.discordapp.com/attachments/804405775988555776/908701358042017812/unknown.png)
### 11. Agar transaksi bisa lebih fokus berjalan, maka dilakukan redirect website agar mudah mengingat website transaksi jual beli kapal. Setiap mengakses google.com, akan diredirect menuju super.franky.yyy.com dengan website yang sama pada soal shift modul 2. Web server super.franky.yyy.com berada pada node Skypie.
* Tambahkan konfigurasi pada `/etc/squid/squid.conf` di node Water7 selaku proxy server menjadi seperti berikut :
![11-A](https://cdn.discordapp.com/attachments/804405775988555776/908933233025122304/unknown.png)
* Untuk mengeceknya kita coba akses google pada node client menggunakan command `lynx google.com`. Hasilnya akan teredirect ke `super.franky.d15.com`.
![11-B](https://cdn.discordapp.com/attachments/804405775988555776/908934098066767882/unknown.png)
* Untuk mengatur domain `super.franky.d15.com` agar dapat dikenali sebagai IP Node Skypie, kita perlu mengatur pada DNS Server, yaitu node EniesLobby. Pada EniesLobby ubah configurasi `/etc/bind/named.conf.local` menjadi seperti berikut :
![11-C](https://cdn.discordapp.com/attachments/804405775988555776/908944308399374376/unknown.png)
* Setelah itu kita atur dns nya dengan membuat file baru pada `/etc/bind/jualbelikapal/super.franky.d15.com` dan mengubah isi konfigurasinya menjadi seperti berikut :
![11-D](https://cdn.discordapp.com/attachments/804405775988555776/908944714496086056/unknown.png)
* Setelah itu kita restart bind9 dengan command `service bind9 restart`.
* Lalu kita mengatur site untuk `super.franky.d15.com` yang terletak pada Node Skypie. Pada node skypie, kita lakukan `apt-get update`, `apt-get install apache2`, `apt-get install php` dan `apt-get install libapache2-mod-php7.0`. Kita copy file `000-default.conf` dengan nama `super.franky.d15.com.conf` menggunakan perintah `cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/super.franky.d15.com.conf`. Lalu kita edit isi dari `super.franky.d15.com.conf` menjadi seperti berikut :
![11-E](https://cdn.discordapp.com/attachments/804405775988555776/908973236157546536/unknown.png)
* Karena file dari super.franky telah disediakan, maka kita bisa langsung mendownloadnya dengan perintah `wget https://raw.githubusercontent.com/FeinardSlim/Praktikum-Modul-2-Jarkom/main/super.franky.zip` pada folder `/var/www/`. Setelah itu kita unzip dengan perintah `unzip super.franky.zip`. pada folder `/var/www/`. Maka ketika kita akses `lynx super.franky.d15.com` pada node client, akan ditampilkan isi folder dari super.franky yang telah kita ekstrak tadi
![11-F](https://cdn.discordapp.com/attachments/804405775988555776/908975807953780746/unknown.png)

### 12. Saatnya berlayar! Luffy dan Zoro akhirnya memutuskan untuk berlayar untuk mencari harta karun di super.franky.yyy.com. Tugas pencarian dibagi menjadi dua misi, Luffy bertugas untuk mendapatkan gambar (.png, .jpg), sedangkan Zoro mendapatkan sisanya. Karena Luffy orangnya sangat teliti untuk mencari harta karun, ketika ia berhasil mendapatkan gambar, ia mendapatkan gambar dan melihatnya dengan kecepatan 10 kbps
* Untuk melakukannya, kita tambahkan konfigurasi squid yang terletajk di `/etc/squid/squid.conf` pada node Water7 selaku proxy server. Maka file `/etc/squid/squid.conf` akan terlihat seperti berikut :
![12-A](https://cdn.discordapp.com/attachments/804405775988555776/908979541442191400/unknown.png)
* Setelah itu kita ubah nameserver yang terletak di file `/etc/resolv.conf` pada node Water7 menjadi IP miliki EniesLobby dengan perintah `nameserver 192.199.2.2`.
![12-B](https://cdn.discordapp.com/attachments/804405775988555776/908981520495173643/unknown.png)
* Untuk mengeceknya kita coba akses `lynx super.franky.d15.com` pada node client dengan username `luffybelikapald15` dan password `luffy_d15`, lalu kita masuk ke folder `public/images/` dan mendownload file `background-franky.jpg`. Apabila bisa dan ternyata speed download menurun hingga `1.2 KiB/sec` maka konfigurasi telah berhasil dilakukan
![12-C](https://cdn.discordapp.com/attachments/804405775988555776/908982876501078046/unknown.png)
* Sedangkan Zoro bertugas mencari file sisanya, otomatis zoro tidak diperbolehkan mengakses file image berupa `.jpg` dan `.png`. Maka untuk mengeceknya kita coba akses `lynx super.franky.d15.com` pada node client dengan username `zorobelikapald15` dan password `zoro_d15`. Lalu kita coba mengakses file `car.jpg`. Apabila forbidden, maka konfigurasi telah benar.
![12-D](https://cdn.discordapp.com/attachments/804405775988555776/908983832718508062/unknown.png)


### 13. Sedangkan, Zoro yang sangat bersemangat untuk mencari harta karun, sehingga kecepatan kapal Zoro tidak dibatasi ketika sudah mendapatkan harta yang diinginkannya
* Pada soal ini, kita tidak perlu melakukan apapun, karena pada konfigurasi `/etc/squid/squid.conf` di nomor 12, pembatasan kecepatan hanya diterapkan pada username `luffybelikapald15` sehingga secara default, user `zorobelikapald15` tidak akan dibatasi. Untuk mengeceknya maka kita coba akses `lynx super.franky.d15.com` pada node client,dan gunakan username `zorobelikapald15` dan password `zoro_d15`. Lalu kita masuk folder `public/images` dan download file `roboto-frankyasdjklkljqwennmn.listingbro`. Apabila file terbuka dengan sangat cepat, menandakan bahwa speed limit nya tidak dibatasi sehingga dapat download dengan cepat.
![13-A](https://cdn.discordapp.com/attachments/804405775988555776/908987399407812609/Keren.gif)
