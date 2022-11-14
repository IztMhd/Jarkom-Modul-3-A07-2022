# Jarkom-Modul-3-A07-2022

## Anggota Kelompok

- I Putu Bagus Adhi Pradana (5025201010)
- Izzati Mukhammad (5025201075)
- Muhammad Damas Abhirama (5025201271)

## Nomor 1 dan 2
Loid bersama Franky berencana membuat peta tersebut dengan kriteria WISE sebagai DNS Server, Westalis sebagai DHCP Server, Berlint sebagai Proxy Server (1), dan Ostania sebagai DHCP Relay (2)

Membuat topologi sebagai berikut

Lalu melakukan setting pada masing masing node pada `network configuration`
- Ostania

      auto eth0
      iface eth0 inet dhcp

      auto eth1
      iface eth1 inet static
        address 192.172.1.1
        netmask 255.255.255.0

      auto eth2
      iface eth2 inet static
        address 192.172.2.1
        netmask 255.255.255.0

      auto eth3
      iface eth3 inet static
        address 192.172.3.1
        netmask 255.255.255.0
        
- WISE

      auto eth0
      iface eth0 inet static
        address 192.172.2.2
        netmask 255.255.255.0
        gateway 192.172.2.1
        
- Westalis
      
      auto eth0
      iface eth0 inet static
        address 192.172.2.4
        netmask 255.255.255.0
        gateway 192.172.2.1
        
- Berlint

      auto eth0
      iface eth0 inet static
        address 192.172.2.3
        netmask 255.255.255.0
        gateway 192.172.2.1
        
- SSS/Garden/Eden/NewstonCastle/KemonoPark
  
      auto eth0
      iface eth0 inet dhcp
      
Kemudian pada Ostania melakukan seperti berikut

    apt-get update
    iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 192.172.0.0/16
    apt-get install isc-dhcp-relay -y
      
Lalu edit konfigurasi pada `/etc/default/isc-dhcp-relay`

    echo "
    # What servers should the DHCP relay forward requests to?
    SERVERS=\"192.172.2.4\"

    # On what interfaces should the DHCP relay (dhrelay) serve DHCP requests?
    INTERFACES=\"eth1 eth3 eth2\"

    # Additional options that are passed to the DHCP relay daemon?
    OPTIONS=\"\"
    " > /etc/default/isc-dhcp-relay
    service isc-dhcp-relay restart

      
## Nomor 3
Semua client yang ada HARUS menggunakan konfigurasi IP dari DHCP Server.
Client yang melalui Switch1 mendapatkan range IP dari 192.172.1.50 - 192.172.1.88 dan 192.172.1.120 - 192.172.1.155

Pada Westalis sama seperti sebelumnya kita perlu melakukan hal seperti berikut

    apt-get update
    apt-get install isc-dhcp-relay -y
    echo 'INTERFACES="eth0"' > /etc/default/isc-dhcp-server
    service isc-dhcp-server restart
    
Kemudian melakukan set ip seperti yang diminta, yaitu seperti berikut

      echo "
      subnet 192.172.2.0 netmask 255.255.255.0 {
      }
      subnet 192.172.1.0 netmask 255.255.255.0 {
          range 192.172.1.50 192.172.1.88;
          range 192.172.1.120 192.172.1.155;
          option routers 192.172.1.1;
          option broadcast-address 192.172.1.255;
          option domain-name-servers 192.172.2.2;

      }" > /etc/dhcp/dhcpd.conf

## Nomor 4
Client yang melalui Switch3 mendapatkan range IP dari 192.172.3.10 - 192.172.3.30 dan 192.172.3.60 - 192.172.3.85

Sama seperti pada nomor 3 lakukan seperti berikut

      echo "
      subnet 192.172.3.0 netmask 255.255.255.0 {
          range 192.172.3.10 192.172.3.30;
          range 192.172.3.60 192.172.3.85;
          option routers 192.172.3.1;
          option broadcast-address 192.172.3.255;
          option domain-name-servers 192.172.2.2;
          default-lease-time 300;
          max-lease-time 6900;

      }" >> /etc/dhcp/dhcpd.conf

## Nomor 5
Client mendapatkan DNS dari WISE dan client dapat terhubung dengan internet melalui DNS tersebut

Sebelum melakukan konfigurasi pada `WISE` kita perlu melakukan hal berikut terlebih dahulu
      
      echo "nameserver 192.168.122.1" > /etc/resolv.conf
      apt-get update
      apt-get install bind9 -y
      service bind9 start
            
Kemudian pada `WISE` kita melakukan konfigurasi sebagai berikut
      
      echo "
      options {
            directory \"/var/cache/bind\";
            
            forwarders {
                  192.168.122.1;
            };
            
            // dnssec-validation auto;
            allow-query { any; };
            auth-nxdomain no;    # conform to RFC135
            listen-on-v6 { any; };
      };
      " > /etc/bind/named.conf.options
      service bind9 restart
            
## Nomor 6
Lama waktu DHCP server meminjamkan alamat IP kepada Client yang melalui Switch1 selama 5 menit sedangkan pada client yang melalui Switch3 selama 10 menit. Dengan waktu maksimal yang dialokasikan untuk peminjaman alamat IP selama 115 menit.

