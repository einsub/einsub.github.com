---
layout: post
title: "[번역] React 설명서, 설치편: CDN 링크"
date: 2018-12-02 21:55:47
author: Reid
categories:
  - engineering
tags:
  - 프로그래밍
  - react
  - javascript
published: false
---
> 이 문서는 버전 16.6.3의 [React Docs](https://reactjs.org/docs/cdn-links.html) 문서를 번역한 것입니다. 전문 번역이 아닌 개인 학습 목적의 작업 결과이므로 오역이 있을 수 있습니다. 그런 부분을 발견하시면 너그러히 이해해주시고 지적해주시면 고맙겠습니다.

# CDN 링크

*React와 ReactDOM은 CDN을 통해 사용 가능합니다.*

```html
<script crossorigin src="https://unpkg.com/react@16/umd/react.development.js"></script>
<script crossorigin src="https://unpkg.com/react-dom@16/umd/react-dom.development.js"></script>
```

위는 개발용 버전이므로 프로덕션에 적합하지 않습니다. 프로덕션용으로 minify된 최적의 React 버전은 다음과 같습니다:

```html
<script crossorigin src="https://unpkg.com/react@16/umd/react.production.min.js"></script>
<script crossorigin src="https://unpkg.com/react-dom@16/umd/react-dom.production.min.js"></script>
```

`react`와 `react-dom`의 특정 버전을 로드하려면, `16`을 특정 버전 번호로 바꾸면 됩니다.

## crossorigin 속성은 왜 붙었을까?

CDN을 통해 React를 제공할 때, crossorigin 속성을 유지하세요:

```html
<script crossorigin src="..."></script>
```

사용 중인 CDN이 `Access-Control-Allow-Origin: *` HTTP 헤더를 설정하는지 확인하세요:

<img style="display: block; margin-left: auto; margin-right: auto" src="https://reactjs.org/static/cdn-cors-header-89baed0a6540f29e954065ce04661048-dd807.png">

이 방법은 React 16과 그 이후 버전에서 더 나은 [에러 처리](https://reactjs.org/blog/2017/07/26/error-handling-in-react-16.html)를 돕습니다.