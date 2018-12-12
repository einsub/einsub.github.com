---
layout: post
title: "Tookan API 톺아보기"
date: 2018-12-12 10:45:14
author: Reid
categories:
  - engineering
tags:
  - tookan
published: true
---
> 참조 사이트:<br />
https://tookanapi.docs.apiary.io/#

# Task

## 종류
- 픽업 (Pickup Task)
- 배달 (Delivery Task)
- 픽업과 배달 (Pickup & Delivery Task)
- 심부름? (Appointment Task)

## 상태
- **Assigned (0)**: 태스크가 에이전트에게 배정됨
- **Started (1)**: 에이전트가 태스크를 시작함
- **Successful (2)**: 태스크가 성공적으로 완료됨
- **Failed (3)**: 태스크가 실패함
- **InProgress (4)**: 현재 태스크가 수행 중이며, 에이전트가 목표 지점에 근접했음
- **Unassigned (6)**: 태스크가 아직 에이전트에게 할당되지 않음
- **Accepted (7)**: 배정된 에이전트가 태스크를 수락함
- **Decline (8)**: 에이전트가 태스크를 거부함
- **Cancel (9)**: 에이전트가 태스크를 취소함
- **Deleted (10)**: 대시보드에서 태스크를 제거함

## API
- Task 생성
  - csv로부터 생성
  - Tookan의 주문 양식을 이용한 고객의 task 생성
  - multiple task 생성: 한번에 여러 배달 업무를 수행하는 task
- Task 관리
  - 수정 / 삭제
  - 시작 / 취소 / 배정 / 자동배정
  - 상태(status) 변경
- Task 가져오기
  - 검색
  - 상세 정보 가져오기
  - 주문 번호로부터 task 가져오기

---

# Agent
픽업과 배달을 담당

## API
- Agent 관리
  - 생성 / 수정 / 삭제
  - 차단: 배정을 금지 시키는 듯
  - 태그 관리
- Agent 가져오기
  - 프로필 가져오기
  - 스케줄 가져오기
  - 히스토리 가져오기
  - 실시간 위치 가져오기
- Task 배정
- Notification 보내기

---

# Team
에이전트들을 묶는 그룹 단위

## API
- Team 관리
  - 생성 / 수정 / 삭제
- Team 가져오기
  - 상세 정보 가져오기
  - Job과 Agent 정보 가져오기

---

# Manager
Task와 Agent를 다루는 관리자. Team 단위로 권한을 제한 할 수도 있음.

## API
- Manager 관리
  - 생성 / 삭제
- Manager 가져오기
  - 전체 manager 가져오기

---

# Customers
픽업 배달을 요청하는 고객

## API
- Customer 관리
  - 생성 / 수정 / 삭제
- Customer 가져오기
  - 정보 가져오기
  - 프로필 가져오기
  - 전화번호로 검색
  - 이름으로 검색

---

# User
API 사용자로 서비스 전반의 기능들을 관리. 

## API
- User 등록
- User 관리
  - 전화번호 변경
  - 비밀번호 변경
  - 이메일 중복 확인

---

# Merchant
사장님. 대시보드에 직접 로그인해서 agent나 task등을 관리 할 수 있음. Tookan 마켓플레이스에서 **Multi Merchant Marketplace**를 활성화해야 merchant를 생성 할 수 있음.

## API
- Merchant 관리
  - 생성 / 수정 / 삭제
  - 차단
- Merchant 가져오기
  - 검색
  - 상세 정보 가져오기
  - 리포트 가져오기
  - Team 가져오기
- Merchant Task 관리
  - 생성 / 수정 / 삭제
- Agent 관리
  - 가용 Agent 검색
  - Task 할당

---

# Geofence
fleet이 활동하는 구역(reigon)을 정의

## API
- Reigon 관리
  - 생성 / 수정 / 삭제
- Reigon 가져오기
  - 정보 가져오기
  - 상세 정보 가져오기
- 특정 Agent로부터 region을 제거

---

# Mission
여러 workflow로부터 만들어진 task들을 합쳐서 mission을 생성하고 이를 여러 agent에게 할당 가능

## API
- Mission Task 만들기
- Mission 목록 보기
- Mission 삭제

---

