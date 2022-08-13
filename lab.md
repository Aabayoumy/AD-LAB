

```powershell
#Ref:  Set up a NAT network | Microsoft Docs![image](https://user-images.githubusercontent.com/21128234/184476599-d948f84f-d15c-4eee-b96c-42fc6d64acd6.png)
New-VMSwitch -SwitchName "NAT" -SwitchType Internal
Get-NetAdapter
New-NetIPAddress -IPAddress 172.30.0.1 -PrefixLength 24 -InterfaceIndex 24 
New-NetNat -Name NAT17230 -InternalIPInterfaceAddressPrefix 172.30.0.0/24![image](https://user-images.githubusercontent.com/21128234/184476596-6b3a98eb-3c5c-4f48-b419-39aec4e19ed0.png)
```
