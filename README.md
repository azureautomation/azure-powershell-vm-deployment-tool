Azure Powershell VM Deployment Tool
===================================

            

 



PowerShell
Edit|Remove
powershell
Function CreateVnet {
param(
[string]$VNETName = $VNetName,
[string]$VNETResourceGroupName = $vNetResourceGroupName,
[string]$AddRange = $AddRange,
[string]$Location = $Location,
[string]$SubnetAddPrefix1 = $SubnetAddPrefix1,
[string]$SubnetNameAddPrefix1 = $SubnetNameAddPrefix1,
[string]$SubnetAddPrefix2 = $SubnetAddPrefix2,
[string]$SubnetNameAddPrefix2 = $SubnetNameAddPrefix2,
[string]$SubnetAddPrefix3 = $SubnetAddPrefix3,
[string]$SubnetNameAddPrefix3 = $SubnetNameAddPrefix3,
[string]$SubnetAddPrefix4 = $SubnetAddPrefix4,
[string]$SubnetNameAddPrefix4 = $SubnetNameAddPrefix4,
[string]$SubnetAddPrefix5 = $SubnetAddPrefix5,
[string]$SubnetNameAddPrefix5 = $SubnetNameAddPrefix5,
[string]$SubnetAddPrefix6 = $SubnetAddPrefix6,
[string]$SubnetNameAddPrefix6 = $SubnetNameAddPrefix6,
[string]$SubnetAddPrefix7 = $SubnetAddPrefix7,
[string]$SubnetNameAddPrefix7 = $SubnetNameAddPrefix7,
[string]$SubnetAddPrefix8 = $SubnetAddPrefix8,
[string]$SubnetNameAddPrefix8 = $SubnetNameAddPrefix8
)
Write-Host 'Network Preparation in Process..'
$subnet1 = New-AzureRmVirtualNetworkSubnetConfig -AddressPrefix $SubnetAddPrefix1 -Name $SubnetNameAddPrefix1
$subnet2 = New-AzureRmVirtualNetworkSubnetConfig -AddressPrefix $SubnetAddPrefix2 -Name $SubnetNameAddPrefix2
$subnet3 = New-AzureRmVirtualNetworkSubnetConfig -AddressPrefix $SubnetAddPrefix3 -Name $SubnetNameAddPrefix3
$subnet4 = New-AzureRmVirtualNetworkSubnetConfig -AddressPrefix $SubnetAddPrefix4 -Name $SubnetNameAddPrefix4
$subnet5 = New-AzureRmVirtualNetworkSubnetConfig -AddressPrefix $SubnetAddPrefix5 -Name $SubnetNameAddPrefix5
$subnet6 = New-AzureRmVirtualNetworkSubnetConfig -AddressPrefix $SubnetAddPrefix6 -Name $SubnetNameAddPrefix6
$subnet7 = New-AzureRmVirtualNetworkSubnetConfig -AddressPrefix $SubnetAddPrefix7 -Name $SubnetNameAddPrefix7
$subnet8 = New-AzureRmVirtualNetworkSubnetConfig -AddressPrefix $SubnetAddPrefix8 -Name $SubnetNameAddPrefix8
New-AzureRmVirtualNetwork -Location $Location -Name $VNetName -ResourceGroupName $vNetResourceGroupName -AddressPrefix $AddRange -Subnet $subnet1,$subnet2,$subnet3,$subnet4,$subnet5,$subnet6,$subnet7,$subnet8 –Confirm:$false -WarningAction SilentlyContinue -Force | Out-Null
Get-AzureRmVirtualNetwork -Name $VNetName -ResourceGroupName $vNetResourceGroupName | Get-AzureRmVirtualNetworkSubnetConfig -WarningAction SilentlyContinue | Out-Null
Write-Host 'Network Preparation completed' -ForegroundColor White
$LogOut = 'Completed Network Configuration of $VNetName'
Log-Command -Description $LogOut -LogFile $LogOutFile
}

