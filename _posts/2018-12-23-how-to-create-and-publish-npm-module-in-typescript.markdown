---
layout: post
title: "[번역] TypeScript로 NPM 모듈을 만들어 배포하기"
date: 2018-12-23 12:53:40 
author: Reid
categories:
  - engineering
tags:
  - typescript
  - npm
  - module
published: true
---
> 원본 사이트:<br />
https://codeburst.io/https-chidume-nnamdi-com-npm-module-in-typescript-12b3b22f0724

![](https://cdn-images-1.medium.com/max/800/1*JNwD5fwbppQRU_g-cr24kQ.png)

## 소개

이 글에서 우리는 JavaScript 개발자와 TypeScript 개발자 양쪽 모두가 사용 할 수 있는 TypeScript 모듈을 만들어 볼 것입니다.

대부분의 npm 모듈은 Type 정의를 포함하고 있지 않기 때문에, TypeScript 개발자들은 보통 `npm i @types/<module_name> -D`의 추가적인 명령으로 npm 모듈을 사용 할 수 있게 됩니다. 여기서 우리는 JavaScript와 TypeScript에 import 가능한 npm TypeScript 모듈을 만드는 방법을 알아봅니다.

## NPM이 뭔가요?

NPM은 오픈소스 node.js 프로젝트, 모듈, 기타 자원등의 온라인 레포지터리입니다. [http://npmjs.org](http://npmjs.org/)에서 확인 하실 수 있습니다.

NPM은 node.js의 공식 패키지 매니저로서, 레포지터리와 상호 작용을 돕는 명령행 인터페이스(CLI)를 제공합니다. 이 유틸리티는 node.js를 설치 할 떄 자동으로 함께 설치됩니다. API 문서는 [https://npmjs.org/doc/](https://npmjs.org/doc/)에서 찾아보실 수 있고, 터미널에서 `npm`을 입력해도 확인이 가능합니다.

## Node.js 설치

Node.js의 [다운로드 페이지](https://nodejs.org/en/download/)에서 원하는 버전을 다운로드 받을 수 있습니다. 거기에는 윈도우와 맥을 위한 설치 파일도 있고, 미리 컴파일 된 리눅스 바이너리와 소스 코드가 얻을 수 있습니다. 리눅스 유저는 [여기](https://nodejs.org/en/download/package-manager/)에서 설명하듯 패키지 매니저로부터 설치가 가능합니다.

설치가 되었으면 버전을 확인해봅시다.

```bash
node -v
v6.10.0
```

Node.js 설치가 잘 된 것 같으면 함께 설치 된 npm도 확인해봅시다.

```bash
npm -v
3.10.10
```

## Node.js 모듈

Node.js의 모듈은 하나 혹은 여러 JavaScript 파일로 이루어졌으며 간단하거나 혹은 복잡한 기능들을 제공합니다. 이 모듈은 Node.js 애플리케이션을 통해 재사용이 가능합니다.

Node.js의 각 모듈은 각각의 context를 가지고 있으므로, 다른 모듈에 영향을 받거나 전역 공간을 오염시키지 않습니다. 그리고 각 모듈은 별개의 폴더에 별개의 .js 파일로 존재 할 수 있습니다.

Node.js는 [CommonJS 모듈 표준](http://requirejs.org/docs/commonjs.html)을 구현했습니다. CommonJS는 웹 서버, 데스크탑, 콘솔 애플리케이션을 위한 JavaScript 표준을 정의한 지원자들의 그룹입니다.

그 어떤 단수 단어도 복수 단어로 반환해주는 모듈을 만들어봅시다.

## GitHub 레포지터리 만들기

1. GitHub에서 **mypluralize**라는 새로운 레포지터리를 만듭니다. 이 때 **README** 체크박스를 선택하고 **Node**용 **.gitignore** 파일을 추가하고 **MIT** 라이센스를 선택합니다.
2. 로컬에 clone을 합니다.

```bash
git clone https://github.com/philipszdavido/mypluralize.git
```

## npm 패키지 초기화

새로운 모듈을 만들 때 package.json 파일에 패키지에 대한 설명을 추가해야 합니다.

```bash
npm init
```

이 명령은 `package.json` 파일을 만들기 위해 여러가지 질문을 던지는데, 이를 바탕으로 프로젝트의 모든 종속성을 작성합니다. 이 파일은 이후 개발 과정에서 생기는 추가적인 종속성이 있을 때 마다 업데이트 될 것입니다.

```text
name: (project-name) project-name
version: (0.0.0) 0.0.1
description: The Project Description
entry point: //leave empty
test command: //leave empty
git repository: //the repositories url
keywords: //leave empty
author: // your name
license: N/A
```

Node Package Manager로 프로젝트의 초기화 작업이 끝나면 프로젝트의 루트 디렉토리에 다음과 같은 `package.json` 파일이 만들어집니다.

```json
{
  "name": "project-name",
  "version": "0.0.1",
  "description": "Project Description",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "repository": {
    "type": "git",
    "url": "the repositories url"
  },
  "author": "your name",
  "license": "N/A"
}
```

초기화 과정에서 질문을 받아 만드는 과정을 생략하고 싶다면, 다음처럼 명령을 내립니다.

```bash
npm init -y
```

이 방법으로 초기화를 하면 옵션을 묻지 않고 모두 기본값을 이용합니다.

## npm 기본 설정

```bash
npm set init.author.name “Chidume Nnamdi”
npm set init.author.email “kurtwanger40@gmail.com”
npm set init.author.url “http://twitter.com/ngArchangel"
```

여러분의 인적 정보는 **~/.npmrc** 파일에 저장되고, npm 패키지를 초기화 할 때마다 이를 이용하게 됩니다. 따라서 매번 이 정보를 입력 할 필요가 없습니다.

## 의존 라이브러리(dependencies) 설치

이제 동작하는 Node 프로젝트가 생겼습니다. 그 폴더로 이동합니다.

```bash
cd mypluralize
```

package.json 파일은 이렇게 생겼습니다.

```json
{
  "name": "mypluralize",
  "version": "1.0.1",
  "description": "A Node.js module that returns the plural form of any noun",
  "main": "index.js",
  "repository": {
    "type": "git",
    "url": "git+https://github.com/philipszdavido/mypluralize.git"
  },
  "keywords": [],
  "author": "Chidume Nnamdi <kurtwanger40@gmail.com>",
  "license": "ISC",
  "bugs": {
    "url": "https://github.com/philipszdavido/mypluralize/issues"
  },
  "homepage":"https://github.com/philipszdavido/mypluralize#readme"
}
```

## typescript 설치

이제 typescript를 설치 할 때입니다.

```bash
npm i typescript -D
```

## tsconfig 파일 설정

`tsconfig.json` 파일을 만듭시다. `tsconfig.json` 파일이 위치한 디렉토리는 TypeSript 프로젝트의 루트 디렉토리가 됩니다. `tsconfig.json` 파일은 프로젝트를 컴파일하기 위한 루트 파일과 컴파일러 옵션들을 설정 할 수 있습니다.

파일은 다음과 같은 방식으로 수동으로 만들 수 있습니다.

```json
{
  "compilerOptions": {
    "target": "es5",
    "module": "commonjs",
    "declaration": true,
    "outDir": "./dist",
    "strict": true
  }
}
```

`tsc --init` 명령을 이용하면 `tsconfig.json` 파일을 자동 생성 할 수 있습니다. `tsc`를 전역적으로 사용하고 싶다면 다음 명령을 입력합니다.

```bash
npm i -g typescript -D
npm i -g typings -D
```

이제, `tsconfig.json` 파일을 만들어봅시다.

```bash
tsc --init
```

시스템에 TypeScript가 전역으로 설치하지 않고, 프로젝트에 한정적으로 사용하고 싶다면

```bash
npm i typescript -D
```

명령으로 프로젝트의 `node_modules` 폴더에 로컬로 설치 할 수 있습니다. 이렇게 하면 다음의 경로로 `tsc`를 참조 할 수 있습니다.

```bash
./node_modules/.bin/tsc --init
```

tsc는 `node_modules/.bin` 폴더에 cmd가 있으므로, 그냥 `tsc --init`을 이용 할 수 있게 됩니다.

이제 만들어진 `tsconfig.json`을 다시 확인해봅시다.

```json
{
  "compilerOptions": {
    "target": "es5",
    "module": "commonjs",
    "declaration": true,
    "outDir": "./dist",
    "strict": true
  }
}
```

> **"target": "es5"** => ECMAScript 타겟 버전을 설정합니다: 'ES3' (기본), 'ES5', 'ES2015', 'ES2016', 'ES2017', 'ESNEXT'<br/><br/>
**"module": "commonjs"** => 모듈 코드 생성 방식을 설정합니다: 'none', 'commonjs', 'amd', 'system', 'umd', 'es2015', 'ESNext'<br/><br/>
**"declaration": "true"** => 상응하는 'd.ts' 파일을 생성합니다.<br/><br/>
**"outDir": "dist"** => 결과물을 저장 할 디렉토리를 설정합니다.

## Node 모듈 작성

lib 폴더를 만들고 index.ts 파일을 추가합니다.

```bash
mkdir lib && touch index.ts
```

실제 코드를 작성하기 전에, npm 모듈인 pluralize의 도움을 받아야 하므로 추가로 설치해줍니다.

```bash
npm i pluralize -S
```

index.ts

```typescript
import * as pluralize from 'pluralize'

/**
* @Method: Returns the plural form of any noun.
* @Param: {string}
* @Return: {string}
*/
export function getPlural(str: any): string {
  return pluralize.plural(str)
}
```

이 코드가 IDE(VS Code, Atom, Sublimt Text) 등에서 type 에러를 발생시키는 것을 확인 할 수 있습니다.

```text
import * as pluralize from 'pluralize'

Could not find a declaration file for module 'pluralize'
```

이는 pluralize 모듈이 `.d.ts` 파일이 없기 때문입니다. 이 이슈를 해결하기 위해 다음 명령을 실행하세요.

```bash
npm i @types/pluralize -D
```

타입 정의 파일을 가져와서 `node_modules` 폴더의 `@types` 폴더에 위치시킵니다. 정의 파일을 가져온 다음에는 에러 메시지가 사라집니다.

## 프로젝트 빌드

코드를 빌드하는 스크립트를 `package.json`에 추가합니다. 

```json
{
  "name": "mypluralize",
  "version": "1.0.1",
  "description": "A Node.js module that returns the plural form of any noun",
  "main": "index.js",
  "scripts": {
    "build": "tsc"
  },
  "repository": {
    "type": "git",
    "url": "git+https://github.com/philipszdavido/mypluralize.git"
  },
  "keywords": [],
  "author": "Chidume Nnamdi <kurtwanger40@gmail.com>",
  "license": "ISC",
  "bugs": {
    "url": "https://github.com/philipszdavido/mypluralize/issues"
  },
  "homepage":"https://github.com/philipszdavido/mypluralize#readme"  ,
  "devDependencies": {
    "typescript": "^2.5.3"
  },
  "dependencies": {
    "@types/pluralize": "0.0.27",
    "pluralize": "^7.0.0"
  }
}
```

이제 다음 명령을 실행합니다.

```bash
npm run build
```

Voila! 코드가 JavaScript로 컴파일 되었습니다! dist라는 디렉토리가 만들어지고 그 안에 `index.js`와 `index.d.ts` 파일이 생성되었습니다. `index.js`는 우리가 작성한 코드가 JavaScript로 컴파일 된 것이고, `index.d.ts` 파일은 TypeScript에서 사용 할 수 있도록 우리 모듈의 타입이 정의 된 것입니다. `package.json` 파일을 조금 더 수정해봅시다.

> `main` 속성을 `dist/index.js`를 바라보도록 수정합니다.<br/>
`types` 속성을 만들고 `dist/index.d.ts`로 설정합니다.

```json
{
  "name": "mypluralize",
  "version": "1.0.1",
  "description": "A Node.js module that returns the plural form of any noun",
  "main": "dist/index.js",
  "types" : "dist/index.d.ts",
  "scripts": {
    "build": "tsc"
  },
  "repository": {
    "type": "git",
    "url": "git+https://github.com/philipszdavido/mypluralize.git"
  },
  "keywords": [],
  "author": "Chidume Nnamdi <kurtwanger40@gmail.com>",
  "license": "ISC",
  "bugs": {
    "url": "https://github.com/philipszdavido/mypluralize/issues"
  },
  "homepage":"https://github.com/philipszdavido/mypluralize#readme"  ,
  "devDependencies": {
    "typescript": "^2.5.3"
  },
  "dependencies": {
    "@types/pluralize": "0.0.27",
    "pluralize": "^7.0.0"
  }
}
```

## 테스트 코드 작성

Mocha 테스팅 프레임워크와 Chai assertion 라이브러리를 이용 할 것입니다.

```bash
npm i mocha -D
npm i chai -D
```

test 폴더를 만들고 test.js 파일을 추가합니다.

```bash
mkdir test && touch test/test/js
```

test.js

```javascript
'use strict';
var expect = require('chai').expect;
var index = require('../dist/index.js');

describe('getPlural function test', () => {
  it('should return Boys', () => {
    var result = index.getPlural('Boy');
    expect(result).to.equal('Boys');
  })
  it('should return Girls', () => {
      var result = index.getPlural('Girl');
      expect(result).to.equal('Girls');
  });
  it('should return Geese', () => {
      var result = index.getPlural('Goose');
      expect(result).to.equal('Geese');
  });
  it('should return Toys', () => {
      var result = index.getPlural('Toy');
      expect(result).to.equal('Toys');
  });
  it('should return Men', () => {
      var result = index.getPlural('Man');
      expect(result).to.equal('Men');
  });
})
```

## 테스트 구동

`package.json`에 테스트 스크립트를 추가합니다.

```json
"scripts": {
  "build": "tsc",
  "test": "mocha --reporter spec"
}
```

이제 테스트 스크립트를 구동합니다.

```bash
npm run test
```

![](https://cdn-images-1.medium.com/max/800/1*rsvn0ZQGEqQ5wta9-S7yDA.jpeg)

## README 작성

````
# mypluralize
A Node.js module that returns the plural form of any noun
## Installation 
```sh
npm install mypluralize --save
yarn add mypluralize
bower install pluralize --save
```
## Usage

### Javascript
```javascript
var pluralise = require('mypluralize');
var boys = pluralise.getPlural('Boy');
```
```sh
Output should be 'Boys'
```
### TypeScript
```typescript
import { getPlural } from 'mypluralize';
console.log(getPlural('Goose'))
```
```sh
Output should be 'Geese'
```
### AMD
```javascript
define(function(require,exports,module){
  var pluralise = require('mypluralize');
});
```
## Test 
```sh
npm run test
```
````

## Git으로 commit, push

```bash
git add .
git commit -m "Initial release"
git tag v1.0.1
git push origin master --tags
```

## npm으로 퍼블리시

모듈이 설치 될 때, 불필요한 폴더나 파일들이 있을 수 있습니다. 코드를 퍼블리시 하기 전에 이것들을 제외시켜야 합니다. lib 폴더는 퍼블리시 되어서는 안됩니다. `.npmignore` 파일을 만들고 다음 내용을 추가하세요.

.npmignore

```bash
lib/
```

이제 다음 명령을 입력해서 모듈을 퍼블리시 합니다.

```bash
npm publish
```

## 참고 사항

- 같은 이름의 패키지가 이미 존재하면 안됩니다.

퍼블리시를 하기 위해서는 npm 레지스트리에 유저가 존재해야 합니다. 만약 이미 만들어둔게 없다면 `npm adduser` 명령을 통해 만들어둡니다. 만들고 나서는 `npm login`으로 클라이언트에 인증 정보를 저장합니다.

테스트: `npm config ls`를 실행해서 클라이언트에 인증 정보가 잘 저장되었는지 확인 할 수 있습니다. [https://npmjs.com/~](https://npmjs.com/~)에 방문하면 레지스트리에 추가가 되었는지도 확인이 가능합니다.

모두 문제 없이 처리되었다면 cmd에 다음처럼 보여져야 합니다.

![](https://cdn-images-1.medium.com/max/800/1*MBhIl9QM8XH_crBvL6w60g.jpeg)

## 지속적인 통합(Continuous Integration) 추가하기

지속적인 통합을 위해 Travis CI를 이용하겠습니다. 다음 단계를 수행하여 프로젝트에 Travis CI를 연동하세요.

1. [Travis CI](https://travis-ci.org/)를 방문합니다.

![](https://cdn-images-1.medium.com/max/800/1*QZPENNZ3idS4BJm2UycxZA.png)

2. 'Sign in with Github'을 클릭합니다.

![](https://cdn-images-1.medium.com/max/800/1*dcQgbrDs4f0f8QV6HPk4dw.png)

3. 통합을 원하는 레포지터리를 체크합니다.

![](https://cdn-images-1.medium.com/max/800/1*420eF2yzmYKw3w5UXRPDOA.png)

4. `.travis.yml`을 레포지터리에 추가합니다.

```yaml
language : node_js
node_js :
 - stable
install:
 - npm install
script:
 - npm test
```

5. git에 commit하고 push합니다.

git에 push를 하면 첫 번째 빌드가 동작합니다. Travis에 로그인해서 빌드 상태를 확인합니다.

![](https://cdn-images-1.medium.com/max/800/1*zIyEwgUDZGFzilpgNrKksw.png)

## 커버리지(coverage) 데이터 추가

[Coveralls](https://coveralls.io/)는 소스 코드에 대한 테스트 정도를 분석해주는 호스팅 분석 웹 서비스입니다. Coveralls는 지속적으로 새로운 코드 커버리지를 포함시켜 추적해줍니다.

Coveralls Cloud를 이용하기 위해서는 아래의 조건이 필수적입니다.
(Coveralls 엔터프라이즈는 다양한 레포지터리-호스팅 옵션을 제공합니다.)

- **코드는 반드시 [Github](http://github.com/) 혹은 [BitBucket](http://bitbucket.org/)에서 호스팅 되어야 합니다.**

Coveralls 서비스는 언어와 CI 도구에 독립적입니다만, 레포지터리 호스팅만큼 모든 가능성을 위한 쉬운 솔루션을 아직 구축하지 못했습니다. 계정을 만드는 것은 빠르고 간단합니다. 사용할 git 서비스(나중에 계정을 연동할 수 있습니다)의 "Sign in" 버튼을 클릭하고, Coveralls에 인증을 해서 레포지터리에 접근합니다. 입력해야 할 양식 같은 건 없습니다.

GitHub 계정으로 Coveralls에 로그인하면 "**ADD REPO**" 버튼을 클릭하고 코드 커버리지 통계를 원하는 레포지터리를 활성화시킵니다.

![](https://cdn-images-1.medium.com/max/800/1*_ZaPrud44---x9qrGjmlLA.jpeg)

## Istanbul로 코드 커버리지 얻기

코드 커버리지를 분석하는 도구는 시중에 많이 있습니다만, [istanbul](https://gotwarlost.github.io/istanbul/)은 간단하고 효과적이어서, 이걸로 선택을 했습니다.

먼저, istanbul을 `devDependency`로 설치합니다:

```bash
npm i istanbul -D
npm i coveralls -D
```

이제 코드 커버리지를 위해 스크립트를 추가합니다.

```json
"cover": "istanbul cover node_modules/mocha/bin/_mocha test/*.js -- -R spec"
```

이제 `package.json`에는 테스트를 위한 두 가지 스크립트가 모두 포함되어 있습니다:

```json
"scripts": {
    "test": "mocha --reporter spec",
    "cover": "istanbul cover node_modules/mocha/bin/_mocha test/*.js -- -R spec"
  },
```

`.travis.yml` 파일에 이를 반영해서 업데이트해줍니다.

```yaml
language : node_js 
node_js :  
 - stable 
install:  
 - npm install 
script:  
 - npm run cover 
# Send coverage data to Coveralls
after_script: "cat coverage/lcov.info | node_modules/coveralls/bin/coveralls.js"
```

이제 다음 명령으로 테스트를 실행하고 코드 커버리지 통계를 얻어낼 수 있습니다

```bash
npm run cover
```

![](https://cdn-images-1.medium.com/max/800/1*Z5Q_8GKKGo2BWxJuQFbFbw.jpeg)

이제 GitHub으로 **commit**하고 **push**하세요.

Travis는 Istanbul을 불러내서, `lcovonly` 코드 커버리지 리포트를 구동하고, 그 정보를 `coverage/lcov.info`에 저장한 후, Coveralls 라이브러리로 데이터를 pipe 시킵니다.

Coveralls에 로그인해서 모든 것들이 잘 처리되었는지 살펴보세요.

![](https://cdn-images-1.medium.com/max/800/1*oaz6bSNrrmeir_n-zlbI1Q.jpeg)

![](https://cdn-images-1.medium.com/max/800/1*be4LmmF5fSzDRQtTeWV-WQ.jpeg)

## 뱃지 추가

이제 이것들을 Git과 npm 레포지토리로 가져와봅시다.

### Travis

Travis CI의 레포지토리 옆에 있는 **settings** 토글 버튼을 누르고, 뱃지 아이콘을 클릭합니다.

**Markdown**을 선택하고, 코드를 README에 추가합니다.

![](https://cdn-images-1.medium.com/max/800/1*gngr9-5ZuE23BiOIHWp5bg.jpeg)

### Coveralls

Coveralls에 로그인하고, 레포지터리를 누른 후 "**BADGE URLS**" 드롭다운을 클릭합니다. Markdown 코드를 **README**에 추가합니다.

![](https://cdn-images-1.medium.com/max/800/1*f7oK0wqZsE7ELpu9kg57Zg.jpeg)

GitHub에 **commit**하고 **push** 합니다.

Travis에서 테스트를 구동할 때, 여러분은 GitHub README에서 빌드 상태를 확인 할 수 있습니다.

![](https://cdn-images-1.medium.com/max/800/1*WiF_BOW6AWl0j0nKz8YMNA.jpeg)

npm에도 추가해야 합니다. 이 변경 사항을 npm에 전달하는 방법은 새 버전을 릴리즈 하는 것 뿐입니다.

```bash
npm version patch -m "Version %s - add sweet badges"
```

%s = 새 버전 번호

이 명령은 **package.json**의 버전 번호를 올려주고, 새 커밋을 추가한 후 이 릴리즈 번호로 태깅을 해줍니다.

**참고**: **npm version** 작업을 하기 전에, Git 작업 디렉토리는 깨끗한 상태여야 합니다.

버전업을 하고 나면 다음 명령을 실행합니다.

```bash
git push && git push --tags (혹은 git push origin master --tags)
npm publish
```

이제 여러분의 npm 모듈 페이지에 가보시면 새 릴리즈와 함께 두 개의 예쁜 뱃지를 확인하실 수 있습니다.

## 결론

이렇게 TypeScript로 npm 모듈을 작성하고 퍼블리시 해 보았습니다. 여기서 우리가 달성한 대단한 점은, 모듈을 실행하지 않고도 JavaScript 혹은 TypeScript 프로젝트에서 완벽하게 사용 할 수 있다는 것입니다.

```bash
npm i @types/mypluralize -D
```

어떤 문제라도 있으면, 저에게 연락을 주십시요.

- [Facebook](https://facebook.com/philip.david.5011)
- [Twitter](https://twitter.com/ngArchangel)
- [Gmail](http://kurtwanger40@gmail.com/)
- [Github](https://github.com/philipszdavido)

## Github 코드

- 전체 소스 코드 [Github 레포지터리](https://github.com/philipszdavido/mypluralize)