**Azure Web Application and VM Deployment Project using Terraform and Azure DevOps**

**Project Overview**

> This project deploys a web application and a virtual machine on Azure using Infrastructure as Code (Terraform), monitored using Azure Monitor, and deployed via Azure DevOps.
> The architecture ensures private communication, secure access, diagnostics logging, and alerting for operational awareness.


**Step 1: Set Up Azure Resources Planning**

**Region:** Central India

**Resource Groups:**
rg-dev-network-01 (for network resources),  rg-dev-application-01 (for application resources)

**Subnet Planning:**
snet-dev-web, snet-dev-app, snet-dev-data, snet-dev-pep



**Step 2: Terraform Setup (Local or Repo)**

**1️⃣ Create Directory Structure**

**Run the following commands in your terminal to set up the directories:**

> mkdir azure-project && cd azure-project
>
> mkdir modules main tfstate
 
**This creates:**

azure-project/ → Your root directory.

modules/ → Where you store reusable Terraform modules.

main/ → Contains your primary Terraform configuration files.

tfstate/ → Stores Terraform state files (best to use remote storage in production).

 **2️⃣ Create Main Terraform Files**
 
**Inside the main/ directory, create the following files:**

> cd main
>
> touch main.tf variables.tf outputs.tf providers.tf

**What Each File Does**

> main.tf → Your core Terraform configurations (resources, modules, etc.).
> 
> variables.tf → Defines input variables for your Terraform setup.
> 
> outputs.tf → Stores output values that Terraform exposes after deployment.
> 
> providers.tf → Specifies which cloud provider (Azure, AWS, etc.) and authentication settings to use.