# End of Provision VNET Function
Function CreateNSG {
param(
[string]$NSGName = $NSGName,
[string]$Location = $Location,
[string]$VNETResourceGroupName = $vNetResourceGroupName
)
Write-Host 'Network Security Group Preparation in Process..'
$httprule = New-AzureRmNetworkSecurityRuleConfig -Name 'FrontEnd_HTTP' -Description 'HTTP Exception for Web frontends' -Protocol Tcp -SourcePortRange '80' -DestinationPortRange '80' -SourceAddressPrefix '*' -DestinationAddressPrefix '10.120.0.0/21' -Access Allow -Direction Inbound -Priority 200
$httpsrule = New-AzureRmNetworkSecurityRuleConfig -Name 'FrontEnd_HTTPS' -Description 'HTTPS Exception for Web frontends' -Protocol Tcp -SourcePortRange '443' -DestinationPortRange '443' -SourceAddressPrefix '*' -DestinationAddressPrefix '10.120.0.0/21' -Access Allow -Direction Inbound -Priority 201
$sshrule = New-AzureRmNetworkSecurityRuleConfig -Name 'FrontEnd_SSH' -Description 'SSH Exception for Web frontends' -Protocol Tcp -SourcePortRange '22' -DestinationPortRange '22' -SourceAddressPrefix '*' -DestinationAddressPrefix '10.120.0.0/21' -Access Allow -Direction Inbound ` -Priority 203
$nsg = New-AzureRmNetworkSecurityGroup -ResourceGroupName $vNetResourceGroupName -Location $Location -Name $NSGName -SecurityRules $httprule,$httpsrule, $sshrule –Confirm:$false -WarningAction SilentlyContinue -Force | Out-Null
Get-AzureRmNetworkSecurityGroup -Name $NSGName -ResourceGroupName $vNetResourceGroupName -WarningAction SilentlyContinue | Out-Null
Write-Host 'Network Security Group configuration completed' -ForegroundColor White
$LogOut = 'Security Rules added for $NSGName'
$secrules =Get-AzureRmNetworkSecurityGroup -Name $NSGName -ResourceGroupName $vNetResourceGroupName -ExpandResource NetworkInterfaces | Get-AzureRmNetworkSecurityRuleConfig | Ft Name,Description,Direction,SourcePortRange,DestinationPortRange,DestinationPortRange,SourceAddressPrefix,Access
$defsecrules = Get-AzureRmNetworkSecurityGroup -Name $NSGName -ResourceGroupName $vNetResourceGroupName -ExpandResource NetworkInterfaces | Get-AzureRmNetworkSecurityRuleConfig -DefaultRules | Ft Name,Description,Direction,SourcePortRange,DestinationPortRange,DestinationAddressPrefix,SourceAddressPrefix,Access
Log-Command -Description $LogOut -LogFile $LogOutFile
$LogOut = 'Completed NSG Configuration of $NSGName'
Log-Command -Description $LogOut -LogFile $LogOutFile
}

Function MakeImagePlanInfo_Bitnami_Lamp {
param(
	[string]$VMName = $VMName,
	[string]$Publisher = 'bitnami',
	[string]$offer = 'lampstack',
	[string]$Skus = '5-6',
	[string]$version = 'latest',
	[string]$Product = 'lampstack',
	[string]$name = '5-6'
)
Write-Host 'Image Creation in Process - Plan Info - LampStack' -ForegroundColor White
Write-Host 'Publisher:'$Publisher 'Offer:'$offer 'Sku:'$Skus 'Version:'$version
$global:VirtualMachine= Set-AzureRmVMPlan -VM $VirtualMachine -Name $name -Publisher $Publisher -Product $Product
$global:VirtualMachine = Set-AzureRmVMOperatingSystem -VM $VirtualMachine -linux -ComputerName $VMName -Credential $Credential1
$global:VirtualMachine = Set-AzureRmVMSourceImage -VM $VirtualMachine -PublisherName $Publisher -Offer $offer -Skus $Skus -Version $version
$LogOut = 'Completed image prep 'Publisher:'$Publisher 'Offer:'$offer 'Sku:'$Skus 'Version:'$version'
Log-Command -Description $LogOut -LogFile $LogOutFile
}
Function AddDiskImage {
Write-Host 'Completing image creation...' -ForegroundColor White
$global:osDiskCaching = 'ReadWrite'
$global:OSDiskName = $VMName + 'OSDisk'
$global:OSDiskUri = $StorageAccount.PrimaryEndpoints.Blob.ToString() + 'vhds/' + $OSDiskName + '.vhd'
$global:VirtualMachine = Set-AzureRmVMOSDisk -VM $VirtualMachine -Name $OSDiskName -VhdUri $OSDiskUri -CreateOption 'FromImage' -Caching $osDiskCaching -WarningAction SilentlyContinue
}
function Provvms {
	 param (
	[string]$ResourceGroupName = $ResourceGroupName,
	[string]$Location = $Location
	 )
	$ProvisionVMs = @($VirtualMachine);
try {
   foreach($provisionvm in $ProvisionVMs) {
		New-AzureRmVM -ResourceGroupName $ResourceGroupName -Location $Location -VM $VirtualMachine –Confirm:$false -WarningAction SilentlyContinue | Out-Null
		$LogOut = 'Completed Creation of $VMName from $vmMarketImage'
		Log-Command -Description $LogOut -LogFile $LogOutFile
		WriteResults # Provides Post-Deployment Description
						}
	}
catch {
	Write-Host -foregroundcolor Red `
	'$($_.Exception.Message)'; `
	break
}
	 }
Function CreateVPN {
Write-Host 'VPN Creation can take up to 45 minutes!'
New-AzureRmLocalNetworkGateway -Name LocalSite -ResourceGroupName $vNetResourceGroupName -Location $Location -GatewayIpAddress $LocalNetPip -AddressPrefix $LocalAddPrefix -ErrorAction Stop -WarningAction SilentlyContinue | Out-Null
Write-Host 'Completed Local Network GW Creation'
$vpnpip= New-AzureRmPublicIpAddress -Name vpnpip -ResourceGroupName $vNetResourceGroupName -Location $Location -AllocationMethod Dynamic -ErrorAction Stop -WarningAction SilentlyContinue
$vnet = Get-AzureRmVirtualNetwork -Name $VNetName -ResourceGroupName $vNetResourceGroupName -ErrorAction Stop -WarningAction SilentlyContinue
$subnet = Get-AzureRmVirtualNetworkSubnetConfig -Name 'GatewaySubnet' -VirtualNetwork $vnet -WarningAction SilentlyContinue
$vpnipconfig = New-AzureRmVirtualNetworkGatewayIpConfig -Name vpnipconfig1 -SubnetId $subnet.Id -PublicIpAddressId $vpnpip.Id -WarningAction SilentlyContinue
New-AzureRmVirtualNetworkGateway -Name vnetvpn1 -ResourceGroupName $vNetResourceGroupName -Location $Location -IpConfigurations $vpnipconfig -GatewayType Vpn -VpnType RouteBased -GatewaySku Standard -ErrorAction Stop -WarningAction SilentlyContinue | Out-Null
Write-Host 'Completed VNET Network GW Creation'
Get-AzureRmPublicIpAddress -Name vpnpip -ResourceGroupName $ResourceGroupName -WarningAction SilentlyContinue
Write-Host 'Configure Local Device with Azure VNET vpn Public IP'
}
Function ConnectVPN {
[PSObject]$gateway1 = Get-AzureRmVirtualNetworkGateway -Name vnetvpn1 -ResourceGroupName $vNetResourceGroupName -WarningAction SilentlyContinue
[PSObject]$local = Get-AzureRmLocalNetworkGateway -Name LocalSite -ResourceGroupName $vNetResourceGroupName -WarningAction SilentlyContinue
New-AzureRmVirtualNetworkGatewayConnection -ConnectionType IPSEC  -Name sitetosite -ResourceGroupName $vNetResourceGroupName -Location $Location -VirtualNetworkGateway1 $gateway1 -LocalNetworkGateway2 $local -SharedKey '4321avfe' -Verbose -Force -RoutingWeight 10 -WarningAction SilentlyContinue| Out-Null
}

Function CreateVnet { 
param( 
[string]$VNETName = $VNetName, 
[string]$VNETResourceGroupName = $vNetResourceGroupName, 
[string]$AddRange = $AddRange, 
[string]$Location = $Location, 
[string]$SubnetAddPrefix1 = $SubnetAddPrefix1, 
[string]$SubnetNameAddPrefix1 = $SubnetNameAddPrefix1, 
[string]$SubnetAddPrefix2 = $SubnetAddPrefix2, 
[string]$SubnetNameAddPrefix2 = $SubnetNameAddPrefix2, 
[string]$SubnetAddPrefix3 = $SubnetAddPrefix3, 
[string]$SubnetNameAddPrefix3 = $SubnetNameAddPrefix3, 
[string]$SubnetAddPrefix4 = $SubnetAddPrefix4, 
[string]$SubnetNameAddPrefix4 = $SubnetNameAddPrefix4, 
[string]$SubnetAddPrefix5 = $SubnetAddPrefix5, 
[string]$SubnetNameAddPrefix5 = $SubnetNameAddPrefix5, 
[string]$SubnetAddPrefix6 = $SubnetAddPrefix6, 
[string]$SubnetNameAddPrefix6 = $SubnetNameAddPrefix6, 
[string]$SubnetAddPrefix7 = $SubnetAddPrefix7, 
[string]$SubnetNameAddPrefix7 = $SubnetNameAddPrefix7, 
[string]$SubnetAddPrefix8 = $SubnetAddPrefix8, 
[string]$SubnetNameAddPrefix8 = $SubnetNameAddPrefix8 
) 
Write-Host 'Network Preparation in Process..' 
$subnet1 = New-AzureRmVirtualNetworkSubnetConfig -AddressPrefix $SubnetAddPrefix1 -Name $SubnetNameAddPrefix1 
$subnet2 = New-AzureRmVirtualNetworkSubnetConfig -AddressPrefix $SubnetAddPrefix2 -Name $SubnetNameAddPrefix2 
$subnet3 = New-AzureRmVirtualNetworkSubnetConfig -AddressPrefix $SubnetAddPrefix3 -Name $SubnetNameAddPrefix3 
$subnet4 = New-AzureRmVirtualNetworkSubnetConfig -AddressPrefix $SubnetAddPrefix4 -Name $SubnetNameAddPrefix4 
$subnet5 = New-AzureRmVirtualNetworkSubnetConfig -AddressPrefix $SubnetAddPrefix5 -Name $SubnetNameAddPrefix5 
$subnet6 = New-AzureRmVirtualNetworkSubnetConfig -AddressPrefix $SubnetAddPrefix6 -Name $SubnetNameAddPrefix6 
$subnet7 = New-AzureRmVirtualNetworkSubnetConfig -AddressPrefix $SubnetAddPrefix7 -Name $SubnetNameAddPrefix7 
$subnet8 = New-AzureRmVirtualNetworkSubnetConfig -AddressPrefix $SubnetAddPrefix8 -Name $SubnetNameAddPrefix8 
New-AzureRmVirtualNetwork -Location $Location -Name $VNetName -ResourceGroupName $vNetResourceGroupName -AddressPrefix $AddRange -Subnet $subnet1,$subnet2,$subnet3,$subnet4,$subnet5,$subnet6,$subnet7,$subnet8 –Confirm:$false -WarningAction SilentlyContinue -Force | Out-Null 
Get-AzureRmVirtualNetwork -Name $VNetName -ResourceGroupName $vNetResourceGroupName | Get-AzureRmVirtualNetworkSubnetConfig -WarningAction SilentlyContinue | Out-Null 
Write-Host 'Network Preparation completed' -ForegroundColor White 
$LogOut = 'Completed Network Configuration of $VNetName' 
Log-Command -Description $LogOut -LogFile $LogOutFile 
} 
 
# End of Provision VNET Function 
Function CreateNSG { 
param( 
[string]$NSGName = $NSGName, 
[string]$Location = $Location, 
[string]$VNETResourceGroupName = $vNetResourceGroupName 
) 
Write-Host 'Network Security Group Preparation in Process..' 
$httprule = New-AzureRmNetworkSecurityRuleConfig -Name 'FrontEnd_HTTP' -Description 'HTTP Exception for Web frontends' -Protocol Tcp -SourcePortRange '80' -DestinationPortRange '80' -SourceAddressPrefix '*' -DestinationAddressPrefix '10.120.0.0/21' -Access Allow -Direction Inbound -Priority 200 
$httpsrule = New-AzureRmNetworkSecurityRuleConfig -Name 'FrontEnd_HTTPS' -Description 'HTTPS Exception for Web frontends' -Protocol Tcp -SourcePortRange '443' -DestinationPortRange '443' -SourceAddressPrefix '*' -DestinationAddressPrefix '10.120.0.0/21' -Access Allow -Direction Inbound -Priority 201 
$sshrule = New-AzureRmNetworkSecurityRuleConfig -Name 'FrontEnd_SSH' -Description 'SSH Exception for Web frontends' -Protocol Tcp -SourcePortRange '22' -DestinationPortRange '22' -SourceAddressPrefix '*' -DestinationAddressPrefix '10.120.0.0/21' -Access Allow -Direction Inbound ` -Priority 203 
$nsg = New-AzureRmNetworkSecurityGroup -ResourceGroupName $vNetResourceGroupName -Location $Location -Name $NSGName -SecurityRules $httprule,$httpsrule, $sshrule –Confirm:$false -WarningAction SilentlyContinue -Force | Out-Null 
Get-AzureRmNetworkSecurityGroup -Name $NSGName -ResourceGroupName $vNetResourceGroupName -WarningAction SilentlyContinue | Out-Null 
Write-Host 'Network Security Group configuration completed' -ForegroundColor White 
$LogOut = 'Security Rules added for $NSGName' 
$secrules =Get-AzureRmNetworkSecurityGroup -Name $NSGName -ResourceGroupName $vNetResourceGroupName -ExpandResource NetworkInterfaces | Get-AzureRmNetworkSecurityRuleConfig | Ft Name,Description,Direction,SourcePortRange,DestinationPortRange,DestinationPortRange,SourceAddressPrefix,Access 
$defsecrules = Get-AzureRmNetworkSecurityGroup -Name $NSGName -ResourceGroupName $vNetResourceGroupName -ExpandResource NetworkInterfaces | Get-AzureRmNetworkSecurityRuleConfig -DefaultRules | Ft Name,Description,Direction,SourcePortRange,DestinationPortRange,DestinationAddressPrefix,SourceAddressPrefix,Access 
Log-Command -Description $LogOut -LogFile $LogOutFile 
$LogOut = 'Completed NSG Configuration of $NSGName' 
Log-Command -Description $LogOut -LogFile $LogOutFile 
} 
 
