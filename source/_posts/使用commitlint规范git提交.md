---
title: 使用commitlint规范git提交
date: 2018-02-05 22:06:11
tags:
    - JavaScript
    - git
categories: Notes
photos:
---



[commitlint](http://marionebl.github.io/commitlint/): git 提交信息规范与验证

[husky](https://github.com/typicode/husky/tree/master): 使ghook更容易

[standard-version](https://github.com/conventional-changelog/standard-version): 自动生成CHANGELOG 并发布版本



安装

```
npm install --save-dev @commitlint/{config-conventional,cli}
npm i --save-dev standard-version
npm install husky --save-dev
```

#### 配置

1. commitlint

```
echo "module.exports = {extends: ['@commitlint/config-conventional']}" > commitlint.config.js
// commitlint.config.js
 module.exports = {
  extends: ['@commitlint/config-conventional'],
  rules: {
  'type-enum': [2, 'always', [
     "feat", "fix", "docs", "style", "refactor", "perf", "test", "build", "ci", "chore", "revert"
   ]],
  'scope-empty': [2, 'never'],
  'subject-full-stop': [0, 'never'], 
  'subject-case': [0, 'never']
  }}; 

```

2. tandard-version 和 husky

```
// package.json
 "scripts": {
  "lint": "eslint .",
  "commitmsg": "commitlint -e $GIT_PARAMS",
  "release": "standard-version",
  "validate": "npm prune",
  "pre-commit": "npm run lint",
  "pre-push": "npm run validate",
  "npmi": "npm i",
  "post-merge": "npm run npmi",
  "post-rewrite": "npm run npmi"
 }
```