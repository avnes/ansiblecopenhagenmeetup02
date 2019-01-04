# Using Azure Resource Manager modules for Ansible

It's demo time! Please note that the directory containing the Ansible Vault is not shared, but all the public variables are. They are also properly named, so you can easily create your own vault with your secret variables.

## Test Login to Azure

Show the demo/login.yml playbook, and explain the steps.

```bash
mkvirtualenv ansiblecopenhagenmeetup02
cp -r /usr/lib64/python2.7/site-packages/selinux $VIRTUAL_ENV/lib64/python2.7/site-packages
cp /usr/lib64/python2.7/site-packages/_selinux.so $VIRTUAL_ENV/lib64/python2.7/site-packages
cd demo
pip install -r requirements.txt
ansible-playbook --inventory ../../my_ansible/inventories/azure/hosts.yml --vault-password-file ~/.my_vault login.yml
```

## Create one Azure SQL server and two databases

```bash
workon ansiblecopenhagenmeetup02
cd demo
ansible-playbook --inventory ../../my_ansible/inventories/azure/hosts.yml --vault-password-file ~/.my_vault --extra-vars "ansible_python_interpreter=${VIRTUAL_ENV}/bin/python" sql.yml
```

While it is provisioning, explain the role tasks and default variables.

## Provision one Azure Cosmos DB using inline parameters

```bash
workon ansiblecopenhagenmeetup02
cd demo
ansible-playbook --inventory ../../my_ansible/inventories/azure/hosts.yml --vault-password-file ~/.my_vault --extra-vars "ansible_python_interpreter=${VIRTUAL_ENV}/bin/python" cosmosdb.yml
```

## Provision two Azure Cosmos DBs using parameters from a list variable

```bash
workon ansiblecopenhagenmeetup02
cd demo
ansible-playbook --inventory ../../my_ansible/inventories/azure/hosts.yml --vault-password-file ~/.my_vault --extra-vars "ansible_python_interpreter=${VIRTUAL_ENV}/bin/python" --extra-vars '{"az_number_of":2}' cosmosdb.yml
```

While it is provisioning, explain the role tasks and default variables.

## Provision three Azure Cosmos DBs using list variables and Jinja2

```bash
workon ansiblecopenhagenmeetup02
cd demo
ansible-playbook --inventory ../../my_ansible/inventories/azure/hosts.yml --vault-password-file ~/.my_vault --extra-vars "ansible_python_interpreter=${VIRTUAL_ENV}/bin/python" --extra-vars '{"az_number_of":3}' cosmosdb.yml
```

While it is provisioning, explain the role tasks and default variables.

[Back](README.md)
