# Domain Enumeration

Get current domain&#x20;

Get-Domain (PowerView)&#x20;

Get-ADDomain (ActiveDirectory Module)&#x20;

Get object of another domain&#x20;

Get-Domain -Domain moneycorp.local&#x20;

Get-ADDomain -Identity moneycorp.local

Get domain SID for the current domain&#x20;

Get-DomainSID (Get-ADDomain).DomainSID

Get domain policy for the current domain&#x20;

Get-DomainPolicyData (Get-DomainPolicyData).systemaccess

Get domain policy for another domain&#x20;

(Get-DomainPolicyData -domain moneycorp.local).systemaccess

Get domain controllers for the current domain&#x20;

Get-DomainController&#x20;

Get-ADDomainController

Get domain controllers for another domain&#x20;

Get-DomainController -Domain moneycorp.local&#x20;

Get-ADDomainController -DomainName moneycorp.local - Discover

