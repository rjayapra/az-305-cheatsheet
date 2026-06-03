# AZ-104 Resource Creation Reference Guide (Azure CLI + PowerShell)

This guide provides exam-focused resource creation commands for AZ-104 using both Azure CLI and Azure PowerShell.

## 1. Setup and Authentication

### Azure CLI
```bash
# Install check
az version

# Sign in
az login

# List subscriptions
az account list --output table

# Set active subscription
az account set --subscription "<subscription-name-or-id>"

# Confirm active subscription
az account show --output table
```

### Azure PowerShell
```powershell
# Install Az module (first time)
Install-Module -Name Az -Scope CurrentUser -Repository PSGallery -Force

# Import module
Import-Module Az

# Sign in
Connect-AzAccount

# List subscriptions
Get-AzSubscription

# Set active subscription
Set-AzContext -Subscription "<subscription-name-or-id>"

# Confirm context
Get-AzContext
```

## 2. Common Variables

Use consistent names to avoid mistakes.

### Azure CLI
```bash
LOCATION="eastus"
RG="rg-az104-lab"
VNET="vnet-az104"
SUBNET="subnet-frontend"
NSG="nsg-az104"
VM="vm-az104"
STG="staz104demo001"
KV="kv-az104-demo-001"
LAW="law-az104-demo"
RSV="rsv-az104-demo"
```

### Azure PowerShell
```powershell
$Location = "EastUS"
$RgName = "rg-az104-lab"
$VnetName = "vnet-az104"
$SubnetName = "subnet-frontend"
$NsgName = "nsg-az104"
$VmName = "vm-az104"
$StorageName = "staz104demo001"
$KvName = "kv-az104-demo-001"
$LawName = "law-az104-demo"
$RsvName = "rsv-az104-demo"
```

## 3. Resource Groups, Tags, and Locks

### Azure CLI
```bash
# Create resource group
az group create --name $RG --location $LOCATION

# Add/update tags
az group update --name $RG --set tags.Environment=Lab tags.Owner=AZ104

# Add lock (CanNotDelete)
az lock create --name "lock-rg-delete" --lock-type CanNotDelete --resource-group $RG
```

### Azure PowerShell
```powershell
# Create resource group
New-AzResourceGroup -Name $RgName -Location $Location

# Add/update tags
Set-AzResourceGroup -Name $RgName -Tag @{ Environment = "Lab"; Owner = "AZ104" }

# Add lock (CanNotDelete)
New-AzResourceLock -LockName "lock-rg-delete" -LockLevel CanNotDelete -ResourceGroupName $RgName
```

## 4. Microsoft Entra ID and RBAC (Core Creation Tasks)

### Azure CLI
```bash
# Create user
az ad user create \
  --display-name "AZ104 User1" \
  --user-principal-name "az104user1@<tenant-domain>" \
  --password "<StrongPassword!123>"

# Create group
az ad group create --display-name "AZ104-Admins" --mail-nickname "az104-admins"

# Assign RBAC role at RG scope
az role assignment create \
  --assignee "<user-object-id-or-upn>" \
  --role "Contributor" \
  --scope "/subscriptions/<subscription-id>/resourceGroups/$RG"

# Create user-assigned managed identity
az identity create --name "uami-az104" --resource-group $RG --location $LOCATION
```

### Azure PowerShell
```powershell
# Create user
$PasswordProfile = New-Object -TypeName Microsoft.Open.AzureAD.Model.PasswordProfile
$PasswordProfile.Password = "<StrongPassword!123>"
New-AzADUser -DisplayName "AZ104 User1" -UserPrincipalName "az104user1@<tenant-domain>" -PasswordProfile $PasswordProfile -MailNickname "az104user1"

# Create group
$Group = New-AzADGroup -DisplayName "AZ104-Admins" -MailNickname "az104-admins"

# Assign RBAC role at RG scope
New-AzRoleAssignment -SignInName "az104user1@<tenant-domain>" -RoleDefinitionName "Contributor" -ResourceGroupName $RgName

# Create user-assigned managed identity
New-AzUserAssignedIdentity -Name "uami-az104" -ResourceGroupName $RgName -Location $Location
```

## 5. Storage Resources

## 5.1 Storage Account, Container, and Blob

### Azure CLI
```bash
# Create storage account
az storage account create \
  --name $STG \
  --resource-group $RG \
  --location $LOCATION \
  --sku Standard_LRS \
  --kind StorageV2

# Create blob container using Entra auth
az storage container create --name "data" --account-name $STG --auth-mode login

# Upload a blob
az storage blob upload --account-name $STG --container-name "data" --name "sample.txt" --file "./sample.txt" --auth-mode login
```

