---
layout: post
title: "[번역] Node.js 프로덕션 환경을 위한 Best Practice 모음"
date: 2019-03-31 00:58:24
author: Reid
categories:
  - engineering
tags:
  - node.js
  - production
  - deploy
  - express
  - best practice
published: true
---

> 원문: [Checklist: Node.JS production best practices (August 2018)](https://goldbergyoni.com/checklist-best-practice-of-node-js-in-production/) - [Yoni Goldberg](mailto:me@goldbergyoni.com)

Node.js 애플리케이션을 프로덕션 환경에서 서비스하기 위한 모범 사례(Best Practice) 컬렉션에 오신 것을 환영합니다. 이 글은 최고의 블로그들로부터 얻은 훌륭한 지식들을 요악, 정리하는 것이 목표입니다.

**놓치지 마세요**: 각 모범 사례들에는 "GIST Popup" 아이콘이 있습니다. 클릭하면 추가 설명과 인용구, 코드 예제들을 확인 하실 수 있습니다. (역주: 이 번역글에서는 따로 팝업을 띄우지 않고 "GIST"의 내용을 펼쳐서 작성하였습니다.)

Yoni Goldberg - Node [컨설팅](https://goldbergyoni.com/5-nodejs-services-that-i-offer/)과 [교육](https://courses.goldbergyoni.com/)을 제공하는 독립 Node.JS 컨설턴트. 70개 이상의 모범 사례를 담은 [GitHub](https://github.com/i0natan/nodebestpractices)을 참고하세요.

# 1. 모니터링하기

모니터링은 고객이 찾아내기 전에 먼저 문제를 찾아내는 게임입니다. 이것은 엄청나게 중요합니다. 요 근래 모니터링 업계는 압도적으로 많은 기능들을 제공하고 있으며, 이는 선택을 하는데 어려움을 주는 요인이 됩니다. 그러므로 반드시 확인해야 하는 기본적인 지표들을 정의하는 것 부터 시작하고, 그 후 기호에 따라 추가적인 기능들을 설치하는 것이 좋습니다. 그렇게 최종 목표를 향해 하나씩 필요한 것들을 채워나가는 방식으로 접근하세요.

**OR**: 실패 === 실망한 고객들. 간단합니다.

## 좀 더 자세히

기본적으로 모니터링은 프로덕션 서비스에서 문제가 발생하고 있음을 쉽고 빠르게 알 수 있게 해줍니다. 이메일이나 슬랙 등으로 알림을 받는 것 처럼 말이죠. 문제는 은행 잔고의 압박을 받지 않고도 만족할만한 도구들을 찾을 수 있느냐는 것입니다. 제안하건데, 이상 없이 잘 돌아가고 있다는 걸 눈으로 확인시켜 줄 수 있는 핵심적인 지표들을 정의하는 것 부터 시작하십시요. CPU, 서버 메모리, Node 프로세스 메모리 (1.4GB 미만), 마지막 1분 동안 에러의 수, 프로세스가 재시작한 횟수, 평균 응답 시간 등을 말이죠. 그 후 여러분의 기호에 따라 보다 고급 기능들을 살펴보고 위시 리스트에 올려두세요. 고급 모니터링 기능들의 예를 들자면: DB 프로파일링, 크로스 서비스 측정 (예: 비즈니스 트랜잭션 측정), 프론트엔드 통합, 커스텀 BI 클라이언트에 raw 데이터 노출시키기, 슬랙 알림 등 여러가지가 있습니다.

고급 기능을 구현하려면 DataDog, newrelic 같은 제품을 돈을 내고 사용해야 합니다. 하지만 안타깝게도, 기본 기능을 구현하는 것 역시 만만치 않습니다. 기본 기능을 제공하는 제품 역시 어떤 제품은 하드웨어만 다루고 어떤 제품은 Node 프로세스만을 다루기 때문에 결국은 추가적인 설치가 필요합니다. 예를 들어, 클라우드 기반의 모니터링 솔루션([AWS CloudWatch](https://aws.amazon.com/cloudwatch/), [Google StackDriver](https://cloud.google.com/stackdriver/) 같은 것들)은 문제가 생기면 즉시 하드웨어를 측정한 지표들을 제공해주지만, 그 안의 앱이 어떻게 동작하는지는 아무것도 알려주지 않습니다. 반면에, ElasticSearch와 같은 로그 기반의 솔루션들은 대체로 하드웨어 관점에서의 접근이 부족합니다. 놓치고 있는 측정 지표들을 기준으로 점차 선택을 넓히는 것이 해결책입니다. 인기있는 방식을 하나 예로 들어보자면, 애플리케이션 로그들은 [Elastic Stack](https://www.elastic.co/products)으로 보내고 추가 에이전트(예: [Beat](https://www.elastic.co/products))로 하드웨어 관련 정보를 공유하여 전체 그림을 완성시키는 것입니다.

### 모니터링 예제: AWS cloudwatch 기본 대시보드, 앱 내부의 지표는 얻기가 어렵습니다

[![](/assets/2019-03-30-cloudwatch.png)](/assets/2019-03-30-cloudwatch.png)

### 모니터링 예제: StackDriver 기본 대시보드, 앱 내부의 지표는 얻기가 어렵습니다

[![](/assets/2019-03-30-stackdriver-1-1024x576.jpg)](2019-03-30-grafana-dashboard2-1-1024x551)

### 모니터링 예제: Grafana를 UI 레이어로 사용하면 raw 데이터를 시각화 해줍니다.

[![](/assets/2019-03-30-grafana-dashboard2-1-1024x551.png)](/assets/2019-03-30-grafana-dashboard2-1-1024x551.png)

## 다른 블로거의 이야기들

블로그 [Rising Stack](http://mubaloo.com/best-practices-deploying-node-js-applications)

> 모든 서비스들에 대해 다음의 지표들을 관찰 하는 것이 좋습니다:<br/>
- **에러율**: 에러는 사용자에게 직접 노출되기 때문에, 고객들이 즉시 영향을 받습니다.
- **응답시간**: 늦은 응답시간은 고객과 비즈니스에 직접적인 영향을 줍니다.
- **처리율**: 트래픽은 에러율과 지연시간이 커지는 원인을 파악하는데 도움을 줍니다.
- **포화싱테**: 서비스가 얼마나 "가득"찼는지 알려줍니다. CPU 사용량이 90%인 시스템이 더 많은 트래픽을 처리 할 수 있을까요?

# 2. 스마트 로깅으로 서비스의 투명성 향상시키기

로그는 디버그 데이터들로 가득찬 의미없는 창고가 될 수도 있고, 앱의 이야기를 전달하는 아름다운 대시보드가 될 수도 있습니다. 첫 날부터 로깅 플랫폼에 대한 계획을 세우십시요. 로그를 수집하고 저장하고 분석하기 위한 계획을 세워서, 원하는 정보를 실제로 얻어낼 수 있도록 합니다.

**OR**: 당신은 원인은 알 수 없이 결과만 보여지는 블랙박스만 남겨놓게 됩니다. 추가 정보를 얻기 위해 로그를 위한 코드들을 다시 작성해야 할 것입니다.

## 좀 더 자세히

어쨌든 로그문을 출력하고 에러 등의 핵심 지표들을(예: 한 시간 동안 얼마나 많은 에러가 발생했는지, 가장 느린 API 엔드포인트가 무엇인지) 추적 할 수 있는 인터페이스는 분명히 필요합니다. 요구사항을 모두 만족시키는 강력한 로깅 프레임워크에 적당히 투자해보는건 어떻습니까? 이걸 이루기 위해서는 다음 세 단계의 사려깊은 결단이 필요합니다.

### 1. 똑똑한 로깅

최소한 [Winston](https://github.com/winstonjs/winston)이나 [Bunyan](https://github.com/trentm/node-bunyan) 같은 이름있는 로깅 라이브러리를 사용해서 각 트랜잭션의 처음과 끝에 의미있는 정보들을 적습니다. 로그는 JSON 형식을 사용하고, 모든 맥락들을 담고 있는 속성들(예를 들어, 사용자ID, 작업 종류 등)을 제공해서 운영팀이 이런 필드들로 액션을 취할 수 있게 하는 것이 좋습니다. 각 로그 라인마다 고유한 트랜잭션 ID를 포함시키는 것도 필요합니다. 자세한 내용은 아래의 16번 항목을 참고하세요. 고려해야 할 마지막 요점은 Elastic Beat와 같은 에이전트를 이용해 메모리나 CPU같은 시스템 자원을 기록하는 것입니다.

### 2. 똑똑한 수집

서버의 파일 시스템에 수집된 정보들을 수집, 집계, 가공하여 화면으로 보여주기 위한 시스템으로 주기적으로 전달해야 합니다. 예를 들어, Elastic stack은 데이터를 수집하고 보여주는 모든 컴포넌트을 무료로 사용 할 수 있도록 해주는 도구로 유명합니다. 유료 제품들은 보통 편리한 설치와 호스팅을 할 필요가 없다는 장점 이외에는 비슷한 기능을 제공하고 있습니다.

### 3. 똑똑한 시각화

수집한 정보들을 검색하는 것이 가능해지면, 로그 검색이 쉬워졌다는 것 만으로 만족할 수 있겠습니다만 추가로 코딩을 하는 노력 없이도 더 많은 것들을 이루어 낼 수 있습니다. 이제 우리는 에러율, 일간 CPU 평균 처리량, 지난 한 시간동안 얼마나 많은 신규 사용자들이 유입되었는지 등 앱을 관리하거나 향상시키는데 도움이 되는 중요한 운영 지표들을 확인 할 수 있게 되었습니다.

### 시각화 예제: Kibana(Elastic stack의 한 부분)은 로그 내용을 검색하는 고급 기능을 제공합니다.

[![](/assets/2019-03-30-kibana-raw-1024x637.png)](/assets/2019-03-30-kibana-raw-1024x637.png)

### 시각화 예제: Kibana(Elastic stack의 한 부분)은 로그 데이터를 기반으로 시각화를 할 수 있습니다.

[![](/assets/2019-03-30-kibana-graph-1024x550.jpg)](/assets/2019-03-30-kibana-graph-1024x550.jpg)

## 다른 블로거의 이야기들

블로그 [Strong Loop](https://strongloop.com/strongblog/compare-node-js-logging-winston-bunyan/)

> 몇 가지 로깅 서비스의 요구 사항을 짚어봅시다.<br />
1. 각 로그 라인에 타임스탬프를 기록하여 각각의 로그가 언제 발생했는지 알 수 있어야 합니다.
2. 로깅 포맷은 인간뿐만 아니라 기계에 의해서도 쉽게 처리될 수 있어야 합니다.
3. 여러 목적지를 대상으로 설정 할 수 있도록 스트림을 사용해야 합니다. 예를 들어, 파일에 추적 로그를 기록하면서, 에러가 발생하면 추가로 오류 파일에도 기록을 하고 이메일까지 발송하는 것 처럼 말입니다.

# 3. 가능한 모든 것들(예: gzip, SSL)을 reverse proxy에 위임하기

Node는 압축, SSL 종료 등 CPU에 민감한 작업을 하기에는 적당하지 않습니다. nginx, HAProxy, Cloud vendor 서비스와 같은 "진짜" 미들웨어 서비스를 사용하세요.

**OR**: 여러분의 불쌍한 싱글 쓰레드는 중요한 비즈니스 로직을 수행 하는 대신 네트워킹 작업들만 바쁘게 처리하게 되고, 그에 따라 자연히 성능은 저하 될 것입니다.

## 좀 더 자세히

정적 파일 제공, gzip 인코딩, 요청 쓰로틀링, SSL 종료 등과 같은 네트워크 관련 작업들에 대해 Express의 풍부한 미들웨어를 사용하고 싶어지는 유혹은 클 수 있습니다. 이런 작업들은 CPU를 오랫동안 바쁘게 구동시켜야 하기 때문에 싱글 쓰레드 모델에서는 성능 저하가 발생합니다. (기억하세요. Node의 실행 모델은 짧은 작업 또는 비동기 IO 작업에 최적화 되어 있습니다). 더 나은 접근 방식은 네트워킹 작업을 전문적으로 처리하는 도구를 사용하는 것입니다. 가장 널리 사용되는 도구로는 nginx와 HAproxy가 있습니다. 거대한 클라우드 업체들도 node.js 프로세스가 감당해야 하는 부하를 줄이기 위해 이런것들을 사용하고 있습니다.

### 코드 예졔: 일반적인 nginx 설정

```nginx
gzip on;

#defining gzip compression
gzip_comp_level 6;
gzip_vary on;upstream myApplication {
  server 127.0.0.1:3000;
  server 127.0.0.1:3001;
  keepalive 64;
}

#defining web server
server {
  listen 80;
  listen 443 ssl;ssl_certificate /some/location/sillyfacesociety.com.bundle.crt;
  error_page 502 /errors/502.html;
  #handling static content
  location ~ ^/(images/|img/|javascript/|js/|css/|stylesheets/|flash/|media/|static/|robots.txt|humans.txt|favicon.ico) {
    root /usr/local/silly_face_society/node/public;
    access_log off;
    expires max;
  }
}
```

## 다른 블로거들의 이야기들

블로그 [Mubaloo](http://mubaloo.com/best-practices-deploying-node-js-applications)

> 이 함정에 빠지기는 매우 쉽습니다. Express 같은 패키지를 보고 생각하죠 "대단해! 이걸로 시작해야겠어!". 코드를 작성하는 것만으로 원하는 동작을 수행하는 애플리케이션을 만들 수 있습니다. 훌륭합니다. 솔직히 이야기해서, 여러분은 단지 전투에서 이긴 것 뿐입니다. 서버를 구동하고 HTTP 포트에서 접속을 기다릴 때, 여러분은 비로소 전쟁에서 졌다는 것을 깨달을 것 입니다. 매우 중요한 사실을 잊고 있기 때문입니다. Node는 웹 서버가 아닙니다. 애플리케이션에 접속이 들어오자마자 문제가 발생하기 시작합니다. 연결이 끊어지거나 asset 제공이 갑자기 멈추고, 최악의 경우 서버가 다운됩니다. 여러분이 지금 하고 있는 것은, 이미 입증 된 웹 서버들이 정말 잘하는 복잡한 작업들을 Node 혼자 처리하도록 하는 것입니다. 왜 바퀴를 다시 발명합니까?<br /><br />
이것은 단지 하나의 이미지에 대한 하나의 요청에 불과합니다. 애플리케이션은 데이터베이스를 읽거나 복잡한 로직을 수행하는 것 같은 중요한 작업을 위해 사용되어야 한다는 것을 기억해야 합니다. 당신의 편의를 위해 애플리케이션을 절름발이로 만들 수는 없습니다.

블로그 [Argteam](http://blog.argteam.com/coding/hardening-node-js-for-production-part-2-using-nginx-to-avoid-node-js-load)

> Express.js는 미들웨어를 통해 정적 파일들을 처리하는 기능을 내장하고 있지만 결코 사용해서는 안됩니다. Nginx는 정적 파일을 다루는 작업을 훨씬 더 잘 수행 할 수 있으며 비동기 컨텐츠에 대한 요청으로 Node 프로세스가 먹통이 되는 것을 막을 수 있습니다.

# 4. 종속성(dependencies) 잠그기

모든 환경에서 코드가 동일해야 하지만 놀랍게도 NPM을 사용하면 패키지를 설치할 때 마다 최신 버전을 가져 오려고 합니다. NPM의 설정 파일인 `.npmrc`를 사용해서 이 문제를 해결해야 합니다. `.npmrc`는 각 환경에 패키지의 최신 버전이 아닌 일치하는 버전을 저장하게 해줍니다. 또 다른 방법으로, 미세한 제어를 위해 NPM `shrinkwrap`을 사용 할 수 있습니다. **업데이트 : NPM5부터 기본적으로 종속성이 잠깁니다. 새로운 패키지 관리자인 Yarn 역시 마찬가지로 동작합니다.**

**OR**: QA 담당자가 코드를 철저히 테스트하고도 프로덕션 환경에서 다르게 동작하는 버전을 승인하게 됩니다. 더 문제가 되는 것은 동일한 프로덕션 환경의 클러스터에서 돌아가는 여러 서버가 서로 다른 코드를 실행할 수도 있다는 점 입니다

## 좀 더 자세히

여러분의 코드가 다양한 외부 패키지에 의존성이 있고, 그 중 `momentjs-2.1.4`를 사용한다고 가정 해 봅시다. 프로덕션 환경에 배포 할 때 NPM이 2.1.5 버전을 가져오는 경우가 발생 할 수도 있습니다. 이는 불행하게도 새로운 버그를 발생시킵니다. NPM 설정 파일을 이용해서 `-save-exact=true`를 설정해두면 NPM이 기존에 설치된 것과 정확하게 일치하는 버전을 참조하도록 지시하고 다음에 "NPM install"(프로덕션 환경이나 Docker 컨테이너 안에서 테스트를 위해 배포될 때)을 실행할 때 같은 종속성을 가진 버전을 내려받게 합니다. 또 다른 방법으로는 어떤 패키지와 버전이 설치되어야 하는지를 명시하는 `.shrinkwrap` 파일(NPM을 사용해서 쉽게 생성 할 수 있는)을 사용하는 것입니다. 이렇게 해두면 어떤 환경에서도 새 버전을 가져 오지 않게 할 수 있습니다.

**업데이트: NPM5부터는 `.shrinkwrap`을 사용하여 종속성이 자동으로 잠금 시킬 수 있습니다. 떠오르는 패키지 관리자인 Yarn은 기본으로 종속성을 잠급니다**.

### 코드 예제: 정확한 버전을 사용하도록 NPM에 지시하는 .npmrc 파일

```
// save this as .npmrc file on the project directory
save-exact: true
```

### 코드 예제: 정확한 depedency 트리를 추출하는 shirnkwrap.json 파일

```json
{
  "name": "A",
  "dependencies": {
    "B": {
      "version": "0.0.1",
      "dependencies": {
        "C": {
          "version": "0.1.0"
        }
      }
    }
  }
}
```

### 코드 예제: NPM 5 종속성 잠금 파일 - package.json

```json
{
  "name": "package-name",
  "version": "1.0.0",
  "lockfileVersion": 1,
  "dependencies": {
    "cacache": {
      "version": "9.2.6",
      "resolved": "https://registry.npmjs.org/cacache/-/cacache-9.2.6.tgz",
      "integrity": "sha512-YK0Z5Np5t755edPL6gfdCeGxtU0rcW/DBhYhYVDckT+7AFkCCtedf2zru5NRbBLFk6e7Agi/RaqTOAfiaipUfg=="
    },
    "duplexify": {
      "version": "3.5.0",
      "resolved": "https://registry.npmjs.org/duplexify/-/duplexify-3.5.0.tgz",
      "integrity": "sha1-GqdzAC4VeEV+nZ1KULDMquvL1gQ=",
      "dependencies": {
        "end-of-stream": {
          "version": "1.0.0",
          "resolved": "https://registry.npmjs.org/end-of-stream/-/end-of-stream-1.0.0.tgz",
          "integrity": "sha1-1FlucCc0qT5A6a+GQxnqvZn/Lw4="
        },
```

# 5. 올바른 도구로 프로세스의 실행 상태 유지하기

프로세스는 계속 실행 중이어야 하고, 오류가 발생해서 꺼지더라도 재시작 되어야 합니다. 간단한 상황에서는 PM2와 같은 도구로 충분하지만, 요즘은 'dockerized' 세상입니다. 그러므로 도커 클러스터를 관리하기 위한 도구도 고려되어야 합니다.

**OR**: 명확한 전략 없이 수십개의 인스턴스를 실행하거나, 너무 많은 도구(클러스터 관리, Docker, PM2)들을 사용하면 devops는 혼돈의 카오스가 될 것 입니다.

## 좀 더 자세히

Node는 장애가 발생하면 기본적으로 프로세스가 보호 받고 재시작이 가능해야합니다. 컨테이너를 사용하지 않고 조그만 애플리케이션을 만드는 개발자에게 [PM2](https://www.npmjs.com/package/pm2-docker)는 복잡하지 않으면서도 재시작이 기능하고 Node와의 풍부한 통합을 제공하기 때문에 안성맞춤입니다. 뛰어난 Linux 기술을 가진 사람들은 systemd를 사용해서 Node를 서비스로 실행할 수 있습니다. Docker 등의 컨테이너 기술을 사용하는 앱의 경우 일반적으로 함께 제공되는 클러스터 관리 도구(예: [AWS ECS](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/Welcome.html), [Kubernetes](https://kubernetes.io/) 등)로 모니터링을 하거나 스스로 컨테이너를 회복시키는 기능을 이용 할 수 있습니다. 풍부한 클러스터 관리 기능에 컨테이너의 재시작 기능까지 있는데, PM2 같은 다른 도구를 굳이 사용해서 문제를 어렵게 만들 필요가 있을까요? 그에 대해 확답을 내리기는 힘듭니다. 프로세스를 가장 앞에서 보호해 줄 녀석으로 PM2를(컨테이너 전용 버전으로 [pm2-docker](https://www.npmjs.com/package/pm2-docker)가 대부분 사용됨) 사용해야 하는 좋은 이유가 있습니다. 프로세스를 재시작 하는 것은 컨테이너의 재시작보다 훨씬 빠르고, 호스팅 컨테이너가 우아한(graceful) 재시작을 요청 할 때, 서버에 플래그를 지정하는 것처럼 Node에 특화된 기능들을 제공하기 때문입니다. 어떤 이들은 불필요한 레이어를 만드는 것을 원하지 않을 수도 있습니다. 이 쯤에서 정리하겠습니다. 모두에게 딱 맞는 해결책은 없습니다. 다만 어떤 옵션이 있는지는 알고 있어야 합니다.

## 다른 블로거의 이야기들

블로그 [Express Production Best Practices](https://expressjs.com/en/advanced/best-practice-performance.html)

> 개발 과정에서 여러분은 간단히 `node server.js`를 입력하는 식으로 앱을 실행시켰습니다. 그러나 프로덕션 환경에서 이런 식으로 프로세스를 시작하는 것은 재앙을 불러일으키는 일입니다. 만약 앱이 크래시가 난다면 다시 시작시킬 때까지 앱은 오프라인 상태가 됩니다. 애플리케이션을 자동으로 재시작 시키기 위해 프로세스 관리자를 사용하십시오. 프로세스 관리자는 배포를 쉽게 하고 가용성을 높이며 실행 시 애플리케이션의 관리를 돕는 애플리케이션을 위한 "컨테이너"입니다.

블로그 [Medium: Node 클러스터링 이해가기](https://medium.com/@CodeAndBiscuits/understanding-nodejs-clustering-in-docker-land-64ce2306afef#.cssigr5z3)

> Docker 컨테이너는 프로세스를 최소화시키기 위해 설계된 가볍고 간소화된 가상 환경입니다. 자신의 리소스를 관리하고 조정해야하는 프로세스는 더 이상 가치가 없습니다. Kubernetes, Mesos, Cattle과 같은 관리 스택은 이러한 리소스를 인프라 차원에서 관리해야한다는 개념을 대중화시켰습니다. CPU와 메모리 리소스는 "스케줄러"에 의해 할당되고 네트워크 리소스는 스택이 제공하는 로드 밸런서에 의해 관리됩니다.

# 6. 오류 관리 Best Practice 만족시키기

Node.JS 환경을 안정적으로 유지시키기 위한 작업 중 오류에 대한 처리가 가장 오랜 시간이 걸리고 고통스럽니다. 이것은 주로 '단일 스레드' 모델과 비동기 흐름에서 오류 경로에 대해 적절히 처리 할 전략이 없기 때문에 발생합니다. 여기에는 간단한 방법이란 없습니다. 오류라는 짐승을 잡기 위해서는 그것을 이해하고 길들이는데 공을 들여야 합니다. 제 [오류 처리 Best Practice](https://goldbergyoni.com/checklist-best-practices-of-node-js-error-handling/)로 이를 좀 더 빠르게 해드릴 수는 있습니다.

**OR**: 사용자가 잘못된 JSON을 보냈다는 이유로 프로세스에 문제가 발생하고 추적 오류는 사라지며 사용자에게 에러 스택 정보가 공개되는 등 미친일들이 계속됩니다.

# 7. 모든 CPU 코어 활용하기

Node는 기본적으로 단일 CPU 코어에서 실행되고 다른 모든 프로그램은 유휴(idle) 상태가 됩니다. 여러 Node 프로세스를 띄워서 모든 CPU를 활용하는 것은 선택이 아니라 필수입니다. 중형 애플리케이션의 경우 Node 클러스터 또는 PM2를 사용할 수도 있습니다. 보다 큰 애플레키에션에서는 Docker 클러스터(예: K8S, ECS) 또는 Linux init 시스템(예: systemd)을 기반으로 하는 배포 스크립트를 사용해서 프로세스를 복제하는 것이 좋습니다.

**OR**: 앱이 리소스(!) 중 25%만 활용하고 있을 가능성이 있습니다. 보통의 서버는 CPU 코어가 4개 이상인 점을 떠올려 보세요. 크게 고민하지 않고 배포하는 경우에 단 한개의 코어만 사용하는 상태로 서비스가 구동 될 수 있습니다. AWS Beanstalk와 같은 PaaS 서비스를 사용하더라도 말이죠!

## 좀 더 자세히

Node가 단일 스레드=단일 프로세스=단일 CPU로 실행된다는 것은 놀랄 일도 아닙니다. 4개 또는 8개의 CPU가 있는 비싼 하드웨어를 사서 단 한 개만 사용한다는 건 그냥 개념이 없는 겁니다. 중간 규모의 애플리케이션에게 가장 적당하고 빠르게 적용 할 수 있는 해결책은 Node의 클러스터 모듈을 사용하는 것입니다. 이 모듈은 10줄의 코드만으로 각각의 코어에 프로세스를 생성하고 라운드 로빈 방식으로 프로세스간 요청을 라우팅합니다. 더 좋은 방법은 간단한 인터페이스와 멋진 모니터링 UI로 클러스터링 모듈을 사용 가능한 PM2를 사용하는 것입니다. 일반적인 애플리케이션에서는 이 방법이 잘 작동하지만, 최고 수준의 성능과 강력한 devops 플로우가 필요한 애플리케이션에게는 적합하지 않습니다. 고급 사용 사례의 경우, nginx 같은 특별한 도구를 사용해서 커스텀 배포 스크립트를 작성하고 로드 밸런싱을 해주거나 AWS ECS나 Kubernetees 같은 컨테이너 엔진을 이용해야 합니다. 이것들은 프로세스를 배치하고 복제하는 고급 기능들을 가지고 있습니다.

### 밸런싱 비교: Node 클러스터 vs nginx

[![](/assets/2019-03-30-nginx-vs-cluster.png)](/assets/2019-03-30-nginx-vs-cluster.png)

## 다른 블로거의 이야기들

[Node.JS 문서](https://nodejs.org/api/cluster.html#cluster_how_it_works)

> 두 번째 접근 방법인 Node 클러스터는 이론상 최고의 성능을 제공해야 합니다. 그러나 실제로는 운영 체제의 스케줄러 변경으로 인해 배포가 매우 불균형해지는 경향이 있습니다. 모든 접속의 70% 이상이 총 8개의 프로세스 중 단지 두개의 프로세스에서만 종료되는 부하가 관찰되었습니다.

블로그 [StrongLoop](https://strongloop.com/strongblog/best-practices-for-express-in-production-part-two-performance-and-reliability/)

> Node의 클러스터 모듈을 사용하여 클러스터링이 가능합니다. 이렇게 하면 마스터 프로세스가 작업 프로세스를 생성하고, 들어오는 연결을 작업자에게 배분 할 수 있습니다. 그러나 이 모듈을 직접 사용하지 않고 자동으로 처리해주는 도구들 하나 사용하는 것이 훨씬 낫습니다. 예를 들어 `node-pm` 또는 `cluster-service`와 같은 것을 말이죠.

[클러스터 모듈, iptables, Nginx를 비교](https://medium.com/@fermads/node-js-process-load-balancing-comparing-cluster-iptables-and-nginx-6746aaf38272)

> Node 클러스터는 설정과 구현이 간단하고, 다른 소프트웨어에 의존성 없이 Node 단계에서 사용 할 수 있습니다. 마스터 프로세스는 작업자 프로세스와 거의 비슷하게 작동하지만 다른 솔루션들에 비해 요청률은 조금 떨어진다는 점만 기억하세요.

# 8. 유지 보수를 위한 엔드포인트(endpoint) 만들기

보안 처리 된 API를 통해 메모리의 사용이나 REPL등과 같은 시스템 관련 정보를 제공하는 엔드포인트를 만듭니다. 테스트 도구를 사용하는 것은 굉장히 추천 할 만 일이지만, 코드를 사용해서 유용한 정보를 얻는 등의 작업을 쉽게 수행 할 수 있습니다.

**OR**: 단지 서비스의 상태 확인만을 위해 코드를 작성해서 프로덕션으로 배포 하는 일이 잦아집니다.

## 좀 더 자세히

유지 보수를 위한 엔드포인트는 코드로 동작하는 일반적인 보안 HTTP API입니다. 이 엔드포인트는 운영팀이 여러 유용한 기능을 호출하거나 정보를 확인 하는 용도로 사용됩니다. 예를 들어 프로세스의 헤드 덤프(메모리 스냅샷)를 얻거나 메모리 누수가 있는지 보고하고 REPL 명령을 직접 실행할 수도 있습니다. 이 엔드포인트는 기존의 devop 도구(모니터링 제품, 로그 등)가 특정 유형의 정보는 수집하지 못한다거나 이러한 도구를 구매, 설치하지 않기로 한 경우에 필요합니다. 가장 좋은 방법은 프로덕션을 모니터링하고 유지 관리하는데 필요한 전문적인 외부 도구를 사용하는 것입니다. 보통은 이 방법이 보다 강력하고 정확합니다. 하지만 전문적이지 않은 일반 도구는 Node 또는 여러분의 앱이 얻고자 하는 정보를 제공하지 못 할 수 있습니다. 예를 들어, GC가 컴파일 싸이클을 완료 한 순간의 메모리 스냅샷이 필요한 경우가 있을 수 있습니다. 몇몇 NPM 라이브러리들은 이런 요구를 아주 잘 수행하지만, 일반적인 모니터링 도구들은 이런 기능이 없을 가능성이 큽니다.

### 코드 예제: 코드를 통해 헤드 덤프 생성

```javascript
var heapdump = require('heapdump');
 
router.get('/ops/headump', (req, res, next) => {
  logger.info(`About to generate headump`);
  heapdump.writeSnapshot(function (err, filename) {
    console.log('headump file is ready to be sent to the caller', filename);
    fs.readFile(filename, "utf-8", function (err, data) {
      res.end(data);
    });
  });
});
```

## 볼거리 추천

[Node.js 앱 프로덕션 준비하기](https://www.youtube.com/watch?v=lUsNne-_VIk)

# 9. APM 제품을 사용하여 오류 및 다운타임 발견하기

제품의 성능을 모니터링하는 제품(APM이라고도 함)은 기존의 한계를 뛰어넘어 제품의 코드나 API를 사전에 측정하고 서비스 전반에서 사용자의 경험을 측정합니다. 예를 들어 일부 APM 제품은 최종 사용자 측면에서 너무 느리게 로드되는 트랜잭션을 짚어주며 근본적인 원인을 제안 할 수 있습니다.

**OR**: API 성능, 가동 중지 시간을 측정하는 데 많은 노력을 기울여야 합니다. 실제 시나리오에서 가장 느린 코드가 어디있는지 찾을 수 없고, 이것이 UX에 미치는 영향 또한 알지 못할 것입니다

## 좀 더 자세히

APM, 애플리케이션 프로그램 성능 모니터링은 애플리케이션의 성능을 고객의 관점에서, End to End 모니터링하는 것을 목표로 합니다. 전통적인 모니터링 솔루션은 예외나 stand-alone의 기술적 지표(예: 오류 추적, 서버 엔드 포인트 속도 저하 등)에 중점을 두고 있지만 실제로는 일부 미들웨어 서비스가 느린 경우가 발생하면 코드의 예외 없이도 얼마든지 사용자를 실망시킬 수 있습니다. APM 제품은 말단에서 말단까지(End to End) 유저 경험을 측정합니다. 예를 들자면, 프론트 엔드 UI에서부터 다중 분산 서비스를 포함하는 거대한 시스템까지를 측정 할 수 있습니다. 일부 APM 제품은 여러 계층에 걸쳐진 트랜잭션이 얼마나 빠른지 파악 할 수 있습니다. 이로써 사용자 경험에 문제가 없는지 지적 할 수 있습니다. 이런 제품은 매력적이지만 상대적으로 가격이 비싸기 때문에, 선형적인 모니터링을 넘어서야 하는 대규모의 복잡한 제품에 권장됩니다.

### APM 예제 - 교차 서비스 앱 성능을 시각화 한 상용 제품

[![](/assets/2019-03-30-cross-service-performance-1024x537.png)](/assets/2019-03-30-cross-service-performance-1024x537.png)

### APM 예제 - 사용자 경험 점수를 보여주는 상용 제품

[![](/assets/2019-03-30-end-to-end-transactions-time.png)](/assets/2019-03-30-end-to-end-transactions-time.png)

### APM 예제 - 느린 코드 경로를 보여주는 상용 제품

[![](/assets/2019-03-30-slow-code-1024x633.png)](/assets/2019-03-30-slow-code-1024x633.png)

# 10. 코드를 출시 가능한 상태로 만들기

출시 이후의 모습을 머릿속에 그리며 코드를 작성하고, 첫날부터 배포 계획을 세웁니다. 약간 모호하게 들릴 것 같아서, 유지 보수와 밀접하게 관련있는 개발 팁 몇개를 정리했습니다.

**OR**: IT/devops 세계 챔피언도 잘못 쓰여진 시스템을 구할 수 없을 것입니다.

## 좀 더 자세히

다음은 유지 보수 및 안정성에 크게 영향을 주는 개발 팁 목록입니다.

- 12가지 주요 가이드 - [12가지 주요 가이드](https://12factor.net/)와 친숙해지세요.
- 무상태(stateless) 유지: 특정 웹 서버에 로컬 데이터를 저장하지 않습니다.
- 캐시: 캐시를 적극적으로 써야 하지만, 캐시 불일치로 오히려 문제를 일으키지는 마세요.
- 테스트 메모리: 'memwatch'와 같은 도구로 개발 과정에서 메모리의 사용과 누출을 측정합니다.
- 이름 있는 함수: 일반적으로 메모리 프로파일러가 메소드 이름 당 메모리 사용량을 제공하기 때문에 익명 함수(즉, 인라인 콜백)의 사용을 최소화하세요.
- CI 도구 사용: CI 도구를 사용하여 릴리즈 전에 오류를 찾아냅니다. 예를 들어, 참조(reference) 오류나 정의되지 않은(undefined) 변수를 탐지하려면 ESLint를 사용하세요. 동기(synchronous)식 API 코드를 탐지하려면 [-trace-sync-io](https://nodejs.org/api/cli.html#cli_trace_sync_io) 옵션을 사용하십시오.
- 똑똑한 로그 작성: 가급적이면 각 로그 문장에 JSON 형식의 문맥(context) 정보를 포함시켜서, Elastic과 같은 로그 수집 도구가 검색 할 수 있도록 하세요. 또한 각각의 요청이 동일한 트랜잭션이라는 것을 알아챌 수 있도록 로그에 `transaction-id`를 포함시키세요.
- 오류 관리: 오류 처리는 Node.JS 사이트의 아킬레스건입니다. 많은 Node 프로세스가 사소한 오류로 크래시가 나는 반면, 어떤 Node 프로세스는 오류가 발생하더라도 크래시가 나지 않고, 오류 상태로 정지합니다. 오류를 어떻게 처리할지 전략을 설정하는 것은 절대적으로 중요합니다. [오류 처리 Best Practice](https://goldbergyoni.com/checklist-best-practices-of-node-js-error-handling/)에서 보다 많은 정보를 확인하세요.

# 11. 보안 체크 리스트를 확실하게 확인하기

Node는 자체적으로 몇 가지 고유한 보안 문제를 안고 있습니다. 이 글에서는 직접적인 보안 사항들을 그룹으로 묶어 보았습니다. "보안이 잘 된" 시스템은 훨씬 더 광범위한 보안 분석이 필요하다는 사실은 굳이 이야기하지 않겠습니다.

**OR**: 언론에 보도되는 보안 누출 이슈보다 더 중요한 것이 무엇일까요? 여러분이 금방 잊어버린 쉬운 보안 이슈들입니다.

## 좀 더 자세히

보안은 전문 사육사가 길들여야하는 커다란 코끼리와 비슷합니다. 다음은 대부분의 앱에서 바로 적용 되어야하는 보안 사항들입니다. 물론 이것은 빙산의 일각 일 뿐입니다.

1. VPN: 여러분의 머신에 접근 할 수 있는 개인 네트워크(VPC, VPN)를 생성합니다.
2. TLS: SSL/TLS로 비즈니스 트랜잭션을 보호합니다.
3. Paramterized SQL: 저장 프로시저(Stored procedure) 또는 매개 변수화 된 쿼리를 사용하여 SQL 주입(injection) 공격을 피합니다.
4. 신중한 HTTP 헤더: Express를 사용할 때 기본적으로 전송되는 HTTP 헤더(예:X-XSS-Proection Header)를 사용합니다. 이것은 직접 코딩하기 쉬워 보이지만 실제로는 브라우저마다 특정 헤더를 다른 방식으로 처리하고 헤더 값은 현재 유저 에이전트와 일치해야 하므로 구현이 어렵습니다. 패키지 `npm Helmet`이 이 지루한 작업을 대신 할 수 있습니다. 
5. 안전한 쿠키 사용: `httpOnly`, `path`, `domain`과 같은 쿠키 속성을 설정하여 쿠키 사용을 제한하고 통상적인 공격을 예방하세요. 아래 코드 예제를 참고하십시오.

## 다른 블로거의 이야기들

블로그 [익스프레스 보안 베스트 프랙티스](https://expressjs.com/en/advanced/best-practice-security.html)

> Helmet을 사용하고 싶지 않다면 적어도 `X-Powered-By` 헤더를 비활성화하십시오(기본값은 enabled). 공격자는 이 헤더를 사용하여 Express를 실행하는 응용 프로그램을 감지 한 다음 대상에 대한 공격을 시작합니다. 따라서 `app.disable()` 메소드로 헤더를 해제하는 것이 가장 좋습니다. `app.disable('x-powered-by')`

### 코드 예제: 'Helmet' - 몇 줄의 코드로 다양한 유형의 공격을 차단하는 NPM 패키지

```javascript
//using a single line of code will attach 7 protecting middleware to Express appapp.use(helmet());
//additional configurations can be applied on demand, this one mislead the caller to think we’re using PHP 🙂
app.use(helmet.hidePoweredBy({ setTo: 'PHP 4.2.0' }));//other middleware are not activated by default and requires explicit configuration .
app.use(helmet.referrerPolicy({ policy: 'same-origin' }));
```

### 코드 예제 : 쿠키를 안전하게 설정하기

```javascript
var session = require('cookie-session');
var express = require('express');
var app = express();
var expiryDate = new Date( Date.now() + 60 * 60 * 1000 ); // 1 hour
app.use(session({
      name: 'session',keys: ['key1', 'key2'],cookie: { secure: true,
      httpOnly: true,
      domain: 'example.com',
      path: 'foo/bar',
      expires: expiryDate
    }
  })
);
```

# 12. 메모리 사용을 측정하고 보호하기

Node.js는 메모리에 대한 논란이 있습니다. v8 엔진은 메모리 사용량(1.4GB)에 대한 제한(soft limit)이 있으며, Node 코드에 메모리의 누수가 발생하는 경로가 있다고 알려져 있습니다. 따라서 Node 프로세스 메모리를 관찰하는 것은 필수적입니다. 작은 응용 프로그램에서는 쉘(shell) 명령을 사용하여 주기적으로 메모리를 측정 할 수 있지만 중간 규모나 대형 응용 프로그램에서는 메모리를 감시하기 위한 방안으로 모니터링 시스템을 도입하는 것이 좋습니다.

**OR**: [월마트](https://www.joyent.com/blog/walmart-node-js-memory-leak)에서 발생한 것처럼 하루에도 수백 메가 바이트의 프로세스 메모리가 누출 될 수 있습니다.

## 좀 더 자세히

완벽한 세계에서 웹 개발자는 메모리 누수를 처리 할 필요가 없어야 합니다. 하지만 현실 세계에서는 이미 알려진 Node의 메모리 문제가 있으므로 반드시 기억하고 있어야 합니다. 무엇보다도 메모리 사용을 지속적으로 모니터링 해야 합니다. 소규모 사이트에서는 Linux 명령이나 NPM 도구와 node-inspector 및 memwatch와 같은 라이브러리를 사용하여 수동으로 측정 할 수 있습니다. 이 수동 작업의 주요한 단점은 사람이 적극적으로 모니터링을 해야한다는 것입니다. 중요한 프로덕션 서비스의 경우 누출 발생시 경고로 알려주는 강력한 모니터링 도구(예: AWS CloudWatch, DataDog 또는 유사한 사전 예방 시스템)를 사용하는 것이 절대적으로 중요합니다. 누출을 방지하기 위한 몇 가지 개발 가이드 라인을 제시합니다. 전역 레벨에 데이터를 저장하지 않기, 동적으로 크기가 할당되는 데이터를 이용해 스트림을 사용하기, `let` 및 `const`를 사용하여 변수 범위를 제한하기 등을 기억하세요.

## 다른 블로거의 이야기들

블로그 [Dyntrace](http://apmblog.dynatrace.com/)

> 알고 계시겠지만, Node.js에서 JavaScript는 V8에 의해 네이티브 코드로 컴파일됩니다. 최종 네이티브 데이터 구조는 V8에서 단독으로 관리합니다. 즉, JavaScript에서 메모리를 능동적으로 할당하거나 해제 할 수 없습니다. V8은 가비지 콜렉션이라는 잘 알려진 메커니즘을 사용하여 이 문제를 해결합니다.

블로그 [Dyntrace](http://blog.argteam.com/coding/hardening-node-js-for-production-part-2-using-nginx-to-avoid-node-js-load)

> 이 테스트 방식은 항상 동일한 절차를 수행하지만 결과는 명백하게 다를 수 있습니다. 얼마간의 시간동안 적당한 양의 메모리를 할당시켜 힙 메모리 덤프를 만들고, 덤프들을 비교하여 무엇이 메모리를 증가시키는지 파악하세요.

블로그 [Dyntrace](http://blog.argteam.com/coding/hardening-node-js-for-production-part-2-using-nginx-to-avoid-node-js-load)

> Node.js는 약 1.5GB의 메모리를 사용하려고 합니다. 이 메모리는 적은 메모리를 가진 시스템에서 실행할 때 제약을 줍니다. 특히 가비지 컬렉션이 비용이 많이 드는 작업이므로 충분히 예상 할 수 있는 현상입니다. 그 해결책은 Node.js 프로세스에 추가적으로 매개 변수를 전달하는 것입니다. 

```sh
node –max_old_space_size=400 server.js –production
```

> 가비지 컬렉션이 왜 비용이 많이 들까요? V8 자바스크립트 엔진은 세계를 정지시키고 쓰레기를 수집하는 방식을 이용합니다. 다시 말해, 가바지 컬렉션이 수행되는 동안 프로그램은 실행을 멈춘다는 뜻입니다

# 13. Node에서 프론트엔드 asset 가져 오기

단일 스레드 모델로 인해 많은 정적 파일들을 처리 할 때 Node의 성능이 실제로 저하되기 때문에 전용 미들웨어(nginx, S3, CDN)를 사용하여 프론트엔드의 컨텐츠를 제공해야합니다.

**OR**: 수백개의 html/image/angular/react 파일들을 스트리밍하느라 정작 동적 컨텐츠를 제공하라고 만들어진 Node의 단일 쓰레드는 매우 바빠질 겁니다.

## 좀 더 자세히

고전적인 웹 애플리케이션에서 백엔드는 브라우저에 프론트엔드와 그래픽 리소스들을 제공합니다. 따라서 Node의 세계에서도 클라이언트에 정적 파일들을 제공하기 위해 Express의 정적 미들웨어를 사용하는 것이 일반적인 접근 방식처럼 보입니다. 하지만 Node는 한 번에 여러 파일을 제공하도록 최적화되지 않은 단일 스레드를 사용하기 때문에, 일반적인 웹 애플리케이션처럼 다루면 안됩니다. 대신, 이런 작업을 처리하는데 훨씬 최적화되고 더 높은 처리량을 보여주는 Reverse Proxy나 Cloud Storage, 또는 CDN(예: Nginx, AWS S3, Azure Blob 저장소 등)을 사용하십시오. 예를 들어, nginx와 같은 미들웨어는 파일 시스템과 네트워크를 직접 연결하고 다중 스레드 접근 방식을 구현하여 여러 요청을 처리할 때 발생할 수 있는 간섭을 최소화합니다.

다음 중 하나의 최적 솔루션을 사용 할 수 있습니다.

1. Reverse Proxy: 정적 파일들은 Node의 코드들과 같은 레벨에 위치하며, 정적 파일 폴더에 대한 요청만 Node 앞에 있는 Nginx 같은 Proxy에 의해 처리됩니다. 이 방식을 사용하면 Node 애플리케이션이 정적 파일을 배포하는 책임은 갖지만, 실제로 전달하는 역할은 하지 않습니다. 여러분의 프론트엔드 동료는 cross-origin 요청을 방지 할 수 있으므로, 이 방식을 무척 좋아할 것입니다.
2. Cloud Storage: 정적 파일은 Node 앱 콘텐츠의 일부가 아니며 AWS S3, Azure BlobStorage 또는 이러한 목적을 위해 탄생 한 다른 클라우드 서비스에 업로드됩니다. 이 방식을 사용하면 Node가 정적 파일의 배포를 책임지지 않기 때문에, 각자 다른 팀에서 처리한다 하더라도 Node와 프론트엔드간에 완벽한 디커플링이 되기 때문에 문제가 되지 않습니다.

### 코드 예제: 정적 파일을 제공하기 위한 일반적인 nginx 구성

```nginx
  gzip on;
  #defining gzip compression
  keepalive 64;
}
#defining web server
server {
  listen 80;
  listen 443 ssl;#handling static content
  location ~ ^/(images/|img/|javascript/|js/|css/|stylesheets/|flash/|media/|static/|robots.txt|humans.txt|favicon.ico) {
    root /usr/local/silly_face_society/node/public;
    access_log off;
    expires max;
  }
```

## 다른 블로거의 이야기들
 
블로그 [StrongLoop](https://strongloop.com/strongblog/best-practices-for-express-in-production-part-two-performance-and-reliability/)

> 개발 중에는 [res.sendFile()](http://expressjs.com/4x/api.html#res.sendFile)을 사용하여 정적 파일을 제공 할 수 있습니다. 하지만 이 기능을 프로덕션 환경에서는 수행하지 마십시오. 이 기능은 모든 파일 요청에 대해 파일 시스템을 읽어야 하기 때문에 상당한 지연이 발생하여 앱의 전반적인 성능에 영향을 미칩니다. `res.sendFile()`은 보다 효율적인 `sendfile` 시스템 콜(system call)로 구현되지 않았다는 점에 유의해야 합니다. 대신 Express 애플리케이션 용 파일 제공에 최적화 된 `serve-static` 미들웨어 같은 것들을 사용하십시오. 보다 나은 방법은 정적 파일을 제공하기 위해 Reverse Proxy를 사용하는 것입니다.

# 14. 무상태를 유지하며, 자주 서버를 죽이기

모든 유형의 데이터(예: 사용자 세션, 캐시, 업로드 된 파일)는 외부의 데이터 저장소에 저장되어야 합니다. 서버를 주기적으로 '죽이거나' 명시적으로 무상태를 유지시키는 'serverless' 플랫폼(예: AWS Lambda)을 사용하십시오

**OR**: 특정 서버에서 장애가 발생하면 오류가 발생한 머신만 죽는 대신 서비스가 동작을 멈추게 됩니다. 게다가 특정 서버에 의존적이기 때문에 수평 확장성(sacling-out)은 더욱 어려워 질 것입니다

## 좀 더 자세히

서버에서 어떤 설정이나 데이터가 누락되서 심각한 문제가 발생 한 적이 있습니까? 이는 배포에 포함되지 않은 일부 로컬 자산에 대한 불필요한 종속성 때문일 수 있습니다. 성공적으로 동작하는 여러 제품들은 서버를 불사조처럼 여겨서 아무런 문제 없이 서버를 죽이고 다시 살립니다. 즉, 서버는 잠시 동안 코드를 실행 한 다음 교체되는 하드웨어 조각일 뿐입니다. 이 접근 방법은 (a) 부작용없이 서버를 동적으로 추가 및 제거하여 확장 할 수 있고 (b) 각 서버의 상태를 알아야 할 필요가 없어지므로 유지 관리가 간단해집니다.

### 코드 예제: 안티 패턴
```javascript
//Typical mistake 1: saving uploaded files locally in a server
var multer = require('multer') //express middleware for fetching uploads
var upload = multer({ dest: 'uploads/' })
app.post('/photos/upload', upload.array('photos', 12), function (req, res, next) {
})
//Typical mistake 2: storing authentication sessions (passport) in a local file or memory
var FileStore = require
('session-file-store')(session);
app.use(session({
  store: new FileStore(options),
  secret: 'keyboard cat'
}));//Typical mistake3: storing information on the global object
Global.someCacheLike.result = {somedata}
```

## 다른 블로거의 이야기들

블로그 [Martin Fowler](http://mubaloo.com/best-practices-deploying-node-js-applications)

> 어느 날 저는 운영 자격 테스트를 하는 상상을 해보았습니다. 자격에 대한 평가는 동료와 내가 회사 데이터 센터에 나타나 야구 방망이, 전기톱, 물총으로 중요한 프로덕션 서버를 설정하는 것입니다. 평가는 운영 팀이 모든 응용 프로그램을 다시 실행하는 데 걸리는 시간을 기준으로 합니다. 바보같은 상상 일 수도 있지만 여기에 지혜가 숨어 있습니다. 야구 방망이를 쓰지 않는 대신 정기적으로 서버를 태워버려야 합니다. 그리고 서버는 불사조처럼 정기적으로 재에서 다시 태어나야 합니다.

# 15. 자동으로 취약점을 탐지하는 도구 사용하기

가장 신뢰할 만한 Express와 같은 의존성(dependency)도 시스템을 위험에 빠뜨리는 취약점이 수시로 발견됩니다. 이는 취약점을 지속적으로 확인하고 경고해주는(로컬 혹은 github에서) 커뮤니티나 도구들을 사용하여 쉽게 길들일 수 있으며 일부 취약점은 즉시 패치 할 수도 있습니다. (역주: 여기서 의존성은 의존적인 라이브러리나 프레임워크들을 통칭하고 있습니다)

**OR**: 별다른 도구없이 취약점으로부터 코드를 깨끗하게 유지하려면 새로운 위협에 대한 뉴스를 지속적으로 살펴야 합니다. 이는 매우 지루한 일이 될 것입니다

## 좀 더 자세히
StrongLoop의 블로그의 다음과 같은 이야기를 정말 좋아합니다. "앱의 보안 안정성은 앱의 의존성 중 가장 약한 의존성만큼만 강하다." 코드 의존성은 실제로 가장 유명하고 힘든 테스트를 거친 패키지조차도 종종 취약점을 갖는 경향이 있습니다. 예를 들어, 이전 버전의 Express에서 cross-site scripting 공격에 사용자를 노출시킬 수 있는 위협 요소가 감지되었습니다. 다행히도 nsp나 snyk과 같은 커뮤니티 및 상용 도구(최소한 퍼블릭 리포지토리에는 무료 플랜이 있는)들은 이러한 위협을 자동으로 감지하고, 팀에 경고를 하거나 이후 이러한 취약점을 자동으로 패치 할 수 있습니다

## 다른 블로거들의 이야기

[프로덕션 환경에서의 우수 사례](https://strongloop.com/strongblog/best-practices-for-express-in-production-part-one-security/)

> 응용 프로그램의 종속성을 관리하는 것은 강력하고 편리합니다. 그러나 사용하는 패키지에는 응용 프로그램에 영향을 줄 수 있는 심각한 보안 취약점이 있을 수 있습니다. 앱의 보안은 의존성의 "약한 연결(weekest link)"만큼 강력합니다. 다행히도 사용중인 third-party 패키지를 확인하기 위해 사용할 수있는 유용한 도구가 두 가지 있습니다. ? 와 requireSafe. 이 두 가지 도구는 대체로 똑같은 기능을 수행하므로 둘 다 사용하는 건 과한 일이지만, "미안해하기 전에 안전한게 낫다"는 말처럼 보안을 위해서는 유효한 일일 것입니다. (역주: 두 가지를 소개하려고 한 것 같은데 원글에 한가지가 빠져 있습니다))

### 코드 예제 : 전형적인 nginx 설정

```javascript
//using a single line of code will attach 7 protecting middleware to Express appapp.use(helmet());
//additional configurations can be applied on demand, this one mislead the caller to think we’re using PHP 🙂
app.use(helmet.hidePoweredBy({ setTo: 'PHP 4.2.0' }));//other middleware are not activated by default and requires explicit configuration .
app.use(helmet.referrerPolicy({ policy: 'same-origin' }));
```

# 16. 각 로그 트랜잭션 문에 'TransactionId' 지정하기

하나의 요청을 위한 모든 로그들에 동일한 식별자로 `transaction-id : {some value}`를 기록합니다. 이렇게하면 로그의 오류를 검사 할 때 이전과 이후의 상황을 쉽게 유추 할 수 있습니다. 불행하게도 비동기적인 측면으로 인해 Node에서 이걸 이루어내기는 쉽지 않습니다. 아래의 코드 예제를 확인하세요.

**OR**: 맥락없이(이전에 어떤 일이 있었는지) 프로덕션 오류 로그를 보기 때문에, 문제를 이해하기가 훨씬 어렵고 느리게 만듭니다.

## 좀 더 자세히

일반적으로 로그는 모든 컴포넌트와 요청들의 창고입니다. 의심스러운 라인이나 오류가 감지되면 맥락에 따라 그 플로우(예: 사용자 "John"이 무언가를 사려고 하는 것 같은)에 포함되는 다른 로그 항목들을 함께 살필 수 있어야 합니다. 요청/트랜잭션이 여러 머신에 걸쳐있는 마이크로 서비스와 같은 환경에서는 이것이 더욱 중요하고 도전적인 일이 됩니다. 같은 요청에 대해서 고유한 트랜잭션 ID를 할당하여 이 문제를 해결하십시오. 한 행을 검색 할 때 그 ID를 이용해서 동일한 플로우를 가진 트랜잭션 ID의 모든 로그들을 검색 할 수 있습니다. 그러나 모든 요청을 단일 스레드가 처리하는 Node의 모델로는 이를 달성하는 것이 쉽지 않습니다. 요청 레벨에서 데이터를 그룹화 할 수있는 라이브러리를 사용하십시요. 다음의 코드 예제를 참조하세요. 다른 마이크로 서비스를 호출 할 때, HTTP 헤더 "x-transaction-id"를 이용해 트랜잭션 id를 전달하고, 문맥을 유지 할 수 있습니다.

### 코드 예제: 전형적인 nginx 설정

```javascript
//when receiving a new request, start a new isolated context and set a transaction Id. The following example is using the NPM library continuation-local-storage to isolate requests
var createNamespace = require('continuation-local-storage').createNamespace;
var session = createNamespace('my session');

router.get('/:id', (req, res, next) => {
  session.set('transactionId', 'some unique GUID');
  someService.getById(req.params.id);
  logger.info('Starting now to get something by Id');
}

//Now any other service or components can have access to the contextual, per-request, data
class someService {
  getById(id) {
    logger.info(“Starting now to get something by Id”);
    //other logic comes here
  }
}

//Logger can now append transaction-id to each entry, so that entries from the same request will have the same value
class logger {
  info (message) {
    console.log(`message ${session.get(‘transactionId')}`);
  }
}
```

# 17. NODE_ENV=production 설정하기

환경 변수 `NODE_ENV`를 `production` 또는 `development`으로 설정하여 최적화가 활성화 되어야 하는지 여부를 표시합니다. 많은 npm 패키지들은 현재 환경을 바탕으로 프로덕션을 위해 코드를 최적화 할지 결정합니다.

**OR**: 이 간단한 속성 하나를 생략하면 성능이 크게 저하 될 수 있습니다. 예를 들어 서버 렌더링으로 Express를 사용 할 때, `NODE_ENV`를 생략하면 처리 속도가 느려집니다.

## 좀 더 자세히

프로세스의 환경 변수는 일반적으로 실행중인 프로그램이 설정을 위해 사용하는 키-값 쌍의 모음입니다. 어떤 변수든 사용할 수 있지만, Node는 `NODE_ENV`라는 변수를 사용하여 현재 프로덕션 환경에서 구동중인지 여부를 설정하도록 권장합니다. 이 설정으로 컴포넌트가 캐싱을 사용하지 못하게 하거나 디버그 용 로그를 출력하게 하는 등 개발을 위한 기능들을 제공 할 수 있게 합니다. 요즘의 배포 도구 인 Chef, Puppet, CloudFormation 등은 배포 중에 환경 변수를 설정 할 수 있습니다.

### 코드 예제: NODE_ENV 환경 변수 설정 및 읽기
```javascript
//Using a command line, initializing node process and setting before environment variables
Set NODE_ENV=development&& set otherVariable=someValue&& node

//Reading the environment variable using code
If (process.env.NODE_ENV === “production”) {
  useCaching = true;
}
```

### 다른 블로거들의 이야기

블로그 [ynatrace: 성능에 대해](https://www.dynatrace.com/blog/the-drastic-effects-of-omitting-node_env-in-your-express-js-applications/)

> Node.js에는 `NODE_ENV`라는 변수를 사용하여 현재 모드를 설정하는 규칙이 있습니다. 우리는 실제로 `NODE_ENV`가 설정되어 있지 않으면 'development'로 기본 설정되어 있음을 알 수 있습니다. `NODE_ENV`를 프로덕션 환경으로 설정하면 Node.js가 CPU 사용량이 약간 떨어지는 동안 약 2에서 3배 만큼의 처리량이 뛰는 걸 확인 할 수 있습니다. 다시 한번 강조합니다: `NODE_ENV`를 프로덕션 환경으로 설정하면 응용 프로그램이 3배 빨라집니다!

[![](/assets/2019-03-30-node_env-performance.png)](/assets/2019-03-30-node_env-performance.png)

# 18. 자동, 원자성(atomic) 및 제로 다운 타임 배포 설계하기

팀이 배포를 많이 하면 프로덕트의 심각한 문제를 일으킬 가능성이 낮아진다는 연구가 있습니다. 사람이 직접 하는 위험한 작업이나 서비스 점검 시간이 필요하지 않은 신속하고 자동화 된 배포로 배포 프로세스가 크게 향상됩니다. Docker와 CI 도구를 결합하여 능률적인 배포를 하는 것이 업계의 표준이 되었습니다.

**OR**: 긴 배포 시간 -> 길어진 점검 시간과 사람으로 인한 오류 발생 -> 팀 구성이 불안정하고 배포가 어려움 -> 배포도 적어지고 기능도 적어짐

# 19. 각 배포에서 NPM 버전을 업데이트하기

새 버전이 출시 될 때마다 package.json의 버전 속성을 높여서 프로덕션 환경에서 어떤 버전이 배포되는지 명확하게 알 수 있어야 합니다. 다른 서버가 다른 버전을 사용 할 수 있는 마이크로서비스 환경에서는 더욱 그렇습니다. `npm version` 명령은 자동으로 이를 수행하게 해줍니다.

**OR** : 개발자는 종죵 분산 시스템의 오류를 발견하고 수정하려 하지만, 단지 올바른 버전이 제대로 배포되지 않아서 발생한 오류라는 것을 깨닫게 됩니다.

# 20. Nodejs의 LTS 릴리즈 사용하기

Node.js의 LTS 버전을 사용하여 주요 버그 수정, 보안 업데이트 및 성능 향상을 받아야 합니다.

**OR**: 새로 발견 된 버그나 취약점이 프로덕션 환경에서 악용될 수 있으며 애플리케이션이 다양한 모듈의 지원을 받지 못할 수 있기 때문에 유지 관리가 어려워 집니다.

# 21. 진정한 카오스를 모니터링하기 (원숭이 사용하기)

우리의 인생은 예상치 못한 일이 프로덕션 과정에서도 일어날 수 있다는 것을 알려줍니다. 어떤 이름을 지정만 하면 서버가 크래시가 나고 SSL 인증서 유효성이 취소 될 수도 있으며 이벤트 루프가 블럭되고 DNS 레코드가 변경 될 수 있습니다. 이것은 희귀한 조건처럼 들리지만 종종 일어나는 일이며 대개 폭풍과 같은 크나큰 영향을 줍니다. 이러한 혼란스러운 조건을 시뮬레이션하지 않고 애플리케이션이 살아 남을 수 있는지 혹은 이 장애 상황을 기록으로 남길 수 있는지 미리 알아 보는 것은 매우 힘든 일입니다. [Netflix chaos-monkey](https://github.com/Netflix/chaosmonkey)는 이런 카오스적인 혼돈 상황을 생성하여 시뮬레이션 해볼 수 있는 가장 유명한 도구입니다. 제가 만든 [Nodejs chaos monkey](https://www.npmjs.com/package/chaos-monkey)는 Node와 프로세스의 혼돈에 초점을 맞추고 있습니다.

**OR**: 여기에는 다른 방법이 없습니다. 머피의 법칙은 자비없이 당신의 서비스에 발생 할 것입니다.

# 22. 계속 지켜봐주십시오. 더 많은 내용이 곧 채워질 예정입니다.

post-mortem debugging, libuv thread pool 최적화하기, 프로덕션 스모크 테스트의 작성 등과 같은 다른 Best Practice에 대해 곧 글을 작성하려고 합니다. 업데이트 되었는지 확인을 원하시면 제 [트위터](https://twitter.com/nodepractices/)나 [페이스북](https://www.facebook.com/nodepractices/) 페이지를 팔로우하세요.

