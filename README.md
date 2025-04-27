**Azure Web Application and VM Deployment Project using Terraform and Azure DevOps**

**Project Overview**
```
 This project deploys a web application and a virtual machine on Azure using Infrastructure as Code (Terraform), monitored using Azure Monitor, and deployed via Azure DevOps.
 The architecture ensures private communication, secure access, diagnostics logging, and alerting for operational awareness.
```

**Step 1: Set Up Azure Resources Planning**
 
**Region:** Central India

**Resource Groups:**
rg-dev-network-01 (for network resources),  rg-dev-application-01 (for application resources)

**Subnet Planning:**

snet-dev-web, snet-dev-app, snet-dev-data, snet-dev-pep



**Step 2: Terraform Setup (Local or Repo)**

**1️⃣ Create Directory Structure**

**Run the following commands in your terminal to set up the directories:**
```
mkdir azure-project && cd azure-project

mkdir modules main tfstate
 ```
**This creates:**
 ```
azure-project/ → Your root directory.

modules/ → Where you store reusable Terraform modules.

main/ → Contains your primary Terraform configuration files.

tfstate/ → Stores Terraform state files (best to use remote storage in production).
 ```

 **2️⃣ Create Main Terraform Files**
 
**Inside the main/ directory, create the following files:**
 ```
 cd main
 touch main.tf variables.tf outputs.tf providers.tf
 ```
**What Each File Does**
 ```
 main.tf → Your core Terraform configurations (resources, modules, etc.).
 
 variables.tf → Defines input variables for your Terraform setup.
 
outputs.tf → Stores output values that Terraform exposes after deployment.

 providers.tf → Specifies which cloud provider (Azure, AWS, etc.) and authentication settings to use.

 ```
**Step 3: Terraform Code**

# **Create a providers.tf file Provide which platform you are using  like Azure or Aws or Gcp in the providers file such that terraform will call the api of that platforms **
**providers.tf**





```

terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 4.0"  # Ensures Terraform uses version 4.x
    }
  }
}

provider "azurerm" {
  features {}
}
```



**main.tf** 




```
resource "azurerm_resource_group" "network" {
  name     = "rg-dev-network-01"
  location = "Central India"
}

resource "azurerm_virtual_network" "vnet" {
  name                = "vnet-dev-01"
  address_space       = ["10.1.0.0/20"]
  location            = azurerm_resource_group.network.location
  resource_group_name = azurerm_resource_group.network.name
}

resource "azurerm_subnet" "web" {
  name                 = "snet-dev-web"
  resource_group_name  = azurerm_resource_group.network.name
  virtual_network_name = azurerm_virtual_network.vnet.name
  address_prefixes     = ["10.1.0.0/22"]
}

resource "azurerm_network_security_group" "web_nsg" {
  name                = "nsg-snet-dev-web"
  location            = azurerm_resource_group.network.location
  resource_group_name = azurerm_resource_group.network.name
}

resource "azurerm_subnet" "app" {
  name                 = "snet-dev-app"
  resource_group_name  = azurerm_resource_group.network.name
  virtual_network_name = azurerm_virtual_network.vnet.name
  address_prefixes     = ["10.1.4.0/22"]
}

resource "azurerm_network_security_group" "app_nsg" {
  name                = "nsg-snet-dev-app"
  location            = azurerm_resource_group.network.location
  resource_group_name = azurerm_resource_group.network.name
}

resource "azurerm_subnet" "data" {
  name                 = "snet-dev-data"
  resource_group_name  = azurerm_resource_group.network.name
  virtual_network_name = azurerm_virtual_network.vnet.name
  address_prefixes     = ["10.1.8.0/22"]
}

resource "azurerm_network_security_group" "data_nsg" {
  name                = "nsg-snet-dev-data"
  location            = azurerm_resource_group.network.location
  resource_group_name = azurerm_resource_group.network.name
}


resource "azurerm_subnet" "pep" {
  name                 = "snet-dev-pep"
  resource_group_name  = azurerm_resource_group.network.name
  virtual_network_name = azurerm_virtual_network.vnet.name
  address_prefixes     = ["10.1.12.0/22"]
}

resource "azurerm_network_security_group" "pep_nsg" {
  name                = "nsg-snet-dev-pep"
  location            = azurerm_resource_group.network.location
  resource_group_name = azurerm_resource_group.network.name
}


resource "azurerm_public_ip" "vm_ip" {
  name                = "pip-dev-vm"
  location            = azurerm_resource_group.network.location
  resource_group_name = azurerm_resource_group.network.name
  allocation_method   = "Dynamic"
  sku                 = "Basic"
}

resource "azurerm_network_interface" "dev_vm_nic" {
  name                = "nic-dev-vm"
  location            = azurerm_resource_group.network.location
  resource_group_name = azurerm_resource_group.network.name

  ip_configuration {
    name                          = "internal"
    subnet_id                     = azurerm_subnet.web.id
    private_ip_address_allocation = "Dynamic"
    public_ip_address_id          = azurerm_public_ip.vm_ip.id
  }
}

resource "azurerm_linux_virtual_machine" "dev_vm" {
  name                            = "dev-vm"
  location                        = azurerm_resource_group.network.location
  resource_group_name             = azurerm_resource_group.network.name
  network_interface_ids           = [azurerm_network_interface.dev_vm_nic.id]
  size                            = "Standard_B1s"
  admin_username                  = "azureuser"
  disable_password_authentication = true

  admin_ssh_key {
    username   = "azureuser"
    public_key = file("~/.ssh/id_rsa.pub")  # Point to your public key
  }

  os_disk {
    caching              = "ReadWrite"
    storage_account_type = "Standard_LRS"
    name                 = "dev-os-disk"
  }

  source_image_reference {
    publisher = "Canonical"
    offer     = "0001-com-ubuntu-server-jammy"
    sku       = "22_04-lts"
    version   = "latest"
  }

  custom_data = filebase64("docker-install.sh")     # Create a docker_install.sh file in same folder(docker_install.sh is they in github files)
}





```