### Azure PowerShell
```powershell
# Create storage account
$Storage = New-AzStorageAccount -ResourceGroupName $RgName -Name $StorageName -Location $Location -SkuName Standard_LRS -Kind StorageV2

# Create blob container
$Ctx = $Storage.Context
New-AzStorageContainer -Name "data" -Context $Ctx

# Upload a blob
Set-AzStorageBlobContent -File ".\sample.txt" -Container "data" -Blob "sample.txt" -Context $Ctx
```

## 5.2 Azure File Share

### Azure CLI
```bash
az storage share-rm create --storage-account $STG --resource-group $RG --name "profiles" --quota 100
```

### Azure PowerShell
```powershell
New-AzRmStorageShare -ResourceGroupName $RgName -StorageAccountName $StorageName -Name "profiles" -QuotaGiB 100
```

## 6. Networking Resources

## 6.1 VNet and Subnet

### Azure CLI
```bash
az network vnet create \
  --resource-group $RG \
  --name $VNET \
  --address-prefixes 10.10.0.0/16 \
  --subnet-name $SUBNET \
  --subnet-prefixes 10.10.1.0/24

az network vnet subnet create \
  --resource-group $RG \
  --vnet-name $VNET \
  --name "subnet-backend" \
  --address-prefixes 10.10.2.0/24
```

### Azure PowerShell
```powershell
$SubnetConfig1 = New-AzVirtualNetworkSubnetConfig -Name $SubnetName -AddressPrefix "10.10.1.0/24"
$Vnet = New-AzVirtualNetwork -Name $VnetName -ResourceGroupName $RgName -Location $Location -AddressPrefix "10.10.0.0/16" -Subnet $SubnetConfig1

Add-AzVirtualNetworkSubnetConfig -Name "subnet-backend" -VirtualNetwork $Vnet -AddressPrefix "10.10.2.0/24"
$Vnet | Set-AzVirtualNetwork
```

## 6.2 NSG and Rules

### Azure CLI
```bash
# Create NSG
az network nsg create --resource-group $RG --name $NSG

# Allow RDP
az network nsg rule create \
  --resource-group $RG \
  --nsg-name $NSG \
  --name "Allow-RDP" \
  --priority 1000 \
  --direction Inbound \
  --access Allow \
  --protocol Tcp \
  --source-address-prefixes "*" \
  --source-port-ranges "*" \
  --destination-address-prefixes "*" \
  --destination-port-ranges 3389

# Associate NSG to subnet
az network vnet subnet update --resource-group $RG --vnet-name $VNET --name $SUBNET --network-security-group $NSG
```

### Azure PowerShell
```powershell
# Create NSG
$Nsg = New-AzNetworkSecurityGroup -ResourceGroupName $RgName -Name $NsgName -Location $Location

# Create and add RDP rule
$Rule = New-AzNetworkSecurityRuleConfig -Name "Allow-RDP" -Protocol Tcp -Direction Inbound -Priority 1000 -SourceAddressPrefix * -SourcePortRange * -DestinationAddressPrefix * -DestinationPortRange 3389 -Access Allow
$Nsg.SecurityRules.Add($Rule)
$Nsg | Set-AzNetworkSecurityGroup

# Associate NSG to subnet
$Vnet = Get-AzVirtualNetwork -Name $VnetName -ResourceGroupName $RgName
Set-AzVirtualNetworkSubnetConfig -Name $SubnetName -VirtualNetwork $Vnet -AddressPrefix "10.10.1.0/24" -NetworkSecurityGroup $Nsg
$Vnet | Set-AzVirtualNetwork
```

## 6.3 Public IP and NIC

### Azure CLI
```bash
az network public-ip create --resource-group $RG --name "pip-$VM" --sku Standard

az network nic create \
  --resource-group $RG \
  --name "nic-$VM" \
  --vnet-name $VNET \
  --subnet $SUBNET \
  --network-security-group $NSG \
  --public-ip-address "pip-$VM"
```

### Azure PowerShell
```powershell
$Pip = New-AzPublicIpAddress -Name "pip-$VmName" -ResourceGroupName $RgName -Location $Location -AllocationMethod Static -Sku Standard
$Nic = New-AzNetworkInterface -Name "nic-$VmName" -ResourceGroupName $RgName -Location $Location -SubnetId (Get-AzVirtualNetwork -Name $VnetName -ResourceGroupName $RgName).Subnets[0].Id -PublicIpAddressId $Pip.Id -NetworkSecurityGroupId (Get-AzNetworkSecurityGroup -Name $NsgName -ResourceGroupName $RgName).Id
```

## 7. Compute Resources

