---
layout: post
title: "[번역] React 설명서, 주요 개념: Hello World"
date: 2018-12-02 22:13:47
author: Reid
categories:
  - engineering
tags:
  - 프로그래밍
  - react
  - javascript
published: false
---
> 이 문서는 버전 16.6.3의 [React Docs](https://reactjs.org/docs/hello-world.html) 문서를 번역한 것입니다. 전문 번역이 아닌 개인 학습 목적의 작업 결과이므로 오역이 있을 수 있습니다. 그런 부분을 발견하시면 너그러히 이해해주시고 지적해주시면 고맙겠습니다.

# Hello World

*가장 작은 React 예제는 이렇게 생겼습니다:*

```jsx
ReactDOM.render(
  <h1>Hello, world!</h1>,
  document.getElementById('root')
)
```

이 예제는 페이지에 "Hello, world!"를 출력합니다.

[CodePen에서 실행하기](https://reactjs.org/redirect-to-codepen/hello-world)

위 링크를 눌러서 온라인 편집기를 띄워보세요. 마음대로 바꿔보고 그게 결과에 어떤 영향을 미치는지 확인합니다. 이 가이드 대부분의 페이지에는 이렇게 수정 가능한 예제들을 담고 있습니다.

---

## 이 가이드를 읽는 방법

이 가이드에서 우리는 React 앱의 빌딩 블록을 다루게 됩니다. 그것은 바로 element와 component입니다. 이것들을 마스터하고 나면, 작고 재사용 가능한 조각들로 복잡한 앱도 만들 수 있게 됩니다.

> Tip<br/>
이 가이드는 **차근차근 개념을 이해하며 배우는**것을 선호하는 사람들에게 적합합니다. 직접 해보면서 배우길 원하는 사람들은 [튜토리얼](https://reactjs.org/tutorial/tutorial.html)을 읽어보세요. 이 가이드와 튜토리얼은 서로 상호 보완재가 될 수 있습니다.

주요 React의 개념을 배우는 단계별 가이드의 첫번째 챕터입니다. 네비게이션 사이드바에서 모든 챕터들의 목록을 살펴 볼 수 있습니다. 모바일 기기에서 읽는다면, 화면의 오른쪽 하단 버튼을 눌러 네비게이션 하실 수 있습니다.

이 가이드의 모든 챕터는 이전 챕터의 지식을 기반으로 만들어집니다. **React에 대한 거의 모든 것들은 이 "주요 개념들"에서 배우게 되며 이 챕터들은 사이드바에 순서대로 보여집니다**. 예를 들어, ["JSX 소개"](2018-12-01-[main-concepts]-introducing-jsx.markdown)는 이 챕터 다음에 있는 것 처럼 말이죠.