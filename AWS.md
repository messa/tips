
Amazon Web Services (AWS)
=========================

Overview of AWS compoments
--------------------------

Top AWS components and services you should know about:

- EC2 - basically VPS
  - EBS (Elastic Block Service) - hosted storage service, block based - acts as disk volumes for EC2
- Route53 - hosted DNS
- S3 - hosted storage service (file based)
- IAM - user and access management for your AWS account

(TODO)


Identity and Access Management (IAM)
------------------------------------

### S3 IAM policy for read/write access only to a single bucket

Consists of three parts:

- allow list bucket contents (applied on bucket itself)
- allow file operations (applied on bucket contents)
- allow list buckets (applied on whole S3 account)
  - this doesn't allow to list or get files, just list bucket names in your account

Policy document:

    {
      "Statement": [
        {
          "Effect": "Allow",
          "Action": [
            "s3:ListBucket",
            "s3:GetBucketLocation",
            "s3:ListBucketMultipartUploads"
          ],
          "Resource": "arn:aws:s3:::examplebucketname",
          "Condition": {}
        },
        {
          "Effect": "Allow",
          "Action": [
            "s3:AbortMultipartUpload",
            "s3:DeleteObject",
            "s3:DeleteObjectVersion",
            "s3:GetObject",
            "s3:GetObjectAcl",
            "s3:GetObjectVersion",
            "s3:GetObjectVersionAcl",
            "s3:PutObject",
            "s3:PutObjectAcl",
            "s3:PutObjectAclVersion"
          ],
          "Resource": "arn:aws:s3:::examplebucketname/*",
          "Condition": {}
        },
        {
          "Effect": "Allow",
          "Action": "s3:ListAllMyBuckets",
          "Resource": "*",
          "Condition": {}
        }
      ]
    }
