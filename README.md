# kinesis-firehose-iceberg
kinesis-firehose-iceberg

# Labs
```

export AWS_ACCOUNT_ID="XX"
export BUCKET_NAME="XX"
export AWS_REGION="XX"

aws iam create-policy --policy-name FirehoseGlueS3Policy --policy-document '{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "glue:GetTable",
                "glue:GetDatabase",
                "glue:UpdateTable"
            ],
            "Resource": [
                "arn:aws:glue:'"$AWS_REGION"':'"$AWS_ACCOUNT_ID"':catalog",
                "arn:aws:glue:'"$AWS_REGION"':'"$AWS_ACCOUNT_ID"':database/*",
                "arn:aws:glue:'"$AWS_REGION"':'"$AWS_ACCOUNT_ID"':table/*/*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:AbortMultipartUpload",
                "s3:GetBucketLocation",
                "s3:GetObject",
                "s3:ListBucket",
                "s3:ListBucketMultipartUploads",
                "s3:PutObject",
                "s3:DeleteObject"
            ],
            "Resource": [
                "arn:aws:s3:::'"$BUCKET_NAME"'",
                "arn:aws:s3:::'"$BUCKET_NAME"'/*"
            ]
        }
    ]
}'


aws iam create-role --role-name FirehoseGlueS3Role --assume-role-policy-document '{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Service": "firehose.amazonaws.com"
            },
            "Action": "sts:AssumeRole"
        }
    ]
}'

aws iam attach-role-policy --role-name FirehoseGlueS3Role --policy-arn arn:aws:iam::867098943567:policy/FirehoseGlueS3Policy



FIREHOSE
[
  {
    "DestinationDatabaseName": "default",
    "DestinationTableName": "my_table",
    "UniqueKeys": [
      "customer_id","phone_numbers","city","date","address"
    ],
    "S3ErrorOutputPrefix": "s3://XXX/error-logs/"
  }
]


DEFINE SCHEMA IN GLUE
[
  {
    "Name": "customer_id",
    "Type": "string",
    "Comment": "customer_id"
  },
  {
    "Name": "phone_numbers",
    "Type": "string",
    "Comment": "Phone numbers"
  },
  {
    "Name": "city",
    "Type": "string",
    "Comment": "City"
  },
  {
    "Name": "date",
    "Type": "date",
    "Comment": "Date"
  },
  {
    "Name": "address",
    "Type": "string",
    "Comment": "Address"
  }
]


# --------
CLEANUP
# -------
aws iam detach-role-policy --role-name FirehoseGlueS3Role --policy-arn arn:aws:iam::$AWS_ACCOUNT_ID:policy/FirehoseGlueS3Policy
aws iam delete-role --role-name FirehoseGlueS3Role
aws iam delete-policy --policy-arn arn:aws:iam::$AWS_ACCOUNT_ID:policy/FirehoseGlueS3Policy

```

# Python Script
```
try:
    import re
    import os
    import json
    import boto3
    import datetime
    import uuid
    from datetime import datetime
    import json
    from faker import Faker
    import random
    import faker
except Exception as e:
    print("Error : {} ".format(e))


def main(DeliveryStreamName=''):
    for i in range(1, 25):
        faker = Faker()
        json_data = {
            "name": faker.name(),
            "phone_numbers": faker.phone_number(),
            "city": faker.city(),
            "address": faker.address(),
            "date": str(faker.date()),
            "customer_id": str(random.randint(1, 5)),
            "new_col": "newcol"
        }
        print(json_data)

        client = boto3.client(
            "firehose"
        )

        response = client.put_record(
            DeliveryStreamName=DeliveryStreamName,
            Record={
                'Data': json.dumps(json_data)
            }
        )
        print(response)


# main(DeliveryStreamName='PUT-ICE-CnxUh')
main(DeliveryStreamName='')




```

