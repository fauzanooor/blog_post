---
layout: post
title:  "Menghitung RCU dan WCU untuk DynamoDB AWS"
date:   2020-02-25
---
Menghitung RCU dan WCU untuk DynamoDB AWS
====
Jadi pada DynamoDB di AWS itu, bakalan nge-charge untuk activity mulai dari reading, writing, dan juga penyimpanan datanya. Nha untuk cara nge-charges nya sendiri ada 2 metode yaitu on-demand dan provisioned :
- **on-demand** itu kek modelan pay-as-you-go, jadi bakalan otomatis nge-charges apa yang dipakai aja, dan juga jadi kita ngga perlu ngatur untuk kira - kira berapa banyak throughput read/write nya yang akan kita pakai nantinya.
	- Secara general, kelebihan pakai metode ini itu bisa diimplementasikan untuk workload yang masih baru/development karena kita kan belum tau untuk throughput yang akan dipakai rata - rata nya berapa, nha jadi dengan metode ini kita bisa monitoring dan review dahulu..
	- Kalau kekurangan-nya, siap - siap jadi bengkak billing-nya, kalau misalkan nantinya tiba2 ada activity read/write yang besar, karena jika ada throughput yang besar DynamoDB otomatis juga bakal menyesuaikan-nya jadi besar..

- **Provisioned**, kalau metode ini kebalikan-nya dari on-demand, yaitu kita perlu mengatur throughput read/write nya yang kira - kira akan dipakai berapa, jadi secara cost juga akan terkontrol
	- Secara general kelebihan pakai metode ini yaitu cost lebih terkontrol karena jika sudah ditentukan berapa throughput read/write yang akan dipakai, maka cost nya ngga akan lebih dari itu, bisa dibilang kekurangan-nya dari on-demand seperti yang di atas, bisa di-solve dengan metode provisioned ini.
	- Dan untuk kekurangan-nya, bisa dibilang juga kalau tiba - tiba ada activity read/write yang tinggi sedangkan sudah mencapai maximum capacity unit yang udah ditentuin sebelumnya, maka efek samping-nya aplikasi-nya bakal jadi lambat ataupun error. Tapi sebenernya kalau ada case kek gini palingan bisa langsung diupgrade maximum capacity units-nya, jadi menurutku sih ini kekurangan yang ngga terlalu harus dipermasalahkan sih, menurutmu gimana?

Oiya istilah disini untuk *throughput read/write* dengan *read/write capacity units* itu sama yak, yang mana untuk mendapatkan nilai (angka) dari hal tersebut adanya dilakukan perhitungan dahulu, tetapi sebelum dilakukan perhitungan, ada 2 istilah yang mungkin perlu diketahui dahulu yaitu :
- **Strong consistent**, dan 
- **eventual consistent**.

nah untuk istilah di atas itu dipakai untuk metode *replikasi/syncronize* pada database ya atau bisa dibilang DB yang sudah redundan. Dan berikut untuk penjelasan singkat-nya :
- **Stong Consistent (SC)**, yang berarti data yang berada di tiap nodes DB-nya itu akan selalu up-to-date (sama).
	- Misal ada activity write ke *node A*, maka activity write tersebut **akan langsung** di-write ke *node B*. Nah kan selama proses write ke node - node lain itu pasti sangat akan ketergantungan dengan latency antar-nodes, kenapa? karena kalau latency nya jelek (besar) maka proses write-nya pasti akan lambat dan akan berimpact negatif ke services yang terhubung dengan database-nya, dan juga yang berarti kalau ingin latency bagus (kecil) maka diperlukan-nya koneksi antar-nodes yang baik juga _(baca: pasti ada cost tambahan hehe)_

- **Eventual Consistent (EC)**, nah kalau yang ini itu data dari tiap nodes-nya itu tetap akan sama, tetapi pada proses penyamaan data dari tiap nodes tersebut akan ada delay/gap-nya.
	- Misal ada activity write ke _node A_, maka activity write tersebut __tidak akan langsung__ di-write ke _node B_, dan akan di-write ke node B-nya tergantung rentang waktu yang telah ditentukan. Oleh karena itu kalo pakai metode ini jadi ngga perlu latency yang bagus, yang jelek pun gapapa, yang penting data tiap nodes itu nantinya _(eventually)_ bakalan sama walaupun akan lebih memakan waktu lama + kemungkinan besar bakalan ada _stale data_.

