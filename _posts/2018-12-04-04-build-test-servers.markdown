---
layout: post
title: "Amazon CLI로 ECS에 서버 띄우기 4 - 테스트용 서버와 docker 이미지화"
date: 2018-12-04 08:03:00
author: Reid
categories:
  - engineering
tags:
  - amazon
  - aws
  - ecs
  - docker
  - amazon cli
published: false
---

# 테스트용 서버를 만들어 docker 이미지화 하기
## 테스트용 서버 제작
테스트를 위해 서버 애플리케이션 2개를 만들어 서로 통신을 하게 만들겁니다. server는 클라이언트의 요청을 받아 data-store에게 전달합니다. data-store는 그 요청에 대한 응답을 server로 전달하고, server는 최종적으로 그 응답을 처음 요청한 클라이언트에게 내려주게 됩니다. 단순히 클라이언트와 서버의 통신 이외에 서버간의 내부 통신도 구현해보면 ecs를 이해하는데 도움이 될 것 같습니다.

### 개념도
![](/assets/ecs-test.png)

### 코드
server 코드는 다음과 같습니다:
```javascript
const express = require('express');
const app = express();
const request = require('request')

app.get('/', (req, res) => {
  request('http://localhost:4000/', (err, response, body) => {
    console.log('데이터 스토어로부터 응답을 받았습니다:')
    console.log(body)
    res.send(body)
  })
});

app.listen(3000, () => console.log('Server listening on port 3000!'));
```

클라이언트로부터 요청을 받으면 바로 4000번 포트의 서버로 요청을 보내고, 내려받은 응답을 클라이언트로 내려줍니다.

두 번째 레이어의 data-store 코드는 다음과 같습니다:

```javascript
const express = require('express');
const app = express();

app.get('/', (req, res) => {
  console.log('서버로부터 요청을 받았습니다. 응답을 전송합니다.')
  res.send('응~답');
});

app.listen(4000, () => console.log('Data store listening on port 4000!'));
```

`http://localhost:3000`으로 요청을 보내면 브라우저 화면에는 '응~답'이라는 글씨가 그려질 겁니다.

## docker 이미지화 하기

