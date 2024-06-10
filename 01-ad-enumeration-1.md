# Enumerating Domains

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

Get domain computer objects:

```powershell
Get-DomainComputer | select -ExpandProperty dnshostname
```

Get details of the DA group, can get SIDs, Membernames:

```powershell
Get-DomainGroup -Identity "Domain Admins"
```

Get members of the Domain Admins group:

```powershell
Get-DomainGroupMember -Identity "Domain Admins"
```

Get members of the Enterprise Admins group:

```powershell
Get-DomainGroupMember -Identity "Enterprise Admins"
```

If you are not in the root domain, this command will not work. You need to query the root domain as EA group is only present in the root of the forest:

```powershell
Get-DomainGroupMember -Identity "Enterprise Admins" -Domain moneycorp.local
```

## ADModule

Start a PowerShell session using Invisi-Shell to avoid enhanced logging. (\AD\Tools\InvisiShell\RunWithRegistryNonAdmin.bat)

```powershell
C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
Import-Module C:\AD\Tools\ADModule-master\Microsoft.ActiveDirectory.Management.dll
Import-Module C:\AD\Tools\ADModule-master\ActiveDirectory\ActiveDirectory.psd1
```

Get all the users in the current domain using the ADModule:

```powershell
Get-ADUser -Filter *
```

Filter by the properties you want to see:

```powershell
Get-ADUser -Filter * -Properties * | select Samaccountname,Description
```

List all the computers in domain:

```powershell
Get-ADComputer -Filter *
```

Get domain computer objects:

```powershell
Get-ADComputer -Filter * -Properties * | select DNSHostName
```

Get Domain Administrators:

```powershell
Get-ADGroupMember -Identity 'Domain Admins'
```

Get the Enterprise Administrators:

```powershell
Get-ADGroupMember -Identity 'Enterprise Admins' -Server moneycorp.local
```

