# MyApp: Build simple container in EKS (AWS managed kubernetes)

## Overview

This setup will deploy an EKS cluster in AWS, using CloudFormation. In this cluster, we will deploy a simple helloworld container.


### Infra
- Cloud: AWS
- [EKS](https://aws.amazon.com/eks): managed Kubernetes container orchestrator (on 3 availability zones for redundancy)
- [Docker Hub](https://hub.docker.com/): container registry to store hello image
- App: a simple hello world in nginx (folder `hello`)
- Code source: Github
- Deployment: [Terraform](https://www.terraform.io/) describes all components to be deployed. One command line will setup the infra


## Deploy

### Prerequisites
Please setup on your laptop:
- AWS cli and AWS account to deploy in `ap-south-1`
- Docker and Compose
- Github personal token with `admin:repo_hook, repo` rights from [here](https://github.com/settings/tokens)

### Test app on your laptop
Check the app locally:
```
cd hello
docker-compose up -d
curl localhost 8080
```

### Deploy to AWS
- Set a unique project prefix and your github token:
```
cd terraform
export TAG=hello-nugie   <-- please change to your prefix!
export GITHUBTOKEN=xxxx   <-- You token here
```
- Deploy EKS and CodePipeline: 
```
terraform init
terraform apply -var gitHubToken=$GITHUBTOKEN -var tag=$TAG
```

### Check EKS
- Cd `cd ..`
- Setup your kubeconfig: `aws eks --region eu-west-1 update-kubeconfig --name $TAG`
- Test: `kubectl get svc`
- Deploy hello app (using dockerhub hello image): `kubectl apply -f hello/hello.yml` 
- Test the app by curling the public DNS `EXTERNAL-IP:PORT` listed here: `kubectl get all`
- Curl: `curl [hostname_public]:8080`
- Delete deploy: `kubectl delete -f hello/hello.yml`

### Destroy all
To delete EKS, CodeBuild, CodePipeline (carefull, ALL will be deleted)
```
aws s3 rb s3://${TAG}-codebuild --force
aws s3 rb s3://${TAG}-codepipeline --force
cd terraform
terraform destroy
```