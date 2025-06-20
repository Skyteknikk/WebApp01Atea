# WebApp01Atea
Atea task on deploying a webapp with Sql server with terraform
Here is a clean and organized version of the Terraform project to deploy a WebApp and SQL database on Azure, with GitHub Actions CI/CD:

📁 Final Project Structure
graphql
Copier
Modifier
webapp-sql-terraform/
│
├── main.tf                # Main infrastructure definitions
├── variables.tf           # All input variables
├── terraform.tfvars       # Actual values for variables
├── outputs.tf             # Outputs like WebApp URL
│
└── .github/
    └── workflows/
        └── deploy.yml     # GitHub Actions CI/CD workflow
🛠️ Terraform Files
main.tf
h
Copier
Modifier
provider "azurerm" {
  features {}
  client_id       = var.client_id
  client_secret   = var.client_secret
  subscription_id = var.subscription_id
  tenant_id       = var.tenant_id
}

resource "azurerm_resource_group" "main" {
  name     = var.resource_group_name
  location = var.location
}

resource "azurerm_app_service_plan" "app_service_plan" {
  name                = var.app_service_plan_name
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
  sku {
    tier = "Standard"
    size = "S1"
  }
}

resource "azurerm_app_service" "web_app" {
  name                = var.webapp_name
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
  app_service_plan_id = azurerm_app_service_plan.app_service_plan.id

  site_config {
    always_on = true
  }

  app_settings = {
    "WEBSITE_RUN_FROM_PACKAGE" = "1"
  }
}

resource "azurerm_mssql_server" "sql_server" {
  name                         = var.sql_server_name
  resource_group_name          = azurerm_resource_group.main.name
  location                     = azurerm_resource_group.main.location
  version                      = "12.0"
  administrator_login          = var.sql_admin
  administrator_login_password = var.sql_password
}

resource "azurerm_mssql_database" "sql_db" {
  name           = var.sql_db_name
  server_id      = azurerm_mssql_server.sql_server.id
  sku_name       = "S0"
  collation      = "SQL_Latin1_General_CP1_CI_AS"
  max_size_gb    = 2
}
variables.tf
hcl
Copier
Modifier
# Azure Authentication
variable "client_id" {}
variable "client_secret" {}
variable "subscription_id" {}
variable "tenant_id" {}

# Infrastructure Settings
variable "resource_group_name" {}
variable "location" { default = "East US" }

# Web App
variable "app_service_plan_name" {}
variable "webapp_name" {}

# SQL Database
variable "sql_server_name" {}
variable "sql_db_name" {}
variable "sql_admin" {}
variable "sql_password" {
  sensitive = true
}
terraform.tfvars
hcl
Copier
Modifier
# Azure auth (use GitHub secrets in CI/CD)
client_id        = "your-client-id"
client_secret    = "your-client-secret"
subscription_id  = "your-subscription-id"
tenant_id        = "your-tenant-id"

# Resources
resource_group_name     = "webapp-rg"
location                = "East US"
app_service_plan_name   = "webapp-service-plan"
webapp_name             = "mywebapp12345"
sql_server_name         = "sqlserver12345"
sql_db_name             = "webappdb"
sql_admin               = "sqladminuser"
sql_password            = "StrongP@ssword123"
outputs.tf
hcl
Copier
Modifier
output "webapp_url" {
  value = azurerm_app_service.web_app.default_site_hostname
}
🚀 GitHub Actions CI/CD
.github/workflows/deploy.yml
yaml
Copier
Modifier
name: Deploy WebApp & SQL with Terraform

on:
  push:
    branches:
      - main

jobs:
  terraform:
    name: 'Terraform Deployment'
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Code
        uses: actions/checkout@v3

      - name: Set up Terraform
        uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: 1.6.0

      - name: Terraform Init
        run: terraform init

      - name: Terraform Plan
        run: terraform plan -var-file="terraform.tfvars"

      - name: Terraform Apply
        run: terraform apply -auto-approve -var-file="terraform.tfvars"
🔐 GitHub Secrets Configuration
Go to your GitHub repo:

Settings → Secrets and variables → Actions → New repository secret

Add:

ARM_CLIENT_ID

ARM_CLIENT_SECRET

ARM_SUBSCRIPTION_ID

ARM_TENANT_ID

Then replace hardcoded auth values in terraform.tfvars with:

hcl
Copier
Modifier
client_id        = var.client_id
client_secret    = var.client_secret
subscription_id  = var.subscription_id
tenant_id        = var.tenant_id
In deploy.yml, inject secrets as environment variables if needed.

✅ Done
Now, whenever you push to the main branch, GitHub Actions will:

Run Terraform

Deploy the WebApp and SQL DB to Azure

Print the WebApp URL in the GitHub Actions log

Would you like a downloadable ZIP or a GitHub template repo version?




