---
layout: post
title:  "VPN ke On-Premise via Azure"
date:   2020-03-21
---
Sebenernya gue agak bingung sih buat namain judulnya apaan, tapi mungkin kalo gue jelasin case nya baru pada paham harusnya hehe

Penjelasan Kasus
===
Jadi pada artikel in dibuat itu sedang tinggi-nya tingkat penyebaran virus _corona_, nha hampir seluruh kantor di Jakarta itu kan disuruh WFH (Work From Home) aja, dan kasusnya bermula disini.

Jadi di salah satu perusahaan tersebut di dalam Head Office-nya (_on-premise_) itu ada 1 server penting yang harus bisa diakses oleh seluruh karyawan-nya (katakanlan 100 karyawan), berarti kan tiap karyawan-nya itu harus disediakan koneksi VPN ke HO tersebut, namun di kantor tersebut hanya ada router dengan spesifikasi kecil yang hanya bisa menampung 10 connection vpn. 

Terus misalkan beli router yang dengan spek lebih besar gimana? Nha karena pada saat itu sedang kasus corona masih tinggi, jadi order router dengan spesifikasi yang diinginkan akan lama sampai-nya, sedangkan pada kasus ini butuh urgent.

Nah dari kasus tersebut, ternyata Azure ini punya solusi-nya haha. yaitu bisa dibilang menggabungkan VPN point-to-site (P2S) dan site-to-site (site-to-site), topologi-nya kek gini :

<p align="center">
  <img src="https://raw.githubusercontent.com/fauzanooor/blog_post/master/img/2020-03-21-VPN-ke-On-premise-via-Azure/topologi.png">
</p>

Seperti yang kita tahu P2S di Azure itu bisa sampai 1000 connections *tergantung sku ya, dan bisa dibaca - baca di sini <https://docs.microsoft.com/en-us/azure/vpn-gateway/point-to-site-about#gwsku/>, jadi seharusnya solusi ini akan men-solve masalah tersebut.

## Topology Describes
- End-user yang akan memakai VPN P2S dari Azure-nya itu memakai segment **192.168.150.0/24**
- Segment di VNET Azure itu memakai segment **172.168.0.0/16**
- Segment di sisi Head Office (on-premise) nya memakai segment **172.19.0.0/24**
- Connection flow nya seperti ini 

---
End-user <---> **VPN Point-to-Site** <---> Azure <---> **VPN Site-to-Site** <---> Head Office

---

Step 1 - Site-to-Site VPN
===

## Azure Site
1. Buat VPN Gateway, Local Network Gateway, dan juga Connection-nya. Untuk detail langkah - langkah nya mungkin bisa dilihat di dokumentasi Azure nya ini aja <https://docs.microsoft.com/en-us/azure/vpn-gateway/vpn-gateway-about-vpngateways/>, karena ngga ada konfig khusus buat VPN di Azure nya ini.
    - Tetapi pada tutorial disini, menggunakan konfig seperti ini ya :
    - VPN Gateway - SKU : **VpnGw1**
    - VPN Gateway - Type : **Route Based**
    - VPN Gateway - Generation : **Generation1**
    - Local Network Gateway - Address Space : **segment End-user dan segment HO** 

<p align="center">
  <img src="https://raw.githubusercontent.com/fauzanooor/blog_post/master/img/2020-03-21-VPN-ke-On-premise-via-Azure/vgw.png">
  <img src="https://raw.githubusercontent.com/fauzanooor/blog_post/master/img/2020-03-21-VPN-ke-On-premise-via-Azure/lng.png">
</p>

## On-premise Site (HO)
1. Mungkin untuk langkah - langkah konfig s2s vpn-nya tidak gue jelasin ya, karena kan setiap kantor itu punya router yang beda - beda juga, tapi untuk referensi-nya pokoke parameter-nya ngikutin yang di dokumentasi ini <https://docs.microsoft.com/en-us/azure/vpn-gateway/vpn-gateway-about-vpn-devices/>

2. Kalo udah, jangan lupa tambahin segment **Azure dan End-user** ya
   - Contoh pada tutor ini yaitu segment 172.168.0.0/16 (*Azure*) dan 192.168.150.0/24 (*End-user*)
   - Dan gambar di bawah ini contoh-nya jika menggunakan mikrotik

<p align="center">
  <img src="https://raw.githubusercontent.com/fauzanooor/blog_post/master/img/2020-03-21-VPN-ke-On-premise-via-Azure/ipsec-mikrotik.png">
</p>


