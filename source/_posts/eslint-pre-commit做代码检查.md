---
title: eslint+pre-commit做代码检查
date: 2016-05-23 17:44:22
tags: eslint git hooks
---

前端团队大到5+人数时，代码的规范就会变得很重要。介绍下目前我们的使用的方案，eslint结合pre-commit工具，提交代码时自动做代码检查。

### 工具

- [eslint](http://eslint.org/) 代码检查工具
- [pre-commit](https://github.com/observing/pre-commit) pre-commit钩子辅助工具

<!-- more -->

### 配置

#### eslint
按你们团队规范配置即可，我们用的是eslint-config-standard

.eslintrc

```
{
  "extends": "standard"
}
```

#### pre-commit

package.json

```
{
  ...
  "devDependencies": {
    "eslint": "^2.4.0",
    "eslint-config-standard": "^5.1.0",
    "eslint-plugin-promise": "^1.1.0",
    "eslint-plugin-standard": "^1.3.2",
    "pre-commit": "^1.1.2"
  },
  "scripts": {
    "eslint": "eslint $(git diff-index --cached HEAD --name-only --diff-filter ACMR | egrep '.js$')"
  },
  "pre-commit": ["eslint"],
  "precommit.silent": true
}
```

每次commit都只会检查此次commit包含的js文件。如果不符合规范，则无法提交成功。
