---
layout: post
title: "Text Normalized Search 정규화 검색"
date: 2019-04-14 23:18:15
author: Reid
categories:
  - engineering
tags:
  - mongodb
  - normalize
  - search
  - slugify
  - node.js
published: false
---

MongoDB는 Text Index라는 특수 인덱스를 제공하므로 이를 이용하여 손쉽게 문자열 검색을 지원 할 수 있습니다. Text Index는 가중치 부여, 와일드카드<sup>wildcard</sup> 검색, 대소문자 무시, 성조 무시, 다양한 언어셋 지원 등 문자열을 검색하기 위한 기본적인 기능들을 제공합니다. Text Index로 성조를 무시하며 검색하는 방법을 설명한 이전 글을 참고하세요.

[MongoDB Text Index로 발음구별기호가 있는 문자 검색하기](https://blog.ull.im/engineering/2019/03/17/search-diacritic-text-with-mongodb-text-index.html)

이번에는 Text Index 없이 간단하게 문자열을 검색 할 수 있는 정규화 검색에 대해 소개해 보겠습니다. 이 방법은 MongoDB의 Text Index를 사용 할 수 없다거나, 고급 문자열 검색 기능이 없는 기타 데이터베이스 등을 사용할 때 활용이 가능합니다.

# 아이디어

기본적인 구현 아이디어는 검색하고자 하는 문자열 필드와 함께 정규화 된 필드 하나를 더 마련해두고 이를 이용하여 검색을 수행하는 것 입니다. 정규화 된 필드는 알파벳 이외의 따옴표, 마침표, 성조 등의 특수 문자들이 제거되어 검색 시 적중 확률을 높여줍니다. `title` 이라는 필드를 검색 대상으로 한다면, 다음과 같은 절차로 정규화 검색이 진행됩니다.

1. 데이터베이스에 저장을 할 때, `title`에 들어갈 데이터를 정규화하여 `titleSlug` 등에 추가로 저장합니다.
2. 데이터베이스에서 `title`이 수정 될 때는 반드시 `titleSlug`도 함께 수정해줍니다.
3. 검색을 할 때 입력되는 `keyword`도 데이터베이스 쿼리를 하기 전 정규화를 시킵니다.
4. `keyword`를 `titleSlug`와 정규표현식으로 비교하여 검색을 수행합니다.

# 한계

구현이 간단하고 대체로 좋은 검색 기회를 제공하지만, 한계는 존재합니다. 여타 데이터베이스나 검색 엔진이 제공하는 다양한 검색 방법들은 전혀 사용 할 수 없습니다. 그러므로 검색어를 단어별로 쪼개서 검색하거나 유사어, 퍼지<sup>fuzzy</sup> 검색 등이 필요한 본문 검색 등에는 이 방법을 사용해서는 안됩니다. 카테고리 검색, 제목 검색 등 검색어가 한 단어면 충분한 텍스트 검색에 사용하는 것이 좋습니다.

# 튜토리얼

