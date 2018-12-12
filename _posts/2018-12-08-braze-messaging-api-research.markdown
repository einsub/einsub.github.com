---
layout: post
title: "Braze의 messaging API"
date: 2018-12-08 08:37:22
author: Reid
categories:
  - engineering
tags:
  - breaze
  - messaging
  - push
published: true
---
# 용어 설명
  - external_user_ids
    - Braze SDK에 설정한 애플리케이션의 ID
    - 캠페인의 경우 segment와 동시에 설정되어 있으면 둘 모두를 포함하는 사용자들이 대상이 됨
  - user_aliases
    - 사용자를 처음 등록 할 때 설정하는 alias를 이용해 발송
  - send_id
    - 발송한 알림을 track하기 위한 id
    - broadcast로 발송하면 직접 입력하지 않아도 자동으로 만들어줌
  - audience
    - 사용자 속성을 기반으로 논리 연산의 조합으로 filter를 거는 기능
    - 속성, 푸시 구독 여부, 이메일 구독 여부, 마지막 사용 시간 등을 기준으로 잡을 수 있음
  - recipient_subscription_state
    - opted_in: 옵트 인 된 유저에게만 발송
    - subscribed: 옵트 인 되었거나 구독 중인 사용자에게만 발송
    - all: 구독하지 않은 유저에게까지 모두에게 발송, transactional 이메일 보내기에 유용

---

# Send Endpoints

## POST messages/send

- 설계된 사용자군에게 임시 메시지를 즉시 발송 가능
- `segment_id`, `external_user_ids`, `audience` 중 하나는 입력해야 함
- `segment_id`는 segment에 등록된 사용자에게 발송
- `external_user_ids`, `user_aliases`는 해당 사용자들에게 직접 발송
- 함께 입력되면 해당 사용자들 중 segment에 등록된 사용자에게 발송

**POST** https://YOUR_REST_API_URL/messages/send
```text
{
  "api_key": API Key,
  "broadcast": external_user_ids나 aliases가 없으면 true로 설정,
  "external_user_ids": 외부 사용자 ID,
  "user_aliases": 사용자 alias,
  "segment_id": Segment ID,
  "audience": Connected Audience,
  "campaign_id": Campaign ID,
  "send_id": Send ID,
  "override_frequency_capping": 캠페인에 대한 frequency_capping 무시,
  "recipient_subscription_state": Push 알림 구독 상태,
  "messages": {
    "apple_push": Apple Push Object,
    "android_push": Android Push Object,
    "windows_phone8_push": Windows Phone 8 Push Object,
    "windows_universal_push": Windows Universal Push Object,
    "kindle_push": Kindle/FireOS Push Object,
    "web_push": Web Push Object,
    "email": Email Object
  }
}
```

## POST campaigns/trigger/send

- API Triggered Delivery를 이용하여 발송, 대시보드에 메시지 내용을 저장해놓고, 메세지를 보낼 시점과 누구에게 보낼지를 지정

**POST** https://YOUR_REST_API_URL/campaigns/trigger/send
```text
{
  "api_key": API Key,
  "campaign_id": Campaign ID,
  "send_id": Send ID,
  "trigger_properties": 요청의 모든 유저들에게 적용 될 개인화된 key-value 쌍,
  "broadcast": recipients가 없으면 true로 설정,
  "audience": Connected Audience,
  // 'audience'는 audience 사용자들에게 발송
  "recipients": 값이 제공되지 않고 broadcast가 false가 아니면 campaign의 대상이 되는 segment 전체에게 발송 [
    {
      // "external_user_id", "user_alias" 둘 중 하나 필수
      "user_alias": 사용자 Alias,
      "external_user_id": 외부 사용자 Id,
      "trigger_properties": 요청의 유저에게 적용 될 개인화된 key-value 쌍으로 위 trigger_properties와 충돌이 나는 경우 이 값으로 override됨
    },
    ...
  ]
}
```

---

# Schedule Endpoints

## POST messages/schedule/create

- 특정 타임존 시각, 각 유저별 타임존 시각, 최적 시각 등으로 보내는 시각 설정 가능
- 캠페인, 캔버스, 정해진 시각에 보내야 하는 메시지등의 스케줄을 생성
- segment가 타겟인 경우, 요청 기록은 Developer Console에 저장됨

