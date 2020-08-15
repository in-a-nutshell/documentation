Are you looking to build a docker image where you would need to make AWS API calls during the docker build process, for example download files from Amazon S3 to the docker container and use the docker image with AWS services like ECS, SageMaker etc. ?

In this blog, we'll look at how we can pass AWS credentials, i.e. AWS access key, secret access key and session token to docker, so that docker can use these credentials to make AWS API calls.

When making any AWS API calls, AWS uses your security credentials to verify who you are and whether you have permission to access the resources that you are requesting. AWS uses the security credentials to authenticate and authorize your requests(1).

To make a specific AWS API call, you'll need to make sure that the policy attached to the IAM user/role has the necessary permissions to make the AWS API calls. You can read more about policies and permissions in the AWS documentation(2).

Below is a sample Dockerfile and a bash script that can be used to achieve this:

Dockerfile:
```docker
FROM python:3

# Set/initialise ARGS necessary for build.
ARG AWS_ACCESS_KEY_ID=""
ARG AWS_SECRET_ACCESS_KEY=""
ARG AWS_SESSION_TOKEN=""

# Set environment vars for AWS CLI to work properly
ENV AWS_ACCESS_KEY_ID=${AWS_ACCESS_KEY_ID}
ENV AWS_SECRET_ACCESS_KEY=${AWS_SECRET_ACCESS_KEY}
ENV AWS_SESSION_TOKEN=${AWS_SESSION_TOKEN}
ENV AWS_DEFAULT_REGION="us-east-1"

ENV PATH="/tmp:${PATH}"

RUN pip install awscli boto3

# RUN AWS CLI commands, using the credentials passed to the docker container:
# Replace bucket and prefix with S3 path to your object(s)
RUN aws s3 sync s3://bucket/prefix /tmp/prefix

WORKDIR /tmp
```

Sample bash script to build the docker image and push it to an ECR repository in your account:

```bash
#!/bin/bash

# The name of your docker image
image_name=test-docker-image

account=$(aws sts get-caller-identity --query Account --output text)

# Get the region defined in the current configuration (default to us-east-1 if none defined)
region=$(aws configure get region)
region=${region:-us-east-1}

fullname="${account}.dkr.ecr.${region}.amazonaws.com/${image_name}:latest"

# If the repository doesn't exist in ECR, create it.
aws ecr describe-repositories --repository-names "${image_name}" > /dev/null 2>&1

if [ $? -ne 0 ]
then
    aws ecr create-repository --repository-name "${image_name}" > /dev/null
fi

# Get the login command from ECR and execute it directly
$(aws ecr get-login --region ${region} --no-include-email)

credentials=$(aws sts get-session-token --duration-seconds 900)

# You can also get credentials from a role.
# Uncomment the below line to get credentials from role
# credentials=$(aws sts assume-role --role-arn "arn:aws:iam::111222333444:role/myrolename" --role-session-name AWSCLI-Session)

access_key=$(echo "$credentials"  | jq -r '.Credentials' | jq -r '.AccessKeyId')
secret_access_key=$(echo "$credentials"  | jq -r '.Credentials' | jq -r '.SecretAccessKey')
session_token=$(echo "$credentials"  | jq -r '.Credentials' | jq -r '.SessionToken')

# Build the docker image locally with the image name and then push it to ECR
# with the full name.

docker build  -t ${image_name} --build-arg AWS_ACCESS_KEY_ID=$access_key --build-arg AWS_SECRET_ACCESS_KEY=$secret_access_key --build-arg AWS_SESSION_TOKEN=$session_token .
docker tag ${image_name} ${fullname}

docker push ${fullname}
```

References:

(1) https://docs.aws.amazon.com/general/latest/gr/aws-security-credentials.html  
(2) https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies.html
