# Jarkom-Modul-2-B24-2023
## DNS dan Web Server
Anggota kelompok:
| Nama Anggota | NRP |
| ------------- | ------------- |
| Gilang Aliefidanto  | 5025211119  |
| Vito Febrian Ananta  | 5025211224  |

### 1. Yudhistira akan digunakan sebagai DNS Master, Werkudara sebagai DNS Slave, Arjuna merupakan Load Balancer yang terdiri dari beberapa Web Server yaitu Prabakusuma, Abimanyu, dan Wisanggeni.
1. Pada soal pertama kami diminta untuk membuat topologi yang telah ditentukan oleh asisten. Kami juga melakukan pengaturan terhadap IP supaya akan membedakan dengan kelompok lain.
2. Setelah mengatur baik di Virtual Machine dan di GNS3, kami membuat project pada GNS 3 dan membuat topologi sesuai dengan perintah dari asisten.
![topologi](resource/topologi.png)

3. Setelah itu, setting semua node ubuntu:
- RouterPandudewanata
```
auto eth0
iface eth0 inet dhcp

auto eth1
iface eth1 inet static
address 192.190.1.1
netmask 255.255.255.0

auto eth2
iface eth2 inet static
address 192.190.2.1
netmask 255.255.255.0

auto eth3
iface eth3 inet static
address 192.190.3.1
netmask 255.255.255.0
```
- SadewaClient
```
auto eth0
iface eth0 inet static
address 192.190.1.2
netmask 255.255.255.0
gateway 192.190.1.1
```
- NakulaClient
```
auto eth0
iface eth0 inet static
address 192.190.1.3
netmask 255.255.255.0
gateway 192.190.1.1
```
- YudhistiraDNSMaster
```
auto eth0
iface eth0 inet static
address 192.190.3.2
netmask 255.255.255.0
gateway 192.190.3.1
```
- WekudaraDNSSlave
```
auto eth0
iface eth0 inet static
address 192.190.2.2
netmask 255.255.255.0
gateway 192.190.2.1
```
- ArjunaLoadBalancer
```
auto eth0
iface eth0 inet static
address 192.190.2.3
netmask 255.255.255.0
gateway 192.190.2.1
```
- AbimanyuWebServer
```
auto eth0
iface eth0 inet static
address 192.190.2.4
netmask 255.255.255.0
gateway 192.190.2.1
```
PrabakusumaWebServer
```
auto eth0
iface eth0 inet static
address 192.190.2.5
netmask 255.255.255.0
gateway 192.190.2.1
```
WisanggeniWebServer
```
auto eth0
iface eth0 inet static
address 192.190.2.6
netmask 255.255.255.0
gateway 192.190.2.1
```
4. Setelah itu, reload semua node dan cari IP public
RouterPandudewanata
```
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 192.190.0.0/16
```
Node ubuntu lainnya
```
echo nameserver 192.168.122.1 > /etc/resolv.conf
```
5. Kemudian reload lagi semua node

### 2. Buatlah website utama pada node arjuna dengan akses ke arjuna.yyy.com dengan alias www.arjuna.yyy.com dengan yyy merupakan kode kelompok.
1. Pada `YudhistiraDNSMaster`, update dan install bind9
```
apt-get update
apt-get install bind9 -y
```
2. Pada file `/etc/bind/named.conf.local`, tulislah seperti di bawah ini
```
zone "arjuna.b24.com" {
	type master;
	file "/etc/bind/jarkom/arjuna.b24.com";
};
```
3. Buatlah directory `/etc/bind/jarkom`
```
mkdir /etc/bind/jarkom
```
4. Copy `/etc/bind/db.local` ke `/etc/bind/jarkom/arjuna.b24.com`
```
cp /etc/bind/db.local /etc/bind/jarkom/arjuna.b24.com
```
5. Tulislah file `/etc/bind/jarkom/arjuna.b24.com` seperti di bawah ini
```
$TTL	604800
@	IN	SOA	arjuna.b24.com.	root.arjuna.b24.com. (
			2023101201		; Serial
			604800		; Refresh
			86400			; Retry
			2419200		; Expire
			604800 )		; Negative Cache TTL
;
@	IN	NS		arjuna.b24.com.
@	IN	A		192.190.2.3		; IP Arjuna
www	IN	CNAME	arjuna.b24.com.
```
6. Kemudian pada `SadewaClient` dan `NakulaClient`, tambah alamat IP YudhistiraDNSMaster
```
echo nameserver 192.190.3.2 >> /etc/resolv.conf
```

