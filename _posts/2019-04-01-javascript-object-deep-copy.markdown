---
layout: post
title: "JavaScript object의 deep merge 방법 알아보기"
date: 2019-04-01 23:18:15
author: Reid
categories:
  - engineering
tags:
  - javascript
  - deep merge
  - assign
  - es6
  - spread
  - deepmerge
published: true
---

# Swallow Merge

## Object.assign

JavaScript의 두 오브젝트를 병합<sup>merge</sup>하는 방법으로 `Object.assign`을 가장 먼저 떠올릴 수 있습니다. 이 함수는 오브젝트들을 병합 시켜주지만, **DEEP** merge는 아니라는 점을 유의해야 합니다. 어떤 차이가 있는지 알아보겠습니다.

```javascript
const A1 = {
  B: {
    C: 'A1.B.C'
  }
}

const A2 = {
  B: {
    D: 'A2.B.D'
  }
}
```

`Object.assign(A1, A2)`의 결과는 무엇일까요?

```javascript
{
  B: {
    D: 'A2.B.D'
  }
}
```

의도한 결과인가요? 혹시 아래와 같은 결과를 원한건 아니었나요?

```javascript
{
  B: {
    C: 'A1.B.C',
    D: 'A2.B.D'
  }
}
```

`Object.assign`은 일치하는 key가 있으면 그 value를 그냥 바꿔치기 해버립니다. 이 방식을 **swallow merge**라고 합니다. **deep merge**는 일치하는 key가 있더라도 그 자식 요소들이 서로 다른 key를 포함하고 있으면 그 자식들을 함께 병합합니다. 따라서, 깊이가 있는 오브젝트를 병합하려 할 때 무턱대고 `Object.assign`을 사용하면 의도한 결과가 나오지 않을 수 있습니다.

## ES6 Spread 연산자

```javascript
const result = { ...A, ...B }
console.log(result)

{
  C: {
    E: 'C.E'
  }
}
```

ES6의 [spread 연산자](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/Spread_syntax)도 `Object.assign` 함수와 똑같은 결과를 보여줍니다. 결국, deep merge를 하기 위해서는 직접 구현하거나 라이브러리를 가져다 쓰는 방법밖에 없습니다.

> 참고: `Object.assign`은 첫 번째 인자 자체를 변경합니다. 입력되는 원본의 파라메터 변수가 변경되기를 원하지 않는다면 같은 결과를 내지만 입력 파라메터를 변경하지 않는 Spread 연산자를 이용하세요.

# Deep Merge

## Deep merge의 고민거리

deep merge를 써서 해결하면 되지만 한가지 더 고민 할 것이 남아있습니다. deep merge가 하위 레벨의 자식 요소들을 병합시키기는 하지만, 배열을 어떻게 병합시켜야 하는지에 대한 근본적인 문제가 있습니다.

```
const A1 = {
  B: [
    {
      C: 'C1',
      D: 'D'
    }
  ]
}

const A2 = {
  B: [
    {
      C: 'C2',
      E: 'E'
    }
  ]
}
```

위와 같은 두 오브젝트가 있다고 가정해봅시다. 이 둘을 deep merge 하면 어떤 결과가 나와야 할까요? 아마 이 부분에서 의견이 갈릴 수 있습니다. 첫번째 의견은 이렇습니다.

첫 번째:
```
{
  B: [
    {
      C: 'C2',
      D: 'D',
      E: 'E'
    }
  ]
}
```

첫 번째와 두 번째 배열 속 오브젝트가 서로 같은 `C`라는 key를 가지고 있기 때문에 복제되는 오브젝트의 `C2`라는 값이 원본 오브젝트의 `C`를 덮어씌웠습니다. 하지만, 또 이런 의견이 있을 수도 있습니다.

두 번째:
```
{
  B: [
    {
      C: 'C1',
      D: 'D'
    },
    {
      C: 'C2',
      E: 'E'
    }
  ]
}
```

`B` 배열의 각각의 값이 별개로 병합 되기를 바랄 뿐, 그 배열 값 자체가 또 병합되는 것을 원하지 않을 수 있습니다. 이런 점을 잘 고민하지 않으면 deep merge를 구현하거나 라이브러리를 가져다 쓸 때 원하는 결과를 얻지 못 할 수 있습니다. 아래에 소개하는 deep merge 구현 방법들에서는 위 예제의 결과가 어떻게 출력되는지도 함께 소개하겠습니다.

## 직접 구현하기

인터넷에서 쉽게 구할 수 있는 좋은 예제가 있어서 가져왔습니다.

https://gist.github.com/ahtcx/0cd94e62691f539160b32ecda18af3d6

```javascript
// Merge a `source` object to a `target` recursively
const merge = (target, source) => {
  // Iterate through `source` properties and if an `Object` set property to merge of `target` and `source` properties
  for (let key of Object.keys(source)) {
    if (source[key] instanceof Object) Object.assign(source[key], merge(target[key], source[key]))
  }

  // Join `target` and modified `source`
  Object.assign(target || {}, source)
  return target
}
```

첫 번째 방식의 결과가 나왔습니다.

```javascript
merge(A1, A2)

{
  B: [
    {
      C: 'C2',
      E: 'E',
      D: 'D'
    }
  ]
}
```

