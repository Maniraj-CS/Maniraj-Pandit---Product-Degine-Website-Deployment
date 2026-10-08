# Deploy the Website to Amazon S3

This repository contains the files and setup used to deploy a static portfolio website to Amazon S3.

When you push a change to the `main` branch, GitHub Actions runs a workflow. The workflow copies files from this repository to an S3 bucket. It does not build the website first.

## How deployment works

1. You push a change to the `main` branch on GitHub.
2. GitHub Actions starts the workflow in `.github/workflows/main.yml`.
3. The workflow gets the repository files and connects to AWS using GitHub secrets.
4. The AWS CLI copies the files to the S3 bucket named `maniraj-pandit-projec-degine`.

The workflow uses the AWS region `ap-southeast-2`.

## S3 bucket setup

Create an S3 bucket named `maniraj-pandit-projec-degine` in the `ap-southeast-2` region. The bucket name and region must match the values in the workflow.

To host a website from the bucket:

1. Open the bucket in the AWS console.
2. Open **Properties** and turn on **Static website hosting**.
3. Set the index document to `index.html`.

The bucket policy below lets anyone read files in the bucket. Only make the files public if you want anyone to be able to visit the website.

## S3 bucket policy

Add this policy under the bucket's **Permissions → Bucket policy** page:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::maniraj-pandit-projec-degine/*"
    }
  ]
}
```

In simple terms:

- `Effect: "Allow"` means the rule gives permission.
- `Principal: "*"` means anyone can use this permission.
- `Action: "s3:GetObject"` lets people read or download files. It does not let them upload or delete files.
- `Resource` says which files the rule covers. The `/*` means all files inside this bucket.

AWS may block public access by default. For this policy to work, the bucket's **Block Public Access** settings must allow public access. Review AWS's warning before changing these settings.

## GitHub secrets

The workflow uses secret values to connect to AWS. Add these under your GitHub repository's **Settings → Secrets and variables → Actions**:

| Secret name | What to add |
| --- | --- |
| `AWS_ACCESS_KEY_ID` | Your AWS access key ID |
| `AWS_SECRET_ACCESS_KEY_ID` | The matching AWS secret access key |

The second secret name may look unusual, but it must match the name used in the workflow. Do not write AWS keys in this README or anywhere in the repository.

The AWS user for these keys needs permission to list the bucket and add, change, and delete files in it.

## Workflow file

The workflow is in `.github/workflows/main.yml`. Here is what its main parts do:

- **When it runs:** It runs when you push a change to the `main` branch. A push to another branch does not start this deployment.
- **Where it runs:** `runs-on: ubuntu-latest` asks GitHub to provide a temporary Linux computer.
- **Get the repository:** `actions/checkout` downloads the repository files onto that computer.
- **Connect to AWS:** `aws-actions/configure-aws-credentials` reads the AWS secrets and sets the region to `ap-southeast-2`.
- **Copy files to S3:** `aws s3 sync . s3://maniraj-pandit-projec-degine --delete` copies files from the repository's top-level folder (`.`) to the bucket.

The `--delete` option removes files from the bucket if those files are no longer in the repository. This keeps the bucket similar to the repository, but it can also remove files that someone added to the bucket by hand.

### Can this use `rsync`?

The command in the workflow is `aws s3 sync`. It is an AWS CLI command made to copy files to Amazon S3. It is not the separate Linux `rsync` command.

The normal `rsync` command does not copy files directly to an S3 bucket. You can set up extra tools to use rsync-style copying, but that adds extra setup. For this project, `aws s3 sync` is the simpler choice and already does the job.

## Check a deployment

After pushing to `main`, open the repository's **Actions** tab on GitHub and select the latest workflow run. If it fails, check that:

- Both GitHub secrets are added with the exact names shown above.
- The AWS keys have the required bucket permissions.
- The bucket name and AWS region match the workflow.
