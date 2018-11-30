---
layout: post
title: "event-stream 공격 사건의 전말"
date: 2018-11-30 15:43:12
author: Reid
categories:
  - engineering
tags:
  - javascript
  - 보안
  - 취약점
published: true
---
최근 자바스크립트 업계에 큰 화제를 몰고 온 `event-stream` 공격 사건에 대해 살펴보니 과정이 워낙 드라마틱해서 요약 정리해봅니다. 더 자세한 내용을 알고 싶으신 분은 아래 링크를 참고하세요.

이 글은 [event-stream vulnerability explained](https://schneid.io/blog/event-stream-vulnerability-explained/) 를 참고로 작성되었습니다.

## 시작
- [event-stream](https://github.com/dominictarr/event-stream)은 주간 다운로드가 2백만명에 육박하는 널리 알려진 패키지
- 올 9월에 right9ctrl이라는 사용자가 프로젝트 관리를 맡겠다고 원 제작자에게 요청, 그리고 수락됨
- 9월 9일 [flatmap-stream](https://github.com/hugeglass/flatmap-stream)이라는 라이브러리를 사용하는 코드가 [추가](https://github.com/dominictarr/event-stream/commit/e3163361fed01384c986b9b4c18feb1fc42b8285)됨
- 9월 16일 flatmap-stream을 없애고 [직접 flatmap 구현 코드](https://github.com/dominictarr/event-stream/commit/908fee5c65d4eb02809a84a1ebc3e5df1f935cd1)를 작성
- 외부 라이브러리를 썼다가 성능등의 이슈로 자기가 직접 구현하는 것은 워낙 흔한 일이라서, 여기까지 아주 평범한 오픈 소스 프로젝트의 일상이었음

## 공격
- flatmap-stream은 겉보기에는 이상할게 없는 라이브러리였지만, 단 1명의 기여자밖에 없었고 심지어 다운로드도 전혀 되지 않았음
- minified된 파일에 의심스러운 코드가 포함되어 있음
- [unminified](https://github.com/dominictarr/event-stream/issues/116) 해보니 `./test/data.js`를 require 하고 있는데, AES256으로 암호화된 문자열 배열을 담고 있음
- 이 암호화된 데이터를 푸는 key가 무엇이냐! 그것은 바로 의존성 최상위 패키지의 descrption이었음.
- npm은 스크립트가 부모 패키지에서 구동될 때 `npm_package_description`이라는 환경 변수를 설정
- 즉, 이 의심스러운 코드는 flatmap-stream 패키지를 포함하는 모든 부모 패키지들을 대상으로 그 description을 이용해 암호화된 데이터의 복호화를 시도하고 있었음 
- 대부분의 부모 패키지는 그 description이 올바른 AES256 키가 아니므로 오류가 날 것이므로, 단 하나만을 타겟으로 한 공격으로 유추됨
- `event-stream`과 의존성이 있는 모든 패키지들을 전수조사 해본 결과 [copay-dash](https://www.npmjs.com/package/copay-dash)라는 비트코인 wallet 패키지가 그 대상이었다는 것이 밝혀짐
- 이 패키지의 description은 "A Secure Bitcoin Wallet"이었고, 이걸 키로 암호를 풀자 `test/data.js` 내용이 복호화되었고, 그것은 다름 아닌 소스 코드였음
- 이 코드는 Copay의 프로세스에 스크립트를 injection시켜서 비트코인을 가로채고 있었음

## 발각
- 공격자의 사소한 실수로 인해 발각
- 공격자가 deprecate된 `crypto.createDecipher`함수를 사용하는 바람에 유저들에게 warning 메시지가 띄워진 것

## 요약
- 이 신박한 범죄자는 널리 쓰이는 오픈소스 프로젝트의 관리자로 잠입, 일주일간 비트코인을 훔치는 코드를 배포
- 일주일 후 패키지를 제거했지만, 이미 배포된 버전을 사용하는 소프트웨어들은 비트코인을 실어나르고 있었음
- 특정한 상황, 일주일동안 배포된 버전의 `event-stream`이 포함된 `copay-dash` 패키지를 이용하는 소프트웨어를 사용한 유저만 공격당하고 있었겠지만, npm의 복잡한 의존성과 취약점이 전 세계적으로 공개된 사건.
- 아울러 오픈소스 프로젝트의 신원이 불확실한 기여자가 이렇게 큰 충격을 만들어 낼 수 있다는 점에 주목하며 여러 논쟁거리를 생산하는 중.