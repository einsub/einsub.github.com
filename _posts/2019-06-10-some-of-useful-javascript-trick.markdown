---
layout: post
title: "유용한 JavaScript 코딩 기법들"
date: 2019-06-10 21:48:15
author: Reid
categories:
  - engineering
tags:
  - javascript
  - tip
  - trick
  - es6
  - lodash
  - asnyc
published: true
---

개발자 웹에는 코딩 트릭을 소개하는 글들이 자주 포스팅 됩니다. 특히 자바스크립트 처럼 명세가 빠르게 바뀌는 언어는 더 짧고 간결한 코딩 기법들이 계속 나타납니다. 뉴스를 읽다가 이런 기술을 소개하는 페이지를 만나면 알고 있는 기술인지 한번쯤 훑어보게 되는데요. 이 글은 그렇게 한번 훑어보며 제가 알고 있는 사실과 비교하고 쓸만한지 따져보는 글입니다.

> 원문: [https://devinduct.com/blogpost/26/8-useful-javascript-tricks](https://devinduct.com/blogpost/26/8-useful-javascript-tricks) - [Milos Protic](mailto:admin@devinduct.com)

## 1. 배열 선언과 동시에 값 채워넣기

```javascript
let array = Array(5).fill('');
console.log(array)
```

배열을 선언하면서 동시에 동일한 값을 채워넣는 작업은 생각보다 흔히 쓰이지 않습니다. 특히 고정 길이 배열이 아닌 동적 길이 배열을 사용하는 자바스크립트에서는 더욱 그렇죠. 저는 차라리 이 코드를 보며 배열로 for 루프를 없애는 코딩 기술이 떠올랐습니다.

# 빈 배열로 for 루프 없애기

```javascript
let sum = 0
for (let i = 0; i < 10; ++i) {
  sum += i
}
```

최근 함수형 프로그래밍이 인기를 얻으면서 for, while의 사용 빈도가 현저하게 줄어들고 있습니다. `let` 변수의 불완전성으로 인해 루프 안에서 상태가 달라질지 모른다는 두려움이 있기 때문입니다. (함수형 프로그래밍에 대해서는 [이 글](https://blog.ull.im/engineering/2019/04/07/functional-programming-with-javascript-in-3-steps.html)을 참고) 이미 존재하는 배열을 어떤 함수로 전달하여 원하는 동작을 취하는 것은 별로 어렵지 않습니다. 함수형으로 코딩을 하다보면 자주 만나는 상황이니까요. 하지만 순수한 for 루프처럼 정해진 횟수만큼 수행해야 하는 작업이라면 좀 난감합니다. [async](https://caolan.github.io/async/v3/docs.html#times)나 [lodash](https://lodash.com/docs/4.17.11#times)의 `times(n)` 함수를 이용 할 수는 있지만 가볍게 순수 자바스크립트로 작성하고 싶다면 아래 트릭을 사용할 수 있습니다.

```javascript
const sum = [...Array(10)].reduce((acc, _, i) => acc += i, 0)
```

`Array(n)`으로 배열의 크기를 지정한 후 spread 연산자로 펼쳐주면 `undefined`의 아이템들이 뿌려지는데 이 것들을 `[]`로 다시 묶어 배열로 활용하는 것이 핵심입니다. 보기에는 괜히 복잡해보이는 점이 아쉽지만, 함수형 프로그래밍을 이어갈 수 있게 해주는 것이 너무 큰 장점입니다.

## 2. 유일 값 배열 구하기

```javascript
const cars = [
  'Mazda', 
  'Ford', 
  'Renault', 
  'Opel', 
  'Mazda'
]
const uniqueWithArrayFrom = Array.from(new Set(cars));
console.log(uniqueWithArrayFrom); // outputs ["Mazda", "Ford", "Renault", "Opel"]

const uniqueWithSpreadOperator = [...new Set(cars)];
console.log(uniqueWithSpreadOperator);// outputs ["Mazda", "Ford", "Renault", "Opel"]
```

`Set` 내장 객체를 이용하여 배열의 중복값을 제거하는 트릭입니다. `Array.from` 함수가 `arrayLike`를 인자로 받기 때문에 가능한 코드입니다. `arrayLike`는 '유사 배열 객체', '순회 가능한 객체'를 일컫는데 이들은 `String`, `Set`, `Map`, `arguments`와 같은 것들이 있습니다.

# lodash를 이용한 유일 값 배열 구하기

```javascript
import * as _ from 'lodash'

const cars = [
  'Mazda', 
  'Ford', 
  'Renault', 
  'Opel', 
  'Mazda'
]

const uniqCars = _.uniq(cars)
console.log(uniqCars)
```

lodash의 [uniq](https://lodash.com/docs/4.17.11#uniq) 함수를 이용하면 좀 더 직관적으로 배열의 아이템들이 유니크하다는 것을 잘 표현 할 수 있지 않을까 생각합니다만, 1번 섹션처럼 순수 자바스크립트를 이용하려면 `Array.from(new Set())`을 이용하는 것도 좋아 보입니다.

## 3. spread 연산자를 이용한 객체 병합

이 예제는 두 가지 경우의 병합 방식을 보여줍니다.

첫 번째 예제는 두 오브젝트의 병합입니다. spread 연산자 예제에 자주 나오는 방식이므로 사용해보신 적이 있으실겁니다. 이 연산자가 나오기 전에 주로 사용했던 방식으로 `Object.assign`이 있습니다. 하지만, 이 함수는 인자로 넘어가는 객체 자신을 변경시켜 버리기 때문에 함수형 프로그래밍의 컨셉에 부합하지 않습니다.

```javascript
// merging objects
const product = { name: 'Milk', packaging: 'Plastic', price: '5$' }
const manufacturer = { name: 'Company Name', address: 'The Company Address' }

const productManufacturer = { ...product, ...manufacturer };
console.log(productManufacturer); 
// outputs { name: "Company Name", packaging: "Plastic", price: "5$", address: "The Company Address" }
```

두 번째 예제는 객체들이 담긴 배열로부터 하나의 객체을 만듭니다. `reduce` 함수, spread 연산자 그리고 동적으로 객체 키를 설정하는 기술([Enhanced Object Literals](https://codetower.github.io/es6-features/#Enhanced%20Object%20Literals))을 이용합니다. 이 코딩 기술은 json 객체를 조작 할 때 굉장히 유용하게 사용 될 수 있으니 익숙해지시면 좋습니다.

```javascript
// merging an array of objects into one
const cities = [
  { name: 'Paris', visited: 'no' },
  { name: 'Lyon', visited: 'no' },
  { name: 'Marseille', visited: 'yes' },
  { name: 'Rome', visited: 'yes' },
  { name: 'Milan', visited: 'no' },
  { name: 'Palermo', visited: 'yes' },
  { name: 'Genoa', visited: 'yes' },
  { name: 'Berlin', visited: 'no' },
  { name: 'Hamburg', visited: 'yes' },
  { name: 'New York', visited: 'yes' }
];

const result = cities.reduce((accumulator, item) => {
  return {
    ...accumulator,
    [item.name]: item.visited
  }
}, {});

console.log(result);
/* outputs
Berlin: "no"
Genoa: "yes"
Hamburg: "yes"
Lyon: "no"
Marseille: "yes"
Milan: "no"
New York: "yes"
Palermo: "yes"
Paris: "no"
Rome: "yes"
*/
```

## 4. Array.map 없이 배열 맵핑하기

```javascript
const cities = [
  { name: 'Paris', visited: 'no' },
  { name: 'Lyon', visited: 'no' },
  { name: 'Marseille', visited: 'yes' },
  { name: 'Rome', visited: 'yes' },
  { name: 'Milan', visited: 'no' },
  { name: 'Palermo', visited: 'yes' },
  { name: 'Genoa', visited: 'yes' },
  { name: 'Berlin', visited: 'no' },
  { name: 'Hamburg', visited: 'yes' },
  { name: 'New York', visited: 'yes' }
];

const cityNames = Array.from(cities, ({ name}) => name);
console.log(cityNames);
// outputs ["Paris", "Lyon", "Marseille", "Rome", "Milan", "Palermo", "Genoa", "Berlin", "Hamburg", "New York"]
```

`Array.map`을 사용하지 않고, `Array.from`을 이용하여 새로운 배열을 만들어내는 예제 코드입니다. 처음 알게 된 트릭이지만, 코드가 크게 향상된 것 같지는 않아서 굳이 구분하여 사용 할 필요가 있을까 생각합니다.

# Array.map vs Array.from

혹시 성능의 이점이 있지 않을까 해서 간단히 테스트를 해 보았습니다. 10번을 수행해서 걸린 시간을 출력해봤는데, `map`이 `from`에 비해 근소하게 빠른 결과가 나왔네요. 꼭 필요하지 않다면 그냥 `map`을 쓰는 것이 좋아보입니다.

```javascript
const nums = array(100000).fill(1)

console.time('elapsed')
nums.map(v => v * 9)
console.timeend('elapsed')

/*
19.942ms
17.738ms
19.132ms
17.752ms
19.036ms
16.826ms
17.578ms
17.710ms
17.385ms
17.143ms
*/
```

```javascript
const nums = array(100000).fill(1)

console.time('elapsed')
array.from(nums, v => v * 9)
console.timeend('elapsed')

/*
20.360ms
18.897ms
18.503ms
19.231ms
21.684ms
18.285ms
18.567ms
18.561ms
21.863ms
18.659ms
*/
```

## 5. 조건부 객체 속성

```javascript
const getUser = (emailIncluded) => {
  return {
    name: 'John',
    surname: 'Doe',
    ...emailIncluded && { email : 'john@doe.com' }
  }
}

const user = getUser(true);
console.log(user); // outputs { name: "John", surname: "Doe", email: "john@doe.com" }

const userWithoutEmail = getUser(false);
console.log(userWithoutEmail); // outputs { name: "John", surname: "Doe" }
```

조건에 따라 객체의 인자가 달라져야 하는 경우, 겨우 이것 하나 때문에 객체를 두 벌 만들어서 골라야 하는가 하는 자괴감이 들었던 분들에게 유용한 예제입니다.

```javascript
...emailIncluded && { email : 'john@doe.com' }
```

이 코드는 `emailIncluded`가 `true`라면 `email` key/value를 객체에 추가하고 `false`라면 추가하지 않도록 합니다. 여기서 주의해야 할 부분은 연산자 수행 순서입니다. `...` 연산자로 펼쳐진 다음 `&&`를 만나는 것이 아니라, `&&` 연산자로 AND 처리가 된 후에 그 결과에 대해 `...` 연산자로 펼쳐지는 것입니다. 연산자 우선 순위를 잘 모른다면 헷갈리는 코드가 되기 십상입니다. 제가 만약 이 코드를 프로덕션에 사용한다면 다음처럼 괄호를 포함하고 싶을 것 같습니다.

```javascript
...(emailIncluded && { email : 'john@doe.com' })
```

이 트릭도 `if` 구문을 없애고 싶은 테크니션들의 절규와 같은 것인데요. 이 코드를 `if`가 있는 형태로 작성해볼까요?

```javascript
const getUser = (emailIncluded) => {
  const obj = {
    name: 'John',
    surname: 'Doe',
  }
  if (emailIncluded) {
    obj.email = 'john@doe.com'
  }
  return obj
}
```

어떤가요? 코드가 살짝 길어지긴 했어도 이 예제가 더 보기 좋은가요? 사람마다 호불호가 많이 갈릴 것 같은 주제입니다. 다만, 유용함 때문에 코드 컨벤션을 파괴하는 예는 예전에도 있었죠.

```javascript
const name = 'Reid'
const username = name || 'Guest'
console.log(username)
```

또는

```javascript
const user = { name: 'Reid' }
// const user = undefined
const message = user && 'User exists'
if (message) {
  console.log(message)
}
```

## 6. 데이터 destructing 하기

여기 이 예제는 매우 중요합니다. destructing을 자주 사용하는 사람 중에도 개별 변수가 아니라 object의 속성으로 분해/저장하는 방식은 익숙하지 않은 사람들이 있으니까요. 저처럼요.

```javascript
const rawUser = {
  name: 'John',
  surname: 'Doe',
  email: 'john@doe.com',
  displayName: 'SuperCoolJohn',
  joined: '2016-05-05',
  image: 'path-to-the-image',
  followers: 45
}

let user = {}, userDetails = {};
({ name: user.name, surname: user.surname, ...userDetails } = rawUser);

console.log(user); // outputs { name: "John", surname: "Doe" }
console.log(userDetails); // outputs { email: "john@doe.com", displayName: "SuperCoolJohn", joined: "2016-05-05", image: "path-to-the-image", followers: 45 }
```

가장 쉽고 자주 사용되는 destructing은 다음과 같습니다:

```javascript
const rawUser = {
  name: 'John',
  surname: 'Doe',
  email: 'john@doe.com',
  displayName: 'SuperCoolJohn',
  joined: '2016-05-05',
  image: 'path-to-the-image',
  followers: 45
}

const { name, email } = rawUser
```

`rawUser` 객체에서 원하는 객체만 pick 해서 새로운 변수를 만듭니다. 하지만, 실제로 코딩을 하다보면 이렇게 개별 변수가 아니라 집어낸 몇몇 속성들로 새로운 객체를 만들어야 하는 경우가 더 많습니다. 그럴 때 이 기술을 이용해야 합니다.

```javascript
const rawUser = {
  name: 'John',
  surname: 'Doe',
  email: 'john@doe.com',
  displayName: 'SuperCoolJohn',
  joined: '2016-05-05',
  image: 'path-to-the-image',
  followers: 45
}

let user = {};
({ name: user.name, email: user.email } = rawUser)
console.log(user)
// { name: 'John', email: 'john@doe.com' }
```

[lodash](https://lodash.com/docs/4.17.11#pick)를 이용해서 좀 더 가독성 좋게 만드는 것도 추천합니다.

```javascript
const user = _.pick(rawUser, ['name', 'email'])
```

## 7. 동적 속성 이름

```javascript
const dynamic = 'email';
let user = {
  name: 'John',
  [dynamic]: 'john@doe.com'
}
console.log(user); // outputs { name: "John", email: "john@doe.com" }
```

ES6에 추가된 유용한 기능 중 하나로 객체의 key를 동적으로 생성 할 수 있습니다. 이런 방식이 없었던 시절에는 아래와 같은 처리가 불가피했습니다.

```javascript
const dynamic = 'email';
let user = {
  name: 'John'
}
user[dynamic] = 'john@doe.com'
console.log(user); // outputs { name: "John", email: "john@doe.com" }
```

간단하지만, 굉장히 유용한 트릭입니다.

## 8. 문자열 템플릿

ES6에 포함 된 신기술 중 가장 쉽고 널리 사용되는 것 중 하나가 아닐까 생각합니다. 문자열을 `+`로 더해가며 조합하던 방식과 비교하면 실수를 줄일 수 있고, 가독성도 살릴 수 있습니다.

```javascript
const user = {
  name: 'John',
  surname: 'Doe',
  details: {
    email: 'john@doe.com',
    displayName: 'SuperCoolJohn',
    joined: '2016-05-05',
    image: 'path-to-the-image',
    followers: 45
  }
}

const printUserInfo = (user) => { 
  const text = `The user is ${user.name} ${user.surname}. Email: ${user.details.email}. Display Name: ${user.details.displayName}. ${user.name} has ${user.details.followers} followers.`
  console.log(text);
}

printUserInfo(user);
// outputs 'The user is John Doe. Email: john@doe.com. Display Name: SuperCoolJohn. John has 45 followers.'
```

여러 줄에 걸쳐 문자열을 작성하는 방법도 간단합니다. 전체 문자열을 ``로 감싼 상태에서 그냥 엔터키를 쳐주면 됩니다.

```javascript
const text = `The user is ${user.name} ${user.surname}.
Email: ${user.details.email}. 
Display Name: ${user.details.displayName}.
${user.name} has ${user.details.followers} followers.`
/*
The user is John Doe.
Email: john@doe.com. 
Display Name: SuperCoolJohn.
John has 45  followers.
*/
```

# 나가며

자바스크립트는 언어의 새로운 트렌드에 발맞춰 계속 발전하고 있습니다. 다른 언어에서 검증 된 유용한 기술이나, 함수형 언어가 가져야 할 필수 조건들을 하나씩 장착하고 있죠. 앞으로도 자바스크립트의 변화는 계속 될 것입니다.

이런 흐름을 무시하면 기술을 코드에 적용하는 것은 둘째 치고, 다른 사람이 작성한 코드를 이해하는 것 부터 문제가 생깁니다. 결국 자주 사용하면서 몸에 익혀야 하는 수 밖에 없습니다. 새로운 기술들을 모두 외워두고 필요할 때 마다 사용하는 것은 어려운 일입니다. 이 때 유념해두면 좋은 팁이 있습니다. 

- `for`, `while` 루프는 함수형으로 대체 가능한지 고민한다.
- `let` 변수를 쓰지 않으려고 노력한다. 
- 자바스크립트 내장 함수나 객체를 우선 살피고, 없는 경우 [lodash](https://lodash.com/)나 [underscore](https://underscorejs.org/), [async](https://caolan.github.io/async/v3/index.html)와 같은 외부 모듈을 확인한다.

일단 이것들만 기억하고 코드를 작성해도 많은 변화가 찾아 올겁니다. 처음에는 더디고 느리겠지만 점차 몸에 익숙해지면 어떤 상황에 어떤 기술을 사용해야 할 지 떠오르게 됩니다. 한번 도전해보세요!
