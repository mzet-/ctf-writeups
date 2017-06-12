## Vulnerability:

This is classical sql injection vulnerability where user supplied input via form is used to build sql query.

## Exploitation:

by submitting following input:
```
Smith' OR 1=1 --
```

we can retrieve all DB records.

## Mitigation:
1) validate input data
2) use prepared statements/stored procedures (https://www.owasp.org/index.php/SQL_Injection_Prevention_Cheat_Sheet) 
