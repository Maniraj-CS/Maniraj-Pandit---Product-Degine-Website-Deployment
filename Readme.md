# Deploy a Website to Amazon S3

## Introduction

This project explains how to deploy a static website to Amazon S3 using GitHub Actions.

GitHub Actions automatically uploads website files to an S3 bucket whenever new changes are pushed to the `main` branch.

This process helps automate website deployment without uploading files manually.

## How Deployment Works

1. Changes are made to the website files.
2. The changes are pushed to the `main` branch on GitHub.
3. GitHub Actions starts the deployment workflow.
4. The workflow connects to AWS using GitHub Secrets.
5. AWS CLI uploads the website files to the S3 bucket.
6. The website files are updated in the S3 bucket.

## Amazon S3 Setup

Create an S3 bucket with the following details:

* **Bucket Name:** `maniraj-pandit-projec-degine`
* **AWS Region:** `ap-southeast-2`

The bucket name and AWS region must match the values in the GitHub Actions workflow.

### Enable Static Website Hosting

Follow these steps to enable website hosting:

1. Open the AWS Management Console.
2. Go to Amazon S3.
3. Open the required S3 bucket.
4. Select the **Properties** tab.
5. Find **Static website hosting**.
6. Enable static website hosting.
7. Set the index document to `index.html`.
8. Save the changes.

The `index.html` file is the main page of the website.

## S3 Bucket Policy

A bucket policy controls access to files stored in an S3 bucket.

To allow public access to website files, add the following policy under **Permissions → Bucket policy**.

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

### Explanation of the Policy

* `Effect: Allow` — Allows the specified action.
* `Principal: "*"` — Applies the permission to everyone.
* `Action: s3:GetObject` — Allows files to be read or downloaded.
* `Resource` — Specifies which S3 objects the policy covers.
* `/*` — Includes all objects inside the bucket.

**Important:** This policy allows anyone to read the files in the bucket. Public access should only be enabled for files intended to be shared publicly.

AWS Block Public Access settings may need to be adjusted for this policy to work. Review the security implications before making changes.

## GitHub Secrets

GitHub Secrets securely stores sensitive information used by GitHub Actions.

Add the required secrets under:

**GitHub Repository → Settings → Secrets and variables → Actions**

| Secret Name                | Description                                                     |
| -------------------------- | --------------------------------------------------------------- |
| `AWS_ACCESS_KEY_ID`        | AWS access key ID                                               |
| `AWS_SECRET_ACCESS_KEY_ID` | AWS secret access key, if this is the name used in the workflow |

The secret names must match the names referenced in the workflow file.

**Security Note:** AWS credentials should never be written directly in the workflow file, README, or other public repository files.

The AWS user must have the required permissions to access the S3 bucket and upload, update, or delete objects.

## GitHub Actions Workflow

The workflow file is located at:

`.github/workflows/main.yml`

This file defines the steps required to automate the deployment process.

### Main Components

* **Trigger:** Starts the workflow when changes are pushed to the `main` branch.
* **Runner:** `ubuntu-latest` provides a temporary Linux environment for running the workflow.
* **Checkout:** `actions/checkout` downloads the repository files.
* **AWS Credentials:** `aws-actions/configure-aws-credentials` configures AWS access using GitHub Secrets.
* **AWS Region:** Specifies the AWS region as `ap-southeast-2`.
* **File Synchronization:** AWS CLI synchronizes the repository files with the S3 bucket.

## AWS S3 Sync Command

The following command is used to synchronize files with the S3 bucket:

```bash
aws s3 sync . s3://maniraj-pandit-projec-degine --delete
```

### Command Explanation

* `aws s3 sync` — Synchronizes files between a local directory and an S3 bucket.
* `.` — Represents the current directory.
* `s3://maniraj-pandit-projec-degine` — Specifies the destination S3 bucket.
* `--delete` — Removes destination files that are not present in the source directory.

The `--delete` option helps keep the S3 bucket synchronized with the repository. However, it can also remove files uploaded manually to the bucket.

### AWS S3 Sync vs Linux Rsync

`rsync` is a Linux command used to synchronize files between locations.

`aws s3 sync` is an AWS CLI command designed to synchronize files with Amazon S3.

The standard Linux `rsync` command does not directly support Amazon S3 without additional tools or configuration.

For this project, `aws s3 sync` is the simpler option.

## Check Deployment Status

After pushing changes to the `main` branch, follow these steps:

1. Open the GitHub repository.
2. Select the **Actions** tab.
3. Open the latest workflow run.
4. Check the deployment status.

If the deployment fails, check the following:

* GitHub Secrets are configured correctly.
* Secret names match the workflow file.
* AWS credentials are valid.
* The AWS user has the required S3 permissions.
* The bucket name is correct.
* The AWS region is set to `ap-southeast-2`.
* The source directory contains the required website files.

## Key Learnings

This project covers the following topics:

* Amazon S3 bucket creation and configuration.
* Static website hosting on Amazon S3.
* GitHub Actions workflow automation.
* AWS CLI and S3 file synchronization.
* GitHub Secrets for managing AWS credentials.
* Automated deployment using a CI/CD workflow.

## Conclusion

This project demonstrates how GitHub Actions can automate the deployment of a static website to Amazon S3.

It provides practical experience with AWS, GitHub Actions, CI/CD automation, and cloud storage.

The project can be extended with additional deployment steps and improvements as more DevOps concepts are learned.
