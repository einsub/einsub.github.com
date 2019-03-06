---
layout: post
title: "VS Code로 node.js 애플리케이션을 디버깅 할 때의 소소한 팁"
date: 2019-03-06 16:57:24
author: Reid
categories:
  - engineering
tags:
  - nodejs
  - vscode
  - debugging
published: true
---

[VS Code](https://code.visualstudio.com/)로 node.js 애플리케이션을 디버깅 할 때, 브레이크포인트를 잡아 라인 단위로 진행을 시키다보면 async 함수를 지나갈 때 어김없이 node의 `emitHookFactory` 함수로 프로세스가 이동됩니다. 비동기 처리를 하다보니 어쩔 수 없는 일이지만, 이게 디버깅을 심각하게 방해하죠.

바로 요놈.

``` javascript
function emitHookFactory(symbol, name) {
  // Called from native. The asyncId stack handling is taken care of there
  // before this is called.
  // eslint-disable-next-line func-style
  const fn = function(asyncId) {
    active_hooks.call_depth += 1;
    // Use a single try/catch for all hook to avoid setting up one per
    // iteration.
    try {
      for (var i = 0; i < active_hooks.array.length; i++) {
        if (typeof active_hooks.array[i][symbol] === 'function') {
          active_hooks.array[i][symbol](asyncId);
        }
      }
	...
```

디버깅을 할 때 node의 내부 코드들은 Step into 되지 않고 건너 뛰도록 설정을 변경하면 더 이상 괴롭지 않습니다. `VS Code`의 launch 설정 파일에 다음과 같이 `skipFiles`를 설정합니다. **Debug/Open Configurations**에서 `launch.json` 파일을 찾으실 수 있습니다. 코어하게 node 코드도 함께 디버깅을 해야 할 때는 이 설정을 지우셔야 합니다.

```json
"skipFiles": [
    "<node_internals>/**"
]
```

해피 디버깅!

