# Azure Resource Manager templates

An Azure Resource Manager template, or ARM for short, is a JSON file that describes zero to many Azure resources. To quote the official documentation at <https://azure.microsoft.com/en-us/resources/templates/>:

> *Azure Resource Manager allows you to provision your applications using a declarative template.*
> *In a single template, you can deploy multiple services along with their dependencies.*

When Azure evaluates an ARM template, it will look at the delta between the current state, and the new state, and hence changes will be incremental. This evaluation happens on the Azure side; in contrast to Terraform, where the evaluation happens locally.

If you apply an empty ARM template to an existing resource group, and set deployment mode to complete, all resources will be wiped, like in the example below:

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {},
    "variables": {},
    "resources": [],
    "outputs": {}
}
```

When using ARM you will always supply a template, and most of the time you will also supply a parameter file.

## azure_rm_deployment

The **azure_rm_deployment** module lets you supply an ARM template either as YAML or JSON file. Optionally you can also supply a parameter file in either YAML, JSON or as inline parameters (YAML). You can even "template the template" with Jinja2.

Note: When using JSON files, you need to serve them over HTTP. That is why I personally prefer local YAML files. Lifeline: <https://www.json2yaml.com/>

```yaml
- name: Provision Azure Cosmos DB from template
  azure_rm_deployment:
    subscription_id: "{{ az_subscription_id }}"
    client_id: "{{ az_client_id }}"
    secret: "{{ az_client_secret }}"
    tenant: "{{ az_tenant_id }}"
    location: "{{ az_region }}"
    resource_group_name: "{{ az_resource_group }}"
    template: "{{ template_cosmosdb }}"
    parameters: "{{ item }}"
  loop: "{{ COSMOSDB }}"
  when: COSMOSDB is defined
```

The Azure CosmosDB example will be covered in full later in this presentation.
