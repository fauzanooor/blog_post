---
layout: post
title: "ArangoDB Clustering"
date: 2020-08-28
---

Penjelasan Singkat
===

**ArangoDB** ini merupakan suatu database server yang berjenis NoSQL

**ArangoDB** ini juga support untuk High Availability dan Scalable. Dan untuk mendapatkan HA dan Scalable tersebut, ArangoDB ini diharuskan untuk disetup menjadi cluster, dan cluster dari ArangoDB ini mempunyai 3 komponen utama, yaitu :

1. **Agents (Agency)**, untuk fungsi utama dari komponen ini untuk menyimpan seluruh konfigurasi, dan melakukan sinkronisasi pada seluruh services yang ada di dalam cluster-nya.
   
2. **Coordinator**, untuk fungsi utama dari komponen ini untuk menyediakan endpoint yang dapat diakses dari client atau luar, mengeksekusi queries nya, dan juga pada komponen ini tau pada DB-Server mana saja data - data-nya disimpan, yang berarti juga kurang lebih seperti load balancer-nya. Oiya, komponen ini stateless, yang berarti boleh dimatiin (shutdown) atau restart kapan aja dibutuhin (yang penting ada 1 server cadangan yang buat backup nya ya, biar redudancy)
   
3. **DB-Server**, kalo fungsi komponen ini untuk menyimpan data dari database itu sendiri.

<p align="center">
  <img src="https://www.arangodb.com/docs/stable/images/cluster_topology.png">
</p>


Contoh Implementasi
===

## Informasi Topologi
- Pada contoh implementasi ini, menggunakan 3 nodes :
  - 1 node untuk komponen Agency dan Coordinator (hostname : arango1.db.bbrtmr)
    - Agency :
      - Endpoint address : 192.168.0.225:8531
    - Coordinator :
      - Endpoint address : 192.168.0.225:8529
  - 2 nodes untuk komponen DB-server
      - Endpoint address – db1 : 192.168.0.226:8530 (hostname : arango2.db.bbrtmr)
      - Endpoint address – db2 : 192.168.0.227:8530 (hostname : arango3.db.bbrtmr)
- OS untuk seluruh nodes untuk Ubuntu 18.04 LTS
- Spesifikasi untuk seluruh nodes, yaitu :
  - Compute : 2 vCores, 4 GB RAM
  - Storage : 40 GB, SSD
- Public access : 
  - IP Public atau IP yang dapat diakses oleh _enduser_ hanya untuk server yang mem-provide komponen **Coordinator** saja, yang lainnya tidak perlu. Jadi pada contoh implementasi disini IP Public nya hanya ditempel di server arango1.db.bbrtmr

<p align="center">
  <img src="https://raw.githubusercontent.com/fauzanooor/blog_post/master/img/2020-08-28-ArangoDB-Clustering/topology_1.png">
</p>

> :warning:
> Btw arsitektur dari topologi ini cuma buat testing aja ya, soale secara agency dan coordinator komponen nya itu ngga redundan, dan autentikasi pada konfig di bawah ini itu ngga dipake.

## Config
### Download & Installasi ArangoDB

Lakukan installasi arangoDB dengan commands di bawah ini, pada seluruh nodes
```
# Buat direktori arangodb
mkdir /opt/arangodb; cd /opt/arangodb

# Download key repository dari ArangoDB nya
curl -OL https://download.arangodb.com/arangodb37/DEBIAN/Release.key
sudo apt-key add - < Release.key

# Tambah arangodb repository dan install arangodb dan screen
echo 'deb https://download.arangodb.com/arangodb37/DEBIAN/ /' | sudo tee /etc/apt/sources.list.d/arangodb.list
sudo apt-get install apt-transport-https
sudo apt-get update
sudo apt-get install arangodb3=3.7.2-1 screen

# Pastikan service arangodb nya yang untuk standalone mati, karena secara default setelah installasi arangodb selesai, service nya sudah nyala
sudo systemctl stop arangodb3
sudo systemctl disable arangodb3
```

