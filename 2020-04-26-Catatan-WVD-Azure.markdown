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

---

Seperti itu..
