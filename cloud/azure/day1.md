Day 1 — Azure SSH Key Pair
🎯 Task

Create an RSA SSH key pair named xfusion-kp in Azure.

🧠 Concepts

Azure CLI

Resource groups

SSH key pairs

RSA keys

Public vs private keys

🔧 Commands
# Check Azure authentication
az account show

# Find resource groups
az group list --output table

# Create SSH key
az sshkey create \
  --name xfusion-kp \
  --resource-group kml_rg_main-e0a9ad31e60c4363

# Verify
az sshkey show \
  --name xfusion-kp \
  --resource-group kml_rg_main-e0a9ad31e60c4363

💡 Key takeaway

ssh-rsa indicates an RSA SSH public key.
The private key must be kept secret.