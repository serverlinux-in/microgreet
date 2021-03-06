# microgreet
A simple service to demonstrate docker, swarm, ecs, cloudformation

## Pre-reqs
* You will need a docker version compatible with 17.12.1. So any docker version greater or equal to 17.12.1 should work.
* This has been tested on a mac but should work on a linux system as well. For windows the volume mounting and docker.sock locations may be different
* To run the cloudformation you need to use aws cli and need to make sure the credentials are set wither in the ~/.aws/credentials or as env variables AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY
The docker file is multi-stage. One stage is for testing and the other is to build the final image
### To Test the code before production
```
docker build --target tester -t miladino/hello-django:test .
```
The output of this step will show the test results
### To build the final image
```
docker build --target builder -t miladino/hello-django:1.3 .
```
### Run standalone container 
```
docker container run -d -p 80:8000 miladino/hello-django:1.3
```
### Run swarm for dev 
```
./deploy-swarm.sh <image-tag>
```
```
./deploy-swarm.sh 1.3
```
### Run compose  
```
./deploy-compose.sh <image-tag>
```
```
./deploy-compose.sh 1.3
```
You can also use the jenkins image to build and test.
### To run the prod stack
Make sure the aws cli credentials are set.
##### These parameters are required:
- VpcId = the vpc you would like to use 
- SubnetId = the subnet to use (belonging to VpcId)
- ExecutionRoleArn = The execution IAM role for the task. This is needed for cloudwatch log creation
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ecr:GetAuthorizationToken",
                "ecr:BatchCheckLayerAvailability",
                "ecr:GetDownloadUrlForLayer",
                "ecr:BatchGetImage",
                "logs:CreateLogStream",
                "logs:PutLogEvents"
            ],
            "Resource": "*"
        }
    ]
}
```
##### These parameters are optional:
- DefaultContainerCpu = Amount of CPU for the containers. default 256
- DefaultContainerMemory = Amount of CPU for the containers. default 512
- DefaultServiceCpuScaleOutThreshold = Average CPU value to trigger auto scaling out. default 50


Make sure a cluster with the name provided to  --stack-name does not exist other wise the stack will not get created.
In the example belw the stack name is greetings-stack.
```
aws cloudformation create-stack --stack-name greetings-stack --capabilities "CAPABILITY_IAM" "CAPABILITY_NAMED_IAM" \
--template-body file://cloudformation-prod.json \
--parameters ParameterKey=VpcId,ParameterValue=vpc-###### \
ParameterKey=SubnetId,ParameterValue=subnet-##### \
ParameterKey=ExecutionRoleArn,ParameterValue='arn:aws:iam::#########:role/ecsTaskExecutionRole'
```
You can also use the aws console to run the cloudformation template

### TO access the service
- navigate to localhost if you are deploying locally
- navigate to the public ip of the task. This value can be found in the aws console. Go to ECS>your cluster>your service>Tasks>click on task number
### JENKINS
There is a jenkins image that has been made that you can use. The image is quite big. Did not have time to improve the image size.
```
docker container run -d -p 8080:8080 -v /var/run/docker.sock:/var/run/docker.sock miladino/hello-jenkins:2.0
```
##### Creds
username:admin
password:AdminPassword

The Project is called Build-Greetings-Version. The parameter it asks for is the git tag you would like to use so make sure it exists.
You can use 1.4 as a test since that tag is present on github

#### Improvements:
- Add nginx infront of service
- create the network with cloudformation (vpc, subnet)
- create a hosted zone on route53 and separate the nginx service from the greeting service
- take in image tags as parameters in the cloudformation
- output the public ip of the task(Not sure if possible)