# Jarkom-Modul-2-F03-2022

### Kelompok F03

| **No** | **Nama**                   | **NRP**    | **Pembagian Pekerjaan** |
| ------ | -------------------------- | ---------- | ----------------------- |
| 1      | Naufal Fabian Wibowo    | 05111940000223 | Mengerjakan Lapres |
| 2      | Angela Oryza Prabowo          | 5025201022 | Mengerjakan No 1-7, Menambahkan Dokumentasi |
| 3      | Helmi Taqiyuddin | 5025201152 | Mengerjakan Lapres |

``` IP Kelompok F03 = 10.30``` 


## 1
> WISE akan dijadikan sebagai DNS Master, Berlint akan dijadikan DNS Slave, dan Eden akan digunakan sebagai Web Server. Terdapat 2 Client yaitu SSS, dan Garden. Semua node terhubung pada router Ostania, sehingga dapat mengakses internet (1).

**Router network configuration Ostania:**

auto eth0
iface eth0 inet dhcp
```
auto eth1
iface eth1 inet static
	address 10.30.1.1
	netmask 255.255.255.0

auto eth2
iface eth2 inet static
	address 10.30.2.1
	netmask 255.255.255.0

auto eth3
iface eth3 inet static
	address 10.30.3.1
	netmask 255.255.255.0
```

iptables setup:
```
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 10.30.0.0/16
```


**DNS master network configuration WISE:**
```
auto eth0
iface eth0 inet static
	address 10.30.2.2
	netmask 255.255.255.0
	gateway 10.30.2.1
```

Instal bind9:
```
apt update
apt install bind9 -y
```


**DNS slave network configuration Berlint:**
```
auto eth0
iface eth0 inet static
	address 10.30.3.2
	netmask 255.255.255.0
	gateway 10.30.3.1
```

Instal bind9:
```
apt update
apt install bind9 -y
```


**Web server network configuration Eden:**

```
auto eth0
iface eth0 inet static
	address 10.30.3.3
	netmask 255.255.255.0
	gateway 10.30.3.1
```


**Client network configuration SSS:**
```
auto eth0
iface eth0 inet static
	address 10.30.1.2
	netmask 255.255.255.0
	gateway 10.30.1.1
```

Instal dnsutils:
```
apt update
apt install dnsutils -y
```


**Client network configuration Gardent :**
```
auto eth0
iface eth0 inet static
	address 10.30.1.3
	netmask 255.255.255.0
	gateway 110.30.1.1
```

Instal dnsutils:

```
apt update
apt install dnsutils -y
```



## 2
> Untuk mempermudah mendapatkan informasi mengenai misi dari Handler, bantulah Loid membuat website utama dengan akses wise.yyy.com dengan alias www.wise.yyy.com pada folder wise (2).

## 3
> Setelah itu ia juga ingin membuat subdomain eden.wise.yyy.com dengan alias www.eden.wise.yyy.com yang diatur DNS-nya di WISE dan mengarah ke Eden (3).

## 4
> Buat juga reverse domain untuk domain utama (4).
### Reverse Domain
#### Pada WISE
```
nano /etc/bind/named.conf.local

zone "2.3.30.10.in-addr.arpa" {
    type master;
    file "/etc/bind/wise/2.3.30.10.in-addr.arpa";
};

cp /etc/bind/db.local /etc/bind/wise/2.3.30.10.in-addr.arpa
```

#### Host pada Client
```
apt-get update

apt-get install dnsutils

host -t PTR 10.30.3.2
```

## 5
> Agar dapat tetap dihubungi jika server WISE bermasalah, buatlah juga Berlint sebagai DNS Slave untuk domain utama (5).

### Konfigurasi Berlint Sebagai DNS Slave
#### Pada WISE
```
nano /etc/bind/named.conf.local

zone "wise.f03.com" {
    type master;
    notify yes;
    also-notify {10.30.2.2;};
    allow-transfer {10.30.2.2;};
    file "/etc/bind/wise/wise.f03.com";
};

service bind9 restart
```

#### Pada Berlint
```
apt-get update

apt-get install bind9 -y

nano /etc/bind/named.conf.local

zone "wise.f03.com" {
    type slave;
    masters {10.30.3.2;};
    file "/var/lib/bind/wise.f03.com";
};

service bind9 restart
```


