---
title: JavaScript - Check If a String Contains a Substring
date: 2020-04-26T18:30:34.282666Z
description: How to check if a string contains a substring in JavaScript
---

There are basically two common ways to check if a string contains a substring:

```javascript
const str = 'A quick brown fox jumps over the lazy dog';
```

‌

## Modern JavaScript way \(ES6\)

```javascript
str.includes('fox'); // return true
str.includes('cat'); // returns false
```

This method was introduced in ES6 and hence [**is not available in some older browsers like Internet Explorer**](https://caniuse.com/#feat=es6-string-includes).  

## Alternative Way \(E65\)

This method is supported across all browsers.

```javascript
str.indexOf('fox'); // returns 14 the starting index of the substring
str.indexOf('cat'); // return -1 since it is not present
```

`indexOf` returns 0 or greater if the substring is present otherwise it returns -1.

## Case Insensitive

Both the methods are case-sensitive and if you want it to be case insensitive then make sure both the strings are in the same case by using the `toLowerCase()` or `toUpperCase()` method on them beforehand like this:

```javascript
str.includes('a '); // false because str has "A " and not "a "
str.toLowerCase().includes('a '); // true
```

‌
