# DeepaLab — AWS Lab

This repository contains CloudFormation templates and a GitHub Actions workflow to deploy them.

**Purpose**: practice packaging and deploying nested CloudFormation stacks (S3, EC2, etc.) via CI.

**Files to know**
- **`lab1/stack_input_dev.yml`**: wrapper stack that calls nested templates. See [lab1/stack_input_dev.yml](lab1/stack_input_dev.yml).
- **`lab1/s3-create-bucket.yml`**: S3 bucket template referenced by the wrapper. See [lab1/s3-create-bucket.yml](lab1/s3-create-bucket.yml).
- **`lab1/cfn-params.env.example`**: example parameters file you can copy to `lab1/cfn-params.env`. See [lab1/cfn-params.env.example](lab1/cfn-params.env.example).
- **Workflow**: [`.github/workflows/deploy-s3-cfn.yml`](.github/workflows/deploy-s3-cfn.yml) — packages nested templates and deploys on changes.

**Required repository secrets** (Settings → Secrets & variables → Actions)
- **`AWS_ACCESS_KEY_ID`**: access key of an IAM user with CloudFormation/S3 permissions.
- **`AWS_SECRET_ACCESS_KEY`**: secret key for the IAM user.
- **`AWS_REGION`**: AWS region to deploy to (e.g., `us-east-1`).
- **`CFN_ARTIFACTS_BUCKET`**: S3 bucket where `aws cloudformation package` will upload nested template artifacts.
- Optional: **`CFN_UAI`** and **`CFN_BUCKETNAME`** — used by pushes (alternatively provide via workflow_dispatch or `lab1/cfn-params.env`).

Security note: never commit real secrets. Use the repository Secrets UI or `lab1/cfn-params.env` only for non-sensitive example values.

How it works
- The workflow triggers on pushes to `lab1/stack_input_dev.yml`, `lab1/s3-create-bucket.yml`, or other `lab1/*.yml` files.
- It collects `UAI` and `BucketName` (priority order: workflow inputs → `lab1/cfn-params.env` → repo secrets `CFN_UAI`/`CFN_BUCKETNAME`).
- It validates inputs (UAI pattern, bucket name chars and final length), runs `aws cloudformation package` to upload nested templates to `CFN_ARTIFACTS_BUCKET`, then deploys the packaged wrapper.

Quick start
1. Add the required secrets listed above.
2. (Optional) Create a params file from the example:

```bash
cp lab1/cfn-params.env.example lab1/cfn-params.env
# Edit lab1/cfn-params.env and set UAI and BucketName
```

3. Push a change to one of the templates, e.g. edit and commit `lab1/stack_input_dev.yml`:

```bash
git add lab1/stack_input_dev.yml
git commit -m "update wrapper" 
git push
```

4. Or run the workflow manually and provide parameters:

```bash
gh workflow run deploy-s3-cfn.yml --field UAI=uai0123456 --field BucketName=my-bucket-name
```

Troubleshooting
- If you see "Missing UAI or BucketName", supply values via `workflow_dispatch` inputs, `lab1/cfn-params.env`, or the secrets `CFN_UAI`/`CFN_BUCKETNAME`.
- If packaging fails, ensure `CFN_ARTIFACTS_BUCKET` exists and the IAM user has `s3:PutObject` permission for it.

IAM permissions (recommended minimum)
- Allow CloudFormation stack actions and S3 actions for the target bucket and `CFN_ARTIFACTS_BUCKET`. If your templates create roles, include `iam:PassRole` for the roles used by the stacks.

Want me to add a README section with exact IAM policy JSON and a CI test example? Reply and I will add it.

**IAM policy example**

Below is a recommended minimal IAM policy you can attach to the IAM user used by GitHub Actions. Replace the placeholders (`ACCOUNT_ID`, `CFN_ARTIFACTS_BUCKET`, `deepalab-s3-bucket`) with your account/bucket names and adjust resources as needed for least-privilege.

```json
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Effect": "Allow",
			"Action": [
				"cloudformation:CreateStack",
				"cloudformation:UpdateStack",
				"cloudformation:DeleteStack",
				"cloudformation:DescribeStacks",
				"cloudformation:ValidateTemplate",
				"cloudformation:GetTemplate",
				"cloudformation:CreateChangeSet",
				"cloudformation:ExecuteChangeSet",
				"cloudformation:DeleteChangeSet",
				"cloudformation:ListStackResources",
				"cloudformation:DescribeStackEvents",
				"cloudformation:ListStacks"
			],
			"Resource": "*"
		},
		{
			"Effect": "Allow",
			"Action": [
				"s3:PutObject",
				"s3:GetObject",
				"s3:ListBucket",
				"s3:CreateBucket",
				"s3:PutBucketPolicy",
				"s3:PutBucketAcl",
				"s3:PutEncryptionConfiguration",
				"s3:PutBucketVersioning",
				"s3:PutBucketPublicAccessBlock",
				"s3:PutBucketLogging",
				"s3:PutBucketTagging",
				"s3:DeleteObject"
			],
			"Resource": [
				"arn:aws:s3:::CFN_ARTIFACTS_BUCKET",
				"arn:aws:s3:::CFN_ARTIFACTS_BUCKET/*",
				"arn:aws:s3:::deepalab-s3-bucket",
				"arn:aws:s3:::deepalab-s3-bucket/*"
			]
		},
		{
			"Effect": "Allow",
			"Action": [
				"iam:PassRole"
			],
			"Resource": "arn:aws:iam::ACCOUNT_ID:role/*"
		}
	]
}
```

Notes:
- `cloudformation` actions are scoped to `*` because CloudFormation controls many resources; for stronger restrictions you can scope to specific stack ARNs after creating them.
- `iam:PassRole` is required only if your templates create or use IAM roles (the workflow uses `CAPABILITY_NAMED_IAM`). Replace `ACCOUNT_ID` with your AWS account ID and restrict the role ARN pattern if possible.
- Ensure the user also has permission to write objects to `CFN_ARTIFACTS_BUCKET` since `aws cloudformation package` uploads nested templates there.

If you want, I can generate a least-privilege policy tailored to the exact resources your templates create — say `deepalab-stack-dev` and the specific role ARNs. Reply if you'd like that.
