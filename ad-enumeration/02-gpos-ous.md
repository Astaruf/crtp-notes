---
description: Enumerate OUs and GPOs
---

# 02 - GPOs, OUs

Enumerate following for the domain:

* List all the OUs
* List all the computers in the StudentMachines OU.
* List the GPOs
* Enumerate GPO applied on the StudentMachines OU.

To list all OUs (Organizational Units):

```powershell
Get-DomainOU #PowerView
Get-ADOrganizationalUnit -Filter * -Properties * #ADModule
```

Get only OU names:

```powershell
Get-DomainOU | select -ExpandProperty name
```

List all the computers in the StudentsMachines OU:

```powershell
(Get-DomainOU -Identity StudentMachines).distinguishedname | %{Get-DomainComputer -SearchBase $_} | select name
```

List all GPOs (Group Policy Objects):

```powershell
Get-DomainGPO
```

List GPO applied on the StudentMachines OU, we need to copy a part of the gplink attribute from the output of the below command:

```powershell
(Get-DomainOU -Identity StudentMachines).gplink
```

The part you need to copy will look like:

```powershell
{7478F170-6A0C-490C-B355-9E4618BC785D}
```

Then, to see GPO applied to StudentMachines, use:

```powershell
Get-DomainGPO -Identity '{7478F170-6A0C-490C-B355-9E4618BC785D}'
```

Do the previous commands all in one command:

```powershell
Get-DomainGPO -Identity (Get-DomainOU -Identity StudentMachines).gplink.substring(11,(Get-DomainOU -Identity StudentMachines).gplink.length-72)
```

Get GPO(s) which use Restricted Groups or groups.xml for interesting users:

```powershell
Get-DomainGPOLocalGroup
```

Get users which are in a local group of a machine using GPO:

```powershell
Get-DomainGPOComputerLocalGroupMapping -ComputerIdentity dcorp-student1
```

Get machines where the given user is member of a specific group:

```powershell
Get-DomainGPOUserLocalGroupMapping -Identity student1 -Verbose
```
