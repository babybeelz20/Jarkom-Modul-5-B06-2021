# Jarkom-Modul-5-B06-2021

## Anggota Kelompok : 
- 05111940000048 | Bagaskoro Kuncoro Ardi 
- 05111940000096 | Stefanus Albert Kosim 

#

### **A**

Tugas pertama kalian yaitu membuat topologi jaringan sesuai dengan rancangan yang diberikan Luffy

### Topology

![topology](https://github.com/babybeelz20/Jarkom-Modul-5-B06-2021/blob/main/screenshot/topologi.png)

Keterangan : 	
- Doriki adalah DNS Server
- Jipangu adalah DHCP Server
- Maingate dan Jorge adalah Web Server
- Jumlah Host pada Blueno adalah 100 host
- Jumlah Host pada Cipher adalah 700 host
- Jumlah Host pada Elena adalah 300 host
- Jumlah Host pada Fukurou adalah 200 host

### **Jawab**

Pada Doriki, diinstal bind9 dan file pada `etc/bind/named.conf.options` diubah dengan menyalakan forwarders menuju `192.168.122.1`, mengkomen `dnssec-validation auto;` dan menambahkan `allow-query{ any; };`

```
options {
        directory \"/var/cache/bind\";

        forwarders {
            192.168.122.1;
        };

        // dnssec-validation auto;
        allow-query{ any; };

        auth-nxdomain no;    # conform to RFC1035
        listen-on-v6 { any; };
}
```

### **B**

Luffy ingin meminta kalian untuk membuat topologi tersebut menggunakan teknik CIDR atau VLSM.

### **Jawab**

Pertama-tama adalah menentukan subnet yang ada pada topologi. IP dibagi dengan menggunakan teknik VLSM dengan pembagian subnet sebagai berikut

![pembagian subnet](https://github.com/babybeelz20/Jarkom-Modul-5-B06-2021/blob/main/screenshot/subnetting.png)

Setelah IP dibagi, kemudian dibuat tree pembagian untuk mempermudah dalam mencari NID tiap subnetnya

![Pohon VLSM](https://github.com/babybeelz20/Jarkom-Modul-5-B06-2021/blob/main/screenshot/tree.jpg)

Dari NID dan netmask yang diperoleh dari tree, dibuat tabel pembagian IP berikut

![pembagian IP](https://github.com/babybeelz20/Jarkom-Modul-5-B06-2021/blob/main/screenshot/pembagian_ip.jpeg)

### **C**

Setelah melakukan subnetting, lakukan Routing agar setiap perangkat pada jaringan tersebut dapat terhubung.

### **Jawab**

Routing pada topologi yang pembagiannya dilakukan dengan VLSM adalah sebagai berikut:

- Pada router Guanhao dan Water7 yang berada di ujung hanya memiliki default routing yang mengarah ke IP Foosha yang terhubung dengan kedua router tersebut.
- Pada router Foosha terdapat 6 routing yaitu routing ke subnet A1, A2, A3 dengan gateway IP Water7, dan routing ke subnet A6,A7,A8 dengan gateway IP Guanhao

Foosha :

```
route add -net 10.10.0.8 netmask 255.255.255.248 gw 10.10.0.2
route add -net 10.10.0.128 netmask 255.255.255.128 gw 10.10.0.2
route add -net 10.10.4.0 netmask 255.255.252.0 gw 10.10.0.2
route add -net 10.10.0.16 netmask 255.255.255.248 gw 10.10.0.6
route add -net 10.10.2.0 netmask 255.255.254.0 gw 10.10.0.6
route add -net 10.10.1.0 netmask 255.255.255.0 gw 10.10.0.6
```

Water7 dan Guanhao :

```
route add -net 0.0.0.0 netmask 0.0.0.0 gw 10.10.0.1
route add -net 0.0.0.0 netmask 0.0.0.0 gw 10.10.0.5
```

### **D**

Tugas berikutnya adalah memberikan ip pada subnet Blueno, Cipher, Fukurou, dan Elena secara dinamis menggunakan bantuan DHCP server. Kemudian kalian ingat bahwa kalian harus setting DHCP Relay pada router yang menghubungkannya.

### **Jawab**

Untuk memberikan ip dinamis kepada subnet client, maka perlu menginstall DHCP Server di Node Jipangu dengan perintah `apt-get install isc-dhcp-server -y`. Lalu pada interfaces diisi dengan eth0 yaitu interface yang terhubung dengan router Water7

```
# Defaults for isc-dhcp-server initscript
# sourced by /etc/init.d/isc-dhcp-server
# installed at /etc/default/isc-dhcp-server by the maintainer scripts

#
# This is a POSIX shell fragment
#

# Path to dhcpd's config file (default: /etc/dhcp/dhcpd.conf).
#DHCPD_CONF=/etc/dhcp/dhcpd.conf

# Path to dhcpd's PID file (default: /var/run/dhcpd.pid).
#DHCPD_PID=/var/run/dhcpd.pid

# Additional options to start dhcpd with.
#       Don't use options -cf or -pf here; use DHCPD_CONF/ DHCPD_PID instead
#OPTIONS=""

# On what interfaces should the DHCP server (dhcpd) serve DHCP requests?
#       Separate multiple interfaces with spaces, e.g. "eth0 eth1".
INTERFACES="eth0"
```

Kemudian sesuai dengan pembagian IP yang sudah ada sebelumnya, tiap-tiap client dibagi dengan range yang sesuai dengan jumlah Host yang ada

```
// Subnet DNS dan DHCP server, untuk memastikan bahwa gatewaynya adalah IP Water7
subnet 10.10.0.8 netmask 255.255.255.248 {
	option routers 10.10.0.9;
}

// Subnet Blueno
subnet 10.10.0.128 netmask 255.255.255.128 {
	range 10.10.0.130 10.10.0.231;
	option routers 10.10.0.129;
	option broadcast-address 10.10.0.255;
	option domain-name-servers 10.10.0.10;
	default-lease-time 360;
	max-lease-time 7200;
}

// Subnet Cipher
subnet 10.10.4.0 netmask 255.255.252.0 {
	range 10.10.4.2 10.10.6.192;
	option routers 10.10.4.1;
	option broadcast-address 10.10.7.255;
	option domain-name-servers 10.10.0.10;
	default-lease-time 360;
	max-lease-time 7200;
}

// Subnet Elena
subnet 10.10.2.0 netmask 255.255.254.0 {
	range 10.10.2.2 10.10.3.48;
	option routers 10.10.2.1;
	option broadcast-address 10.10.3.255;
	option domain-name-servers 10.10.0.10;
	default-lease-time 360;
	max-lease-time 7200;
}

// Subnet Fukurou
subnet 10.10.1.0 netmask 255.255.255.0 {
	range 10.10.1.2 10.10.1.203;
	option routers 10.10.1.1;
	option broadcast-address 10.10.1.255;
	option domain-name-servers 10.10.0.10;
	default-lease-time 360;
	max-lease-time 7200;
}
```

Setelah DHCP Server diatur, selanjutnya adalah menginstall DHCP Relay pada tiap-tiap router yang menghubungkan DHCP Server dengan client-client yaitu router Water7 dengan interface `eth0, eth1, eth2, eth3`, router Foosha dengan interface `eth1, eth2`, router Guanhao dengan interface `eth0, eth2, eth3`. Pada servers yang ada di config, diisikan IP DHCP Server yaitu `10.10.0.11` sehingga konfigurasinya adalah sebagai berikut:

```
# Water7
SERVERS="10.10.0.11"
INTERFACES="eth0 eth1 eth2 eth3"
OPTIONS=""

# Foosha
SERVERS="10.10.0.11"
INTERFACES="eth1 eth2"
OPTIONS=""

# Guanhao
SERVERS="10.10.0.11"
INTERFACES="eth0 eth2 eth3"
OPTIONS=""
```

### **Soal 1**

Agar topologi yang kalian buat dapat mengakses keluar, kalian diminta untuk mengkonfigurasi Foosha menggunakan iptables, tetapi Luffy tidak ingin menggunakan MASQUERADE.

### **Jawab**

Karena konfigurasinya tidak menggunakan MASQUERADE, maka IP Foosha yang terhubung ke NAT harus diganti menjadi static, namun karena IP nya tidak diketahui dan tidak dicari berdasarkan pembagian IP sebelumnya, IP yang terhubung ke Foosha terlebih dahulu diubah menjadi DHCP. Ketika node nya dibuka, catat IP yang disewakan pada Foosha dan ganti konfigurasinya menjadi statis dengan IP yang digunakan adalah IP yang disewakan tadi, serta gatewaynya adalah **192.168.122.1**. Karena gateway pada konfigurasi ini diisi, maka tidak perlu menambah routing di node tersebut. 

Setelah IP nya menjadi static, jalankan perintah berikut

```
iptables -t nat -A POSTROUTING -s 10.10.0.0/16 -o eth0 -j SNAT --to-source 192.168.122.2
```

Untuk mengecek apakah semua node sudah dapat mengakses luar topologi maka dicoba ping dari tiap node ke **google.com**

![Testing nomor 1](https://github.com/babybeelz20/Jarkom-Modul-5-B06-2021/blob/main/screenshot/testing1.png)

### **Soal 2**

Kalian diminta untuk mendrop semua akses HTTP dari luar Topologi kalian pada server yang merupakan DHCP Server dan DNS Server demi menjaga keamanan.

### **Jawab**

Untuk mempermudah melakukan filtering paket yang berasal dari luar topologi, konfigurasi diatur di router foosha dengan interface yang terhubung dengan NAT yaitu eth0 dengan perintah berikut

```
iptables -A FORWARD -d 10.10.0.8/29 -i eth0 -p tcp --dport 80 -j DROP
```

Pada perintah diatas, semua paket yang berasal dari interface eth0 yaitu paket dari luar topologi akan di drop jika mengakses subnet A1 dengan port 80.

Testing dilakukan dengan melihat nmap port 80 dengan IP Doriki dan Jipangu di router Foosha. Jika *state* nya adalah filtered, maka konfigurasi berhasil.

![Testing nomor 2](https://github.com/babybeelz20/Jarkom-Modul-5-B06-2021/blob/main/screenshot/testing2.png)

### **Soal 3**

Karena kelompok kalian maksimal terdiri dari 3 orang. Luffy meminta kalian untuk membatasi DHCP dan DNS Server hanya boleh menerima maksimal 3 koneksi ICMP secara bersamaan menggunakan iptables, selebihnya didrop.

### **Jawab**

Untuk membatasi banyak koneksi ICMP yang dapat dilakukan dalam satu waktu oleh DHCP dan DNS Server, digunakan perintah berikut di Doriki dan Jipangu

```
iptables -A INPUT -p icmp -m connlimit --connlimit-above 3 --connlimit-mask 0 -j DROP
```

Testing dilakukan dengan melakukan ping dari ke-4 pc client ke Doriki dan Jipangu

![Testing nomor 3 Doriki](https://github.com/babybeelz20/Jarkom-Modul-5-B06-2021/blob/main/screenshot/doriki3.gif)

![Testing nomor 3 Jipangu](https://github.com/babybeelz20/Jarkom-Modul-5-B06-2021/blob/main/screenshot/jipangu3.gif)

### **Soal 4**

Akses dari subnet Blueno dan Cipher hanya diperbolehkan pada pukul 07.00 - 15.00 pada hari Senin sampai Kamis. Selain itu di reject.

### **Jawab**

Untuk membatasi akses berdasarkan waktu di DNS Server, maka dilakukan konfigurasi di Doriki sebagai berikut

```
iptables -A INPUT -s 10.10.0.128/25 -m time --timestart 07:00 --timestop 15:00 --weekdays Mon,Tue,Wed,Thu -j ACCEPT
iptables -A INPUT -s 10.10.4.0/22 -m time --timestart 07:00 --timestop 15:00 --weekdays Mon,Tue,Wed,Thu -j ACCEPT
iptables -A INPUT -s 10.10.0.128/25 -j REJECT 
iptables -A INPUT -s 10.10.4.0/22 -j REJECT 
```

Konfigurasi diatas akan menolak (REJECT) paket masuk di Doriki yang berasal dari subnet Blueno dan Cipher di setiap waktu, dan akan menerima paket yang masuk dari subnet Blueno dan Cipher di hari Senin, Selasa, Rabu, Kamis pada pukul 07:00 hingga 15:00.

![Testing nomor 4 Accept](https://github.com/babybeelz20/Jarkom-Modul-5-B06-2021/blob/main/screenshot/testing4accept.png)

![Testing nomor 4 Reject](https://github.com/babybeelz20/Jarkom-Modul-5-B06-2021/blob/main/screenshot/testing4reject.png)

### **Soal 5**

Akses dari subnet Elena dan Fukuro hanya diperbolehkan pada pukul 15.01 hingga pukul 06.59 setiap harinya. Selain itu di reject.

### **Jawab**

Untuk membatasi akses berdasarkan waktu di DNS Server, maka dilakukan konfigurasi di Doriki sebagai berikut

```
iptables -A INPUT -s 10.10.2.0/23 -m time --timestart 06:59 --timestop 15:01 -j REJECT
iptables -A INPUT -s 10.10.1.0/24 -m time --timestart 06:59 --timestop 15:01 -j REJECT
```

Konfigurasi diatas akan menolak (REJECT) paket masuk di Doriki yang berasal dari subnet Elena dan Fukurou pada pukul 06:59 hingga 15:01. Sisanya akan diterima.

![Testing nomor 5 Accept](https://github.com/babybeelz20/Jarkom-Modul-5-B06-2021/blob/main/screenshot/testing5accept.png)

![Testing nomor 5 Reject](https://github.com/babybeelz20/Jarkom-Modul-5-B06-2021/blob/main/screenshot/testing5reject.png)


### **Soal 6**

Karena kita memiliki 2 Web Server, Luffy ingin Guanhao disetting sehingga setiap request dari client yang mengakses DNS Server akan didistribusikan secara bergantian pada Jorge dan Maingate.

### **Jawab**

Untuk mendistribusikan paket yang menuju DNS Server ke Jorge dan Maingate secara bergantian digunakan konfigurasi berikut di router Guanhao

```
iptables -t nat -A PREROUTING -p tcp -d 10.10.0.10 -m statistic --mode nth --every 2 --packet 0 -j DNAT --to-destination 10.10.0.18
iptables -t nat -A PREROUTING -p tcp -d 10.10.0.10 -j DNAT --to-destination 10.10.0.19
iptables -t nat -A POSTROUTING -p tcp -d 10.10.0.18 -j SNAT --to-source 10.10.0.10
iptables -t nat -A POSTROUTING -p tcp -d 10.10.0.19 -j SNAT --to-source 10.10.0.10
```

Konfigurasi diatas akan mengalihkan paket yang menuju DNS Server ke Jorge untuk paket pertama dari 2 paket yang diterima oleh router, kemudian Maingate akan menerima yang tidak diterima oleh Jorge yaitu paket kedua dari 2 paket yang diterima oleh router.

![Testing nomor 5](https://github.com/babybeelz20/Jarkom-Modul-5-B06-2021/blob/main/screenshot/testing6.png)