Step 2 - Point-to-Site VPN
===
## Azure Site
1. Masuk ke menu VNG, kemudian klik menu **point-to-site configuration** > configure now

<p align="center">
  <img src="https://raw.githubusercontent.com/fauzanooor/blog_post/master/img/2020-03-21-VPN-ke-On-premise-via-Azure/p2s-configure.png">
</p>

{:start="2"}
2. Dan isi setiap form yang ada, sebagai contoh seperti di bawah ini :
   - Address pool (segment yang dipakai oleh End-user) : 192.168.150.0/24
   - Tunnel type : OpenVPN (SSL)
   - Authentication Type : Azure certificate
   - Root certificate - name : nama root certificate yang akan digunakan
   - Root certificate - public certificate data : public key dari root certificate-nya, untuk tutorial cara pembuatan certificate ini bisa mengikuti pada link ini : <https://docs.microsoft.com/en-us/azure/vpn-gateway/vpn-gateway-certificates-point-to-site#rootcert/>

<p align="center">
  <img src="https://raw.githubusercontent.com/fauzanooor/blog_post/master/img/2020-03-21-VPN-ke-On-premise-via-Azure/p2s-configured.png">
</p>

{:start="3"}
3. Kalau step 1 dan 2 sudah, bisa download P2S VPN Client yang dari Azure nya, dengan cara klik tombol **Download VPN Client** pada menu point-to-site configuration tersebut, dan nanti akan terdownload 1 file berformat **.zip**

<p align="center">
  <img src="https://raw.githubusercontent.com/fauzanooor/blog_post/master/img/2020-03-21-VPN-ke-On-premise-via-Azure/p2s-vpn-client.png">
</p>


## End-user Site
1. Jika menggunakan OpenVPN yang seperti pada tutorial disini, bisa download dahulu openvpn client-nya pada link ini <https://openvpn.net/community-downloads/>
2. Jika sudah, sebelum profile openvpn nya diimport, diharuskan mengedit profile openvpn yang sudah didownload pada step sebelumnya (file P2S yang **.zip**)
3. Download atau clone repo package ini https://github.com/RZomerman/AzureGWOpenVPN, dan jalankan dengan command seperti di bawah ini

```CreateOvpn.ps1 -PFXFile .\childcer.pfx -P2SZipFile '.\vgw-p2s-s2s.zip'```

> keterangan : 
> - childcer.pfx : Client/child certificate yang bisa diexport mengikuti tutorial pada link ini <https://docs.microsoft.com/en-us/azure/virtual-wan/certificates-point-to-site#clientexport/>
> - vgw-p2s-s2s.zip : nama file .zip yang telah didownload pada step sebelumnya

4. Jika sudah dijalankan command tersebut, maka akan menghasilkan 2 files, yang bernama *P2SOpenVPN.ovpn* dan vpnconfig.ovpn.
5. Import file *P2SOpenVPN.ovpn* tersebut dengan menggunakan aplikasi OpenVPN Client
6. Koneksikan OpenVPN-nya dengan profile openvpn yang sudah diimport, klik kanan pada icon openvpn yang berada di-toolbars, kemudian klik **connect**, dan tunggu hingga connected dan icon openvpn tersebut menjadi berwarna hijau

<p align="center">
  <img src="https://raw.githubusercontent.com/fauzanooor/blog_post/master/img/2020-03-21-VPN-ke-On-premise-via-Azure/openvpn-connection.png">
</p>

7. Jika sudah, kita lanjut ke step terakhir dan step yang paling penting. Yaitu menambahkan statik route dengan destination segment on-premise (HO) dan via gateway yang sama seperti yang digunakan dengan P2S-nya

- Cara check gateway nya itu bisa dengan ketik `route print` pada powershell/cmd, dan check gateway dari segment yang berasal dari P2S-nya 

<p align="center">
  <img src="https://raw.githubusercontent.com/fauzanooor/blog_post/master/img/2020-03-21-VPN-ke-On-premise-via-Azure/p2s-route-table.png">
</p>

- Dan untuk contoh command untuk menambahkan statik route-nya yaitu :
  - pada windows : ``route add 172.19.0.0 mask 255.255.255.0 192.168.150.1``


Kesimpulan
===
Jadi kesimpulannya itu sebenernya secara deployment nya sama persis dengan deployment VPN seperti biasanya, hanya saja pada koneksi s2s-nya di-*advertise* atau ditambahkan juga segment P2S-nya. Dan pada sisi end-user-nya diharuskan menambahkan statik route yang mengarah ke segment on-premise nya via gateway P2S-nya.


---

Seperti itu..