**POST** https://YOUR_REST_API_URL/messages/schedule/create
```text
{
  "api_key": API Key,
  // 'segment_id', 'external_user_ids', 'audience' 중 하나는 입력해야 함
  // 'segment_id'는 segment에 등록된 사용자에게 발송
  // 'external_user_ids', 'user_aliases'는 해당 사용자들에게 직접 발송
  // 함께 입력되면 해당 사용자들 중 segment에 등록된 사용자에게 발송
  "broadcast": external_user_ids나 aliases가 없으면 true로 설정,
  "external_user_ids": 외부 사용자 ID,
  "user_aliases": 사용자 alias,
  "segment_id": Segment ID,
  "audience": Connected Audience,
  "campaign_id": Campaign ID,
  "send_id": Send ID,
  "override_messaging_limits": 캠페인에 대한 전역 rate limit 무시,
  "recipient_subscription_state": opted in된 유저에게만 발송 ('opted_in'), 구독했거나 opted in 된 유저에게만 발송 ('subscribed'), 구독하지 않은 유저에게까지 모두에게 발송 ('all'), 마지막 옵션은 transactional 이메일 보내기 등에 유용
  "schedule": {
    "time": 메시지 전송 시각,
    "in_local_time": 각자의 로컬 시각에 보낼 것인가,
    "at_optimal_time": 최적 추천 시각에 보낼 것인가,
  },
  "messages": {
    "apple_push": Apple Push Object,
    "android_push": Android Push Object,
    "windows_push": Windows Phone 8 Push Object,
    "windows8_push": Windows Universal Push Object,
    "kindle_push": Kindle/FireOS Push Object,
    "web_push": Web Push Object,
    "email": Email object,
    "webhook": Webhook object
  }
}
```

## campaigns/trigger/schedule/create

**POST** https://YOUR_REST_API_URL/campaigns/trigger/schedule/create
```text
{
  "api_key": API Key,
  "campaign_id": Campaign ID,
  "send_id": Send ID,
  // 'recipients'는 제공된 사용자 id들 중 캠페인 세그먼트에 포함된 사용자들에게만 발송
  "recipients": Recipient Object 배열,
  // trigger properties와 Recipient Object간에 충돌이 발생하면, Recipient Object가 사용됨
  "audience": Connected Audience,
  // 'audience'는 audience 사용자들에게만 발송
  // 'recipients'와 'audience'가 설정되지 않았고 broadcast가 false가 아니면 메시지는 캠페인의 대상이 되는 모든 segment들에게 메시지를 발송
  "broadcast": recipients가 없으면 true로 설정, 
  "trigger_properties": 요청의 모든 유저들에게 적용 될 개인화된 key-value 쌍,
  "schedule": {
    "time": 메시지 전송 시각,
    "in_local_time": 각자의 로컬 시각에 보낼 것인가,
    "at_optimal_time": 최적 추천 시각에 보낼 것인가,
  }
}
```

## POST messages/schedule/update

**POST** https://YOUR_REST_API_URL/messages/schedule/update
```text
{
  "api_key": API Key,
  "schedule_id": 수정 할 스케줄 ID,
  "schedule": {
    // optional, see create schedule documentation
  },
  "messages": {
    // optional, see create schedule documentation
  }
}
```

## POST campaigns/trigger/schedule/update

**POST** https://YOUR_REST_API_URL/campaigns/trigger/schedule/update
```text
{
  "api_key": API Key,
  "campaign_id": Campaign ID,
  "schedule_id": 수정 할 schedule ID,
  "schedule": {
    // required, see create schedule documentation
  }
}
```

## POST messages/schedule/delete

**POST** https://YOUR_REST_API_URL/messages/schedule/delete
```text
{
  "api_key": API Key,
  "schedule_id": 삭제 할 schedule ID
}
```

## POST campaigns/trigger/schedule/delete

**POST** https://YOUR_REST_API_URL/campaigns/trigger/schedule/delete
```text
{
  "api_key": API Key,
  "campaign_id": Campaign ID,
  "schedule_id": 삭제 할 schedule ID
}
```

---

# 메시지 전송 트래킹을 위한 Send IDs 생성

## POST /sends/id/create

- 메시지를 보낼 때 마다 따로 캠페인을 생성하지 않고도, 메시지를 전송하고 메시지 퍼포먼스를 트랙킹 할 수 있음
- 프로그램 적으로 메시지를 만들어서 전송 할 때 유용
- 주어진 앱 그룹 당 일간 100개까지 제한

**POST** https://YOUR_REST_API_URL/sends/id/create
```text
{
  "api_key": API Key,
  "campaign_id": Campaign ID,
  "send_id": Send DI
}
```

---

# 근접한 스케줄의 캠페인과 캔버스 얻기

```bash
$ curl https://rest.iad-01.braze.com/messages/scheduled_broadcasts?api_key=X&end_time=2017-09-01T00:00:00-04:00
```