Function MakeImagePlanInfo_Bitnami_Lamp { 
param( 
    [string]$VMName = $VMName, 
    [string]$Publisher = 'bitnami', 
    [string]$offer = 'lampstack', 
    [string]$Skus = '5-6', 
    [string]$version = 'latest', 
    [string]$Product = 'lampstack', 
    [string]$name = '5-6' 
) 
Write-Host 'Image Creation in Process - Plan Info - LampStack' -ForegroundColor White 
Write-Host 'Publisher:'$Publisher 'Offer:'$offer 'Sku:'$Skus 'Version:'$version 
$global:VirtualMachine= Set-AzureRmVMPlan -VM $VirtualMachine -Name $name -Publisher $Publisher -Product $Product 
$global:VirtualMachine = Set-AzureRmVMOperatingSystem -VM $VirtualMachine -linux -ComputerName $VMName -Credential $Credential1 
$global:VirtualMachine = Set-AzureRmVMSourceImage -VM $VirtualMachine -PublisherName $Publisher -Offer $offer -Skus $Skus -Version $version 
$LogOut = 'Completed image prep 'Publisher:'$Publisher 'Offer:'$offer 'Sku:'$Skus 'Version:'$version' 
Log-Command -Description $LogOut -LogFile $LogOutFile 
} 
Function AddDiskImage { 
Write-Host 'Completing image creation...' -ForegroundColor White 
$global:osDiskCaching = 'ReadWrite' 
$global:OSDiskName = $VMName + 'OSDisk' 
$global:OSDiskUri = $StorageAccount.PrimaryEndpoints.Blob.ToString() + 'vhds/' + $OSDiskName + '.vhd' 
$global:VirtualMachine = Set-AzureRmVMOSDisk -VM $VirtualMachine -Name $OSDiskName -VhdUri $OSDiskUri -CreateOption 'FromImage' -Caching $osDiskCaching -WarningAction SilentlyContinue 
} 
function Provvms { 
     param ( 
    [string]$ResourceGroupName = $ResourceGroupName, 
    [string]$Location = $Location 
     ) 
    $ProvisionVMs = @($VirtualMachine); 
try { 
   foreach($provisionvm in $ProvisionVMs) { 
        New-AzureRmVM -ResourceGroupName $ResourceGroupName -Location $Location -VM $VirtualMachine –Confirm:$false -WarningAction SilentlyContinue | Out-Null 
        $LogOut = 'Completed Creation of $VMName from $vmMarketImage' 
        Log-Command -Description $LogOut -LogFile $LogOutFile 
        WriteResults # Provides Post-Deployment Description 
                        } 
    } 
catch { 
    Write-Host -foregroundcolor Red ` 
    '$($_.Exception.Message)'; ` 
    break 
} 
     } 
