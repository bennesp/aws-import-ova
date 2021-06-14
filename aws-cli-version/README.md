# How to import an OVA into AWS as an AMI using the aws-cli

## 0. (Required) Create an S3 bucket and upload the ova

```
aws s3 mb name-of-the-bucket --region eu-west-1

aws s3 cp path-of-the-file.ova s3://name-of-the-bucket/
```

## 1. (Required) Create role named `vmimport`

NOTE: The name **must** be vmimport, you cannot choose it.

```
aws iam create-role --role-name vmimport --assume-role-policy-document "file://trust-policy.json"
```

## 2. (Required) Create a policy for the created role

If you want you can restrict the policy to the right bucket. Leave "*" otherwise

```
aws iam put-role-policy --role-name vmimport --policy-name vmimport --policy-document file://role-policy.json
```

## 3. (Required) Import the ova

```
aws ec2 import-image --disk-containers "file://importvm.json" --region eu-west-1
```

## 4. (Optional) Monitor the import process

```
aws ec2 describe-import-image-tasks --import-task-ids import-ami-0f26976011b39b197 --region eu-west-1
```

or with `watch`:

```
watch -n 10 "aws ec2 describe-import-image-tasks --import-task-ids import-ami-0f26976011b39b197 --region eu-west-1"
```

## 5. (Optional) Give the AMI a name

The only way is to copy the image into a new one and delete the old one.

Here `ami-123de3` is the ID of the imported AMI

```
aws ec2 copy-image --source-image-id ami-123de3 --source-region eu-west-1 --region eu-west-1 --name <new-name>
```

WAIT for the completion of the copy with:
```
aws ec2 describe-images --region eu-west-1 --filters=Name=name,Values=<new-name>
```

and then:
```
aws ec2 deregister-image --image-id ami-123de3 --region us-east-1
```

## 6. (Optional) Clean resources

Delete S3 bucket and ALL its content
```
aws s3 rb --force s3://name-of-the-bucket
```

Delete role and policy:
```
aws iam delete-role-policy --role-name vmimport --policy-name vmimport
aws iam delete-role --role-name vmimport
```
