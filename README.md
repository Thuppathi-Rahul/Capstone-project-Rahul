> **Azure Web Application & VM Deployment Project**

**Project Overview**
This project automates the deployment of a secure web application and virtual machine (VM) on Microsoft Azure using Infrastructure as Code (Terraform). The architecture includes private networking, monitoring with Azure Monitor, and deployment via Azure DevOps pipelines.

Key features:

Private connectivity for PaaS services using Private Endpoints

Secure VM access restricted to specific IP addresses

Comprehensive monitoring with Application Insights and Log Analytics

Alerting for critical events like VM shutdown

Resource locks to prevent accidental deletion

Architecture Components
Architecture Diagram (Placeholder for actual diagram)

Core Resources Deployed
Networking

Virtual Network with multiple subnets (web, app, data, private endpoints)

Network Security Groups with restrictive rules

Private DNS Zones for private endpoint resolution

Compute

Ubuntu 22.04 LTS Virtual Machine

App Service Plan and Linux Web App

Monitoring & Management

Log Analytics Workspace

Application Insights

Diagnostic Settings for all resources

Alert rules with email notifications

Security

Resource locks

NSG rules restricting access

Private endpoints for secure connectivity

Implementation Steps
1. Infrastructure Planning
Region: Central India
Resource Groups:

rg-dev-network-01 - For network-related resources

rg-dev-application-01 - For application resources

Subnet Structure:

Subnet Name	Address Range	Purpose
snet-dev-web	10.1.0.0/22	Web-facing services
snet-dev-app	10.1.4.0/22	Application services
snet-dev-data	10.1.8.0/22	Database services
snet-dev-pep	10.1.12.0/22	Private Endpoints
2. Terraform Setup
Directory Structure:

azure-project/
├── modules/       # Reusable Terraform modules
├── main/          # Primary configuration files
│   ├── main.tf
│   ├── variables.tf
│   ├── outputs.tf
│   └── providers.tf
└── tfstate/       # For state files (local - use remote in production)
3. Key Terraform Configurations
Network Security:

SSH access restricted to a single IP address

Deny all other SSH attempts

Separate NSGs for each subnet

Private Connectivity:

Private DNS Zone (privatelink.azurewebsites.net)

Private Endpoint for Web App

VNet integration for DNS resolution

Monitoring:

Application Insights integrated with Web App

Log Analytics workspace collecting:

Web App HTTP logs

VM metrics

NSG flow logs

Diagnostic settings for all resources

Alerting:

Email alert when VM CPU drops below 1% (indicating shutdown)

Action group configured with admin email

4. Azure DevOps Pipeline
Pipeline Components:

Terraform Installation - Installs required Terraform version

Initialization - terraform init

Validation - terraform validate

Planning - terraform plan

Application - terraform apply

Pipeline YAML Highlights:

yaml
steps:
- task: TerraformInstaller@0
  displayName: 'Install Terraform'
  
- task: TerraformCLI@0
  displayName: 'Terraform init'
  
- task: TerraformCLI@0
  displayName: 'Terraform validate'
  
- task: TerraformCLI@0
  displayName: 'Terraform plan'
  
- task: TerraformCLI@0
  displayName: 'Terraform apply'
5. Verification Steps
Resource Validation

Confirm all resources are provisioned in correct resource groups

Verify subnet assignments and NSG associations

Connectivity Tests

SSH to VM (only from allowed IP)

Verify web app accessibility through private endpoint

Monitoring Verification

Check Application Insights for web app data

Validate diagnostic settings for all resources

Test alert by stopping VM

Security Checks

Confirm resource locks are in place

Verify NSG rules are properly restricting access

KQL Query for Log Analysis
kql
// KQL query to view logs for last 24 hours
AzureActivity
| where TimeGenerated > ago(24h)
| project TimeGenerated, OperationName, Caller, ResourceGroup, Resource
| order by TimeGenerated desc
Outputs
After successful deployment, the following outputs are available:

Web App Private FQDN: [webapp-name].azurewebsites.net

Web App Private Endpoint IP: [private-ip]

VM Public IP: [public-ip]

SSH Command: ssh azureuser@[public-ip]

Cost Considerations
The implementation uses free-tier eligible SKUs where possible:

VM: Standard_B1s (burstable)

App Service Plan: S1 (shared)

Log Analytics: PerGB2018 pricing tier

Maintenance & Next Steps
Production Recommendations:

Use remote state storage (Azure Storage)

Implement Terraform workspaces for environments

Add more granular monitoring and alerting

Scaling Options:

Upgrade VM and App Service SKUs

Implement auto-scaling rules

Add additional monitoring solutions

Security Enhancements:

Implement Azure Policy for compliance

Add network watcher for advanced monitoring

Configure Azure Security Center

Troubleshooting
Common issues and solutions:

SSH Access Problems

Verify your public IP matches the allowed IP in NSG rules

Check VM boot diagnostics for startup issues

Web App Connectivity

Validate private endpoint connection status

Check DNS resolution in the VNet

Alert Notifications

Verify email address in action group

Check alert rule criteria matches expected conditions
