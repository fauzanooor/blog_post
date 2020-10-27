---
layout: post
title:  "Catatan WVD Azure"
date:   2020-04-26
---

Modul Pendukung
===

Jadi ada 2 module yang akan dipakai untuk me-management WVD di Azure ini via powershell, yaitu modul `Microsoft.RDInfra.RDPowershell` dan modul `Az`, dan berikut ini cara installasi nya :

Buka **powershell** dan **run as administrator**, dan ubah policy menjadi unrestricted dulu, biar hidup lebih mudah walaupun dapat mengancam nyawa.

```
Set-executionpolicy -executionpolicy unrestricted
```

Install modul - modul nya

```
Install-Module -Name Microsoft.RDInfra.RDPowerShell -Force
Install-Module -Name Az -AllowClobber -Force
```

Import modul - modul nya

```
Import-Module -Name Microsoft.RDInfra.RDPowerShell
Import-Module -Name Az
```

Masuk ke tenant WVD via Powershell
===
```
Add-RdsAccount -DeploymentUrl "https://rdbroker.wvd.microsoft.com"
```


Buat Nge-check
===

### - Check siapa aja user yang lagi pake WVD nya
```
Get-RdsUserSession -TenantName "{nama tenant}" -HostPoolName "{nama pool}" | format-table -property SessionHostName,UserPrincipalName,CreateTime,SessionState
```
contoh output :
```
SessionHostName           UserPrincipalName    CreateTime           SessionState
---------------           -----------------    ----------           ------------
wvd-1.buburtimor.xyz      wvd1@buburtimor.xyz  4/26/2020 1:50:37 PM Disconnected
wvd-3.buburtimor.xyz      wvd5@buburtimor.xyz  4/10/2020 3:09:15 PM Active
```

### - Check status Host atau VM yang ada pada pool-nya
```
Get-rdssessionhost -TenantName "{nama tenant}" -HostPoolName "{nama pool}" | format-table -property SessionHostName,HostPoolName,Status,UpdateState,LastHeartBeat
```
contoh output :
```
SessionHostName           HostPoolName      status      UpdateState LastHeartBeat        
---------------           ------------      ------      ----------- -------------        
wvd-1.buburtimor.xyz      bbrtmr-pool-0     Available   Succeeded   4/25/2020 11:28:11 PM
wvd-2.buburtimor.xyz      bbrtmr-pool-0     Available   Succeeded   4/25/2020 11:11:13 PM
wvd-3.buburtimor.xyz      bbrtmr-pool-0     Available   Succeeded   4/25/2020 10:09:57 PM
```

### - Check user yang ada di dalam Application Group
```
Get-RdsAppGroupUser -TenantName "{nama tenant}" -HostPoolName "{nama pool}" -AppGroupName "{nama application group}" | Format-table -property UserPrincipalName,HostPoolName

```
contoh output :
```
UserPrincipalName    HostPoolName    
-----------------    ------------    
wvd1@buburtimor.xyz  bbrtmr-pool-0
wvd2@buburtimor.xyz  bbrtmr-pool-0
wvd3@buburtimor.xyz  bbrtmr-pool-0
wvd4@buburtimor.xyz  bbrtmr-pool-0
wvd5@buburtimor.xyz  bbrtmr-pool-0
```

### - Check aplikasi yang telah dipublish pada remote application group
```
Get-RdsRemoteApp -TenantName "{nama tenant}" -HostPoolName "{nama pool}" -AppGroupName "{nama application group}" | Format-Table -Property HostPoolName,AppGroupName,RemoteAppName,FriendlyName
```
contoh output :
```
HostPoolName     AppGroupName             RemoteAppName FriendlyName
------------     ------------             ------------- ------------
bbrtmr-pool-0    Remote Application Group WordPad       WordPad
```

### - Check settingan pool
```
Get-RdsHostPool -TenantName "{nama tenant}" | Format-Table -Property TenantName,TenantGroupName,HostPoolName,LoadBalancerType,AssignmentType,Persistent,MaxSessionLimit
```
contoh output :
```
TenantName       TenantGroupName      HostPoolName     LoadBalancerType AssignmentType Persistent MaxSessionLimit
----------       ---------------      ------------     ---------------- -------------- ---------- ---------------
bbrtmr-tenant-0  Default Tenant Group bbrtmr-pool-0    BreadthFirst                     False     5
bbrtmr-tenant-0  Default Tenant Group bbrtmr-pool-1    BreadthFirst                     False     3
bbrtmr-tenant-0  Default Tenant Group bbrtmr-pool-2    BreadthFirst                     False     2
```