Pada `Westalis` dilakukan script sebagai berikut

      echo "
      subnet 192.172.2.0 netmask 255.255.255.0 {
      }
      subnet 192.172.1.0 netmask 255.255.255.0 {
          ...
          default-lease-time 300;
          max-lease-time 6900;

      }" > /etc/dhcp/dhcpd.conf
      
      echo "
      subnet 192.172.3.0 netmask 255.255.255.0 {
          ...
          default-lease-time 300;
          max-lease-time 6900;

      }" >> /etc/dhcp/dhcpd.conf


## Nomor 7
Loid dan Franky berencana menjadikan Eden sebagai server untuk pertukaran informasi dengan alamat IP yang tetap dengan IP 192.172.3.13

Menambahakan konfigurasi berikut pada `Westalis`
      
      echo"
      host Eden {
            hardware ethernet 22:e5:4a:ff:62:41;
            fixed-address 192.172.3.13;
      }" >> /etc/dhcp/dhcpd.conf
      
Dan pada interface `WISE` sebagau berikut

      auto eth0
      iface eth0 inet dhcp
      hwaddress ether 22:e5:4a:ff:62:41

## Note
Pada Proxy Server di Berlint, Loid berencana untuk mengatur bagaimana Client dapat mengakses internet. Artinya setiap client harus menggunakan Berlint sebagai HTTP & HTTPS proxy. Adapun kriteria pengaturannya adalah sebagai berikut:
1. Client hanya dapat mengakses internet diluar (selain) hari & jam kerja (senin-jumat 08.00 - 17.00) dan hari libur (dapat mengakses 24 jam penuh)
2. Adapun pada hari dan jam kerja sesuai nomor (1), client hanya dapat mengakses domain loid-work.com dan franky-work.com (IP tujuan domain dibebaskan)
3. Saat akses internet dibuka, client dilarang untuk mengakses web tanpa HTTPS. (Contoh web HTTP: http://example.com)
4. Agar menghemat penggunaan, akses internet dibatasi dengan kecepatan maksimum 128 Kbps pada setiap host (Kbps = kilobit per second; lakukan pengecekan pada tiap host, ketika 2 host akses internet pada saat bersamaan, keduanya mendapatkan speed maksimal yaitu 128 Kbps)
5. Setelah diterapkan, ternyata peraturan nomor (4) mengganggu produktifitas saat hari kerja, dengan demikian pembatasan kecepatan hanya diberlakukan untuk pengaksesan internet pada hari libur

Untuk memenuhi kriteria kriteria diatas dilakukan konfigurasi sebagai berikut:
Pada `Berlint`

      echo"
      acl AVAILABLE_WORKING time MTWHF 08:00-17:00
      acl WORKING_SITES dstdomain "/etc/squid/access.acl"
      acl WEEKEND time AS 00:00-23:59
      acl ssl_ports port 443
      acl unsafe_port port 80

      delay_pools 1
      delay_class 1 1
      delay_access 1 allow WEEKEND
      delay_access 1 deny all
      delay_parameters 1 16000/16000

      http_port 8080
      visible_hostname Berlint

      http_access allow all
      http_access allow ssl_ports !WORKING_SITES !AVAILABLE_WORKING
      http_access allow WORKING_SITES AVAILABLE_WORKING
      http_access deny all" >> /etc/squid/squid.conf
      
      echo"
      loid-work.com
      franky-work.com
      " >> /etc/squid/access.acl
      
Setelah dilakukan konfigurasi diatas dilakukan restart pada squid dengan `service squid restart`
      
Pada `WISE`
 
      echo"
      zone "loid-work.com" {
            type master;
            file "/etc/bind/jarkom3/loid-work.com";
            allow-transfer {192.172.3.13;};
      };

      zone "franky-work.com" {
            type master;
            file "/etc/bind/jarkom3/franky-work.com";
            allow-transfer {192.172.3.13;};
      };" >> /etc/bind/named.conf.local
      
      echo"
      ;
      ; BIND data file for local loopback interface
      ;
      $TTL    604800
      @       IN      SOA     loid-work.com. root.loid-work.com. (
                              2022111401      ; Serial
                               604800         ; Refresh
                                86400         ; Retry
                              2419200         ; Expire
                               604800 )       ; Negative Cache TTL
      ;
      @       IN      NS      loid-work.com.
      @       IN      A       192.172.2.2
      @     IN      AAAA   ::1
      ">>/etc/bind/jarkom3/loid-work.com
      
      echo"
      ;
      ; BIND data file for local loopback interface
      ;
      $TTL    604800
      @       IN      SOA     franky-work.com. root.franky-work.com. (
                              20221111401      ; Serial
                               604800         ; Refresh
                                86400         ; Retry
                              2419200         ; Expire
                               604800 )       ; Negative Cache TTL
      ;
      @       IN      NS      franky-work.com.
      @       IN      A       192.172.2.2
      @     IN      AAAA   ::1
      ">>/etc/bind/jarkom3/franky-work.com

Setelah dilakukan konfigurasi diatas dilakukan restart pada bind dengan `service bind9 restart`
 
      
      
      