# Webhook
Tookan에서 우리 서비스로 정보를 전달. 넘어온 job_id와 job_status로 상태를 체크할 수 있음. 넘어온 요청이 tookan으로부터 넘어온 것인지 검증하기 위해 tookan_shared_secret 파라메터를 비교. task_history=1을 붙여서 요청하면 태스크 히스토리도 함께 보내줌.

## 샘플
```json
{ 
  "user_id": "1",
  "team_id": "56",
  "session_id": "",
  "auto_assignment": "0",
  "timezone": "-330",
  "sign_image": "https://app.tookanapp.com",
  "order_id": "",
  "tookan_shared_secret": "",
  "dispatcher_id": "",
  "completed_by_admin": "0",
  "has_pickup": "1",
  "has_delivery": "1",
  "is_routed": "0",
  "is_active": "1",
  "is_customer_rated": "0",
  "pickup_delivery_relationship": "cd7dd3e12424de76141787af773782c9100",
  "total_distance_travelled": "0",
  "total_time_spent_at_task_till_completion": "15",
  "fleet_id": "3635",
  "fleet_name": "SumeetRana",
  "fleet_email": "JonSnow@example.com",
  "fleet_rating": "0",
  "customer_id": "1074",
  "customer_username": "sesas",
  "customer_email": "",
  "customer_comment": "",
  "customer_phone": "+13334445555",
  "job_id": "236365",
  "job_type": "1",
  "job_state": "Successful",
  "job_status": "2",
  "job_token": "cd7dd3e12424de76141787af773782c9100",
  "job_hash": "c48fcd2646c282ad3576e9b096c1d798",
  "job_address": "114, Sansome St Ste 250, San Francisco, CA, USA",
  "job_pickup_phone": "+917837173739",
  "job_description": "adsfasd",
  "job_latitude": "37.7913917",
  "job_longitude": "-122.4008504",
  "job_pickup_name": "Jon Snow",
  "job_pickup_email": "sumeet@clicklabs.in",
  "job_pickup_address": "CDCL,MadhyaMarg,Chandigarh,India",
  "job_pickup_longitude": "-122.4008504",
  "job_pickup_latitude": "30.7397101",
  "custom_fields": [
    {
      "label": "Customer_Name",
      "value": 1,
      "required": 1,
      "data_type": "Text",
      "app_side": 1,
      "data": "",
      "template_id": "booking_information",
      "item_id": 0,
      "fleet_data": "harry"
    },
    {
      "label": "CustomerImage",
      "value": 1,
      "required": 1,
      "data_type": "Image",
      "app_side": 1,
      "data": "",
      "template_id": "booking_information",
      "item_id": 0,
      "fleet_data": "['https: //tookan.s3.amazonaws.com/task_images/zJOw1452514280545-TOOKAN_11012016_053825.jpg']"
    },
    {
      "label": "Vehicle_selected",
      "value": 1,
      "required": 1,
      "data_type": "Dropdown",
      "app_side": 1,
      "data": "Bike, Van, Truck",
      "template_id": "booking_information",
      "item_id": 0,
      "dropdown": [
          {
              "id": 0,
              "value": "Bike"
          },
          {
              "id": 1,
              "value": " Van"
          },
          {
              "id": 2,
              "value": " Truck"
          }
      ]
    },
    {
      "label": "Date",
      "value": 1,
      "required": 1,
      "data_type": "Date",
      "app_side": 1,
      "data": "",
      "template_id": "booking_information",
      "item_id": 0
    },
    {
      "label": "Amount",
      "value": 1,
      "required": 1,
      "data_type": "Number",
      "app_side": 1,
      "data": "",
      "template_id": "booking_information",
      "item_id": 0
    },
    {
      "label": "Gender",
      "value": 1,
      "required": 1,
      "data_type": "Dropdown",
      "app_side": 0,
      "data": "Male,Female",
      "template_id": "booking_information",
      "item_id": 0,
      "dropdown": [
        {
          "id": 0,
          "value": "Male"
        },
        {
          "id": 1,
          "value": "Female"
        }
      ]
    },
    {
      "label": "Gender",
      "value": 1,
      "required": 1,
      "data_type": "Dropdown",
      "app_side": 1,
      "data": "Male,Female",
      "template_id": "booking_information",
      "item_id": 0,
      "dropdown": [
        {
          "id": 0,
          "value": "Male"
        },
        {
          "id": 1,
          "value": "Female"
        }
      ]
    },
    {
      "label": "ID_Image",
      "value": 1,
      "required": 1,
      "data_type": "Image",
      "app_side": 1,
      "data": "",
      "template_id": "booking_information",
      "item_id": 0,
      "fleet_data": "['https: //tookan.s3.amazonaws.com/task_images/PZyQ1452514382208-TOOKAN_11012016_054006.jpg']"
    }
  ],
  "task_history": [
    {
      "id": 235973,
      "job_id": 85185,
      "fleet_id": 3829,
      "fleet_name": "bobby singh",
      "latitude": "30.7192552",
      "longitude": "76.8102558",
      "type": "state_changed",
      "description": "Status updated from Assigned to Unassigned",
      "creation_datetime": "2016-01-11T09:34:17.000Z"
    },
    {
      "id": 236365,
      "job_id": 85185,
      "fleet_id": 3635,
      "fleet_name": "Rusty B",
      "latitude": "30.7193512",
      "longitude": "76.8102679",
      "type": "state_changed",
      "description": "Accepted at",
      "creation_datetime": "2016-01-11T12:11:02.000Z"
    },
    {
      "id": 236366,
      "job_id": 85185,
      "fleet_id": 3635,
      "fleet_name": "Rusty B",
      "latitude": "30.7193512",
      "longitude": "76.8102679",
      "type": "state_changed",
      "description": "Started at",
      "creation_datetime": "2016-01-11T12:11:04.000Z"
    },
    {
      "id": 236367,
      "job_id": 85185,
      "fleet_id": 3635
      "fleet_name": "Rusty B",
      "latitude": "30.7195829",
      "longitude": "76.8101409",
      "type": "state_changed",
      "description": "Arrived at",
      "creation_datetime": "2016-01-11T12:11:07.000Z"
    },
    {
      "id": 236368,
      "job_id": 85185,
      "fleet_id": 3635,
      "fleet_name": "Rusty B",
      "latitude": "30.7183782",
      "longitude": "76.8096116",
      "type": "custom_field_updated",
      "description": "
        {
          'label':'CustomerImage',
          'value':1,'required':1,
          'data_type':'Image',
          'app_side':1,
          'data':'',
          'template_id':'booking_information',
          'item_id':0,
          'fleet_data':'https: //tookan.s3.amazonaws.com/task_images/zJOw1452514280545-TOOKAN_11012016_053825.jpg'
        }",
      "creation_datetime": "2016-01-11T12:12:46.000Z"
    },
    {
      "id": 236376,
      "job_id": 85185,
      "fleet_id": 3635,
      "fleet_name": "Rusty B",
      "latitude": "30.7183782",
      "longitude": "76.8096116",
      "type": "text_added",
      "description": "vcvvvv",
      "creation_datetime": "2016-01-11T12:13:14.000Z"
    },
    {
      "id": 236377,
      "job_id": 85185,
      "fleet_id": 3635,
      "fleet_name": "Rusty B",
      "latitude": "30.7194513",
      "longitude": "76.8103432",
      "type": "image_added",
      "description": "https://tookan.s3.amazonaws.com/acknowledgement_images/JhTU1452514406350-TOOKAN_11012016_054030.jpg",
      "creation_datetime": "2016-01-11T12:13:27.000Z"
    },
    {
      "id": 236378,
      "job_id": 85185,
      "fleet_id": 3635,
      "fleet_name": "Rusty B",
      "latitude": "30.7194513",
      "longitude": "76.8103432",
      "type": "signature_image_added",
      "description": "https://tookan.s3.amazonaws.com/acknowledgement_images/srA51452514416346-task_signature.png",
      "creation_datetime": "2016-01-11T12:13:36.000Z"
    },
    {
      "id": 236379,
      "job_id": 85185,
      "fleet_id": 3635,
      "fleet_name": "Rusty B",
      "latitude": "30.7194513",
      "longitude": "76.8103432",
      "type": "state_changed",
      "description": "Successful at",
      "creation_datetime": "2016-01-11T12:13:42.000Z"
    }
  ]
}
```