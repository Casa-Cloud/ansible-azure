

# Run Create Resource Group Playbook for Dev Env
```
alokadhao@192 ansible-IAC % ansible-playbook -i \
inventories/dev playbooks/infra-complete.yml

ansible-playbook -i \
inventories/dev playbooks/00.destroy.yaml
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
az ad app create --display-name infra
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
az ad sp create --id f584d2bc-2f90-4347-9104-af952f2c59af
```

## Create a Certificate and attach it to Application created above
```
az ad app credential reset \
--id <AppId of Application> \
--cert "@./service-principal.pem"

az ad app credential reset \
--id f584d2bc-2f90-4347-9104-af952f2c59af \
--cert "@./service-principal.pem"
```

## List Certificate of application
```
az ad app credential list --id <AppId of Aplication> --cert
az ad app credential list --id f584d2bc-2f90-4347-9104-af952f2c59af --cert
Certificate KeyId is unique ID of certificate
```
## Get SP ID
```
az ad sp list -o table

Get AppID of the SP which is same as Appicaiton CliendID 

Output:- f584d2bc-2f90-4347-9104-af952f2c59af

```

## Attach RBAC role to SP to associate it to subscription
```
az role assignment create \
--assignee <AppId of Application OR AppId of SP its the same >
--role "<role>" \
--subscription <subsciptionID>
--scope /subscriptions/<subscriptionID>
```
### RBAC Contributer to Grants full access to manage all resources, but does not allow you to assign roles in Azure RBAC

##### User reqire Azure Role 'USER ACCESS ADMINISTRATOR' to run this query.

```
az role assignment create \
--assignee f584d2bc-2f90-4347-9104-af952f2c59af \
--role "Contributor" \
--scope /subscriptions/3f9865ad-7775-47bb-ba40-18f30d2bb648 \
--subscription 3f9865ad-7775-47bb-ba40-18f30d2bb648
```

### As Contributor role is unable to create other users hence 
1. User Administrator role is Azure Entra Roles and not IAM roles 
2. Azure Entra roles cannot be assigned via az command as it is limitation.
3. You need to go to Entra, Roles and Administrators -> Search for "User Administrator"
-> Add Assignment, and if App01 not found then search and select Applcation

### As Contributor role do not have RBAC to other users during playbook.. hence need to add User Access Administrator. This Role is part of Azure roles so we can use az role command
```
az role assignment create \
--assignee f584d2bc-2f90-4347-9104-af952f2c59af \
--role "User Access Administrator" \
--scope /subscriptions/3f9865ad-7775-47bb-ba40-18f30d2bb648 \
--subscription 3f9865ad-7775-47bb-ba40-18f30d2bb648
```

## Delete Role assignment
```
az role assignment delete \
--assignee e6503702-ec3a-4266-90a5-d742273f5797 \
--role "Contributor"
```

OR 

```
az role assignment delete \
--ids="/subscriptions/b63a40da-5629-452a-9350-e48460eae13f/providers/Microsoft.Authorization/roleAssignments/910f27fb-73df-4cc4-a17d-18dfef10732c"
```

## List role assigned to SP
```
az role assignment list \
--assignee f584d2bc-2f90-4347-9104-af952f2c59af

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
