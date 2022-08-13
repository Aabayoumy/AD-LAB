

```powershell
#Ref:  Set up a NAT network : https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/user-guide/setup-nat-network
New-VMSwitch -SwitchName "NAT" -SwitchType Internal
Get-NetAdapter
New-NetIPAddress -IPAddress 172.30.0.1 -PrefixLength 24 -InterfaceIndex 24 
New-NetNat -Name NAT17230 -InternalIPInterfaceAddressPrefix 172.30.0.0/24
```
