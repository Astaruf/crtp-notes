---
description: Forests, internal/external trusts
---

# 04 - AD Enumeration

* Enumerate all domains in the moneycorp.local forest.
* Map the trusts of the dollarcorp.moneycorp.local domain.
* Map External trusts in moneycorp.local forest.
* Identify external trusts of dollarcorp domain. Can you enumerate trusts for a trusting forest?

## PowerView

Let's enumerate all domains in the current forest:

```
Get-ForestDomain -Verbose
```

To map all the trusts of the dollarcorp domain:

```
Get-DomainTrust
```

