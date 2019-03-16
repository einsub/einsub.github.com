---
layout: post
title: "MongoDB Text Index로 발음구분기호가 있는 문자 검색하기"
date: 2019-03-17 04:19:45
author: Reid
categories:
  - engineering
tags:
  - mongodb
  - text search
  - text index
  - diacritic insensitivity
published: true
---

해외를 대상으로 하는 서비스를 만들 때 언어는 가끔 골치 아픈 문제가 될 수 있습니다. 특히 라틴어 계열이지만 영어와는 다른 알파벳을 가진 언어들이 그렇습니다. 그리고 의외로 아시아에는 '쯔꾸옥응으'라는 알파벳 비슷하게 생긴 문자를 사용하는 베트남이 있습니다. 먼저 베트남어가 어떻게 생겼는지 한번 살펴봅시다.

베트남어는 모양도 독특하지만 발음도 매우 독특합니다.<br />
번역> Tiếng Việt có hình dạng độc đáo nhưng rất độc đáo trong cách phát âm.

구글신의 능력을 빌렸기 때문에 뜻이 정확하게 일치하는지 보장은 할 수 없습니다. 여기서 그건 중요한게 아니고, 다만 생긴걸 구경해보자는 취지입니다. 알파벳 위, 아래에 붙은 diacritic, 발음 구분 부호가 현란합니다. 

이렇게 알파벳과 비슷하게 생긴, 라틴어 계열 문자 체계를 가진 나라의 서비스들은 대부분 발음 구별 부호를 제외한 순수 영어 알파벳으로도 검색을 할 수 있습니다. 예를 들면, 'Tieng'으로 검색을 해도 윗 예문이 검색됩니다. 아무래도 베트남어를 사용하지 않는 관광객이나, 편리한 입력을 위해서라도 그래야만 할 것 같습니다.

이렇게 영어 알파벳은 아니지만, 같은 라틴어 계열 문자를 영어 알파벳으로 검색을 하려면 어떻게 해야 할까요? 테스트를 해봅시다.

## 준비 작업

베트남어가 MongoDB에서 검색이 제대로 되는지 테스트하기 위해 준비 데이터를 넣겠습니다. 토막 상식으로 베트남 문자인 '쯔꾸옥응으'의 나무 위키 설명을 베트남어로 집어넣었습니다.

```javascript
// 베트남어를 표기하기 위해 고안된, 로마자를 이용한 문자 체계.
db.vnsentences.insert({ sentence: 'Một hệ thống chữ viết La Mã được thiết kế để thể hiện tiếng Việt.' })
// 원래 베트남어를 표기하기 위한 문자로는 쯔놈이 있었으나 이는 사용하기에 불편했다.
db.vnsentences.insert({ sentence: 'Ban đầu, có một tsunome cho tiếng Việt, nhưng nó bất tiện khi sử dụng.' })
// 16세기 포르투갈 선교사들이 베트남어를 로마자로 적는 첫 시도를 했고
db.vnsentences.insert({ sentence: 'Các nhà truyền giáo Bồ Đào Nha vào thế kỷ 16 đã thực hiện nỗ lực đầu tiên viết tiếng Việt bằng chữ La Mã' })
// 17세기 알렉상드르 드 로드에 의해 쯔꾸옥응으가 고안되었다.
db.vnsentences.insert({ sentence: 'Alexandre de Lód thế kỷ 17 đã được nghĩ ra như một Tsuchikoku.' })
// 쯔꾸옥응으가 고안된 이후에도 베트남에서는 일단 고유문자인 쯔놈을 사용했으나
db.vnsentences.insert({ sentence: 'Ngay cả sau khi phát minh ra Tsuchuoko, tại Việt Nam,' })
// 프랑스 식민 지배 시절 1885년 청불전쟁에서 승리한 프랑스는
db.vnsentences.insert({ sentence: 'Pháp thắng cuộc Chiến tranh Pháp năm 1885 trong thời Pháp thuộc' })
// 베트남에서 프랑스어의 공용화를 원활하게 추진하기 위한 수단으로
db.vnsentences.insert({ sentence: 'Là một phương tiện để tạo điều kiện cho việc phổ biến tiếng Pháp tại Việt Nam' })
// 로드의 표기법인 쯔꾸옥응으를 보급하였다.
db.vnsentences.insert({ sentence: 'Các ký hiệu của con đường đã được lan truyền.' })
```

이제 준비 작업이 끝났습니다. 검색을 시도해봅시다.

## 검색이 될까?

Mongo Shell에서 직접 쿼리를 던져봤습니다.

