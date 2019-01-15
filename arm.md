# Using Azure Resource Manager templates

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

## Ansible module azure_rm_deployment

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

This Azure Cosmos DB example will be covered in full later in this presentation.

### Tips & Tricks when working with azure_rm_deployment

The official module documentation at <https://docs.ansible.com/ansible/latest/modules/azure_rm_deployment_module.html> contains a link to <https://github.com/azure/azure-quickstart-templates> where you will find JSON templates for a large number of resources or combination of resources. Some of these templates are a bit outdated though, and you often need to tweak them a bit to get them working. It is still a really good source to get started with ARM templates for use with Ansible.

Another way to get a template is to create the resource manually through the Azure Portal at <https://portal.azure.com/>. Just before submitting the resource, for creating there will most of the time be a link to download the template in Azure Portal, because Azure Portal itself is using ARM template to create the resources. Also if you forget to do this, you can shortly afterwards navigate to the resource group where you created the new resource, and look at the *Deployment* page. Here you will also be able to view the latest templates that have been applied, including which parameters that were used during deployment.

When applying properties to a resource, you will not be able to do as above. I guess Azure Portal is relying on the Azure REST API when updating a resource, and hence you will not see a deployment template for changes.

Your final lifeline is to navigate to a resource group and look at the *Automation script*, which will let you download a template containing all resources in that resource group (except some Azure Insights resources). Please note that this template will be extremely large; often spanning 10 000+ lines, and instead of parameters, you will have hard coded values. This makes that template less than ideal for reusable automation, but it is still great blueprint when creating your own parameterized templates.

[Back](README.md) [Next](ansible.md)
