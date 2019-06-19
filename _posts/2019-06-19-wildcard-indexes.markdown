---
layout: post
title: "MongoDB 4.2의 새기능: 와일드카드 인덱스(Wildcard Indexes)"
date: 2019-06-19 23:28:15
author: Reid
categories:
  - engineering
tags:
  - mongodb
  - indexing
  - 몽고DB
  - 인덱스
published: true
---

# 기존의 인덱스

MongoDB 4.2에 새로 추가되는 와일드카드 인덱스에 대해 알아봅시다. MongoDB에서 인덱스를 만드는 과정을 떠올려보면, 자주 검색에 활용되거나 부하가 클 걸로 예상되는 필드를 찾아서 걸어줘야 합니다. 만약 인덱스가 제대로 성능을 발휘하지 못하면 `explain()`으로 점검하고 `hint()`로 인덱스를 걸어보면서 가장 좋은 조합을 찾아봅니다. 시간이 좀 걸리기는 하지만, 검색 조건이 명확하고 부하 예측이 어느 정도 된다면 인덱스를 만들어 줄 필드를 찾아내는 것은 그렇게 어려운 일은 아닙니다. 하지만 다음과 같은 경우는 어떤가요?

```json
{
   "type":"book",
   "title":"The Red Book",
   "attributes": {
       "color":"red",
       "size":"large",
       "inside": {
           "bookmark":1,
           "postitnote":2
       },
       "outside": {
           "dustcover": "worn"
       }
   }
}
{
   "type":"book",
   "title":"The Blue Book",
   "attributes": {
       "color":"blue",
       "size":"small",
       "inside": {
           "map":1
       },
       "outside": {
           "librarystamp": "Local Library"
       }
   }
}
{
   "type":"book",
   "title":"The Green Book",
   "attributes": {
       "color":"green",
       "size":"small",
       "inside": {
           "map":1,
           "bookmark":2
       },
       "outside": {
           "librarystamp": "Faraway Library",
           "dustcover": "good"
       }
   }
}
```

우리는 책을 검색하는 기능을 구현하려 합니다. 상세 검색을 할 때에는 `attributes` 하위의 속성들로 책을 찾을 수 있어야 합니다. 예를 들어, `bookmark`가 `2`인 책을 찾고 싶습니다.

```javascript
db.example.find({ "attributes.inside.bookmark": 2} })
```

어떤 상황에서는 `size`가 `large`인 책을 찾고 싶습니다.

```javascript
db.example.find({ "attributes.size": "large"} })
```

이런 경우, `attributes` 하위의 모든 요소들에 인덱스를 만들어줘야 합니다. 이런 상황에서 와일드카드 인덱스가 효과적으로 사용 될 수 있습니다.


# 와일드카드 인덱스


윗 단락에서 예를 들었던 `bookmark` 검색을 MongoDB가 어떻게 처리하는지 살펴보기 위해 `explain()`을 사용해봅시다.

```javascript
> db.example.find({ "attributes.inside.bookmark": 2 }).explain()
{
    "queryPlanner" : {
        "plannerVersion" : 1,
        "namespace" : "test.example",
        "indexFilterSet" : false,
        "parsedQuery" : {
            "attributes.inside.bookmark" : {
                "$eq" : 2
            }
        },
        "queryHash" : "F33E15E9",
        "planCacheKey" : "F33E15E9",
        "winningPlan" : {
            "stage" : "COLLSCAN",
            "filter" : {
                "attributes.inside.bookmark" : {
                    "$eq" : 2
                }
            },
            "direction" : "forward"
        },
        "rejectedPlans" : [ ]
    },
    "ok" : 1
}
```

Query Planner가 선택한 최종 계획은 콜렉션 스캔(COLLSCAN)입니다. 다시 말해, 이 필드를 검색하기 위해 모든 문서를 다 살펴본 것입니다. 와일드카드 인덱스를 만들어 줄 때입니다.

```javascript
> db.example.createIndex({ "attributes.$**": 1 });
{
    "createdCollectionAutomatically" : false,
    "numIndexesBefore" : 1,
    "numIndexesAfter" : 2,
    "ok" : 1
}
```

처음 보는 방식의 인덱스 생성입니다. 필드 하위에 `$**`를 붙여서 이 필드의 하위 모든 문서에 와일드카드 색인을 만드는 것입니다. 이렇게 하면, 내부적으로 해당 필드의 하위 모든 필드에 인덱스를 만듭니다. 하위 필드가 문서라면 그 하위 요소들을 순회하면서 인덱스를 만드는 작업을 반복합니다.

다시 한번 동일한 쿼리를 날려보면 이번에는 콜렉션 스캔(COLLSCAN)이 아니라 인덱스 스캔(IXSCAN)으로 바뀐 것을 확인 할 수 있습니다.

```javascript
> db.example.find({ "attributes.inside.bookmark": 2 }).explain()
{
    "queryPlanner" : {
        "plannerVersion" : 1,
        "namespace" : "test.example",
        "indexFilterSet" : false,
        "parsedQuery" : {
            "attributes.inside.bookmark" : {
                "$eq" : 2
            }
        },
        "queryHash" : "F33E15E9",
        "planCacheKey" : "92EE47A6",
        "winningPlan" : {
            "stage" : "FETCH",
            "inputStage" : {
                "stage" : "IXSCAN",
                "keyPattern" : {
                    "$_path" : 1,
                    "attributes.inside.bookmark" : 1
                },
                "indexName" : "attributes.$**_1",
                "isMultiKey" : false,
                "multiKeyPaths" : {
                    "$_path" : [ ],
                    "attributes.inside.bookmark" : [ ]
                },
                "isUnique" : false,
                "isSparse" : false,
                "isPartial" : false,
                "indexVersion" : 2,
                "direction" : "forward",
                "indexBounds" : {
                    "$_path" : [
                        "[\"attributes.inside.bookmark\", \"attributes.inside.bookmark\"]"
                    ],
                    "attributes.inside.bookmark" : [
                        "[2.0, 2.0]"
                    ]
                }
            }
        },
        "rejectedPlans" : [ ]
    },
    "ok" : 1
}
```

# 마치며

MongoDB와 같은 문서형 데이터베이스는 관계형 데이터베이스와 달리 고정 된 스키마가 필요하지 않습니다. 이로 인해 데이터셋을 유연하게 설계 할 수 있다는 장점을 가지고 있습니다. 하지만, 빠른 검색을 위해 인덱스를 지정하려고 할 때 고정된 스키마가 없다는 점이 난감 할 때가 있습니다. 특히 특정 필드가 단순히 하나의 값이 아닌 구조가 각각 다른 하위 문서처럼 다형(polymorphic)적인 데이터 일 때 더욱 그렇습니다. 와일드카드 인덱스는 이런 문서형 데이터베이스가 가지고 있던 단점을 극복하기 위해 좋은 해결책이 될 것으로 보입니다.

참고: https://www.mongodb.com/blog/post/coming-in-mongodb-42--1-wildcard-indexes
