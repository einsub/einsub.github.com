---
layout: post
title: "MongoDB와 Mongoose에서 update를 사용 할 때 주의할 점"
date: 2019-03-08 00:46:24
author: Reid
categories:
  - engineering
tags:
  - nodejs
  - mongodb
  - mongoose
published: true
---

처음부터 [Mongoose](https://mongoosejs.com/)를 ODM(Object Document Mapper)으로 썼던 탓에, [MongoDB](https://www.mongodb.com/) 문법을 Mongoose로 배웠습니다. 기본적인 쿼리 인터페이스는 이 둘 간에 큰 차이가 없기 때문에 이후 [MongoDB Driver API](http://mongodb.github.io/node-mongodb-native/2.0/api/index.html)를 사용해 코드를 작성 할 때도 별다른 불편함을 느끼지 못했습니다. 오히려 오리지널의 API가 가진 불편함을 해소시켜주는 Mongoose의 친절한 부분에 고마워하기도 했지요. 별 생각없이 `update` 쿼리를 작성했다가 혼돈의 카오스를 경험하기 전까지는요.

DB 코드를 작성하다가 update 쿼리를 보고 불현듯 그때의 당황스러웠던 기억이 떠올라서 글을 써봅니다.

### Mongoose, MongoDB 너희의 update는...

Mongoose와 MongoDB의 `update` 쿼리는 동작하는 방식이 다릅니다. MongoDB는 업데이트 할 내용을 그대로 replace 해버리는 반면, Mongoose는 merge 하는 것 처럼 동작합니다. Mongoose가 왜 원본의 방식을 따르지 않고 이렇게 방식을 바꾸어 놓았을까요? Mongoose API 문서를 보면 그 이유를 설명하고 있습니다.

```javascript
var query = { name: 'borne' };
Model.update(query, { name: 'jason bourne' }, options, callback)

// is sent as
Model.update(query, { $set: { name: 'jason bourne' }}, options, callback)
// if overwrite option is false. If overwrite is true, sent without the $set wrapper.
```
> All top level keys which are not atomic operation names are treated as set operations. This helps prevent accidentally overwriting all documents in your collection with { name: 'jason bourne' }.

기본적으로 mongoose는 입력받는 데이터에 `$set` 연산자가 없으면 자기가 알아서 붙여준다는 것입니다. 친절하게도 **우리가 실수로 문서를 통째로 덮어씌워버리는 걸 막아줄게**라고 설명서에 적어놓았습니다.

하지만, MongoDB API는 어떤 방식으로 동작할까요? 아래 비교 예제를 살펴보시죠.

### Mongoose에서 `Model.update` 쿼리 사용하기

```javascript
const mongoose = require('mongoose');

const Menu = mongoose.model('MenuMongoose', new mongoose.Schema({
  name: String,
  price: Number
}));

(async () => {
  const conn = await mongoose.connect('mongodb://localhost/dev');
  await new Menu({ name: 'Americano', price: 4500 }).save();
  await new Menu({ name: 'Latte', price: 5000 }).save();
  // Mongoose Model.update를 이용한 update
  await Menu.update({ name: 'Americano' }, { price: 10000 });
  const menus = await Menu.find().lean();
  console.log(menus);
  await conn.disconnect();
})();
```

우리가 기대했던 그대로입니다.

```javascript
[
  { _id: 5c807a308c90a52efb4e4003, name: 'Americano', price: 10000, __v: 0 },
  { _id: 5c807a308c90a52efb4e4004, name: 'Latte', price: 5000, __v: 0 }
]
```

### MongoDB에서 `Collection.update` 쿼리 사용하기

```javascript
const MongoClient = require('mongodb').MongoClient;

(async () => {
  const client = await MongoClient.connect('mongodb://localhost');
  const col = client.db('dev').collection('menumongodb');
  await col.insert({ name: 'Americano', price: 4500 })
  await col.insert({ name: 'Latte', price: 5000 })
  // MongoDB Driver API를 이용한 update
  await col.update({ name: 'Americano' }, { price: 10000 })
  const menus = await col.find().toArray()
  console.log(menus)
  await client.close()
})()
```

뭔가 이상하죠? Americano 어디갔어?
```javascript
[
  { _id: 5c807976b79c7f2e34614f1c, price: 10000 },
  { _id: 5c807976b79c7f2e34614f1d, name: 'Latte', price: 5000 }
]
```

### 이게 문제가 되는 것은...

이렇게 MongoDB는 아무런 미련 없이 원본 데이터를 홀라당 바꿔버립니다. 이런 이유로 MongoDB에서 update 쿼리로 기존 문서를 그저 변경만 하고 싶을 뿐이라면 `$set` 연산자를 이용해야 하는 것입니다. 이렇게요.

```javascript
await col.update({ name: 'Americano' }, { $set: { price: 10000 } })
```

그렇습니다. Mongoose를 먼저 배워서 이런 사정을 몰랐던 사람들에게는 매우 치명적일 수 있었던 것이죠. 프로젝트는 Mongoose를 사용하고, 간단한 스크립트는 MongoDB Driver API로 작성하던 저는, 영문도 모른채 데이터들이 아름답게 minimize 되어 있는 것을 보았습니다.

### 서로 다른 길을 가는 MongoDB와 Mongoose

MongoDB API에서 `update` 함수는 결국 deprecated 되었습니다. 이 함수는 [`updateOne`](http://mongodb.github.io/node-mongodb-native/2.0/api/Collection.html#updateOne), [`updateMany`](http://mongodb.github.io/node-mongodb-native/2.0/api/Collection.html#updateMany)로 분화되었는데, 사용자에게 변경 할 문서가 하나인지 여러개인지를 직접 묻게 된것이죠. 아마도 많은 사용자들이 조그만 실수 하나로 전체 데이터를 minimize 시키는 충격적인 결과를 심심치 않게 보았기 때문이 아닐까 싶습니다. 

재미있는 사실을 문서에서 찾아 볼 수 있습니다. deprecated 된 update 함수의 명세에서 이 두 번째 파라메터에 대한 설명이 이렇게 적혀있습니다:

```plaintext
document object  The update document.
```

하지만 새로 만들어진 함수들에는 이렇게 적혀있죠:

```plaintext
update   object  The update operations to be applied to the document.
```

이제 더 이상 update 함수의 인자는 **문서**가 아니고 **오퍼레이션**이니, 헷갈리지 말라고 명백하게 밝히고 있습니다. 사실 두 인자는 여전히 동일한 방식으로 동작하지만, 오해가 생길 소지를 없앤 것이죠. 하지만 Mongoose 처럼 `$set` 연산자가 없어도, 알아서 `$set`을 붙여주는 처리는 하지 않았습니다.

Mongoose는 `$set`의 필수 조건을 없앤 대신, 옵션에 overwrite 항목을 넣어서 명시적으로 replace 여부를 묻는 반면, MongoDB는 여전히 연산자를 넣지 않으면 replace를 하는 방식을 고수하고 있습니다.

두 update 방식의 차이를 몰랐던 분이 계셨다면, 분명 미래의 어느 한 시점에 예정되어 있던 여러분 DB의 minimize화를 제가 막아드린 것으로...
