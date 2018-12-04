---
layout: post
title: "Amazon CLI로 ECS에 서버 띄우기 5 - Elastic Container Registry(ECR)에 Docker 이미지 등록하기"
date: 2018-12-04 08:04:00
author: Reid
categories:
  - engineering
tags:
  - amazon
  - aws
  - ecs
  - docker
  - amazon cli
  - ecr
  - elastic container registry
published: false
---
# Elastic Container Registry(ECR)에 이미지 등록하기 

## ECR에 대한 Docker 인증

Docker CLI로 내리는 명령이 ECR로 이미지를 등록하고 가져오는 처리가 가능하도록 해줘야 합니다. 이를 위해 docker에 ECR을 인증해줘야 하는데, Amazon에서는 이를 간단하게 해주는 get-login 명령을 제공합니다.

```bash
$ aws ecr get-login --no-include-email

docker login -u AWS -p 비밀번호 https://본인의_AWS_아이디.dkr.ecr.ap-northeast-2.amazonaws.com
```

결과로 나온 `docker login...` 영역을 통째로 복사해서 터미널에 붙여넣고 실행합니다. 이 로그인은 12시간동안 유효하고, 그 이후에는 다시 로그인 과정을 거쳐야 합니다. awslabs에서 제공하는 [Amazon ECR Crendential Helper](https://github.com/awslabs/amazon-ecr-credential-helper)를 이용하면 그런 수고로움 없이 자동으로 로그인 과정을 진행합니다.

## Repository 만들기

[이전 챕터](2018-12-03-build-test-servers.md)에서 제작한 docker 이미지들을 올릴 repository를 만들어야 합니다.

```bash
$ aws ecr create-repository --repository-name ecs-test-server
{
    "repository": {
        "registryId": "본인의_AWS_아이디",
        "repositoryName": "ecs-test-server",
        "repositoryArn": "arn:aws:ecr:ap-northeast-2:본인의_AWS_아이디:repository/ecs-test-server",
        "createdAt": 1543838155.0,
        "repositoryUri": "본인의_AWS_아이디.dkr.ecr.ap-northeast-2.amazonaws.com/ecs-test-server"
    }
}
```

```bash
$ aws ecr create-repository --repository-name ecs-test-data-store
{
    "repository": {
        "registryId": "본인의_AWS_아이디",
        "repositoryName": "ecs-test-data-store",
        "repositoryArn": "arn:aws:ecr:ap-northeast-2:본인의_AWS_아이디:repository/ecs-test-data-store",
        "createdAt": 1543838248.0,
        "repositoryUri": "본인의_AWS_아이디.dkr.ecr.ap-northeast-2.amazonaws.com/ecs-test-data-store"
    }
}
```

## 이미지 등록하기

다시 한번 우리가 만들어둔 이미지들을 확인해봅시다:

```bash
$ docker images
REPOSITORY            TAG                 IMAGE ID            CREATED             SIZE
ecs-test-server       latest              6a8a2e2eb5d0        3 hours ago         896MB
ecs-test-data-store   latest              86b07d85d96f        5 hours ago         891MB
node                  8                   06f7c071f445        5 days ago          889MB
```

이 중 `ecs-test-server`와 `ecs-test-data-store`를 올려야 합니다.

먼저 각 이미지에 target 설정을 합니다. ecs-test-server를 ECR로 올려야 하기 때문에 올라갈 ECR repository 주소를 명시해야 하는데 이는 `docker tag` 명령을 이용해서 가능합니다. 주소를 명시하지 않으면 기본으로 docker.io의 레포지토리로 설정됩니다. 이미지에 대한 tag는 따로 설정하지 않았기 때문에 latest 기본 태그를 이용하게 됩니다.

```
$docker tag ecs-test-server 본인의_AWS_아이디.dkr.ecr.ap-northeast-2.amazonaws.com/ecs-test-server

$docker tag ecs-test-data-store 본인의_AWS_아이디.dkr.ecr.ap-northeast-2.amazonaws.com/ecs-test-data-store
```

이제 ECR로 올릴 준비를 마쳤습니다. 현재의 이미지 목록을 다시 한번 살펴보면 동일한 이미지가 local용과 ECR용으로 구분 된 것을 볼 수 있습니다.

```bash
$ docker images
REPOSITORY                                                              TAG                 IMAGE ID            CREATED             SIZE
본인의_AWS_아이디.dkr.ecr.ap-northeast-2.amazonaws.com/ecs-test-server       latest              6a8a2e2eb5d0        4 hours ago         896MB
ecs-test-server                                                         latest              6a8a2e2eb5d0        4 hours ago         896MB
본인의_AWS_아이디.dkr.ecr.ap-northeast-2.amazonaws.com/ecs-test-data-store   latest              86b07d85d96f        5 hours ago         891MB
ecs-test-data-store                                                     latest              86b07d85d96f        5 hours ago         891MB
node                                                                    8                   06f7c071f445        5 days ago          889MB
```

마지막으로 ECR로 올리는 명령을 수행합니다:

```bash
$ docker push 본인의_AWS_아이디.dkr.ecr.ap-northeast-2.amazonaws.com/ecs-test-server

The push refers to repository [본인의_AWS_아이디.dkr.ecr.ap-northeast-2.amazonaws.com/ecs-test-server]
0cc269ea8130: Pushed
df66e9d317c1: Pushed
56070592d8e0: Pushed
08e647fbce04: Pushed
807ae764c964: Pushed
657d51419301: Pushed
4a6166f16a0e: Pushed
e02b32b1ff99: Pushed
f75e64f96dbc: Pushed
8f7ee6d76fd9: Pushed
c23711a84ad4: Pushed
90d1009ce6fe: Pushed
latest: digest: sha256:a4f65e42ef8d6e79c2c3716b2098376f7b310be90ea580b2039d2c0e01b404a8 size: 2842
```

```
$ docker push 본인의_AWS_아이디.dkr.ecr.ap-northeast-2.amazonaws.com/ecs-test-data-store

The push refers to repository [본인의_AWS_아이디.dkr.ecr.ap-northeast-2.amazonaws.com/ecs-test-data-store]
148ad4293523: Pushed
eff128fc87a2: Pushed
c037366e1e11: Pushed
08e647fbce04: Pushed
807ae764c964: Pushed
657d51419301: Pushed
4a6166f16a0e: Pushed
e02b32b1ff99: Pushed
f75e64f96dbc: Pushed
8f7ee6d76fd9: Pushed
c23711a84ad4: Pushed
90d1009ce6fe: Pushed
latest: digest: sha256:738ff5b5d9a992f8643ed5cc2ce4590c3030bfce4ade4664897c07ae1c0a4f7e size: 2842
```

![](/assets/ecr-list.png)

ECR 페이지에서 이미지가 정상적으로 등록된 것을 확인 할 수 있습니다.

## 마무리


지금까지 우리는 내 컴퓨터에서 작성한 애플리케이션을 로컬 docker 환경에서 이미지화 하여 실행해보았고, ECS에서 실행시키기 위해 ECR repository에 이미지를 등록했습니다.

이제 남은 것은 ECS에 클러스터를 만들고 이 이미지들을 기반으로 서비스를 만들어 돌려보는 일만 남았습니다.