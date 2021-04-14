# React - QUEST

This test can be found [here](https://github.com/rearc/quest)

## Completed Quest Items

What do i have to submit ?
1. Terraform and/or Cloudformation template(s)
	- This repo contains all Cloudformation templates
2. Dockerfile
	- Located `docker/quest/Dockerfile`
3. Screenshot of the secret page
    - ![](https://user-images.githubusercontent.com/7052925/114751663-a9486c00-9d12-11eb-864e-60c990f7cc59.png)
4. URL of your stack
	- [Quest App](https://jordan-quest-alb-2119162710.us-west-2.elb.amazonaws.com/)

## Repo Overview

![](https://user-images.githubusercontent.com/7052925/114756938-7c975300-9d18-11eb-8284-628b63e4aaeb.png)

The repository consists of a set of nested templates that deploy the following:

- A tiered VPC with public and private subnets, spanning an AWS region.
- A highly available ECS cluster deployed across two Availability Zones and backed by Fargate.
- A pair of NAT gateways (one in each zone) to handle outbound traffic.
- An Application Load Balancer (ALB) to the public subnets to handle inbound traffic.
- ALB path-based routes for the ECS service to route the inbound traffic.
## Template Details

| Template  | Description                    |
| ------------- | ------------------------------ |
| `template.yaml`      | This is the root template - deploy it to CloudFormation and it includes all of the others automatically.       |
| `infrastructure/vpc/template.yaml`   | This template deploys a VPC with a pair of public and private subnets spread across two Availability Zones. It deploys an Internet gateway, with a default route on the public subnets. It deploys a pair of NAT gateways (one in each zone), and default routes for them in the private subnets.     |
| `infrastructure/security-groups/template.yaml`      | This template contains the security groups required by the entire stack. They are created in a separate nested template, so that they can be referenced by all of the other nested templates.      |
| `infrastructure/alb/template.yaml`   | This template deploys an ALB to the public subnets, which exposes the ECS services.   |
| `infrastructure/ecs/template.yaml`   | This template deploys an ECS cluster to the private subnets. Along with a Service for a Docker image.   |

## Manual Deployment

Instructions on how to setup this repo on an AWS account using the AWS CLI.

### Prerequisites

- AWS CLI
- S3 Bucket
- ARN of SSL certificate
- URI of Docker image

### Setup

Clone this repo and navigate into the projects root directory.

##### Example #1

`cd <project_root_directory>`

### Package

Package and upload the Cloudformation templates into the S3 Bucket.

| Flag | Description | Required |
| ------------- | ------------------------------ | ------------- |
| `s3-bucket` | The S3 Bucket name where the templates will be uploaded. | Yes |

##### Example #1

`aws cloudformation package --template-file template.yaml --output-template packaged.yaml --s3-bucket <bucket_name>`

### Deploy

Deploy the packaged templates into your AWS account with the required parameters.

| Flag | Description | Required |
| ------------- | ------------------------------ | ------------- |
| `stack-name` | The name to be prefixed on all resources created.       | Yes |

| Flag | Description | Required |
| ------------- | ------------------------------ | ------------- |
| `SSLCert` | The SSL Certificate that will be attached to the Application Load Balancer. | Yes |
| `Image` | The URI of the Docker image to be deployed into ECS. | Yes |
| `Secret` | The secret word found in the Quest. | No |

##### Example #1

`aws cloudformation deploy --template-file ./packaged.yaml --stack-name <project_name> --parameters  SSLCert=<ssl_cert_arn> Image=<docker_image_uri> Secret=<secret_word>`

##### Example #2

`aws cloudformation deploy --template-file ./packaged.yaml --stack-name jordan-quest --parameter-overrides SSLCert="arn:aws:acm:us-west-2:11110000000:certificate/c084ddaf-f361-4f9b-90ec-48f1305bd770" Image="httpd:2.4" Secret="TwelveFactor"`

## Considerations

A posible TODO list:

- Deploy Private Link endpoints for ECR, ECS, and S3 to keep communication on the AWS backend.
- Remove the NATs for security purposes.
- Create a Lambda function for creating and importing the self-signed SSL certification into ACM. 

## Notes

1. For the first task `Deploy the app in AWS and find the secret page. Use Linux 64-bit as your OS (Amazon Linux preferred)`
    - I opted to use Fargate for simplicity (serveless) and mostly as an excuse to get better with the service.
    - Fargate `1.4.0` runs on an on Amazon Linux 2 based image [Versions](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/platform_versions.html#available_pv)

2. For the question `What if i find a bug ?`
    - Farget `1.4.0` uses the container runtime `Containerd` instead of `Docker`. For the `/docker` page of Quest it reports `You dont seem to be running in a Docker container` even though it is a container. This may have something to do with it depending on how you detect if the app is running in a docker container (Spitballing here) [Versions](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/platform_versions.html#available_pv)

