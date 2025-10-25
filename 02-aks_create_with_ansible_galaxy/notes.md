## Get Ansible Collections to project 

```
mkdir -p collections
cat > collections/requirements.yml <<'YAML'
collections:
  - name: casa_cloud.core
    version: "==1.0.0"
  - name: casa_cloud.containers
    version: "==1.0.0"
  - name: casa_cloud.networking
    version: "==1.0.0"
  - name: casa_cloud.management
    version: "==1.0.0"
  - name: casa_cloud.security
    version: "==1.0.0"
  - name: casa_cloud.analytics
    version: "==1.0.0"
  - name: casa_cloud.database
    version: "==1.0.0"
  - name: casa_cloud.compute
    version: "==1.0.0"
  - name: casa_cloud.storage
    version: "==1.0.0"
  - name: casa_cloud.web
    version: "==1.0.0"
  - name: casa_cloud.github
    version: "==1.0.0"
  - name: casa_cloud.docker
    version: "==1.0.0"
  - name: casa_cloud.localhost
    version: "==1.0.0"
  - name: casa_cloud.helm
    version: "==1.0.0"
YAML
```

# Install the collections 
```
ansible-galaxy collection install -r collections/requirements.yml
```

## Validate the playbook if it can read the colleciton and roles 

```

alokadhao@192 02-aks_create_with_ansible_galaxy % ansible-playbook -i inventory/hosts -c local playbooks/00.destroy.yaml --syntax-check

[WARNING]: Unable to parse /Users/alokadhao/Documents/github/imagincloud/ansible-azure/02-aks_create_with_ansible_galaxy/inventory/hosts as an inventory source
[WARNING]: No inventory was parsed, only implicit localhost is available
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'
ERROR! the role 'login_azure_sp_cert_01' was not found in /Users/alokadhao/Documents/github/imagincloud/ansible-azure/02-aks_create_with_ansible_galaxy/playbooks/roles:/Users/alokadhao/.ansible/roles:/usr/share/ansible/roles:/etc/ansible/roles:/Users/alokadhao/Documents/github/imagincloud/ansible-azure/02-aks_create_with_ansible_galaxy/playbooks

The error appears to be in '/Users/alokadhao/Documents/github/imagincloud/ansible-azure/02-aks_create_with_ansible_galaxy/playbooks/00.destroy.yaml': line 28, column 15, but may
be elsewhere in the file depending on the exact syntax problem.

The offending line appears to be:

      ansible.builtin.import_role:
        name: login_azure_sp_cert_01
              ^ here
```