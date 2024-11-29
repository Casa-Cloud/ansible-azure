Setup Roles Path.

```
source setup.sh
```
By default, running a shell script as ./setup.sh runs it in a subshell, 
which means that the environment variables defined in the script will 
not be reflected in your current shell session after the script ends.
To make the exported variables available in your current shell, 
you need to source the script instead of executing it in a subshell.

# Get default role path
```
 ansible-config dump | grep ROLES_PATH
DEFAULT_ROLES_PATH(default) = ['/Users/alokadhao/.ansible/roles', '/usr/share/ansible/roles', '/etc/ansible/roles']
```

```
# To check all ansible configuration includeing the roles_path use command ansible-config dump | grep DEFAULT_ROLES_PATH
DEFAULT_ROLES_PATH(/Users/alokadhao/Documents/github/imagincloud/ansible-azure/01-aks_create/ansible.cfg) = ['/Users/alokadhao/Documents/github/imagincloud/ansible-azure/01-aks_create/roles', '/Users/alokadhao/Documents/github/imagincloud/ansible-azure/00-GlobalRoles/00-Core', '/Users/alokadhao/Documents/github/imagincloud/ansible-azure/00-GlobalRoles/02-Containers', '/Users/alokadhao/Documents/github/imagincloud/ansible-azure/00-GlobalRoles/03-Networking', '/Users/alokadhao/Documents/github/imagincloud/ansible-azure/00-GlobalRoles/06-Management']
```

# Run Create Resource Group Playbook for Dev Env
```
alokadhao@192 ansible-IAC % ansible-playbook -i \
inventories/dev playbooks/infra-complete.yml
```

# Create Certificate for Service principal
## Create Certificate Signing Request.

```
openssl req \
-newkey rsa:4096 -nodes -keyout "service-principal.key" \
-out "service-principal.csr"

Output:
service-princilap.csr
service-principal.key
``` 

## Generate Certificate using CSR and Public Key
```
openssl x509 \
-signkey "service-principal.key" \
-in "service-principal.csr" \
-req -days 365 -out "service-principal.crt"
```

## Create a PEM file 
```
cat service-principal.crt service-principal.key > service-principal.pem
output:- 
service-principal.pem
```

# Create Application in Microsoft EntraID

## Login to Azure CLI
```
az login
```

## Create Application
```
az ad app create --display-name app02
```

## List Applcation
```
az ad app list -o table
```

## Check Service principal list
```
az ad sp list -o table
```

## Create Service Prinicipal and attach it to Application created above
```
az ad sp create --id <AppId of Aplication>
az ad sp create --id 80b022e1-16bf-4924-94f8-b2a8b2824eaa
```

## Create a Certificate and attach it to Application created above
```
az ad app credential reset \
--id <AppId of Application> \
--cert "@./service-principal.pem"

az ad app credential reset \
--id 80b022e1-16bf-4924-94f8-b2a8b2824eaa \
--cert "@./service-principal.pem"
```

## List Certificate of application
```
az ad app credential list --id <AppId of Aplication> --cert
az ad app credential list --id 80b022e1-16bf-4924-94f8-b2a8b2824eaa --cert
Certificate KeyId is unique ID of certificate
```

## Get SP ID
```
az ad sp list 

Get AppID of the SP which is same as Appicaiton CliendID 

Output:- e6503702-ec3a-4266-90a5-d742273f5797

```

## Attach RBAC role to SP to associate it to subscription
```
az role assignment create \
--assignee <AppId of Application OR AppId of SP its the same >
--role "Contributor" \
--subscription <subsciptionID>
--scope /subscriptions/<subscriptionID>


az role assignment create \
--assignee 80b022e1-16bf-4924-94f8-b2a8b2824eaa \
--role "Contributor" \
--scope /subscriptions/b63a40da-5629-452a-9350-e48460eae13f \
--subscription b63a40da-5629-452a-9350-e48460eae13f
```

## Delete Role assignment
```
az role assignment delete \
--assignee e6503702-ec3a-4266-90a5-d742273f5797 \
--role "Contributor"
```

## List role assigned to SP
```
az role assignment list \
--assignee 80b022e1-16bf-4924-94f8-b2a8b2824eaa

Output:-
[
  {
    "condition": null,
    "conditionVersion": null,
    "createdBy": "5b6a6b5b-2c8d-47f1-8327-8f6e1a7d3dd1",
    "createdOn": "2024-10-06T06:11:07.278565+00:00",
    "delegatedManagedIdentityResourceId": null,
    "description": null,
    "id": "/subscriptions/b63a40da-5629-452a-9350-e48460eae13f/providers/Microsoft.Authorization/roleAssignments/3513b1fa-67f0-4160-a344-b6bb7af7e630",
    "name": "3513b1fa-67f0-4160-a344-b6bb7af7e630",
    "principalId": "4c0cfa47-8f38-4f21-bee7-e58e6e47cd9f",
    "principalName": "e6503702-ec3a-4266-90a5-d742273f5797",
    "principalType": "ServicePrincipal",
    "roleDefinitionId": "/subscriptions/b63a40da-5629-452a-9350-e48460eae13f/providers/Microsoft.Authorization/roleDefinitions/b24988ac-6180-42a0-ab88-20f7382dd24c",
    "roleDefinitionName": "Contributor",
    "scope": "/subscriptions/b63a40da-5629-452a-9350-e48460eae13f",
    "type": "Microsoft.Authorization/roleAssignments",
    "updatedBy": "5b6a6b5b-2c8d-47f1-8327-8f6e1a7d3dd1",
    "updatedOn": "2024-10-06T06:11:07.278565+00:00"
  }
]
```

# Login using Service Principal with Certificate

```
az login \
--service-principal \
-u <AppId of Aplication> \
-p <path to PEM file> \
--tenant <tenant ID>

az login \
--service-principal \
-u 80b022e1-16bf-4924-94f8-b2a8b2824eaa \
--certificate ./service-principal.pem \
--tenant 4abb5e5e-af15-4a34-b3c2-18f4a9303ee4

```

# Create Deployment
kubectl create deployment helloworld --image=aloka/testci:8 

# Port Forwarding
kubectl port-forward helloworld-cff5bddc4-57ldg 9085:9085


# Portforward a service 
kubectl port-forward service/<service-name> <local-port>:<service-port> [flags]

## Port forward service app1
kubectl port-forward service/aks-helloworld-one 8056:80 -n ingress-basic

## Port forward service app2
kubectl port-forward service/aks-helloworld-two 8057:80 -n ingress-basic