Function CreateVPN { 
Write-Host 'VPN Creation can take up to 45 minutes!' 
New-AzureRmLocalNetworkGateway -Name LocalSite -ResourceGroupName $vNetResourceGroupName -Location $Location -GatewayIpAddress $LocalNetPip -AddressPrefix $LocalAddPrefix -ErrorAction Stop -WarningAction SilentlyContinue | Out-Null 
Write-Host 'Completed Local Network GW Creation' 
$vpnpip= New-AzureRmPublicIpAddress -Name vpnpip -ResourceGroupName $vNetResourceGroupName -Location $Location -AllocationMethod Dynamic -ErrorAction Stop -WarningAction SilentlyContinue 
$vnet = Get-AzureRmVirtualNetwork -Name $VNetName -ResourceGroupName $vNetResourceGroupName -ErrorAction Stop -WarningAction SilentlyContinue 
$subnet = Get-AzureRmVirtualNetworkSubnetConfig -Name 'GatewaySubnet' -VirtualNetwork $vnet -WarningAction SilentlyContinue 
$vpnipconfig = New-AzureRmVirtualNetworkGatewayIpConfig -Name vpnipconfig1 -SubnetId $subnet.Id -PublicIpAddressId $vpnpip.Id -WarningAction SilentlyContinue 
New-AzureRmVirtualNetworkGateway -Name vnetvpn1 -ResourceGroupName $vNetResourceGroupName -Location $Location -IpConfigurations $vpnipconfig -GatewayType Vpn -VpnType RouteBased -GatewaySku Standard -ErrorAction Stop -WarningAction SilentlyContinue | Out-Null 
Write-Host 'Completed VNET Network GW Creation' 
Get-AzureRmPublicIpAddress -Name vpnpip -ResourceGroupName $ResourceGroupName -WarningAction SilentlyContinue 
Write-Host 'Configure Local Device with Azure VNET vpn Public IP' 
} 
Function ConnectVPN { 
[PSObject]$gateway1 = Get-AzureRmVirtualNetworkGateway -Name vnetvpn1 -ResourceGroupName $vNetResourceGroupName -WarningAction SilentlyContinue 
[PSObject]$local = Get-AzureRmLocalNetworkGateway -Name LocalSite -ResourceGroupName $vNetResourceGroupName -WarningAction SilentlyContinue 
New-AzureRmVirtualNetworkGatewayConnection -ConnectionType IPSEC  -Name sitetosite -ResourceGroupName $vNetResourceGroupName -Location $Location -VirtualNetworkGateway1 $gateway1 -LocalNetworkGateway2 $local -SharedKey '4321avfe' -Verbose -Force -RoutingWeight 10 -WarningAction SilentlyContinue| Out-Null 
}




 


