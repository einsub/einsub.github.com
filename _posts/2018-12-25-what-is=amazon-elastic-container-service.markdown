---
layout: post
title: "Amazon CLI로 서버를 ECS에서 구동하기 1 - 시작"
date: 2018-12-25 17:04
author: Reid
categories:
  - platform
tags:
  - amazon
  - aws
  - ecs
  - docker
  - amazon cli
published: false
---
# Elastic Container Service
> 참고: 이 서비스를 이용하기 위해서는 Docker에 대한 기본적인 지식이 있어야 합니다. 아마존의 ['Docker란 무엇입니까?'](https://aws.amazon.com/ko/docker/)를 참고하세요.

Docker의 출현은 소프트웨어 산업에 큰 영향을 미치고 있습니다. Amazon, Google, Microsoft로 대표되는 세계 3대 클라우드 서비스는 모두 Docker 이미지를 담는 컨테이너 서비스를 주력으로 하고 있습니다.

Amazon은 Elastic Container Service(ECS)로 Docker 컨테이너 서비스를 운용하고 있습니다. 얼마전까지 EC2 Container Service 였던걸로 기억하는데, Fargate를 런칭하면서 좀 더 큰 범위를 포괄하는 이름으로 변경한 것 같습니다. Fargate가 출시되기 전에는 컨테이너가 구동될 머신의 EC2 사양도 직접 설정하는 등 말처럼 간단하지만은 않은 서비스였습니다. 하지만 Fargate는 이 하위 레벨의 복잡한 설정들을 추상화시켜 감추었습니다. 그리고 ECS를 serverless한 서비스로 홍보하고 있습니다.

이 글은 간단한 서버 애플리케이션을 작성해서 ECS를 통해 배포하고 동작시키는 과정을 Amazon CLI로 진행합니다. 특히 작년에 미국 region을 시작으로 지금은 서울 region까지 서비스가 확대된 Fargate를 활용합니다. 직전 프로젝트에서 Fargate 없는 ECS를 사용하여 백엔드를 구축했었는데 지금은 얼마나 편리해졌을지 기대됩니다. 

# ECS 구성
- 한 region 내의 새 VPC 혹은 기존의 VPC에서 서비스를 시작 할 수 있습니다.
- 클러스터는 VPC내에 만들어지고 여러 Availability Zone(AZ)에 걸쳐서 생성 될 수 있습니다. AZ 장애에 대한 대처가 유연합니다.
- 클러스터에는 Task나 Service를 만들 수 있습니다. Task는 단발성 작업, Service는 웹서버와 같이 오랫동안 실행되고 장애 발생 시 자동으로 복구되어야 하는 등의 작업에 유용합니다.
- Fargate Launch Type으로 태스크가 구동되면 EC2 인스턴스나 Auto Scale 등의 설정을 ECS가 스스로 합니다.
- 각각의 태스크는 Elastic Network Interace(ENI)를 통해 통신합니다.
- 사용자는 서버 애플리케이션의 docker 이미지를 AWS Container Registry에 올립니다.
- ECS는 새로 업데이트 된 docker 이미지로 자동 갱신하고 서비스를 제공합니다.

![](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/images/overview-fargate.png)

