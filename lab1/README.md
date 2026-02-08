# S3 Bucket CloudFormation Template 

This folder contains a CloudFormation template that creates an S3 bucket.

Files:

 - [s3-create-bucket.yml](s3-create-bucket.yml): CloudFormation template to create an S3 bucket. The bucket name is formed as `app-<UAI>-<BucketName>`, where `UAI` is a required 10-digit number and `BucketName` is a required logical name.

Deploy with AWS CLI (example):

```bash
# Deploy (UAI is required and must start with 'uai' followed by 7 digits; total length 10). Created bucket will be app-<UAI>-<BucketName>
aws cloudformation deploy \
  --template-file lab1/s3-create-bucket.yml \
  --stack-name my-s3-stack \
  --parameter-overrides UAI=uai0123456 BucketName=my-biz-bucket
```

Notes:
- `UAI` is required and must start with 'uai' followed by exactly 7 digits (for a total length of 10 characters), for example `uai0123456`.
- `BucketName` is required and must be 3-48 characters using lowercase letters, numbers and hyphens. The final bucket name `app-<UAI>-<BucketName>` must be unique across AWS S3 (global) and cannot exceed 63 characters.
- The template blocks public access by default via PublicAccessBlockConfiguration.
 - The created S3 bucket is tagged with the following keys:
   - `CreatedByStack`: the CloudFormation stack name that created the bucket (`AWS::StackName`).
   - `UAI`: the provided UAI value.
   - `BucketName`: the final bucket name (`app-<UAI>-<BucketName>`).
