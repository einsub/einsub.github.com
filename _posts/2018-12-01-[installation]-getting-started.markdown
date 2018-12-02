---
layout: post
title: "[번역] React 설명서, 설치편: 시작하기"
date: 2018-12-01 21:05:01
author: Reid
categories:
  - engineering
tags:
  - 프로그래밍
  - react
  - javascript
published: false
---
> 이 문서는 버전 16.6.3의 [React Docs](https://reactjs.org/docs/getting-started.html) 문서를 번역한 것입니다. 전문 번역이 아닌 개인 학습 목적의 작업 결과이므로 오역이 있을 수 있습니다. 그런 부분을 발견하시면 너그러히 이해해주시고 지적해주시면 고맙겠습니다.

# 시작하기
*이 페이지에서 우리는 React 문서와 관련 자료들에 대해 개략적으로 살펴볼 것입니다.*

**React**는 유저 인터페이스를 만드는 자바스크립트 라이브러리입니다. [홈페이지](https://reactjs.org/)나 [튜토리얼](https://reactjs.org/tutorial/tutorial.html)에서 React에 대한 모든 것들을 배워보세요.

---

- [React 한번 해보기](#react-한번-해보기)
- [React 배우기](#react-배우기)
- [소식 챙겨듣기](#소식-챙겨듣기)
- [버전 별 문서](#버전-별-문서)
- [뭔가 놓친게 있을까?](#뭔가-놓친게-있을까?)

---

## React 한번 해보기

React는 점진적으로 적용 할 수 있게 설계되었으므로, **필요한 만큼 쓰는 것이 가능**합니다. 여러분이 React를 살짝 맛만 보고 싶을 뿐이거나, 간단한 HTML 페이지에 약간의 상호작용을 추가해보고 싶다거나, 복잡한 React 앱을 만들어 보고 싶을 때, 어떤 경우라도 이 섹션에서 소개하는 링크들은 시작하기에 도움이 될 것입니다.

### 온라인 놀이터

React로 이것저것 만져보고 싶다면, 온라인 코딩 놀이터를 사용해보세요. [CodePen](https://reactjs.org/redirect-to-codepen/hello-world)이나 [CodeSandbox](https://codesandbox.io/s/new)에서 Hello World 템플릿을 실행해 볼 수 있습니다.

사용중인 텍스트 편집기를 선호하신다면, [HTML 파일](https://raw.githubusercontent.com/reactjs/reactjs.org/master/static/html/single-file-example.html)을 다운로드 하고 편집해서 여러분의 브라우저에서 직접 열어 볼 수도 있습니다. 이 방법은 런타임 코드 변환 속도가 느리기 때문에, 간단한 데모를 이용 할 때에만 추천합니다.

### React를 웹사이트에 추가하기

일분만에 HTML 페이지에 React를 추가 할 수 있습니다. 그 후에는 조금씩 더 그 영역을 확장해갈수도 있고, 그대로 몇개의 동적 위젯에 포함시키는 정도로 둘 수도 있습니다.
팁
### 새 React 앱 만들기

[스크립트 태그가 포함된 간단한 HTML 페이지](https://reactjs.org/docs/add-react-to-a-website.html)는 가장 훌륭한 React 프로젝트의 시작 방법입니다. 설정하는데 일분밖에 걸리지 않습니다.

애플리케이션이 커질수록 다른 기능들을 추가하고 통합하는 것이 필요합니다. 규모가 큰 애플리케이션에 적합한 여러 자바스크립트 툴체인이 있습니다. 이것들은 사용하면 별다른 설정 없이도 풍부한 React 생태계의 모든 장점을 그대로 얻을 수 있습니다.

---

## React 배우기

사람들은 모두 다른 환경, 다른 학습 방식을 가지고 있습니다. 이론 먼저 배우는 걸 선호할 수도 있고, 아예 처음부터 직접 뭔가 만들어보는 걸 선호할 수도 있습니다. 어떤 방법이라도 이 섹션은 여러분께 도움이 될 것입니다.

- 직접 해보는 걸 선호하신다면, [튜토리얼](https://reactjs.org/tutorial/tutorial.html)로 시작하세요.
- React의 개념들을 하나씩 배워보고 싶다면, [주요 컨셉 가이드](2018-12-01-[main-concepts]-hello-world.markdown)로 시작하세요.

낯선 기술들이 대부분 그렇듯, React도 러닝 커브(learning curve)가 있습니다. 인내심을 가지고 연습하시면, 결국 얻어낼 수 있을 것입니다.

### 첫번째 예제

[React 홈페이지](https://reactjs.org/)에는 라이브 편집기에 작성된 몇개의 작은 React 예제들이 있습니다. 지금 React에 대해서 아무것도 모른다해도, 그 예제의 코드를 바꿔보면서 결과가 어떻게 바뀌나 살펴 볼 수 있습니다.

### 초보자를 위한 React

이 React 문서가 여전히 이해하기 어렵다면, [Tania Rascia의 React 훑어보기](https://www.taniarascia.com/getting-started-with-react/)를 확인해보세요. 가장 중요한 React의 개념들을 초보자도 이해하기 쉽도록 상세히 소개하고 있습니다. 그 문서를 다 읽고 나서 다시 한번 이 문서를 읽어보세요.

### 디자이너를 위한 React

디자인을 하던 분이시라면, [이 자료들](http://reactfordesigners.com/)이야말로 시작하는데 가장 도움이 될 것입니다.

### 자바스크립트 자료들

이 문서는 여러분이 자바스크립트 언어에 어느정도 익숙하다고 가정합니다. 전문가일 필요는 없습니다만, 자바스크립트에 대한 이해가 부족하다면 React를 공부하면서 자바스크립트까지 공부해야 합니다. 이것은 무척 어려운 일입니다.

그런 분들께는 [이 자바스크립트 훑어보기](https://developer.mozilla.org/en-US/docs/Web/JavaScript/A_re-introduction_to_JavaScript)를 추천합니다. 대략 30분에서 1시간 정도의 시간을 투자해서, 자바스크립트에 대한 자신감을 회복 할 수 있습니다.

> Tip<br/>
자바스크립트가 헷갈리면, 언제든 [MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript)과 [javascript.info](http://javascript.info/)를 참고하세요. 더불어 [자바스크립트 커뮤니티](https://reactjs.org/community/support.html)에서 질문을 해볼 수도 있습니다.

### 튜토리얼

직접 작성하며 배우고 싶다면, [튜토리얼](https://reactjs.org/tutorial/tutorial.html)을 이용하세요. 이 튜토리얼에서 우리는 React로 tic-tac-toe 게임을 만들어 볼 것입니다. 게임을 만드는 것에 흥미가 없다고 지나쳐버리지 마시고 한번 확인해보세요. 튜토리얼에서 배운 기술들은 React를 깊이 이해하는데 크게 도움이 되고 모든 React 앱들의 기본이 됩니다.

### 단계별 가이드

React의 개념들을 단계별로 배워보고 싶다면, [주요 컨셉 가이드](2018-12-01-[main-concepts]-hello-world.markdown)가 가장 좋은 선택입니다. 챕터마다 이전 챕터의 기술을 기반으로 배워나가기 때문에 결국 모든 것들을 익힐 수 있게 됩니다.

### React 방식으로 생각하기

많은 React 사용자들이 [React 방식으로 생각히기](2018-12-01-[main-concepts]-thinking-in-react.markdown)가 그들에게 마지막 영감을 준 글이었다고 이야기합니다. 이것은 아마도 가장 오래된 React 참고서겠지만, 여전히 유효합니다.

### 추천 코스

때로 공식 문서보다 책이나 비디오를 선호하시는 분들도 있습니다. 그런 분들꼐서 참고하실 수 있는 무료 [추천 자료](https://reactjs.org/community/courses.html)를 제공합니다.

### 고급 개념들

[주요 개념들](2018-12-01-[main-concept]-hello-world.markdown#주요-개념들)에 익숙해져서 React를 조금 다루어 볼 수 있다면, 좀 더 고급의 주제를 살펴볼 시기입니다. 이 섹션은 [context](2018-12-01-[main-concepts]-context.markdown)나 [refs](2018-12-01-[main-concepts]-forwarding-refs.markdown)처럼 조금 덜 사용되지만, 강력한 React의 기능들을 소개합니다.

### API Reference

이 섹션은 특정 React API에 대해 더 자세하게 배워보고 싶을 때 유용합니다. 예를 들어, [React.Component API reference](https://reactjs.org/docs/react-component.html)는 `setState()`가 어떻게 동작하는지, 각각의 lifecycle 메소드들은 어디에 사용되는지 등을 설명하고 있습니다.

### 용어와 FAQ

[용어](https://reactjs.org/docs/glossary.html)는 React 문서에서 가장 자주 사용되는 구문들에 대한 설명을 확인할 수 있습니다. FAQ 섹션은 [AJAX request 만들기](https://reactjs.org/docs/faq-ajax.html), [컴포넌트의 state](https://reactjs.org/docs/faq-state.html), [파일 구조](https://reactjs.org/docs/faq-structure.html) 등의 주요 토픽들에 대한 짧은 질문과 답변을 담고 있습니다.

## 소식 챙겨듣기

[React 블로그](https://reactjs.org/blog/)는 React 팀의 공식 뉴스를 올리는 곳입니다. 새로운 릴리즈의 출시 소식과 deprecate 소식 등 중요한 뉴스들이 가장 먼저 올라옵니다.

트위터에서 [@reactjs 계정](https://twitter.com/reactjs)을 팔로우 할 수도 있습니다. 하지만 기본적으로 블로그를 읽으면 중요한 뉴스를 놓치는 일은 없을 것입니다.

블로그에 React의 모든 릴리즈들을 매번 올릴 수는 없습니다. 그런 자세한 변경 사항이 궁금할 때에는 [React 레포지토리의 CHANGELOG.md 파일](https://github.com/facebook/react/blob/master/CHANGELOG.md)나 [릴리즈 페이지](https://github.com/facebook/react)를 참고하세요.

---

## 버전 별 문서

이 문서는 언제나 React 최신 버전을 기반으로 업데이트됩니다. React 16 부터는 이전 버전의 문서는 [별도의 페이지](https://reactjs.org/versions)에서 확인 가능합니다. 이전 버전의 문서는 그 당시의 릴리즈에만 국한된다는 걸 기억하세요. 그 것들은 더 이상 업데이트 되지 않습니다.

---

## 뭔가 놓친게 있을까?

이 문서에서 뭔가 빠뜨린게 있따거나, 헷갈리는 부분이 있으면 [문서 레포지토리에 이슈](https://github.com/reactjs/reactjs.org/issues/new)를 만들어서 개선 의견을 올려주세요. 트위터 [@reactjs 계정](https://twitter.com/reactjs)으로 연락주셔도 됩니다. 여러분께 의견을 듣는 것은 언제나 환영입니다!