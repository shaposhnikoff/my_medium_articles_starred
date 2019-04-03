
# Setting IP security (IPsec) di Centos 7 dengan Strongswan dan Cron


> # IP security (IPsec) merupakan sebuah protokol yang digunakan untuk mengamankan transmisi datagram dalam sebuah internetwork berbasis TCP/IP (wikipedia).

*Bismillah*

Pada kesempatan kali ini akan dibahas bagaimana cara konfigurasi IPsec untk melakukan koneksi H2H(host to host) agar bisa melakukan transaksi dengan aman. Berikut ini langkah-langkahnya :

* Install strongswan :yum install strongswan

* Setting ipsec.conf : nano /etc/ipsec.conf, tuliskan kode berikut

    con-priv
     left=103.1.1.1 # ip publik server
     leftsubnet=10.1.1.1/32 # ip private server
     right=103.2.2.2 # ip publik host yang akan diakses
     rightsubnet=10.2.2.2/32 # ip private host yang akan diakses
     ike=aes256-sha2_256-modp1024!
     esp=aes256-sha2_256!
     ikelifetime=1h
     lifetime=8h
     type=tunnel
     auto=start
     authby=secret
     keyexchange=ikev1

Untuk melakukan setting **ipsec.conf** pastikan untuk menyesuaikan kofigurasi **ipsec.conf** sesuai dengan host yang akan diakses. Karena perlu penyesuain agar berhasil dikoneksikan.

* setting ipsec.secret : nano /etc/ipsec.secrets

    103.1.1.1 103.2.2.2 : PSK "key-psk"

Sama seperti waktu setting ipsec.conf tadi, pastikan menyesuaikan dengan host yang akan diakses.

* restart ipsec : ipsec start

* aktifkan ipsec yang akan digunakan : ipsec up con-priv

Sampai disini sudah selesai untuk setting IPsec-nya, tapi sewaktu-waktu koneksinya tidak stabil. Agar tetap stabil perlu dilakukan restart secara berkala. Kita bisa memanfaatkan fitur **cron** di centos.

* tambahkan kode berikut di cron, dengan menggunakan code crontab -e , anda akan masuk ke vim editor,

    20 * * * * /usr/sbin/ipsec restart > /dev/null 2>&1

Notes : Gunakan perintah :w untuk menyimpan, dan gunakan :q untuk keluar dari vim editor.

* cek apakah schedule sudah terdaftar apa belum : crontab -l

Demikian tutorial melakukan setting IPsec di centos dengan menggunakan **strongswan** dan **cron** di centos 7, semoga bermanfaat.
