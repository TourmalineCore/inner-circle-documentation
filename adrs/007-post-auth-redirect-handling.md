# 007: Post-Authorization Redirect Handling

## Status

Accepted (2026-07-24)

## Context

Currently, after successful authorization, users are redirected to ```/employees``` regardless of their original destination. This creates a poor user experience, particularly in scenarios like QR code scanning where users expect to land on a specific resource page after authentication.


## Desicion

Implement a **returnUrl** query parameter system with security validations to redirect users to their intended destination after login.

### Technical Implementation

1. **Authentication flow changes**

***auth-ui service***: modify the ```useAuthenticated.ts``` file to use returnUrl from query parameters. Replace ```window.location.href = /employees``` with ```window.location.href = returnUrl || "/"```.

2. **URL validation** to prevent Open Redirect vulnerabilities (probably in auth-ui) 

```
function isSafeReturnUrl(returnUrl) {
  try {
    const url = new URL(returnUrl, window.location.origin)
    return url.origin === window.location.origin
  } catch {
    return false
  }
}
```

If validation fails, fall back to "/".

**Example attack: unsafe redirect after login**

A web app redirects the user to the URL in the returnUrl query parameter after login, without checking it.

The victim sees malicious a link ```http://localhost:30090/auth?returnUrl=http://attacker-site.com/fake-login``` that starts with our own host, so they trust it. Without validation, after login the app redirects them to the attacker's site, which can copy our UI and steal the victim's credentials.

With ```isSafeReturnUrl``` added:
```
isSafeReturnUrl("http://attacker-site.com/fake-login")
// new URL(...).origin === "http://attacker-site.com"
// "http://attacker-site.com" !== "http://localhost:30090" -> false
// falls back to "/"
```

A legitimate case, for example, returning to the book copy page after login via QR scan, passes validation and works correctly:

```
isSafeReturnUrl("http://localhost:30090/copy/1?s=2222")
// origin "http://localhost:30090" === "http://localhost:30090" -> true
```

Our site runs on a single host, so it's enough to check that the redirect stays on that host.

#### Alternative
Using ```startsWith``` instead of ```origin```

An earlier version of this check used ```returnUrl.startsWith("http://localhost:30090/")```

#### Advantages:
- trailing slash blocks the obvious domain-spoofing trick (```http://localhost:30090.bad-domain.com```)

#### Disadvatnages: 
- plain string-prefix check is still weaker than parsing the URL and comparing origins
- browsers always lowercase the scheme and host when parsing a URL, but ```startsWith``` does not:

```
new URL('HTTP://LocalHost:30090/employees').origin === 'http://localhost:30090' // -> true -> /employees
'HTTP://LocalHost:30090/employees'.startsWith('http://localhost:30090/')  // -> false -> /
```

3. **Nested redirect protection**

In each UI service, replace ```window.location.href = /auth``` with ```window.location.href = `/auth?returnUrl=${encodeURIComponent(window.location.href)}` ``` in the ```RequireAccessToken``` file.

Why ```encodeURIComponent``` is required? It preserves multi-parameter URLs.

Without ```encodeURIComponent```:

```http://localhost:30090/auth?returnUrl=http://localhost:30090/books?copyId=1&s=2222```. Here ```?copyId=1&s=2222``` would be truncated without proper encoding.

With ```encodeURIComponent```:

```
`http://localhost:30090/auth?returnUrl=${encodeURIComponent(window.location.href)}`
// http://localhost:30090/auth?returnUrl=http%3A%2F%2Flocalhost%3A30090%2Fbooks%2Fcopy%2F1%3FcopyId%3D1%26s%3D2epq
```
The ```&``` inside the value is now encoded (%26), so it's not confused with the outer query parameters, and auth-ui restores the full original URL correctly.
