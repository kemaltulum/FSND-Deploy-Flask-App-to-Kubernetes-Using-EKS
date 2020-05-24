# Deploying a Flask API

This is the project starter repo for the fourth course in the [Udacity Full Stack Nanodegree](https://www.udacity.com/course/full-stack-web-developer-nanodegree--nd004): Server Deployment, Containerization, and Testing.

In this project you will containerize and deploy a Flask API to a Kubernetes cluster using Docker, AWS EKS, CodePipeline, and CodeBuild.

The Flask app that will be used for this project consists of a simple API with three endpoints:

- `GET '/'`: This is a simple health check, which returns the response 'Healthy'. 
- `POST '/auth'`: This takes a email and password as json arguments and returns a JWT based on a custom secret.
- `GET '/contents'`: This requires a valid JWT, and returns the un-encrpyted contents of that token. 

The app relies on a secret set as the environment variable `JWT_SECRET` to produce a JWT. The built-in Flask server is adequate for local development, but not production, so you will be using the production-ready [Gunicorn](https://gunicorn.org/) server when deploying the app.

## Initial setup
1. Fork this project to your Github account.
2. Locally clone your forked version to begin working on the project.

## Dependencies

- Docker Engine
    - Installation instructions for all OSes can be found [here](https://docs.docker.com/install/).
    - For Mac users, if you have no previous Docker Toolbox installation, you can install Docker Desktop for Mac. If you already have a Docker Toolbox installation, please read [this](https://docs.docker.com/docker-for-mac/docker-toolbox/) before installing.
 - AWS Account
     - You can create an AWS account by signing up [here](https://aws.amazon.com/#).
     
## Project Steps

Completing the project involves several steps:

1. Write a Dockerfile for a simple Flask API
2. Build and test the container locally
3. Create an EKS cluster
4. Store a secret using AWS Parameter Store
5. Create a CodePipeline pipeline triggered by GitHub checkins
6. Create a CodeBuild stage which will build, test, and deploy your code

For more detail about each of these steps, see the project lesson [here](https://classroom.udacity.com/nanodegrees/nd004/parts/1d842ebf-5b10-4749-9e5e-ef28fe98f173/modules/ac13842f-c841-4c1a-b284-b47899f4613d/lessons/becb2dac-c108-4143-8f6c-11b30413e28d/concepts/092cdb35-28f7-4145-b6e6-6278b8dd7527).

## Commands 

```bash
export TOKEN=`curl -d '{"email":"test@test.com","password":"password123"}' -H "Content-Type: application/json" -X POST localhost:8080/auth  | jq -r '.token'`

echo $TOKEN

curl --request GET 'http://127.0.0.1:8080/contents' -H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE1OTEzOTIzOTYsIm5iZiI6MTU5MDE4Mjc5NiwiZW1haWwiOiJ0ZXN0QHRlc3QuY29tIn0.5S5cXZ8fY2qyUiJ2gKQJE1B5PfDOCU29A9KBZOF40Zg" | jq .

sudo docker run -p 80:8080 --env-file=env_file jwt-api-test


ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)


TRUST="{ \"Version\": \"2012-10-17\", \"Statement\": [ { \"Effect\": \"Allow\", \"Principal\": { \"AWS\": \"arn:aws:iam::${ACCOUNT_ID}:root\" }, \"Action\": \"sts:AssumeRole\" } ] }"

aws iam create-role --role-name UdacityFlaskDeployCBKubectlRole --assume-role-policy-document "$TRUST" --output text --query 'Role.Arn'


echo '{ "Version": "2012-10-17", "Statement": [ { "Effect": "Allow", "Action": [ "eks:Describe*", "ssm:GetParameters" ], "Resource": "*" } ] }' > /tmp/iam-role-policy 

 aws iam put-role-policy --role-name UdacityFlaskDeployCBKubectlRole --policy-name eks-describe --policy-document file:///tmp/iam-role-policy

 kubectl get -n kube-system configmap/aws-auth -o yaml > /tmp/aws-auth-patch.yml

- rolearn: arn:aws:iam::044501667047:role/UdacityFlaskDeployCBKubectlRole
    username: build
    groups:
      - system:masters

  kubectl patch configmap/aws-auth -n kube-system --patch "$(cat /tmp/aws-auth-patch.yml)"


aws ssm put-parameter --name JWT_SECRET --value "YourJWTSecret" --type SecureString


kubectl get services simple-jwt-api -o wide

http://aa52388250eb746468e5513a58ab3935-2105079550.eu-central-1.elb.amazonaws.com

export TOKEN=`curl -d '{"email":"test@test.com","password":"test"}' -H "Content-Type: application/json" -X POST http://aa52388250eb746468e5513a58ab3935-2105079550.eu-central-1.elb.amazonaws.com/auth  | jq -r '.token'`
curl --request GET 'aa52388250eb746468e5513a58ab3935-2105079550.eu-central-1.elb.amazonaws.com/contents' -H "Authorization: Bearer ${TOKEN}" | jq 
```