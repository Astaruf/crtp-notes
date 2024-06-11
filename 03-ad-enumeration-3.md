---
description: Enumerate ACLs, permissions
---

# 03 - AD Enumeration 3

Enumerate following for the domain:

* ACL for the Domain Admins group
* All modify rights/permissions for the current user

## PowerView

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

