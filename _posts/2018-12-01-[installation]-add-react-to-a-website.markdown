---
layout: post
title: "[번역] React 설명서, 설치편: 웹사이트에 React 추가하기"
date: 2018-12-02 10:51:53
author: Reid
categories:
  - engineering
tags:
  - 프로그래밍
  - react
  - javascript
published: false
---
> 이 문서는 버전 16.6.3의 [React Docs](https://reactjs.org/docs/add-react-to-a-website.html) 문서를 번역한 것입니다. 전문 번역이 아닌 개인 학습 목적의 작업 결과이므로 오역이 있을 수 있습니다. 그런 부분을 발견하시면 너그러히 이해해주시고 지적해주시면 고맙겠습니다.

# 웹사이트에 React 추가하기

*React를 원하는 만큼만 사용하기*

React는 점진적으로 적용 할 수 있게 설계되었으므로, **필요한 만큼만 적용하는 것이 가능**합니다. 이미 만들어둔 페이지에 약간의 인터랙션만 추가하고 싶다면, React 컴포넌트는 훌륭한 선택이 될 것입니다.

대부분의 웹사이트는 단일 페이지 앱(single-page apps)이 아니고, 될 필요도 없습니다. **빌드하는데 별도의 도구가 필요없는 몇 줄의 코드**로 여러분의 웹사이트의 일부에 React를 적용해보세요. 그 후에는 조금씩 더 그 영역을 확장해갈수도 있고, 그대로 몇개의 동적 위젯에 포함시키는 정도로 둘 수도 있습니다.

---

- [일분안에 React 추가하기](#일분안에-react-추가하기)
- [옵션: JSX로 React 작성해보기](#옵션-jsx로-react-작성해보기)(bundler가 따로 필요 없습니다!)

---

## 일분안에 React 추가하기

이 섹션에서 우리는 이미 만들어진 HTML 페이지에 React 컴포넌트를 어떻게 추가하는지 알아 볼 것입니다. 여러분 자신의 웹사이트여도 되고, 연습을 위해서 새 HTML 파일을 만들어도 됩니다.

이 작업에는 복잡한 도구나 추가 인스톨이 필요없습니다 -- **이 섹션을 수료하기 위해 여러분에게 필요한 것은 단지 인터넷과 시간입니다.**

옵션: [전체 예제 다운로드 (2KB)](https://gist.github.com/gaearon/6668a1f6986742109c00a581ce704605/archive/f6c882b6ae18bde42dcf6fdb751aae93495a2275.zip)

### 1단계: HTML에 DOM 컨테이너 추가하기

우선 HTML 페이지를 열고, React로 뭔가 보여주고 싶은 그 영역에 비어있는 `<div>` 태그 하나를 추가합니다. 예를 들어:

```jsx
<!-- ... 기존 HTML ... -->

<div id="like_button_container"></div>

<!-- ... 기존 HTML ... -->
```

`<div>`에 유니크한 `id` 속성을 부여했습니다. 이렇게 해두면 나중에 자바스크립트 코드에서 이 `id`로 `<div>`를 찾아내서, React 컴포넌트를 그릴 수 있게 됩니다.

> Tip<br/>
`<body>` 태그 **어디에나** 이런 식으로 "컨테이너" `<div>`의 배치가 가능합니다. 여러분이 필요한만큼 페이지안에 개별 DOM 컨테이너들을 만들어 둘 수 있습니다. 보통 그것들은 비어있겠지만, 내용이 들어있다면 React는 그것들을 변경합니다.

### 2단계: 스크립트 태그 추가하기

다음으로, `</body>` 태그의 마지막 부분에 3개의 `<script>` 태그를 추가합니다:

```html
  <!-- ... 생략된 HTML ...-->

  <!-- React 불러오기. -->
  <!-- 참고: 배포 시에, "development.js"를 "production.js"로 변경하세요. -->
  <script src="https://unpkg.com/react@16/umd/react.development.js" crossorigin></script>
  <script src="https://unpkg.com/react-dom@16/umd/react-dom.development.js" crossorigin></script>

  <!-- React 컴포넌트 불러오기. -->
  <script src="like_button.js"></script>

</body>
```

앞선 두 태그는 React를 불러오고, 마지막 태그는 여러분의 컴포넌트 코드를 불러오게 됩니다.

### 3단계: React 컴포넌트 만들기

HTML 페이지와 같은 위치에 `like_button.js` 파일을 생성합니다.

[스타터 코드](https://cdn.rawgit.com/gaearon/0b180827c190fe4fd98b4c7f570ea4a8/raw/b9157ce933c79a4559d2aa9ff3372668cce48de7/LikeButton.js)를 열고 그 내용을 새로 만든 파일에 붙여넣습니다.

> Tip<br/>
이 코드는 `LikeButton`이라는 React 컴포넌트를 정의합니다. 아직 뭐가 뭔지 이해가 안되더라도 걱정하지 마세요. 이후에 [튜토리얼](https://reactjs.org/tutorial/tutorial.html)과 [주요 개념들](2018-12-01-[main-concepts]-hello-world.markdown) 설명에서 React의 빌딩 블록에 대해 다룰 것입니다. 지금은, 그냥 화면에 뭔가 나오게만 해보는 겁니다.

`like_button.js`파일의 [스타터 코드]() 아랫쪽에 다음 두 줄을 추가합니다:

```jsx
// ... 스타터 코드 ...

const domContainer = document.querySelector('#like_button_container');
ReactDOM.render(e(LikeButton), domContainer);
```

이 두 라인의 역할은 아까 HTML 페이지에 추가한 그 `<div>`를 찾아서 그 안에 React 컴포넌트인 "Like" 버튼을 보여주는 것 입니다.

### **끝났습니다!**

4단계는 없습니다. **여러분은 방금 웹사이트에 첫 번째 React 컴포넌트를 추가했습니다.**

다음 섹션에서는 React를 도입하는 다양한데 유용한 팁들을 제공합니다.

[전체 예제 소스 코드 보기](https://gist.github.com/gaearon/6668a1f6986742109c00a581ce704605)

[전체 예제 다움로드 받기 (2KB)](https://gist.github.com/gaearon/6668a1f6986742109c00a581ce704605/archive/f6c882b6ae18bde42dcf6fdb751aae93495a2275.zip)

### Tip: 컴포넌트 재사용하기

일반적으로 HTML 페이지내의 여러곳에서 React 컴포넌트를 재사용하게 됩니다. 아래는 "Like" 버튼을 3곳에서 보여주는 것과 함께 파라메터도 전달하는 예제입니다.

[전체 예제 소스 코드 보기](https://gist.github.com/gaearon/faa67b76a6c47adbab04f739cba7ceda)

[전체 예제 다움로드 받기 (2KB)](https://gist.github.com/gaearon/faa67b76a6c47adbab04f739cba7ceda/archive/9d0dd0ee941fea05fd1357502e5aa348abb84c12.zip)

> Note<br/>
이 방법을 사용하면 React로 만든 부분들을 서로 독립적으로 분리시켜 컴포넌트화 할 수 있습니다. React 코드안에서는 [컴포넌트 컴포지션](2018-12-01-[main-concepts]-components-and-props.markdown#컴포넌트-컴포지션)이 더 편리합니다.

### Tip: 프로덕션 배포를 위해 자바스크릅트 minify하기

minify되지 않은 자바스크립트 코드는 페이지가 심각하게 느려 질 수 있다는 점을 꼭 기억하세요.

배포하는 HTML의 React를 불러 오는 파일명 끝에 `production.min.js`가 붙어 있는지 확인하세요. 그리고 여러분의 스크립트들을 이미 minify 해두었다면, 사이트는 프로덕션에 올릴 준비가 된 것입니다.

```html
<script src="https://unpkg.com/react@16/umd/react.production.min.js" crossorigin></script>
<script src="https://unpkg.com/react-dom@16/umd/react-dom.production.min.js" crossorigin></script>
```

스크립트를 minify하는 방법을 모른다면, [여기](https://gist.github.com/gaearon/42a2ffa41b8319948f9be4076286e1f3)서 설정하는 법을 익혀보세요.

---

## 옵션: JSX로 React 작성해보기

위에서 우리는 브라우저가 네이티브하게 지원하는 기능들에만 의존해서 예제들을 작성해보았습니다. 그래서 화면에 보여줄 인자를 React에 전달하기 위해 자바스크립트 함수를 호출해야만 했습니다.

```jsx
const e = React.createElement;

// "Like" <button> 보여주기
return e(
  'button',
  { onClick: () => this.setState({ liked: true })},
  'Like'
)
```

하지만, [JSX](2018-12-01-[main-concepts]-introducing-jsx.markdown)를 사용하는 또 다른 옵션도 있습니다:

```jsx
// "Like" <button> 보여주기
return (
  <button onClick={() => this.setState({ liked: true })}>
    Like
  </button>
)
```

위 두 코드는 동일하게 동작합니다. **JSX는 필수가 아닙니다**만, React와 다른 라이브러리들 모두에 대해 UI 코드를 작성하는데 도움이 되기 때문에 많은 사람들이 선호합니다. 

이 [online converter](http://babeljs.io/repl#?babili=false&browsers=&build=&builtIns=false&spec=false&loose=false&code_lz=Q&debug=false&forceAllTransforms=false&shippedProposals=false&circleciRepo=&evaluate=false&fileSize=false&sourceType=module&lineWrap=true&presets=es2015%2Creact%2Cstage-2%2Cstage-3&prettier=true&targets=Node-6.12&version=6.26.0&envVersion=)로 JSX를 시험해 볼 수 있습니다.

### JSX 빠르게 적용하기

페이지에 `<script>` 태그를 추가해서 JSX를 빠르게 적용해볼 수 있습니다:

```html
<script src="https://unpkg.com/babel-standalone@6/babel.min.js"></script>
```

이제 `type="text/babel"` 속성만 추가하면 어떤 `<script>` 태그에서도 JSX를 사용 할 수 있습니다. [JSX를 사용한 HTML 예제](https://raw.githubusercontent.com/reactjs/reactjs.org/master/static/html/single-file-example.html)를 다운로드해서 확인해볼 수 있습니다.

이 방법은 간단한 데모를 만들어서 학습하는데 도움이 됩니다. 하지만, 웹사이트가 느려질 수 있기 때문에 **프로덕션 환경에는 적합하지 않습니다**. 프로덕션에 적용해야 하는 시점에는 이 `<script>`태그와 `type="text/babel"` 속성을 모두 지우고, JSX preprocessor가 모든 `<script>` 태그를 자동으로 변환 할 수 있게 설정해줘야 합니다. 이에 대해 다음 섹션에서 배워 볼 것입니다.

### 프로젝트에 JSX 추가하기

JSX를 프로젝트에 추가하는데 bundler같은 복잡한 도구나 개발 서버는 필요하지 않습니다. 기본적으로, JSX를 추가하는 것은 **CSS preprocessor를 추가하는 것과 비슷**합니다. 유일한 요구사항은 여러분의 컴퓨터에 [Node.js](https://nodejs.org/)를 설치하는 것입니다.

터미널에서 프로젝트 폴더로 이동한 다음, 이 두 명령어를 입력합니다:

1. `npm init -y` 실행 (실패하는 경우, 이 [해결방법](https://gist.github.com/gaearon/246f6380610e262f8a648e3e51cad40d)을 고려])
2. `npm install babel-cli@6 babel-preset-react-app@3` 실행

> Tip<br/>
여기서 우리는 **JSX preprocessor를 설치하기 위해서만 npm을 사용**하고 있으며 다른 것들은 필요 없습니다. 이제 React와 애플리케이션 모두 그대로 `<script>` 태그 코드를 유지 할 수 있습니다.

축하합니다! 여러분은 방금 **프로덕션용 JSX 설정**을 프로젝트에 추가했습니다.

### JSX Preprocessor 구동하기

`src` 폴더를 만들고 다음 터미널 명령을 실행합니다:

```bash
npx babel --watch src --out-dir . --presets react-app/prod
```

> 참고<br/>
npx는 오타가 아닙니다. [npm 5.2+에서 제공하는 패키지 구동 도구](https://medium.com/@maybekatz/introducing-npx-an-npm-package-runner-55f7d4bd282b)입니다.<p/>
만약 "You have mistakenly installed the `babel` package"라는 오류 메시지를 본다면, 아마도 [이전 단계](#프로젝트에-jsx-추가하기)를 지나쳤을 가능성이 있습니다. 동일한 폴더에서 다시 명령을 수행해보세요.

명령이 끝나기를 기다릴 필요는 없습니다. 이 명령은 JSX에 대한 자동화된 watcher를 시작합니다.

[JSX 스타터 코드](https://cdn.rawgit.com/gaearon/c8e112dc74ac44aac4f673f2c39d19d1/raw/09b951c86c1bf1116af741fa4664511f2f179f0a/like_button.js)로 `src/like_button.js` 파일을 만들었다면, watcher는 preprocess된 `like_button.js`를 만들게 되는데 이 파일은 브라우저에 적합하도록 plain 자바스크립트로 만들어집니다. JSX로 소스 파일을 수정하면 watcher는 자동으로 변환을 시작합니다.

여기에 더해서, 이 기능은 클래스 같은 최신 자바스크립트 문법을 구형 브라우저와의 호환 걱정 없이 사용할 수 있게 해줍니다. 이 도구의 이름은 Babel이며, [Babel 문서](http://babeljs.io/docs/en/babel-cli/)에서 더 자세한 내용을 확인할 수 있습니다.

이 빌드 도구들에 익숙해졌다면, [다음 섹션](2018-12-02-[installation]-create-a-new-react-app.markdown)에서 가장 유명하고 이해하기 쉬운 툴체인을 배워 볼 수 있습니다. 아니라면, 그냥 script 태그로도 괜찮습니다.