# Using Azure Resource Manager modules for Ansible

It's demo time! Please note that the directory containing the Ansible Vault is not shared, but all the public variables are. They are also properly named, so you can easily create your own vault with your secret variables.

```bash
cd demo
mkvirtualenv ansiblecopenhagenmeetup02
pip install -r requirements.txt
ANSIBLE_CONFIG=./role.cfg; export ANSIBLE_CONFIG
ansible-playbook --inventory ../../my_ansible/inventories/azure/hosts.yml --vault-password-file ~/.my_vault login.yml
```

[Back](README.md)
