

```powershell
#Ref:  Set up a NAT network : https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/user-guide/setup-nat-network
New-VMSwitch -SwitchName "NAT" -SwitchType Internal
Get-NetAdapter
New-NetIPAddress -IPAddress 172.30.0.1 -PrefixLength 24 -InterfaceIndex 24 
New-NetNat -Name NAT17230 -InternalIPInterfaceAddressPrefix 172.30.0.0/24![image](https://user-images.githubusercontent.com/21128234/184476596-6b3a98eb-3c5c-4f48-b419-39aec4e19ed0.png)
```