### 3. Dengan cara yang sama seperti soal nomor 2, buatlah website utama dengan akses ke abimanyu.yyy.com dan alias www.abimanyu.yyy.com.
1. Pada file `/etc/bind/named.conf.local`, tambahkan tulisan
```
zone "abimanyu.b24.com" {
	type master;
	file "/etc/bind/jarkom/arjuna.b24.com";
};
```
2. Copy `/etc/bind/db.local` ke `/etc/bind/jarkom/abimanyu.b24.com`
```
cp /etc/bind/db.local /etc/bind/jarkom/abimanyu.b24.com
```
3. Tulislah file `/etc/bind/jarkom/abimanyu.b24.com` seperti di bawah ini
```
$TTL	604800
@	IN	SOA	abimanyu.b24.com.	root.abimanyu.b24.com. (
			2023101201		; Serial
			604800		; Refresh
			86400			; Retry
			2419200		; Expire
			604800 )		; Negative Cache TTL
;
@	IN	NS		abimanyu.b24.com.
@	IN	A		192.190.2.4		; IP Abimanyu
www	IN	CNAME	abimanyu.b24.com.

```

### 4. Kemudian, karena terdapat beberapa web yang harus di-deploy, buatlah subdomain parikesit.abimanyu.yyy.com yang diatur DNS-nya di Yudhistira dan mengarah ke Abimanyu.
1. Tambahkan teks ini pada file `/etc/bind/jarkom/abimanyu.b24.com`
```
parikesit	IN	A	192.190.2.4	; IP Abimanyu
```

### 5. Buat juga reverse domain untuk domain utama. (Abimanyu saja yang direverse)
1. Pada file `/etc/bind/named.conf.local` di `YudhistiraDNSMaster`, tambahkan tulisan
```
zone "2.190.192.in-addr.arpa" {
    	type master;
    	file "/etc/bind/jarkom/2.190.192.in-addr.arpa";
};
```
2. Copy `/etc/bind/db.local` ke `/etc/bind/jarkom/abimanyu.b24.com`
```
cp /etc/bind/db.local /etc/bind/jarkom/2.190.192.in-addr.arpa
```
3. Tulislah file `/etc/bind/jarkom/2.190.192.in-addr.arpa` seperti di bawah ini
```
$TTL	604800
@	IN	SOA	abimanyu.b24.com.	abimanyu.b24.com. (
			2023101201	; Serial
			604800		; Refresh
			86400		; Retry
			2419200		; Expire
			604800	)	; Negative Cache TTL
;
2.190.192.in-addr.arpa.	IN	NS	abimanyu.b24.com.
4			IN	PTR	abimanyu.b24.com.
```


### 6. Agar dapat tetap dihubungi ketika DNS Server Yudhistira bermasalah, buat juga Werkudara sebagai DNS Slave untuk domain utama.
1. Pada file `/etc/bind/named.conf.local` di `YudhistiraDNSMaster`, tambahkan tulisan pada arjuna.B24.com dan abimanyu.B24.com:
```
zone "arjuna.b24.com" {
	type master;
	notify yes;
	also-notify { 192.190.2.2; };
    	allow-transfer { 192.190.2.2; };
	file "/etc/bind/jarkom/arjuna.b24.com";
};

zone "abimanyu.b24.com" {
	type master;
	notify yes;
	also-notify { 192.190.2.2; };
    	allow-transfer { 192.190.2.2; };
	file "/etc/bind/jarkom/abimanyu.b24.com";
};

zone "2.190.192.in-addr.arpa" {
    	type master;
    	file "/etc/bind/jarkom/2.190.192.in-addr.arpa";
};
```
2. Kemudian pada `WerkudaraDNSSlave`, update dan install bind9
```
apt-get update
apt-get install bind9 -y
```
3.  Pada file `/etc/bind/named.conf.local`, tuliskan seperti di bawah ini:
```
zone "arjuna.b24.com" {
	type slave;
    	masters { 192.190.3.2; };
    	file "/var/lib/bind/arjuna.b24.com";
};

zone "abimanyu.b24.com" {
	type slave;
    	masters { 192.190.3.2; };
    	file "/var/lib/bind/abimanyu.b24.com";
};
```
4. Kemudian pada `SadewaClient` dan `NakulaClient`, tambah alamat IP YudhistiraDNSMaster
```
echo nameserver 192.190.2.2 >> /etc/resolv.conf
```


