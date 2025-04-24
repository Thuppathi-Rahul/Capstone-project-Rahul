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