Dan untuk penjelasan lebih detail + analogy yang menurutku cukup bagus, bisa dibaca pada artikel ini [https://hackernoon.com/eventual-vs-strong-consistency-in-distributed-databases-282fdad37cf7](https://hackernoon.com/eventual-vs-strong-consistency-in-distributed-databases-282fdad37cf7)

Lanjut ke perhitungan-nya, jadi secara perhitungan-nya itu dibagi jadi 2 ya, perhitungan untuk yang **Read** dan perhitungan untuk yang __Write__.

- __Read__, hal yang perlu diketahui dahulu :
	- 1 Read Capacity Unit (RCU) = 1 strong consistent read request
	- 1 Read Capacity Unit (RCU) = 2 eventual consistent read request
	- 2 Read Capacity Unit (RCU) = 1 transactional read request
	- Yang berarti per-_transactional_ read request membutuhkan 2 RCU
	- Ukuran pada per-satu item itu **maksimal 4 KB**
	- Jika ukuran 1 item lebih dari 4KB maka akan dibulatkan ke-atas sesuai perkelipatan 4
		- **Misal (1)** : jika ukuran item yang akan di-read itu 5, maka akan dibulatkan ke atas menjadi 8 (kelipatan 4), dan jadi ada 2 RCU
		- **Misal (2)** : jika ukuran item yang akan di-read itu 11, maka akan dibulatkan ke atas menjadi 12 (kelipatan 4), dan jadi ada 3 RCU
	- Pada eventual consistent, jika nilai RCU dari EC nya mempunyai koma (misal kek 3.5, 4.1, 5.9) itu dibulatkan ke atas juga (misal jadi 4, 5, 6).
	- Rumus :
		- Strong consistent (SC) :
			- yang perlu diketahui dahulu yaitu :
				- Ukuran per-1 item/data nya dan sudah dibulatkan ke atas (misal 5KB menjadi 8KB) = **``A``**
				- Jumlah item/data nya (misal 3) = **``B``**
			- Rumus :
				- **``A / 4 x B``**
			- Misal **(1)** :
				- Diketahui : A = 8 KB dan B = 3
				- Ditanya : RCU of Strong Consistent ?
				- Jawab : 
					- **``A / 4 x B``**
					- **``8 / 4 x 3 = 6 RCU``**
			- Misal **(2)** :
				- Diketahui : A = 36 KB dan B = 1
				- Ditanya : RCU of Strong Consistent ?
				- Jawab : 
					- **``A / 4 x B``**
					- **``36 / 4 x 1 = 9 RCU``**
				
		- Eventual consistent (EC) :
			- yang perlu diketahui dahulu :
				- Read Capacity Unit (RCU) (misal 6) = **``C``**
				- Jadi untuk mencari EC diharuskan menghitung RCU nya dari strong consistent dahulu, karena EC ini itu mempunyai nilai 1/2 (setengah) dari SC
			- Rumus :
				- **``C / 2``**
			- Misal **(1)** :
				- Diketahui : C = 6
				- Ditanya : RCU of Eventual Consistent ?
				- Jawab : 
					- **``C / 2``**, dan dibulatkan ke atas
					- **``6 / 2 = 3 RCU``**
			- Misal **(2)** :
				- Diketahui : C = 9
				- Ditanya : RCU of Eventual Consistent ?
				- Jawab : 
					- **``C / 2``**, dan dibulatkan ke atas
					- **``9 / 2 = 4.5 = 5 RCU``**

-   Write, hal yang perlu diketahui dahulu :
	- 1 Write Capacity Unit (WCU) = minimal 1 KB per-item
	- 2 Write Capacity Unit (WCU) = 1 Transactional Write Request
	- Yang berarti "per-transactional write request" membutuhkan 2 WCU
	- Jika ukuran item terdapat koma (misal kek 3.1, 4.9, 5.5) itu dibulatkan ke atas juga (misal jadi 4, 5, 6)
	- Karena ukuran WCU itu minimal 1 KB, jadi kalo misalkan ada ukuran yang di bawah itu, maka dibulatkan ke atas jadi 1 KB.
		- Misal, ukuran item-nya 100 Bytes, maka jadi dibulatkan jadi 1 KB
	- Nah misalkan item-nya ini ada banyak, dengan contoh kek gini :
		- Contoh :
			- Item A : 2.5 KB
			- Item B : 500 B
			- Maka cara menghitung-nya itu dibulatkan dahulu ke-atas sebelum dijumlahkan tiap items-nya. Jadi seperti ini :
				- **``Item A : 2.5 KB = 3 KB``**
				- **``Item B : 500 B = 1 KB``**
				- **``WCU = 3 KB + 1 KB = 4 KB``**

---

Seperti itu..