이제 이 두 서버를 docker 이미지화 시켜야 합니다. 만들어진 이미지들은 [Amazon Elastic Container Registry(ECR)](https://aws.amazon.com/ko/ecr/)로 올려서 ECS 컨테이너에 탑재시키게 됩니다.

일단은 로컬 환경에서 docker 이미지로 만들어 구동해봅시다.

[Docker 공식 홈페이지](https://www.docker.com)의 [다운르도 페이지](https://www.docker.com/products/docker-desktop)에서 자신의 운영체제에 맞는 버전을 받아서 설치합니다. 다운로드를 하기 위해서는 회원 가입을 해야 하는데 이는 [Docker Hub](https://hub.docker.com/)에서 공개 이미지를 다운로드 받을 때도 사용되니 미리 해두는게 좋습니다.

### Dockerfile 생성

각 프로젝트 폴더에 Dockerfile을 생성하여 다음과 같이 작성합니다:

```docker
FROM node:8

# 앱 디렉토리 생성
WORKDIR /usr/src/app

# 앱 의존성 라이브러리들 설치
# wildcard가 들어간 건 package.json과 package-lock.json 모두가 복사되도록 하기 위해서입니다.
COPY package*.json ./

RUN npm install
# 프로덕션에서 빌드 할 때는 아래 구문을 이용하세요.
# RUN npm install --only=production

# 앱 소스를 번들링합니다.
COPY . .

EXPOSE 3000
CMD [ "node", "index.js" ]
```
server는 `EXPOSE 3000`, data-store는 `EXPOSE 4000`으로 각각 자기의 listening 포트를 명시해줘야 합니다.

로컬에 설치된 모듈들과 디버깅 로그들이 복사되는 걸 방지하기 위해 .dockerignore 파일을 만들어서 무시할 파일이나 폴더를 추가합니다.

```text
node_modules
npm-debug.log
```

### Docker 이미지 빌드하기

이제 Docker 이미지를 만들 준비가 끝났습니다. 이미지 빌드 과정에서 로그인을 요청하는 경우가 발생하면 `docker login` 명령으로 로그인을 하셔야 합니다.

server 프로젝트 폴더에서 다음 명령을 입력해서 이미지를 생성합니다.
```bash
$ docker build -t ecs-test-server .
```

data-store 프로젝트 폴더에서 다음 명령을 입력해서 이미지를 생성합니다.
```bash
$ docker build -t ecs-test-data-store .
```

빌드 과정은 다음처럼, 기반 이미지를 내려받고 우리가 Dockerfile에서 설정한 명령들은 순차적으로 실행하여 최종 이미지를 만들어냅니다.

```bash
$ docker build -t ecs-test-server .

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
Successfully tagged ecs-test-server:latest
```

2개의 이미지가 정상적으로 생성되었다면 `docker images` 명령의 결과로 다음처럼 보여집니다.

```bash
$ docker images

REPOSITORY            TAG                 IMAGE ID            CREATED              SIZE
ecs-test-data-store   latest              b398a7e59bb5        About a minute ago   891MB
ecs-test-server       latest              d1ff015a2f5e        10 minutes ago       896MB
node                  8                   06f7c071f445        4 days ago           889MB
```

### 컨테이너 사이의 네트워킹 구현

`docker run` 명령으로 이미지를 실행합니다. `-p` 옵션은 외부 포트와 내부 포트를 매핑 시켜줍니다. 이제 클라이언트는 이 서버에 접근하기 위해 50000번 포트로 접근하게 되며, 컨테이너 내부에서 이 서버 인스턴스의 리스닝 포트 3000번으로 연결이 됩니다. `-d` 옵션을 붙여서 실행하면 백그라운드로 구동됩니다만, 우리는 콘솔 로그를 확인하기 위해 포그라운드로 구동했습니다.

```bash
$ docker run -p 50000:3000 ecs-test-server
Server listening on port 3000!
```

```bash
$ docker run -d ecs-test-data-store
Data store listening on port 4000!
```

```bash
$ docker ps -a
CONTAINER ID        IMAGE                 COMMAND             CREATED             STATUS              PORTS                     NAMES
a313e09570a8        ecs-test-data-store   "node index.js"     About an hour ago   Up About an hour    4000/tcp                  festive_shockley
2f8ef13478b4        ecs-test-server       "node index.js"     About an hour ago   Up About an hour    0.0.0.0:50000->3000/tcp   stupefied_bhabha
```

브라우저에서 `http://localhost:50000/`로 접속해보는데 응답이 없습니다. server 콘솔에 다음과 같은 결과가 나타납니다.

```bash
데이터 스토어로부터 응답을 받았습니다:
undefined
```

이렇게 오류가 나는 이유는 server 컨테이너가 `localhost`로는 data-store 컨테이너의 인스턴스를 찾지 못하기 때문입니다. 현재 각 컨테이너는 내부적으로 완전히 독립적인 머신처럼 동작합니다. docker의 기본 네트워킹은 `default bridge network` 방식인데, 이 네트워킹을 바탕으로 구동중인 컨테이너의 사설 ip로 접근은 가능하지만, 컨테이너 이름으로 접근하는 것은 불가능합니다. 이를 가능하게 하려면 사용자가 직접 네트워크를 만들어줘야 합니다.

일단 현재 docker에서 제공하는 네트워크 방식들을 살펴봅시다:

```bash
16:30 $ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
9d1dc010f42c        bridge              bridge              local
944f9093e65f        host                host                local
775577ff5276        none                null                local
```

우리가 사용중인 bridge 네트워크의 상태를 살펴봅시다:

```bash
$ docker inspect bridge
[
    {
        "Name": "bridge",
        "Id": "9d1dc010f42c21c8ae2de8d49aa9c803259b320d3166f8843a6fe20d6c93cb8a",
        "Created": "2018-12-03T07:00:23.41814951Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "2f8ef13478b47a303b337e1a935385a2a73edddaf9631f7c9b52fc2d56b54f09": {
                "Name": "stupefied_bhabha",
                "EndpointID": "74a7f963783d98363c56664c53f34b4eeade2703f320608fa7aceae7aa76a297",
                "MacAddress": "02:42:ac:11:00:03",
                "IPv4Address": "172.17.0.3/16",
                "IPv6Address": ""
            },
            "a313e09570a863673db92c8e308821aa0cda3785aeb2c9a3778711deaa8d10df": {
                "Name": "festive_shockley",
                "EndpointID": "a2a49190e54416d762f37ea973d78e1a4b74f68ba5af74dff88ef66c885b3fcc",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]
```

`Containers` 항목을 살펴보면 `172.17.0.2`와 `172.17.0.3`으로 binding되어 있는 것을 볼 수 있습니다. 컨테이너간의 통신을 ip로 바라보도록 하면 ip가 변경되는 경우 대처가 불가능하기 때문에 추천하지 않습니다. 컨테이너의 이름을 바라볼 수 있도록 직접 네트워크를 만들어 이용해봅시다.

```bash
$ docker network create --driver bridge ecs-test

69c6ea9737bcefbe70c4aae11d6f17fb0e3dfc719cd05b88859e544941dcef64

$ docker network ls

NETWORK ID          NAME                DRIVER              SCOPE
9d1dc010f42c        bridge              bridge              local
69c6ea9737bc        ecs-test            bridge              local
944f9093e65f        host                host                local
775577ff5276        none                null                local
```

server와 data-store가 새로 만들어진 network를 이용하도록 컨테이너 구동 시 파라메터를 넘겨야 합니다. 일단 실행중인 컨테이너를 종료하고 제거합니다. 이미지로부터 컨테이너를 새로 구동 할 때는 컨테이너의 이름을 직접 부여하겠습니다. 참고로 아래 `stop`과 `rm` 명령에 넘기는 컨테이너의 이름은 이름을 직접 부여하지 않을 때 Docker가 자동으로 부여하는 임시 이름입니다.

```bash
$ docker stop festive_shockley
festive_shockley
$ docker stop stupefied_bhabha
stupefied_bhabha
$ docker rm festive_shockley
festive_shockley
$ docker rm stupefied_bhabha
stupefied_bhabha
```

### Docker 이미지 실행하기

server의 소스 코드에서 data-store에 접속하는 주소를 우리가 부여할 컨테이너 이름으로 변경하고 다시 빌드합니다.

```javascript
const express = require('express');
const app = express();
const request = require('request')

a pp.get('/', (req, res) => {
  // http://ecs-test-data-store:4000/ 으로 변경합니다.
  request('http://ecs-test-data-store:4000/', (err, response, body) => {
    console.log('데이터 스토어로부터 응답을 받았습니다:')
    console.log(body)
    res.send(body)
  })
});

app.listen(3000, () => console.log('Server listening on port 3000!'));
```

```bash
$ docker build -t ecs-test-server .
```

빌드가 완료되면 server와 data-store에 이름을 부여해서 다시 실행하고 브라우저에서 `http://localhost:50000`으로 다시 접속해봅니다.

```bash
$ docker run --network=ecs-test --name=ecs-test-server -p 50000:3000 ecs-test-server
Server listening on port 3000!
데이터 스토어로부터 응답을 받았습니다:
응~답
```

```bash
$ docker run --network=ecs-test --name=ecs-test-data-store ecs-test-data-store
Data store listening on port 4000!
서버로부터 요청을 받았습니다. 응답을 전송합니다.
```

드디어 잘 동작합니다!

## 마치며

이제 우리는 애플리케이션을 Docker 이미지로 만들고 구동하는데 성공했습니다. 이제 이 이미지를 Amazon ECR에 올려서 ECS가 이미지를 가져다 컨테이너에서 구동하는데 사용하도록 할 것입니다.

도커화(Dockerize) 시키는데 가장 어렵고 신경을 써야 하는 부분은 Service discovery입니다. 이 문서의 경우처럼 간단히 두 애플리케이션이 한 네트워크 환경안에서 바라보게 하는 것은 그리 어렵지 않은 문제지만, scale을 위해 여러 컨테이너를 띄워 최적의 컨테이너를 연결해준다거나 새로운 컨테이너의 추가, 기존 컨테이너의 삭제 등에 따른 연결 환경의 변화 등에 대응하는 것은 매우 까다로운 일입니다.

Amazon ECS는 이 문제를 어떻게 해결하는지 차차 알아봅시다.