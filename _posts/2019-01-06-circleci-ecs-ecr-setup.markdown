---
layout: post
title: "AWS ECS/ECR 프로젝트를 위한 CircleCI 설정"
date: 2019-01-06 22:41:23
author: Reid
categories:
  - engineering
tags:
  - aws
  - ecr
  - ecs
  - circleci
  - continuous integration
  - continuous distribution
  - ci/cd
published: true
---
## CircleCI 환경 변수 설정

CircleCI는 미리 설정해둔 환경변수를 빌드 과정에서 사용 할 수 있습니다. 기본으로 탑재된 환경변수 외에 사용자가 직접 추가 할 수 있으므로, AWS 계정 정보등을 입력해두면 편리합니다. 참고로 AWS_ACCESS_KEY_ID와 AWS_SECRET_ACCESS_KEY, AWS_DEFAULT_REGION은 configure 과정에서 필요하므로 꼭 입력해야 합니다.

|name|value|
|---|---|
|APP_NAME|MyApp|
|AWS_ACCOUNT_ID|268238064276|
|AWS_ACCESS_KEY_ID|AKICMLISJFNLASDJCKAW|
|AWS_SECRET_ACCESS_KEY|AZ9mD+CidjSDjncv93SDjmcSjF92jCpajfWjjQnx|
|AWS_DEFAULT_REGION|ap-northeast-2|
|AWS_ECR_URL|268238064276.dkr.ecr.ap-northeast-2.amazonaws.com|

## .circleci/config.yml 

CircleCI가 repository로 push된 프로젝트의 `.circleci/config.yml`을 읽어서 빌드/배포 프로세스를 진행합니다. job을 build와 deploy 두 단계로 나누었으며, workflows 항목에서 정의 한 것처럼 master 브랜치만 build 이후 deploy 단계까지 진행합니다. 


```yaml
version: 2.0
jobs:
  build:
    docker:
      - image: docker:18.09.0
    steps:
      - checkout
      - setup_remote_docker
      - run:
          name: Install dependencies
          command: |
            apk update && apk add curl
      - run:
          name: Build Docker image
          command:
            >-
            docker build \
              -t $AWS_ECR_URL/$APP_NAME:$CIRCLE_SHA1 \
              -t $AWS_ECR_URL/$APP_NAME:latest .
      - run:
          name: Run unit tests
          command: |
            docker run $AWS_ECR_URL/$APP_NAME:latest yarn test
      - run:
          # 빌드한 docker 이미지를 파일로 저장합니다.
          name: Save Docker image layer cache
          command: |
            mkdir cache
            docker save -o cache/$APP_NAME.tar $AWS_ECR_URL/$APP_NAME
      - persist_to_workspace:
          # 다음 job에서 docker 이미지를 사용 할 수 있도록 workspace에 저장해둡니다.
          root: .
          paths:
              - cache
  deploy:
    docker:
      - image: docker:18.09.0
    steps:
      - checkout
      - setup_remote_docker
      - attach_workspace:
          # build 단계에서 사용한 workspace를 붙여줍니다.
          at: workspace
      - run:
          name: Install dependencies
          command: |
            apk update && apk add py-pip bash jq
            pip install awscli==1.16.60
      - run:
          # 이전에 빌드해둔 파일로부터 docker 이미지를 불러옵니다.
          name: Load Docker image layer cache
          command: |
            docker load -i workspace/cache/$APP_NAME.tar
      - deploy:
          # ECS에 deploy하는 복잡한 단계는 별개의 shell script로 진행합니다.
          name: Deploy to ECS
          command: |
            chmod +x ./deploy.sh
            ./deploy.sh
workflows:
  version: 2
  build-deploy:
    jobs:
      - build
      - deploy:
          requires:
            - build
          filters:
            # master 브랜치만 deploy를 진행합니다.
            branches:
              only: master
```

## deploy.sh

이 스크립트가 진행하는 작업은 다음과 같습니다:

1. AWS ECR 로그인합니다.
2. MyApp docker 이미지를 ECR에 push 합니다.
3. task definition을 등록합니다. task 정의가 바뀌었다면 여기서 수정해서 등록합니다.
4. 현재 서비스에 설정된 desired count를 가져옵니다.
5. 가져온 desired count로 서비스를 업데이트시킵니다.
6. 서비스에 등록된 task definition이 방금 올린 revision을 가질 때 까지 10초 간격으로 10분간 대기합니다.
7. 배포가 성공하면 **Deployed!** 메시지가 출력됩니다.

```sh
#!/bin/bash

set -e
set -u
set -o pipefail

JQ="jq --raw-output --exit-status"

project="MyApp"
family="${project}"
cluster="MyCluster"
service="${project}"
task="${project}"
image_name="${project}"
repo_url="${AWS_ECR_URL}/${image_name}"

task_def="[
  {
    \"name\": \"$task\",
    \"image\": \"$repo_url:$CIRCLE_SHA1\",
    \"portMappings\": [
      {
        \"containerPort\": 80,
        \"hostPort\": 80,
        \"protocol\": \"tcp\"
      }
    ],
    \"logConfiguration\": {
      \"logDriver\": \"awslogs\",
      \"options\": {
        \"awslogs-group\": \"awslogs-my-app\",
        \"awslogs-region\": \"$AWS_DEFAULT_REGION\"
      }
    },
    \"essential\": true,
    \"entryPoint\": [
      \"yarn\",
      \"serve\"
    ]
  }
]"

push_image_to_ecr() {
  aws --version
  aws configure set default.region $AWS_DEFAULT_REGION
  aws configure set default.output json

  echo "Logging into ECR"
  eval $(aws ecr get-login --no-include-email)

  echo "Pushing image "
  docker push $repo_url:$CIRCLE_SHA1
  docker push $repo_url:latest
}

register_definition() {
  if revision=$(aws ecs register-task-definition \
    --container-definitions "${task_def}" \
    --family "${family}" \
    --execution-role-arn "arn:aws:iam::$AWS_ACCOUNT_ID:role/ecsTaskExecutionRole" \
    --network-mode "awsvpc" \
    --requires-compatibilities "FARGATE" \
    --cpu "256" \
    --memory "512" | $JQ '.taskDefinition.taskDefinitionArn'); then
    echo "Revision: ${revision}"
  else
    echo "Failed to register task definition"
    return 1
  fi
}

deploy_cluster() {
  desired_count=$(aws ecs describe-services --cluster ${cluster} --services ${service} | $JQ '.services[0].desiredCount')
  echo "Desired count: ${desired_count}"

  if [[ $(aws ecs update-service --cluster ${cluster} --service ${service} --desired-count ${desired_count} --task-definition ${revision} | \
        $JQ '.service.taskDefinition') != $revision ]]; then
    echo "Error updating service."
    return 1
  else
    echo "Success updating service."
  fi

  for attemp in {1..60}; do
    if stale=$(aws ecs describe-services --cluster ${cluster} --services ${service} | \
              $JQ ".services[0].deployments | .[] | select(.taskDefinition != \"$revision\") | .taskDefinition"); then
      echo "Waiting for stale deployments: "
      echo "$stale"
      sleep 10
    else
      echo "Deployed!"
      return 0
    fi
  done
  echo "Service update took too long."
  return 1
}

push_image_to_ecr
register_definition
deploy_cluster
```
