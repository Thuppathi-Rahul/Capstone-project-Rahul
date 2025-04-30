**Azure Web Application and VM Deployment Project using Terraform and Azure DevOps**

**Project Overview**
```
 This project deploys a web application and a virtual machine on Azure using Infrastructure as Code (Terraform), monitored using Azure Monitor, and deployed via Azure DevOps.
 The architecture ensures private communication, secure access, diagnostics logging, and alerting for operational awareness.
```

**Step 1: Set Up Azure Resources Planning**
 
**Region:** Central Canada

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


**providers.tf**
>
># Create a providers.tf file provide which platform you are using  like Azure or Aws or Gcp in the providers file such that terraform will call the api of that platforms 
>


```


# Configure the Azure provider, you can have many
# if you use azurerm provider, it's source is hashicorp/azurerm
# short for registry.terraform.io/hashicorp/azurerm


terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 4.12.0"
    }
  }

  required_version = ">= 1.9.0"
}
# configures the provider

provider "azurerm" {
  features {}
  subscription_id = "000000000000000000000000000"
}

```



**main.tf** 




```
# Variables
variable "prefix" {
  default = "Rahul"
  type    = string
}

# Resource Group for Networking
resource "azurerm_resource_group" "network" {
  name     = "${var.prefix}-rg-dev-network"
  location = "canadacentral"
}
# Resource Group for Application
resource "azurerm_resource_group" "application" {
  name     = "${var.prefix}-rg-dev-application"
  location = "canadacentral"
}


# Virtual Network
resource "azurerm_virtual_network" "vnet" {
  name                = "${var.prefix}-vnet-dev"
  address_space       = ["10.1.0.0/20"]
  location            = azurerm_resource_group.network.location
  resource_group_name = azurerm_resource_group.network.name
}

# Subnets
resource "azurerm_subnet" "web" {
  name                 = "${var.prefix}-snet-dev-web"
  resource_group_name  = azurerm_resource_group.network.name
  virtual_network_name = azurerm_virtual_network.vnet.name
  address_prefixes     = ["10.1.0.0/22"]
}

resource "azurerm_subnet" "app" {
  name                 = "${var.prefix}-snet-dev-app"
  resource_group_name  = azurerm_resource_group.network.name
  virtual_network_name = azurerm_virtual_network.vnet.name
  address_prefixes     = ["10.1.4.0/22"]
}

resource "azurerm_subnet" "data" {
  name                 = "${var.prefix}-snet-dev-data"
  resource_group_name  = azurerm_resource_group.network.name
  virtual_network_name = azurerm_virtual_network.vnet.name
  address_prefixes     = ["10.1.8.0/22"]
}

resource "azurerm_subnet" "pep" {
  name                 = "${var.prefix}-snet-dev-pep"
  resource_group_name  = azurerm_resource_group.network.name
  virtual_network_name = azurerm_virtual_network.vnet.name
  address_prefixes     = ["10.1.12.0/22"]
}



resource "azurerm_network_security_group" "web_nsg" {
  name                = "${var.prefix}-nsg-snet-dev-web"
  location            = azurerm_resource_group.network.location
  resource_group_name = azurerm_resource_group.network.name
}

resource "azurerm_network_security_group" "app_nsg" {
  name                = "${var.prefix}-nsg-snet-dev-app"
  location            = azurerm_resource_group.network.location
  resource_group_name = azurerm_resource_group.network.name
}
resource "azurerm_network_security_rule" "allow_ssh_from_my_ip" {
  name                        = "Allow-SSH-From-My-IP"
  priority                    = 100
  direction                   = "Inbound"
  access                      = "Allow"
  protocol                    = "Tcp"
  source_port_range           = "*"
  destination_port_range      = "22"
  source_address_prefix       = "*"
  destination_address_prefix  = "*"
  resource_group_name         = azurerm_resource_group.network.name
  network_security_group_name = azurerm_network_security_group.web_nsg.name
}


resource "azurerm_network_security_group" "data_nsg" {
  name                = "${var.prefix}-nsg-snet-dev-data"
  location            = azurerm_resource_group.network.location
  resource_group_name = azurerm_resource_group.network.name
}

resource "azurerm_network_security_group" "pep_nsg" {
  name                = "${var.prefix}-nsg-snet-dev-pep"
  location            = azurerm_resource_group.network.location
  resource_group_name = azurerm_resource_group.network.name
}



# Associate NSGs with subnets
resource "azurerm_subnet_network_security_group_association" "web" {
  subnet_id                 = azurerm_subnet.web.id
  network_security_group_id = azurerm_network_security_group.web_nsg.id
}

resource "azurerm_subnet_network_security_group_association" "app" {
  subnet_id                 = azurerm_subnet.app.id
  network_security_group_id = azurerm_network_security_group.app_nsg.id
}

resource "azurerm_subnet_network_security_group_association" "data" {
  subnet_id                 = azurerm_subnet.data.id
  network_security_group_id = azurerm_network_security_group.data_nsg.id
}

resource "azurerm_subnet_network_security_group_association" "pep" {
  subnet_id                 = azurerm_subnet.pep.id
  network_security_group_id = azurerm_network_security_group.pep_nsg.id
}



# Public IP for VM
resource "azurerm_public_ip" "vm_ip" {
  name                = "${var.prefix}-pip-dev-vm"
  location            = azurerm_resource_group.application.location
  resource_group_name = azurerm_resource_group.application.name
  allocation_method   = "Static"
  sku                 = "Basic"
}

# NIC for VM
resource "azurerm_network_interface" "dev_vm_nic" {
  name                = "${var.prefix}-nic-dev-vm"
  location            = azurerm_resource_group.application.location
  resource_group_name = azurerm_resource_group.application.name

  ip_configuration {
    name                          = "internal"
    subnet_id                     = azurerm_subnet.web.id
    private_ip_address_allocation = "Dynamic"
    public_ip_address_id          = azurerm_public_ip.vm_ip.id
  }
}

# VM
resource "azurerm_linux_virtual_machine" "dev_vm" {
  name                  = "${var.prefix}-dev-vm"
  location              = azurerm_resource_group.application.location
  resource_group_name   = azurerm_resource_group.application.name
  network_interface_ids = [azurerm_network_interface.dev_vm_nic.id]
  size                  = "Standard_B1s"
  admin_username        = "azureuser"
  disable_password_authentication = true

  admin_ssh_key {
    username   = "azureuser"
    public_key = file("~/.ssh/id_rsa.pub")
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

  custom_data = filebase64("docker-install.sh")
}

# App service plan

resource "azurerm_service_plan" "asp" {
  name                = "${var.prefix}-asp"
  resource_group_name = azurerm_resource_group.application.name
  location            =  azurerm_resource_group.application.location
  os_type             = "Linux"
  sku_name            = "S1"
}

# Web app
resource "azurerm_linux_web_app" "webapp" {
  name                = "${var.prefix}-webapp"
  resource_group_name = azurerm_resource_group.application.name
  location            = azurerm_service_plan.asp.location
  service_plan_id     = azurerm_service_plan.asp.id

  site_config {
    application_stack {
      dotnet_version = "8.0" #Using dotnet for deploying web application
    }
  }

  app_settings = {
    "WEBSITES_ENABLE_APP_SERVICE_STORAGE" = "false"
  }
}
# from GitHub we are pulling the repo and runnning the web app

resource "azurerm_app_service_source_control" "scm" {
  app_id    = azurerm_linux_web_app.webapp.id
  repo_url  = "https://github.com/Thuppathi-Rahul/Rahul-capstone-webapp"  
  branch    = "main"
}



```
