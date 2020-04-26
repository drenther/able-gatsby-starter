---
title: How to Redirect With JavaScript
date: 2020-04-26T18:50:26.081608Z
description: In this article, we will explore various ways to redirect to a new URL from the frontend.
---

JavaScript can be used to redirect to a valid URL in multiple ways:

```javascript
const URL_VALUE = 'https://example.com';

// various examples
window.location = URL_VALUE;
window.location.href = URL_VALUE;
window.location.assign(URL_VALUE);
```

All of these three ways work exactly the same way. They can be thought of as redirected via a anchor link `<a href="https://example.com">` click. The URL of the document that caused the redirect stays in the browser history and hence the user can navigate back to it.

```javascript
window.location.replace(URL_VALUE);
```

This method works kind of like an HTTP Redirect because the URL of the document that caused the redirect is purged and as a result the user cannot navigate back to it.

## Redirect using HTML

Redirects can also be done within HTML without any JavaScript.

```html
<meta http-equiv="refresh" content="0; URL='https://example.com'" />
```

Adding this meta tag to the `<head>` of a document can redirect the page to given URL. The number in the content field followed by a semicolon determines the time interval before the redirect takes place. But this behaviour may not be well supported in all browsers and should hence be avoided unless absolutely necessary.

It works kind of the like the [**REFRESH HTTP Header**](http://www.otsukare.info/2015/03/26/refresh-http-header).