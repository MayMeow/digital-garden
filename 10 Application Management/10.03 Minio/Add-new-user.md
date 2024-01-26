# Add new user to Minio server and assign bucket

## Create user

```bash
mc admin user add mystorage <NEW-USER-ACCESS-KEY> <NEW-USER-SECRET-KEY>
```

*New users dont have any access on server, You can just login so you will need to setup policy and assign it to newly created user*

## Create Bucket

```bash
mc mb mystorage/my-site
```

## Create policy.

Add this policy content to `policy-name.json`.

```json
{
    "Version": "2012-10-17",
    "Statement": [
      {
        "Action": [
          "s3:GetObject",
          "s3:PutObject",
          "s3:DeleteObject",
          "s3:GetBucketLocation",
          "s3:ListBucket",
          "s3:ListAllMyBuckets"
        ],
        "Effect": "Allow",
        "Resource": [
          "arn:aws:s3:::my-site/*"
        ],
        "Sid": "Public"
      }
    ]
  }
```

## Install policy to server

```bash
mc admin policy add mystorage policy-name policy-name.json
```

## Assign policy to user

```bash
mc admin policy set mystorage "policy-name" user=<NEW-USER-ACCESS-KEY>
```
