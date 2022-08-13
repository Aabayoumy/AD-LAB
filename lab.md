
### Set up a NAT network 
https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/user-guide/setup-nat-network
```powershell
New-VMSwitch -SwitchName "NAT" -SwitchType Internal
Get-NetAdapter
New-NetIPAddress -IPAddress 172.30.0.1 -PrefixLength 24 -InterfaceIndex 24 
New-NetNat -Name NAT17230 -InternalIPInterfaceAddressPrefix 172.30.0.0/24
```
### Create DC01 VM 
https://dev.to/joeneville_/automate-hyper-v-vm-creation-with-powershell-1fgc
```powershell
# This script is in two parts. First we declare the variables to be applied.
$vm = "DC01" # name of VM, this just applies in Windows, it isn't applied to the OS guest itself.
$image = " D:\iso\SERVER_2022_EVAL_x64FRE_en-us.iso"
$vmswitch = "NAT" # name of your local vswitch
$port = "port1" # port on the VM
$vlan = 1 # VLAN that VM traffic will be send in
$cpu =  2 # Number of CPUs
$ram = 4GB # RAM of VM. Note this is not a string, not in quotation marks
$path_to_disk = "E:\Hyper-V\Virtual Hard Disks\" # Where you want the VM's virtual disk to reside
$disk_size = 20GB # VM storage, again, not a string

# Create a new VM
New-VM  $vm -Generation 2
# Set the CPU and start-up RAM
Set-VM $vm -ProcessorCount $cpu -MemoryStartupBytes $ram 
# Create the new VHDX disk - the path and size.
New-VHD -Path $path_to_disk$vm-disk1.vhdx -SizeBytes $disk_size
# Add the new disk to the VM
Add-VMHardDiskDrive -VMName $vm -Path $path_to_disk$vm-disk1.vhdx
# Assign the OS ISO file to the VM
Add-VMDvdDrive -VMName $vm -Path $image
# Set DVD Drive as first bootrm
Set-VMFirmware -VMName $vm -FirstBootDevice ( Get-VMDvdDrive -VMName $vm)
# Remove the default VM NIC named 'Network Adapter'
Remove-VMNetworkAdapter -VMName $vm 
# Add a new NIC to the VM and set its name
Add-VMNetworkAdapter -VMName $vm -Name $port
# Configure the NIC as access and assign VLAN
#Set-VMNetworkAdapterVlan -VMName $vm -VMNetworkAdapterName $port
# Connect the NIC to the vswitch
Connect-VMNetworkAdapter -VMName $vm -Name $port -SwitchName $vmswitch
Set-VM -Name $vm -CheckpointType Disabled
Disable-VMIntegrationService -VMName $vm "Time Synchronization"
Enable-VMIntegrationService -VMName $vm "Guest Service Interface"
```
#### Start new created VM and install windows with Full Desktop option & set administrator passsword to P@ssw0rd!

### Install Active Directory on DC01 VM 
```powershell
$plainPassword = "P@ssw0rd"
$password = $plainPassword | ConvertTo-SecureString -asPlainText -Force
$credential = New-Object System.Management.Automation.PSCredential("administrator", $password)
$vm = "DC01"

Write-Host "Set the network confugration , HostName and restart"
$session = New-PSSession -Vmname $vm -Credential $credential -Verbose
$code = {
netsh int ip set address "ethernet" static 172.30.0.10 255.255.255.0 172.30.0.1 1
netsh interface ipv4 add dnsserver name=Ethernet address=172.30.0.10 index=1 validate=no
netsh interface ipv4 add dnsserver name=Ethernet address=8.8.8.8 index=2 validate=no
Rename-Computer DC01 -Force -Restart
Exit
}
Start-Sleep -Seconds 30
Invoke-Command -Session $session -ScriptBlock $code
while((Get-VM -Name $vm).HeartBeat -notlike  'Ok*')
{Start-Sleep -Seconds 10}
Write-host $((Get-VM -Name $vm).HeartBeat)
$session = New-PSSession -Vmname $vm -Credential $credential -Verbose
$code = {
 $plainPassword = "P@ssw0rd"
 $password = $plainPassword | ConvertTo-SecureString -asPlainText -Force
 $credential = New-Object System.Management.Automation.PSCredential("administrator", $password)
$domainName = "contoso"
 $domain = "$domainName.local"
Write-Host "Installing management tools"
 Import-Module ServerManager
 Add-WindowsFeature RSAT-AD-PowerShell,RSAT-AD-AdminCenter
Write-Host "Deploying Active Directory Domain..."
 Install-WindowsFeature AD-domain-services, DNS -IncludeAllSubFeature -IncludeManagementTools -Restart
 Import-Module ADDSDeployment
 Install-ADDSForest `
 -SafeModeAdministratorPassword $password `
 -CreateDnsDelegation:$false `
 -DatabasePath "C:\Windows\NTDS" `
 -DomainMode "7" `
 -DomainName $domain `
 -DomainNetbiosName $domainName `
 -ForestMode "7" `
 -InstallDns:$true `
 -LogPath "C:\Windows\NTDS" `
 -NoRebootOnCompletion:$true `
 -SysvolPath "C:\Windows\SYSVOL" `
 -Force:$true
Restart-Computer -Force -Verbose
}
Invoke-Command -Session $session -ScriptBlock $code
```
