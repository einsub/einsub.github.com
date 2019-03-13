---
layout: post
title: "[번역] Mongoose 모델과 단위 테스트: 최종 가이드"
date: 2019-03-10 10:55:24
author: Reid
categories:
  - engineering
tags:
  - javascript
  - node.js
  - mongodb
  - mongoose
  - unit test
  - sinon
published: true
---

> 원문: [https://codeutopia.net/blog/2016/06/10/mongoose-models-and-unit-tests-the-definitive-guide/](https://codeutopia.net/blog/2016/06/10/mongoose-models-and-unit-tests-the-definitive-guide/)

Mongoose는 node.js로 보다 쉽게 MongoDB를 사용하게 해주는 훌륭한 도구입니다. 모델 관련 로직을 코드 사방에 뿌려놓는 대신, Mongoose의 모델을 이용하면 설계가 매우 쉬워집니다. 데이터를 쿼리하는 것도 쉽고 빠르며, 사용자가 직접 정의한 쿼리 로직도 필요하다면 모델에 숨겨 둘 수 있습니다.

하지만 어떻게 이 모든 로직을 테스트 할 수 있을까요? 처음으로 테스트를 시도하려고 할 때, Mongoose의 모델은 약간 위압적으로 느껴질 수 있습니다. 가장 이상적인 방법은 데이터베이스에 연결을 하지 않고도 테스트를 할 수 있는 것입니다. 데이터베이스에 연결하면 결국 상태를 관리해야 하기 때문에 테스트가 느리고 설정이 어려워집니다. 하지만, 어떻게 해야 할까요?

사실, 독자들에게 Mongoose에 대한 질문을 가장 흔하게 받습니다.

그럼 Mongoose 모델을 테스트하는 방법을 알아보겠습니다. 일단, 가장 일반적인 테스트 상황을 예로 들어 보겠습니다.

## 시작

Mongoose 모델 테스트는 처음에는 꽤 복잡해 보입니다. Schema, Model, Validator, Finder, Statics 같은 것들 때문에 말이죠.

뿐만 아니라 우리가 이해해야 하는 두 가지가 있습니다:

1. Mongoose 모델 로직 자체의 테스트 - validation 같은 것들
2. 모델을 사용하는 코드의 테스트 - Finder를 사용하거나 그 외의 MongoDB를 쿼리하는 방식들

이게 왜 처음에는 도전적인 일로 느껴질지 이해가 됩니다. 하지만 제가 보여 드릴 테스트 방식을 숙달한다면 매우 간단한 일이 될 것입니다. 가장 좋은 점이요? Mongoose 모델을 테스트 할 때 사용하는 방식은 여러분이 다른 코드를 테스트 할 때 사용하는 방식과 똑같습니다.

우리가 사용할 도구는 테스트를 실행하기 위한 mocha, assertion을 위한 chai, 그리고 필요한 곳에 stub을 만들기 위한 sinon입니다.

다음과 같은 방법으로 설정 할 수 있습니다.

```bash
npm install -g mocha
```

```bash
npm install --save sinon chai
```

## Part 1. 모델 테스트

먼저 모델 오브젝트의 각 부분을 테스트하는 방법부터 살펴보겠습니다.

### Model validation 테스트

좋은 모델이 갖춰야 하는 가장 중요한 것 중 하나는 유효성 검사입니다. 여러분은 데이터베이스에 잘못된 데이터가 들어가는 것을 원하지 않겠죠. MongoDB 자체로는 그런 것에 관심이 없습니다. 데이터가 이상해보여도 그냥 넣어버리죠.

Mongoose는 일반적으로 데이터가 MongoDB로 전송되기 전, `save()`를 호출 할 때 오브젝트의 유효성 검사를 합니다. 이 때 유효성 검사를 하게 되는 validate() 함수를 사용해서 테스트를 작성 할 수 있습니다. 이 함수는 데이터베이스 연결이 필요없습니다.

예를 들어, 다음과 같은 스키마와 모델이 있다고 가정해봅시다.

```javascript
var mongoose = require('mongoose');

var memeSchema = new mongoose.Schema({
    name: { type: String }
});

module.exports = mongoose.model('Meme', memeSchema);
```

이 컬렉션에 name이 없이 저장되면 안되겠죠. 어떻게 테스트 할 수 있을까요?

```javascript
var expect = require('chai').expect;

var Meme = require('../src/Meme');

describe('meme', function() {
    it('should be invalid if name is empty', function(done) {
        var m = new Meme();

        m.validate(function(err) {
            expect(err.errors.name).to.exist;
            done();
        });
    });
});
```

위의 테스트를 실행하면 스키마에 `required` 속성이 누락되었음을 빠르게 알 수 있습니다. name에 `required`를 포함 시키면, 테스트가 통과됩니다.

```javascript
var memeSchema = new mongoose.Schema({
    name: { type: String, required: true }
});
```

좋습니다. 아주 간단하죠. 다른 방식의 유효성 검사는 어떨까요?

그렇습니다. 이게 전부입니다. 이 똑같은 패턴을 사용해서 모든 종류의 유효성 검사 로직을 테스트 할 수 있습니다.

1. 우리가 의도한 상태로 검사를 수행할 모델을 생성
2. 콜백과 함께 `validate`를 호출
3. 콜백에서, 에러를 찾아내기 위해 assert를 수행

조금 더 고급 예제를 살펴보겠습니다.

dank가 true일 때에만 repost를 허용하고 싶습니다.

```javascript
var memeSchema = new mongoose.Schema({
    name: { type: String, required: true },
    dank: { type: Boolean },
    repost: {
        type: Boolean,
        validate: function(v) {
            return v === true && this.dank === true;
        }
    }
});
```

`repost`의 validator 함수는 자기 뿐만이 아니라 `dank` 역시 true일때에만 통과시킵니다.

이제 테스트를 작성해보겠습니다. 이전에 했던 것과 똑같은 방식으로 테스트 할 수 있습니다.

```javascript
it('should have validation error for repost if not dank', function(done) {
    //1. 유효성 검사가 실패하도록 모델을 생성
    var m = new Meme({ repost: true });
 
    //2. validate 실행
    m.validate(function(err) {
        //3. 원하는 에러 속성 체크
        expect(err.errors.repost).to.exist;
        done();
    });
});
 
it('should be valid repost when dank', function(done) {
    //1. 유효성 검사가 성공하도록 모델을 생성
    var m = new Meme({ repost: true, dank: true });
 
    //2. validate 실행
    m.validate(function(err) {
        //3. 이제 발생해서는 안되는 에러 속성 체크
        expect(err.errors.repost).to.not.exist;
        done();
    });
});
```

이번에는 실패와 성공 모두를 테스트했습니다. 이 유효성 검사는 더 복잡하기 때문에 두 조건을 모두 테스트하는 것이 좋습니다.

보시다시피, 이렇게 서로 다른 종류의 테스트도 제가 작성했던 것과 동일한 단계로 수행 할 수 있습니다.

### 모델 인스턴스 메소드 테스트

일반적으로 모델에는 두 가지 종류의 인스턴스 메소드가 있습니다.

1. 데이터베이스를 사용하지 않는 인스턴스 메소드
2. 데이터베이스를 사용하는 인스턴스 메소드

첫번째 테스트는 간단합니다. 원하는 매개 변수로 함수를 호출하고 반환값이나 콜백을 확인하면 됩니다.

두번째는 조금 더 어려울 수 있습니다. repost가 true인 meme가 있는지 확인하는 함수가 있다고 가정해봅시다.

```javascript
memeSchema.methods.checkForReposts = function(cb) {
    this.model('Meme').findOne({
        name: this.name,
        repost: true
    }, function(err, val) {
        cb(!!val);
    });
};
```

이 메소드는 같은 이름을 가지고, `repost`가 `true`인 문서를 MongoDB에 요청합니다. 이걸 테스트 하는 방법을 알아보겠습니다.

먼저 함수가 쿼리를 올바르게 수행하는지 확인하기 위해, 다음처럼 테스트를 작성 할 수 잇습니다.

```javascript
it('should check for reposts with same name', sinon.test(function() {
    this.stub(Meme, 'findOne');
    var expectedName = '이 이름은 나중에 assert를 할 때 사용됩니다';
    var m = new Meme({ name: expectedName });

    m.checkForReposts(function() { });

    sinon.assert.calledWith(Meme.findOne, {
        name: expectedName,
        repost: true
    });
}));
```

먼저 데이터베이스에 접근하지 않도록 `Meme.findOne`을 stub 시킵니다. 이렇게 stub을 해두면 나중에 Sinon을 이용해서 올바른 매개 변수로 호출되었는지 여부를 확인 할 수 있습니다.

그 다음 확인 할 이름을 담는 `expectedName` 변수를 설정합니다. 이 이름으로 새로운 Meme 오브젝트를 만들고 `checkForReposts`를 호출합니다.

마지막으로, `sinon.assert.calledWith`를 이용해서 stub된 finder 함수가 올바르게 호출되었는지 확인합니다. 기대하는 name 값을 변수에 저장했기 때문에, 여기 다시 입력 할 필요가 없고, 예상 값을 손쉽게 확인 할 수 있기 때문에 코드가 한결 간단해졌습니다.

`findOne`에 매개변수가 잘 전달되었는지 확인했다면 이번에는 `findOne`의 결과가 올바르게 처리되었는지 확인하려고 합니다. 이를 위해서 또 다른 테스트가 필요합니다. repost가 존재하는 경우 콜백 함수는 `true`를 담아 호출되어야 합니다.

```javascript
it('should call back with true when repost exists', sinon.test(function(done) {
    var repostObject = { name: 'foo' };
    this.stub(Meme, 'findOne').yields(null, repostObject);
    var m = new Meme({ name: 'some name' });

    m.checkForReposts(function(hasReposts) {
        expect(hasReposts).to.be.true;
        done();
    });
}));
```

이전과 마찬가지로 `Meme.findOne`에 대한 stub을 만들었습니다. 이 경우에는, `null`과 `repostObject`의 값을 반환하게 했습니다. stub의 `yields` 함수는 지정한 값들을 사용하여 콜백 함수를 자동으로 호출합니다. 이 경우 `null`을 전달해서 에러가 없다는 것을 명시하고 Mongoose 모델을 찾은 것처럼 `repostObject`를 반환합니다.

이번에는 `checkForReposts`에 콜백을 사용하여 올바른 값으로 호출했는지 확인하기 위해 assert를 수행합니다.

### 정적 함수 테스트

정적 Mongoose 모델 함수를 테스트하는 것은 인스턴스 메소드를 테스트하는 것과 똑같습니다. 차이점은 테스트 중에 여러분이 모델 인스턴스를 만들지 않는다는 것입니다.

1. 모든 데이터베이스 접근자를 stub 시킴
2. DB 쿼리 이외의 로직을 테스트하는 경우 stub을 설정하여 값을 반환
3. `sinon.assert.calledWith`를 사용하여 DB 쿼리가 올바르게 작성되었는지 확인

## Part 2. Mongoose 모델을 사용하는 테스트 코드

지금까지는 모델 자체 테스트에 대해 알아보았고, 이번에는 모델을 사용하는 코드를 테스트하는 방법을 살펴 보겠습니다.

대부분의 경우 매우 간단합니다. stub을 사용하여 대부분의 테스트를 수행 할 수 있습니다.

코드 예제와 테스트 방법에 대해 살펴보겠습니다.

## 데이터를 쿼리하는 함수 테스트

앱에서 모델을 얻는 가장 일반적인 방법은 데이터베이스를 쿼리하는 것입니다. 이것은 서비스가 될 수도 있고, 헬퍼 함수가 될 수도 있으며, 어쩌면 Express route가 될 수도 있습니다.

다음 함수를 가정 해봅시다.

```javascript
var Meme = require('./Meme');
 
module.exports.allMemes = function(req, res) {
    Meme.find({ repost: req.params.reposts }, function(err, memes) {
        res.send(memes);
    });
};
```

이 함수는 Express를 이용해 route 됩니다. reposts 플래그를 옵셔널하게 취해서 모든 meme를 로드하고, JSON으로 전송합니다.

`Meme.find`를 stub 시켜서 손쉽게 테스트 할 수 있습니다.

```javascript
var expect = require('chai').expect;
var sinon = require('sinon');
 
var routes = require('../src/routes');
var Meme = require('../src/Meme');
 
describe('routes', function() {
    beforeEach(function() {
        sinon.stub(Meme, 'find');
    });
 
 
    afterEach(function() {
        Meme.find.restore();
    });
 
    it('should send all memes', function() {
        var a = { name: 'a' };
        var b = { name: 'b' };
        var expectedModels = [a, b];
        Meme.find.yields(null, expectedModels);
        var req = { params: { } };
        var res = {
            send: sinon.stub()
        };
 
        routes.allMemes(req, res);
 
        sinon.assert.calledWith(res.send, expectedModels);
    });
});
```

이 경우 `beforeEach`와 `afterEach`를 사용해서 자동으로 finder 함수를 stub, restore 할 수 있습니다. 여러 테스트에서 동일한 stub이 필요한 경우 이런 hook들을 이용하면 편리합니다.

앞의 테스트와 마찬가지로, 여기서도 몇 가지 예상 데이터를 설정합니다. 여기서 find 함수가 반환해야 하는 것은 Meme 모델 목록입니다. 이 경우에도 결과를 만들어내기 위해 stub을 설정합니다.

route의 `req`와 `res` 매개 변수도 여기서 설정이 가능합니다. `res`의 경우 `send`함수로 stub을 설정해서 나중에 assert에 활용 할 수 있습니다.

route를 호출 한 후, `sinon.assert.calledWith`를 사용하여 올바른 동작을 확인합니다.

repost 플래그를 사용하여 동작을 확인하는 테스트를 수행 할 수도 있습니다.

```javascript
it('should query for non-reposts if set as request parameter', function() {
    Meme.find.yields(null, []);
    var req = {
        params: {
            reposts: true
        }
    };
    var res = { send: sinon.stub() };

    routes.allMemes(req, res);

    sinon.assert.calledWith(Meme.find, { repost: true });
});
```

이전과 매우 비슷한 방법을 사용하고 있음을 알 수 있습니다. stub을 설정하고, 데이터를 설정하고, 함수를 호출하고, assert를 합니다.

똑같은 패턴으로 많은 것을 테스트 할 수 있습니다. 실제로 이전 모델과 동일한 패턴이 반복됩니다.

## Bonus: 테스트 데이터 다루기
(이를 포함하도록 제안한 [Valeri Karpov](https://twitter.com/code_barbarian)에게 감사드립니다)

이런 테스트를 작성 할 때 동일한 유형의 테스트 데이터를 재사용 할 수 있습니다. 예를 들어 모델을 사용하여 다른 route나 모듈을 테스트 할 때 테스트를 위해 가짜 데이터를 만들어야 합니다.

위에서 보았듯이 테스트 과정에서 테스트 데이터를 인라인으로 정의하는 것이 시작입니다. 그러나 점점 더 많은 테스트를 거치다 보면 같은 테스트 데이터를 반복해서 정의하게 됩니다. 특히 일부 코드가 모델의 속성 중 하나를 사용한다고 가정 할 때, 테스트에서 데이터가 올바르게 보이도록 하는 것은 어려운 문제가 될 수 있습니다.

테스트들마다 데이터를 copy/paste하지 않으려면 데이터를 만드는 헬퍼 함수를 만들어야 합니다.

예를 들어, 다음과 같은 코드로 작성된 `test/factories.js` 파일을 만들 수 있습니다. 어떤 타입의 오브젝트를 만드는 함수는 보통 factory라고 부릅니다.

```javascript
module.exports.validMeme = function() {
  return {
    name: 'Some name here',
    dank: false,
    repost: false
  };
};
 
module.exports.repostMeme = function() {
  return {
    name: 'Some name here',
    dank: false,
    repost: true
  };
};
```

이전의 테스트를 이렇게 여러 테스트 데이터가 있는 테스트로 다시 작성 할 수 있습니다.

```javascript
var factories = require('./factories');

it('should send all memes', function() {
  var a = factories.validMeme();
  var b = factories.validMeme();
  var expectedModels = [a, b];
  Meme.find.yields(null, expectedModels);
  var req = { params: { } };
  var res = {
    send: sinon.stub()
  };

  routes.allMemes(req, res);

  sinon.assert.calledWith(res.send, expectedModels);
});
```

이 방식의 장점은 테스트에 필요한 코드의 양을 줄이고 더 쉽게 유지 보수할 수 있다는 것입니다. 예를 들어 모델 필드에 새 필드를 추가할때, 수십개의 테스트들을 수정하기보다 factory 하나만 업데이트하면 됩니다.

## 결론

Mongoose와 같은 라이브러리를 사용하는 단위 테스트 앱은 처음에는 복잡해보일 수 있습니다. 그러나 기본적인 것들을 배우고 적용하면 동일한 테스트 패턴이 계속 반복되는 것을 알 수 있습니다.

모델을 테스트하거나 그것들을 이용해서 코드를 작성 할 때, 요점은 무엇이 stub 되어야 할지 알아내는 것입니다. 보통은 데이터베이스와 대화가 필요한 무엇인가가 있다면, 바로 그것이 stub 되어야 합니다. 일단 그렇게 해두면, 이제 stub이 뭔가를 하도록 설정만 하면 끝입니다. 

설정이 완료된 프로젝트를 예제와 함께 [Github](https://github.com/jhartikainen/mongoose-testing-examples)에서 구할 수 있습니다.

Sinon.js를 사용해서 다른 종류의 테스트를 쉽게하는 방법에 대해 보다 더 알고 싶나요? 제가 작성한 무료 [Sinon.js in the Real-world guide](http://codeutopia.net/go/sinon-pdf-download-page/)를 확인해 보세요. 저의 최고의 sinon.js 컨텐츠가 한 곳에 모여 있습니다!  