``` javascript
> db.vnsentences.find({ sentence: /Mot/ })
```

아무것도 찾아내지 못했네요. 이번에는 제대로 된 베트남어 단어를 넣어보겠습니다.

```javascript
> db.vnsentences.find({ sentence: /Một/ })
{ "_id" : ObjectId("5c8d3f55ef308f8ef3a34df4"), "sentence" : "Một hệ thống chữ viết La Mã được thiết kế để thể hiện tiếng Việt." }
```

검색이 잘 됩니다. 일단, 영문자로 검색은 불가능하다는 걸 알았습니다.

## MongoDB Text Index

MongoDB는 문자열 검색을 위해 'Text Index'라는 특별한 인덱스를 지원합니다. 이 인덱스는 DB에 저장된 문서들의 텍스트 필드를 효과적으로 검색하는 다양한 기능을 제공합니다. 정확하게 일치하는 단어만 검색하는게 아니라 토큰화(tokenize)하여 인덱스를 만들기 때문에 유연한 검색을 할 수 있게 됩니다. 또, 가중치를 부여해서 특정 단어에 대한 검색 결과를 상위에 배치시킬 수도 있습니다.

인덱스를 걸고, 간단한 쿼리를 던져보겠습니다.

```javascript
> db.vnsentences.createIndex({ sentence: 'text' })
> db.vnsentences.find({ $text: { $search: 'Ló nghĩ' }})
{ "_id" : ObjectId("5c8d3f55ef308f8ef3a34df7"), "sentence" : "Alexandre de Lód thế kỷ 17 đã được nghĩ ra như một Tsuchikoku." }
```

검색에 사용한 'Ló nghĩ'는 사실 우리가 입력한 예문에 없는 조합입니다. 검색된 문장을 자세히 보시면 'Lód'라는 단어와 'nghĩ'라는 단어가 서로 떨어져 있고, 심지어 'Ló'는 정확한 단어도 아니었습니다. 하지만, Text 인덱스는 훌륭하게 검색을 해냅니다.

## Diacritic Insensitivity, 발음 구분 기호 무시하기

아직 꺼내지 않은 Text 인덱스의 기능은 바로 '발음 구분 기호 무시하기'입니다. 이 인덱스는 기본 기능으로 발음 구분 기호가 없는 영문자 알파벳으로 베트남 문자열을 검색 해낼 수 있습니다. 그러니까 첫 예제로 시도해보았지만 실패했던 'Mot'으로 'Một' 검색하기가 가능해집니다.

```javascript
> db.vnsentences.find({ $text: { $search: 'Mot' }})
{ "_id" : ObjectId("5c8d3f55ef308f8ef3a34df7"), "sentence" : "Alexandre de Lód thế kỷ 17 đã được nghĩ ra như một Tsuchikoku." }
{ "_id" : ObjectId("5c8d3f55ef308f8ef3a34df5"), "sentence" : "Ban đầu, có một tsunome cho tiếng Việt, nhưng nó bất tiện khi sử dụng." }
{ "_id" : ObjectId("5c8d3f55ef308f8ef3a34df4"), "sentence" : "Một hệ thống chữ viết La Mã được thiết kế để thể hiện tiếng Việt." }
{ "_id" : ObjectId("5c8d3f55ef308f8ef3a34dfa"), "sentence" : "Là một phương tiện để tạo điều kiện cho việc phổ biến tiếng Pháp tại Việt Nam" }
```

## 마치며

세상에는 높은 정확도와 다양한 옵션을 지원하는 훌륭한 텍스트 검색 솔루션들이 있습니다. 하지만, 그렇게 높은 수준의 검색이 굳이 필요없는 프로젝트에서는 MongoDB Text 인덱스만으로도 충분히 좋은 효과를 볼 수 있습니다.

하지만 이 훌륭한 Text 인덱스를 사용 할 수 없는 경우가 있습니다. Text 인덱스는 MongoDB의 '특수 인덱스(Special Index)' 입니다. 그리고 MongoDB는 하나의 쿼리에 두 개 이상의 특수 인덱스를 사용 할 수 없다는 제한 사항이 있습니다. 예를 들어, 전에 다루었던 Geospatial 쿼리 중에 거리순 정렬을 지원하는 쿼리들은 2dsphere라는 특수 인덱스를 사용합니다. 따라서 거리 기반 검색을 하면서 Text 인덱스를 사용하는 것을 불가능합니다.

다음에는 이처럼 Text 인덱스의 사용이 불가능한 상황에 유용하게 써먹을 수 있는 또 다른 우회 기법들을 소개하겠습니다.