## 7.1 Virtual Machine

### Azure CLI
```bash
az vm create \
  --resource-group $RG \
  --name $VM \
  --nics "nic-$VM" \
  --image Win2022Datacenter \
  --admin-username azureadmin \
  --admin-password "<StrongPassword!123>" \
  --size Standard_B2s
```

### Azure PowerShell
```powershell
$Cred = Get-Credential
$VmConfig = New-AzVMConfig -VMName $VmName -VMSize "Standard_B2s"
$VmConfig = Set-AzVMOperatingSystem -VM $VmConfig -Windows -ComputerName $VmName -Credential $Cred -ProvisionVMAgent -EnableAutoUpdate
$VmConfig = Set-AzVMSourceImage -VM $VmConfig -PublisherName "MicrosoftWindowsServer" -Offer "WindowsServer" -Skus "2022-datacenter-azure-edition" -Version "latest"
$VmConfig = Add-AzVMNetworkInterface -VM $VmConfig -Id $Nic.Id
New-AzVM -ResourceGroupName $RgName -Location $Location -VM $VmConfig
```

## 7.2 Managed Disk and Snapshot

### Azure CLI
```bash
az disk create --resource-group $RG --name "disk-data01" --size-gb 64 --sku Standard_LRS

az vm disk attach --resource-group $RG --vm-name $VM --name "disk-data01"

az snapshot create --resource-group $RG --name "snap-disk-data01" --source "disk-data01"
```

### Azure PowerShell
```powershell
$DiskConfig = New-AzDiskConfig -Location $Location -CreateOption Empty -DiskSizeGB 64 -SkuName Standard_LRS
$Disk = New-AzDisk -ResourceGroupName $RgName -DiskName "disk-data01" -Disk $DiskConfig

$VmObj = Get-AzVM -ResourceGroupName $RgName -Name $VmName
$VmObj = Add-AzVMDataDisk -VM $VmObj -Name $Disk.Name -ManagedDiskId $Disk.Id -Lun 1 -CreateOption Attach
Update-AzVM -ResourceGroupName $RgName -VM $VmObj

$SnapConfig = New-AzSnapshotConfig -SourceUri $Disk.Id -Location $Location -CreateOption Copy
New-AzSnapshot -ResourceGroupName $RgName -SnapshotName "snap-disk-data01" -Snapshot $SnapConfig
```

## 7.3 Virtual Machine Scale Set (VMSS)

### Azure CLI
```bash
az vmss create \
  --resource-group $RG \
  --name "vmss-az104" \
  --image Ubuntu2204 \
  --admin-username azureuser \
  --generate-ssh-keys \
  --instance-count 2 \
  --upgrade-policy-mode automatic
```

### Azure PowerShell
```powershell
$VmssConfig = New-AzVmssConfig -Location $Location -SkuCapacity 2 -SkuName "Standard_B2s" -UpgradePolicyMode Automatic
Set-AzVmssStorageProfile -VirtualMachineScaleSet $VmssConfig -ImageReferencePublisher "Canonical" -ImageReferenceOffer "0001-com-ubuntu-server-jammy" -ImageReferenceSku "22_04-lts-gen2" -ImageReferenceVersion "latest"
Set-AzVmssOsProfile -VirtualMachineScaleSet $VmssConfig -AdminUsername "azureuser" -ComputerNamePrefix "vmssaz104"
New-AzVmss -ResourceGroupName $RgName -Name "vmss-az104" -VirtualMachineScaleSet $VmssConfig
```

## 8. App Service Resources

### Azure CLI
```bash
# Create App Service plan
az appservice plan create --name "asp-az104" --resource-group $RG --location $LOCATION --sku S1 --is-linux

# Create web app
az webapp create --resource-group $RG --plan "asp-az104" --name "<unique-webapp-name>" --runtime "DOTNETCORE:8.0"
```

### Azure PowerShell
```powershell
# Create App Service plan
New-AzAppServicePlan -Name "asp-az104" -Location $Location -ResourceGroupName $RgName -Tier "Standard" -WorkerSize "Small" -Linux

# Create web app
New-AzWebApp -Name "<unique-webapp-name>" -ResourceGroupName $RgName -Location $Location -AppServicePlan "asp-az104"
```

## 9. Key Vault

### Azure CLI
```bash
# Create Key Vault
az keyvault create --name $KV --resource-group $RG --location $LOCATION

# Create a secret
az keyvault secret set --vault-name $KV --name "DbPassword" --value "<StrongPassword!123>"
```

