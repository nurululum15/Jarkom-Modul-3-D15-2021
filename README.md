# Laporan Praktikum Modul 3

## Anggota  Kelompok D15 :

## Richard Asmarakandi (05111940000017)

## Nurul Izzatil Ulum (05111940000058)



## SOAL

### 1. Luffy bersama Zoro berencana membuat peta tersebut dengan kriteria EniesLobby sebagai DNS Server, Jipangu sebagai DHCP Server, Water7 sebagai Proxy Server.

### 2. Foosha sebagai DHCP Relay.

### 3. Semua client yang ada HARUS menggunakan konfigurasi IP dari DHCP Server. Client yang melalui Switch1 mendapatkan range IP dari [prefix IP].1.20 - [prefix IP].1.99 dan [prefix IP].1.150 - [prefix IP].1.169.

### 4. Client yang melalui Switch3 mendapatkan range IP dari [prefix IP].3.30 - [prefix IP].3.50.

### 5. Client mendapatkan DNS dari EniesLobby dan client dapat terhubung dengan internet melalui DNS tersebut.

### 6. Lama waktu DHCP server meminjamkan alamat IP kepada Client yang melalui Switch1 selama 6 menit sedangkan pada client yang melalui Switch3 selama 12 menit. Dengan waktu maksimal yang dialokasikan untuk peminjaman alamat IP selama 120 menit.

### 7. Luffy dan Zoro berencana menjadikan Skypie sebagai server untuk jual beli kapal yang dimilikinya dengan alamat IP yang tetap dengan IP [prefix IP].3.69.

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
* Kita generate password menggunakan `htpasswd` pada Water7, selaku pemilik proxy server. Pertama-tama kita install apache2-utils dengan `apt-get install apache2-utils`. setelah itu kita gunakan command `htpasswd -b -c -m /etc/squid/passwd luffybelikapald15 luffyd15` untuk menambahkan user baru dengan nama luffybelikapald15 dan password luffyd15.
* Lalu untuk menambahkan user zoro, kita gunakan command `htpasswd -b -m /etc/squid/passwd zorobelikapald15 zorod15`. kita hilangkan `-c` karena berarti membuat file, sedangkan file telah dibuat saat kita menambahkan user untuk luffybelikapald15.
![9-A](https://cdn.discordapp.com/attachments/804405775988555776/908621368927588352/unknown.png)
* Maka setelah digenerate, file `/etc/squid/passwd` akan terlihat seperti berikut :
![9-B](https://media.discordapp.net/attachments/804405775988555776/908621459444883486/unknown.png?width=842&height=473)
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


### 12. Saatnya berlayar! Luffy dan Zoro akhirnya memutuskan untuk berlayar untuk mencari harta karun di super.franky.yyy.com. Tugas pencarian dibagi menjadi dua misi, Luffy bertugas untuk mendapatkan gambar (.png, .jpg), sedangkan Zoro mendapatkan sisanya. Karena Luffy orangnya sangat teliti untuk mencari harta karun, ketika ia berhasil mendapatkan gambar, ia mendapatkan gambar dan melihatnya dengan kecepatan 10 kbps

### 13. Sedangkan, Zoro yang sangat bersemangat untuk mencari harta karun, sehingga kecepatan kapal Zoro tidak dibatasi ketika sudah mendapatkan harta yang diinginkannya