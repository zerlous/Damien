---
title: "BEM规范生成工具类wxs实现"
date: 2021-05-18T15:16:01+08:00
tags: ["javascript","wxs"]
author: "Me"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Desc Text."
disableHLJS: true # to disable highlightjs
disableShare: false
disableHLJS: false
searchHidden: true
cover:
    image: "<image path/url>" # image path/url
    alt: "<alt text>" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: true # only hide on current single page
---

---
`array.wxs`
```Javascript
function isArray(array) {
  return array && array.constructor === 'Array';
}

module.exports.isArray = isArray;

```
--- 
`object.wxs` 对象keys数组化
```Javascript
var REGEXP = getRegExp('{|}|"', 'g');

function keys(obj) {
  return JSON.stringify(obj)
    .replace(REGEXP, '')
    .split(',')
    .map(function(item) {
      return item.split(':')[0];
    });
}

module.exports.keys = keys;
```
--- 
```Javascript
var array = require('./array.wxs');
var object = require('./object.wxs');
var PREFIX = 'van-';

function join(name, mods) {
  name = PREFIX + name;
  mods = mods.map(function(mod) {
    return name + '--' + mod;
  });
  mods.unshift(name);
  return mods.join(' ');
}

function traversing(mods, conf) {
  if (!conf) {
    return;
  }

  if (typeof conf === 'string' || typeof conf === 'number') {
    mods.push(conf);
  } else if (array.isArray(conf)) {
    conf.forEach(function(item) {
      traversing(mods, item);
    });
  } else if (typeof conf === 'object') {
    object.keys(conf).forEach(function(key) {
      conf[key] && mods.push(key);
    });
  }
}
// BEM规则返回classnames string
// bem('button', {disabled: true, readonly: false}) => return 'van-button van-button--disabled';
function bem(name, conf) {
  var mods = [];
  traversing(mods, conf);
  return join(name, mods);
}
```