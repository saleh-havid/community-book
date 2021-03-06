# Implementasi OpenFlow pada OpenWRT

Pada tulisan ini akan dijelaskan bagaimana membuat TP-LINK TL-1043ND v1 menjadi openflow enabled switch menggunakan sistem Openwrt Barrier Breaker (14.07). Ada dua metode untuk menjadikan sistem OpenWRT menjadi openflow enabled switch yaitu:

* Metode Pantou dengan menambahkan package Pantou untuk versi openflow v1.0 atau package CPqD untuk versi openflow openflow v 1.3.
* Metode Open vSwtich dengan menambahkan package schuza untuk Open vSwitch versi 1.9.0 atau package Roan Huang untuk Open vSwitch 2.3.0.

Tulisan ini dapat diaplikasikan untuk hardware lain yang didukung oleh Openwrt, penulis sudah menguji beberapa tipe hardware lain seperti TP-Link 1043ND v2, TP-Link WR740N an TP-Link MR3420. Perbedaan mendasar pada tiap hardware yaitu pada konfigurasi pada network interfaces. Untuk konfigurasi network interfaces tiap hardware dapat merujuk pada http://wiki.openwrt.org/toh/start.

#####Pengenalan Openwrt
Openwrt adalah distribusi linux yang dikembangkan untuk embeded devices. Openwrt mempunyai beberapa versi, versi yang saat ini sedang dikembangkan disebut versi Trunk/Bleding Edge, code name untuk versi trunk pada saat ini adalah Chaos Calmer. Pada saat tulisan ini dibuat Openwrt mempunyai versi stabil yaitu Barrier Breaker (14.07) dan Barrier Breaker (12.09). Pada tulisan ini versi Openwrt yang dipakai adalah versi Barrier Breaker (14.07).

