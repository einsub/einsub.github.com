---
layout: post
title: "ECS CLI로 Fargate 클러스터 생성하기"
date: 2018-12-24 01:22:52
author: Reid
categories:
  - engineering
tags:
  - aws
  - ecs
  - fargate
published: true
---
> 참고 페이지: https://docs.aws.amazon.com/ko_kr/AmazonECS/latest/developerguide/ECS_CLI_installation.html

> 참고: 예제는 모두 macOS 기준

## ECS CLI
- 로컬 개발 환경에서 클러스터 생성, 업데이트, 모니터링을 간편하게 할 수 있음
- [Docker Compose](https://docs.docker.com/compose/) [버전3](https://docs.docker.com/compose/compose-file/)를 지원
- AWS 관리용이 아니라 로컬에서의 개발 및 테스트 도구로 사용 할 수도 있음

## AWS 계정 설정

## 설치

1. 다운로드
    ```sh
    $ sudo curl -o /usr/local/bin/ecs-cli https://s3.amazonaws.com/amazon-ecs-cli/ecs-cli-darwin-amd64-latest
    ```
2. 유효성 확인
    - 출력되는 두 문자열을 비교
    ```sh
    $ curl -s https://s3.amazonaws.com/amazon-ecs-cli/ecs-cli-darwin-amd64-latest.md5 && md5 -q /usr/local/bin/ecs-cli
    ```
3. 실행 권한 부여
    ```sh
    $ sudo chmod +x /usr/local/bin/ecs-cli
    ```
4. 정상 설치 확인
    ```sh
    $ ecs-cli --version
    ```

## IAM 역할 생성
- ECR에서 컨테이너 이미지를 가져오거나 awslogs 로그 드라이버를 사용하기 위해 **작업 실행 역할**이 필요
1. `task-execution-assume-role.json` 파일 생성
    ```json
    {
      "Version": "2012-10-17",
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
2. 작업 실행 역할 생성
    ```sh
    $ aws iam --region us-east-1 create-role --role-name ecsTaskExecutionRole --assume-role-policy-document file://task-execution-assume-role.json
    ```
3. 작업 실행 역할과 정책 연결
    ```sh
    $ aws iam --region us-east-1 attach-role-policy --role-name ecsTaskExecutionRole --policy-arn arn:aws:iam::aws:policy/service-role/AmazonECSTaskExecutionRolePolicy
    ```

## 구성

- ECS CLI가 사용자를 대신해 API 요청을 하기 위해 자격 증명을 부여
- 구성 정보는 `~/.ecs`에 저장

1. 기본 설정
    ```sh
    $ ecs-cli configure --cluster tutorial --region us-east-1 --default-launch-type FARGATE --config-name tutorial
    ```
2. ECS 설정
    - `cluster_name`: 만들 클러스터 이름
    - `launch_type`: 사용할 시작 유형
    - `region_name`: AWS 리전
    - `config-name`: 부여하고 싶은 설정 이름
    ```sh
    $ ecs-cli configure profile --access-key AWS_ACCESS_KEY_ID --secret-key AWS_SECRET_ACCESS_KEY --profile-name tutorial
    ```

## 클러스터와 보안 그룹 생성
1. 클러스터 생성
    - `ecs-cli up` 명령으로 클러스터 생성
    - Fargate 시작 유형은 빈 클러스터와 퍼블릭 서브넷 두 개로 구성된 VPC를 생성
    ```sh
    $ ecs-cli up
    ```
    > 나중에 사용하기 위해 VPC와 Subnet을 기억해둡니다.
2. 보안 그룹 생성
    - 이전의 VPC ID를 입력하여 보안 그룹을 생성
    ```sh
    $ aws ec2 create-security-group --group-name "my-sg" --description "My security group" --vpc-id "VPC_ID"
    ```
3. 엑세스 허용
    - 보안 그룹 규칙을 추가하여 80번 포트에 대한 인바운드 접근을 허가
    ```sh
    $ aws ec2 authorize-security-group-ingress --group-id "security_group_id" --protocol tcp --port 80 --cidr 0.0.0.0/0
    ```

## Compose 파일 생성
- WordPress 애플리케이션을 생성하는 간단한 Docker compose 파일
- Docker compose v3 사용
- 웹 서버로의 인바운드 트래픽을 위해 80번 포트를 노출
- 컨테이너 로그는 CloudWatch 로그 그룹으로 이동
- Fargate 권장 방식

docker-compose.yml
```yaml
version: '3'
services:
  wordpress:
    image: wordpress
    ports:
      - "80:80"
    logging:
      driver: awslogs
      options: 
        awslogs-group: tutorial
        awslogs-region: us-east-1
        awslogs-stream-prefix: wordpress
```

- ECS의 고유한 파라메터를 전달하기 위해 추가 파일을 생성해야 함
- VPC, 서브넷, 보안그룹 등을 설정

ecs-params.yml
```yaml
version: 1
task_definition:
  task_execution_role: ecsTaskExecutionRole
  ecs_network_mode: awsvpc
  task_size:
    mem_limit: 0.5GB
    cpu_limit: 256
run_params:
  network_configuration:
    awsvpc_configuration:
      subnets:
        - "subnet ID 1"
        - "subnet ID 2"
      security_groups:
        - "security group ID"
      assign_public_ip: ENABLED
```

## Compose 파일 배포
- **ecs-cli compose service up** 명령으로 클러스터에 배포
- 프로젝트 이름은 현재 디렉토리 이름이 기본으로 설정되지만 `--project-name`으로 재정의 할 수 있음
```sh
$ ecs-cli compose --project-name tutorial service up --create-log-groups --cluster-config tutorial
```

## 실행중인 컨테이너 보기
- **ecs-cli compose service ps** 명령으로 실행 중인 클러스터 확인
```sh
$ ecs-cli compose --project-name tutorial service ps --cluster-config tutorial
```
> 나중에 사용하기 위해 task-id를 기억해둡니다.

## 컨테이너 로그 확인
- **ecs-cli logs** 명령으로 로그 확인
- 기억해두었던 task-id를 입력
```sh
$ ecs-cli logs --task-id a06a6642-12c5-4006-b1d1-033994580605 --follow --cluster-config tutorial
```

## 클러스터 작업 규모 조정
- **ecs-cli compose service scale** 명령으로 작업 개수를 변경 가능
- 애플리케이션 실행 개수를 2개로 늘림
```sh
$ ecs-cli compose --project-name tutorial service scale 2 --cluster-config tutorial
```

## 리소스 정리
- **ecs-cli compose service down** 명령으로 컨테이너 중지
- 정리해두지 않으면 계속 요금이 부과될 것임
```sh
$ ecs-cli compose --project-name tutorial service down --cluster-config tutorial
```

- 클러스터 종료
```sh
$ ecs-cli down --force --cluster-config tutorial
```