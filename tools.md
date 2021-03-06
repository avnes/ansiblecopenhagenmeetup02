# Tools to automate Azure resources

List of tools I use (or have evaluated) to automate or manage Azure resources. There are probably more options out there, but this is just to give you an idea of the competition.

| Tool             | Vendor    | Documentation                                                                                                                 |
| ---------------- | --------- | ----------------------------------------------------------------------------------------------------------------------------- |
| Powershell       | Microsoft | <https://docs.microsoft.com/en-us/powershell/scripting/setup/installing-powershell-core-on-macos-and-linux?view=powershell-6> |
| Azure CLI        | Microsoft | <https://docs.microsoft.com/en-us/cli/azure/reference-index?view=azure-cli-latest>                                            |
| Azure Python API | Microsoft | <https://docs.microsoft.com/en-us/python/api/overview/azure/?view=azure-python>                                               |
| Terraform        | HashiCorp | <https://www.terraform.io/docs/providers/azurerm/index.html>                                                                  |
| Azure REST API   | Microsoft | <https://docs.microsoft.com/en-us/rest/api/?view=Azure>                                                                       |
| Ansible          | Red Hat   | <https://docs.ansible.com/ansible/latest/scenario_guides/guide_azure.html>                                                    |

I will provide a quick tour of the competition, so you have a reference when we discuss the options for Ansible.

## Powershell

Refer to <https://docs.microsoft.com/en-us/powershell/scripting/install/installing-powershell-core-on-linux?view=powershell-6#fedora> for how to install Powershell on Linux

### Install Azure module

```bash
sudo pwsh

pwsh> Install-Package -Name Az -Source https://www.powershellgallery.com/api/v2 -ProviderName NuGet -ExcludeVersion -Destination /usr/local/share/powershell/Modules
pwsh> install-module -name az
```

```bash
source ~/bin/azure.sh # which sets environment variables AZURE_CLIENT_ID, AZURE_CLIENT_SECRET, AZURE_TENANT_ID and AZURE_RM_NAME

pwsh

# Login to Azure using service-principal access
pwsh> import-module az

pwsh> Connect-AzAccount -Credential (New-Object -TypeName pscredential -ArgumentList $env:AZURE_CLIENT_ID, (ConvertTo-SecureString $env:AZURE_CLIENT_SECRET -AsPlainText -Force)) -ServicePrincipal -TenantId $env:AZURE_TENANT_ID;

# List all the App Services

pwsh> Get-AzWebApp -ResourceGroupName $env:AZURE_RM_NAME;
```

### Powershell Pros

* Open Source on <https://github.com/PowerShell/PowerShell>
* Can run on Windows, Linux and macOS

### Powershell Cons

* A bit alien syntax if you come from a Linux or macOS background.

## Azure CLI

```bash
hide_me=$(az login --service-principal \
--username $AZURE_CLIENT_ID \
--password $AZURE_CLIENT_SECRET \
--tenant $AZURE_TENANT_ID)

az configure --defaults group=$AZURE_RM_NAME

status=$(az webapp show --name ${APP_NAME} | grep state | cut -d ":" -f2 | cut -d "\"" -f2)

echo
echo "${APP_NAME} is ${status}"
echo
```

### Azure CLI Pros

* Open Source on <https://github.com/Azure/azure-cli>
* Well documented.
* Easy to template with Jinja2.
* Available on Linux as RPM and as Docker image.
* The "official" client for Azure management.
* Integrates well with the Linux shell

### Azure CLI Cons

* No guarantee for machine readable output. Some commands returns nothing, other returns plain text and finally some returns JSON objects. There is an option called "--output json", but that is still not a guarantee for getting a JSON object back.

## Azure Python API

```py
#!/usr/bin/env python

import os
from azure.common.credentials import ServicePrincipalCredentials
from azure.mgmt.web import WebSiteManagementClient

subscription_id = os.environ['AZURE_SUBSCRIPTION_ID']
group_name = os.environ['AZURE_RM_NAME']

credentials = ServicePrincipalCredentials(
    client_id=os.environ['AZURE_CLIENT_ID'],
    secret=os.environ['AZURE_CLIENT_SECRET'],
    tenant=os.environ['AZURE_TENANT_ID']
)

web_client = WebSiteManagementClient(credentials, subscription_id)

def get_web_client():
    return web_client

def get_web_operator():
    return get_web_client().web_apps


def get_app_services():
    return get_web_operator().list()


def get_app_services_names():
    names = []
    for service in get_app_services():
        names.append(service.name)
    names.sort()
    return names


def print_app_services_names():
    for name in get_app_services_names():
        print(name)


if __name__ == "__main__":
    print_app_services_names()
```

### Azure Python API Pros

* Open Source and available through <https://pypi.python.org/pypi/azure>
* Fairly easy to program against, using the full power of Python.

### Azure Python API Cons

* Documentation seems auto generated, and could be better.
* Missing some features, like specifying Local Git deployment option.

## Terraform

### main.tf

```hcl
resource "azurerm_virtual_network" "vnet" {
  name                = "VNET-${var.environment}"
  resource_group_name = "${var.resource_group_name}"
  address_space       = "${var.address_space}"
  location            = "${var.location}"
}
```

