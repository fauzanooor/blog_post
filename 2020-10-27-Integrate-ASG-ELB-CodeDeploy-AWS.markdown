---
layout: post
title: "Integrate ASG + ELB + CodeDeploy - AWS"
date: 2020-10-27
---

**\[Integrating\] AWS ELB + ASG + CodeDeploy**
==============================================

Persiapan
---------

### **Topology**

![](https://raw.githubusercontent.com/fauzanooor/blog_post/draft/img/2020-10-27-Integrate-ASG-ELB-CodeDeploy-AWS/topology.png)

### **VPC**

-   VPC Name : Bangau-VPC

-   VPC CIDR : 10.0.0.0/16

-   Region : Singapore

-   Subnets :

    -   Subnet 1 – Name : bangau-public-a

        -   Availability Zone : A

        -   CIDR : 10.0.0.0/24

        -   Keterangan : Untuk public subnet

    -   Subnet 2 – Name : bangau-public-b

        -   Availability Zone : B

        -   CIDR : 10.0.10.0/24

        -   Keterangan : Untuk public subnet

    -   Subnet 3 – Name : bangau-private-a

        -   Availability Zone : A

        -   CIDR : 10.0.1.0/24

        -   Keterangan : Untuk private subnet

    -   Subnet 4 – Name : bangau-private-b

        -   Availability Zone : B

        -   CIDR : 10.0.2.0/24

        -   Keterangan : Untuk private subnet

-   Route Table

  RouteTable Name        SubnetAssociate        Destination   Target
  ---------------------- ------------------ --- ------------- --------------
  bangau-rtb-private-a   bangau-private-a       10.0.0.0/16   local
                                                0.0.0.0/0     bangau-ngw-1
  bangau-rtb-private-b   bangau-private-b       10.0.0.0/16   Local
                                                0.0.0.0/0     bangau-ngw-2
  bangau-rtb-public      bangau-public-a        10.0.0.0/16   Local
                         bangau-public-b        0.0.0.0/0     bangau-igw

-   Internet gateway

    -   Name : bangau-igw

    -   Attached to : bangau-VPC

-   Nat Gateway

    -   NatGateway 1 – Name : bangau-ngw-1

        -   Attached to : bangau-public-a

    -   NatGateway 2 – Name : bangau-ngw-2

        -   Attached to : bangau-public-b

### **VPC – Screenshots**

![](https://raw.githubusercontent.com/fauzanooor/blog_post/draft/img/2020-10-27-Integrate-ASG-ELB-CodeDeploy-AWS/vpc1.png)

![](https://raw.githubusercontent.com/fauzanooor/blog_post/draft/img/2020-10-27-Integrate-ASG-ELB-CodeDeploy-AWS/vpc2.png)

![](https://raw.githubusercontent.com/fauzanooor/blog_post/draft/img/2020-10-27-Integrate-ASG-ELB-CodeDeploy-AWS/vpc3.png)

![](https://raw.githubusercontent.com/fauzanooor/blog_post/draft/img/2020-10-27-Integrate-ASG-ELB-CodeDeploy-AWS/vpc4.png)

![](https://raw.githubusercontent.com/fauzanooor/blog_post/draft/img/2020-10-27-Integrate-ASG-ELB-CodeDeploy-AWS/vpc5.png)

### **Security Groups**

Melakukan pembuatan security group untuk instances dan ALB nya, dengan
detail sebagai berikut :

-   Security group name : bangau-sg-ec2

  **Rules Type**   **Type**      **Protocol**   **Port range**   **Source**
  ---------------- ------------- -------------- ---------------- ------------
  Inbound          HTTP          TCP            80               0.0.0.0/0
                   HTTPS         TCP            443              0.0.0.0/0
                   SSH           TCP            22               0.0.0.0/0
  Outbound         All traffic   All            All              0.0.0.0/0

-   Security group name : bangau-sg-alb

  **Rules Type**   **Type**      **Protocol**   **Port range**   **Source**
  ---------------- ------------- -------------- ---------------- ------------
  Inbound          HTTP          TCP            80               0.0.0.0/0
                   HTTPS         TCP            443              0.0.0.0/0
  Outbound         All traffic   All            All              0.0.0.0/0

### **Security Groups – Screenshot**

![](https://raw.githubusercontent.com/fauzanooor/blog_post/draft/img/2020-10-27-Integrate-ASG-ELB-CodeDeploy-AWS/sg1.png)

### **EC2 Sebagai Golden Image / Instance**

Golden image atau instance ini dibuat untuk tujuan kalau nantinya ada
perubahan konfigurasi pada instances yang telah diassigned pada ASG,
serta instance ini juga bisa digunakan sebagai jumphost untuk koneksi ke
instance yang tidak mempunyai IP Public. Dan berikut ini detail dari
instance nya :

-   Instance name : bangau-golden

-   Region : Singapore

-   Public IP : 18.136.187.242

-   Private IP : 10.0.0.117

-   Subnet : bangau-public-a (10.0.0.0/24)

-   Specs

    -   Instance type : t2.micro

    -   Storage : 8G, gp2 (EBS)

-   AMI :

    -   ID : ami-015a6758451df3cb9

    -   Name : Amazon Linux 2 (x64)

-   Service/Application Installed

    -   Nginx with default config, but for the default web page is
        changed

    -   SSH

    -   codedeploy-agent (detail installasi akan dijelaskan di bawah)

-   IAM Role : none

-   Security group : bangau-sg-ec2

### **EC2 Sebagai Golden Image / Instance – Screenshots**

![](https://raw.githubusercontent.com/fauzanooor/blog_post/draft/img/2020-10-27-Integrate-ASG-ELB-CodeDeploy-AWS/ec2golden1.png)

![](https://raw.githubusercontent.com/fauzanooor/blog_post/draft/img/2020-10-27-Integrate-ASG-ELB-CodeDeploy-AWS/ec2golden2.png)

![](https://raw.githubusercontent.com/fauzanooor/blog_post/draft/img/2020-10-27-Integrate-ASG-ELB-CodeDeploy-AWS/ec2golden3.png)

![](https://raw.githubusercontent.com/fauzanooor/blog_post/draft/img/2020-10-27-Integrate-ASG-ELB-CodeDeploy-AWS/ec2golden4.png)

![](https://raw.githubusercontent.com/fauzanooor/blog_post/draft/img/2020-10-27-Integrate-ASG-ELB-CodeDeploy-AWS/ec2golden5.png)

![](https://raw.githubusercontent.com/fauzanooor/blog_post/draft/img/2020-10-27-Integrate-ASG-ELB-CodeDeploy-AWS/ec2golden6.png)

### **Pembuatan IAM Roles**

IAM Roles yang dibuat disini untuk tujuan mengintegrasikan CodeDeploy
nya dengan ASG-nya. Untuk hal tersebut diperlukan 2 roles, yang satu
untuk EC2 dan yang satu lagi untuk CodeDeploy. Dan berikut ini detail
konfigurasi nya :

-   Role name : bangau-ec2-codedeploy

    -   Trusted entity : EC2

    -   Permission : AmazonEC2RoleforAWSCodeDeploy

-   Role name : CodeDeployServiceRole

    -   Trusted entity : CodeDeploy

    -   Permission : AWSCodeDeployRole

![](https://raw.githubusercontent.com/fauzanooor/blog_post/draft/img/2020-10-27-Integrate-ASG-ELB-CodeDeploy-AWS/iam1.png)

### **Initial Configuration of Golden Instance for Golden Image**

1.  Masuk ke SSH pada instance bangau-golden, kemudian install nginx

```
$ sudo amazon-linux-extras install nginx1 -y
$ sudo cd /usr/share/nginx/html/
$ sudo rm -rf \*
$ sudo vi index.html
```

```
<html>
    <head>
        <meta charset="utf-8">
        <title>Sample Deployment</title>
        <style>
            body {
                color: #ffffff;
                background-color: lime;
                font-family: Arial, sans-serif;
                font-size: 14px;
                }
            h1 {
                font-size: 500%;
                font-weight: normal;
                margin-bottom: 0;
                }
            h2 {
                font-size: 200%;
                font-weight: normal;
                margin-bottom: 0;
                }
       </style>
    </head>
    <body>
        <div align="center">
            <h1>Congratulations</h1>
            <h2>This application was deployed using AWS CodeDeploy.</h2>
            <p>For next steps, read the <a href="http://aws.amazon.com/documentation/codedeploy">AWS CodeDeploy Documentation</a>.</p>
        </div>
    </body>
</html>
```
```
$ sudo systemctl restart nginx
$ sudo systemctl enable nginx
```

2.  Install agent CodeDeploy
```
$ sudo su
# yum install ruby -y
# wget <https://aws-codedeploy-ap-southeast-1.s3.ap-southeast-1.amazonaws.com/latest/install>
# chmod +x install
# ./install auto
# systemctl start codedeploy-agent
# systemctl enable codedeploy-agent
# systemctl status codedeploy-agent
```

Capture Golden Instance as a AMI Golden Image
---------------------------------------------

1.  Masuk ke service EC2, pilih instance **bangau-golden** > klik
    kanan > image > create image

![](https://raw.githubusercontent.com/fauzanooor/blog_post/draft/img/2020-10-27-Integrate-ASG-ELB-CodeDeploy-AWS/ami1.png)

2.  Isi nama image-nya (bisa pakai format ini
    {nama-instance-ddmmyyy-hhmm}, kemudian centang **enable** no reboot
    agar ketika pembuatan image ini tidak me-restart instance yang
    sedang akan dibuat image-nya, dan klik tombol **create image**

![](https://raw.githubusercontent.com/fauzanooor/blog_post/draft/img/2020-10-27-Integrate-ASG-ELB-CodeDeploy-AWS/ami2.png)

3.  Masuk ke menu **AMIs** yang ada di sisi kiri, kemudian tunggu hingga
    image yang baru dibuat selesai (state pending -> available)

![](https://raw.githubusercontent.com/fauzanooor/blog_post/draft/img/2020-10-27-Integrate-ASG-ELB-CodeDeploy-AWS/ami3.png)

![](https://raw.githubusercontent.com/fauzanooor/blog_post/draft/img/2020-10-27-Integrate-ASG-ELB-CodeDeploy-AWS/ami4.png)

Pembuatan Elastic Load Balancer
-------------------------------

### **Pembuatan Target Groups**

1.  Pergi ke services EC2 > Load Balancing > Target Groups, dan
    klik tombol **Create Target Group**

![](https://raw.githubusercontent.com/fauzanooor/blog_post/draft/img/2020-10-27-Integrate-ASG-ELB-CodeDeploy-AWS/elb1.png)

1.  Dan berikut ini detail konfigurasi dari Target Group nya :

    a.  Target type : Instances

    b.  Target group name : bangau-tg

    c.  Protocol : HTTP

    d.  Port : 80

    e.  VPC : bangau-vpc

    f.  Health check protocol : HTTP

    g.  Health check path : /

![](https://raw.githubusercontent.com/fauzanooor/blog_post/draft/img/2020-10-27-Integrate-ASG-ELB-CodeDeploy-AWS/elb2.png)

![](https://raw.githubusercontent.com/fauzanooor/blog_post/draft/img/2020-10-27-Integrate-ASG-ELB-CodeDeploy-AWS/elb3.png)

1.  Dan jika target group sudah selesai dibuat, maka akan muncul seperti
    ini

![](https://raw.githubusercontent.com/fauzanooor/blog_post/draft/img/2020-10-27-Integrate-ASG-ELB-CodeDeploy-AWS/elb4.png)

### **Pembuatan Application Load Balancer**

1.  Pergi ke services EC2 > Load Balancing > Load Balancers, dan
    klik tombol **Create Load Balancer**

![](https://raw.githubusercontent.com/fauzanooor/blog_post/draft/img/2020-10-27-Integrate-ASG-ELB-CodeDeploy-AWS/elb5.png)

1.  Dan berikut ini detail konfigurasi load balancer nya :

    a.  Load balancer type : Application Load Balancer

    b.  Scheme : internet-facing

    c.  IP address type : ipv4

    d.  Listener
    - Load balancer protocol : HTTP
    - Load balancer port : 80
  
    e.  VPC : bangau-vpc

    f.  Availability zones : ap-southeast-1a : bangau-public-a

    g.  Availability zones : ap-southeast-1b : bangau-public-b

![](https://raw.githubusercontent.com/fauzanooor/blog_post/draft/img/2020-10-27-Integrate-ASG-ELB-CodeDeploy-AWS/elb6.png)

    h.  Security group : bangau-sg-lb
    
    i.  Target group : bangau-tg

Dan jika sudah selesai dibuat, maka akan menjadi seperti screenshot di bawah ini

![](https://raw.githubusercontent.com/fauzanooor/blog_post/draft/img/2020-10-27-Integrate-ASG-ELB-CodeDeploy-AWS/elb7.png)

![](https://raw.githubusercontent.com/fauzanooor/blog_post/draft/img/2020-10-27-Integrate-ASG-ELB-CodeDeploy-AWS/elb8.png)

![](https://raw.githubusercontent.com/fauzanooor/blog_post/draft/img/2020-10-27-Integrate-ASG-ELB-CodeDeploy-AWS/elb9.png)

Pembuatan Auto Scaling Group
----------------------------

### **Pembuatan Launch Configurations**

Sebenarnya bisa juga menggunakan *Launch Template* dan bahkan lebih baik
pakai Launch Template ini dibandingkan dengan Launch Configurations,
namun pada tutorial ini memakai Launch Configuration saja, biar lebih
cepet hehe.

1.  Masuk ke services EC2 > Auto Scaling > Launch Configurations,
    kemudian klik tombol **Create launch configurations**

![](https://raw.githubusercontent.com/fauzanooor/blog_post/draft/img/2020-10-27-Integrate-ASG-ELB-CodeDeploy-AWS/asg1.png)

1.  Dan berikut ini untuk detail konfigurasi launch configurations nya :

    -   Name : bangau-lc

    -   AMI : bangau-golden-26102020-1156 *(AMI yang berasal dari
        capture golden instance pada langkah sebelumnya)*

    -   Instance type : t2.micro

    -   IAM instance profile : bangau-ec2-codedeploy

    -   Security group : existing : bangau-sg-ec2

    -   Key pair : *(menggunakan keypair yang sama seperti golde
        instance)*

![](https://raw.githubusercontent.com/fauzanooor/blog_post/draft/img/2020-10-27-Integrate-ASG-ELB-CodeDeploy-AWS/asg2.png)

![](https://raw.githubusercontent.com/fauzanooor/blog_post/draft/img/2020-10-27-Integrate-ASG-ELB-CodeDeploy-AWS/asg3.png)

![](https://raw.githubusercontent.com/fauzanooor/blog_post/draft/img/2020-10-27-Integrate-ASG-ELB-CodeDeploy-AWS/asg4.png)

Dan jika sudah selesai, maka akan menjadi seperti ini

![](https://raw.githubusercontent.com/fauzanooor/blog_post/draft/img/2020-10-27-Integrate-ASG-ELB-CodeDeploy-AWS/asg5.png)

### **Pembuatan Auto Scaling Groups**

Setelah launch configuration telah dibuat, maka ASG juga sudah dapat
dibuat.

1.  Masuk ke services EC2 > Auto Scaling > Launch Configurations,
    kemudian klik tombol **Create Auto Scaling Group**

![](https://raw.githubusercontent.com/fauzanooor/blog_post/draft/img/2020-10-27-Integrate-ASG-ELB-CodeDeploy-AWS/asg6.png)

1.  Dan berikut ini detail konfigurasi ASG nya :

    -   AutoScalingGroup name : bangau-asg

    -   (*Switch to Launch Configuration*) Launch configuration :
        banga-lc-1

    -   Network

        -   VPC : bangau-vpc

        -   Subnets : bangau-private-a, bangau-private-b

        -   Load balancing : enabled : bangau-tg

        -   Group Size

            -   Desired : 1

            -   Minimum : 1

            -   Maximum : 3

        -   Scaling policies : Target tracking scaling policy

            -   Metric type : Average CPU utilization

            -   Target value : 50%

            -   Instances need : (default) 300

![](https://raw.githubusercontent.com/fauzanooor/blog_post/draft/img/2020-10-27-Integrate-ASG-ELB-CodeDeploy-AWS/asg7.png)
![](https://raw.githubusercontent.com/fauzanooor/blog_post/draft/img/2020-10-27-Integrate-ASG-ELB-CodeDeploy-AWS/asg8.png)
![](https://raw.githubusercontent.com/fauzanooor/blog_post/draft/img/2020-10-27-Integrate-ASG-ELB-CodeDeploy-AWS/asg9.png)
![](https://raw.githubusercontent.com/fauzanooor/blog_post/draft/img/2020-10-27-Integrate-ASG-ELB-CodeDeploy-AWS/asg10.png)
![](https://raw.githubusercontent.com/fauzanooor/blog_post/draft/img/2020-10-27-Integrate-ASG-ELB-CodeDeploy-AWS/asg11.png)

Dan jika sudah selesai, maka nantinya pada registered target instance di
Target Group akan muncul secara otomatis 1 instance yang di-*initiate*
oleh ASG-nya, seperti berikut ini

![](https://raw.githubusercontent.com/fauzanooor/blog_post/draft/img/2020-10-27-Integrate-ASG-ELB-CodeDeploy-AWS/asg12.png)

Pembuatan CodeDeploy
--------------------

Ada 3 komponen yang digunakan oleh CodeDeploy, yaitu

1.  Aplications

2.  Deployment Group

3.  Deployment

Dan masing – masing punya keterkaitan satu sama lain, jika ingin membuat
*Deployment*, diharuskan untuk membuat *Deployment Group* dahulu, namun
jika ingin membuat *Deployment Group*, diharuskan membuat *Application*
terlebih dahulu.

Dan sebelum melakukan pembuatan ketiga komponen tersebut, kita perlu
membuat satu file yang bernama **appspec.yml**, yang mana jika kita
integrasi dengan EC2-nya, file ini berguna untuk 2 hal, yaitu :

1.  Apa saja yang harus diinstall atau diubah pada instance-nya (EC2)
    dari aplikasi yang sudah direvisi berdasarkan dari S3 atau
    GitHub-nya

2.  Dan juga bisa sebagai lifecycle event hook, yang berarti kita dapat
    mengontrol bagaimana flow proses updating aplikasi kita akan
    berjalan (seperti gambar di samping kanan)

Sebagai contoh, misal ketika ingin melakukan update ke versi yang lebih
baru, ternyata diperlukan mematikan service *nginx* terlebih dahulu
sebelum dilakukan update, maka pada event *BeforeInstall* diharuskan
untuk membuat script yang melakukan stop service nginx-nya. Dan kemudian
pada event *AfterInstall* membuat script lagi untuk melakukan start
service nginx nya kembali. Dan juga lebih jelasnya bisa dibaca referensi
dari sini

<https://docs.aws.amazon.com/codedeploy/latest/userguide/reference-appspec-file.html>

![](https://raw.githubusercontent.com/fauzanooor/blog_post/draft/img/2020-10-27-Integrate-ASG-ELB-CodeDeploy-AWS/cd1.png)

### **Contoh Aplikasi**

Contoh aplikasi disini berbasis web, dengan menggunakan service nginx,
dan hanya mempunyai file *index.html* yang sama seperti pada langkah
[Initial Configuration Golden
Instance](#initial-configuration-of-golden-instance-for-golden-image)
nya. Dan kemudian kita update background nya yang tadinya berwarna
*lime*, menjadi warna kuning. Untuk hal ini, ada 3 files yang
dibutuhkan, yaitu :

1.  file index.html yang telah diupdate background-nya menjadi warna
    kuning,

2.  file appspec.yml, yang mempunyai isi seperti berikut ini

```
version: 0.0
os: linux
files:
  - source: /index.html
    destination: /usr/share/nginx/html/
hooks:
  BeforeInstall:
    - location: scripts/remove\_unmodified\_file
      timeout: 300
      runas: root
```
- source : file/folder yang akan di*-put* pada instance(s) nya
- destination : path dari folder file source akan ditempatkan
- BeforeInstall : location : script file yang akan dijalankan sebelum aplikasinya dilakukan update (*sebelum index.html di-copy ke /usr/share/nginx/html/*)
- BeforeInstall : timeout : masa tunggu ketika script-nya dijalankan
- BeforeInstall : runas : user yang akan digunakan untuk menjalankan scriptnya

3.  File remove\_unmodified\_file, yang mana berisi action yang
    dilakukan sebelum aplikasinya diupdate, dan pada contoh aplikasi
    disini sebelum dilakukan upate file index.html yang lama (yang sudah
    berada di dalam instance sebelumnya dengan background color warna
    lime, akan dihapus)

```
#!/bin/bash
rm -rf /usr/share/nginx/html/index.html
```

Dan jika ke-3 files tersebut sudah siap, upload semua files tersebut ke
github.

### **Pembuatan Application**

1.  Pergi ke service **CodeDeploy** > Application > dan klik
    tombol **Create Application**

![](https://raw.githubusercontent.com/fauzanooor/blog_post/draft/img/2020-10-27-Integrate-ASG-ELB-CodeDeploy-AWS/cd2.png)

1.  Dan berikut ini detail konfigurasi application nya :

    a.  Application name : bangau-app-2

    b.  Compute platform : EC2/On-premises

![](https://raw.githubusercontent.com/fauzanooor/blog_post/draft/img/2020-10-27-Integrate-ASG-ELB-CodeDeploy-AWS/cd3.png)

### **Pembuatan Deployment Group**

1.  Masuk ke application yang telah dibuat, kemudian pada menu
    **Deployment Groups**, klik tombol **Create deployment group**

![](https://raw.githubusercontent.com/fauzanooor/blog_post/draft/img/2020-10-27-Integrate-ASG-ELB-CodeDeploy-AWS/cd4.png)

1.  Dan berikut ini detail konfigurasi deployment group nya :

    a.  Deployment group name : bangau-dg-2

    b.  Service role : CodeDeployServiceRole

    c.  Deployment type : Blue/green

    d.  Environment configuration : Automatically copy Amazon EC2 Auto
        Scaling group : bangau-asg

    e.  Deployment settings : Reroute traffice immediately
    - Terminate the original instances in the deployment group
    - 0 days, 0 hours, 5 minutes
    - Deployment configuration : CodeDeployDefault.AllatOnce
    
    f.  Load balancer : Application Load Balancer or Network Load
        Balancer : bangau-tg

![](https://raw.githubusercontent.com/fauzanooor/blog_post/draft/img/2020-10-27-Integrate-ASG-ELB-CodeDeploy-AWS/cd5.png)
![](https://raw.githubusercontent.com/fauzanooor/blog_post/draft/img/2020-10-27-Integrate-ASG-ELB-CodeDeploy-AWS/cd6.png)
![](https://raw.githubusercontent.com/fauzanooor/blog_post/draft/img/2020-10-27-Integrate-ASG-ELB-CodeDeploy-AWS/cd7.png)
![](https://raw.githubusercontent.com/fauzanooor/blog_post/draft/img/2020-10-27-Integrate-ASG-ELB-CodeDeploy-AWS/cd8.png)

Dan jika sudah selesai, maka deployment group nya seperti berikut ini

![](https://raw.githubusercontent.com/fauzanooor/blog_post/draft/img/2020-10-27-Integrate-ASG-ELB-CodeDeploy-AWS/cd9.png)


### **Pembuatan Deployment**

1.  Masuk ke deployment yang baru dibuat, dan klik tombol **Create
    deployment**

![](https://raw.githubusercontent.com/fauzanooor/blog_post/draft/img/2020-10-27-Integrate-ASG-ELB-CodeDeploy-AWS/cd10.png)

2.  Dan berikut ini konfigurasi detail dari deployment nya :

    a.  Revision type : My application is stored in GitHub
    - GitHub token name : fauzanooor
    - Repository name : {nama repository yang dari github nya}
    -   Commit ID : {commit ID yang dari github nya}

    b.  Additional deployment behavior settings
    -   Content options : Fail deployment

![](https://raw.githubusercontent.com/fauzanooor/blog_post/draft/img/2020-10-27-Integrate-ASG-ELB-CodeDeploy-AWS/cd11.png)

![](https://raw.githubusercontent.com/fauzanooor/blog_post/draft/img/2020-10-27-Integrate-ASG-ELB-CodeDeploy-AWS/cd12.png)

![](https://raw.githubusercontent.com/fauzanooor/blog_post/draft/img/2020-10-27-Integrate-ASG-ELB-CodeDeploy-AWS/cd13.png)

Dan nanti jika sudah selesai dibuat, maka akan redirect ke halaman
seperti berikut ini

![](https://raw.githubusercontent.com/fauzanooor/blog_post/draft/img/2020-10-27-Integrate-ASG-ELB-CodeDeploy-AWS/cd14.png)
![](https://raw.githubusercontent.com/fauzanooor/blog_post/draft/img/2020-10-27-Integrate-ASG-ELB-CodeDeploy-AWS/cd15.png)