## Lodash 이용하기

자바스크립트 유틸리티 라이브러리인 lodash에는 [merge](https://lodash.com/docs/#merge)라는 이름으로 deep merge 함수를 제공하고 있습니다.

```javascript
import * as _ from 'lodash'

_.merge(A1, A2)

{
  B: [
    {
      C: 'C2',
      D: 'D',
      E: 'E'
    }
  ]
}
```

첫 번째 방식의 결과가 나왔습니다. 위의 직접 구현한 방식과 동일한 결과가 나왔지만 요소의 순서가 바뀐 것이 재미있습니다.

## deepmerge 이용하기

[deepmerge](https://www.npmjs.com/package/deepmerge)는 NPM의 유명한 deep merge 라이브러리입니다.

```javascript
import deepmerge from 'deepmerge'

deepmerge(A1, A2)

{
  B: [
    {
      C: 'C1',
      D: 'D'
    },
    {
      C: 'C2',
      E: 'E'
    }
  ]
}
```

재미있게도 이번에는 두번째 방식의 결과가 나타났습니다. 

이처럼 각 deep merge 구현 방식의 차이를 잘 이해하고, 원하는 방식으로 동작하는지 충분히 테스트를 한 후에 프로젝트에 적용하시기 바랍니다.

> 참고: 직접 구현 방식과 lodash를 이용한 구현은 `Object.assign`처럼 첫 번째 목표 오브젝트의 값을 변경시킵니다. 반면 deepmerge는 원본을 변경시키지 않습니다.

## 내가 원하는 방식으로 deep merge 하기

사실 제가 이 글을 쓰게 된 이유는 제가 원하는 방식의 deep merge가 필요했기 때문입니다. 원본에 이미 존재하는 key는 그대로 둔 채 복사해오는 오브젝트에만 존재하는 key들을 복사해서 새로운 객체를 만들고 싶었습니다.

```javascript
const A1 = {
  B: 'B1',
  C: {
    D: 'D1'
  }
}

const A2 = {
  B: 'B2',
  C: {
    D: 'D2',
    E: 'E2'
  }
}
```

이 둘의 deep merge 결과는 위에서 소개한 모든 deep merge 라이브러리에서 다음과 같이 나옵니다.

```javascript
{
  B: 'B2',
  C: {
    D: 'D2',
    E: 'E2'
  }
}
```

하지만 제가 원하는 것은 아래처럼, 원본을 손상시키지 말고 새로운 요소들만 추가하는 것입니다.

```javascript
{
  B: 'B1',
  C: {
    D: 'D1',
    E: 'E2'
  }
}
```

lodash의 `merge` 함수와 deepmerge 라이브러리의 함수는 모두 병합 로직을 커스터마이즈 할 수 있는 기능을 제공합니다. 하지만, 이 커스터마이즈 방식에도 차이가 있습니다.

deepmerge는 3번째 인자에 전달하는 커스터마이즈 함수로 오브젝트의 key들이 넘어오지만 `object`와 `array`를 담은 key만 넘어오고 `string`, `boolean`, `number`와 같은 primitive 데이터들은 넘어오지 않습니다. 그러므로 커스터마이즈  대상이 한정적이어서 제가 의도한 방식은 구현이 어렵습니다.

```javascript
deepmerge(A1, A2, { customMerge: (key) => {
  // array, object를 value로 가진 key만 커스터마이즈 됩니다.
  return (a1, a2) => {
    return a1 || a2
  })
}
```

반면 lodash의 `mergeWith` 함수를 이용하면 모든 key들을 순회 할 수 있기 때문에 제가 원하는 방식의 커스터마이즈가 가능합니다. 아래의 커스터마이즈 함수는 오리지널 오브젝트에 값이 있으면 그 값을 이용하고, 없으면 두번째 인자인 대상 오브젝트의 값을 이용합니다. 하지만 주의 사항이 있는데, `a1`과 `a2`의 둘 모두가 오브젝트인 경우, 원래의 알고리즘이라면 그 내부까지 순회하며 병합을 해주겠지만, 커스터마이즈 함수에서 값을 반환해버리면 더 이상의 비교 없이 선택된 오브젝트로 병합 됩니다. 이는 swallow merge처럼 구현되어 버린다는 의미입니다.

이런 상황에서는 `undefined`를 반환해야 합니다. 이렇게 처리를 하면 `mergeWith`는 원래의 자기 알고리즘대로 처리하기 때문에 deep merge를 이어 나갈 수 있게 됩니다.

```javascript
_.mergeWith(A1, A2, (a1, a2) => {
  return (_.isObject(a1) && _.isObject(a2)) ? undefined : a1 || b2
})
```

# 마치며

지금까지 살펴본 것처럼, 두 오브젝트를 병합하는 규칙은 하나만 있는 것이 아닙니다. 상황에 따라 올바른 라이브러리를 선택하고, 때로는 직접 구현해야 할 수도 있습니다. 중요한 것은 테스트입니다. 원하는 상황을 만들어 놓고 테스트를 통해 정말 자신이 원하는 형태의 병합이 이루어지는지 확인한 후에 프로젝트에 적용해야 합니다.