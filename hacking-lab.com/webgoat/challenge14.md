## Vulnerability:

sql injection

## Exploitation:

Following input adds record to calaries table:
```
jsmith'; INSERT INTO salaries (userid, salary) VALUES ('mzet', 999999) --
```

## Mitigation:
1) validate input data

2) use prepared statements/stored procedures (https://www.owasp.org/index.php/SQL_Injection_Prevention_Cheat_Sheet) 
