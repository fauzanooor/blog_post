---
layout: post
title:  "Pindah Subscription Azure antar Tenant"
date:   2020-03-09
---
Sebelum bahas ke topik utamanya tentang mindahin resources-nya, gue coba jelasin dahulu hirarki subscription dan tenant di Azure itu apa yak, biar sama dulu persepsi kita hehe

Tenant
===
- Kalo berdasarkan tingkatan, ini yang paling utama (bawah), dan dibuat-nya itu yang paling pertama
- *Tenant* ini punya nama lain yaitu *organization* dan juga *directory*
- Tenant ini berkaitannya dengan AAD (Azure Active Directory)
- Tenant sendiri itu ada 2 bentuk, yaitu :
  - Tenant ID (Contoh : **12321001-456e-0987-098c-abcd05babcdef**)
  - Tenant name (Contoh : **kangbuburtimor.onmicrosoft.com**)
- Product Cloud milik Microsoft, seperti Office 365 dan Azure itu pasti mempunyai tenant 
- Setiap subscription itu pasti ada tenant-nya, tetapi setiap tenant tidak tentu ada subscriptionnya
  - Yang berarti untuk memakai subscription, diharuskan membuat tenant dahulu
- 1 Tenant itu bisa pake multiple subscriptions

Subscription
===
- Kalo berdasarkan tingkatan, ini yang satu tingkat persis dari tenant, dan dapat dipakai setelah tenant terbuat
- Kalo cuma ada subscription doang, tapi ngga ada tenant-nya, maka subscription nya ngga bisa dipake
- Tujuan-nya ada subscription itu agar bisa memakai resources atau fitur - fitur dari yang disediakan oleh Azure nya
  -  Misal, mau buat Virtual Machine, maka harus punya subscription juga. Kalo cuma punya tenant aja, yaa ngga bisa..
- Sama seperti tenant, Subscription juga ada 2 bentuk, yaitu :
  - Subscription ID (Contoh : **1a2345b1-234c-561d-234c-a1234b5612c3**)
  - Subscription name (Contoh : **Visual Studio Enterprise - MPN**)
- 1 Subscription itu ngga bisa dipake di-multiple tenant
- Dan satu lagi, Subscription itu juga punya offer, yang mana offer ini adalah tipe dari subscription yang dipake. 
  - Misal, offer dengan tipe Pay-As-You-Go (0003P), Visual Studio Enterprise - MPN (0029P), Azure in CSP (0145P) itu tiap offer-nya punya masing - masing terms & benefit yang berbeda sesuai dengan tujuan dari offer itu ditujukan.
  - Terus, list tipe offer buat Azure itu apa aja sih? Bisa check disini <https://azure.microsoft.com/en-us/support/legal/offer-details/>


Analogi
===
Jadi kalo di-analogikan, Azure itu seperti Hotel yang mana di dalam hotel itu kan ada lobby dan kamar hotel beserta kunci kamar-nya.

Lobby ini kita anggap sebagai **tenant**, yang mana misalkan orang (user) kalo cuma masuk ke lobby doang itu kan gratis, tapi ngga bakal bisa menikmati fasilitas dari hotel tersebut.

Kamar hotel ini kita anggap sebagai **resources yang disediakan oleh Azure**-nya, karena kan di dalam kamar hotel tersebut ada fasilitas seperti kasur, TV, dan lain - lain, yang mana fasilitas tersebut bisa kita samakan seperti VMs, Load Balancer, VNet, dll.

Nha, sedangkan untuk dapat menikmati fasilitas kamar hotel tersebut (azure resources), kita harus sewa alias bayar dulu kan ya, dan kalo udah bayar baru kita dapat kunci kamar hotel-nya. Nha kunci kamar hotel ini yang bisa kita analogikan sebagai **subscription**, yang mana dengan memiliki kunci (subscription) ini, kita bisa memakai fasilitas - fasilitas yang disediakan oleh Hotel tersebut.

