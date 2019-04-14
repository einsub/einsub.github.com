---
layout: post
title: "JavaScript 함수형 프로그래밍 3단계로 설명하기"
date: 2019-04-07 18:18:15
author: Reid
categories:
  - engineering
tags:
  - javascript
  - funcitonal programming
  - pure functions
  - immutability
  - declarative patterns
published: true
---

> 원문: [https://medium.com/@alexnault/functional-programming-with-javascript-in-3-tips-f282934947e5](https://medium.com/@alexnault/functional-programming-with-javascript-in-3-tips-f282934947e5) - [Alex Nault](https://medium.com/@alexnault)

**순수 함수, 불변성 그리고 선언전 패턴... 분명 이것들이 좋아지실 겁니다**

함수형 프로그래밍은 1930년대로 거슬러 올라간 수학 개념 인 람다<sup>lambda</sup> 미적 분학에 그 뿌리를 두고 있습니다. 수학과 친숙하지 않은 분들은 기겁하실 수도 있지만, 그러지 않으셔도 됩니다. 수학 이론 없이도 **몇 가지 원칙들을 통해 함수형 프로그래밍의 놀라운 점과 그 소프트웨어가 갖는 이점을 소개하려고 합니다.**

자 시작해 봅시다!

# 순수 함수<sup>pure functions</sup>

함수는 다음의 경우에 순수하다고 할 수 있습니다.

- 동일한 입력에 대해 항상 동일한 출력을 반환합니다.

그리고

- 부작용이 없습니다.

**즉, 순수 함수는 입력과 출력을 매핑 시킬 수 있습니다.**

함수를 호출하는 쪽과 그 순수 함수는 인자와 반환값을 통해서만 서로 통신이 가능합니다.

```javascript
// 순수하지 않습니다 (반환값 없이 부가 효과를 이용하고 있습니다)
function addTaco(array) {
   array.push("taco");
}

// 순수하지 않습니다 (인자 대신 공유 변수를 이용하고 있습니다)
function addTaco() {
   return [...globalArray, "taco"];
}

// 순수합니다
function addTaco(array) {
   return [...array, "taco"];
}
```

**순수 함수는 예측이 가능하고, 디버그가 쉬우며 테스트하는 것은 더욱 쉽습니다.** 그들의 [참조 투명성](https://ko.wikipedia.org/wiki/%EC%B0%B8%EC%A1%B0_%ED%88%AC%EB%AA%85%EC%84%B1)<sup>referentially transparent</sup> 덕분에, 함수 결과를 캐싱하여 반복적으로 사용 할 수 있는 [메모이제이션](https://ko.wikipedia.org/wiki/%EB%A9%94%EB%AA%A8%EC%9D%B4%EC%A0%9C%EC%9D%B4%EC%85%98)<sup>memoization</sup> 최적화 기법이 사용 가능합니다.

모든 함수가 순수 할 수는 없습니다. I/O작업, 난수 처리, 네트워킹, 데이터베이스 등을 처리 할 때는 본질적으로 순수 할 수 없습니다. 그럼에도 불구하고 함수가 다룰 수 있는 영역을 더 넓게 하고, 핵심을 간단하게 유지하기 위해 이러한 컨셉은 소프트웨어의 전반에 걸쳐 적용되어야 합니다.

# 불변성<sup>immutability</sup>

JavaScript에서 모든 원시 자료형(boolean, string 등)은 불변합니다. 객체나 배열의 경우는 그렇지 않지만, 이들 역시 마치 그런 것 처럼 다룰 수 있습니다.

**불변한 객체나 배열은 만들어진 후에는 더 이상 수정 할 수 없습니다.**

수정하고 싶다면, 복제해서 새로운 변수에 할당해야 합니다.

```javascript
// mutable
const bands = ["Metallica", "Queen"];
bands.push("Nirvana");

// immutable
const someBands = ["Metallica", "Queen"];
const bands = [...someBands, "Nirvana"];
```

이것들은 오직 단 하나의 상태만을 가지고 있기 때문에, **불변한 객체나 배열은 그 내용을 변경해서 무효화 시킬 수가 없고 이로 인해 부작용을 막을 수 있습니다.**

불변 데이터는 **[스레드로부터 안전](https://ko.wikipedia.org/wiki/%EC%8A%A4%EB%A0%88%EB%93%9C_%EC%95%88%EC%A0%84)<sup>thread-safe</sup>**하고 **[오류에 대해 국소적](https://stackoverflow.com/questions/29842845/what-is-failure-atomicity-used-by-j-bloch-and-how-its-beneficial-in-terms-of-i/29843251#29843251)<sup>failure atomic</sup>**입니다.

> 역주: 'failure atomic'은 그대로 번역하면 '오류가 원자적'이라고 해야겠지만, 이해가 어려울 것 같아서 나름대로 의역을 했습니다. 순수 함수의 오류가 원본 데이터에 전혀 영향을 주지 않기 때문에 오류가 국소적으로 격리되어 있다는 뜻으로 이해할 수 있겠습니다.

보너스: 불변 객체나 배열은 "변경" 여부를 알아내기 위해 깊은<sup>deep</sup> 비교를 할 필요가 없습니다. 단지 참조에 대한 검사만 해도 되기 때문에, 효율적으로 상태를 관리해야 하는 상황(예를 들어, React 애플리케이션)에서 빠른 변경 감지를 가능하게 하는 주요 역할을 합니다.

# 선언적 패턴<sup>declarative patterns</sup>

- 선언적<sup>declarative</sup> 프로그래밍은 **우리가 이루고자 하는 것**을 설명합니다.
- 명령형<sup>imperative</sup> 프로그래밍은 **우리가 그것을 이루는 방법**을 설명합니다.

즉, 명령형 프로그래밍은 단계 별 할 일을 명시적으로 지시하는 반면, 선언적 프로그래밍은 일련의 선언들로 구성됩니다.

```javascript
const names = ["Han", "Chewbacca", "Luke", "Leia"];

// 명령형 패턴
const shortNames = [];
for (let i = 0; i < names.length; i++) {
   if (names[i].length < 5) {
      shortNames.push(names[i]);
   }
}

// 선언적 패턴
const shortNames = names.filter(name => name.length < 5);
```

**선언적 패턴은 세부적인 구현 방법이 아니라 다루는 문제의 도메인에 초점을 맞춥니다.** 이 패턴은 보통 더 간결하고, 더 에러가 적은 코드를 생상합니다. 또한, 이해하기 훨씬 쉽습니다.

그런데 여러분은 이 과정에서 `shortNames`가 어떻게 변하지 않는지 알고 있나요?

JavaScript는 [`map`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map), [`filter`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/filter), [`reduce`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/reduce)등의 유용한 [고차<sup>higher-order functions</sup> 함수](https://ko.wikipedia.org/wiki/%EA%B3%A0%EC%B0%A8_%ED%95%A8%EC%88%98)들을 가지고 있습니다. 선언적 코드를 작성할 때 필수적이니 광범위하게 사용하세요.

# 마무리

여러분의 도구함에 함수형 프로그래밍을 담아 둘 수 있습니다! 이제 여러분은 이 원칙들이 여러분에게 잘 맞는지 살펴 볼 수 있습니다. 프로젝트에 적용해서 효과가 있는지, 그렇지 않은지를 확인하십시요.

전반적으로 이 글은 간신히 표면을 긁어 본 정도에 지나지 않지만, 함수형 프로그래밍에 대한 조금의 이해를 드리고 여러분의 호기심을 자극할 수 있었기를 바랍니다.

질문이 있으면 주저하지 마세요. 최선을 다해 답해 드리겠습니다!