### Setup Agency + Coordinator
Masuk ke server Agency (192.168.0.225 – arango1.db.bbrtmr), kemudian jalankan comands di bawah ini :
```
# masuk ke direktori /opt/arangodb, dan buat direktori untuk menyimpan data dari komponen Agency + Coordinator server
cd /opt/arangodb
mkdir agent coordinator

# Ubah permission dan ownership
chmod +x agent; chown arangodb:arangodb agent
chmod +x coordinator; chown arangodb:arangodb coordinator

# Buat session screen baru untuk arangodb agency
screen -R arangodb-agency

# Setup komponen Agency
arangod --server.endpoint tcp://0.0.0.0:8531 \
  --agency.my-address tcp://192.168.0.225:8531 \
  --server.authentication false \
  --agency.activate true \
  --agency.size 1 \
  --agency.supervision true \
  --database.directory agent 

# Detach dari screen-nya dengan cara tekan tombol ctrl+A kemudian D

# Buat session screen baru untuk arangodb coordinator
screen -R arangodb-coordinator

# Setup komponen Coordinator
arangod --server.authentication=false \
  --server.endpoint tcp://0.0.0.0:8529 \
  --cluster.my-address tcp://192.168.0.225:8529 \
  --cluster.my-role COORDINATOR \
  --cluster.agency-endpoint tcp://192.168.0.225:8531 \
  --database.directory coordinator &

# Detach dari screen-nya dengan cara tekan tombol ctrl+A kemudian D
```

### Setup Komponen DB-Server
Masuk ke server DB yang pertama (192.168.0.226 – arango2.db.bbrtmr), kemudian jalankan comands di bawah ini :
```
# masuk ke direktori /opt/arangodb, dan buat direktori untuk menyimpan data dari komponen DB-Server nya
cd /opt/arangodb
mkdir dbserver

# Ubah permission dan ownership
chmod +x dbserver; chown arangodb:arangodb dbserver

# Buat session screen baru untuk arangodb DB-Server
screen -R arangodb-dbserver

# Setup komponen DB-Server
arangod --server.authentication=false \
  --server.endpoint tcp://0.0.0.0:8530 \
  --cluster.my-address tcp://192.168.0.226:8530 \
  --cluster.my-role DBSERVER \
  --cluster.agency-endpoint tcp://192.168.0.225:8531 \
  --database.directory dbserver &

# Detach dari screen-nya dengan cara tekan tombol ctrl+A kemudian D
```

Masuk ke server DB yang kedua (192.168.0.227 – arango3.db.bbrtmr), kemudian jalankan comands di bawah ini :
```
# masuk ke direktori /opt/arangodb, dan buat direktori untuk menyimpan data dari komponen DB-Server nya
cd /opt/arangodb
mkdir dbserver

# Ubah permission dan ownership
chmod +x dbserver; chown arangodb:arangodb dbserver

# Buat session screen baru untuk arangodb DB-Server
screen -R arangodb-dbserver

# Setup komponen DB-Server
arangod --server.authentication=false \
  --server.endpoint tcp://0.0.0.0:8530 \
  --cluster.my-address tcp://192.168.0.227:8530 \
  --cluster.my-role DBSERVER \
  --cluster.agency-endpoint tcp://192.168.0.225:8531 \
  --database.directory dbserver &
```

### Finalisasi
Jika sudah selesai semua, maka ArangoDB sudah dapat diakses via web browser dengan port 8529, dan berikut ini contoh screenshots-nya

<p align="center">
  <img src="https://raw.githubusercontent.com/fauzanooor/blog_post/master/img/2020-08-28-ArangoDB-Clustering/dashboard_1.png">
  <img src="https://raw.githubusercontent.com/fauzanooor/blog_post/master/img/2020-08-28-ArangoDB-Clustering/dashboard_2.png">
</p>