### 7. Seperti yang kita tahu karena banyak sekali informasi yang harus diterima, buatlah subdomain khusus untuk perang yaitu baratayuda.abimanyu.yyy.com dengan alias www.baratayuda.abimanyu.yyy.com yang didelegasikan dari Yudhistira ke Werkudara dengan IP menuju ke Abimanyu dalam folder Baratayuda.
1. Tambahkan teks ini pada file `/etc/bind/jarkom/abimanyu.b24.com`
```
ns1	IN	A		192.190.2.4	; IP Abimanyu
baratayuda	IN	NS	ns1
```
2. Kemudian ubah teks pada file `/etc/bind/named.conf.options` pada `YudhistiraDNSMaster` seperti di bawah ini
```
options {
        directory "/var/cache/bind";

        // If there is a firewall between you and nameservers you want
        // to talk to, you may need to fix the firewall to allow multiple
        // ports to talk.  See http://www.kb.cert.org/vuls/id/800113

        // If your ISP provided one or more IP addresses for stable
        // nameservers, you probably want to use them as forwarders.
        // Uncomment the following block, and insert the addresses replacing
        // the all-0s placeholder.

        // forwarders {
        //      0.0.0.0;
        // };

        //========================================================================
        // If BIND logs error messages about the root key being expired,
        // you will need to update your keys.  See https://www.isc.org/bind-keys
        //========================================================================
        //dnssec-validation auto;
        allow-query{any;};

        auth-nxdomain no;    # conform to RFC1035
        listen-on-v6 { any; };
};
```
3. Kemudian ubah teks pada file `/etc/bind/named.conf.options` pada `WerkudaraDNSSlave` seperti di bawah ini
```
options {
        directory "/var/cache/bind";

        // If there is a firewall between you and nameservers you want
        // to talk to, you may need to fix the firewall to allow multiple
        // ports to talk.  See http://www.kb.cert.org/vuls/id/800113

        // If your ISP provided one or more IP addresses for stable
        // nameservers, you probably want to use them as forwarders.
        // Uncomment the following block, and insert the addresses replacing
        // the all-0s placeholder.

        // forwarders {
        //      0.0.0.0;
        // };

        //========================================================================
        // If BIND logs error messages about the root key being expired,
        // you will need to update your keys.  See https://www.isc.org/bind-keys
        //========================================================================
        //dnssec-validation auto;
        allow-query{any;};

        auth-nxdomain no;    # conform to RFC1035
        listen-on-v6 { any; };
};
```
4. Kemudian pada file `/etc/bind/named.conf.local`, tambahkan tulisan di bawah ini:
```
zone "baratayuda.abimanyu.b24.com" {
	type master;
file "/etc/bind/baratayuda/baratayuda.abimanyu.b24.com";
};
```
5. Kemudian buat directory `/etc/bind/baratayuda`
```
mkdir /etc/bind/baratayuda
```
6. Copy `/etc/bind/db.local` ke `/etc/bind/baratayuda/baratayuda.abimanyu.b24.com`
```
cp /etc/bind/db.local /etc/bind/baratayuda/baratayuda.abimanyu.b24.com
```
7. Tulislah file `/etc/bind/baratayuda/baratayuda.abimanyu.b24.com` seperti di bawah ini
```
$TTL	604800
@	IN	SOA	baratayuda.abimanyu.b24.com. root.baratayuda.abimanyu.b24.com. (
			2023101201	; Serial
			604800		; Refresh
			86400		; Retry
			2419200		; Expire
			604800	)	; Negative Cache TTL
;
@	IN	NS	baratayuda.abimanyu.b24.com.
@	IN	A	192.190.2.4	; IP Abimanyu
www	IN	CNAME	baratayuda.abimanyu.b24.com.
```

### 8. Untuk informasi yang lebih spesifik mengenai Ranjapan Baratayuda, buatlah subdomain melalui Werkudara dengan akses rjp.baratayuda.abimanyu.yyy.com dengan alias www.rjp.baratayuda.abimanyu.yyy.com yang mengarah ke Abimanyu.
1. Tambahkan teks ini pada file `/etc/bind/baratayuda/baratayuda.abimanyu.b24.com`
```
$TTL	604800
@	IN	SOA	baratayuda.abimanyu.b24.com. root.baratayuda.abimanyu.b24.com. (
			2023101201	; Serial
			604800		; Refresh
			86400		; Retry
			2419200		; Expire
			604800	)	; Negative Cache TTL
;
@	IN	NS	baratayuda.abimanyu.b24.com.
@	IN	A	192.190.2.4	; IP Abimanyu
www	IN	CNAME	baratayuda.abimanyu.b24.com.
rjp	IN	A	192.190.2.4	; IP Abimanyu
www.rjp	IN	CNAME	baratayuda.abimanyu.b24.com.
```

