---
description: Forests, internal/external trusts
---

# 04 - Forests, Trusts

* Enumerate all domains in the moneycorp.local forest.
* Map the trusts of the dollarcorp.moneycorp.local domain.
* Map External trusts in moneycorp.local forest.
* Identify external trusts of dollarcorp domain. Can you enumerate trusts for a trusting forest?

## PowerView

Get details about the current forest:

```powershell
#PowerView
Get-Forest
Get-Forest -Forest eurocorp.local
#ADModule
Get-ADForest
Get-ADForest -Identity eurocorp.local
```

Let's enumerate all domains in the current forest:

```powershell
Get-ForestDomain -Verbose

#Find interesting info in Name field
Get-ForestDomain | select Name
```

Get all global catalogs for the current forest:

```powershell
#PowerView
Get-ForestGlobalCatalog
Get-ForestGlobalCatalog -Forest eurocorp.local
#ADModule
Get-ADForest | select -ExpandProperty GlobalCatalogs
```

Map trusts of a forest (no Forest trusts in the lab):

<pre class="language-powershell"><code class="lang-powershell"><strong>#PowerView
</strong><strong>Get-ForestTrust
</strong>Get-ForestTrust -Forest eurocorp.local
#ADModule
Get-ADTrust -Filter 'msDS-TrustForestTrustInfo -ne "$null"'
</code></pre>

To map all the trusts of the dollarcorp domain:

```powershell
Get-DomainTrust

#Find interesting info in SourceName,TargetName,TrustDirection,TrustAttributes
Get-DomainTrust | select SourceName,TargetName,TrustDirection,TrustAttributes
```

Get a list of all domain trusts for the current domain:

```powershell
#PowerView
Get-DomainTrust
Get-DomainTrust -Domain us.dollarcorp.moneycorp.local

#ADModule
Get-ADTrust
Get-ADTrust -Identity us.dollarcorp.moneycorp.local
```

Now, to list only the external trusts in the moneycorp.local forest:

```powershell
Get-ForestDomain | %{Get-DomainTrust -Domain $_.Name} | ?{$_.TrustAttributes -eq "FILTER_SIDS"}
#Find interesting info in SourceName,TargetName,TrustDirection,TrustAttributes
```

To identify external trusts of the dollarcorp domain, we can use the below command:

<pre class="language-powershell"><code class="lang-powershell">Get-DomainTrust | ?{$_.TrustAttributes -eq "FILTER_SIDS"}

<strong>&#x3C;# Output example:
</strong>SourceName      : dollarcorp.moneycorp.local
TargetName      : eurocorp.local
TrustType       : WINDOWS_ACTIVE_DIRECTORY
TrustAttributes : FILTER_SIDS
TrustDirection  : Bidirectional
WhenCreated     : 11/12/2022 8:15:23 AM
WhenChanged     : 6/5/2024 9:21:13 AM
<strong>#>
</strong></code></pre>

Given that the relationship is bi-directional, we can obtain information from the eurocorp.local forest. To use the command below, we need either a bi-directional trust or a one-way trust from eurocorp.local to dollarcorp.&#x20;

Let's list the trusts for the eurocorp.local forest:

```powershell
Get-ForestDomain -Forest eurocorp.local | %{Get-DomainTrust -Domain $_.Name}
```

{% hint style="info" %}
Notice the error above. It occurred because PowerView attempted to list trusts even for eu.eurocorp.local. Because external trust is non-transitive it was not possible!
{% endhint %}

## ADModule

Import the AD Module in a PowerShell session started using Invisi-Shell:

```powershell
C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
Import-Module C:\AD\Tools\ADModule-master\Microsoft.ActiveDirectory.Management.dll
Import-Module C:\AD\Tools\ADModule-master\ActiveDirectory\ActiveDirectory.psd1
```

Use the below command to enumerate all the domains in the current forest:

```powershell
(Get-ADForest).Domains

<# Output example
dollarcorp.moneycorp.local
moneycorp.local
us.dollarcorp.moneycorp.local
#>
```

To map all the trusts in the current domain, we can use the below command:

```powershell
Get-ADTrust -Filter * 
#Interesting info in Name field
```

To list all the trusts in the moneycorp.local forest:

```powershell
Get-ADForest | %{Get-ADTrust -Filter *} 

#Interesting info in fields: Direction, Source, Target
Get-ADForest | %{Get-ADTrust -Filter *} | select Direction, Source, Target
```

To list only the external trusts in moneycorp.local domain:

```powershell
(Get-ADForest).Domains | %{Get-ADTrust -Filter '(intraForest -ne $True) -and (ForestTransitive -ne $True)' -Server $_} 
#Interesting info in fields: ForestTransitive, IntraForest, Source, Target
```

Identify external trusts of the dollarcorp domain, we can use the below command. The output is same as above because there is just one external trust in the entire forest. Otherwise, output of the aboce command would be different than the below one:

```powershell
Get-ADTrust -Filter '(intraForest -ne $True) -and(ForestTransitive -ne $True)' 
#Interesting info in fields: ForestTransitive, IntraForest, Source, Target
```

Because we have trust relationship with eurocorp.local, we can enumerate trusts for it:

```powershell
Get-ADTrust -Filter * -Server eurocorp.local
#Interesting info in fields: IntraForest, Source, Target
```

