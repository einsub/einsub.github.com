---
layout: post
title: "Mongoose가 'undefined'를 처리하는 방식에 대해"
date: 2019-03-22 15:37:45
author: Reid
categories:
  - engineering
tags:
  - mongodb
  - mongoose
  - undefined
published: true
---

MongoDB 컬렉션에 문서를 업데이트하는 코드를 넣고 테스트를 하다가 업데이트하는 필드 중 하나에 뜬금없이 `null`값이 들어가 있는 이상한 현상을 발견했습니다. 테스트 코드를 만들어 확인해보니, 문서의 특정 필드가 `undefined`인채로 `FindOneAndUpdate` 함수가 호출되면 그 필드는 `null`로 변환되어 저장되고 있습니다. 지금까지 `undefined` 값을 가진 필드는 MongoDB에 입력되기 전에 무시되는 걸로 알고 있었는데 말이죠.

좀 더 테스트를 해보고 내린 결론은 `save()`를 할 때는 `undefined`가 무시되고, `update()`를 하면 `undefined`가 `null`로 치환되어 저장된다는 것입니다. 이 것이 update 연산자로 연산을 하기 위해서 어쩔 수 없이 존재하는 문제인지, 아니면 다른 이유가 있는것인지까지는 확인해보지 못했습니다. 하지만 필드를 명시적으로 `null`로 업데이트하는 것이 아닌데도 불구하고, `undefined` 값이 필드에 저장되는 것은 조금 이상합니다.

**이 이슈에서 기억해두실 점은, '[mongoose](https://mongoosejs.com/)를 사용하면서 특정 필드의 값이 채워지지 않은채로 입력값이 들어오는 경우 `null` 필드가 만들어질 수 있다'입니다**

[Mongoose issue 페이지](https://github.com/Automattic/mongoose/issues/3486)에 가서 보니 프로젝트 오너가 undefined를 지원하지 않는 js-bson 때문에, 내부에서 null로 자동 변환된다고 설명하네요. 이것은 아마도 mongoose가 아니라 mongoDB 쪽의 문제로 보입니다. (관련 페이지: [mongodb/js-bson#134](https://github.com/mongodb/js-bson/issues/134)). 질문자도 저와 똑같은 궁금증으로 질문을 올린 것 같은데, 오너는 질문의 의도를 정확하게 파악하지 못한 것 같습니다.

아무리 내부적인 한계가 있다지만, `save()`를 할 때는 알아서 제거해주던 것을, `update()`를 할 때는 그대로 둔다는 건 API 인터페이스의 일관성이 없는 문제로 보입니다. 특히, save()는 mongoose 프레임워크에서 지원하는 것이니 MongoDB 탓만 할게 아닐 것 같고요.

아래는 테스트 코드입니다:

```javascript
const mongoose = require('mongoose')

const UndefinedTest = mongoose.model('UndefinedTest', new mongoose.Schema({
  name: String,
  job: String,
  family: {
    name: String,
    age: Number
  },
}));

(() => {
  mongoose.connect('mongodb://localhost/dev', {
    useNewUrlParser: true
  }, async (err) => {
    const db = mongoose.connection

    await UndefinedTest.deleteMany({})

    const saveData = {
      name: 'Reid',
      job: undefined,
      family: {
        name: undefined,
        age: 17
      }
    }
    await new UndefinedTest(saveData).save()
    const saveDoc = await UndefinedTest.find({ name: 'Reid' }).lean()
    console.log('Create a document with undefined fiels')
    console.table(saveDoc)

    const updateData = {
      job: undefined,
      family: {
        name: undefined,
        age: 27
      }
    }
    await UndefinedTest.findOneAndUpdate({ name: 'Reid' }, { $set: updateData })
    const updateDoc = await UndefinedTest.find({ name: 'Reid' }).lean()
    console.log('Update the document with undefined fiels')
    console.table(updateDoc)

    await db.close()
  })
})()
```

```plaintext
Create a document with undefined fiels
┌─────────┬──────────────────────────┬────────┬─────────────┬─────┐
│ (index) │           _id            │  name  │   family    │ __v │
├─────────┼──────────────────────────┼────────┼─────────────┼─────┤
│    0    │ 5c947208fb19935055d655cf │ 'Reid' │ { age: 17 } │  0  │
└─────────┴──────────────────────────┴────────┴─────────────┴─────┘

Update the document with undefined fiels
┌─────────┬──────────────────────────┬────────┬─────────────────────────┬─────┬──────┐
│ (index) │           _id            │  name  │         family          │ __v │ job  │
├─────────┼──────────────────────────┼────────┼─────────────────────────┼─────┼──────┤
│    0    │ 5c947208fb19935055d655cf │ 'Reid' │ { name: null, age: 27 } │  0  │ null │
└─────────┴──────────────────────────┴────────┴─────────────────────────┴─────┴──────┘
```