## 6
> Karena banyak informasi dari Handler, buatlah subdomain yang khusus untuk operation yaitu operation.wise.yyy.com dengan alias www.operation.wise.yyy.com yang didelegasikan dari WISE ke Berlint dengan IP menuju ke Eden dalam folder operation (6).

### Membuat Subdomain
#### Pada Wise 
```
nano /etc/bind/wise/wise.f03.com

ns1  IN   A   10.30.2.2 ; IP Berlint
operation IN    NS  ns1


nano /etc/bind/named.conf.options
allow-query{any;};

service bind9 restart
```

#### Pada Berlint
```
nano /etc/bind/named.conf.options

allow-query{any;};

nano /etc/bind/named.conf.local

diganti menjadi = operation.wise.f03.com

cp /etc/bind/db.local /etc/bind/wise/operation.wise.f03.com

nano /etc/bind/wise/operation.wise.f03.com

operation.wise.f03.com. IN SOA operation.wise.f03.com. root.operation.wise.f03.com. (
 2022100601 ; serial -> 2 ; serial
 604800 ; refresh
 86400 ; retry
 2419200 ; expire
 604800 ; minimum ttl
)
@ IN NS operation.wise.f03.com.
@ IN A  10.30.2.2 ; IP Berlint

service bind9 restart

ping operation.wise.f03.com -c 5
```

## 7 
> Untuk informasi yang lebih spesifik mengenai Operation Strix, buatlah subdomain melalui Berlint dengan akses strix.operation.wise.yyy.com dengan alias www.strix.operation.wise.yyy.com yang mengarah ke Eden (7).

```
nano /etc/bind/operation/operation.wise.f03.com

operation.wise.f03.com. IN SOA operation.wise.f03.com. root.operation.wise.f03.com. (
 2022100601 ; serial -> 2 ; serial
 604800 ; refresh
 86400 ; retry
 2419200 ; expire
 604800 ; minimum ttl
)
@ IN NS operation.wise.f03.com.
@ IN A  10.30.2.2 ; IP Berlint
strix IN A  10.30.2.2 ; IP Berlint

service bind9 restart

ping strix.operation.wise.f03.com -c 5
```

## 8 
> Setelah melakukan konfigurasi server, maka dilakukan konfigurasi Webserver. Pertama dengan webserver www.wise.yyy.com. Pertama, Loid membutuhkan webserver dengan DocumentRoot pada /var/www/wise.yyy.com (8).

## 9
> Setelah itu, Loid juga membutuhkan agar url www.wise.yyy.com/index.php/home dapat menjadi menjadi www.wise.yyy.com/home (9).

## 10
> Setelah itu, pada subdomain www.eden.wise.yyy.com, Loid membutuhkan penyimpanan aset yang memiliki DocumentRoot pada /var/www/eden.wise.yyy.com (10). 

## 11
> Akan tetapi, pada folder /public, Loid ingin hanya dapat melakukan directory listing saja (11). 

## 12
> Tidak hanya itu, Loid juga ingin menyiapkan error file 404.html pada folder /error untuk mengganti error kode pada apache (12).

## 13
>  Loid juga meminta Franky untuk dibuatkan konfigurasi virtual host. Virtual host ini bertujuan untuk dapat mengakses file asset www.eden.wise.yyy.com/public/js menjadi www.eden.wise.yyy.com/js (13).

## 14
> Loid meminta agar www.strix.operation.wise.yyy.com hanya bisa diakses dengan port 15000 dan port 15500 (14) 

## 15
> dengan autentikasi username Twilight dan password opStrix dan file di /var/www/strix.operation.wise.yyy (15) 

## 16
> dan setiap kali mengakses IP Eden akan dialihkan secara otomatis ke www.wise.yyy.com (16).

## 17
> Karena website www.eden.wise.yyy.com semakin banyak pengunjung dan banyak modifikasi sehingga banyak gambar-gambar yang random, maka Loid ingin mengubah request gambar yang memiliki substring “eden” akan diarahkan menuju eden.png. Bantulah Agent Twilight dan Organisasi WISE menjaga perdamaian! (17)