![](https://raw.githubusercontent.com/fauzanooor/blog_post/master/img/2020-03-09-Pindah-Subscription-Azure-antar-Tenant/analogy.png "Analogy")

Sampai sini, sudah paham bedanya tenant dan subscription di Azure kan? Nha ada lagi requirement nya nih, yaitu 

Requirement
===
1. Subscription yang akan dipindahkan
2. User yang mempunyai role **owner**, di level subscription nya
3. User tersebut, juga mempunyai akses ke-2 tenant-nya (tenant lama dan yg baru)

Langkah - Langkah
===

### Langkah 1 - **Assessment**

![](https://raw.githubusercontent.com/fauzanooor/blog_post/master/img/2020-03-09-Pindah-Subscription-Azure-antar-Tenant/step-01.png "Langkah 1")

- Tenant lama : (kiri) cakwejkt.onmicrosoft.com
  - Subscription : *subs_buburtimor*
  - User : *kang@cakwejkt.onmicrosoft.com*
- Tenant baru : (kanan) buburtimor
  - Subscription : (ngga punya)
  - User : *kang@buburtimor.xyz*

### Langkah 2 - **Inviting User**
![](https://raw.githubusercontent.com/fauzanooor/blog_post/master/img/2020-03-09-Pindah-Subscription-Azure-antar-Tenant/step-02.png "Langkah 2")
1. Pada tenant lama (cakwejkt.onmicrosoft.com), masuk ke menu subscription > pilih subscription-nya > Access Control (IAM) > Add > Add role assignment
   - Role : Owner
   - Assign access to : Azure AD user, group, or service principal
   - Select : (isi akun yang akan diinvite), dalam tutor ini *kang@buburtimor.xyz*
   - Save
  
![](https://raw.githubusercontent.com/fauzanooor/blog_post/master/img/2020-03-09-Pindah-Subscription-Azure-antar-Tenant/step-01-flow.png)

2. Nanti kalo sudah, akan muncul notif tentang link invitation-nya, dan juga akan dikirimkan email tentang link invitation-nya

![](https://raw.githubusercontent.com/fauzanooor/blog_post/master/img/2020-03-09-Pindah-Subscription-Azure-antar-Tenant/step-01-notif.png)

atau

![](https://raw.githubusercontent.com/fauzanooor/blog_post/master/img/2020-03-09-Pindah-Subscription-Azure-antar-Tenant/step-01-notif-email.png)

3. Klik invitation link tersebut, dan pilih tombol **accept**

![](https://raw.githubusercontent.com/fauzanooor/blog_post/master/img/2020-03-09-Pindah-Subscription-Azure-antar-Tenant/step-01-notif-web.png)


### Langkah 3 - **Verifikasi Subscription**
![](https://raw.githubusercontent.com/fauzanooor/blog_post/master/img/2020-03-09-Pindah-Subscription-Azure-antar-Tenant/step-01.png)

Verifikasi kalo user kang@buburtimor.xyz udah punya subscription **subs_buburtimor** di tenant *cakwejkt.onmicrosoft.com*

![](https://raw.githubusercontent.com/fauzanooor/blog_post/master/img/2020-03-09-Pindah-Subscription-Azure-antar-Tenant/step-01-verify.png)


### Langkah 4 - **Pindahin Subscription ke Tenant lain**
![](https://raw.githubusercontent.com/fauzanooor/blog_post/master/img/2020-03-09-Pindah-Subscription-Azure-antar-Tenant/step-04.png)

1. Masih pada tenant *cakwejkt.onmicrosoft.com*, pergi ke menu subscription > pilih subscription nya > change directory 

![](https://raw.githubusercontent.com/fauzanooor/blog_post/master/img/2020-03-09-Pindah-Subscription-Azure-antar-Tenant/step-04-change-dir.png)

2. Pilih nama tenant baru-nya (*buburtimor.xyz*), dan save

![alt](https://raw.githubusercontent.com/fauzanooor/blog_post/master/img/2020-03-09-Pindah-Subscription-Azure-antar-Tenant/step-04-change-dir-lg.png)

3. Nanti akan muncul notif bahwa subscription *subs_buburtimor* telah dipindahkan ke tenant yang baru, namun untuk proses pemindahan ini butuh waktu sekitar 30 menit

![alt](https://raw.githubusercontent.com/fauzanooor/blog_post/master/img/2020-03-09-Pindah-Subscription-Azure-antar-Tenant/step-04-change-dir-notif.png)


### Langkah 5 - **Verifikasi Subscription telah pindah ke Tenant Baru**
![alt](https://raw.githubusercontent.com/fauzanooor/blog_post/master/img/2020-03-09-Pindah-Subscription-Azure-antar-Tenant/step-05.png)

1. Pindah ke tenant baru (buburtimor.xyz), dengan cara klik tombol directory yang disisi kanan atas > Dan pilih directory *buburtimor.xyz*

![alt](https://raw.githubusercontent.com/fauzanooor/blog_post/master/img/2020-03-09-Pindah-Subscription-Azure-antar-Tenant/step-05-change-dir.png)

2. Dan jika kita masuk ke menu subscription-nya, maka pada tenant *buburtimor.xyz* ini sudah ada subscription *subs_buburtimor* nya

![alt](https://raw.githubusercontent.com/fauzanooor/blog_post/master/img/2020-03-09-Pindah-Subscription-Azure-antar-Tenant/step-05-yes-subs.png)

3. Dan juga jika kita check subscription pada tenant lama (*cakwejkt.onmicrosoft.com*) sudah tidak mempunyai subscription apapun

![alt](https://raw.githubusercontent.com/fauzanooor/blog_post/master/img/2020-03-09-Pindah-Subscription-Azure-antar-Tenant/step-05-no-subs.png)
