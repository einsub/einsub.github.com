---
layout: post
title: "AWS CLI로 Fargate ECS Service를 띄우고 Service discovery로 통신하기"
date: 2019-01-01 19:24:12
author: Reid
categories:
  - engineering
tags:
  - aws
  - ecs
  - fargate
  - service discovery
  - iam
  - docker
  - load balancer
  - ecr
  - vpc
  - subnet
  - router
  - auto scaling
published: true
---

이 문서는 Amazon Elastic Container Service(ECS)에 Docker 이미지를 올려 서비스를 구동하는 절차를 담은 튜토리얼입니다. 서버 두 개를 각각의 서비스에 띄워 AWS Service Discovery를 이용해 통신합니다. ECS의 구동 방식으로는 Fargate를 이용합니다.

> 참고: 본 문서의 모든 예제는 macOS를 기준으로 작성되었습니다.

## 목차

- [AWS CLI 구성](#aws-cli-구성)
    - [Python 설치](#python-설치)
    - [AWS CLI 설치](#aws-cli-설치)
- [IAM 사용자 생성과 권한 설정](#iam-사용자-생성과-권한-설정)
    - [Administrator 사용자로 인증](#administrator-사용자로-인증)
    - [ECS 관리용 사용자 만들기](#ecs-관리용-사용자-만들기)
    - [사용자에게 AWS Managed Policy 연결](#사용자에게-aws-managed-policy-연결)
    - [사용자에게 Custom Policy 연결](#사용자에게-custom-policy-연결)
    - [Task 실행 역할 연결](#task-실행-역할-연결)
    - [ECS 관리용 사용자로 인증](#ecs-관리용-사용자로-인증)
- [애플리케이션 작성](#애플리케이션-작성)
    - [Gate server 작성](#gate-server-작성)
    - [Data server 작성](#data-server-작성)
    - [.dockerignore 작성](#dockerignore-작성)
- [로컬 환경의 Docker에서 실행하기](#로컬-환경의-docker에서-실행하기)
    - [Docker 설치](#docker-설치)
    - [Dockerfile 작성](#dockerfile-작성)
    - [Docker 이미지 빌드](#docker-이미지-빌드)
    - [Docker 네트워크 환경 설정](#docker-네트워크-환경-설정)
    - [구동 테스트](#구동-테스트)
- [ECR에 이미지 올리기](#ecr에-이미지-올리기)
    - [ECR 로그인](#ecr-로그인)
    - [Repository 만들기](#repository-만들기)
    - [ECR에 이미지 등록하기](#ecr에-이미지-등록하기)
- [VPC 인프라 구성](#vpc-인프라-구성)
    - [기존의 VPC 확인](#기존의-vpc-확인)
    - [VPC 만들기](#vpc-만들기)
    - [Subnet 만들기](#subnet-만들기)
    - [Internet Gateway 만들기](#internet-gateway-만들기)
    - [NAT Gateway 만들기](#nat-gateway-만들기)
    - [Routing Table 만들기](#routing-table-만들기)
    - [Subnet을 Routing Table에 연결](#subnet을-routing-table에-연결)
- [Load Balancer 구성](#load-balancer-구성)
    - [Security Group 만들기](#security-group-만들기)
    - [Load Balancer 만들기](#load-balancer-만들기)
    - [Target Group 등록](#target-group-등록)
    - [Listener 생성](#listener-생성)
- [Service Discovery 구성](#service-discovery-구성)
    - [Namespace 생성](#namespace-생성)
    - [Service Discovery의 Service 생성](#service-discovery의-service-생성)
    - [Private DNS로 주소 변경](#private-dns로-주소-변경)
- [ECS 구성](#ecs-구성)
    - [Cluster 생성](#cluster-생성)
    - [Log Group 생성](#log-group-생성)
    - [Task Definition 작성 및 등록](#task-definition-작성-및-등록)
    - [Security Group 만들기](#security-group-만들기)
    - [Service 생성](#service-생성)
- [Auto Scaling Group 설정](#auto-scaling-group-설정)
    - [Service를 Scale 가능한 타겟으로 등록](#service를-scale-가능한-타겟으로-등록)
    - [Scaling Policy 연결](#scaling-policy-연결)
- [최종 테스트](#최종-테스트)
    - [동작 테스트](#동작-테스트)
    - [Scale 테스트](#scale-테스트)
- [ECS의 유지보수](#ecs의-유지보수)
    - [Task Definition 업데이트](#task-definition-업데이트)
    - [Service 강제 배포](#service-강제-배포)
    - [Task Scaling](#task-scaling)
- [리소스 초기화](#리소스-초기화)

---

## AWS CLI 구성

AWS의 각종 기능을 terminal에서 명령어만으로 동작시키기 위해 AWS CLI를 설치합니다. 이미 설치되어 있다면 [IAM 사용자 생성과 권한 설정](#iam-사용자-생성과-권한-설정) 섹션으로 넘어갑니다.

### Python 설치

`python --version` 명령으로 이미 설치되어 있는지 여부를 확인하고 설치 되어 있지 않다면 다음 명령으로 설치합니다.

```sh
$ brew install python
```

Python 패키지 관리자인 pip도 `pip --version`으로 설치 여부를 확인하고, 없으면 다음 명령으로 설치합니다.

```bash
$ curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
$ python get-pip.py --user
```

### AWS CLI 설치

```bash
$ pip install awscli --upgrade --user
```

위 명령을 입력하면 출력되는 결과 메시지의 마지막 문장을 참고하여 python의 bin 폴더를 PATH에 추가합니다.

‘**/Users/leeinsub/Library/Python/2.7/bin**’ which is not on PATH. Consider adding this directory to PATH or, if you prefer to suppress this warning, use --no-warn-script-location.

```bash
export PATH=${HOME}/Library/Python/2.7/bin:${PATH}
```

정상적으로 설치가 되었는지 확인합니다.

```bash
$ aws --version
aws-cli/1.16.60 Python/2.7.15 Darwin/18.2.0 botocore/1.12.50
```

## IAM 사용자 생성과 권한 설정

### Administrator 사용자로 인증

ECS 관리용 사용자를 생성하기 위해 일단 administrator 사용자로 인증합니다.

```sh
$ aws configure
AWS Access Key ID [None]: AKACKWJDFANFAKJSDIWE
AWS Secret Access Key [None]: TKKsdnFC8weKJdlwD981KWEnca0/H2Lqwnc9/QA1
Default region name [None]:  ap-northeast-2
Default output format [None]: json
```

### ECS 관리용 사용자 만들기

```sh
$ aws iam create-user --user-name ecs-user
{
    "User": {
        "UserName": "ecs-user",
        "Path": "/",
        "CreateDate": "2018-12-30T12:30:59Z",
        "UserId": "AIDAJ2A57BVIOQ3WUM7JK",
        "Arn": "arn:aws:iam::759375304948:user/ecs-user"
    }
}
```

```sh
$ aws iam create-access-key --user-name ecs-user
{
    "AccessKey": {
        "UserName": "ecs-user",
        "Status": "Active",
        "CreateDate": "2018-12-30T12:31:40Z",
        "SecretAccessKey": "AZ9mD+CidjSDjncv93SDjmcSjF92jCpajfWjjQnx",
        "AccessKeyId": "AKICMLISJFNLASDJCKAW"
    }
}
```

ECS 관리용 사용자로 인증하기 위해 `AccessKeyId`와 `SecretAccessKey`를 기억해 둡니다.

### 사용자에게 AWS Managed Policy 연결

ECS 관리자 권한 정책 연결

```sh
$ aws iam attach-user-policy --policy-arn arn:aws:iam::aws:policy/AmazonECS_FullAccess --user-name ecs-user
```

ECR 관리자 권한 정책 연결

```
$ aws iam attach-user-policy --policy-arn arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryFullAccess --user-name ecs-user
```

### 사용자에게 Custom Policy 연결

NAT Gateway를 생성 할 때 Elastic IP(EIP)를 부여하는 권한과 AutoScaling을 설정 할 때 역할을 전달하는 권한은 직접 policy를 만들어 연결합니다.

다음 파일을 작성해서 `aws iam create-policy` 명령에 인자로 전달합니다.

ecs-user-policy.json
```json
{
	"Version": "2012-10-17",
	"Statement": [{
		"Effect": "Allow",
		"Action": [
			"iam:PassRole"
		],
		"Resource": "arn:aws:iam::759375304948:role/ApplicationAutoscalingECSRole"
	}, {
		"Effect": "Allow",
		"Action": [
			"ec2:DescribeAddresses",
			"ec2:AllocateAddress",
			"ec2:DescribeInstances",
			"ec2:AssociateAddress"
		],
		"Resource": "*"
	}]
}
```

```sh
$ aws iam attach-user-policy --policy-arn arn:aws:iam::759375304948:policy/ecsUserPolicy --user-name ecs-user
```

### Task 실행 역할 연결

ECS 에이전트가 사용자를 대신해 ECR로부터 이미지를 가져오고 CloudWatch에 로그를 전달하기 위해 권한을 위임합니다. 다음 파일을 작성해서 `aws iam create-role` 명령에 인자로 전달합니다.

task-execution-assume-role.json
```json
{
  "Version": "2008-10-17",
  "Statement": [
    {
      "Sid": "",
      "Effect": "Allow",
      "Principal": {
        "Service": "ecs-tasks.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

```sh
$ aws iam create-role --role-name ecsTaskExecutionRole --assume-role-policy-document file://task-execution-assume-role.json
{
    "Role": {
        "AssumeRolePolicyDocument": {
            "Version": "2008-10-17",
            "Statement": [
                {
                    "Action": "sts:AssumeRole",
                    "Principal": {
                        "Service": "ecs-tasks.amazonaws.com"
                    },
                    "Effect": "Allow",
                    "Sid": ""
                }
            ]
        },
        "RoleId": "AROAJ5ERGD2RUG3QWVWNK",
        "CreateDate": "2018-12-30T12:35:42Z",
        "RoleName": "ecsTaskExecutionRole",
        "Path": "/",
        "Arn": "arn:aws:iam::759375304948:role/ecsTaskExecutionRole"
    }
}
```

```sh
$ aws iam attach-role-policy --role-name ecsTaskExecutionRole --policy-arn arn:aws:iam::aws:policy/service-role/AmazonECSTaskExecutionRolePolicy
```

### ECS 관리용 사용자로 인증

후속 작업은 Administrator가 아닌 처음에 만들었던 ECS 관리용 사용자를 이용하기 위해 인증 정보를 변경합니다.

```sh
$ aws configure
AWS Access Key ID [None]: AKICMLISJFNLASDJCKAW
AWS Secret Access Key [None]: AZ9mD+CidjSDjncv93SDjmcSjF92jCpajfWjjQnx
Default region name [None]:  ap-northeast-2
Default output format [None]: json
```

## 애플리케이션 작성

클라이언트의 요청을 받아서 서버 간 통신을 거쳐 클라이언트에게 응답을 내려주는 간단한 구성을 위해 두 개의 서버를 준비합니다. Gate server는 클라이언트로부터 요청을 받아 data server로 넘겨주는 역할을 하고 data server는 '응, 답!'이라는 메시지를 gate server에 반환합니다. 최종적으로 gate server는 요청했던 클라이언트에게 그 메시지를 넘겨줍니다.

최종적인 디렉토리 구조는 다음과 같습니다.

```
ecs-test
├── data-server
│   ├── .dockerignore
│   ├── Dockerfile
│   ├── ecs-service-discovery.json
│   ├── fargate-task.json
│   ├── index.js
│   └── package.json
├── gate-server
│   ├── .dockerignore
│   ├── Dockerfile
│   ├── ecs-service-discovery.json
│   ├── fargate-task.json
│   ├── index.js
│   ├── package.json
│   └── scale-config.json
├── ecs-user-policy.json
└── task-execution-assume-role.json
```

### Gate server 작성

```sh
$ npm install --save express request
```

gate-server/index.js
```javascript
const express = require('express');
const app = express();
const request = require('request')

app.get('/', (req, res) => {
  request('http://localhost:4000/', (err, response, body) => {
    console.log('데이터 서버로부터 응답을 받았습니다:')
    console.log(body)
    res.send(body)
  })
});

app.listen(80, () => console.log('Server listening on port 80!'));
```

### Data server 작성

```sh
$ npm install --save express
```

data-server/index.js
```javascript
const express = require('express');
const app = express();

app.get('/', (req, res) => {
  console.log('서버로부터 요청을 받았습니다. 응답을 전송합니다.')
  res.send('응, 답!');
});

app.listen(4000, () => console.log('Data server listening on port 4000!'));
```

### .dockerignore 작성

로컬에 설치된 모듈들과 디버깅 로그들이 이미지에 포함되는 것을 막기 위해 .dockerignore 파일을 만들어서 무시합니다.

gate-server/.dockerignore<br/>
data-server/.dockerignore

```text
node_modules
npm-debug.log
```

## 로컬 환경의 Docker에서 실행하기

### Docker 설치

[Docker 홈페이지](https://www.docker.com)에서 로그인 후 [다운로드 페이지](https://www.docker.com/products/docker-desktop)에서 자신의 운영체제에 맞는 버전으로 내려 받아 설치합니다.

### Dockerfile 작성

Docker 이미지로 만들기 위해 Dockerfile을 작성합니다. 두 서버 내용은 같습니다.

gate-server/Dockerfile<br />
data-server/Dockerfile
```dockerfile
FROM node:8

# Create app directory
WORKDIR /usr/src/app

# Install app dependencies
# A wildcard is used to ensure both package.json AND package-lock.json are copied
# where available (npm@5+)
COPY package*.json ./

RUN npm install
# If you are building your code for production
# RUN npm install --only=production

# Bundle app source
COPY . .

EXPOSE 80
CMD [ "node", "index.js" ]
```

### Docker 이미지 빌드

각각의 프로젝트 폴더에서 다음 명령으로 Docker 이미지를 빌드합니다. 빌드 과정 중에 로그인을 요청하는 경우가 발생하면 `docker login`으로 Docker 서비스에 로그인을 합니다.

Gate server 빌드

```sh
$ docker build -t gate-server .
Sending build context to Docker daemon  30.72kB
Step 1/7 : FROM node:8
8: Pulling from library/node
54f7e8ac135a: Pull complete
d6341e30912f: Pull complete
087a57faf949: Pull complete
5d71636fb824: Pull complete
0c1db9598990: Pull complete
89669bc2deb2: Pull complete
468c418af6aa: Pull complete
8339c1e330a9: Pull complete
Digest: sha256:773480516f79e0948b02d7a06971b07bf76b08d706cf88a358b5b69dd4c83db0
Status: Downloaded newer image for node:8
 ---> 06f7c071f445
Step 2/7 : WORKDIR /usr/src/app
 ---> Running in 3f62e8f26a36
Removing intermediate container 3f62e8f26a36
 ---> 87cc264d154d
Step 3/7 : COPY package*.json ./
 ---> 9eabba590e1c
Step 4/7 : RUN npm install
 ---> Running in de7d4f0dda2d
npm WARN server@1.0.0 No description
npm WARN server@1.0.0 No repository field.

added 91 packages from 86 contributors and audited 184 packages in 4.721s
found 0 vulnerabilities

Removing intermediate container de7d4f0dda2d
 ---> efb13cc8c88f
Step 5/7 : COPY . .
 ---> 9292d7ea97be
Step 6/7 : EXPOSE 3000
 ---> Running in 1dbbf6250b86
Removing intermediate container 1dbbf6250b86
 ---> e05b1b15ce97
Step 7/7 : CMD [ "npm", "start" ]
 ---> Running in 3461636d3be6
Removing intermediate container 3461636d3be6
 ---> d1ff015a2f5e
Successfully built d1ff015a2f5e
Successfully tagged gate-server:latest
```

Data server 빌드

```sh
$ docker build -t data-server .
```

`docker build` 명령은 Dockerfile에서 명시한 순서대로 기반 이미지를 내려 받고, 명령을 순차적으로 실행하여 이미지를 만들어냅니다. 정상적으로 빌드되었는지는 `docker images` 명령으로 확인이 가능합니다.

```sh
$ docker images
REPOSITORY            TAG                 IMAGE ID            CREATED              SIZE
data-server           latest              b398a7e59bb5        About a minute ago   891MB
gate-server           latest              d1ff015a2f5e        10 minutes ago       896MB
node                  8                   06f7c071f445        4 days ago           889MB
```

### Docker 네트워크 환경 설정

`docker run` 명령으로 이미지를 실행합니다. `-p` 옵션은 외부 포트와 내부 포트를 매핑 시켜주고, `-d` 옵션은 프로세스를 백그라운드에서 실행시킵니다. 실시간으로 컨테이너의 출력을 보고 싶다면 `-d` 옵션을 빼고 foreground로 구동 할 수 있지만 `docker logs gate-server -f` 명령으로 background 구동 중에도 출력을 살펴 볼 수 있습니다.

Gate server 구동

```sh
$ docker run --name gate-server -d -p 80:80 gate-server
Server listening on port 80!
```

Data server 구동

```sh
$ docker run --name data-server -d data-server
Data server listening on port 4000!
```

동작 테스트

```
$ curl http://localhost
```

아무런 응답이 없습니다.

각각의 컨테이너는 독립된 네트워크 환경을 가지므로 gate server가 data server를 `localhost`로 바라 볼 수 없습니다. Docker의 기본 네트워킹 방식은 'default bridge network' 방식인데, 이 상태에서는 구동중인 컨테이너의 사설 ip로 접근이 가능하지만, 컨테이너의 이름으로 접근하는 것은 불가능합니다. 이를 가능하게 하려면 직접 bridge 방식의 네트워크를 생성하여 그 위에서 통신해야 합니다.

다음 명령으로 `ecs-test`라는 이름의 bridge 네트워크를 만듭니다.

```sh
$ docker network create --driver bridge ecs-test
69c6ea9737bcefbe70c4aae11d6f17fb0e3dfc719cd05b88859e544941dcef64
```

생성 된 네트워크를 확인합니다.  c

```sh
$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
9d1dc010f42c        bridge              bridge              local
69c6ea9737bc        ecs-test            bridge              local
944f9093e65f        host                host                local
775577ff5276        none                null                local
```

Gate server에서 바라보는 data server의 주소는 현재 `localhost`로 설정되어 있지만, bridge 네트워킹 방식을 이용하면 컨테이너 이름으로 접근을 해야 합니다. 소스 코드에서 해당 부분을 수정하고 다시 빌드합니다.

gate-server/index.js
```javascript
...

app.get('/', (req, res) => {
  request('http://data-server:4000/', (err, response, body) => {
    console.log('데이터 서버로부터 응답을 받았습니다:')
    console.log(body)
    res.send(body)
  })
});

...
```

```sh
docker build -t gate-server .
```

### 구동 테스트

빌드가 완료되면 생성한 네트워크 위에서 동작하도록 `--network ecs-test` 옵션을 붙여서 두 서버를 재시작합니다. 각각의 프로젝트에서 다음 명령을 실행하여 컨테이너를 제거하고 다시 구동합니다.

```sh
$ docker stop gate-server && docker rm gate-server
$ docker run --network ecs-test --name gate-server -d -p 80:80 gate-server

$ docker stop data-server && docker rm data-server
$ docker run --network ecs-test --name data-server -d data-server
```

다시 한번 테스트를 해봅니다.

```sh
$ curl http://localhost
응, 답!
```

## ECR에 이미지 올리기

Docker CLI로 ECR에 이미지를 등록하고 가져오려면 docker에 ECR 인증을 해주어야 합니다. Amazon에서는 이를 위해 `get-login`이라는 명령을 제공합니다.

### ECR 로그인

```sh
$ aws ecr get-login --no-include-email
```

출력된 `docker login ...` 전체 영역을 복사해 터미널에 붙여넣어 다시 실행합니다. 이 로그인은 12시간 동안 유효하고, 만료된 후에는 다시 로그인 과정을 거쳐야 합니다. awslabs에서 제공하는 [Amazon ECR Crendential Helper](https://github.com/awslabs/amazon-ecr-credential-helper)를 이용하면 그런 수고로움 없이 자동으로 로그인 과정을 진행합니다.

### Repository 만들기

제작한 docker 이미지들을 올릴 ECR repository들을 만듭니다.

```sh
$ aws ecr create-repository --repository-name gate-server
{
    "repository": {
        "registryId": "759375304948",
        "repositoryName": "gate-server",
        "repositoryArn": "arn:aws:ecr:ap-northeast-2:759375304948:repository/gate-server",
        "createdAt": 1546173627.0,
        "repositoryUri": "759375304948.dkr.ecr.ap-northeast-2.amazonaws.com/gate-server"
    }
}
```

데이터 서버도 동일한 명령을 수행합니다.

```bash
$ aws ecr create-repository --repository-name data-server
```

### ECR에 이미지 등록하기

각 이미지에 target 설정을 합니다. gate-server를 ECR로 올려야 하기 때문에 올라갈 ECR repository 주소를 명시해야 하는데 이는 `docker tag` 명령을 이용해서 가능합니다. 주소를 명시하지 않으면 기본으로 docker.io의 레포지토리로 설정됩니다. 이미지에 대한 tag는 따로 설정하지 않았기 때문에 latest 기본 태그를 이용하게 됩니다.

```
$ docker tag gate-server 759375304948.dkr.ecr.ap-northeast-2.amazonaws.com/gate-server

$ docker tag data-server 759375304948.dkr.ecr.ap-northeast-2.amazonaws.com/data-server
```

이제 ECR로 올릴 준비를 마쳤습니다. 현재의 이미지 목록을 다시 한번 살펴보면 동일한 이미지가 local용과 ECR용으로 구분 된 것을 볼 수 있습니다.

```sh
$ docker images
REPOSITORY                                                              TAG                 IMAGE ID            CREATED             SIZE
{AWS ID}.dkr.ecr.ap-northeast-2.amazonaws.com/gate-server               latest              6a8a2e2eb5d0        4 hours ago         896MB
gate-server                                                             latest              6a8a2e2eb5d0        4 hours ago         896MB
{AWS ID}.dkr.ecr.ap-northeast-2.amazonaws.com/data-server               latest              86b07d85d96f        5 hours ago         891MB
data-server                                                             latest              86b07d85d96f        5 hours ago         891MB
node                                                                    8                   06f7c071f445        5 days ago          889MB
```

마지막으로 ECR로 올리는 명령을 수행합니다:

```sh
$ docker push 759375304948.dkr.ecr.ap-northeast-2.amazonaws.com/gate-server
The push refers to repository [759375304948.dkr.ecr.ap-northeast-2.amazonaws.com/gate-server]
b93b0621b8f6: Pushed
e8568c9bad2b: Pushed
30028c99216f: Pushed
a5a71d956aaf: Pushed
8b3b5dcfe6ab: Pushed
72ebf42606f6: Pushed
4a6166f16a0e: Pushed
e02b32b1ff99: Pushed
f75e64f96dbc: Pushed
8f7ee6d76fd9: Pushed
c23711a84ad4: Pushed
90d1009ce6fe: Pushed
latest: digest: sha256:09af7b3f0a2ab4f2efcbbeabf4c26aa03739324a78e1b68705fd1bf55495a2b0 size: 2842
```

```
$ docker push 759375304948.dkr.ecr.ap-northeast-2.amazonaws.com/data-server
```

## VPC 인프라 구성

### 기존의 VPC 확인

이미 VPC가 존재한다면 다음과 같은 결과가 출력됩니다. 이후, load balancer나 Fargate 서비스 등을 만들 때 필요하므로 출력으로 나온 VPC ID를 기억해둡니다.

```sh
$ aws ec2 describe-vpcs
{
    "Vpcs": [
        {
            "VpcId": "vpc-7479991d",
            "InstanceTenancy": "default",
            "CidrBlockAssociationSet": [
                {
                    "AssociationId": "vpc-cidr-assoc-5bc62532",
                    "CidrBlock": "172.31.0.0/16",
                    "CidrBlockState": {
                        "State": "associated"
                    }
                }
            ],
            "State": "available",
            "DhcpOptionsId": "dopt-4a5bbb23",
            "CidrBlock": "172.31.0.0/16",
            "IsDefault": true
        }
    ]
}
```

AWS는 콘솔에 최초로 진입 할 때, 기본 VPC를 생성합니다. 그대로 기본 VPC 위에서 서비스들을 구동하겠다면 건너뛰어 [Load Balancer 구성](#load-balancer-구성) 섹션으로 넘어가도 됩니다. 기본 VPC가 없다거나 새로운 VPC를 만들어 이용하고 싶다면 다음 단계를 수행합니다.

### VPC 만들기

```sh
$ aws ec2 create-vpc --cidr-block 10.0.0.0/16
{
    "Vpc": {
        "VpcId": "vpc-0fd5393a547c8dc04",
        "InstanceTenancy": "default",
        "Tags": [],
        "CidrBlockAssociationSet": [
            {
                "AssociationId": "vpc-cidr-assoc-02c23f99069738e25",
                "CidrBlock": "10.0.0.0/16",
                "CidrBlockState": {
                    "State": "associated"
                }
            }
        ],
        "Ipv6CidrBlockAssociationSet": [],
        "State": "pending",
        "DhcpOptionsId": "dopt-4a5bbb23",
        "CidrBlock": "10.0.0.0/16",
        "IsDefault": false
    }
}
```

Service discovery를 설정 할 때, private DNS 지원을 받을 수 있도록 VPC의 enable DNS hostnames 설정을 켭니다.

```sh
$ aws ec2 modify-vpc-attribute --enable-dns-hostnames --vpc-id vpc-0fd5393a547c8dc04 
```

### Subnet 만들기

두 Availability Zone에 걸쳐 public subnet과 private subnet을 구성합니다. gate server는 public subnet에서 internet gateway를 통해 외부와 통신하고, data server는 private subnet에서 gate server와 내부 통신만 할 것입니다. 고가용성을 위해 public, private subnet은 두 AZ에 걸쳐서 두 개씩 만듭니다. 따러서 총 4개의 subnet이 만들어집니다.

서울 region에서 제공하는 availability zone을 확인합니다.

```sh
$ aws ec2 describe-availability-zones
{
    "AvailabilityZones": [
        {
            "State": "available",
            "ZoneName": "ap-northeast-2a",
            "Messages": [],
            "ZoneId": "apne2-az1",
            "RegionName": "ap-northeast-2"
        },
        {
            "State": "available",
            "ZoneName": "ap-northeast-2c",
            "Messages": [],
            "ZoneId": "apne2-az3",
            "RegionName": "ap-northeast-2"
        }
    ]
}
```

`ap-northeast-2a`와 `ap-northeast-2c`의 두 AZ를 확인 할 수 있습니다.

먼저 public으로 사용 할 subnet을 두 zone에 하나씩 만듭니다.

```sh
$ aws ec2 create-subnet --availability-zone ap-northeast-2a --vpc-id vpc-0fd5393a547c8dc04 --cidr-block 10.0.1.0/24
{
    "Subnet": {
        "AvailabilityZone": "ap-northeast-2a",
        "AvailableIpAddressCount": 251,
        "DefaultForAz": false,
        "Ipv6CidrBlockAssociationSet": [],
        "VpcId": "vpc-0fd5393a547c8dc04",
        "State": "pending",
        "MapPublicIpOnLaunch": false,
        "SubnetId": "subnet-0e8420c6f218aafd6",
        "CidrBlock": "10.0.1.0/24",
        "AssignIpv6AddressOnCreation": false
    }
}
```

```sh
$ aws ec2 create-subnet --availability-zone ap-northeast-2c --vpc-id vpc-0fd5393a547c8dc04 --cidr-block 10.0.2.0/24
{
    "Subnet": {
        "AvailabilityZone": "ap-northeast-2c",
        "AvailableIpAddressCount": 251,
        "DefaultForAz": false,
        "Ipv6CidrBlockAssociationSet": [],
        "VpcId": "vpc-0fd5393a547c8dc04",
        "State": "pending",
        "MapPublicIpOnLaunch": false,
        "SubnetId": "subnet-0a048beef8a45131b",
        "CidrBlock": "10.0.2.0/24",
        "AssignIpv6AddressOnCreation": false
    }
}
```

이번에는 private으로 사용 할 subnet을 두 zone에 하나씩 만듭니다.

```sh
$ aws ec2 create-subnet --availability-zone ap-northeast-2a --vpc-id vpc-0fd5393a547c8dc04 --cidr-block 10.0.3.0/24
{
    "Subnet": {
        "AvailabilityZone": "ap-northeast-2a",
        "AvailableIpAddressCount": 251,
        "DefaultForAz": false,
        "Ipv6CidrBlockAssociationSet": [],
        "VpcId": "vpc-0fd5393a547c8dc04",
        "State": "pending",
        "MapPublicIpOnLaunch": false,
        "SubnetId": "subnet-0b300a115d3bc88ea",
        "CidrBlock": "10.0.3.0/24",
        "AssignIpv6AddressOnCreation": false
    }
}
```

```sh
$ aws ec2 create-subnet --availability-zone ap-northeast-2c --vpc-id vpc-0fd5393a547c8dc04 --cidr-block 10.0.4.0/24
{
    "Subnet": {
        "AvailabilityZone": "ap-northeast-2c",
        "AvailableIpAddressCount": 251,
        "DefaultForAz": false,
        "Ipv6CidrBlockAssociationSet": [],
        "VpcId": "vpc-0fd5393a547c8dc04",
        "State": "pending",
        "MapPublicIpOnLaunch": false,
        "SubnetId": "subnet-0dd0cafc08893fe06",
        "CidrBlock": "10.0.4.0/24",
        "AssignIpv6AddressOnCreation": false
    }
}
```

### Internet Gateway 만들기

외부 인터넷과의 창구 역할을 하는 internet gateway를 만들어서 VPC에 붙입니다.

```sh
$ aws ec2 create-internet-gateway
{
    "InternetGateway": {
        "Tags": [],
        "Attachments": [],
        "InternetGatewayId": "igw-0a70a8e3ef5863237"
    }
}
```

```sh
$ aws ec2 attach-internet-gateway --vpc-id vpc-0fd5393a547c8dc04 --internet-gateway-id igw-0a70a8e3ef5863237
```

### NAT Gateway 만들기

private subnet의 data server도 ecr로부터 docker image를 pull 하기 위해서 외부 인터넷에 접근이 가능해야 합니다. public subnet에 NAT gateway를 만들어 private route table에서 이 gateway를 바라보게 만들어야 합니다.

NAT gateway에 부여 할 EIP를 생성합니다.

```sh
$ aws ec2 allocate-address --domain vpc 
{
    "PublicIp": "52.79.205.138",
    "Domain": "vpc",
    "AllocationId": "eipalloc-01e8127a0bcbbf327",
    "PublicIpv4Pool": "amazon"
}
```

public subnet에 NAT gateway를 만듭니다.

```sh
$ aws ec2 create-nat-gateway --allocation-id eipalloc-01e8127a0bcbbf327 --subnet-id subnet-0e8420c6f218aafd6
{
    "NatGateway": {
        "NatGatewayAddresses": [
            {
                "AllocationId": "eipalloc-01e8127a0bcbbf327"
            }
        ],
        "VpcId": "vpc-0fd5393a547c8dc04",
        "State": "pending",
        "NatGatewayId": "nat-0f175ffc9e6a32c5b",
        "SubnetId": "subnet-0e8420c6f218aafd6",
        "CreateTime": "2018-12-30T14:39:03.000Z"
    }
}
```

### Routing Table 만들기

public subnet에서 사용 할 routing table을 만듭니다.

```sh
$ aws ec2 create-route-table --vpc-id vpc-0fd5393a547c8dc04
{
    "RouteTable": {
        "Associations": [],
        "RouteTableId": "rtb-00a887589127c7869",
        "VpcId": "vpc-0fd5393a547c8dc04",
        "PropagatingVgws": [],
        "Tags": [],
        "Routes": [
            {
                "GatewayId": "local",
                "DestinationCidrBlock": "10.0.0.0/16",
                "State": "active",
                "Origin": "CreateRouteTable"
            }
        ]
    }
}
```

들어온 모든 요청을 internet gateway로 보냅니다.

```sh
$ aws ec2 create-route --route-table-id rtb-00a887589127c7869 --destination-cidr-block 0.0.0.0/0 --gateway-id igw-0a70a8e3ef5863237
{
    "Return": true
}
```

private subnet에서 사용 할 routing table을 만듭니다.

```sh
$ aws ec2 create-route-table --vpc-id vpc-0fd5393a547c8dc04
{
    "RouteTable": {
        "Associations": [],
        "RouteTableId": "rtb-0e546146934985a4b",
        "VpcId": "vpc-0fd5393a547c8dc04",
        "PropagatingVgws": [],
        "Tags": [],
        "Routes": [
            {
                "GatewayId": "local",
                "DestinationCidrBlock": "10.0.0.0/16",
                "State": "active",
                "Origin": "CreateRouteTable"
            }
        ]
    }
}
```

private subnet에서 외부 인터넷을 접근하기 위해 NAT gateway를 연결합니다.

```sh
$ aws ec2 create-route --route-table-id rtb-0e546146934985a4b --destination-cidr-block 0.0.0.0/0 --gateway-id nat-0f175ffc9e6a32c5b
{
    "Return": true
}
```

### Subnet을 Routing Table에 연결

최종적으로 작성한 routing table을 subnet에 연결해줘야 합니다. 먼저 public subnet들을 public용 routing table에 연결합니다.

```sh
$ aws ec2 associate-route-table --route-table-id rtb-00a887589127c7869 --subnet-id subnet-0e8420c6f218aafd6
{
    "AssociationId": "rtbassoc-0d441731317405a38"
}
```

```sh
$ aws ec2 associate-route-table --route-table-id rtb-00a887589127c7869 --subnet-id subnet-0a048beef8a45131b
{
    "AssociationId": "rtbassoc-09b427a04a5cfb80e"
}
```

private subnet들도 private용 routing table에 연결합니다.

```sh
$ aws ec2 associate-route-table --route-table-id rtb-0e546146934985a4b --subnet-id subnet-0b300a115d3bc88ea
{
    "AssociationId": "rtbassoc-0ccf38d4186f65699"
}
```

```sh
$ aws ec2 associate-route-table --route-table-id rtb-0e546146934985a4b --subnet-id subnet-0dd0cafc08893fe06
{
    "AssociationId": "rtbassoc-090a726d7a790a059"
}
```

## Load balancer 구성

ECS 서비스의 scaling과 도메인 설정, health 체크 등을 맡을 load balancer를 설치합니다. 사용자로부터 들어오는 외부의 요청을 지정한 target으로 분기시켜줍니다.

### Security Group 만들기

```sh
$ aws ec2 create-security-group --group-name ecs-test-lb --description "security group for ecs-test load balancer" --vpc-id vpc-0fd5393a547c8dc04
{
    "GroupId": "sg-0dba29849e569112a"
}
```

HTTP(TCP 80포트)로의 모든 접근을 허용합니다.

```sh
$ aws ec2 authorize-security-group-ingress --group-id sg-0dba29849e569112a --protocol tcp --port 80 --cidr 0.0.0.0/0
```

### Load Balancer 만들기

load balancer는 외부와의 통신을 맡아 하기 때문에 public subnet에 설치합니다.

```sh
$ aws elbv2 create-load-balancer --name ecs-test-lb --subnets subnet-0e8420c6f218aafd6 subnet-0a048beef8a45131b --security-groups sg-0dba29849e569112a
{
    "LoadBalancers": [
        {
            "IpAddressType": "ipv4",
            "VpcId": "vpc-0fd5393a547c8dc04",
            "LoadBalancerArn": "arn:aws:elasticloadbalancing:ap-northeast-2:759375304948:loadbalancer/app/ecs-test-lb/39b65ee79e10c657",
            "State": {
                "Code": "provisioning"
            },
            "DNSName": "ecs-test-lb-1055604155.ap-northeast-2.elb.amazonaws.com",
            "SecurityGroups": [
                "sg-0dba29849e569112a"
            ],
            "LoadBalancerName": "ecs-test-lb",
            "CreatedTime": "2018-12-30T13:13:37.280Z",
            "Scheme": "internet-facing",
            "Type": "application",
            "CanonicalHostedZoneId": "ZWKZPGTI48KDX",
            "AvailabilityZones": [
                {
                    "SubnetId": "subnet-0a048beef8a45131b",
                    "ZoneName": "ap-northeast-2c"
                },
                {
                    "SubnetId": "subnet-0e8420c6f218aafd6",
                    "ZoneName": "ap-northeast-2a"
                }
            ]
        }
    ]
}
```

### Target Group 등록

load balancer는 규칙에 따라 각각 다른 target group으로 redirect 시킬 수 있습니다. 이 target group은 load balancer의 80포트로 접근하는 요청들을 처리합니다. `aws elbv2 register-targets` 명령으로 개별 EC2 인스턴스를 target group의 target으로 등록해줄 수 있지만, 우리는 ECS 서비스를 등록 할 때 자동으로 연결시켜 줄 것이므로 생략합니다. 기존에 EC2 인스턴스를 target group으로 등록 할 때에는 `--target-type` 옵션에 해당 instance를 설정했지만, Fargate는 awsvpc 네트워크 모드 위에서 각각의 task들을 Elastic network interface(ENI)로 바라보기 때문에 ip로 설정해주어야 합니다.

```sh
$ aws elbv2 create-target-group --name ecs-test-target-group --target-type ip --protocol HTTP --port 80 --vpc-id vpc-0fd5393a547c8dc04
{
    "TargetGroups": [
        {
            "HealthCheckPath": "/",
            "HealthCheckIntervalSeconds": 30,
            "VpcId": "vpc-0fd5393a547c8dc04",
            "Protocol": "HTTP",
            "HealthCheckTimeoutSeconds": 5,
            "TargetType": "ip",
            "HealthCheckProtocol": "HTTP",
            "UnhealthyThresholdCount": 2,
            "HealthyThresholdCount": 5,
            "TargetGroupArn": "arn:aws:elasticloadbalancing:ap-northeast-2:759375304948:targetgroup/ecs-test-target-group/c5cad998408418e8",
            "Matcher": {
                "HttpCode": "200"
            },
            "HealthCheckPort": "traffic-port",
            "Port": 80,
            "TargetGroupName": "ecs-test-target-group"
        }
    ]
}
```

### Listener 생성

Load balancer에서 요청을 수신하고 수행 할 액션을 정의합니다. 우리는 HTTP 요청을 받아 target group으로 요청을 forward 하는 listener를 생성합니다. HTTPS로 보안 처리를 하고 싶다면 listener를 만들 때, `certificates` 옵션으로 인증서를 전달합니다.

```sh
$ aws elbv2 create-listener --load-balancer-arn "arn:aws:elasticloadbalancing:ap-northeast-2:759375304948:loadbalancer/app/ecs-test-lb/39b65ee79e10c657" --protocol HTTP --port 80 --default-actions Type=forward,TargetGroupArn="arn:aws:elasticloadbalancing:ap-northeast-2:759375304948:targetgroup/ecs-test-target-group/c5cad998408418e8"
{
    "Listeners": [
        {
            "Protocol": "HTTP",
            "DefaultActions": [
                {
                    "TargetGroupArn": "arn:aws:elasticloadbalancing:ap-northeast-2:759375304948:targetgroup/ecs-test-target-group/c5cad998408418e8",
                    "Type": "forward"
                }
            ],
            "LoadBalancerArn": "arn:aws:elasticloadbalancing:ap-northeast-2:759375304948:loadbalancer/app/ecs-test-lb/39b65ee79e10c657",
            "Port": 80,
            "ListenerArn": "arn:aws:elasticloadbalancing:ap-northeast-2:759375304948:listener/app/ecs-test-lb/39b65ee79e10c657/70d45a932ed36d04"
        }
    ]
}
```

## Service Discovery 구성

컨테이너 기반의 마이크로서비스들은 물리적, 논리적으로 분산되어 있기 때문에 각 서비스가 서로를 인식하고 요청을 주고 받는데 어려운 점이 많았습니다. 이런 점을 해소하기 위해 Docker의 swarm, Google의 kubernetes 등이 이용되고 있습니다. AWS는 이를 해결하는 방안으로 ECS 서비스 앞에 load balancer를 설치하는 것이 최선이었습니다. 하지만 이제 AWS Cloud Map 기술 기반의 service discovery 서비스를 출시하면서 이에 대한 지원을 강화했습니다.

service discovery는 Elastic Network Interface(ENI)라는 AWS에서 제공하는 새로운 스택을 이용하며, Route 53에서 private domain으로 ENI를 장착한 리소스들을 바라 볼 수 있게 합니다. 이로 인해 개념적으로 종류에 국한되지 않는 여러 다양한 AWS resource들을 discover 할 수 있습니다.

### Namespace 생성

private dns를 이용하는 `ecs-test`라는 namespace를 생성합니다.

```sh
$ aws servicediscovery create-private-dns-namespace --name ecs-test --vpc vpc-0fd5393a547c8dc04
{
    "OperationId": "chxxhm7juhbv6sqta3dvfb3wwbx5xy2u-jqb5ae5z"
}
```

정상적으로 설치가 되었는지 확인 합니다. `NAMESPACE` 항목은 이후에 servicediscovery service를 만들 때, 사용합니다.

```sh
$ aws servicediscovery get-operation --operation-id chxxhm7juhbv6sqta3dvfb3wwbx5xy2u-jqax5ax5
{
    "Operation": {
        "Status": "PENDING",
        "CreateDate": 1546189469.495,
        "Id": "chxxhm7juhbv6sqta3dvfb3wwbx5xy2u-jqb5ae5z",
        "UpdateDate": 1546189470.02,
        "Type": "CREATE_NAMESPACE",
        "Targets": {
            "NAMESPACE": "ns-um5fgwsfj5lrvj4n"
        }
    }
}
```

### Service Discovery의 Service 생성

gate server는 `gate-server`라는 이름으로 service discovery service를 만듭니다. 이 작업은 앞 섹션에서 만든 `ecs-test`라는 namespace안에서 `gate-server.ecs-test`라는 이름으로 접근이 가능하게 해줍니다. 여기서의 service는 ECS의 것과는 별개라는 점을 유의해야 합니다.

`gate-server`는 Route 53에서 A 레코드로 생성됩니다.

gate server 서비스 생성

```sh
$ aws servicediscovery create-service --name gate-server --dns-config 'NamespaceId="ns-um5fgwsfj5lrvj4n",DnsRecords=[{Type="A",TTL="300"}]' --health-check-custom-config FailureThreshold=1
{
    "Service": {
        "Name": "gate-server",
        "DnsConfig": {
            "DnsRecords": [
                {
                    "Type": "A",
                    "TTL": 300
                }
            ],
            "NamespaceId": "ns-um5fgwsfj5lrvj4n",
            "RoutingPolicy": "MULTIVALUE"
        },
        "CreateDate": 1546189642.633,
        "CreatorRequestId": "7e1b0e59-ad33-453c-90a0-0a65a78d001d",
        "HealthCheckCustomConfig": {
            "FailureThreshold": 1
        },
        "Id": "srv-a7pnztd427fm653b",
        "Arn": "arn:aws:servicediscovery:ap-northeast-2:759375304948:service/srv-a7pnztd427fm653b"
    }
}
```

data server 서비스 생성

```sh
$ aws servicediscovery create-service --name data-server --dns-config 'NamespaceId="ns-um5fgwsfj5lrvj4n",DnsRecords=[{Type="A",TTL="300"}]' --health-check-custom-config FailureThreshold=1
{
    "Service": {
        "Name": "data-server",
        "DnsConfig": {
            "DnsRecords": [
                {
                    "Type": "A",
                    "TTL": 300
                }
            ],
            "NamespaceId": "ns-um5fgwsfj5lrvj4n",
            "RoutingPolicy": "MULTIVALUE"
        },
        "CreateDate": 1546189707.196,
        "CreatorRequestId": "0f0c6b0d-a9dc-48de-bc42-3592493d92d3",
        "HealthCheckCustomConfig": {
            "FailureThreshold": 1
        },
        "Id": "srv-6qx4f3yb7puqdcp3",
        "Arn": "arn:aws:servicediscovery:ap-northeast-2:759375304948:service/srv-6qx4f3yb7puqdcp3"
    }
}
```

### Private DNS로 주소 변경

이제 하나의 namespace 안에서 resource들은 고유의 private dns name을 갖게 됩니다. data server의 URL을 변경합니다.

gate-server/index.js
```javascript
...

app.get('/', (req, res) => {
  request('http://data-server.ecs-test:4000/', (err, response, body) => {
    console.log('데이터 서버로부터 응답을 받았습니다:')
    console.log(body)
    res.send(body)
  })
});

...
```

빌드하고 ECR로 push 합니다.

```sh
$ docker build -t 759375304948.dkr.ecr.ap-northeast-2.amazonaws.com/gate-server .

$ docker push 759375304948.dkr.ecr.ap-northeast-2.amazonaws.com/gate-server
```

## ECS 구성

긴 시간동안 고생하셨습니다. 드디어 우리가 원하는 ECS를 구성 할 수 있게 되었습니다.

### Cluster 생성

서비스들이 포함 될 ECS에서 가장 큰 단위인 cluster를 생성합니다.

```sh
$ aws ecs create-cluster --cluster-name ecs-test
{
    "cluster": {
        "status": "ACTIVE",
        "statistics": [],
        "tags": [],
        "clusterName": "ecs-test",
        "registeredContainerInstancesCount": 0,
        "pendingTasksCount": 0,
        "runningTasksCount": 0,
        "activeServicesCount": 0,
        "clusterArn": "arn:aws:ecs:ap-northeast-2:759375304948:cluster/ecs-test"
    }
}
```

### Log Group 생성

CloudWatch Logs로 로그 정보를 보내기 위해 log group을 미리 생성해두어야 합니다. gate server와 date server용으로 각각 만듭니다.

```sh
$ aws logs create-log-group --log-group-name awslogs-gate-server
```

```sh
$ aws logs create-log-group --log-group-name awslogs-data-server
```

### Task Definition 작성 및 등록

service내에서 돌아갈 task에 대한 정의를 담은 task definition 파일을 작성해야 합니다. 이 파일에는 사용 할 네트워크 모드, 컨테이너의 설정, CPU/memory 등의 하드웨어 구성 등을 설정합니다.

각 서버의 task definition 정의를 fargate-task.json 파일로 작성합니다.

gate-server/fargate-task.json

```json
{
    "family": "ecs-test-gate-server", 
    "networkMode": "awsvpc", 
    "executionRoleArn": "arn:aws:iam::759375304948:role/ecsTaskExecutionRole",
    "containerDefinitions": [
        {
            "name": "gate-server", 
            "image": "759375304948.dkr.ecr.ap-northeast-2.amazonaws.com/gate-server:latest", 
            "portMappings": [
                {
                    "containerPort": 80, 
                    "hostPort": 80, 
                    "protocol": "tcp"
                }
            ], 
            "logConfiguration": {
                "logDriver": "awslogs",
                "options": {
                    "awslogs-group": "awslogs-gate-server",
                    "awslogs-region": "ap-northeast-2",
                    "awslogs-stream-prefix": "awslogs-ecs-test"
                }
            },
            "essential": true, 
            "entryPoint": [
                "node",
                "index.js"
            ]
        }
    ], 
    "requiresCompatibilities": [
        "FARGATE"
    ], 
    "cpu": "256", 
    "memory": "512"
}
```

```sh
$ aws ecs register-task-definition --cli-input-json file://fargate-task.json
{
    "taskDefinition": {
        "status": "ACTIVE",
        "networkMode": "awsvpc",
        "family": "ecs-test-gate-server",
        "placementConstraints": [],
        "requiresAttributes": [
            {
                "name": "ecs.capability.execution-role-ecr-pull"
            },
            {
                "name": "com.amazonaws.ecs.capability.docker-remote-api.1.18"
            },
            {
                "name": "ecs.capability.task-eni"
            },
            {
                "name": "com.amazonaws.ecs.capability.ecr-auth"
            }
        ],
        "cpu": "256",
        "executionRoleArn": "arn:aws:iam::759375304948:role/ecsTaskExecutionRole",
        "compatibilities": [
            "EC2",
            "FARGATE"
        ],
        "volumes": [],
        "memory": "512",
        "requiresCompatibilities": [
            "FARGATE"
        ],
        "taskDefinitionArn": "arn:aws:ecs:ap-northeast-2:759375304948:task-definition/ecs-test-gate-server:3",
        "containerDefinitions": [
            {
                "environment": [],
                "name": "gate-server",
                "mountPoints": [],
                "image": "759375304948.dkr.ecr.ap-northeast-2.amazonaws.com/gate-server:latest",
                "cpu": 0,
                "portMappings": [
                    {
                        "protocol": "tcp",
                        "containerPort": 80,
                        "hostPort": 80
                    }
                ],
                "entryPoint": [
                    "node",
                    "index.js"
                ],
                "essential": true,
                "volumesFrom": []
            }
        ],
        "revision": 3
    }
}
```

data-server/fargate-task.json

```json
{
    "family": "ecs-test-data-server", 
    "networkMode": "awsvpc", 
    "executionRoleArn": "arn:aws:iam::759375304948:role/ecsTaskExecutionRole",
    "containerDefinitions": [
        {
            "name": "data-server", 
            "image": "759375304948.dkr.ecr.ap-northeast-2.amazonaws.com/data-server:latest", 
            "portMappings": [
                {
                    "containerPort": 4000, 
                    "hostPort": 4000, 
                    "protocol": "tcp"
                }
            ], 
            "logConfiguration": {
                "logDriver": "awslogs",
                "options": {
                    "awslogs-group": "awslogs-data-server",
                    "awslogs-region": "ap-northeast-2",
                    "awslogs-stream-prefix": "awslogs-ecs-test"
                }
            },
            "essential": true, 
            "entryPoint": [
                "node",
                "index.js"
            ]
        }
    ], 
    "requiresCompatibilities": [
        "FARGATE"
    ], 
    "cpu": "256", 
    "memory": "512"
}
```

```sh
$ aws ecs register-task-definition --cli-input-json file://fargate-task.json
{
    "taskDefinition": {
        "status": "ACTIVE",
        "networkMode": "awsvpc",
        "family": "ecs-test-data-server",
        "placementConstraints": [],
        "requiresAttributes": [
            {
                "name": "ecs.capability.execution-role-ecr-pull"
            },
            {
                "name": "com.amazonaws.ecs.capability.docker-remote-api.1.18"
            },
            {
                "name": "ecs.capability.task-eni"
            },
            {
                "name": "com.amazonaws.ecs.capability.ecr-auth"
            }
        ],
        "cpu": "256",
        "executionRoleArn": "arn:aws:iam::759375304948:role/ecsTaskExecutionRole",
        "compatibilities": [
            "EC2",
            "FARGATE"
        ],
        "volumes": [],
        "memory": "512",
        "requiresCompatibilities": [
            "FARGATE"
        ],
        "taskDefinitionArn": "arn:aws:ecs:ap-northeast-2:759375304948:task-definition/ecs-test-data-server:2",
        "containerDefinitions": [
            {
                "environment": [],
                "name": "data-server",
                "mountPoints": [],
                "image": "759375304948.dkr.ecr.ap-northeast-2.amazonaws.com/data-server:latest",
                "cpu": 0,
                "portMappings": [
  1 {
                    {
                        "protocol": "tcp",
                        "containerPort": 4000,
                        "hostPort": 4000
                    }
                ],
                "entryPoint": [
                    "node",
                    "index.js"
                ],
                "essential": true,
                "volumesFrom": []
            }
        ],
        "revision": 2
    }
}
```

### Security Group 만들기

Gate server와 data server에 각각의 security group을 만들어줍니다.

```sh
$ aws ec2 create-security-group --group-name ecs-test-gate-server --description "Security group for ecs test" --vpc-id vpc-0fd5393a547c8dc04
{
    "GroupId": "sg-0fb62db1fad8b1e54"
}
```

외부에서 들어오는 요청은 제일 먼저 load balancer가 받고, 이를 gate server로 던져주게 됩니다. 따라서, gate server는 HTTP(TCP 80포트)로의 내부 접근만 허용합니다.

```sh
$ aws ec2 authorize-security-group-ingress --group-id sg-0fb62db1fad8b1e54 --protocol tcp --port 80 --cidr 10.0.0.0/16
```

```sh
$ aws ec2 create-security-group --group-name ecs-test-data-server --description "Security group for ecs test" --vpc-id vpc-0fd5393a547c8dc04
{
    "GroupId": "sg-05f835f36c752cd2f"
}
```

Data server는 TCP 4000 포트로의 내부 접근만 허용합니다.

```sh
$ aws ec2 authorize-security-group-ingress --group-id sg-05f835f36c752cd2f --protocol tcp --port 4000 --cidr 10.0.0.0/16
```

### Service 생성

task definition이 정의되었으면 이제 이 task를 가지고 서비스를 구동 할 service만 만듭니다. service 역시 task와 마찬가지로 전달되어야 할 파라메터들을 json 파일로 만들어 설정하겠습니다. 이전에 만들어둔 load balancer와 service discovery의 arn을 설정하고 gate server는 public subnet에서 생성되도록 설정합니다. 

각 서비스의 설정 파일을 ecs-service-discovery.json 파일로 만들어 저장합니다.

gate-server/ecs-service-discovery.json

```json
{
    "cluster": "ecs-test",
    "serviceName": "ecs-service-gate-server",
    "taskDefinition": "ecs-test-gate-server",
	"loadBalancers": [
	  {
		"targetGroupArn": "arn:aws:elasticloadbalancing:ap-northeast-2:759375304948:targetgroup/ecs-test-target-group/c5cad998408418e8",
		"containerName": "gate-server",
		"containerPort": 80
	  }
	],
    "serviceRegistries": [
       {
          "registryArn": "arn:aws:servicediscovery:ap-northeast-2:759375304948:service/srv-a7pnztd427fm653b"
       }
    ],
    "launchType": "FARGATE",
    "platformVersion": "1.1.0",
    "networkConfiguration": {
       "awsvpcConfiguration": {
          "assignPublicIp": "ENABLED",
          "securityGroups": [ "sg-0fb62db1fad8b1e54" ],
          "subnets": [ "subnet-0e8420c6f218aafd6", "subnet-0a048beef8a45131b" ]
       }
    },
    "desiredCount": 1
}
```

```sh
$ aws ecs create-service --cli-input-json file://ecs-service-discovery.json
{
    "service": {
        "networkConfiguration": {
            "awsvpcConfiguration": {
                "subnets": [
                    "subnet-0e8420c6f218aafd6",
                    "subnet-0a048beef8a45131b"
                ],
                "securityGroups": [
                    "sg-0fb62db1fad8b1e54"
                ],
                "assignPublicIp": "ENABLED"
            }
        },
        "launchType": "FARGATE",
        "enableECSManagedTags": false,
        "loadBalancers": [
            {
                "containerName": "gate-server",
                "targetGroupArn": "arn:aws:elasticloadbalancing:ap-northeast-2:759375304948:targetgroup/ecs-test-target-group/c5cad998408418e8",
                "containerPort": 80
            }
        ],
        "desiredCount": 1,
        "clusterArn": "arn:aws:ecs:ap-northeast-2:759375304948:cluster/ecs-test",
        "serviceArn": "arn:aws:ecs:ap-northeast-2:759375304948:service/ecs-service-gate-server",
        "deploymentConfiguration": {
            "maximumPercent": 200,
            "minimumHealthyPercent": 100
        },
        "createdAt": 1546177347.201,
        "healthCheckGracePeriodSeconds": 0,
        "schedulingStrategy": "REPLICA",
        "placementConstraints": [],
        "deployments": [
            {
                "status": "PRIMARY",
                "networkConfiguration": {
                    "awsvpcConfiguration": {
                        "subnets": [
                            "subnet-0e8420c6f218aafd6",
                            "subnet-0a048beef8a45131b"
                        ],
                        "securityGroups": [
                            "sg-0fb62db1fad8b1e54"
                        ],
                        "assignPublicIp": "ENABLED"
                    }
                },
                "pendingCount": 0,
                "launchType": "FARGATE",
                "createdAt": 1546177347.201,
                "desiredCount": 1,
                "taskDefinition": "arn:aws:ecs:ap-northeast-2:759375304948:task-definition/ecs-test-gate-server:3",
                "updatedAt": 1546177347.201,
                "platformVersion": "1.1.0",
                "id": "ecs-svc/9223370490677428606",
                "runningCount": 0
            }
        ],
        "serviceName": "ecs-service-gate-server",
        "events": [],
        "runningCount": 0,
        "status": "ACTIVE",
        "serviceRegistries": [
            {
                "registryArn": "arn:aws:servicediscovery:ap-northeast-2:759375304948:service/srv-a7pnztd427fm653b"
            }
        ],
        "pendingCount": 0,
        "platformVersion": "1.1.0",
        "placementStrategy": [],
        "propagateTags": "NONE",
        "roleArn": "arn:aws:iam::759375304948:role/aws-service-role/ecs.amazonaws.com/AWSServiceRoleForECS",
        "taskDefinition": "arn:aws:ecs:ap-northeast-2:759375304948:task-definition/ecs-test-gate-server:3"
    }
}
```

data-server는 private subnet에서 만들어지도록 설정합니다.

date-server/ecs-service-discovery.json

```json
{
    "cluster": "ecs-test",
    "serviceName": "ecs-service-data-server",
    "taskDefinition": "ecs-test-data-server",
    "serviceRegistries": [
       {
          "registryArn": "arn:aws:servicediscovery:ap-northeast-2:759375304948:service/srv-6qx4f3yb7puqdcp3"
       }
    ],
    "launchType": "FARGATE",
    "platformVersion": "1.1.0",
    "networkConfiguration": {
       "awsvpcConfiguration": {
          "assignPublicIp": "ENABLED",
          "securityGroups": [ "sg-05f835f36c752cd2f" ],
          "subnets": [ "subnet-0b300a115d3bc88ea", "subnet-0dd0cafc08893fe06" ]
       }
    },
    "desiredCount": 1
}
```

```sh
$ aws ecs create-service --cli-input-json file://ecs-service-discovery.json
{
    "service": {
        "networkConfiguration": {
            "awsvpcConfiguration": {
                "subnets": [
                    "subnet-0b300a115d3bc88ea",
                    "subnet-0dd0cafc08893fe06"
                ],
                "securityGroups": [
                    "sg-05f835f36c752cd2f"
                ],
                "assignPublicIp": "ENABLED"
            }
        },
        "launchType": "FARGATE",
        "enableECSManagedTags": false,
        "loadBalancers": [],
        "desiredCount": 1,
        "clusterArn": "arn:aws:ecs:ap-northeast-2:759375304948:cluster/ecs-test",
        "serviceArn": "arn:aws:ecs:ap-northeast-2:759375304948:service/ecs-service-data-server",
        "deploymentConfiguration": {
            "maximumPercent": 200,
            "minimumHealthyPercent": 100
        },
        "createdAt": 1546177452.469,
        "schedulingStrategy": "REPLICA",
        "placementConstraints": [],
        "deployments": [
            {
                "status": "PRIMARY",
                "networkConfiguration": {
                    "awsvpcConfiguration": {
                        "subnets": [
                            "subnet-0b300a115d3bc88ea",
                            "subnet-0dd0cafc08893fe06"
                        ],
                        "securityGroups": [
                            "sg-05f835f36c752cd2f"
                        ],
                        "assignPublicIp": "ENABLED"
                    }
                },
                "pendingCount": 0,
                "launchType": "FARGATE",
                "createdAt": 1546177452.469,
                "desiredCount": 1,
                "taskDefinition": "arn:aws:ecs:ap-northeast-2:759375304948:task-definition/ecs-test-data-server:2",
                "updatedAt": 1546177452.469,
                "platformVersion": "1.1.0",
                "id": "ecs-svc/9223370490677323338",
                "runningCount": 0
            }
        ],
        "serviceName": "ecs-service-data-server",
        "events": [],
        "runningCount": 0,
        "status": "ACTIVE",
        "serviceRegistries": [
            {
                "registryArn": "arn:aws:servicediscovery:ap-northeast-2:759375304948:service/srv-6qx4f3yb7puqdcp3"
            }
        ],
        "pendingCount": 0,
        "platformVersion": "1.1.0",
        "placementStrategy": [],
        "propagateTags": "NONE",
        "roleArn": "arn:aws:iam::759375304948:role/aws-service-role/ecs.amazonaws.com/AWSServiceRoleForECS",
        "taskDefinition": "arn:aws:ecs:ap-northeast-2:759375304948:task-definition/ecs-test-data-server:2"
    }
}
```

## Auto Scaling Group 설정

### Service를 Scale 가능한 타겟으로 등록

네트워크 트래픽이나 CPU 사용량이 늘어 새로운 서비스 인스턴스를 띄워야 하는 경우가 발생 할 수 있으므로 자동으로 scale in/out 시킬 수 있도록 auto scaling 기능을 활성화시킵니다. 이 설정에서는 최소 1, 최대 4개의 task가 만들어지도록 했습니다.

```sh
$ aws application-autoscaling register-scalable-target --resource-id service/ecs-test/ecs-service-gate-server --service-namespace ecs --scalable-dimension ecs:service:DesiredCount --min-capacity 1 --max-capacity 4 --role-arn arn:aws:iam::759375304948:role/ApplicationAutoscalingECSRole
```

### Scaling Policy 연결

ECS의 scaling 정책은 `StepScaling`, `TargetTrackingScaling`등으로 구분되는데, 여기서는 `TargetTrackingScaling`으로 설정합니다. 이 정책은 scale을 위한 기준을 정하고 해당 기준을 넘으면 scale out, 그 기준보다 낮으면 scale in을 하여  설정한 기준을 유지하는 방식입니다. 말하자면, 보일러의 온도 설정과 비슷합니다. 온도보다 낮으면 기름을 더 쓰고, 온도보다 높으면 기름을 덜 써서 설정한 온도를 유지합니다.

반면 StepScaling은 scale out을 위한 측정치와 scale in을 위한 측정치를 따로 설정 할 수 있습니다. 프로젝트의 요구 사항에 맞도록 알맞게 선택을 합니다.

target tracking config 파일을 작성합니다.

scale-config.json

```json
{
  "TargetValue": 40.0,
  "PredefinedMetricSpecification": 
    {
      "PredefinedMetricType": "ECSServiceAverageCPUUtilization"
    }
}
```

서비스의 CPU utilization 값을 40%로 유지하도록 하는 설정을 적용합니다.

```sh
$ aws application-autoscaling put-scaling-policy --policy-name ecs-test-scale-policy --service-namespace ecs --resource-id service/ecs-test/ecs-service-gate-server --scalable-dimension ecs:service:DesiredCount --policy-type TargetTrackingScaling --target-tracking-scaling-policy-configuration file://scale-config.json
{
    "Alarms": [
        {
            "AlarmName": "TargetTracking-service/ecs-test/ecs-service-gate-server-AlarmHigh-4dfa94cf-738d-4b7f-88ae-b1a5d94451ea",
            "AlarmARN": "arn:aws:cloudwatch:ap-northeast-2:759375304948:alarm:TargetTracking-service/ecs-test/ecs-service-gate-server-AlarmHigh-4dfa94cf-738d-4b7f-88ae-b1a5d94451ea"
        },
        {
            "AlarmName": "TargetTracking-service/ecs-test/ecs-service-gate-server-AlarmLow-acba104c-4d8f-4e82-8637-d27177a14d43",
            "AlarmARN": "arn:aws:cloudwatch:ap-northeast-2:759375304948:alarm:TargetTracking-service/ecs-test/ecs-service-gate-server-AlarmLow-acba104c-4d8f-4e82-8637-d27177a14d43"
        }
    ],
    "PolicyARN": "arn:aws:autoscaling:ap-northeast-2:759375304948:scalingPolicy:ae459d64-938b-4edc-9389-1e4b86ed5708:resource/ecs/service/ecs-test/ecs-service-gate-server:policyName/ecs-test-scale-policy"
}
```

## 최종 테스트

### 동작 테스트

load balancer의 public dns로 요청을 보내봅니다.

```sh
$ curl ecs-test-lb-1055604155.ap-northeast-2.elb.amazonaws.com
응, 답!
```

### Scale 테스트

ApacheBench를 이용해 서비스의 사용량을 늘려 설정해둔 scale 기능이 동작하는지 확인합니다.

```sh
$ ab -n 100000 -c 1000 http://ecs-test-lb-1055604155.ap-northeast-2.elb.amazonaws.com/
```

설정해둔 CPU %를 넘었을 때 scale 작업이 trigger 되는지 확인합니다.

## ECS의 유지보수

### Task Definition 업데이트

운영 중에 task definition이 변경되는 경우가 있습니다. cpu나 memory의 사양을 높이거나 listening 포트가 바뀌는 등의 변화를 반영하려면 다음 명령을 입력합니다. 최초로 task definition을 등록할 때와 동일한 명령입니다. 이 명령이 전달되면 task definition의 revision이 1이 올라가고, service의 task는 재배포됩니다.

`aws ecs register-task-definition` 결과값의 최신 task definition의 revision을 `aws ecs update-service` 명령의 옵션 --task-definition ecs-test-gate-server:`4` 에 명시합니다.

```sh
$ aws ecs register-task-definition --cli-input-json file://fargate-task.json

$ aws ecs update-service --cluster ecs-test --service ecs-service-gate-server --task-definition ecs-test-gate-server:4
```

### Service 강제 배포

애플리케이션이 새로 빌드되면 ECR에 이미지를 push하고, 태스크를 재배포해야 합니다. revision 업데이트 없이 service가 이미지를 새로 가져와서 업데이트하게 하려면 `aws ecs update-service --force-new-deployment` 옵션을 이용합니다.

```sh
$ docker build -t 759375304948.dkr.ecr.ap-northeast-2.amazonaws.com/gate-server .

$ docker push 759375304948.dkr.ecr.ap-northeast-2.amazonaws.com/gate-server

$ aws ecs update-service --cluster ecs-test --service ecs-service-gate-server --force-new-deployment
```

### Task Scaling

강제로 태스크를 scaling 시키고자 할 때는 `aws ecs update-service --desired-count` 옵션을 이용합니다.

```sh
aws ecs update-service --cluster ecs-test --service ecs-service-gate-server --desired-count 0
```

## 리소스 초기화

administrator 사용자로 전환하여 지금까지 만들었던 리소스들을 제거합니다.

```sh
$ aws application-autoscaling deregister-scalable-target --service-namespace ecs --resource-id service/ecs-test/ecs-service-gate-server --scalable-dimension ecs:service:DesiredCount
$ aws ecs update-service --cluster ecs-test --service ecs-service-gate-server --desired-count 0 --force-new-deployment
$ aws ecs update-service --cluster ecs-test --service ecs-service-data-server --desired-count 0 --force-new-deployment
$ aws ecs delete-service --cluster ecs-test --service ecs-service-gate-server
$ aws ecs delete-service --cluster ecs-test --service ecs-service-data-server
$ aws ecs deregister-task-definition --task-definition ecs-test-data-server:2
$ aws ecs deregister-task-definition --task-definition ecs-test-data-server:1
$ aws ecs deregister-task-definition --task-definition ecs-test-gate-server:2
$ aws ecs deregister-task-definition --task-definition ecs-test-gate-server:1
$ aws ecs delete-cluster --cluster ecs-test
$ aws ec2 delete-security-group --group-id sg-05f835f36c752cd2f
$ aws ec2 delete-security-group --group-id sg-0fb62db1fad8b1e54
$ aws servicediscovery delete-service --id srv-6qx4f3yb7puqdcp3
$ aws servicediscovery delete-service --id srv-a7pnztd427fm653b
$ aws servicediscovery delete-namespace --id ns-um5fgwsfj5lrvj4n
$ aws elbv2 delete-load-balancer --load-balancer-arn arn:aws:elasticloadbalancing:ap-northeast-2:759375304948:loadbalancer/app/ecs-test-lb/39b65ee79e10c657
$ aws elbv2 delete-target-group --target-group-arn arn:aws:elasticloadbalancing:ap-northeast-2:759375304948:targetgroup/ecs-test-target-group/c5cad998408418e8
$ aws ec2 delete-security-group --group-id sg-0dba29849e569112a
$ aws ec2 delete-nat-gateway --nat-gateway-id nat-0f175ffc9e6a32c5b
$ aws ec2 delete-subnet --subnet-id subnet-0a048beef8a45131b
$ aws ec2 delete-subnet --subnet-id subnet-0b300a115d3bc88ea
$ aws ec2 delete-subnet --subnet-id subnet-0dd0cafc08893fe06
$ aws ec2 delete-subnet --subnet-id subnet-0e8420c6f218aafd6
$ aws ec2 delete-route-table --route-table-id rtb-0e546146934985a4b
$ aws ec2 delete-route-table --route-table-id rtb-00a887589127c7869
$ aws ec2 detach-internet-gateway --internet-gateway-id igw-0a70a8e3ef5863237 --vpc-id vpc-0fd5393a547c8dc04
$ aws ec2 delete-internet-gateway --internet-gateway-id igw-0a70a8e3ef5863237
$ aws ec2 delete-vpc --vpc-id vpc-0fd5393a547c8dc0o4
$ aws ec2 release-address --allocation-id eipalloc-01e8127a0bcbbf327
$ aws ecr delete-repository --repository-name data-server --force
$ aws ecr delete-repository --repository-name gate-server --force
$ aws logs delete-log-group --log-group-name awslogs-data-server
$ aws logs delete-log-group --log-group-name awslogs-gate-server
```