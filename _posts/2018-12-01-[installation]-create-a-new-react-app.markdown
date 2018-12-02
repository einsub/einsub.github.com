---
layout: post
title: "[번역] React 설명서, 설치편: 새 React 앱 만들기"
date: 2018-12-02 20:16:02
author: Reid
categories:
  - engineering
tags:
  - 프로그래밍
  - react
  - javascript
published: false
---
> 이 문서는 버전 16.6.3의 [React Docs](https://reactjs.org/docs/create-a-new-react-app.html) 문서를 번역한 것입니다. 전문 번역이 아닌 개인 학습 목적의 작업 결과이므로 오역이 있을 수 있습니다. 그런 부분을 발견하시면 너그러히 이해해주시고 지적해주시면 고맙겠습니다.

# 새 React 앱 만들기

*최고의 유저와 개발자 경험을 제공하는 통합 툴체인*

이 페이지는 다음과 같은 일들을 돕는 유명한 툴체인들을 설명합니다.

- 다수의 파일들과 컴포넌트들을 스케일링하기
- npm의 third-party 라이브러리 사용하기
- 일반적인 실수를 조기에 찾아내기
- 개발 환경에서 CSS와 JS를 실시간 수정하기
- 프로덕션 결과물을 최적화하기

여기서 소개하는 툴체인들은 **특별한 설정이 필요 없습니다**.

---

## 어쩌면 툴체인이 필요 없을지도 모릅니다

위에 적은 내용을 문제점으로 인식하지 못했거나, 아직까지 자바스크립트 도구를 이용하는데 크게 불편한 점을 느끼지 못했다면 [HTML 페이지에 `<script>` 태그를 사용하는 정도](2018-12-01-[installation]-add-react-to-a-website.markdown), 그리고 선택적으로 [JSX](2018-12-01-[installation]-add-react-to-a-website.markdown#옵션-jsx로-react-작성해보기)를 사용하는 정도로 React를 이용하는 것이 바람직합니다.

이것이 **기존 웹사이트에 React를 적용하는 가장 쉬운 방법**입니다. 앞으로도 언제나 도움이 될 것 같다고 판단이 될 때 툴체인을 고려해볼 수 있습니다.

---

## 추천 툴체인

React 팀은 우선적으로 다음 솔루션들을 추천합니다:

- **React를 배우는 중**이거나 **single-page app**을 만들고 있다면, [Create React App](#create-react-app)을 사용하세요.
- **Node.js로 서버 렌더링 웹사이트**를 만들고 있다면, [Next.js](#nextjs)를 사용하세요.
- **정적 컨텐츠 기반의 웹사이트**를 만들고 있다면, [Gatsby](#gatsby)를 사용하세요.
- **컴포넌트 라이브러리**나 **기존 코드베이스를 통합**하고 있다면, [보다 유연한 툴체인](#more-flexible-toolchains)를 사용하세요.

### Create React App

[Create React App](http://github.com/facebookincubator/create-react-app)은 **React를 배우기에** 편리한 환경이며, **React로 새로운 single-page 애플리케이션을 만드는** 최적의 선택입니다.

최신 자바스크립트 기능들을 이용 할 수 있도록 개발 환경을 설정해주고, 멋진 개발자 경험을 제공하며, 프로덕션에 배포 할 수 있도록 앱을 최적화해줍니다. Node 6.0 이상과 npm 5.2 이상이 설치되어 있어야 합니다. 프로젝트를 생성하기 위해서 다음을 실행합니다:

```bash
npx create-react-app my-app
cd my-app
npm start
```

> 참고<br/>
npx는 오타가 아닙니다. [npm 5.2+에서 제공하는 패키지 구동 도구](https://medium.com/@maybekatz/introducing-npx-an-npm-package-runner-55f7d4bd282b)입니다.

Create React App은 백엔드 로직이나 데이터베이스는 다루지 않습니다; 다만 프론트엔드의 빌드 파이프라인을 만들어 주기 때문에, 여러분이 원하는 어떤 백엔드와도 함께 사용 할 수 있습니다. 내부적으로는, [Babel](http://babeljs.io/)과 [webpack](https://webpack.js.org/)을 사용하지만 여러분이 이에 대해 자세히 알 필요는 없습니다.

프로덕션에 배포 할 때에는, `npm run build`를 실행하세요. 이 명령은 여러분의 앱을 최적으로 빌드하여 `build` 폴더에 만들어줍니다. 좀 더 배워보고 싶은 분들은 Create React App의 [README](https://github.com/facebookincubator/create-react-app#create-react-app-)와 [사용자 가이드](https://github.com/facebookincubator/create-react-app/blob/master/packages/react-scripts/template/README.md#table-of-contents)를 참고하세요.

### Next.js

[Next.js](https://nextjs.org/)는 React로 **정적이고 서버 렌더링 애플리케이션**을 만드는 널리 알려진 가벼운 프레임워크입니다. 이 도구는 즉시 사용 가능한 **스타일링, 라우팅 솔루션**을 제공합니다. 서버 환경으로 [Node.js](https://nodejs.org/)를 사용합니다.

### Gatsby

[Gatsby](https://www.gatsbyjs.org/)는 React로 **정적 웹사이트**를 만드는 최고의 방법입니다. React 컴포넌트를 사용하지만, 미리 렌더링된 HTML과 CSS를 결과물로 만들어내기 때문에 빠른 로딩 타임을 보장합니다.

[공식 가이드](https://www.gatsbyjs.org/docs/)와 [스타터 키트 갤러리](https://www.gatsbyjs.org/docs/gatsby-starters/)에서 Gatsby를 배워보세요.

### 그 외 유연한 툴체인들

다음 툴체인들은 보다 유연하고 다른 선택들을 제공합니다. 좀 더 경험이 많은 사용자에게 추천합니다:

- [Neutrino](https://neutrinojs.org/)는 [webpack](https://webpack.js.org/)의 강력함과 프리셋의 단순함을 결합하고, [React 앱](https://neutrinojs.org/packages/react/)과 [React 컴포넌트](https://neutrinojs.org/packages/react-components/) 프리셋들을 포함하고 있습니다.
- [nwb](https://github.com/insin/nwb)는 특히 [npm에 React 컴포넌트를 퍼블리시](https://github.com/insin/nwb/blob/master/docs/guides/ReactComponents.md#developing-react-components-and-libraries-with-nwb)할 때 유용합니다. React 앱을 만들 때도 물론 사용 할 수 있습니다.
- [Parcel](https://parceljs.org/)은 [React와 동작](https://parceljs.org/recipes.html#react)하는 빠르고 무설정의 웹 애플리케이션 bundler입니다.
- [Razzle](https://github.com/jaredpalmer/razzle)은 아무런 설정도 필요없는 서버 렌더링 프레임워크로서, Next.js보다 더 유연함을 제공합니다.

---

## 툴체인 만들어보기

자바스크립트 빌드 툴체인은 보통 다음과 같이 구성됩니다:

- Yarn이나 npm과 같은 **패키지 매니저**. 거대한 third-party 패키지 생태계의 장점을 제공합니다. 쉽게 인스톨하고 업데이트 할 수 있습니다.
- [webpack](https://webpack.js.org/)이나 [Parcel](https://parceljs.org/)과 같은 **bundler**. 모듈 코드를 작성해서 작은 패키지에 번들로 함께 묶어 최적화된 로딩 타임을 갖게 해줍니다.
- [Babel](http://babeljs.io/)과 같은 **컴파일러**. 구 브라우저에서도 돌아가는 최신 자바스크립트 코드를 작성 할 수 있게 해줍니다.

아무것도 없는 상태에서 직접 자바스크립트 툴체인을 만들어서 Create React App 같은 기능을 만들어 보고 싶다면, [이 가이드](https://blog.usejournal.com/creating-a-react-app-from-scratch-f3c693b84658)가 도움이 될 것입니다.

여러분이 만든 툴체인이 [프로덕션 환경에서 올바르게 설정](2018-12-01-[advanced-guides]-optimizing-performance.markdown#프로덕션-빌드-사용하기)될 수 있을지 주의하세요.