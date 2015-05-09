
## Analysis of the given RBAC system:

Roles:
[Public]
[User]
[Manager]
[Admin]

Users of the system:
Moe
Larry
Curly
Shemp

Resources:
Public Share (ps)
Time Card Entry (tce)
Performance Review (pr)
Time Card Approval (tca)
Site Manager (sm)
Account Manager (am)

Role assignment looks like this:
Moe is assigned to: [Public]
Larry is assigned to role: [User], [Manager]
Curly is assigned to role: [Public], [Manager]
Shemp is assigned to role: [Admin]

After reviewing allowed access for each user, we can build following role permissions matrix:

Public role has access to: ps
User role has access to: tce, am
Manager role has access to: pr, tca
Admin role has access to: sm, am

## Vulnerability:

System policy states that:
Only the [Admin] group should have access to the 'Account Manager' resource.

There is an issue with [User] role. Larry which is assigned to this role is allowed to access restricted resource 'Account Manager' which is against system policy.

## Exploitation:
If user will be assigned to [User] role he will be able to access restricted 'Account Manager' resource.

## Mitigation:
[User] role shouldn't give access to 'Account Manager'. Role permissions matrix should be fixed. 
