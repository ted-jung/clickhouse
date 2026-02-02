


# init and plan and apply

```
> terraform init
> terraform <plan|apply> -var-file=variables.tfvars
  > terraform plan -var-file=variables.tfvars -out=ted_out.tfplan
  > terraform apply "ted_out.tfplan"

```
