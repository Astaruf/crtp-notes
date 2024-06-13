# BloodHound

{% embed url="https://github.com/BloodHoundAD/BloodHound" %}
BloodHound Legacy
{% endembed %}

{% embed url="https://github.com/SpecterOps/BloodHound" %}
BloodHound Community Edition
{% endembed %}

BloodHound Legacy is present in the C:\AD\Tools

Supply data to BloodHound (Remember to [bypass .NET AMSI](../#bypass-amsi-windows-defender)):

```powershell
. C:\AD\Tools\BloodHound-master\Collectors\SharpHound.ps1
Invoke-BloodHound -CollectionMethod All -Verbose
Invoke-BloodHound -CollectionMethod LoggedOn -Verbose
```

or

```powershell
SharpHound.exe
```

The gathered data can be uploaded to the BloodHound application (both Legacy and Community Edition).

To make BloodHound collection stealthy, use –Steatlh option. This removes noisy collection methods like RDP, DCOM, PSRemote and LocalAdmin:

```powershell
Invoke-BloodHound –Steatlh
```

or

```powershell
SharpHound.exe –-steatlh
```

To avoid detections like MDI:

```powershell
Invoke-BloodHound -ExcludeDCs
```
