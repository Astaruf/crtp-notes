---
description: Enumerate users, computers, domain admins, enterprise admins
---

# 01 - AD Enumeration 1

## Objective

Enumerate following for the domain:

* Users
* Computers
* Domain Administrators
* Enterprise Administrators

## PowerView

Start a PowerShell session using Invisi-Shell to avoid enhanced logging. (\AD\Tools\InvisiShell\RunWithRegistryNonAdmin.bat)

```powershell
C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
```

Load Powerview in the PowerShell Session:

```powershell
. C:\Ad\Tools\PowerView.ps1
```

Get users in the domain:

```powershell
Get-DomainUser
```

Get only a specific property of all users:

```powershell
Get-DomainUser | select -ExpandProperty samaccountname
```

Get list of all properties for users in the current domain:

```powershell
Get-DomainUser -Identity student1 -Properties *
Get-DomainUser -Properties samaccountname,logonCount
```

Search for a particular string in a user's attributes:

```powershell
Get-DomainUser -LDAPFilter "Description=*built*" | Select name,Description
```

Get domain computer objects:

```powershell
Get-DomainComputer | select Name
Get-DomainComputer -OperatingSystem "*Server 2022*"
Get-DomainComputer -OperatingSystem "*Server 2022*" | select Name,operatingsystem
Get-DomainComputer -Ping
```

Get all the groups in the current domain:

```powershell
Get-DomainGroup | select Name
Get-DomainGroup -Domain <targetdomain>
```

Get all groups containing the word "admin" in group name:

```powershell
Get-DomainGroup *admin* | select samaccountname,description
```

Get details of the DA group, can get SIDs, Membernames:

```powershell
Get-DomainGroup -Identity "Domain Admins"
```

Get members of the Domain Admins group:

```powershell
Get-DomainGroupMember -Identity "Domain Admins" -Recurse
```

Get members of the Enterprise Admins group:

```powershell
Get-DomainGroupMember -Identity "Enterprise Admins" -Recurse
```

If you are not in the root domain, this command will not work. You need to query the root domain as EA group is only present in the root of the forest:

```powershell
Get-DomainGroupMember -Identity "Enterprise Admins" -Domain moneycorp.local
```

List all the local groups on a machine (needs administrator privs on non-dc machines):

```
Get-NetLocalGroup -ComputerName dcorp-dc
```

Get members of the local group "Administrators" on a machine (needs administrator privs on non-dc machines):

```powershell
Get-NetLocalGroupMember -ComputerName dcorp-dc -GroupName Administrators
```

Get actively logged users on a computer (needs local admin rights on the target):

```powershell
Get-NetLoggedon -ComputerName dcorp-adminsrv
```

Get locally logged users on a computer (needs remote registry on the target - started by-default on server OS):

```powershell
Get-LoggedonLocal -ComputerName dcorp-adminsrv
```

Get the last logged user on a computer (needs administrative rights and remote registry on the target):

```powershell
Get-LastLoggedOn -ComputerName dcorp-adminsrv
```

Find shares on hosts in current domain:

```powershell
Invoke-ShareFinder -Verbose
```

Find sensitive files on computers in the domain:

```powershell
Invoke-FileFinder -Verbose
```

Get all fileservers of the domain:

```powershell
Get-NetFileServer
```

## ADModule

Start a PowerShell session using Invisi-Shell to avoid enhanced logging. (\AD\Tools\InvisiShell\RunWithRegistryNonAdmin.bat)

<pre class="language-powershell"><code class="lang-powershell">C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
<strong>Import-Module C:\AD\Tools\ADModule-master\Microsoft.ActiveDirectory.Management.dll
</strong>Import-Module C:\AD\Tools\ADModule-master\ActiveDirectory\ActiveDirectory.psd1
</code></pre>

Get all the users in the current domain using the ADModule:

```powershell
Get-ADUser -Filter *
```

Filter by the properties you want to see:

```powershell
Get-ADUser -Filter * -Properties * | select Samaccountname,Description
```

Get list of all properties for users in the current domain:

<pre class="language-powershell"><code class="lang-powershell"><strong>Get-ADUser -Filter * -Properties * | select -First 1 | Get-Member -MemberType *Property | select Name
</strong>Get-ADUser -Filter * -Properties * | select name,logoncount,@{expression={[datetime]::fromFileTime($_.pwdlastset)}}
</code></pre>

Search for a particular string in a user's attributes:

```powershell
Get-ADUser -Filter 'Description -like "*built*"' -Properties Description | select name,Description
```

List all the computers in domain:

```powershell
Get-ADComputer -Filter *
Get-ADComputer -Filter * -Properties *
Get-ADComputer -Filter 'OperatingSystem -like "*Server 2022*"' -Properties OperatingSystem | select Name,OperatingSystem
Get-ADComputer -Filter * -Properties DNSHostName | %{TestConnection -Count 1 -ComputerName $_.DNSHostName}
```

Get domain computer objects:

```powershell
Get-ADComputer -Filter * -Properties * | select DNSHostName
```

Get all the groups in the current domain:

```powershell
Get-ADGroup -Filter * | select Name
Get-ADGroup -Filter * -Properties *
```

Get all groups containing the word "admin" in group name:

```powershell
Get-ADGroup -Filter 'Name -like "*admin*"' | select Name
```

Get Domain Administrators:

```powershell
Get-ADGroupMember -Identity "Domain Admins" -Recursive
```

Get the Enterprise Administrators:

```powershell
Get-ADGroupMember -Identity 'Enterprise Admins' -Server moneycorp.local
```

