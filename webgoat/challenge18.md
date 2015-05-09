## Vulnerability:

While observing AuthCookie for users:

```
webgoat: 65432ubphcfx
aspect: 65432udfqtb
```

We can easily see that AuthCookie is generated from usernames using following formula:

1) 65432 is fixed prefix

2) second part is Caesar cipher with right shift of 1, so 'a' becomes 'b', 'c' becomes' 'd' and so on; but reading characters from end to the beginning

So for username alice AuthCookie is equal to: `65432fdjmb`

## Exploitation:

I used ZAP to intercept request & change AuthCookie cookie to 65432fdjmb and I was recognized as alice by the backend.

## Mitigation:

It shouldn't be possible to calculate AuthCookie for user B knowing AuthCookie cookie for A. Randomness should be incorporated in generating AuthCookie.
