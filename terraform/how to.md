# The Terraform code deploys following resources:

1 ClickHouse service on AWS
The ClickHouse service is available from anywhere.


## init and plan and apply

```
> terraform init
> terraform <plan|apply> -var-file=variables.tfvars
  > terraform plan -var-file=variables.tfvars -out=ted_out.tfplan
  > terraform apply "ted_out.tfplan"

```

## List services 

```
  > curl -u "---:---" \
    https://api.clickhouse.cloud/v1/organizations/-----/services | \
    jq '.result[] | select(.region == "ap-northeast-2" and .tags[].value=="environment")'


  > curl -u "---:---" \
  https://api.clickhouse.cloud/v1/organizations/------/services | jq '.result[] | select(.region == "ap-northeast-2" and (.tags[]? | select(.key =="environment" and .value=="production")))'
```


