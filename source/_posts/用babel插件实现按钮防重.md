---
title: 用babel插件实现按钮防重
date: 2022-04-04 15:45:05
tags: babel
---

业务中总是会出现按钮多次点击，导致接口被重复请求的情况。而目前的解决方案，可归纳为以下两种

### 从组件入手
输出一个通用的组件，处理该问题

```jsx
import React, { useState } from 'react';
import { Button } from 'antd';

export default function PromiseButton(props) {
  const [loading, setLoading] = useState(false);

  const onClick = async (e) => {
    if (typeof props.onClick === 'function') {
      setLoading(true);
      try {
        await props.onClick(e);
      } finally {
        setLoading(false);
      }
    }
  };

  return (
    <Button
      {...props}
      loading={loading || props.loading}
      disabled={loading || props.disabled}
      onClick={onClick}
    />
  );
}
```
在使用到Button的地方，统一用PromiseButton组件替代即可

这个方案的问题在于
1. 场景比较局限，实际业务中并不是只有按钮点击才会触发接口请求
2. 比较难统一，如何保证其他人，其他团队会用这个组件？也许可以用eslint做提示，但比较难推广

### 从请求库入手

一般团队里都会有统一的请求库，可以从请求库入手。举个最简单的例子

```js
const set = new Set();
async function post(url, data) {
  const key = `${url}_${JSON.stringify(data)}`;
  
  if (set.has(key)) {
    return;
  }
  
  set.add(key)
  const res = await fetch(url, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(data)
  }
  set.delete(key)
  return res;
}
```

以请求路径和请求参数作为一个接口请求的唯一标识，利用Set做接口请求状态的管理

这个方案的问题在于无法对多个请求做防重

```jsx
import React from 'react';
import post from './post';

function Test() {
  const handleClick = async () => {
  
    const a = await post('/a', {});
    
    const b = await post('/b', {});
  }

  return <button onClick={handleClick}>接口请求</button>
}
```

比如以上这种情况，预期的是对整个`handleClick`做防重，在`/a`, `/b` 接口都完成前，都不再发送请求。但请求库只能支持对`/a`和`/b`做单独的防重


### babel插件方案

在一次思考babel插件处理async函数的过程中，我突然想到，能不能用类似的思路，给代码里的每一个async方法，都加一个防重处理呢？

```js
// 源代码
const onClick = async () => {
  console.log('async function')
  await post('/a', {});
}

// 经babel插件处理后的代码
function _lock$(fn) {
  let locking = false;

  const _lockedFn$ = async function (...args) {
    if (locking) {
      return;
    }

    try {
      locking = true;
      const r = await fn.apply(this, args);
      return r;
    } catch (e) {
      throw e;
    } finally {
      locking = false;
    }
  };

  return _lockedFn$;
}

const onClick = _lock$(async () => {
  console.log('async function')
  await post('/a', {});
});
```

这个方案很好的解决了原来两个方案的问题

1. 对开发者无感，业务开发不需要想着每个用Button的地方都要用PromiseButton替代
2. 覆盖到所有使用async函数的地方
3. 支持对多个请求做防重


插件源代码奉上[babel-plugin-async-lock](https://github.com/lili21/babel-plugin-async-lock)

