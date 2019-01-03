# meetup-demo

Role to demo how to manage Azure PaaS services with Ansible.

In this role I will demonstrate how you can create resources on Microsoft Azure using different automation strategies with Ansible:

* Use the azure_rm_sqlserver and azure_rm_sqldatabase modules to create Azure SQL server and instances.
* Use the generic azure_rm_deployment module to create one Azure Cosmos DB from static ARM template and inline parameters.
* Use the generic azure_rm_deployment module to create two Azure Cosmos DB from dynamic ARM template using some Jinja.
* Use the generic azure_rm_deployment module to create several Azure Cosmos DBs from highly dynamic ARM template with Jinja2.

## Requirements

* Azure trial account.
* Create service principal in Azure Active Directory.
* Grant the service principal access to the Contributor role.

## Role Variables

TODO

## Dependencies

None

## Example Playbook

```yaml

```

## License

MIT

## Author Information

<https://github.com/avnes>
