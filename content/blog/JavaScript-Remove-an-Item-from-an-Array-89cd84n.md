---
title: JavaScript - Remove an Item from an Array
date: 2020-06-17T17:54:08.380551Z
description: Various ways to remove elements from a JavaScript Array.
---

There are multiple ways to remove a particular element from an Array in JavaScript. We will explore the most commonly used ways.

First, let's define our array:

```javascript
const ourArray = [1, 2, 3, 4, 5, 6];
```

## **Remove last item from array**

### using `pop` method

```javascript
ourArray.pop();
// ourArray = [1, 2, 3, 4, 5]
```

‌

## **Remove first item from array**

### Using `shift` method

```javascript
// ourArray = [1, 2, 3, 4, 5]
ourArray.shift();
// ourArray = [2, 3, 4, 5]
```

‌

## **Remove it based on its index**

### **Using `splice` method**

```javascript
// ourArray = [2, 3, 4, 5]
// remove from 1st index and remove only one element
const removedPart = ourArray.splice(1, 1)
// removedPart = [3]
// ourArray = [2, 4, 5]
```

The `splice` method takes the starting `index` \(from where the removal should start\) as the first argument and `count` \(the number of elements to be removed\) as the second argument. The `splice` can be used for a lot of other things that we will explore some other time.

In our case, we wanted only the element at 1 index to be removed so we passed `index = 1` as well as the `count = 1`.

### **Using `filter` method**

```javascript
const toBeDeletedIndex = 1;

const indexFilteredArray = ourArray.filter(function(element, index) {
  return index !== toBeDeletedIndex;
});
// newArray !== ourArray
// ourArray = [2, 4, 5]
// newArray = [2, 5]


const oddValuesArray = ourArray.filter(function(element) {
  return element % 2 !== 0;
});
// oddValuesArray = [5]
// ourArray = [2, 3, 4]


// shorter version using arrow functions
const evenValuesArray = ourArray.filter( element => element % 2 === 0 );
// evenValuesArray = [2, 4]
// ourArray = [2, 4, 5]
```

The `filter` method takes a callback function \(also called a predicate function in this case\). The callback function is called for each element in the array is passed the following three argument values `element` \(the element itself\), `index` \(index of the current element\) and `array` \(the array itself\).

The return value \(`true` or `false`\) of each of these callback invocation determines whether the element will be kept in the array or not. If the callback returns `false` for an element, then the element is not included in the new array.

Unlike, `pop`, `shift`, and `splice`, the `filter` method returns a new array and doesn’t mutate the original array.