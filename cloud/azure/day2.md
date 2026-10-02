# Day 2 — Create an Azure Virtual Machine

## 🎯 Task

Create an Azure VM named `xfusion-vm` in the existing resource group with the following requirements:

- Region: `centralus`
- Image: Ubuntu 24.04 LTS
- Size: `Standard_B1s`
- OS disk: 30 GB Standard HDD
- NSG: Allow inbound SSH on port 22
- Verify SSH access to the VM

## 🧠 Concepts

- Azure Virtual Machines
- Azure Resource Groups
- Azure CLI
- VM images
- VM sizes
- Managed disks
- Network Security Groups (NSG)
- SSH access
- Public IP addresses

## 🔧 Commands

# List existing resource groups
az group list --output table

# Create the VM
az vm create \
  --resource-group kml_rg_main-2aed8b7979e640a5 \
  --name xfusion-vm \
  --location centralus \
  --image Ubuntu2404 \
  --size Standard_B1s \
  --os-disk-size-gb 30 \
  --storage-sku Standard_LRS \
  --admin-username azureuser \
  --generate-ssh-keys

# Verify the VM
az vm show \
  --resource-group kml_rg_main-2aed8b7979e640a5 \
  --name xfusion-vm \
  --show-details \
  --output table

# Check disk size and type
az vm show \
  --resource-group kml_rg_main-2aed8b7979e640a5 \
  --name xfusion-vm \
  --query "storageProfile.osDisk.{Size:diskSizeGb,Sku:managedDisk.storageAccountType}" \
  --output table

# Get the public IP
az vm show \
  --resource-group kml_rg_main-2aed8b7979e640a5 \
  --name xfusion-vm \
  --show-details \
  --query publicIps \
  --output tsv

# SSH into the VM
ssh azureuser@<PUBLIC_IP>

# Verify the hostname
hostname

# Exit the VM
exit

# Find the NSG attached to the VM's NIC
az network nic list \
  --resource-group kml_rg_main-2aed8b7979e640a5 \
  --query "[].{NIC:name,NSG:networkSecurityGroup.id}" \
  --output table

# Check NSG rules
az network nsg rule list \
  --resource-group kml_rg_main-2aed8b7979e640a5 \
  --nsg-name xfusion-vmNSG \
  --output table

## 📝 Key Takeaway

`az vm create` can create an Azure VM with the required compute, storage, networking, and SSH configuration.

The important options are:

--location centralus
    Creates the VM in the Central US region.

--image Ubuntu2404
    Uses the Ubuntu 24.04 LTS image.

--size Standard_B1s
    Sets the VM size.

--os-disk-size-gb 30
    Creates a 30 GB OS disk.

--storage-sku Standard_LRS
    Uses Standard HDD storage.

--generate-ssh-keys
    Generates SSH keys for secure access.

After creation, always verify the VM and test SSH access.