Buat Join Domain Storage Account Azure ke Active Directory On-Premise
===

Cara ini untuk *case* yang mana mau pake file share yang ada di storage account azure, tapi untuk login ke file share tersebut pake user yang ada di AD on-premise.

Nha itu pertama harus join domain dulu storage account nya ke AD on-premise nya, untuk join domain nya tersebut harus pake powershell yang ada di dalam salah 1 VM yang udah join domain ke AD nya tersebut.

Dan sebelum join domain, ada perlu install modul tambahan dulu yaitu modul `AzFilesHybrid`, yang bisa di-download dari sini <https://github.com/Azure-Samples/azure-files-samples/releases>

Kalo udah di-download, extract file zip-nya yang mana di dalem nya itu nanti ada 3 files. Dan kalo udah di-extract jalanin command di bawah ini :

Copy module files nya ke direktori direktori tempat menyimpan modul - modul powershell
```
.\CopyToPSPath.ps1
```

Import modul nya 
```
Import-Module -Name AzFilesHybrid
```

Login ke Azure Account-nya dan juga pakein subscription azure nya
```
Connect-AzAccount
Select-AzSubscription -SubscriptionId {subscription ID nya}
```

Nha terus kalo udah masuk ke Azure-nya, bisa langsung join-in domain storage account nya :
```
join-AzStorageaccountForAuth `
    -ResourceGroupName "{nama resource group}" -Name "{nama storage account}" `
    -DomainAccountType "ComputerAccount" `
    -OrganizationalUnitName "{nama OU}"
```
btw itu yang *-OrganizationalUnitName* itu optional yak, ini dipake kalo mau dispesifikin mau ditaro di OU mana, kalo default nya ditaro di root directory

Dan untuk yang *-DomainAccountType* itu sebenernya ada 2 pilihan, yaitu tipe ComputerAccount dan tipe ServiceLogonAccount, tapi ini blum tau juga sih gue beda-nya apaan, nanti kalo udah tau gue update lagi artikel ini hehe. Tapi default nya sih pake ComputerAccount yak


Nha kalo udah, kalo kita liat di menu configuration yang ada di storage account-nya, nanti opsi untuk **Active Directory** dan juga info dari **Joined Domain** udah ke-ubah jadi seperti ini contoh nya

<p align="center">
  <img src="https://raw.githubusercontent.com/fauzanooor/blog_post/master/img/2020-04-26-Catatan-WVD-Azure/storage-account-configuration.png">
</p>

Tapii belom selesai, ada 1 step lagi yang perlu dilakuin agar file share yg ada di storage account ini bisa dipake. Dan berikut ini command nya :
```
Set-AzStorageAccount `
        -ResourceGroupName "{nama resource group}" `
        -Name "{nama storage account}" `
        -EnableActiveDirectoryDomainServicesForFile $true `
        -ActiveDirectoryDomainName "{nama domain}" `
        -ActiveDirectoryNetBiosDomainName "{nama netbios domain}" `
        -ActiveDirectoryForestName "{nama forest}" `
        -ActiveDirectoryDomainGuid "{GUID dari AD nya}" `
        -ActiveDirectoryDomainsid "{SID dari AD nya}" `
        -ActiveDirectoryAzureStorageSid "{SID dari storage account nya}"
```
Btw, untuk GUID ataupun SID di atas, bisa dicheck pada AD on-premise nya yak, bukan dari Azure nya. Dan untuk yang SID storage account itu maksudnya kan jadi pas storage account nya udah join domain, nanti ada **computer** baru yang ada di AD on-premise nya, nha jadi SID yg dipake itu SID dari computer tersebut.

Sama contoh satu lagi itu, buat pengingat aja haha :
- GUID itu bentuknya kek gini : 622465bb-2b87-4d50-815f-246e74bfda54
- SID itu bentuknya kek gini : S-1-5-33-45621879992-7698213116-8577482301

Dan kalo udah, nanti dari file share-nya tinggal setup IAM nya aja ke user yang ingin bisa pake file share tersebut.


---

Seperti itu..
