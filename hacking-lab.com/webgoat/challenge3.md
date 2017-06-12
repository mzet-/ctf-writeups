## Vulnerability:

Following line has been found in site's source:
```
<!-- FIXME admin:password1234 --><!-- Use Admin to regenerate database -->
```

## Exploitation:
Use found credentials to login - you gained administrator privileges.

## Mitigation:
1) Do not leave credentials (and any other sensitive informations) in source code.

2) use stronger passwords :) - this one could be also easily brute-forced. 
