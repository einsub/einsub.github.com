---
layout: post
title: "Google, Facebook 등 대형 서비스들의 REST API 에러 처리 비교"
date: 2019-03-14 01:30:24
author: Reid
categories:
  - engineering
tags:
  - rest api
  - error handling
published: true
---

대형 서비스 제공자들의 에러 처리 방식을 분석하면서, 각 서비스들의 에러 형식, 특징 등을 정리해보았습니다.

## TL;DR

**분석한 서비스 목록**
> Google, Twitter, Spotify, AWS S3, Facebook, Github, Naver

**복수 에러 지원 (3/7)**
> Google, Twitter, Github

**오류 발생 위치 전달 (4/7)**
> Google, Twitter, AWS S3, Github

**HTTP 상태 코드 미사용 (1/7)**
> Facebook

**전통적인 code, message 에러 사용 (2/7)**
> Spotify, Naver

**사용자에게 전달할 메시지 포함 (1/7)**
> Facebook

## Google

https://developers.google.com/search-ads/v2/standard-error-responses

### 형식

```json
{
 "error": {
  "errors": [
   {
    "domain": "global",
    "reason": "invalidParameter",
    "message": "Invalid string value: 'asdf'. Allowed values: [mostpopular]",
    "locationType": "parameter",
    "location": "chart"
   }
  ],
  "code": 400,
  "message": "Invalid string value: 'asdf'. Allowed values: [mostpopular]"
 }
}
```

### 특징

- 복수 에러 지원
- `locationType`과 `location`으로 오류가 발생한 상세 정보를 전달
- 오류 코드:
  - 1차: HTTP 상태 코드
  - 2차: `reason` camelCase 문자열 속성

## Twitter

https://developer.twitter.com/en/docs/ads/general/guides/response-codes.html

### 형식

```json
{
  "errors": [
    {
      "parameter": "start_time",
      "details": "invalid date",
      "code": "INVALID_PARAMETER",
      "value": "",
      "message": "Expected time, got \"\" for start_time"
    }
  ],
  "request": {
    "params": {
      "account_id": "hkk5"
    }
  }
}
```

### 특징

- 복수 에러 지원
- `parameter`와 `value`로 잘못 입력된 파라메터와 그 값 정보를 전달
- 요청 한 계정 정보를 전달
- 오류 코드:
  - 1차: HTTP 상태 코드
  - 2차: `code` SNAKE_CASE 문자열 속성

## Spotify

https://developer.spotify.com/documentation/web-api/

### 형식

```json
{
    "error": {
        "status": 400,
        "message": "invalid id"
    }
}
```

### 특징

- 단일 에러 지원
- 극도로 심플
- HTTP 상태 코드가 헤더로 넘어오는데 왜 또 보낼까.
- 오류 코드
  - HTTP 상태 코드

## AWS S3

https://docs.aws.amazon.com/AmazonS3/latest/API/ErrorResponses.html

### 형식

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Error>
  <Code>NoSuchKey</Code>
  <Message>The resource you requested does not exist</Message>
  <Resource>/mybucket/myfoto.jpg</Resource> 
  <RequestId>4442587FB7D0A2F9</RequestId>
</Error>
```

### 특징

- 단일 에러 지원
- JSON이 아니라 XML을 사용
- `Resource`로 오류가 발생한 리소스 정보를 전
- 오류 코드
  - 1차: HTTP 상태 코드
  - 2차: `Code` PascalCase 문자열 속성

## Facebook

https://developers.facebook.com/docs/graph-api/using-graph-api/error-handling/

### 형식

```json
{
  "error": {
    "message": "Message describing the error", 
    "type": "OAuthException", 
    "code": 190,
    "error_subcode": 460,
    "error_user_title": "A title",
    "error_user_msg": "A message",
    "fbtrace_id": "EJplcsCHuLu"
  }
}
```

### 특징

- 단일 에러 지원
- HTTP 상태 코드를 사용하지 않음
- 유저에게 보여줄 에러 메시지의 타이틀과 본문을 함께 전달, GraphQL 때문인가.
- 오류 코드
  - 1차: 숫자
  - 2차: 숫자

## Github

https://developer.github.com/v3/#client-errors

### 형식

```json
{
  "message": "Validation Failed",
  "errors": [
    {
      "resource": "Issue",
      "field": "title",
      "code": "missing_field"
    }
  ]
}
```

### 특징
- 복수 에러 지원
- Twitter와 달리, 메시지는 에러 각각이 아니라 대표로 하나만 전달
- `resource`와 `field`로 오류가 발생한 위치를 전달
- 에러 코드
  - 1차: HTTP 상태 코드
  - 2차: `code` snake_case 문자열 속성


## Naver

https://naver.github.io/naver-openapi-guide/errorcode.html

### 형식

```xml
<?xml version="1.0" encoding="UTF-8"?>
 <result>
 	<errorMessage><![CDATA[Authentication failed (인증 실패하였습니다.)]]></errorMessage>
 	<errorCode><![CDATA[024]]></errorCode>
 </result>
```

```json
{
 	"errorMessage": "Authentication failed (인증에 실패하였습니다.)",
 	"errorCode": "024"
}
```

### 특징

- 단일 에러 지원
- 특정 서비스는 XML 에러 형태가 아직 남아있음
- 극도로 심플
- 오류 코드
  - 1차: HTTP 상태 코드
  - 2차: 숫자처럼 생긴 문자열