[Website Openwrt](https://openwrt.org/)

#####Pengenalan Pantou
Pantou dapat merubah wireless router/access point komersial menjadi openflow enabled switch. Openflow diimplementasikan sebagai aplikasi yang dijalankan pada sistem Openwrt. Pantau dikembangkan pada Openwrt versi Backfire (Linux 2.6.32), Openflow yang dikembangkan adalah versi 1.0 berbasis Stanford reference implementation (userspace).

[Website Pantou](http://archive.openflow.org/wk/index.php/Pantou_:_OpenFlow_1.0_for_OpenWRT)

#####Pengenalan CPqD Openflow 1.3 untuk sistem Openwrt

CPqD mengembangkan versi openflow 1.3 dengan pendekatan Pantou yang dapat dijalankan pada perangkat berbasis openwrt.

[Website CPqD](https://github.com/CPqD/ofsoftswitch13/wiki/OpenFlow-1.3-for-OpenWRT)

#####Pengenalan Open vSwitch
Open vSwitch merupakan multilayer virtual switch dengan kualitas production dengan lisensi Open Source Apache 2.0. Open vSwitch dirancang untuk otomasi jaringan sekala besar dengan menggunakan programmatic extension namun Open vSwitch tetap mendukung manajemen interfaces dan protocol standar seperti NetFlow, sFlow, IPFIX, RSPAN, CLI, LACP, 802.1ag.

[Website Open vSwitch](http://openvswitch.org/)

Beberapa individu maupun kelompok mengembangkan Open vSwitch agar dapat diimplementasikan pada OpenWRT diantaranya schuza yang mengembangkan Open vSwitch versi 1.9.0 dan Roan Huang dari National Taichung University of Education yang mengembangkan Open vSwitch versi 2.3.0. Pada Tulisan ini versi Open vSwitch yang dipakai adalah versi 2.3.0 yang dikembangkan oleh Roan Huang.

[Website Open vSwitch versi Schuza](https://github.com/schuza/openvswitch)

[Website Open vSwitch versi Roan Huang](https://github.com/pichuang/openvwrt)

##Membuat Firmware Openwrt
Pada bagian ini akan dijelaskan bagaimana cara membuat firmware Openwrt untuk TP-LINK TL-1043ND. Hal yang perlu dipersiapkan:

A. Linux (penulis memakai Ubuntu 14.04) dengan sisa hardisk > 10GB (lebih baik jika memiliki sisa hardsik > 20GB).

B. Koneksi Internet

###Langkah langkah Firmware Openwrt
####1. Install packages yang dibutuhkan untuk OpenWrt buildsystem

```
apt-get install build-essential binutils flex bison autoconf gettext texinfo sharutils subversion libncurses5-dev ncurses-term zlib1g-dev gawk gcc-multilib git-core libssl-dev
```

####2. Download Openwrt Barrier Breaker, asumsi folder untuk download adalah ~/ofwrt

```
cd ~/
mkdir ofwrt
cd ~/ofwrt
svn co svn://svn.openwrt.org/openwrt/branches/barrier_breaker
cd barrier_breaker
./scripts/feeds update -a
./scripts/feeds install -a
```

####3. Pilih target system (Atheros AR7xxx/AR9xxx) dan target profile (TP-LINK TL-wr1043nd/ND), pada tulisan ini hardware yang digunakan adalah TP-Link TL-1043ND v1, save dan exit
```
make menuconfig
```

####4. Kemudian lakukan proses build, proses ini cukup memakan waktu. Pastikan anda terhubung dengan internet dan tidak terjadi build error.

```
make prereq
make V=s

```

####5. Setelah berhasil melakukan proses build anda dapat menambahkan package ke firmware yang akan dibuild.
Anda dapat menggunakan openflow versi 1.0 (standford) dengan mengikuti langkah **5.1.A** atau menggunakan openflow versi 1.3 (CPqD) dengan mengikuti langkah **5.1.B** atau menggunakan Open vSwitch 2.3.0 (Roan Huang) dengan mengikuti langkah **5.2**.

####5.1.A Untuk menambahkan package openflow versi 1.0 (standford) ikuti langkah berikut :

```
cd ~/ofwrt/
git clone git://gitosis.stanford.edu/openflow-openwrt
cd ~/ofwrt/barrier_breaker/package/
ln -s ~/ofwrt/openflow-openwrt/openflow-1.0/
cd ~/ofwrt/barrier_breaker/
ln -s ~/ofwrt/openflow-openwrt/openflow-1.0/files
```

#####5.1.A.1 Konfigurasi package openflow dan package yang dibutuhkan oleh openflow.

```
cd ~/ofwrt/barrier_breaker
make menuconfig
```

* Pada menu menu network pilih **openflow**
* Pada menu Kernel Modules->Network Support pilih **kmod-tun** dan **kmod-sched-core & kmod-sched**
* Save dan exit

#####5.1.A.2 Konfigurasi kernel agar support queueing

```
make kernel_menuconfig
```

* Pada menu Networking Support->Networking options-> pilih **Hierarchical Token Bucket (HTB)**
* Save dan exit

#####5.1.A.3 Kemudian lakukan proses build, proses ini cukup memakan waktu. Pastikan anda terhubung dengan internet dan tidak terjadi build error.

```
make prereq
make V=s
```

#####5.1.A.4 Setelah berhasil anda akan mendapatkan firmware Openwrt dengan custom package openflow 1.0
Hasil dari proses build ini adalah firmware openwrt dengan custom package openflow 1.0 untuk TP-Link TL-1043ND yang berada pada folder  **/ofwrt/barrier_breaker/bin/ar71xx$**

```
havid@havid-MS-7756:~/ofwrt/barrier_breaker/bin/ar71xx$ ls
md5sums
openwrt-ar71xx-generic-nbg460n_550n_550nh-u-boot.bin
openwrt-ar71xx-generic-root.squashfs
openwrt-ar71xx-generic-root.squashfs-64k
openwrt-ar71xx-generic-tl-wr1043nd-v1-squashfs-factory.bin
openwrt-ar71xx-generic-tl-wr1043nd-v1-squashfs-sysupgrade.bin
openwrt-ar71xx-generic-tl-wr1043nd-v2-squashfs-factory.bin
openwrt-ar71xx-generic-tl-wr1043nd-v2-squashfs-sysupgrade.bin
openwrt-ar71xx-generic-uImage-gzip.bin
openwrt-ar71xx-generic-uImage-lzma.bin
openwrt-ar71xx-generic-vmlinux.bin
openwrt-ar71xx-generic-vmlinux.elf
openwrt-ar71xx-generic-vmlinux.gz
openwrt-ar71xx-generic-vmlinux.lzma
openwrt-ar71xx-generic-vmlinux-lzma.elf
packages
uboot-ar71xx-nbg460n_550n_550nh
```

####5.1.B Untuk menambahkan package openflow versi 1.3 (CPqD) ikuti langkah berikut :

```
cd ~/ofwrt/
git clone https://github.com/CPqD/openflow-openwrt.git
cd ~/ofwrt/barrier_breaker/package/
ln -s ~/ofwrt/openflow-openwrt/openflow-1.3/
cd ~/ofwrt/barrier_breaker/
ln -s ~/ofwrt/openflow-openwrt/openflow-1.3/files
```

#####5.1.B.1 Konfigurasi package openflow dan package yang dibutuhkan oleh openflow.

```
cd ~/ofwrt/barrier_breaker
make menuconfig
```

* Pada menu menu network pilih **openflow**
* Pada menu Kernel Modules->Network Support pilih **kmod-tun** dan **kmod-sched-core & kmod-sched**
* Save dan exit

#####5.1.B.2 Konfigurasi kernel agar support queueing

```
make kernel_menuconfig
```
* Pada menu Networking Support->Networking options-> pilih **Hierarchical Token Bucket ()**
* Save dan exit

#####5.1.B.3 Kemudian lakukan proses build, proses ini cukup memakan waktu. Pastikan anda terhubung dengan internet dan tidak terjadi build error.

```
make prereq
make V=s
```

#####5.1.B.4 Setelah berhasil anda akan mendapatkan firmware Openwrt dengan custom package openflow 1.3
Hasil dari proses build ini adalah firmware openwrt dengan custom package openflow 1.3 untuk TP-Link TL-1043ND yang berada pada folder  **/ofwrt/barrier_breaker/bin/ar71xx$**

```
havid@havid-MS-7756:~/ofwrt/barrier_breaker/bin/ar71xx$ ls
md5sums
openwrt-ar71xx-generic-nbg460n_550n_550nh-u-boot.bin
openwrt-ar71xx-generic-root.squashfs
openwrt-ar71xx-generic-root.squashfs-64k
openwrt-ar71xx-generic-tl-wr1043nd-v1-squashfs-factory.bin
openwrt-ar71xx-generic-tl-wr1043nd-v1-squashfs-sysupgrade.bin
openwrt-ar71xx-generic-tl-wr1043nd-v2-squashfs-factory.bin
openwrt-ar71xx-generic-tl-wr1043nd-v2-squashfs-sysupgrade.bin
openwrt-ar71xx-generic-uImage-gzip.bin
openwrt-ar71xx-generic-uImage-lzma.bin
openwrt-ar71xx-generic-vmlinux.bin
openwrt-ar71xx-generic-vmlinux.elf
openwrt-ar71xx-generic-vmlinux.gz
openwrt-ar71xx-generic-vmlinux.lzma
openwrt-ar71xx-generic-vmlinux-lzma.elf
packages
uboot-ar71xx-nbg460n_550n_550nh
```

####5.2 Untuk menambahkan package Open vSwitch versi 2.3.0 (Roan Huang) ikuti langkah berikut :

```
cd ~/ofwrt/barrier_breaker/
echo 'src-git openvswitch git://github.com/pichuang/openvwrt.git' >> feeds.conf
./scripts/feeds update openvswitch
./scripts/feeds install -a -p openvswitch
wget https://gist.githubusercontent.com/pichuang/7372af6d5d3bd1db5a88/raw/4e2290e3e184288de7623c02f63fb57c536e035a/openwrt-add-libatomic.patch -q -O - | patch -p1
```


#####5.2.1 Konfigurasi package Open vSwitch dan package yang dibutuhkan oleh Open vSwitch.

```
cd ~/ofwrt/barrier_breaker
make menuconfig
```

* Pada menu network pilih **openvswitch-common, openvswitch-switch** dan **openvswitch-ipsec (Optional)**
* Pada menu  Advanced configuration options (for developers) -> Toolchain Options -> Binutils Version -> pilih **Linaro binutils 2.24**
* **HAPUS PILIHAN / Unselect** Pada menu Advanced configuration options (for developers) -> Target Options -> **Build packages with MIPS16 instructions**
* Save dan exit

#####5.2.2 Untuk package Open vSwitch ada satu command yang perlu ditambahkan setiap sebelum melakukan proses build

```
echo '#CONFIG_KERNEL_BRIDGE is not set' >> .config
```

Kemudian lakukan proses build, proses ini cukup memakan waktu. Pastikan anda terhubung dengan internet dan tidak terjadi build error.

```
make prereq
make V=s

```
#####5.2.3 Setelah berhasil anda akan mendapatkan firmware Openwrt dengan custom package Open vSwitch 2.3.0
Hasil dari proses build ini adalah firmware openwrt dengan custom package Open vSwitch 2.3.0 untuk TP-Link TL-1043ND yang berada pada folder  **/ofwrt/barrier_breaker/bin/ar71xx$**

```
havid@havid-MS-7756:~/ofwrt/barrier_breaker/bin/ar71xx$ ls
md5sums
openwrt-ar71xx-generic-nbg460n_550n_550nh-u-boot.bin
openwrt-ar71xx-generic-root.squashfs
openwrt-ar71xx-generic-root.squashfs-64k
openwrt-ar71xx-generic-tl-wr1043nd-v1-squashfs-factory.bin
openwrt-ar71xx-generic-tl-wr1043nd-v1-squashfs-sysupgrade.bin
openwrt-ar71xx-generic-tl-wr1043nd-v2-squashfs-factory.bin
openwrt-ar71xx-generic-tl-wr1043nd-v2-squashfs-sysupgrade.bin
openwrt-ar71xx-generic-uImage-gzip.bin
openwrt-ar71xx-generic-uImage-lzma.bin
openwrt-ar71xx-generic-vmlinux.bin
openwrt-ar71xx-generic-vmlinux.elf
openwrt-ar71xx-generic-vmlinux.gz
openwrt-ar71xx-generic-vmlinux.lzma
openwrt-ar71xx-generic-vmlinux-lzma.elf
packages
uboot-ar71xx-nbg460n_550n_550nh
```

##Pemasangan Firmware Openwrt
Asumsi pemasangan fimware dari firmware bawaan TP-link, metode yang akan digunakan adalah flashing menggunakan web interfaces.

Ada banyak cara lain untuk memasang Firmware Openwrt, anda dapat melihat lebih lanjut pada website openwrt [Openwrt Generic Flashing](http://wiki.openwrt.org/doc/howto/generic.flashing)


Hal yang perlu dipersiapkan:

A. Hardware yang akan digunakan, penulis memakai TP-Link wr1043nd v1.10

B. Firmware openwrt dengan custom package openflow (Pantou/CPqD) / Open vSwtich yang sesuai dengan hardware yang akan digunakan. penulis memakai firmware hasil build dari langkah diatas yaitu **openwrt-ar71xx-generic-tl-wr1043nd-v1-squashfs-factory.bin**



###Langkah-langkah pemasangan firmware openwrt
![Upgrade Firmware](https://lh4.googleusercontent.com/-YGCDaf1Qtwk/VKz3Mb9neKI/AAAAAAAAAPQ/q16uMgsSVHU/w978-h550-no/1043ND%2BPantou%2BUpgrade.png)
Masuk ke alamat pengaturan wireless router anda, biasanya alamat pengaturan 192.168.0.1 / 192.168.1.1 dengan username admin password admin. Pilih menu System Tools -> Firmware Upgrade. Pilih file yang akan digunakan untuk mengupgrade. Pastikan anda memilih file yang sesuai dengan hardware dan memiliki kata **factory** pada tulisan ini saya menggunakan file **openwrt-ar71xx-generic-tl-wr1043nd-v1-squashfs-factory.bin**, selanjutnya tekan tombol upgrade dan tunggu sampai proses upgrade selesai.

Jika proses upgrade berhasil router akan melakukan restart, ketika selesai proses booting anda dapat mengakses router dengan melakukan telnet ke alamat 192.168.1.1 menggunakan LAN port.

![Upgrade Completed](https://lh4.googleusercontent.com/-7zEW8fjpx94/VKz3MYxX-nI/AAAAAAAAAPU/BGNlI24b_mU/w978-h550-no/1043ND%2BUpgrade%2BPantou%2BCompleted.png)

###Instalasi Package tambahan untuk metode CPqD/Pantou

Pada OpenWRT versi Barrier Breaker, package tc (traffic control) tidak dapat dimasukan dalam proses pembuatan firmware. Kita harus menambahakan package tc secara manual. Hubungkan router yang sudah diupgrade ke internet kemudian ikuti langkah berikut:

```
cd tmp
wget http://downloads.openwrt.org/attitude_adjustment/12.09/ar71xx/generic/packages/tc_3.3.0-1_ar71xx.ipk
opkg install ./tc_3.11.0-1_ar71xx.ipk

cd /sbin
ln -s /usr/sbin/tc

cd /etc
ln -s /lib/functions.sh
```

###Konfigurasi Network Interfaces, Wireless dan Firewall


####/etc/config/network

Secara default konfigurasi network interfaces pada openwrt terdiri dari LAN port, WAN port dan Wireless. Kita perlu merubah konfigurasi network interfaces agar tiap port dapat menjadi Openflow port. Pada konfigurasi network interfaces berikut, port eth0.5 (WAN Port) akan dihubungkan ke kontroller, setelah konfigurasi network di save dan diterapkan IP router akan berubah menjadi **192.168.77.71**. Anda dapat menyesuaikan IP router dengan memodifikasi option 'ipaddr' pada interface eth0.5.

```
config 'switch'
        option 'name' 'rtl8366rb'
        option 'reset' '1'
        option 'enable_vlan' '1'
        option 'enable_learning' '0'

config 'switch_vlan'
        option 'device' 'rtl8366rb'
        option 'vlan' '1'
        option 'ports' '1 5t'

config 'switch_vlan'
        option 'device' 'rtl8366rb'
        option 'vlan' '2'
        option 'ports' '2 5t'

config 'switch_vlan'
        option 'device' 'rtl8366rb'
        option 'vlan' '3'
        option 'ports' '3 5t'

config 'switch_vlan'
        option 'device' 'rtl8366rb'
        option 'vlan' '4'
        option 'ports' '4 5t'

config 'switch_vlan'
        option 'device' 'rtl8366rb'
        option 'vlan' '5'
        option 'ports' '0 5t'

config 'interface' 'loopback'
        option 'ifname' 'lo'
        option 'proto'  'static'
        option 'ipaddr' '127.0.0.1'
        option 'netmask' '255.0.0.0'

config globals 'globals'
        option ula_prefix 'fdab:e26a:5c0e::/48'

config 'interface'
        option 'ifname' 'eth0.1'
        option 'proto' 'static'

config 'interface'
        option 'ifname' 'eth0.2'
        option 'proto' 'static'

config 'interface'
        option 'ifname' 'eth0.3'
        option 'proto' 'static'

config 'interface'
        option 'ifname' 'eth0.4'
        option 'proto' 'static'

config 'interface'
        option 'ifname' 'eth0.5'
        option 'proto' 'static'
        option 'ipaddr' '192.168.77.71'
        option 'netmask' '255.255.255.0'

config 'interface'
        option ifname 'wlan0'
        option proto 'static'
```


####/etc/config/firewall

Karena WAN Port akan dipakai ke arah controller, manajemen router seperti telnet dan ssh akan melalui WAN Port, oleh karena itu kita perlu merubah konfigurasi firewall pada WAN seperti berikut:


```
root@OpenWrt:/# cat /etc/config/firewall
config zone
        option name             wan
        list   network          'wan'
        list   network          'wan6'
        option input            ACCEPT
        option output           ACCEPT
        option forward          ACCEPT
        option masq             1
        option mtu_fix          1
```



####/etc/config/wireless

```
config wifi-device  radio0
        option type     mac80211
        option channel  11
        option hwmode   11ng
        option path     'platform/ar933x_wmac'
        option htmode   HT20
        list ht_capab   SHORT-GI-20
        list ht_capab   SHORT-GI-40
        list ht_capab   RX-STBC1
        list ht_capab   DSSS_CCK-40
        # REMOVE THIS LINE TO ENABLE WIFI:
        #option disabled 1

config wifi-iface
        option device   radio0
        option network  lan
        option mode     ap
        option ssid     OpenVwrt
        option encryption none

```


###Konfigurasi Package Openflow
#### /etc/config/openflow
Untuk metode Pantou/CPqD anda mendefinisikan controller dan port yang akan dapat menjadi openflow port pada /etc/config/openflow. pada konfigurasi berikut port yang akan menjadi openflow port adalah eth0.1 eth0.2 eth0.3 eth0.4 wlan0 dan controller berada pada **tcp:192.168.77.30:6633**, anda dapat menyesuaikan ip controller dan openflow port dengan memodifikasi option 'ofctl' untuk controller dan option 'ofports' untuk openflow port.

```
config 'ofswitch'
        option 'dp' 'dp0'
        option 'dpid' '000000000001'
        option 'ofports' 'eth0.1 eth0.2 eth0.3 eth0.4 wlan0'
        option 'ofctl' 'tcp:192.168.77.30:6633'
        option 'mode'  'outofband'

```

###Konfigurasi Package Open vSwitch

Untuk medote Open vSwtich anda mendefinisikan perlu membuat bridge terlebih dahulu dan menambahkan openflow port ke bridge kemdudian anda dapat mendefinisikan controller. ikuti langkah berikut untuk mengkonfigurasi package Open vSwtich.

```
ovs-vsctl add-br ovs-br
ovs-vsctl add-port ovs-br eth0.1 -- set Interface eth0.1 ofport_request=1
ovs-vsctl add-port ovs-br eth0.2 -- set Interface eth0.2 ofport_request=2
ovs-vsctl add-port ovs-br eth0.3 -- set Interface eth0.3 ofport_request=3
ovs-vsctl add-port ovs-br eth0.4 -- set Interface eth0.4 ofport_request=4
ovs-vsctl add-port ovs-br wlan0 -- set Interface wlan0 ofport_request=5

ovs-vsctl set bridge ovs-br protocols=OpenFlow13
ovs-vsctl set-controller ovs-br tcp:192.168.77.30:6633

```
###Hasil pengujian implementasi

####Hasil pengujian implementasi metode Pantou/CPqD

![CPqD](https://lh5.googleusercontent.com/-2uvKfPdBTCo/VK0GCUkNE3I/AAAAAAAAAPs/zExmXy4jb2E/w978-h550-no/CPqD%2BTCP.png)

#####Ethernet0.1 - Ethernet 0.2

```
havid@havid-MS-7756:~$ ping 10.1.1.2 -c 5
PING 10.1.1.2 (10.1.1.2) 56(84) bytes of data.
64 bytes from 10.1.1.2: icmp_seq=1 ttl=64 time=0.697 ms
64 bytes from 10.1.1.2: icmp_seq=2 ttl=64 time=0.710 ms
64 bytes from 10.1.1.2: icmp_seq=3 ttl=64 time=0.709 ms
64 bytes from 10.1.1.2: icmp_seq=4 ttl=64 time=0.680 ms
64 bytes from 10.1.1.2: icmp_seq=5 ttl=64 time=0.709 ms

havid@havid-MS-7756:~$ iperf -s
------------------------------------------------------------
Server listening on TCP port 5001
TCP window size: 85.3 KByte (default)
------------------------------------------------------------
[  4] local 10.1.1.1 port 5001 connected with 10.1.1.2 port 33811
[ ID] Interval       Transfer     Bandwidth
[  4]  0.0-10.1 sec  38.2 MBytes  31.8 Mbits/sec
[  5] local 10.1.1.1 port 5001 connected with 10.1.1.2 port 33812
[  5]  0.0-10.1 sec  38.4 MBytes  31.9 Mbits/sec
[  4] local 10.1.1.1 port 5001 connected with 10.1.1.2 port 33813
[  4]  0.0-10.1 sec  38.2 MBytes  31.8 Mbits/sec
[  5] local 10.1.1.1 port 5001 connected with 10.1.1.2 port 33814
[  5]  0.0-10.1 sec  38.2 MBytes  31.8 Mbits/sec
[  4] local 10.1.1.1 port 5001 connected with 10.1.1.2 port 33815
[  4]  0.0-10.1 sec  38.4 MBytes  31.9 Mbits/sec
[  5] local 10.1.1.1 port 5001 connected with 10.1.1.2 port 33816
[  5]  0.0-10.1 sec  38.2 MBytes  31.9 Mbits/sec
```

#####Wlan0 - Ethernet 0.2

```
havid@havid-MS-7756:~$ ping 10.1.1.3 -c 5
PING 10.1.1.3 (10.1.1.3) 56(84) bytes of data.
64 bytes from 10.1.1.3: icmp_seq=1 ttl=64 time=1.46 ms
64 bytes from 10.1.1.3: icmp_seq=2 ttl=64 time=1.49 ms
64 bytes from 10.1.1.3: icmp_seq=3 ttl=64 time=1.48 ms
64 bytes from 10.1.1.3: icmp_seq=4 ttl=64 time=1.45 ms
64 bytes from 10.1.1.3: icmp_seq=5 ttl=64 time=2.06 ms

--- 10.1.1.3 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4006ms
rtt min/avg/max/mdev = 1.459/1.593/2.064/0.240 ms
havid@havid-MS-7756:~$

havid@havid-MS-7756:~$ iperf -s
------------------------------------------------------------
Server listening on TCP port 5001
TCP window size: 85.3 KByte (default)
------------------------------------------------------------
[  4] local 10.1.1.1 port 5001 connected with 10.1.1.3 port 51544
[ ID] Interval       Transfer     Bandwidth
[  4]  0.0-10.2 sec  22.1 MBytes  18.3 Mbits/sec
[  5] local 10.1.1.1 port 5001 connected with 10.1.1.3 port 51545
[  5]  0.0-10.2 sec  22.1 MBytes  18.3 Mbits/sec
[  4] local 10.1.1.1 port 5001 connected with 10.1.1.3 port 51546
[  4]  0.0-10.1 sec  21.4 MBytes  17.7 Mbits/sec
[  5] local 10.1.1.1 port 5001 connected with 10.1.1.3 port 51547
[  5]  0.0-10.2 sec  23.1 MBytes  19.0 Mbits/sec
[  4] local 10.1.1.1 port 5001 connected with 10.1.1.3 port 51548
[  4]  0.0-10.2 sec  22.5 MBytes  18.5 Mbits/sec
```

hasil lengkap pengujian anda dapat lihat pada file berikut:

[Dokumentasi Pantou](https://raw.githubusercontent.com/saleh-havid/community-book/master/Implemenasi%20Openflow%20pada%20Openwrt/Assets/Dokumentasi%20Pantou.txt)
[Dokumentasi  CPqD](https://raw.githubusercontent.com/saleh-havid/community-book/master/Implemenasi%20Openflow%20pada%20Openwrt/Assets/Dokumentasi%20%20CPqD.txt)


####Hasil pengujian implementasi metode Open vSwitch

![OvS](https://lh4.googleusercontent.com/-nlBiPWHXmN4/VK0GFAJj-zI/AAAAAAAAAP8/jT6Yt_DpkCE/w978-h550-no/OvS%2BTCP.png)

#####Ethernet0.1 - Ethernet 0.2

```
havid@havid-MS-7756:~$ ping 10.1.1.2 -c 5
PING 10.1.1.2 (10.1.1.2) 56(84) bytes of data.
64 bytes from 10.1.1.2: icmp_seq=1 ttl=64 time=1.16 ms
64 bytes from 10.1.1.2: icmp_seq=2 ttl=64 time=0.315 ms
64 bytes from 10.1.1.2: icmp_seq=3 ttl=64 time=0.292 ms
64 bytes from 10.1.1.2: icmp_seq=4 ttl=64 time=0.261 ms
64 bytes from 10.1.1.2: icmp_seq=5 ttl=64 time=0.251 ms


havid@havid-MS-7756:~$ iperf -s
------------------------------------------------------------
Server listening on TCP port 5001
TCP window size: 85.3 KByte (default)
------------------------------------------------------------
[  4] local 10.1.1.1 port 5001 connected with 10.1.1.2 port 52137
[ ID] Interval       Transfer     Bandwidth
[  4]  0.0-10.0 sec   608 MBytes   509 Mbits/sec
[  5] local 10.1.1.1 port 5001 connected with 10.1.1.2 port 52139
[  5]  0.0-10.0 sec   594 MBytes   497 Mbits/sec
[  4] local 10.1.1.1 port 5001 connected with 10.1.1.2 port 52140
[  4]  0.0-10.0 sec   606 MBytes   508 Mbits/sec
[  5] local 10.1.1.1 port 5001 connected with 10.1.1.2 port 52148
[  5]  0.0-10.0 sec   595 MBytes   498 Mbits/sec
[  4] local 10.1.1.1 port 5001 connected with 10.1.1.2 port 52150
[  4]  0.0-10.0 sec   615 MBytes   514 Mbits/sec
```

#####Wlan0 - Ethernet 0.2

```
havid@havid-MS-7756:~$ ping 10.1.1.3 -c 5
PING 10.1.1.3 (10.1.1.3) 56(84) bytes of data.
64 bytes from 10.1.1.3: icmp_seq=1 ttl=64 time=2.32 ms
64 bytes from 10.1.1.3: icmp_seq=2 ttl=64 time=1.05 ms
64 bytes from 10.1.1.3: icmp_seq=3 ttl=64 time=0.990 ms
64 bytes from 10.1.1.3: icmp_seq=4 ttl=64 time=1.09 ms
64 bytes from 10.1.1.3: icmp_seq=5 ttl=64 time=1.00 ms

--- 10.1.1.3 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4004ms
rtt min/avg/max/mdev = 0.990/1.292/2.321/0.517 ms

havid@havid-MS-7756:~$ iperf -s
------------------------------------------------------------
Server listening on TCP port 5001
TCP window size: 85.3 KByte (default)
------------------------------------------------------------
[  4] local 10.1.1.1 port 5001 connected with 10.1.1.3 port 59249
[ ID] Interval       Transfer     Bandwidth
[  4]  0.0-10.1 sec  23.5 MBytes  19.5 Mbits/sec
[  5] local 10.1.1.1 port 5001 connected with 10.1.1.3 port 59250
[  5]  0.0-10.2 sec  26.0 MBytes  21.4 Mbits/sec
[  4] local 10.1.1.1 port 5001 connected with 10.1.1.3 port 59251
[  4]  0.0-10.1 sec  25.6 MBytes  21.2 Mbits/sec
[  5] local 10.1.1.1 port 5001 connected with 10.1.1.3 port 59252
[  5]  0.0-10.1 sec  25.9 MBytes  21.4 Mbits/sec
[  4] local 10.1.1.1 port 5001 connected with 10.1.1.3 port 59253
[  4]  0.0-10.1 sec  27.4 MBytes  22.7 Mbits/sec

```

hasil lengkap pengujian anda dapat lihat pada file berikut:

[Dokumentasi Open vSwitch](https://raw.githubusercontent.com/saleh-havid/community-book/master/Implemenasi%20Openflow%20pada%20Openwrt/Assets/Dokumentasi%20OvS.txt)

Referensi:

http://wiki.openwrt.org/doc/howto/buildroot.exigence

http://archive.openflow.org/wk/index.php/Pantou_:_OpenFlow_1.0_for_OpenWRT

https://github.com/CPqD/openflow-openwrt

http://openvswitch.org/

https://github.com/pichuang/openvwrt

http://wiki.openwrt.org/toh/tp-link/tl-wr1043nd

http://vishalshahane.blogspot.com/2014/04/compiling-custom-firmware-for-wireless.html



