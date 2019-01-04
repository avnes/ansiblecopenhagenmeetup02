# meetup-demo

Role to demo how to manage Azure PaaS services with Ansible.

In this role I will demonstrate how you can create resources on Microsoft Azure using different automation strategies with Ansible:

* Use the azure_rm_sqlserver and azure_rm_sqldatabase modules to create Azure SQL server and instances.
* Provision one Azure Cosmos DB using inline parameters with the generic azure_rm_deployment module.
* Provision two Azure Cosmos DBs using parameters from a list variable with the generic azure_rm_deployment module.
* Provision three Azure Cosmos DBs using a list variable and Jinja2

## Requirements

* Azure trial account.
* Create service principal in Azure Active Directory.
* Grant the service principal access to the Contributor role.
* Azure SDK modules installed on the host running Ansible. See <https://docs.ansible.com/ansible/latest/scenario_guides/guide_azure.html> for instructions.

## Role Variables

```yaml
az_tenant_id: "{{ vault_az_tenant_id }}"
az_client_id: "{{ vault_az_client_id }}"
az_client_secret: "{{ vault_az_client_secret }}"
az_subscription_id: "{{ vault_az_subscription_id }}"
az_token_url: "https://login.microsoftonline.com/{{ az_tenant_id }}/oauth2/token"
az_token_body: "resource=https://management.core.windows.net/&client_id={{ az_client_id }}&grant_type=client_credentials&client_secret={{ az_client_secret }}"
az_mgnt_url: "https://management.azure.com/subscriptions/{{ azure_subscription_id }}"

az_resource_group: "{{ vault_az_resource_group }}"
az_location: northeurope
```

As shown above, login details and the name of the resource group is fetched from an Ansible inventory, which is not shared here.

## Dependencies

None

## Example Playbook

```yaml
---
- hosts: localhost
  vars:
    az_sql_server_name: ansiblecphsrv
    az_sql_username: "{{ vault_az_sql_username }}"
    az_sql_password: "{{ vault_az_sql_password }}"
    az_sql_database_name_prefix: ansiblecphdb
    az_sql_databases:
      - create_mode: default
        edition: free
      - create_mode: default
        edition: basic

    az_number_of: 1
    az_location_name: "North Europe"

    az_cosmos_db:
      -
        name:
          value: ansiblecphcdb11
        location:
          value: "{{ az_location }}"
        locationName:
          value: "{{ az_location_name }}"
        defaultExperience:
          value: Cassandra
      -
        name:
          value: ansiblecphcdb12
        location:
          value: "{{ az_location }}"
        locationName:
          value: "{{ az_location_name }}"
        defaultExperience:
          value: Cassandra

  roles:
    - meetup-demo
```

Pass in the variable *az_resource* in through --extra-vars from the command line. This variable needs to either have the value *sql* or *cosmosdb*.

## License

MIT

## Author Information

<https://github.com/avnes>
