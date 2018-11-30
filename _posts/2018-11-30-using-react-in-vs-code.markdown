---
layout: post
title: "[번역] Visual Studio Code에서 React 사용하기"
date: 2018-11-30 16:57:53
author: Reid
categories:
  - engineering
tags:
  - 프로그래밍
  - 비주얼스튜디오
  - IDE
  - vs code
  - react
---
> 참고: 개인적인 공부 차원에서 번역한 문서다보니 오역이 있을 수 있습니다. 보다 정확한 내용은 [원문](https://code.visualstudio.com/docs/nodejs/reactjs-tutorial)에서 확인해주세요.

# VS Code에서 React 사용하기

[React](https://facebook.github.io/react/)는 페이스북에서 만든 웹 애플리케이션 유저 인터페이스를 만드는 유명한 자바스크립트 라이브러리입니다. 비주얼 스튜디오 코드는 React.js 인텔리센스와 코드 탐색 기능을 지원합니다.

![Welcome to React](https://code.visualstudio.com/assets/docs/nodejs/reactjs/welcome-to-react.png)

## React에 오신 것을 환영합니다.

이 튜토리얼에서는 `create-react-app` [generator](https://facebook.github.io/react/docs/installation.html#creating-a-new-application)를 사용 할 것입니다. generator를 설치하고 사용하는 것과 함께 React 애플리케이션 서버를 구동하기 위해서는, [Node.js](https://nodejs.org/) 자바스크립트 런타임과 npm(Node.js 패키지 관리자)이 설치되어 있어야 합니다. [npm](https://www.npmjs.com/)은 Node.js에 포함되어 있으며, 이 [링크](https://nodejs.org/en/download/)로 설치 할 수 있습니다.

> 팁: Node.js와 npm이 정상적으로 설치되었는지 확인하려면, `node --version`과 `npm --version`을 입력해보세요.

`create-react-app` generator를 설치하려면, 터미널이나 명령행에서 다음을 입력하세요:

```sh
npm install -g create-react-app
```

이 작업은 설치하는데 몇 분 정도 걸릴 수 있습니다. 이제 다음을 입력해서 새로운 React 애플리케이션을 만들 수 있습니다:

```sh
create-react-app my-app
```

`my-app`은 애플리케이션의 폴더 이름입니다. React 애플리케이션을 만들고, 의존 라이브러리들을 설치하는데 몇 분 정도 걸릴 수 있습니다.

어서 React 애플리케이션을 실행해봅시다. 새로운 폴더로 이동해서 `npm start`를 입력하면 웹 서버가 시작되고 브라우저에 애플리케이션이 열립니다.

```sh
cd my-app
npm start
```

브라우저에 `http://localhost:3000` 주소에 "Welcome to React"라고 적힌 문구를 볼 수 있어야 합니다. VS Code로 애플리케이션을 보기 위해서 얼마간 웹 서버를 띄워놓은채로 두겠습니다.

VS Code로 React 애플리케이션을 열기 위해서는, 다른 터미널(혹은 명령행)을 띄우고 `my-app` 폴더로 이동한 후 `'code .'`를 입력하세요:

```sh
cd my-app
code .
```

### Markdown 미리보기

파일 탐색기에서 찾을 수 있는 하나의 파일은 `README.md` 마크다운 파일입니다. 이 파일은 보통은 애플리케이션이나 React에 대한 여러 대단한 정보들을 담고 있습니다. README를 리뷰하는 좋은 방법은 VS Code [Markdown 미리보기(Preview)](https://code.visualstudio.com/docs/languages/markdown#_markdown-preview)를 이용하는 것입니다. 미리보기는 현재 에디터 그룹(Markdown: Open Preview `⇧⌘V`)에서 여는 것과 새로운 에디터 그룹 측면(Markdown: Open Preview to the Side `⌘K V`)에 여는 것 모두 가능합니다. 훌륭한 포매팅과 헤딩으로의 하이퍼링크 이동, 코드 블럭의 구문 하이라이트 등을 얻을 수 있습니다.

![Markdown preview](https://code.visualstudio.com/assets/docs/nodejs/reactjs/markdown-preview.png)

### 구문 하이라이트와 괄호 매칭

`src` 폴더를 확장하고 `index.js` 파일을 선택하세요. 여러 소스 코드의 요소들에 구문 하이라이트가 되어 있고, 괄호에 커서를 대면 대응하는 괄호가 선택되는 것을 확인 할 수 있습니다.

![Syntax highlighting and bracket matching](https://code.visualstudio.com/assets/docs/nodejs/reactjs/bracket-matching.png)

### 인텔리센스(IntelliSense)

`index.js`에 타이핑을 시작하면, 다음과 같은 똑똑한 제안(suggestions) 혹은 완성문(completions)을 볼 수 있습니다.

![IntelliSense](https://code.visualstudio.com/assets/docs/nodejs/reactjs/suggestions.png)

제안을 선택한 다음 `.`를 입력하면, IntelliSense를 통해 타입과 메소드들을 확인이 가능합니다.

![IntelliSense-methods](https://code.visualstudio.com/assets/docs/nodejs/reactjs/intellisense.png)

VS Code는 자바스크립트 코드 인텔리전스(intelligence)에 TypeScript 언어 서비스를 이용하는데 이를 [Automatic Type Acquisition](https://code.visualstudio.com/docs/languages/javascript#_automatic-type-acquisition) (ATA)라고 부릅니다. ATA는 `package.json`에서 참조하는 npm 모듈에 대한 npm Type Declaration 파일들(`*.d.ts`)을 가져옵니다.

메소드를 선택하면, 파라메터에 대한 도움말을 보게 됩니다:

![IntelliSense-paramters](https://code.visualstudio.com/assets/docs/nodejs/reactjs/parameter-help.png)

### 정의로 이동(Go to Definition), 정의 피킹(Peek definition)

VS Code는 TypeScript 언어 서비스를 통해 에디터에 타입 정의에 대한 정보를 제공 할 수 있습니다. **정의로 이동(Go to Definition)**(F12) 혹은 **정의 피킹(Peek Definition)**(⇧⌘F12)를 통해 가능합니다. `App`위에 커서를 올려놓고 마우스 오른쪽 클릭을 한 후, **정의 피킹**을 선택하세요. [Peek 창](https://code.visualstudio.com/docs/editor/editingevolved#_peek)이 뜨고 `App.js`로부터 `App`의 정의(definition)가 보여질 것입니다.

![Peek definition](https://code.visualstudio.com/assets/docs/nodejs/reactjs/peek-definition.png)
Peek 창을 끄려면 Escape 버튼을 누르세요.

## Hello World!

샘플 애플리케이션을 "Hello World!"로 수정해봅시다. 링크를 추가하여 새 H1 헤더를 선언하고 `ReactDOM.render`의 `<App />` 태그를 `element`로 바꿉니다.

```javascript
import React from 'react'
import ReactDOM from 'react-dom'
import App from './App'
import registerServiceWorker from './registerServiceWorker'
import './index.css'

var element = React.createElement('h1', {className: 'greeting'}, 'Hello, world!')
ReactDOM.render(element, document.getElementById('root'))
registerServiceWorker()
```

`index.js`를 저장하면, 실행중인 서버 인스턴스가 웹 페이지를 업데이트하여 "Hello World!"를 출력할 것입니다.

> 팁: VS Code는 기본적으로 파일 자동 저장을 지원합니다. 자동 저장을 켜려면 **파일(File)** 메뉴의 **자동 저장(Auto Save)** 옵션을 체크하거나 user [setting](https://code.visualstudio.com/docs/getstarted/settings)의 `files.autoSave`를 직접 설정하세요.

![Hello World](https://code.visualstudio.com/assets/docs/nodejs/reactjs/hello-world.png)

## React 디버그하기

클라이언트 쪽의 React 코드를 디버깅 하려면, [Debugger for Chrome](https://marketplace.visualstudio.com/items?itemName=msjsdiag.debugger-for-chrome) 확장(extension)을 설치해야 합니다.

> 참고: 이 튜토리얼은 크롬 브라우저를 설치했다고 가정합니다. Debugger for Chrome 확장의 개발자는 [Safari on iOS](https://marketplace.visualstudio.com/items?itemName=msjsdiag.debugger-for-ios-web)나 [Edge](https://marketplace.visualstudio.com/items?itemName=msjsdiag.debugger-for-edge) 브라우저 버전 역시 제공합니다.

확장 뷰(⇧⌘X)를 열고 검색창에 'chrome'을 입력하세요. Chrome을 참조하는 확장들이 여러개 보일 것입니다.

![Debugging React](https://code.visualstudio.com/assets/docs/nodejs/reactjs/debugger-for-chrome.png)

**Debugger for Chrome**의 **설치** 버튼을 누르세요. 이 버튼이 **설치중**으로 바뀌고, 설치가 완료되면 **다시 로드**로 바뀝니다. **다시 로드**를 누르면 VS Code가 재시작되고 확장이 활성화됩니다.

### 브레이크포인트 설정하기

`index.js`에 브레이크포인트를 설정하려면, 줄번호 왼쪽의 빈 공간을 클릭하면 됩니다. 이렇게 하면 브레이크포인트가 설정되며 빨간 원 모양이 보일 것입니다.

![Set a breakpoint](https://code.visualstudio.com/assets/docs/nodejs/reactjs/breakpoint.png)

### 크롬 디버거 설정하기

[디버거](https://code.visualstudio.com/docs/editor/debugging)에 초기 설정을 합니다. 디버거 뷰(⇧⌘D)로 가서 디버거 설정 파일인 `launch.json`을 생성하기 위해 톱니 버튼을 클릭합니다. **Select Environment** 드롭다운 목록에서 **Chrome**을 선택합니다. 이렇게 하면 웹사이트를 실행 할 설정이 포함된 프로젝트 폴더에 `.vscode` 폴더를 만들고 그 안에 `launch.json` 파일을 생성합니다.

예제를 위해 한가지 변경이 필요합니다: `url` port를 `8080`에서 `3000`으로 바꿔줍니다. `launch.json`은 이렇게 생겼습니다:

```json
{
  "version": "0.2.0",
  "configuration": [
    {
      "type": "chrome",
      "request": "launch",
      "name": "Launch Chrome against localhost",
      "url": "http://localhost:3000",
      "webRoot": "${workspaceFolder}"
    }
  ]
}
```

개발 서버가 실행 중인지("npm start") 확인하세요. 그리고 나서 F5키나 녹색 화살표를 눌러서 디버거를 띄우고 웹 브라우저를 엽니다. 브레이크포인트가 설정된 소스 코드는 디버거가 붙기 전인 구동중에 이미 실행되었기 때문에, 페이지를 다시 리프레시 하지 않는 한 브레이크포인트에 걸리는 일은 없을 겁니다. 페이지를 리프레시 시키면 이제는 브레이크포인트에 걸려야 합니다.

![chrome debugger configuration 1](https://code.visualstudio.com/assets/docs/nodejs/reactjs/hit-breakpoint.png)

소스 코드를 따라 단계별로 진행(F10) 할 수 있고, `element`와 같은 변수 값을 알아 낼 수도 있으며, 클라이언트 쪽 Ract 애플리케이션의 콜 스택도 확인이 가능합니다.

![chrome debugger configuration 2](https://code.visualstudio.com/assets/docs/nodejs/reactjs/debug-variable.png)

**Debugger for Chrome** 확장의 README에는 소스맵으로 작업하기나 문제 해결 등 다른 설정에 대한 많은 내용을 확인할 수 있습니다. 해당 확장 페이지에서 **세부 정보** 탭을 눌러서 VS Code 내에서 직접 읽어볼 수도 있습니다.

![chrome debugger configuration 3](https://code.visualstudio.com/assets/docs/nodejs/reactjs/chrome-debugger-readme.png)

### 라이브 수정과 디버깅

React 앱과 함께 [webpack](https://webpack.js.org/)을 사용중이라면, webpack의 HMR 메커니즘의 장점을 이용해 훨씬 효율적인 워크플로우를 경험 할 수 있습니다. 이 메커니즘은 라이브 수정과 VS Code에서 직접 디버깅을 하는 것을 가능하게 해줍니다. 더 자세한 내용은 [라이브 수정과 VS Code에서 React 앱을 직접 디버그하기](https://medium.com/@auchenberg/live-edit-and-debug-your-react-apps-directly-from-vs-code-without-leaving-the-editor-3da489ed905f) 블로그 포스트를 참고하세요.

## Linting

Linter는 소스 코드를 분석하여 애플리케이션이 실행되기 전에 잠재적인 문제들을 경고해줍니다. VS Code에 포함되어 있는 자바스크립트 언어 서비스에는 기본적으로 구문 오류를 확인하는 기능을 지원하고, 이는 **문제**패널 (**보기 > 문제** ⇧⌘M)에서 실제로 동작하는 것을 볼 수 있습니다.

React 소스 코드에 살짝 오류를 만들어 보면, 빨간 표식이 생기고 **문제** 패널에 에러가 표시됩니다.

![Linting 1](https://code.visualstudio.com/assets/docs/nodejs/reactjs/js-error.png)

Linter는 정교한 분석, 코딩 규칙의 적용, 안티 패턴 탐지 등의 기능을 제공합니다. 유명한 자바스크립트 linter는 ESLint 입니다. ESLint가 ESLint VS Code 확장과 결합하면 훌륭한 Lint 경험을 제공 할 것입니다.

먼저 ESLint 명령행 도구를 설치합니다.

```sh
npm install -g eslint
```

그리고 확장 뷰로 이동 후, 'eslint'를 입력해서 ESLint 확장을 설치합니다.

![Linting 2](https://code.visualstudio.com/assets/docs/nodejs/reactjs/eslint-extension.png)

ESLint 확장이 설치되고 VS Code가 재시작되면, ESLint 설정 파일인 `.eslintrc.json 파일을 만듭니다. 명령 팔레트(Command Palette)(⇧⌘P)에서 **ESLint: Create '.eslintrc.json' File** 명령을 이용해서 생성할 수 있습니다.

![Linting 3](https://code.visualstudio.com/assets/docs/nodejs/reactjs/create-eslintrc.png)

이 명령은 프로젝트 루트 폴더에 `.eslintrc.json` 파일을 생성합니다.

```json
{
    "env": {
        "browser": true,
        "commonjs": true,
        "es6": true,
        "node": true
    },
    "parserOptions": {
        "ecmaFeatures": {
            "jsx": true
        },
        "sourceType": "module"
    },
    "rules": {
        "no-const-assign": "warn",
        "no-this-before-super": "warn",
        "no-undef": "warn",
        "no-unreachable": "warn",
        "no-unused-vars": "warn",
        "constructor-super": "warn",
        "valid-typeof": "warn"
    }
}
```

ESLint는 열려 있는 파일들을 분석하고 `index.js`에 'App'이 정의 되었으나 한번도 사용되지 않았다는 경고를 보여줍니다.

![Linting 4](https://code.visualstudio.com/assets/docs/nodejs/reactjs/app-is-unused.png)

ESLint의 [규칙]()을 수정 할 수 있습니다. ESLint 확장도 `.eslintrc.json`에서 InteliSense를 제공합니다.

![Linting 5](https://code.visualstudio.com/assets/docs/nodejs/reactjs/eslintrc-intellisense.png)

추가 세미콜론에 대한 에러 규칙을 추가해봅시다:

```json
"rules": {
    "no-const-assign": "warn",
    "no-this-before-super": "warn",
    "no-undef": "warn",
    "no-unreachable": "warn",
    "no-unused-vars": "warn",
    "constructor-super": "warn",
    "valid-typeof": "warn",
    "no-extra-semi":"error"
}
```

이제 실수로 한 라인에 여러 세미콜론을 입력하면, 에디터에 에러(빨간 표식)가 보여지고, **문제** 패널에 에러 항목이 나타납니다.

![Linting 6](https://code.visualstudio.com/assets/docs/nodejs/reactjs/extra-semi-error.png)

## 유명한 스타터 키트(Starter kits)

이 튜토리얼에서는 `create-react-app` generator를 사용하여 간단한 React 애플리케이션을 작성하였습니다. 첫 번째  React 애플리케이션을 만드는데 도움이 되는 훌륭한 샘플이나 스타터 키트들이 많이 있습니다.

### VS Code React 샘플
올해 //Build 컨퍼런스에서 [데모](https://channel9.msdn.com/events/Build/2017/T6078)로 사용하기 위해 만들어진 [샘플](https://github.com/Microsoft/vscode-react-sample) React 애플리케이션입니다. 이 샘플은 간단한 TODO 애플리케이션을 만들고, Node.js [Express](https://expressjs.com/) 서버의 소스 코드도 포함합니다. 또한 [Babel](https://babeljs.io/) ES6 transpiler를 사용하고 [webpack](https://webpack.js.org/)을 이용해 사이트의 자원을 묶는 방법을 보여줍니다.

### MERN Stater
완전한 MERN (MongoDB, Express, React, Node.js) 스택 예제를 보려면 [MERN Starter](http://mern.io/)를 보세요. [MongoDB](https://docs.mongodb.com/v3.0/installation/)를 설치하고 실행해야 하지만, 빠르게 MERN 애플리케이션을 실행시킬 수 있습니다. Node.js 서버 디버깅을 설정하는 상세한 내용 등, 유용한 VS Code 특화 문서들이 [vscode-recipes](https://github.com/Microsoft/vscode-recipes/tree/master/MERN-Starter)에 있습니다.

### TypeScript React
TypeScript와 React에 대해 궁금하다면 TypeScript 버전의 `create-react-app` 애플리케이션을 만들어 볼 수 있습니다. 자세한 내용은 [TypeScript Quick Start](https://www.typescriptlang.org/samples/index.html) 사이트인 [TypeScript-React-Starter](https://github.com/Microsoft/TypeScript-React-Starter)를 참고하세요.

### Angular
[Angular](https://angular.io/)는 또 다른 인기있는 웹 프레임워크입니다. VS Code로 Angular 작업을 하는 예제를 보려면 [Chrome Debugging with Angular CLI](https://github.com/Microsoft/vscode-recipes/tree/master/Angular-CLI) recipe를 확인하세요. Angular 애플리케이션을 만들고 [Debugging for Chrome](https://marketplace.visualstudio.com/items?itemName=msjsdiag.debugger-for-chrome)의 `launch.json` 파일을 구성하는 과정을 설명합니다.

## 일반적인 질문들

**Q: 선언적(declarative) JSX에도 IntelliSense를 쓸 수 있나요?**
**A**: 네, 예를 들어 `create-react-app` 프로젝트의 `App.js` 파일을 띄우면, `render()` 메소드 안의 React JSX에 IntelliSense가 사용되는 것을 볼 수 있습니다.