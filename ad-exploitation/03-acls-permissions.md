---
coverY: 0
layout:
  cover:
    visible: false
    size: full
  title:
    visible: true
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# 03 - ACLs, Permissions

Enumerate following for the domain:

* ACL for the Domain Admins group
* All modify rights/permissions for the current user

## PowerView

Get the ACLs associated with the specified object

```powershell
Get-DomainObjectAcl -SamAccountName student1 -ResolveGUIDs
```

Get the ACLs associated with the specified prefix to be used for search:

```powershell
Get-DomainObjectAcl -SearchBase "LDAP://CN=Domain Admins,CN=Users,DC=dollarcorp,DC=moneycorp,DC=local" -ResolveGUIDs -Verbose
```

We can also enumerate ACLs using **ActiveDirectory module** but without resolving GUIDs:

```powershell
(Get-Acl 'AD:\CN=Administrator,CN=Users,DC=dollarcorp,DC=moneycorp,DC=local').Access
```

Get the ACLs associated with the specified path:

```powershell
Get-PathAcl -Path "\\dcorp-dc.dollarcorp.moneycorp.local\sysvol"
```

Enumerate ACLs for the Domain Admins Group:

```powershell
Get-DomainObjectAcl -Identity "Domain Admins" -ResolveGUIDs -Verbose
# ObjectAceType : User-Account-Restrictions
```

Check for modify rights/permissions for the studentx, we can use FindInterestingDomainACL from PowerView:

```powershell
Find-InterestingDomainAcl -ResolveGUIDs | ?{$_.IdentityReferenceName -match "<student name>"}
```

To see permissions for the RDP group:

```powershell
Find-InterestingDomainAcl -ResolveGUIDs | ?{$_.IdentityReferenceName -match "RDPUsers"}
```

