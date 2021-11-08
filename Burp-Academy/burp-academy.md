
### Target

    https://portswigger.net/web-security/all-labs

### Lab: Forced OAuth profile linking

**Vulnerability**

| Vulnerable component: `OAuth client application`
| Exploitation technique: CSRF against `OAuth client application`
| Impact: `OAuth client application` account takeover

Implementation issue in OAuth client application. The application does not contain proper `nonce` parameter during `authorization request` making OAuth flow vulnerable to CSRF attack. Successful exploitation of the flaw allows to associate attacker's controlled account on the identity provider with a victim's account on OAuth client application resulting in an account takeover.

**Exploitation**

**Mitigation**

Implement and properly handle state (`nonce`) during OAuth `authorization request`:

    GET /authz?client_id=ID&response_type=CODE&scope=SCOPE&redirect_uri=REDIRECT_URI&state=NONCE
