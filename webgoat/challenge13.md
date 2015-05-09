## Vulnerability:

sql injection

## Exploitation:

Following input modifies salary for user jsmith:
```
jsmith'; UPDATE salaries SET salary=2 WHERE userid='jsmith
```

## Mitigation:
1) validate input data

2) use prepared statements/stored procedures (https://www.owasp.org/index.php/SQL_Injection_Prevention_Cheat_Sheet) 
