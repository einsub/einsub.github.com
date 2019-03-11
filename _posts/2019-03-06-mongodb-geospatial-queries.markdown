---
layout: post
title: "예제로 배워보는 상황 별 MongoDB 위치 기반 쿼리"
date: 2019-03-06 10:49:01
author: Reid
categories:
  - engineering
tags:
  - mongodb
  - geojson
  - spatial
  - coordinates
published: true
---

몸에 지닌 채 돌아다닐 수 있는 모바일 기기의 장점 덕분에, 요즘의 앱들은 자신의 위치를 기반으로 주변의 장소, 혹은 업소등을 추천하는 기능을 제공하는 경우가 많습니다. 이 글은 MongoDB를 이용해 현재 나의 위치에서 가까운 업소를 찾거나, 해당 업소와의 거리를 구하는 방법을 소개합니다.

가급적이면 서비스에서 활용 가능한 형태의 요구 사항을 가정하고, 이를 구하기 위한 쿼리를 알아보겠습니다. 그보다 먼저, 자주 언급되는 용어들을 정의해봅시다.

## 용어 설명

#### GeoJSON
JSON 형태로 지형 데이터를 정의하는 포맷입니다. 이 포맷에 대한 상세한 명세는 [여기](https://tools.ietf.org/html/rfc7946#section-3.1)에 정의되어 있습니다. [GeoJSON 오브젝트](https://docs.mongodb.com/manual/reference/geojson/)의 기본 문법 형식은 다음과 같습니다:
```plaintext
<field>: { type: <GeoJSON type>, coordinates: <coordinates> }
```
GeoJSON의 `type`에는 `Point`, `LineString`, `Polygon`, `MultiPoint`, `MultiLineString`, `MultiPolygon`, `GeometryCollection`이 있습니다. 정의된 타입들에서 볼 수 있듯이, 좌표점 뿐만이 아니라 선분에서 폴리곤, 여러 도형 집합들까지 다양하게 지원하고 있습니다. 문법 형식에서 `coordinates` 필드는 `type`에 따라 여러 좌표점을 포함 할 수 있습니다.

#### 레거시 좌표(legacy coordinate point)
MongoDB 2.4 이전에 사용되던 지형 데이터 포맷입니다. GeoJSON에 비해 단순한 형태를 갖습니다. 평면 좌표계의 한 점을 표현 할 수 있습니다. 기본 문법 형식은 다음과 같습니다:
```plaintext
<field>: [<longitude>, <latitude>]
```

#### 지형 인덱스 (Geospatial Indexes)
지형 정보에 대한 쿼리를 지원하기 위해 MongoDB는 지형 인덱스를 지원합니다. 인덱스는 **2dsphere**, **2d**의 두 종류입니다. 2dshphere는 지구와 같은 구형태의 지형을 기반으로 계산하는데 사용되고, 2d는 x, y축의 평면 지형을 기반으로 계산하는데 사용됩니다. 다음 형태로 인덱스를 지정 할 수 있습니다.
```javascript
db.collection.createIndex({ <location field>: '2dshpere' })
db.collection.createIndex({ <location field>: '2d' })
```
쿼리 연산자로 무엇을 쓰느냐보다 어떤 종류의 인덱스가 걸려있는가에 따라 계산 방식이 달라집니다. 지구 위에서의 거리 탐색같은 구형 지형 검색은 2d 인덱스를 사용하는 경우, 오차가 발생 할 수 있으므로 2dsphere를 사용하는 것이 좋습니다.

## 샘플 데이터

연습용으로 사용할 데이터를 추가합니다. 제가 제주도에 살고 있는 관계로 저희 집 근처의 아끼는 카페 둘을 넣어두겠습니다. GeoJSON 형태를 연습해보기 위해 사용 할 places 컬렉션에 둘, 레거시 좌표점 형태를 연습해보기 위해 사용 할 legacyplaces 컬렉션에 둘씩 추가합니다. 실제 작업에는 두 방식 중 하나를 선택하면 됩니다.

```javascript
db.places.insert({
  name: '제주커피박물관 Baum',
  location: {
    type: 'Point',
    coordinates: [
      126.8990639, 33.4397954
    ]
  }
})

db.places.insert({
  name: '신산리 마을카페',
  location: {
    type: 'Point',
    coordinates: [
      126.8759347, 33.3765812
    ]
  }
})

db.legacyplaces.insert({
  name: '제주커피박물관 Baum',
  location: [ 126.8990639, 33.4397954 ]
})

db.legacyplaces.insert({
  name: '신산리 마을카페',
  location: [ 126.8759347, 33.3765812 ]
})
```
참고로 Baum은 저희집에서 10km 정도 떨어진 10분 거리에 위치하고 마을카페는 저희 동네인 신산리에 있습니다.

## 내 위치가 특정 영역에 포함되는지 찾는 두 가지 방법

### 첫번째, $geoIntersects

엄밀히 말해서, 지역에 포함되는지 여부를 알려준다기 보다는 주어진 영역과 문서들의 영역에 교집합이 있는지를 찾아주는 쿼리 셀렉터입니다. 하지만, 문서가 GeoJSON 좌표점을 가지고 있는 경우라면 요청하는 GeoJSON 타입에 `Polygon`을 입력해서 해당 좌표의 포함 여부를 알 수 있습니다. 입력값으로 GeoJSON 형태만 허용하며, `$geometry` 표현식을 이용하여 이를 정의합니다.

`$geoIntersects`는 구형 기하학(spherical geometry)을 이용합니다. 지형 인덱스가 필수는 아니지만, 사용할거라면 2dsphere를 선택해야합니다. 이를 이용해 쿼리 성능을 높일 수 있습니다.

```javascript
{
  <location field>: {
    $geoIntersects: {
      $geometry: {
        type: "<GeoJSON 오브젝트 타입>",
        coordinates: [ <coordinates> ]
      }
    }
  }
}
```

#### 예제: 신산리안에 있는 카페를 찾아라.

```javascript
> db.places.find({
  location: {
    $geoIntersects: {
      $geometry: {
        type: 'Polygon',
        coordinates: [
          [[126.86, 33.39], [126.88, 33.39], [126.88, 33.37], [126.86, 33.37], [126.86, 33.39]]
        ]
      }
    }
  }
})

{
  "_id" : ObjectId("5c7d63cfe33988f8dec15f9c"),
  "name" : "신산리 마을카페",
  "location" : {
    "type" : "Point",
    "coordinates" : [ 126.8759347, 33.3765812 ]
  }
}
```

신산리 영역을 나타내는 Polygon으로 `$geoIntersects` 쿼리 셀렉터를 이용해 쿼리를 실행하면 신산리 마을카페만 검색되는 것을 확인 할 수 있습니다.

### 두 번째, $geoWithin

이 쿼리 셀렉터는 확실히 요청하는 영역안에 포함된 문서들을 반환해주는 기능을 합니다. 입력값으로 GeoJSON 형태의 폴리곤 계열 뿐만이 아니라 레거시 좌표점으로 정의된 shape도 허용합니다. shape은 박스, 폴리곤, 원 형태를 지원합니다. GeoJSON 형식을 사용하는 경우, [`$geometry`](https://docs.mongodb.com/manual/reference/operator/query/geometry/) 표현식으로 정의하고 레거시 좌표점을 사용하는 경우, [`$box`](https://docs.mongodb.com/manual/reference/operator/query/box/), [`$polygon`](https://docs.mongodb.com/manual/reference/operator/query/polygon/), [`$center`](https://docs.mongodb.com/manual/reference/operator/query/center/), [`$centerSphere`](https://docs.mongodb.com/manual/reference/operator/query/centerSphere/)으로 정의 할 수 있습니다.

`$geometry`로 GeoJSON 오브젝트를 정의할 때의 문법은 `$geoIntersects`와 동일하며 다음과 같습니다:

```javascript
{
  <location field>: {
    $geoWithin: {
      $geometry: {
        type: "<GeoJSON 오브젝트 타입>",
        coordinates: [ <coordinates> ]
      }
    }
  }
}
```

레거시 좌표점으로 shape을 정의할 때의 문법은 다음과 같습니다:

```javascript
{
  <location field>: {
    $geoWithin: { <shape operator>: <coordinates> }
  }
}
```

`$geoWithin`은 지형 인덱스가 필수는 아니지만, 사용할 것이라면 2d와 2dsphere 둘 모두를 지원하고, 이를 이용해 쿼리 성능을 높일 수 있습니다. 이 쿼리는 `$near`, `$nearSphere`처럼 정렬된 결과를 내려주지는 않는 대신, 더 빠른 속도를 보장합니다.

`$geoWithin`이 받아들이는 Shape 중, `$center`는 원 영역을 표현하는 연산자이며 평면 지형에 특화되어 있습니다. `$center`의 두 번째 파라메터인 radius는 [decimal degrees](http://en.wikipedia.org/wiki/Decimal_degrees) 단위를 사용해야 하는데, 우리에게 필요한 미터나 킬로미터로 변환하기 위해서는 복잡한 [변환 작업](https://en.wikipedia.org/wiki/Decimal_degrees)을 거쳐야 합니다.

반면 `$centerSphere`는 구형 지형을 지원하고, radius는 radian 단위를 이용하므로 어렵지 않게 변환을 할 수 있습니다. 거리를 radian으로 변환하려면 거리에서 지구 전체의 radius 값인 6,378.1km로 나누어줍니다.

`$center`는 평면 기하학을 이용하므로 2d 인덱스밖에 사용 할 수 없지만, `$centerSphere`는 2d와 2dsphere 인덱스 모두를 사용 할 수 있으며, 극지방에서의 오차율을 줄여주려면 2dsphere 인덱스를 이용하는 것이 좋습니다.

#### 예제: 신산리마을회관 근방 5km 내에서 카페를 찾아라.

앞의 예제는 특정 영역에 포함된 카페를 찾는 문제였지만, 이번 문제는 특정한 위치를 기준으로 일정 거리 내의 모든 카페를 찾는 문제입니다. Shape으로 `$centerSphere`를 이용합니다.

```javascript
> db.legacyplaces.find({
  location: {
    $geoWithin: {
      $centerSphere: [[126.876933, 33.381018], 5 / 6378.1]
    }
  }
})

{
  "_id" : ObjectId("5c7dc11ce33988f8dec15f9e"),
  "name" : "신산리 마을카페",
  "location" : [ 126.8759347, 33.3765812 ]
}
```

## 내 위치에서 가까운 곳을 찾는 방법

### $near와 $nearSphere

이전 쿼리 셀렉터들이 영역 안에 포함된 문서들을 반환하는 반면, 이 쿼리 셀렉터들은 영역 안에 포함된 문서들을 가까운 순서대로 정렬해서 반환합니다. GeoJSON을 이용할 때는 2dsphere 인덱스를, 레거시 좌표점을 이용할 때는 2d 인덱스를 사용해야 합니다. `$nearSphere`는 이름 그대로 구형 지형 검색을 할 때 사용합니다.

GeoJSON 포맷을 이용할 때에는 `$geometry` 연산자를 사용해야 하며, 2dsphere 인덱스를 사용 할 수 있습니다.

```javascript
{
  <location field>: {
    $near: {
      $geometry: {
         type: "Point" ,
         coordinates: [ <longitude> , <latitude> ]
      },
      $maxDistance: <distance in meters>,
      $minDistance: <distance in meters>
    }
  }
}
```

```javascript
{
  <location field>: {
    $nearSphere: {
      $geometry: {
        type : "Point",
        coordinates : [ <longitude>, <latitude> ]
      },
      $minDistance: <distance in meters>,
      $maxDistance: <distance in meters>
    }
  }
}
```

하지만, 레거시 좌표점을 이용할 때에는 2d 인덱스를 사용합니다.

```javascript
{
  <location field>: {
    $near: [ <x>, <y> ],
    $maxDistance: <distance in radians>
  }
}
```

```javascript
{
  <location field>: {
    $nearSphere: [ <x>, <y> ],
    $minDistance: <distance in radians>,
    $maxDistance: <distance in radians>
  }
}
```

GeoJSON의 경우 `$minDistance`와 `$maxDistance`를 미터 단위로 설정하여 검색 거리를 조절 할 수 있고, 레거시 좌표점을 이용하는 경우, radian 값으로 설정할 수 있습니다. 단, `$near`는 `$minDistance`의 설정이 불가능합니다.

이 쿼리 셀렉터를 사용 할 때 참고해야 하는 사항 중 하나는 한 쿼리가 특수 인덱스(Special index)를 두 개 이상 가질 수 없다는 점입니다. 그래서 `$text` 쿼리를 이용한 문자열 검색을 동시에 처리 할 수 없습니다.

#### 예제: 성산일출봉에서 12km의 카페들을 가까운 순서대로 찾아라

```javascript
> db.places.find({
  location: {
    $nearSphere: {
      $geometry: {
        type: 'Point',
        coordinates: [ 126.941131, 33.459216 ]
      },
      $minDistance: 1000,
      $maxDistance: 12000
    }
  }
})

{
  "_id" : ObjectId("5c7d63cfe33988f8dec15f9b"),
  "name" : "제주커피박물관 Baum",
  "location" : {
    "type" : "Point",
    "coordinates" : [ 126.8990639, 33.4397954 ]
  }
}
{
  "_id" : ObjectId("5c7d63cfe33988f8dec15f9c"),
  "name" : "신산리 마을카페",
  "location" : {
    "type" : "Point",
    "coordinates" : [ 126.8759347, 33.3765812 ]
  }
}
```

## 가까운 곳을 찾고 거리까지 함께 구하는 방법

### $geoNear aggregation stage

지금까지는 `find`와 같은 단순 쿼리에 사용하는 연산자들이었다면, 이번에는 aggregation 쿼리에 들어가는 stage입니다. aggregation 쿼리에 포함이 가능하므로 MongoDB aggregation 파이프라인의 장점을 최대한 이용 할 수 있습니다.

이 stage는 반환하는 문서 개수를 제한하거나 추가 검색 쿼리를 정의하는 등 다양한 옵션 설정이 가능합니다.

```javascript
{ $geoNear: { <geoNear options> } }
```

설정 가능한 옵션은 다음과 같습니다:

#### Options

- **spherical**: 계산 방식을 지정합니다. `true`인 경우, `$nearSphere` 방식으로 구형 지형을 계산하고, `false`인 경우 `$near` 방식으로 계산하며, 구형 지형은 2dsphere 인덱스를, 평면 지형은 2d 인덱스를 사용합니다.
- limit: (옵션) 가져 올 문서의 최대 개수입니다. 기본값은 100개입니다.
- num: (옵션) limit과 같은 기능이며, 둘 모두 지정되면 num의 값이 우선합니다.
- maxDistance: (옵션) 검색 최대 거리를 지정합니다. GeoJSON 포맷의 경우 미터 단위로 지정하고, 레거시 좌표점의 경우 radian으로 지정합니다.
- query: (옵션) 문서들을 필터링합니다.
- distanceMultiplier: (옵션) 구한 거리 결과에 추가 연산을 실시합니다. 얻은 결과를 추가로 변환해줄 때 유용합니다.
- uniqueDocs: (옵션) 2.6에서 deprecated된 옵션입니다.
- **near**: 기준 좌표점입니다. 2dsphere 인덱스라면 GeoJSON 형식으로, 2d 인덱스라면 레거시 좌표점 형식으로 입력합니다.
- **distanceField**: 거리를 출력 할 필드명입니다.
- includeLocs: (옵션) 거리 계산에 사용된 좌표점을 출력 할 필드입니다. 한 문서가 여러 좌표점을 가지고 있는 경우 유용하게 사용 될 수 있습니다.
- minDistance: (옵션) 검색 최소 거리입니다. 설정되었다면 이 거리 이내의 문서들은 검색도 되지 않습니다.
- key: (옵션) 인덱스가 설정된 필드를 지정합니다. 한 컬렉션의 여러 필드에 인덱스가 설정되어 있다면 반드시 지정해주어야 합니다.

이 `$geoNear` stage는 aggregation 파이프라인의 가장 첫번째로 기술되어야 합니다. MongoDB의 특수 인덱스를 사용해야 하기 때문인데, 이런 이유로 다른 특수 인덱스, 예를 들어 text 인덱스와 함께 사용 될 수 없습니다.

#### 예제: 신산리마을회관에서 10km 이내의 카페들을 가까운 순으로 검색하고 각각의 거리를 구하라

```javascript
> db.places.aggregate([
  {
    $geoNear: {
      spherical: true,
      limit: 10,
      maxDistance: 10000,
      near: {
        type: 'Point',
        coordinates: [126.876933, 33.381018]
      },
      distanceField: 'distance',
      key: 'location'
    }
  }
])

{
  "_id" : ObjectId("5c7d63cfe33988f8dec15f9c"),
  "name" : "신산리 마을카페",
  "location" : {
    "type" : "Point",
    "coordinates" : [ 126.8759347, 33.3765812 ]
  },
  "distance" : 502.5418507012182
}
{
  "_id" : ObjectId("5c7d63cfe33988f8dec15f9b"),
  "name" : "제주커피박물관 Baum",
  "location" : {
    "type" : "Point",
    "coordinates" : [ 126.8990639, 33.4397954 ]
  },
  "distance" : 6858.597026847846
}
```

지금까지의 쿼리들과는 달리 출력된 결과 문서들에 각각 `distance` 항목이 추가되어 있는 것을 볼 수 있습니다. 신산리 마을카페는 약 502m, Baum은 약 6.8km 떨어져 있다고 계산이 되었습니다. 측정해보진 않았지만, 체감상 맞는 것 같네요.

## 나가며

위치 기반 업소 탐색과 거리를 측정하는 등의 작업이 필요한 프로젝트를 진행하면서 잘못된 이해로 몇 번의 시행착오를 겪은 후,  [MongoDB 매뉴얼](https://docs.mongodb.com/manual/tutorial/calculate-distances-using-spherical-geometry-with-2d-geospatial-indexes/)의 딱딱하고 기술적인 문서 말고 필요한 상황에 알맞은 쿼리를 제안해주는 글이 있으면 좋겠다고 생각했습니다. 이 글이 조금이라도 도움이 되셨길 바랍니다.