Why did I create these tools?


Azure is an amazing platform that is still in its infancy and thus still a bit quirky in the way some of the operations are executed via Azure PowerShell regardless of whether ARM json templates or just PowerShell are used for
 automation.


Specifically, the image creation steps are not necessarily standardized. These scripts account for some of those differences for both ARM Json based deployments as well as for ARM PowerShell deployments.


There are also a number of Azure operations which can currently only be executed when a VM is created. Some of these operations include adding a VM to an availability set and provisioning network adapters. Provisioning VMs will
 also always require a Network to place the VM on, as well as Blob storage to place the VHD in, and a Resource Group for all of the items created for the VM. By combining all the steps into a single consolidated script it vastly simplifies the creation process.
 The end result of the script (which in the longer term will be converted to a module) is to provide a flexible automation deployment tool which can be used for either ad-hoc deployments or as part of a release process into Azure. 


 


This script provides the following functionality for deploying IaaS environments in Azure. The script will deploy VNET in addition to numerous Market Place VMs or make use of an existing VNET. The script allows deployment of
 NSGs as well as custom network configuration options. The script can also be leveraged for post deployment of Azure Extensions as well as Availability Sets.


![Image](https://github.com/azureautomation/azure-powershell-vm-deployment-tool/raw/master/supportedos.jpg)



Updates


v11 updates for handling Premium Storage and Boot Diagnostics


**Please 'LIKE THIS' contribution if you find it valuable**



[Github](https://github.com/JonosGit/IaaSDeploymentTool/)


Market Images supported: Redhat 6.7 and 7.2, PFSense 2.5, Windows 2008 R2, Windows 2012 R2, Ubuntu 14.04, CentOs 7.2, SUSE, SQL 2016 (on W2K12R2), R Server on Windows, Windows 2016 (Preview), Checkpoint Firewall, FreeBsd, Puppet,
 Splunk, Bitnami Lamp, Bitnami PostGresSql, Bitnami nodejs, Bitnami Elastics, Bitnami MySql, SharePoint 2013/2016, Barracuda NG, Barracuda SPAM, F5 BigIP, F5 App Firewall, Bitnami JRuby, Bitnami Neos, Bitnami TomCat, Bitnami redis, Bitnami hadoop, Incredibuild,
 VS 2015, Dev15 Preview, Tableau, MS NAV, TFS, Ads Data Science Server, Biztalk 2013/2016, HortonWorks, Cloudera, DataStax


**Related Articles on LinkedIn**

[LinkedIn - Enabling Encryption on IaaS VMs in Azure](https://www.linkedin.com/pulse/flipping-bit-vm-encryption-key-vault-using-azure-powershell-lewis)

[LinkedIn - Gaining visibility into Network Security Groups – Sending Diagnostics to Azure Storage](https://www.linkedin.com/pulse/gaining-visibility-network-stack-sending-nsg-lb-azure-john-n-lewis)

[LinkedIn - Gaining visibility into Network Security Groups – Sending Diagnostics to OMS](https://www.linkedin.com/pulse/gaining-visibility-network-stack-sending-nsg-lb-azure-john-n-lewis)

[LinkedIn - Tips for SQL Server patch Management in Azure](https://www.linkedin.com/pulse/tips-sql-server-security-update-management-azure-john-n-lewis)


[LinkedIn – Forced Cloud Migration](https://www.linkedin.com/pulse/forced-cloud-migration-day-after-on-premise-san-becomes-john-n-lewis?trk=mp-author-card)

[LinkedIn – azrm-vmdeploy.ps1 Version 2 Released](https://www.linkedin.com/pulse/azure-iaas-deployment-tool-now-known-azrm-vmdeployps1-john-n-lewis?trk=hp-feed-article-title-publish)

[LinkedIn – Configuring Internal and External Load Balancers in Azure](https://www.linkedin.com/pulse/configuring-external-load-balancer-azure-rm-via-powershell-lewis?trk=hp-feed-article-title-publish)

[LinkedIn – How to handle file dependencies in hybrid cloud scenarios using Azure Storage](https://www.linkedin.com/pulse/azure-storage-file-shares-how-handle-your-legacy-around-john-n-lewis?trk=hp-feed-article-title-publish)

[LinkedIn – Azure Custom Script Extension](https://www.linkedin.com/pulse/azure-extension-custom-script-upload-vm-blob-storage-now-lewis?trk=hp-feed-article-title-publish)

[LinkedIn – Azure Extensions and OS variation](https://www.linkedin.com/pulse/azure-extensions-ambiguity-what-flavor-operating-system-john-n-lewis?trk=hp-feed-article-title-publish)

[LinkedIn – Azure Extension Linux Updates](https://www.linkedin.com/pulse/building-provisioning-automation-azure-iaas-arm-using-john-n-lewis?trk=mp-author-card)

[LinkedIn – Azure Extensions Linux Backups](https://www.linkedin.com/pulse/building-provisioning-automation-azure-arm-using-powershell-lewis?trk=mp-author-card)

[LinkedIn – Azure Extensions DSC and Automatic Updates](https://www.linkedin.com/pulse/how-automate-automatic-updates-azure-iaas-windows-vms-john-n-lewis?trk=mp-author-card)

[LinkedIn – Azure Extensions Domain Join](https://www.linkedin.com/pulse/azure-iaas-20-arm-powershell-domain-join-extension-john-n-lewis?trk=mp-author-card)

[LinkedIn – Pull all OS Image options for IaaS Azure RM Deployment](https://www.linkedin.com/pulse/azure-powershell-iaas-tips-how-pull-list-every-os-image-john-n-lewis?trk=mp-author-card)

LinkedIn – Confused about 3rd Party Image Configuration in Azure RM?

[LinkedIn – Azure IaaS Deployment Tool now Available in Azure Automation](https://www.linkedin.com/pulse/azure-iaas-deployment-tool-now-available-automation-gallery-lewis?trk=hp-feed-article-title-publish)

[LinkedIn – Chocolatey and Desired State Configuration](https://www.linkedin.com/pulse/sweetest-surprise-week-chocolatey-desired-state-john-n-lewis?trk=hp-feed-article-title-publish)

[LinkedIn – Azure Automation and DSC Module Deployment](https://www.linkedin.com/pulse/azure-automation-desired-state-configuration-module-deployment-lewis?trk=hp-feed-article-title-publish)

[LinkedIn – Deploying VNET to VNET Peering in Azure](https://www.linkedin.com/pulse/deploying-vnet-peering-using-azure-powershell-john-n-lewis)

[LinkedIn – Solving your current automation login challenges using Add-AzureRmAccount](https://www.linkedin.com/pulse/solving-your-automation-login-challenges-azure-forever-john-n-lewis?trk=hp-feed-article-title-publish)

[LinkedIn – A few of the questions you should ask your dev/ops folks before cloud migration](https://www.linkedin.com/pulse/what-few-things-you-need-ask-your-devops-folks-understand-lewis?trk=mp-author-card)

[LinkedIn – The Value Dev/Ops provides through scripted automation solutions](https://www.linkedin.com/pulse/building-provisioning-automation-azure-iaas-arm-using-john-n-lewis-8?trk=mp-author-card)

[LinkedIn – Why take the time to provide PowerShell Scripts for Azure?](https://www.linkedin.com/pulse/why-take-time-provide-powershell-scripts-azure-when-lots-lewis?trk=mp-author-card)

 

**Examples**

AZRM-VMDeploy.ps1 -ActionType Create -vmstrtype managed -vm pf001 -image pfsense -rg ResGroup1 -vnetrg ResGroup2 -addvnet -vnet VNET -sub1 3 -sub2 4 -ConfigIPs DualPvtNoPub -Nic1 10.120.2.7 -Nic2 10.120.3.7

 

AZRM-VMDeploy.ps1 -ActionType Create -vm red76 -image red67 -vmstrtype managed -rg ResGroup1 -vnetrg ResGroup2 -vnet VNET -sub1 7 -ConfigIPs SinglePvtNoPub -Nic1 10.120.6.124 -Ext linuxchefagent -addssh

 

AZRM-VMDeploy.ps1 -ActionType Create -vm win006 -image w2k12 -vmstrtype managed -rg ResGroup1 -vnetrg ResGroup2 -storerg storagerg -vnet VNET -sub1 2 -ConfigIPs Single -CreateNSG -NSGName NSG

 

AZRM-VMDeploy.ps1 -ActionType Create -vm win008 -image w2k16 -vmstrtype managed -rg ResGroup1 -vnetrg ResGroup2 -vnet VNET -sub1 5 -ConfigIPs PvtSingleStat -Nic1 172.10.4.169 -AddFQDN -fqdn mydns1

 

AZRM-VMDeploy.ps1 -ActionType Create -vm ubu004 -image ubuntu -vmstrtype managed -RG ResGroup1 -vnetrg ResGroup2 -VNet VNET -ConfigIPs Single -AddLB -LBType external -LBSubnet 2 -CreateLoadBalancer -LBName mylb -AddAvailabilitySet -AvailSetName myavsetname

 

AZRM-VMDeploy.ps1 -ActionType Create -vm ubu004 -image ubuntu -vmstrtype managed -RG ResGroup1 -vnetrg ResGroup2 -VNet VNET -ConfigIPs Single -AddLB -LBType internal -LBPvtIP 172.10.4.15 -CreateLoadBalancer -LBName mylb -AddAvailabilitySet -AvailSetName myavsetname

 


        
    
TechNet gallery is retiring! This script was migrated from TechNet script center to GitHub by Microsoft Azure Automation product group. All the Script Center fields like Rating, RatingCount and DownloadCount have been carried over to Github as-is for the migrated scripts only. Note : The Script Center fields will not be applicable for the new repositories created in Github & hence those fields will not show up for new Github repositories.