### Azure PowerShell
```powershell
# Create Key Vault
New-AzKeyVault -Name $KvName -ResourceGroupName $RgName -Location $Location

# Create a secret
$SecretValue = ConvertTo-SecureString "<StrongPassword!123>" -AsPlainText -Force
Set-AzKeyVaultSecret -VaultName $KvName -Name "DbPassword" -SecretValue $SecretValue
```

## 10. Monitoring Resources

## 10.1 Log Analytics Workspace

### Azure CLI
```bash
az monitor log-analytics workspace create --resource-group $RG --workspace-name $LAW --location $LOCATION
```

### Azure PowerShell
```powershell
New-AzOperationalInsightsWorkspace -ResourceGroupName $RgName -Name $LawName -Location $Location -Sku PerGB2018
```

## 10.2 Action Group and Metric Alert

### Azure CLI
```bash
# Create action group
az monitor action-group create \
  --resource-group $RG \
  --name "ag-az104" \
  --short-name "ag104"

# Create CPU alert for VM
VM_ID=$(az vm show --resource-group $RG --name $VM --query id -o tsv)
az monitor metrics alert create \
  --name "HighCPU-VM" \
  --resource-group $RG \
  --scopes $VM_ID \
  --condition "avg Percentage CPU > 80" \
  --window-size 5m \
  --evaluation-frequency 1m \
  --action "ag-az104"
```

### Azure PowerShell
```powershell
# Create action group
$EmailReceiver = New-AzActionGroupReceiver -Name "admin" -EmailReceiver -EmailAddress "admin@contoso.com"
Set-AzActionGroup -Name "ag-az104" -ResourceGroupName $RgName -ShortName "ag104" -Receiver $EmailReceiver

# Create CPU alert for VM
$VmObj = Get-AzVM -ResourceGroupName $RgName -Name $VmName
$Condition = New-AzMetricAlertRuleV2Criteria -MetricName "Percentage CPU" -TimeAggregation Average -Operator GreaterThan -Threshold 80
Add-AzMetricAlertRuleV2 -Name "HighCPU-VM" -ResourceGroupName $RgName -WindowSize 00:05:00 -Frequency 00:01:00 -TargetResourceId $VmObj.Id -Condition $Condition -ActionGroupId (Get-AzActionGroup -ResourceGroupName $RgName -Name "ag-az104").Id
```

## 11. Backup Resources

### Azure CLI
```bash
# Create Recovery Services vault
az backup vault create --resource-group $RG --name $RSV --location $LOCATION

# Set vault redundancy
az backup vault backup-properties set --name $RSV --resource-group $RG --backup-storage-redundancy LocallyRedundant

# Enable backup for VM using default policy
az backup protection enable-for-vm --resource-group $RG --vault-name $RSV --vm $VM --policy-name DefaultPolicy
```

### Azure PowerShell
```powershell
# Create Recovery Services vault
$Vault = New-AzRecoveryServicesVault -Name $RsvName -ResourceGroupName $RgName -Location $Location

# Set vault context
Set-AzRecoveryServicesVaultContext -Vault $Vault

# Set vault redundancy
Set-AzRecoveryServicesBackupProperty -Vault $Vault -BackupStorageRedundancy LocallyRedundant

# Enable backup for VM with default policy
$Policy = Get-AzRecoveryServicesBackupProtectionPolicy -Name "DefaultPolicy"
$Container = Get-AzRecoveryServicesBackupContainer -ContainerType "AzureVM" -FriendlyName $VmName -Status "Registered"
$Item = Get-AzRecoveryServicesBackupItem -Container $Container -WorkloadType "AzureVM"
Enable-AzRecoveryServicesBackupProtection -Policy $Policy -Item $Item
```

## 12. Useful Validation Commands

### Azure CLI
```bash
az resource list --resource-group $RG --output table
az vm list --resource-group $RG --output table
az storage account list --resource-group $RG --output table
az network vnet list --resource-group $RG --output table
```

### Azure PowerShell
```powershell
Get-AzResource -ResourceGroupName $RgName
Get-AzVM -ResourceGroupName $RgName -Status
Get-AzStorageAccount -ResourceGroupName $RgName
Get-AzVirtualNetwork -ResourceGroupName $RgName
```

## 13. Cleanup Commands

### Azure CLI
```bash
az group delete --name $RG --yes --no-wait
```

### Azure PowerShell
```powershell
Remove-AzResourceGroup -Name $RgName -Force -AsJob
```

## 14. AZ-104 Exam Notes for Command Questions

- Prefer least-privilege RBAC role assignments.
- Use scoped resource IDs for role assignments and locks.
- For storage data operations in CLI, `--auth-mode login` avoids key-based auth.
- Remember unique naming rules for global services (storage account, web app, key vault).
- Know both create and verification commands; many questions test post-deployment validation.
