# DeepaLab — AWS Lab

Quick-start (short)

- Purpose: package and deploy nested CloudFormation templates from `lab1/` via GitHub Actions.
- Key files: [lab1/stack_input_dev.yml](lab1/stack_input_dev.yml), [lab1/s3-create-bucket.yml](lab1/s3-create-bucket.yml), [.github/workflows/deploy-s3-cfn.yml](.github/workflows/deploy-s3-cfn.yml).

Required repository secrets:
- `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_REGION`.

Parameters source (priority): workflow inputs → `lab1/cfn-params.env` (recommended) → repo secret `CFN_ARTIFACTS_BUCKET` (only if you prefer a secret).

Steps
1. Copy params example and edit:

```bash
cp lab1/cfn-params.env.example lab1/cfn-params.env
# set UAI=, BucketName=, and optionally CFN_ARTIFACTS_BUCKET=
```

2. Push a change to `lab1/` or run the workflow manually:

```bash
git add .; git commit -m "update"; git push
# or
gh workflow run deploy-s3-cfn.yml --field UAI=uai0123456 --field BucketName=my-bucket
```

Notes
- `CFN_ARTIFACTS_BUCKET` must exist before packaging; set it in `lab1/cfn-params.env` (or as a repo secret).
- The workflow validates `UAI` and `BucketName`, packages nested templates, then deploys.

Need the IAM policy JSON or a committed `lab1/cfn-params.env` example? Tell me which and I will add it.