### 9. Arjuna merupakan suatu Load Balancer Nginx dengan tiga worker (yang juga menggunakan nginx sebagai webserver) yaitu Prabakusuma, Abimanyu, dan Wisanggeni. Lakukan deployment pada masing-masing worker.
1. Pada semua node webserver, lakukan update dan install `wget` dan `unzip`
```
apt-get update
apt-get install wget unzip -y
```
2. Kemudian download resource yang diberikan asisten, dan ekstrak file tersebut
```
wget --no-check-certificate 'https://docs.google.com/uc?export=download&id=17tAM_XDKYWDvF-JJix1x7txvTBEax7vX' -O downloaded-zip
unzip downloaded-zip -d /var/www/
mv /var/www/arjuna.yyy.com /var/www/arjuna.b24.com
rm downloaded-zip
```
3. Install nginx, php, dan php-fpm
```
apt-get update && apt install nginx php php-fpm -y
php -v
```
4. Kemudian ubah teks pada file `/etc/nginx/sites-available/arjuna.b24.com` seperti di bawah ini
```
server {

	listen 80;

	root /var/www/arjuna.b24.com;

	index index.php index.html index.htm;
	server_name _;

	location / {
			try_files $uri $uri/ /index.php?$query_string;
	}

	# pass PHP scripts to FastCGI server
	location ~ \.php$ {
	include snippets/fastcgi-php.conf;
	fastcgi_pass unix:/var/run/php/php7.0-fpm.sock;
	}

location ~ /\.ht {
			deny all;
	}

	error_log /var/log/nginx/arjuna.b24.com_error.log;
	access_log /var/log/nginx/arjuna.b24.com_access.log;
}
```
5. Kemudian buat symlink
```
ln -s /etc/nginx/sites-available/arjuna.b24.com /etc/nginx/sites-enabled
```
6. Restart nginx
```
service nginx restart
nginx -t
```
7. Hapus `/etc/nginx/sites-enabled/default`
```
rm -rf /etc/nginx/sites-enabled/default
```
8. Restart nginx lagi
```
service nginx restart
nginx -t
service php7.0-fpm start
service php7.0-fpm restart
```
9. Pada `ArjunaLoadBalancer`, install nginx
```
apt-get update
apt-get install nginx -y
```
10. Buat load balancer menggunakan algoritma `Round robin`
```
upstream myweb  {
	server 192.190.2.4; #IP Abimanyu
	server 192.190.2.5; #IP Prabakusuma
	server 192.190.2.6; #IP Wisanggeni
}

server {
	listen 80;
	server_name arjuna.b24.com;

	location / {
	proxy_pass http://myweb;
	}
}
```
11. Kemudian buat symlink
```
ln -s /etc/nginx/sites-available/lb-arjuna /etc/nginx/sites-enabled
```
12. Restart nginx
```
service nginx restart
nginx -t
nano 9-10-troubleshooting.sh
```
13. Hapus `/etc/nginx/sites-enabled/default`
```
rm -rf /etc/nginx/sites-enabled/default
```
14. Restart nginx lagi
```
service nginx restart
nginx -t
```
15. Pada `NakulaClient` dan `SadewaClient`, install lynx
```
apt-get update
apt-get install lynx -y
```
16. Tambahkan alamat IP Arjuna ke `/etc/resolv.conf`
```
nameserver 192.190.2.3 # IP Arjuna
```
17. Pengujian
```
lynx http://arjuna.b24.com
```

### 10. Kemudian gunakan algoritma Round Robin untuk Load Balancer pada Arjuna. Gunakan server_name pada soal nomor 1. Untuk melakukan pengecekan akses alamat web tersebut kemudian pastikan worker yang digunakan untuk menangani permintaan akan berganti ganti secara acak. Untuk webserver di masing-masing worker wajib berjalan di port 8001-8003.
```
Contoh
    - Prabakusuma:8001
    - Abimanyu:8002
    - Wisanggeni:8003
```
1. Sudah terjawab di nomor 9