### variables.tf

```hcl
variable "resource_group_name" {}

variable "location" {
  default = "westeurope"
}

variable "environment" {}

variable "address_space" {
  type    = "list"
  default = ["10.0.0.0/16"]
}
```

### terraform.tfvars

```hcl
environment = "devops"
resource_group_name = "dummy"
```

### Terraform pros

* Easy to learn syntax.
* Very little boilerplate.
* Very good documentation.
* Stateful.

### Terraform cons

* Some features missing. Can do most, but not all Azure provisioning.
* Needs a central backend for storing state (for instance Consul or Azure storage blob).

## Azure REST API

```yaml
---
- hosts: localhost
  vars:
    az_tenant_id: "{{ vault_az_tenant_id }}"
    az_client_id: "{{ vault_az_client_id }}"
    az_client_secret: "{{ vault_az_client_secret }}"
    az_subscription_id: "{{ vault_az_subscription_id }}"
    az_token_url: "https://login.microsoftonline.com/{{ az_tenant_id }}/oauth2/token"
    az_token_body: "resource=https://management.core.windows.net/&client_id={{ az_client_id }}&grant_type=client_credentials&client_secret={{ az_client_secret }}"
    az_mgnt_url: "https://management.azure.com/subscriptions/{{ az_subscription_id }}"
  tasks:
    - name: Login to Azure
      uri:
        url: "{{ az_token_url }}"
        method: POST
        body: "{{ az_token_body }}"
        headers:
          Content-Type: "application/x-www-form-urlencoded"
        status_code: 200
      register: login
      no_log: True

    - name: Setting Bearer token as fact
      set_fact:
        azure_bearer: "{{ login.json.access_token }}"
      no_log: True

    - name: "List all App Services in resource group"
      uri:
        url: "{{ az_mgnt_url }}/providers/Microsoft.Web/sites?api-version=2016-08-01"
        method: GET
        headers:
          Authorization: "Bearer {{ azure_bearer }}"
        status_code: 200
      register: app_services
      no_log: True

    - name: Print list of App Service names
      debug:
        msg: "{{ item.name }}"
      with_items: "{{ app_services.json.value }}"
```

### Azure REST API Pros

* Good for machine readable operations.
* Works with any programming and scripting language that can do HTTP requests, including Ansible.
* Documentation is extensive, and fairly good.

### Azure REST API Cons

* Difficult to find correct resource URI to use. Lifeline: <https://resources.azure.com/>
* Documentation is not always up to date. Sometimes the API have changed, and the documentation has not.
* Not all operations have meaningful names.
* Resource 'properties' object not fond of loops. Apply properties with caution.

## Ansible

Ansible 2.7 supports 71 modules for managing resources on Azure through Azure Resource Manager.

In order to use Azure Resource Manager modules for Ansible, you also need to install Azure SDK modules:

```bash
pip install 'ansible[azure]'
```

Refer to <https://docs.ansible.com/ansible/latest/scenario_guides/guide_azure.html> for how to deal with authentication.

Most of the Azure Resource Manager modules for Ansible comes in pairs of two:

* azure_rm_\[feature]
* azure_rm_\[feature]_facts

Where the former is used to create/update/delete Azure resources, and the latter is to gather facts from the resources.
The azure_rm_\[feature]_facts modules are useful for documenting resources that have been created manually through the Azure Portal, or through a different automation language.

### Ansible example

```yaml
---
- hosts: all
  vars:
    az_resource_group: dummy
    az_blob_storage:
      - container: "test-container1"
        storage_account: "teststorageacc"
      - container: "test-container2"
        storage_account: "teststorageacc"
  tasks:
    - name: Provision Azure Blob Storage
      azure_rm_storageblob:
        container: "{{ item.container }}"
        storage_account_name: "{{ item.storage_account }}"
        state: "{{ item.state | default('present') }}"
        subscription_id: "{{ az_subscription_id }}"
        client_id: "{{ az_client_id }}"
        secret: "{{ az_client_secret }}"
        tenant: "{{ az_tenant_id }}"
        resource_group: "{{ item.resource_group | default(az_resource_group) }}"
      with_items: "{{ az_blob_storage }}"
```

## Azure and ARM

In order to work with these modules, it is beneficial to know about a concept called Azure Resource Manager (ARM) templates.

By using the Azure Resource Manager modules for Ansible, you take a way a lot of the complexity of ARM, by simply using YAML syntax to describe your needs. However if you need to automate a resource for which there is no Ansible module, or if a module lack features you would like to use, you can use the generic **azure_rm_deployment** module.

### Ansible pros

* Using an Ansible module is simpler than learning Azure Resource Manager.
* You don't (necessarily) need to learn another tool.
* You can use the uri module with Azure REST API to add more complex workflows.
* Create complex and dynamic Azure Resource Manager templates using Jinja2.

### Ansible cons

* Azure Resource Manager templates have a steep learning curve when using azure_rm_deployment.
* Transforming JSON to YAML takes some getting used to.
* When using Jinja2 extensively, there is a high risk of creating a Terraform clone.

[Back](README.md) [Next](arm.md)
