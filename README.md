Infra
=======


DevOps course, practices with [Google Cloud Platform](https://cloud.google.com/).

1.1) Project has scripts for manual setup:
 - install Ruby `install_ruby.sh`
 - install MongoDB `install_mongodb.sh`
 - deploy application 'reddit' from Artemmkin `scripts/deploy.sh`

1.2) Project has scripts to make automatic setup at the time startup new instance:
 - startup script `scripts/startup_script1_1.sh`
 - inner script `scripts/startup_script1_2.sh` with main tasks

--- 

2.1) Use `gcloud` command to build `reddit-app` instance in GCE:
 - use default image from GCE
 - use startup script to make prepare installations

```bash 
$ gcloud compute instances create 
 --boot-disk-size=10GB \
 --image=ubuntu-1604-xenial-v20170815a \
 --image-project=ubuntu-os-cloud \
 --machine-type=g1-small \
 --tags puma-server \
 --restart-on-failure 
 --zone=europe-west1-b \
 --metadata-from-file startup-script=./startup_script1_1.sh 
reddit-app
```

2.2) Use command `gcloud` to build `reddit-app` instance in GCE:
- use [HashiCorp Packer](https://www.packer.io/intro/index.html) to build image with prepared installations
- use a custom bake-image

```bash 
# Create new bake-image
$ packer build \
 -var 'machine_type=f1-micro' \
 -var 'project_id=practice-devops-gcp-1' \
 -var 'source_image=ubuntu-1604-xenial-v20170815a' \
./packer/immutable.json

# Use bake-image and create new instance
$ gcloud compute instances create \
 --image=reddit-base-2-1505095462 \
 --image-project=practice-devops-gcp-1 \
 --machine-type=g1-small \
 --tags puma-server \
 --restart-on-failure \
 --zone=europe-west1-b \
reddit-app
```

--- 

3.1) Use [HashiCorp Terraform](https://www.terraform.io/intro/index.html) to build `reddit-app` instance in GCE:
 - use default image from GCE or after [HashiCorp Packer](https://www.packer.io/intro/index.html)
 - use required file `variables.tf` with definition of all variables
 - internal file `terraform.tfvars` with custom values for all variables without default
```text  
project = "infra-179717"
public_key_path = "~/.ssh/otus_devops_appuser.pub"
private_key_path = "~/.ssh/otus_devops_appuser"
disk_image = "reddit-base-2-1505095462"
``` 
```text  
terraform$ terraform init
terraform$ terraform plan
terraform$ terraform apply
```
 


3.2) Use [HashiCorp Terraform](https://www.terraform.io/intro/index.html) to build `reddit-app` and `reddit-db` instances in GCE:
 - use [HashiCorp Packer](https://www.packer.io/intro/index.html) to build images with prepared installation

```text  
packer build \
 -var 'machine_type=f1-micro' \
 -var 'project_id=infra-179717' \
 -var 'source_image=ubuntu-1604-xenial-v20170815a' \
./packer/db.json

packer build \
 -var 'machine_type=f1-micro' \
 -var 'project_id=infra-179717' \
 -var 'source_image=ubuntu-1604-xenial-v20170815a' \ 
./packer/app.json
``` 
 - use required file `variables.tf` for _each module_ and _each environment_ with definition of needed variables
 - do not forget to create an internal file `terraform.tfvars` with values in runtime
```text
project = "infra-179717"
public_key_path = "~/.ssh/otus_devops_appuser.pub"
private_key_path = "~/.ssh/otus_devops_appuser"
db_disk_image = "reddit-db-1505646807"
app_disk_image = "reddit-app-1505646464"
disk_image = "reddit-base-3-1505269146"
``` 
```text  
terraform/prod$ terraform init
terraform/prod$ terraform plan
terraform/prod$ terraform apply
terraform/prod$ terraform destroy

terraform/stage$ terraform init
terraform/stage$ terraform plan
terraform/stage$ terraform apply
terraform/stage$ terraform destroy
```
 
