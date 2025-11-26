## how to install(mac)
> brew install minio/stable/minio
> minio server /Users/tedj/Ted-lab/minio/data --address ":9100" --console-address ":9101"


Object API (Amazon S3 compatible):

```
   Go:         https://docs.min.io/docs/golang-client-quickstart-guide
   Java:       https://docs.min.io/docs/java-client-quickstart-guide
   Python:     https://docs.min.io/docs/python-client-quickstart-guide
   JavaScript: https://docs.min.io/docs/javascript-client-quickstart-guide
   .NET:       https://docs.min.io/docs/dotnet-client-quickstart-guide
```


FLAGS:

```
FLAGS:
  --config value               specify server configuration via YAML configuration [$MINIO_CONFIG]
  --address value              bind to a specific ADDRESS:PORT, ADDRESS can be an IP or hostname (default: ":9000") [$MINIO_ADDRESS]
  --console-address value      bind to a specific ADDRESS:PORT for embedded Console UI, ADDRESS can be an IP or hostname [$MINIO_CONSOLE_ADDRESS]
  --ftp value                  enable and configure an FTP(Secure) server
  --sftp value                 enable and configure an SFTP server
  --certs-dir value, -S value  path to certs directory (default: "/Users/tedj/.minio/certs")
  --quiet                      disable startup and info messages
  --anonymous                  hide sensitive information from logging
  --json                       output logs in JSON format
  --help, -h                   show help
```

EXAMPLES:

```
  1. Start MinIO server on "/home/shared" directory.
     $ minio server /home/shared

  2. Start single node server with 64 local drives "/mnt/data1" to "/mnt/data64".
     $ minio server /mnt/data{1...64}

  3. Start distributed MinIO server on an 32 node setup with 32 drives each, run following command on all the nodes
     $ minio server http://node{1...32}.example.com/mnt/export{1...32}

  4. Start distributed MinIO server in an expanded setup, run the following command on all the nodes
     $ minio server http://node{1...16}.example.com/mnt/export{1...32} \
            http://node{17...64}.example.com/mnt/export{1...64}

  5. Start distributed MinIO server, with FTP and SFTP servers on all interfaces via port 8021, 8022 respectively
     $ minio server http://node{1...4}.example.com/mnt/export{1...4} \
           --ftp="address=:8021" --ftp="passive-port-range=30000-40000" \
           --sftp="address=:8022" --sftp="ssh-private-key=${HOME}/.ssh/id_rsa"
```

## create a policy(json)

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:ListAllMyBuckets",
                "s3:GetBucketLocation" 
            ],
            "Resource": [
                "arn:aws:s3:::*"
            ],
            "Sid": "AllowListingAllBuckets"
        }
    ]
}
```

## add a user

```
  > mc admin user add tedminio/ ted password
  > mc admin user ls tedminio
  > mc admin user info tedminio ted
  > mc admin policy set tedminio list-buckets-only user=ted
  > mc admin policy attach tedminio readwrite --user ted
  > mc admin accesskey ls tedminio ted
  > cat ted_list_buckets_policy.json
  > mc admin accesskey create tedminio/ ted --access-key tedaccesskey --secret-key tedsecretkey --policy ./ted_list_buckets_policy.json
```

## How to add a new user

```
    > mc admin user add tedminio/ ted magicword
    > mc admin user ls tedminio
    > mc admin user info tedminio ted
```

## How to handle policy (Minio Policy-Based Access Control)
## Attach policies to either user or group

```
    > mc admin policy create tedminio list-buckets-only ~/Ted-lab/minio/ted_list_buckets_policy.json
    > mc admin policy attach set tedminio list-buckets-only user=ted
    > mc admin policy attach tedminio list-buckets-only user=ted
    > mc admin policy attach tedminio ted_list-buckets-only user=ted
    > mc admin policy attach tedminio readwrite --user ted
    > mc admin policy ls tedminio
    > mc admin policy info tedminio
```

## How to create keys for user

```
    > mc admin accesskey ls tedminio ted
    > mc admin accesskey create tedminio/ ted --access-key tedaccesskey --secret-key tedsecretkey --policy ./ted_list_buckets_policy.json
```


# Step3: How to select/insert

```
> SELECT *
    FROM s3('http://localhost:9100/ted-bucket/aapl_stock.csv','minioadmin','minioadmin','CSVWithNames')
   LIMIT 3;


> INSERT INTO FUNCTION s3('http://localhost:9100/ted-bucket/test-data.csv.gz', 'minioadmin','minioadmin','CSV', 'name String, value UInt32', 'gzip')
  VALUES ('test-data', 1), ('test-data-2', 2);
```
